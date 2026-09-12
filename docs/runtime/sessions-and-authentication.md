# Sessions and authentication

A session is the server-side state that turns a sequence of unrelated network messages into one conversation with one user. This document specifies how a session is created, what it contains, where it is stored, how its identifier is rotated, how it expires, how it is bound to a user by a derived token, how devices are recorded and revoked, the four authentication levels an endpoint may declare, the interactive login procedure with its second factor, the non-interactive credential, the re-authentication windows that force a user to prove their identity again, and the cross-site request forgery token derived from the session.

Everything here is observable behaviour. The routing and dispatch that surround it are in [`request-lifecycle.md`](request-lifecycle.md); the transport spellings of the cookie and of the login paths are in [`../interfaces/`](../interfaces/); the permission model the acting user is evaluated against is in [`../overview/security-model.md`](../overview/security-model.md).

## 1. Identity of a session

### 1.1 The identifier

A session identifier is 84 characters drawn from the web-safe alphabet consisting of the 26 lowercase letters, the 26 uppercase letters, the 10 digits, the hyphen and the underscore. It is produced as follows.

1. Build the seed: the decimal text of the current wall-clock instant, as bytes, followed by 64 random bytes.
2. Take the 64-byte digest of the seed with the 512-bit member of the standard secure hash family.
3. Drop the last byte, leaving 63 bytes, which is a multiple of 3.
4. Encode those 63 bytes in the web-safe base-64 alphabet, giving 84 characters and no padding.

Dropping the last byte makes the encoded length a multiple of four and therefore removes the padding characters, which keeps the identifier usable as a file name and inside a cookie. The 63-byte pre-image gives about 218 bits of entropy even under the pessimistic assumption of a case-insensitive storage layer, which makes a collision negligible.

```formula
entropy in bits = 63 bytes × 8 bits ÷ byte = 504 bits before encoding
usable entropy under a case-insensitive store = 84 characters × log2( 38 distinct symbols )
                                              = 84 × 5.2479 = 440.8 bits
```

The pessimistic figure quoted above is the conservative reading of the same computation restricted to the 63-byte pre-image and the birthday bound, which is half of 504 bits reduced by the alphabet collapse; either way the value is far beyond the point at which a collision is a practical concern.

### 1.2 The stable prefix

The **first 42 characters** of the identifier are the *stable prefix*. They are preserved across a soft rotation, specified in section 5.2. Two things depend on this.

- The cross-site request forgery token is derived from the stable prefix, so a token minted before a soft rotation stays valid after it, as specified in section 10.
- The device record stores only the stable prefix, so the device history of a browser survives soft rotations and a device revocation can delete every session record that shares the prefix, as specified in section 7.

A value is accepted as a stable prefix only if it is exactly 42 characters of the same web-safe alphabet. Any other value is refused with `Identifier format incorrect, did you pass in a string instead of a list?`, which prevents a crafted device record from being used to delete the session records of another database.

### 1.3 Storage layout

Sessions are stored one per record, under a directory chosen by the deployment. The record name is the full identifier. It lives in a sub-directory named after the **first two characters** of the identifier, which scatters sessions across at most 4 096 sub-directories. The sub-directory is created on demand with owner-write, group-read and world-read permissions. A missing sub-directory on read is not an error: it means the session does not exist.

A replacement is free to use another medium provided it preserves five properties: retrieval by identifier; enumeration of the identifiers present under a given stable prefix; deletion by stable prefix; deletion by age of last modification; and the fact that an identifier that is well formed but unknown yields an empty session rather than an error.

## 2. Content of a session

| Key | Type | Default | Meaning |
|---|---|---|---|
| `context` | Nested key-value document | empty, then the language is filled in | The execution context carried across requests: the user's language, the user's time zone, the acting user key. |
| `create_time` | Instant | the moment of creation | The basis of the soft-rotation schedule and of the absolute re-authentication window. |
| `db` | Text | none | The database the session is bound to. |
| `debug` | Text | empty | The comma-separated development flags in force, for example the asset flag. |
| `login` | Text | none | The login of the authenticated user. |
| `uid` | Integer | none | The key of the authenticated user. |
| `session_token` | Text | none | The derived binding token of section 6. |
| `_trace` | List of device traces | empty list | The device traces of section 7.1. |

Additional keys written by specific features, each specified where it is used:

| Key | Written by | Meaning |
|---|---|---|
| `pre_login`, `pre_uid` | The first step of an interactive login | A partially authenticated session awaiting a second factor, specified in section 8.5. |
| `next_sid`, `deletion_time` | A soft rotation | The successor identifier and the instant after which the predecessor must be refused, specified in section 5.2. |
| `gc_previous_sessions` | A soft rotation | Marks the successor session as owing a cleanup of its predecessors, specified in section 5.3. |
| `identity-check-last` | A successful identity re-check | The instant of the last identity confirmation; it opens the 600-second grace window of section 9.1. |
| `identity-check-next` | The inactivity watcher | The instant at which the inactivity re-authentication becomes due, specified in section 9.3. |
| `identity-check-1fa` | A two-step identity re-check | The pair of instant and method already provided, which prevents the same factor from being reused as the second one, specified in section 9.4. |
| `profile_session`, `profile_expiration`, `profile_collectors`, `profile_params` | The profiling toggle | The profiling configuration; see [`logging-and-audit.md`](logging-and-audit.md), section 7. |
| `is_websocket_session` | The notification polling fallback | Marks that the client has already performed its first poll, which lets the server detect an expired session on later polls; see [`notification-bus.md`](notification-bus.md), section 9. |
| `_trace_disable` | A privileged technical caller | Suppresses device tracing for automated sessions. No unprivileged path can set it. |

Three flags live on the session object but are never stored.

| Flag | Meaning |
|---|---|
| Dirty | Set whenever a stored key is added, removed or changed to a different value. It drives persistence, as specified in section 4. |
| Persistable | Starts true; set to false when the database was selected by the request header instead of by the cookie, and reduced by logical conjunction with each endpoint's save-session declaration. A non-persistable session is never written and never produces a cookie. |
| Rotation pending | Set by a logout and by the finalization of a login. It forces a hard rotation at the next persistence point. |

Values are normalized on assignment by a round trip through the structured-data representation. Two consequences follow: only structured-data-representable values can be stored, and a value that compares equal to the previous one does not mark the session dirty.

## 3. Creating and loading a session

1. If the request carries no session cookie, or the cookie value is not 84 characters of the web-safe alphabet, create a new empty session with a freshly generated identifier.
2. Otherwise load the stored session for that identifier. If nothing is stored, return an empty session and force it to carry the requested identifier, so that a well-formed unknown cookie is silently adopted rather than rejected.
3. Give every key of section 2 that is missing its default.
4. If the session context has no language, write the request language preference into it; the resolution is in [`request-lifecycle.md`](request-lifecycle.md), section 11.
5. Resolve the database as specified in [`request-lifecycle.md`](request-lifecycle.md), section 5.
6. Clear the dirty flag, so that steps 3 and 4 alone never cause a write.

## 4. Persistence

The persistence decision is evaluated once per request, during post-dispatch, in this exact order.

| Order | Condition | Action |
|---|---|---|
| 1 | The session is not persistable. | Nothing at all: no write, no cookie. |
| 2 | A rotation is pending. | Hard rotation, specified in section 5.1, which itself writes the session. |
| 3 | There is an acting user, the current instant is at least the creation instant plus 10 800 seconds, which is 3 hours, the request path is not one of the three rotation-excluded paths, and the skip-rotation request header is absent. | Soft rotation, specified in section 5.2, which itself writes the session. |
| 4 | The session is dirty. | Write the session. |
| 5 | Otherwise. | Nothing. |

Then, if the session is dirty **or** the cookie value received differs from the current identifier, a session cookie is emitted carrying the current identifier, marked inaccessible to client scripts, with a maximum age equal to the effective inactivity limit of section 11.

The three rotation-excluded paths are the persistent-socket close notification path, the notification polling fallback path and the presence update path. They are excluded because clients call them at high frequency from background workers, and rotating on them would produce a cookie race with the foreground tab.

A caller may force a write of an otherwise unchanged session by marking it dirty explicitly. The interactive login pages do this so that a partially authenticated session survives the redirect to the second-factor page.

## 5. Rotation

### 5.1 Hard rotation

Used at logout, at the finalization of a login, and whenever a rotation is pending.

1. Delete the stored session under the current identifier.
2. Generate a brand-new identifier, unrelated to the previous one.
3. If the session has an acting user, recompute the binding token for the new identifier, as specified in section 6.
4. Clear the rotation-pending flag and set the creation instant to the current instant.
5. Store the session under the new identifier.

Because the stable prefix changes, any cross-site token minted for the previous identifier stops validating, and the device history restarts under a new prefix.

### 5.2 Soft rotation

Used automatically every 10 800 seconds of session age, for sessions that carry an acting user. Its purpose is to bound the lifetime of a stolen cookie while keeping the tokens and the device history stable.

1. Take the stable prefix, which is the first 42 characters of the current identifier.
2. Re-read the stored record for the current identifier, because another request may have rotated already.
3. If the stored record carries a successor identifier, adopt it as the current identifier and stop. No second successor is created.
4. Otherwise build the successor as the stable prefix followed by the last 42 characters of a freshly generated identifier.
5. On the **current** record, set the successor identifier and set the deletion instant to the current instant plus 120 seconds. Write the current record.
6. Mark the in-memory session as owing a predecessor cleanup.
7. Set the current identifier to the successor, and remove the deletion instant and the successor identifier from the in-memory session.
8. Perform the tail shared with the hard rotation: when the session has an acting user, recompute the binding token for the new identifier; clear the rotation-pending flag; set the creation instant to the current instant; write the session under the successor identifier.

Four consequences must be reproduced.

- The successor shares its first 42 characters with its predecessor and differs in the last 42.
- The predecessor record remains readable for 120 seconds and carries a deletion instant. During that window, requests that still send the old cookie are recognised: their session load returns the predecessor record, whose successor key lets the persistent socket and the polling fallback follow the chain to the successor. The mechanism is in [`notification-bus.md`](notification-bus.md), section 7.
- After the deletion instant has passed, the predecessor record is refused: the session check fails and the request is treated as expired.
- Two requests arriving at the same instant with the same old cookie create only one successor, because the second one finds the successor key already present.

### 5.3 Cleanup of predecessors

At the start of every session check, if the session carries the cleanup marker and 120 seconds have passed since its creation instant, every stored session whose record name begins with the stable prefix is deleted, the marker is removed, and the session is written again under its own identifier. The re-write is required because the deletion also removes the current session's own record, which shares the prefix.

This bounds the number of records sharing a prefix to the current one plus those created in the last 120 seconds.

## 6. The binding token

A session that names a user is valid only while it also carries a token derived from that user's security-relevant columns. The token is what makes a password change, a deactivation or a rename invalidate every existing session of that user immediately, everywhere, without any scan of the session store.

### 6.1 Computation

The inputs are five labelled pairs, in this exact order:

| Label | Value |
|---|---|
| The database-secret label | The value of the system parameter `database.secret`. |
| The active label | The user's active flag. |
| The identifier label | The user's key. |
| The login label | The user's login. |
| The password label | The user's stored password verifier. |

1. Remove every pair whose value is unset.
2. Take the textual representation of the remaining tuple of pairs, as bytes; that is the key.
3. Compute the keyed digest of the session identifier, as bytes, under that key, using the 256-bit member of the standard secure hash family.
4. Render the digest in hexadecimal.

The four user columns are exactly the columns declared as session-token columns. A capability package may declare more, and each addition changes the derived token for every user that has a value in the new column. The label of each pair is a fixed literal: the labels take part in the derivation, so renaming one invalidates every session. The reference order is the database secret first, then the user columns sorted by label. Dropping unset values means that adding a new nullable security column does not invalidate existing sessions until that column receives a value for a given user.

If the query that reads those columns returns anything other than exactly one row, which happens when the user was deleted between two requests, every registry cache container is cleared and the computation yields no token at all, which makes the check of section 6.2 fail.

The computation is memoized per session identifier in the registry's default cache container, catalogued in [`caching.md`](caching.md), section 11.

### 6.2 Verification

The verification runs at the start of the authentication step for every request whose session names a user.

1. Perform the predecessor cleanup of section 5.3.
2. If the session carries a deletion instant that has already passed, the check fails.
3. Recompute the expected token for the session's user and identifier.
4. If there is no expected token, the check fails.
5. Compare the expected token with the stored one using a constant-time comparison. On equality, record the device activity of section 7 and the check succeeds. Otherwise the check fails.

On failure the session is logged out while keeping the database, the execution context is rebuilt with no acting user, and the endpoint's declared authentication level is then applied, which for the level requiring a user raises a session-expired failure.

### 6.3 What invalidates tokens and caches

Writing any of the following user columns clears every registry cache container of the database, which drops the memoized tokens and forces recomputation: `group_ids`, `active`, `lang`, `tz`, `company_id`, `company_ids`, `id`, `login` and `password`.

Deleting a user clears every registry cache container as well, and so does removing an application key.

Because the password verifier is one of the token columns, a password change invalidates every session of that user. There is one exception: when the stored verifier is silently upgraded to a stronger derivation during a successful check, the current session's token is recomputed and rewritten in place, so the user who performed the change is not logged out.

## 7. Devices

### 7.1 The device trace

Each session carries a list of device traces. A trace has five members: the platform name, the browser name, the client address, the first-activity instant and the last-activity instant.

On every successful session check the trace list is consulted.

| Case | Action | Returned |
|---|---|---|
| The session carries the trace-suppression flag. | none | nothing |
| A trace matches the request's platform, browser **and** client address, and its last activity is less than 3 600 seconds old. | none | nothing |
| A trace matches on those three members and its last activity is 3 600 seconds old or more. | Update the last-activity instant and mark the session dirty. | the updated trace |
| No trace matches. | Append a new trace with both instants set to the current instant and mark the session dirty. | the new trace |

When a trace is returned, a **User Device Log** record is inserted. The insert is unconditional, so the log is an append-only history with at most one row per hour per combination of session prefix, platform, browser and address.

### 7.2 The User Device Log entity

| Identifier | Full name | Type | Required | Meaning |
|---|---|---|---|---|
| `session_identifier` | Session Identifier | Text, indexed | yes | The 42-character stable prefix of the session. |
| `platform` | Platform | Text | no | The operating system reported by the client. |
| `browser` | Browser | Text | no | The client program reported by the client. |
| `ip_address` | Address | Text | no | The client address. |
| `country` | Country | Text | no | The country resolved from the client address. |
| `city` | City | Text | no | The city resolved from the client address. |
| `device_type` | Device Type | Selection: `computer`, `mobile` | no | `mobile` when the platform is one of `android`, `iphone`, `ipad`, `ipod`, `blackberry`, `windows phone` or `webos`, compared without regard to case, and `computer` otherwise. |
| `user_id` | User | Many-to-one to User, indexed | no | The acting user of the session. |
| `first_activity` | First Activity | Date and time | no | The first activity of this trace. |
| `last_activity` | Last Activity | Date and time, indexed | no | The last activity of this trace. |
| `revoked` | Revoked | Boolean | no | True when the corresponding session record no longer exists. |

Two computed, non-stored fields exist: a flag telling whether the record designates the session making the current request, which is true when the current session identifier begins with the record's stable prefix, and the line-separated list of every distinct client address seen for the same triple of prefix, platform and browser.

Two indexes exist: a composite index over the user, the session identifier, the platform, the browser, the last activity and the key, restricted to non-revoked rows; and an index over the revoked flag, also restricted to non-revoked rows.

The insert is performed on a read/write connection even when the request itself is served on a read-only one: the read-only transaction is rolled back and a separate read/write transaction is opened for the insert. The escalation is specified in [`request-lifecycle.md`](request-lifecycle.md), section 7.

### 7.3 The User Device projection

A read-only projection over the log keeps, for each group of user, session prefix, platform and browser, the single most recent non-revoked row, breaking ties on the larger key. Rows marked revoked are excluded. The default ordering is by last activity, most recent first. When the request has a session, an explicit ordering on the current-device flag is translated into an ordering on equality with the current stable prefix, descending, which surfaces the user's own device first.

### 7.4 Revocation

Revoking a set of devices, which is guarded by the identity check of section 9.1:

1. Collect the distinct stable prefixes of the selected devices.
2. Delete every stored session whose record name begins with one of those prefixes.
3. Mark every log row carrying one of those prefixes as revoked.
4. Log `User <key> revokes devices (<prefixes>)`, with the acting user key and the prefixes substituted.
5. If any of the selected devices is the current one, log the current session out.

### 7.5 Automatic maintenance

Three cleanups run inside the automatic cleanup job of [`scheduled-jobs.md`](scheduled-jobs.md), section 11.

| Cleanup | Rule |
|---|---|
| Device log compaction | Delete every log row for which another row exists with the same prefix, platform, browser and address and a strictly greater last activity. When the job supplies the previous run instant, the deletion is restricted to the groups whose surviving row is at least that recent. |
| Revocation marking | In batches of 100 000, select non-revoked rows whose last activity is older than the effective inactivity limit, ask the session store which of their prefixes have no stored session left, mark those rows revoked, and commit after each batch. |
| Session vacuum | Delete every stored session whose record has not been modified for longer than the effective inactivity limit. |

The session vacuum is the mechanism that enforces the inactivity limit: a session record is rewritten on every request that persists it, so its modification instant is its last-use instant.

## 8. Authentication

### 8.1 The four levels

Every endpoint declares exactly one level. The default, when nothing is declared, is the level requiring a user.

| Level | Precondition | Acting user during the call | Notes |
|---|---|---|---|
| `none` | none | none | The execution context is rebuilt with no acting user. Record operations outside abstract entities are unsafe at this level. This is the only level published when no database is resolved. |
| `public` | none | The session's user when there is one, otherwise the shared public user. | The access rules of the public user apply. |
| `user` | The session must name a user that is neither absent nor the public user. | The session's user. | Otherwise a session-expired failure carrying the text `Session expired` is raised. |
| `bearer` | An application key in the authorization header, or an existing user session together with browser navigation headers. | The key's owner, or the session's user. | Detailed in section 8.3. Endpoints at this level default to not saving the session. |

A cross-origin preflight request is always treated as the level `none`, whatever the endpoint declares, because the browser sends it without credentials.

### 8.2 The authentication step

1. If the session names a user and the binding-token check of section 6.2 fails, log the session out keeping the database and rebuild the execution context with no acting user.
2. Apply the level-specific rule of section 8.1.

Access denials, session-expired failures and protocol failures propagate unchanged. Any other failure inside this step is logged at informational level with its traceback under the message `Exception during request Authentication.` and is converted into a generic access denial, which prevents an internal failure from leaking information through an unauthenticated path.

### 8.3 The application-key level in detail

1. Read the value that follows the bearer scheme in the authorization header, matching the scheme name without regard to case.
2. If a token is present: resolve it to a user for the global scope. If there is no such user, refuse with the unauthorized status, the message `Invalid apikey` and an authentication challenge naming the bearer scheme. If the session already names a different user, refuse with an access denial carrying the message `Session user does not match the used apikey.` Otherwise make the owner the acting user and mark the session non-persistable.
3. Else, if the session names no user, refuse with the unauthorized status, the message "User not authenticated, use an API Key with a Bearer Authorization header." and an authentication challenge naming the bearer scheme.
4. Else, if the request is not a top-level browser navigation, refuse with the unauthorized status, the message `Missing "Authorization" or Sec-headers for interactive usage.` and an authentication challenge naming the bearer scheme.
5. Finally, apply the rule of the level requiring a user.

A top-level browser navigation means that all four request metadata headers are present with these exact values: the destination header is `document`, the mode header is `navigate`, the site header is `none` or `same-origin`, and the user-activation header is `?1`. This is the cross-site request forgery protection for this level: a cross-origin form submission cannot forge those headers.

### 8.4 Application keys

An application key is a long-lived bearer credential owned by one user.

| Property | Value |
|---|---|
| Generated value | 20 random bytes, rendered as 40 hexadecimal characters. |
| Stored form | A derived verifier using a password-based derivation with 6 000 iterations. The clear value is returned once, at creation, and never again. |
| Lookup index | The first 8 hexadecimal characters of the key, stored in clear and indexed together with the owner. |
| Scope | Optional. A key with no scope is global and satisfies any scope; a key with a scope satisfies only that scope. The request layer asks for the scope named `rpc`, which no key ever carries, so only global keys authenticate requests. |
| Expiration | Optional instant. A key whose expiration has passed never matches. |

Verification selects the rows whose index matches, whose owner is active, whose scope is compatible and whose expiration is unset or in the future; verifies the presented key against each candidate verifier; and returns the first owner that matches, or no owner.

Creation rules:

- A user who is not privileged must supply an expiration; otherwise the creation is refused with "The API key must have an expiration date".
- The expiration may not exceed the maximum duration granted by the user's access groups, expressed in days and taken as the largest value among the groups the user belongs to, defaulting to 1 day when every group declares zero. Exceeding it is refused with `You cannot exceed %(duration)s days.`, whose placeholder is that maximum in days.
- An expiration in the past is refused with `You cannot set an expiration date in the past.`
- Only an internal user may create a key at all; otherwise the refusal is "Only internal users can create API keys".
- Creating a key programmatically from another key requires either system privileges or the system parameter `base.enable_programmatic_api_keys` evaluated as true; otherwise the refusal is "Programmatic API keys are not enabled". The number of live keys a user may hold when creating programmatically is capped by the system parameter `base.programmatic_api_keys_limit`, whose default is 10; exceeding it is refused with "Limit of <limit> API keys is reached for programmatic creation". A non-integer value of that parameter is logged and the default is used.
- A scoped key may only generate credentials for its own scope; a global key may generate credentials for any scope. A key that does not belong to the current user, or is otherwise invalid, is refused with "The provided API key is invalid or does not belong to the current user.", and a key that cannot be resolved at all with "The provided API key is invalid."

On the creation form, an expiration that fails the rules above is reported as a notification whose title is "The API key duration is not correct." and whose message is the refusal text. A successful creation opens a window titled "API Key Ready" showing the clear value once.

Removal is guarded by the identity check of section 9.1 and is permitted only for one's own keys or by a system user; otherwise the refusal is "You can not remove API keys unless they're yours or you are a system user". A removal logs the scopes, the acting user and the client address, clears every registry cache container, and closes the window. Expired keys are deleted by the automatic cleanup job.

### 8.5 Interactive login

The credential presented is a key-value document whose type member selects the method. The platform defines the password method; capability packages add others, such as a time-based code, a hardware authenticator or an impersonation token. The password method carries the login and the password.

1. Enter the login cooldown guard for the presented login. The guard trips after the number of consecutive failures given by the system parameter `base.login_cooldown_after`, default 5, and holds for the number of seconds given by `base.login_cooldown_duration`, default 60.
2. Find the single user whose login matches, using the entity's default ordering.
3. If there is none, fail with an access denial.
4. Evaluate the credentials as that user, in an environment marked interactive.
5. If the client supplied a recognised time zone in its dedicated cookie, and the user has no time zone or has never logged in, write that time zone on the user.
6. Create one User Login Log record.
7. Leave the cooldown guard: on failure increment the failure count and record the instant; on success drop the entry.

A failure logs `Login failed for login:<login> from <address>` and a success logs `Login successful for login:<login> from <address>`, with the login and the client address substituted.

Evaluating a password credential:

1. The credential must be of the password type and carry a non-empty password; otherwise the denial is immediate.
2. Read the stored verifier for that user with a direct statement rather than through the record cache, because this must work while the entity definitions are being rebuilt.
3. Verify and, when the derivation is outdated, produce a replacement verifier. If a replacement was produced, write it, flush, clear every registry cache container and, when the check concerns the session's own user, recompute and rewrite the session binding token, which keeps the user logged in.
4. On success return the authentication outcome: the user key, the method name `password`, and the second-factor policy `default`.
5. On failure, and only for non-interactive evaluations, try the presented value as an application key of the global scope; on a match return the outcome with the method name `apikey`.
6. Otherwise deny.

The authentication outcome carries a second-factor policy with three possible values: `skip`, never ask for a second factor; `default`, ask when the user has one configured; and `enforce`, always ask.

After a successful evaluation the caller clears the acting user of the session, writes the presented login and the resolved user key as the pending pair, and then, when the policy is `skip` or the user has no second factor configured, finalizes the session.

**Finalization** removes the pending pair, reads the user's context, which is the language, the time zone and the acting user key, marks a rotation pending, and writes in one step the database name, the login, the acting user, the context and the freshly computed binding token. Because a rotation is pending, the identifier changes completely at the next persistence point, which defeats session fixation.

When the session is not finalized, the browser is redirected to the second-factor page. That page behaves as follows.

- It redirects to the client entry point when the session is already complete.
- It redirects to the login page when there is no partially authenticated session.
- On a plain visit, it looks for the trusted-device cookie; when its value resolves, through the application-key verification, to a key of the scope `browser` belonging to the pending user, the session is finalized immediately and the user is redirected onward.
- On submission, it enters the login cooldown guard, evaluates the second-factor credential, and on success finalizes the session.
- On success with the remember flag set, it creates a key of the scope `browser` named with the browser and the platform, with the city and the country appended in parentheses when the address resolves, whose lifetime in days is the system parameter `auth_totp.trusted_device_age`, defaulting to 90 days when the parameter is absent, non-positive or unparseable, and writes it into a cookie inaccessible to client scripts with the same maximum age and a same-site policy of `Lax`.
- It marks the session dirty in every branch, which lets the partially authenticated state survive the redirect.

An invalid second-factor value reports the message returned by the evaluation; a value that is not a well-formed number reports `Invalid authentication code format.`

Additionally, at a successful interactive login of a user who belongs to the settings administration group, the public base address of the deployment is guessed from the request and written into the system parameter `web.base.url`, unless the parameter `web.base.url.freeze` is set. A failure of that write is logged with its traceback and never fails the login.

### 8.6 Password rules

Passwords are stored as derived verifiers, never in clear. The derivation is a password-based key derivation using the 512-bit member of the standard secure hash family, with a number of iterations equal to the greater of 600 000 and the integer value of the system parameter `password.hashing.rounds`, whose default is 0 and which therefore yields 600 000. A stored verifier produced by a weaker or older scheme still verifies, and triggers the silent upgrade of section 8.5.

Changing one's own password requires presenting the current one; an absent current password is denied outright. An empty new password is refused with `Setting empty passwords is not allowed for security reasons!` Every password change logs `Password change for <login> (#<key>) by <actor login> (#<actor key>) from <address>`.

### 8.7 Non-interactive credential check

The generic remote-call service authenticates every call by the pair of user key and secret. The pair is memoized, so repeated calls with the same pair do not re-derive the verifier. An empty secret is denied immediately. An inactive user is denied. The evaluation is performed in a non-interactive environment, which enables the application-key fallback of section 8.5, step 5.

## 9. Re-authentication windows

### 9.1 The identity check

Certain operations are guarded: revoking devices, removing an application key, creating an application key, and any operation a capability package marks as sensitive. The guard behaves as follows.

1. If the session's last identity confirmation is more recent than the current instant minus 600 seconds, run the guarded operation immediately.
2. Otherwise create a User Identity Check Wizard record holding the pending call, which is the structured-data-representable part of the execution context, the entity name, the record keys, the operation name, and the positional and named arguments, and open it in a modal window titled `Access Control`.

The wizard asks for a credential. On success it sets the last-confirmation instant to the current instant and replays the stored call. On failure it reports `Incorrect Password, try again or click on Forgot Password to reset your password.` Only operations explicitly marked as guarded can be replayed by the wizard, which prevents the wizard from becoming a generic operation invoker.

The window is therefore 600 seconds wide and is refreshed by each successful confirmation.

### 9.2 Group-driven timeouts

An access group may declare four values.

| Identifier | Full name | Type | Meaning |
|---|---|---|---|
| `lock_timeout` | Lock Timeout | Integer, minutes | The absolute session lifetime after which a full re-login is required, regardless of activity. |
| `lock_timeout_mfa` | Lock Timeout Requires Second Factor | Boolean | Whether that re-login must use a second factor. |
| `lock_timeout_inactivity` | Inactivity Lock Timeout | Integer, minutes | The idle time after which an identity confirmation is required. |
| `lock_timeout_inactivity_mfa` | Inactivity Lock Requires Second Factor | Boolean | Whether that confirmation must use a second factor. |

The effective timeouts of a user are derived from every group the user belongs to, directly or by implication, independently for each of the two families:

1. Collect the pairs of minutes and second-factor flag for every implied group with a non-zero value.
2. Take the shortest number of minutes among the pairs that require a second factor, if any.
3. Take the shortest number of minutes among the pairs that do not, if any.
4. If a shortest second-factor value exists, add the pair of that value in seconds and the flag true.
5. If a shortest plain value exists, and either no shortest second-factor value exists or the plain value is strictly smaller than it, add the pair of that value in seconds and the flag false.
6. Sort the result ascending by seconds.

The result is a list, not a single value, because both variants can be in force with different lengths: a shorter one that does not require a second factor and a longer one that does. The list is memoized per user group set and is dropped whenever any of the four values is created, written or deleted. The memoized container is catalogued in [`caching.md`](caching.md), section 11.

**Worked example.** A user belongs to three groups. The first declares an inactivity timeout of 30 minutes without a second factor, the second declares 15 minutes with a second factor, and the third declares 60 minutes with a second factor. The shortest second-factor value is 15 minutes; the shortest plain value is 30 minutes. Because 30 is not strictly smaller than 15, the plain variant is dropped. The result is a single pair: 900 seconds requiring a second factor.

Had the first group declared 10 minutes instead of 30, the result would be two pairs: 600 seconds without a second factor and 900 seconds with one, in that order.

Suggested defaults when a user enables these settings in the interface are 1 440 minutes, one day, with a second factor required for the absolute timeout, and 15 minutes without a second factor for the inactivity timeout.

### 9.3 Evaluating the timeouts

At the authentication step, for endpoints of the level requiring a user with a session that names a user, the two families are evaluated in turn.

| Family | Outcome | Session key read | Default when unset | Subtracted first timeout |
|---|---|---|---|---|
| Absolute | Logout | The creation instant | 0 | 0 |
| Inactivity | Identity check | The inactivity stamp | unset | The shortest inactivity timeout |

For each family, and for each pair of the family's list taken longest first: compute the threshold as the current instant minus the pair's seconds; read the stamp, using the default when the key is unset; and if the stamp is set and the stamp minus the subtracted first timeout is at or before the threshold, produce the family's outcome together with the pair's second-factor flag and, when it is recent, the already-used factor recorded in the session.

The subtraction of the shortest inactivity timeout when evaluating a longer one exists because only the shortest inactivity timeout ever writes the inactivity stamp; a longer variant must therefore measure from that same stamp.

| Outcome | Effect |
|---|---|
| Logout | A session-expired failure is raised with the text `User <key> needs to login again`. The page family turns it into a redirect to the login page; the remote-call families report it in their envelope. |
| Identity check | An identity-check failure is raised with the text `User <key> needs to confirm his identity`, unless the endpoint declares that it is exempt from the identity check. It is logged at debug level only. The page family redirects to the identity-confirmation page carrying the original path and query as the return address; the remote-call families report it, which lets the client open a confirmation dialog. |

### 9.4 The two-step confirmation

When a confirmation is required and the timeout demands a second factor:

1. With no credential supplied, the server answers with the user key, the login and the list of the user's available authentication methods, minus the method already used as a first factor if that use is still within the window.
2. A credential whose type is not among the user's available methods is denied.
3. A time-based code is normalized by removing every whitespace character and reading the remainder as a number before evaluation.
4. On success, if a first factor was already recorded and the method differs, the recorded first factor is cleared and the confirmation is complete.
5. Otherwise, if the outcome's policy is not `skip`, the user has more than one method, and the timeout demands a second factor, the pair of current instant and method is recorded as the first factor, that method is removed from the offered list, and the client is told a second factor is required.
6. When the confirmation completes, the inactivity stamp is removed and the last-confirmation instant is set to the current instant, which also opens the 600-second grace window of section 9.1.

### 9.5 Feeding the inactivity stamp

The inactivity stamp is written from the presence signal carried on the notification transport, not from request traffic, because an idle desktop client still issues background requests.

1. Take the idle duration the client reported, converted from milliseconds to seconds.
2. Take the user's shortest inactivity timeout, in seconds.
3. The session counts as inactive when that timeout is set and either the caller forced it or the idle duration is at least the timeout.
4. When inactive, compute the candidate stamp as the current instant plus the timeout minus the idle duration. If the stamp is unset, or the candidate is earlier than the stored stamp, write the candidate and write the session immediately.
5. When not inactive, if the stamp is set and is in the future, remove it and write the session immediately.

The forced form is used when the notification transport closes, which happens when the user closes the last tab; the session is then treated as idle from that instant. If the user comes back before the threshold, the stamp is removed again and no confirmation is asked.

The writes are explicit because messages on the notification transport do not go through the request post-dispatch step of section 4.

The user's shortest inactivity timeout, in seconds, is also reported to the client inside the session description, for non-public users only, which lets the client display its own countdown.

## 10. The cross-site request forgery token

### 10.1 Minting

```formula
horizon = the current instant truncated to a whole number
        + ( the requested lifetime, or 31 536 000 seconds )
```

1. Read the system parameter `database.secret`. When it is absent, fail with "CSRF protection requires a configured database secret".
2. Build the message as the stable prefix of the session identifier concatenated with the decimal text of the horizon.
3. Compute the keyed digest of the message under the secret, using the 160-bit member of the standard secure hash family, rendered in hexadecimal.
4. The token is that digest, then the letter `o`, then the decimal text of the horizon.

The default lifetime of 31 536 000 seconds, one year, is not a meaningful expiry. It exists so that the message differs between mintings, which defeats compression-based extraction attacks. A caller that wants a real expiry passes one; no platform caller does, so in practice a token stays valid for as long as the stable prefix does.

### 10.2 Validation

1. An empty token is invalid.
2. Split the token at its **last** occurrence of the letter `o`, giving the digest and the horizon.
3. If a horizon is present: a horizon that is not a number is invalid, and a horizon earlier than the current instant is invalid.
4. Recompute the digest from the stable prefix and the horizon text.
5. Compare in constant time.

### 10.3 When it is required

A request is checked when both of the following hold: the method is not one of the four safe methods, which are read, head, options and trace; and the endpoint has not disabled the check. The check is enabled by default for the page transport family and disabled by default for the remote-call families.

- When no database is resolved, the request is instead redirected to the database-selection page.
- The token is read from the parameter named `csrf_token`, which is removed from the parameters before the endpoint is called.
- A present but invalid token logs "CSRF validation failed on path '<path>'", with the path substituted; an absent token logs a longer guidance message. Both produce the bad-request status with the body "Session expired (invalid CSRF token)".

Because the token binds to the **stable prefix**, it survives soft rotations and stops validating after a hard rotation, that is, after a login or a logout.

## 11. Expiry

Three independent limits apply.

| Limit | Length | Enforced by |
|---|---|---|
| Cookie maximum age | The effective inactivity limit. | The browser discards the cookie. |
| Server-side inactivity | The effective inactivity limit. | The session vacuum deletes records whose stored form has not been touched for that long. |
| Absolute re-authentication | The absolute lock timeout of section 9.2, when configured. | A session-expired failure at the authentication step. |

The **effective inactivity limit** is the integer value of the system parameter `sessions.max_inactivity_seconds`, defaulting to 604 800 seconds, which is seven days, when the parameter is absent, unparseable, or when no usable database connection exists. An unparseable value logs the warning `Invalid value for 'sessions.max_inactivity_seconds', using default value.`

The session vacuum runs inside the automatic cleanup job and can be suppressed by a deployment switch that exists for test environments only.

## 12. Logout

1. Take the current database when the caller asks to keep it, and nothing otherwise.
2. Take the current development flags.
3. Clear every key of the session.
4. Re-apply the defaults of section 2, then restore the database and the flags.
5. Set the context language to the request language preference, or to `en_US` when there is no request.
6. Mark a rotation pending.
7. Invoke the post-logout extension point.

The post-logout extension point does nothing in the platform and is where capability packages clear their own session keys.

Because a rotation is pending, the next persistence point performs a hard rotation: the old record is deleted and a brand-new identifier is issued, with the database and the development flags carried over. The user therefore keeps their database selection across a logout while losing every credential-bound value.

The session-expired failure of the page transport family performs a variant of this: the logout keeps the database, then rotates immediately and writes the cookie on the redirect response itself, because the ordinary persistence point is not reached on an error path.

## 13. Worked examples

### 13.1 Soft rotation with a concurrent request

A session was created at instant 0 with the identifier `A`, whose stable prefix is `P`. At instant 10 801 two requests arrive at the same time carrying the cookie `A`.

| Step | Request 1 | Request 2 |
|---|---|---|
| 1 | Loads `A`; the age 10 801 is at least 10 800, so a rotation is due. | Loads `A`; the age 10 801 is at least 10 800, so a rotation is due. |
| 2 | Re-reads `A`: no successor. Creates the successor `B`, which is `P` followed by 42 fresh characters. Writes `A` with the successor `B` and the deletion instant 10 921. | Re-reads `A`: the successor `B` is present. Adopts the identifier `B`. Performs no write of `A`. |
| 3 | Writes `B` with a fresh creation instant, a new binding token and the cleanup marker. | Writes `B`; in whichever order, the content is equivalent. |
| 4 | Emits the cookie `B`. | Emits the cookie `B`. |

At instant 10 950 a third request still carrying the cookie `A` loads the record `A`, sees the deletion instant 10 921 already in the past, and is treated as expired. A request carrying the cookie `A` at instant 10 850 is accepted and follows the chain to `B`.

At any instant after 10 921, a later request on `B` performs the predecessor cleanup: every record whose name begins with `P` is deleted, then `B` is written again.

### 13.2 A password change logs out the other devices

A user is logged in from two browsers with the sessions `S1` and `S2`.

| Step | Effect |
|---|---|
| 1 | From `S1` the user changes the password. The current password is verified and the new verifier is written. |
| 2 | Writing the password column clears every registry cache container, dropping the memoized tokens. |
| 3 | The persistence point of the request that performed the change recomputes and stores the binding token of `S1` from the new verifier, so `S1` remains valid. |
| 4 | The next request on `S2` recomputes the expected token from the new verifier. It differs from the token stored in `S2`, so the check fails, `S2` is logged out keeping the database, and the user is redirected to the login page. |

### 13.3 A device revocation

A user sees three devices, the first of which is the current one.

| Step | Effect |
|---|---|
| 1 | The revoke operation is guarded. The last identity confirmation was 15 minutes ago, which is more than 600 seconds, so the wizard opens. |
| 2 | The user confirms. The last-confirmation instant is set and the pending call is replayed. |
| 3 | The prefixes of the three devices are collected, and every stored session whose name begins with one of them is deleted. |
| 4 | Every log row carrying one of those prefixes is marked revoked. |
| 5 | The current device is among them, so the session is logged out, a rotation is pending, and the user lands on the login page with the database preserved. |

### 13.4 Inactivity lock

The user belongs to a group declaring an inactivity timeout of 15 minutes, that is 900 seconds, without a second factor.

| Instant | Event | Session state |
|---|---|---|
| 0 | The user logs in. | The last-confirmation instant and the inactivity stamp are both unset. |
| 600 | The client reports 600 seconds of idleness. 600 is below 900, so nothing is written. | unchanged |
| 960 | The client reports 960 seconds of idleness. 960 is at least 900, so the stamp becomes 960 + 900 − 960 = 960, which is the current instant. | the stamp is set to 960 |
| 961 | A request at the level requiring a user evaluates the condition: is 960 at or below 961 − 900 = 61? No. Nothing happens. | unchanged |
| 1 870 | The same evaluation: is 960 at or below 1 870 − 900 = 970? Yes. An identity-check failure is raised. | unchanged |
| 1 875 | The user confirms with a password. The stamp is removed and the last-confirmation instant becomes 1 875. | confirmed |

### 13.5 A bearer call from a background script

A script holds a global application key with an expiration three months away.

| Step | Effect |
|---|---|
| 1 | It calls an endpoint of the level `bearer`, presenting the key in the authorization header and sending no cookie. |
| 2 | The key's index, its first 8 hexadecimal characters, selects a small set of candidate rows; the presented value verifies against one of them; the owner becomes the acting user. |
| 3 | The session is marked non-persistable, so no record is written and no cookie is returned. The script therefore holds no session state between calls. |
| 4 | The same script later calls the endpoint with the key **and** a cookie naming a different user. The call is refused with `Session user does not match the used apikey.` |
| 5 | After the expiration has passed, the same key selects the same candidate rows but the expiration filter excludes them, so no owner is found and the call is refused with `Invalid apikey`. |

## 14. Acceptance criteria

1. **Given** a request with no session cookie, **when** it is served, **then** a session identifier of exactly 84 characters from the web-safe alphabet is generated, and no session record is written unless something marks the session dirty.
2. **Given** a well-formed session cookie for which no record exists, **when** the request is served, **then** the request succeeds as an anonymous session carrying that identifier.
3. **Given** a session cookie of the wrong length, **when** the request is served, **then** it is discarded and a brand-new identifier is generated.
4. **Given** a session whose age has just reached 10 800 seconds and that names a user, **when** a request other than the three excluded paths is served without the skip-rotation header, **then** the new identifier shares its first 42 characters with the old one and differs in the last 42.
5. **Given** the same session, **when** two requests rotate concurrently, **then** exactly one successor record is created and both requests emit the same successor identifier.
6. **Given** a session that has just been soft-rotated, **when** a request arrives with the predecessor cookie 60 seconds later, **then** it is accepted; **when** it arrives 130 seconds later, **then** it is treated as expired.
7. **Given** a soft-rotated session, **when** 120 seconds have passed since the successor was created and a request is served on it, **then** every record sharing the stable prefix is deleted and the successor record is written again, leaving exactly one record.
8. **Given** a request on one of the three rotation-excluded paths on a session older than 10 800 seconds, **when** it is served, **then** no rotation happens.
9. **Given** a logged-in user, **when** the user's login is changed by an administrator, **then** every existing session of that user fails its next binding-token check.
10. **Given** a logged-in user, **when** the user changes their own password, **then** the session that performed the change stays valid and every other session of that user fails its next check.
11. **Given** a user whose stored verifier uses an outdated derivation, **when** the user logs in successfully, **then** the verifier is replaced, every registry cache container is cleared, and the user is not logged out.
12. **Given** a session whose user has been deleted, **when** the binding token is recomputed, **then** no token is produced and the check fails.
13. **Given** an endpoint of the level `user` and a session with no acting user, **when** the endpoint is called, **then** a session-expired failure is produced.
14. **Given** an endpoint of the level `public` and a session with no acting user, **when** the endpoint is called, **then** the acting user is the shared public user.
15. **Given** an endpoint of the level `bearer` and a valid global application key in the authorization header, **when** the endpoint is called, **then** the acting user is the key's owner and no session record is written.
16. **Given** an endpoint of the level `bearer`, a valid key, and a session naming a different user, **when** the endpoint is called, **then** the request is denied with `Session user does not match the used apikey.`
17. **Given** an endpoint of the level `bearer`, no authorization header and an authenticated session, **when** the four browser navigation headers are absent, **then** the request is refused with the unauthorized status and a bearer challenge.
18. **Given** an endpoint of the level `bearer` and no authorization header and no authenticated session, **when** the endpoint is called, **then** the refusal message is "User not authenticated, use an API Key with a Bearer Authorization header."
19. **Given** a cross-origin preflight request on an endpoint of the level `user`, **when** it is served, **then** it is treated as the level `none`.
20. **Given** an application key whose expiration instant has passed, **when** it is presented, **then** it does not authenticate, and the automatic cleanup job later deletes it.
21. **Given** a user who is not privileged, **when** they create an application key without an expiration, **then** the creation is refused with "The API key must have an expiration date".
22. **Given** a user whose greatest group duration is 30 days, **when** they create a key expiring in 45 days, **then** the creation is refused with `You cannot exceed %(duration)s days.` naming 30.
23. **Given** a user who is not an internal user, **when** they try to create an application key, **then** it is refused with "Only internal users can create API keys".
24. **Given** a successful first factor and a user with a second factor configured whose policy is `default`, **when** the login is evaluated, **then** the session holds the pending login and user key, holds no acting user, and the client is sent to the second-factor page.
25. **Given** a pending second factor and a valid trusted-device cookie whose key has the scope `browser` and belongs to the pending user, **when** the second-factor page is visited, **then** the session is finalized without asking for a code.
26. **Given** a second-factor value that is not a well-formed number, **when** it is submitted, **then** the message is `Invalid authentication code format.`
27. **Given** a completed login, **when** the session is persisted, **then** the identifier is entirely new, which is a hard rotation, and the binding token matches the new identifier.
28. **Given** a session with no confirmation in the last 600 seconds, **when** a guarded operation is invoked, **then** a confirmation wizard is returned instead of the operation result, and the operation runs only after a successful confirmation.
29. **Given** a session with a confirmation 300 seconds ago, **when** a guarded operation is invoked, **then** it runs immediately.
30. **Given** an operation that is not marked as guarded, **when** the wizard is asked to replay it, **then** the replay is refused.
31. **Given** a user whose groups declare an absolute timeout of 1 440 minutes with a second factor required, **when** a request at the level `user` is served 1 441 minutes after the session was created, **then** a session-expired failure is raised and the user must log in again with a second factor.
32. **Given** the three groups of the worked example of section 9.2, **when** the effective inactivity timeouts are computed, **then** the result is the single pair of 900 seconds requiring a second factor.
33. **Given** a cross-site token minted before a soft rotation, **when** it is validated after that rotation, **then** it is accepted; **when** it is validated after a hard rotation, **then** it is rejected.
34. **Given** a page-family request with an unsafe method and no token, **when** it is dispatched, **then** the response has the bad-request status and the body "Session expired (invalid CSRF token)".
35. **Given** a page-family request with an unsafe method and no database resolved, **when** it is dispatched, **then** the client is redirected to the database-selection page instead of being refused.
36. **Given** no configured database secret, **when** a token is minted, **then** the minting fails with "CSRF protection requires a configured database secret".
37. **Given** a successful session check from a browser whose platform, browser and address match an existing trace less than 3 600 seconds old, **when** the request is served, **then** no device log row is inserted.
38. **Given** the same browser one hour and one second later, **when** the request is served, **then** exactly one device log row is inserted and the trace's last-activity instant is updated.
39. **Given** a request served on a read-only connection that must record a device log row, **when** the row is written, **then** the read-only transaction is rolled back first and the row is written in a separate read/write transaction.
40. **Given** a session carrying the trace-suppression flag, **when** the check succeeds, **then** no trace is created and no device log row is inserted.
41. **Given** a session record that has not been modified for longer than the effective inactivity limit, **when** the automatic cleanup job runs, **then** the record is deleted and the corresponding device rows are eventually marked revoked.
42. **Given** a logout with the database kept, **when** the response is produced, **then** the session holds no acting user, no login and no token, still holds the database and the development flags, and the next persistence point issues a brand-new identifier.
43. **Given** a device revocation that includes the current device, **when** it completes, **then** the current session is logged out.
44. **Given** a stable prefix value that is not 42 characters of the web-safe alphabet, **when** it is used to enumerate sessions, **then** the operation is refused with `Identifier format incorrect, did you pass in a string instead of a list?`

## 15. Reconciliation notes

1. Several system parameters were named by paraphrase. They are reproduced exactly: `database.secret`, `sessions.max_inactivity_seconds`, `password.hashing.rounds`, `base.login_cooldown_after`, `base.login_cooldown_duration`, `base.enable_programmatic_api_keys`, `base.programmatic_api_keys_limit`, `auth_totp.trusted_device_age`, `web.base.url` and `web.base.url.freeze`.
2. The messages concerning the non-interactive credential were rewritten with the credential's name spelled out in full words. They are contractual user-visible text and are reproduced verbatim, with the abbreviation they contain: `Invalid apikey`, `Session user does not match the used apikey.`, "User not authenticated, use an API Key with a Bearer Authorization header.", "The API key must have an expiration date", "You can not remove API keys unless they're yours or you are a system user", "Programmatic API keys are not enabled", "Limit of <limit> API keys is reached for programmatic creation", "Only internal users can create API keys", "The API key duration is not correct.", "API Key Ready", "The provided API key is invalid or does not belong to the current user." and "The provided API key is invalid." One source gave two different spellings of the second of those; the reproduced one is the one above.
3. The two rules governing who may create a key at all, and the two messages returned when a presented key cannot be resolved, were absent. They are specified in section 8.4.
4. The login cooldown guard was named without its thresholds. They are the two parameters of section 8.5, step 1, with their defaults of 5 failures and 60 seconds.
5. The session-expired text raised by the level requiring a user was not given. It is `Session expired`, and it is distinct from the longer text of the absolute timeout, which names the user key.
6. The derivation algorithms were named by their family. They are described by their bit length and family rather than by a product name, because a rebuild must reproduce the algorithm and not a library.
7. The entropy claim of section 1.1 was stated without arithmetic. The two computations are given, and both support the conclusion.
8. The device projection was described as a view. It is described here as a read-only projection, because this repository never names a storage construct.
