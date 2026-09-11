# Identity and Access — Calculations

This file gives every formula and every algorithm of the domain in full, with its rounding rule,
its evaluation order and at least one worked numeric example. Nothing here is a summary: each
section is meant to be implementable from the text alone.

Notation used throughout:

- `×` multiplication, `÷` division, `+` addition, `−` subtraction, `=` equality.
- `floor(x)` is the greatest whole number not exceeding *x*.
- `x mod y` is the non-negative remainder of *x* divided by *y*.
- Byte strings are written as sequences of octets; hexadecimal is written with the digits 0–9 and
  a–f.
- "the current moment" means the wall-clock moment on the server, in coordinated universal time
  unless a section says otherwise.

---

## 1. The transitive closure of group implication

### 1.1 Definition

Let *G* be the set of all groups. Implication is a relation *→* on *G*: *a → b* means "every member
of *a* is a member of *b*". The **superset closure** of a set *S ⊆ G* is

```formula
superset_closure(S) = S ∪ { b : there is a in S with a →* b }
```

where *→\** is the reflexive transitive closure of *→*. The **subset closure** is the same with the
relation reversed:

```formula
subset_closure(S) = S ∪ { b : there is a in S with b →* a }
```

### 1.2 The compiled definition

The graph is compiled once into a table keyed by group identifier, holding for each group:

| Entry | Content |
|---|---|
| references | the external references of the group, or, when it has none, the decimal rendering of its identifier |
| supersets | the identifiers of the groups it directly implies |
| disjoints | the identifiers of the groups it is declared disjoint from |

The rows are produced in ascending identifier order so the compiled structure is deterministic.

### 1.3 The algorithm

**Preconditions.** A set *S* of group identifiers; the compiled table.

1. Let *result* be a copy of *S*; let *frontier* be a copy of *S*.
2. While *frontier* is not empty:
   1. Let *next* be the empty set.
   2. For each identifier *g* in *frontier*: for each identifier *h* in the supersets of *g*: if *h*
      is not in *result*, add *h* to *result* and to *next*.
   3. Set *frontier* to *next*.
3. Return *result*.

**Postcondition.** *result* is `superset_closure(S)`. The loop terminates because *result* grows
strictly at each iteration and *G* is finite; a cycle in the implication graph is therefore harmless
and simply makes the members of the cycle mutually implied.

The subset closure uses the same algorithm on the reversed edges.

### 1.4 Worked example

Groups and their direct implications:

| Identifier | Group | Directly implies |
|---|---|---|
| 1 | Role / User | 5 |
| 2 | Role / Administrator | 3, 4 |
| 3 | Access Rights | 1 |
| 4 | Bypass HTML Field Sanitize | — |
| 5 | Technical Features | — |
| 10 | Sales / User: own documents only | 1 |
| 11 | Sales / User: all documents | 10 |
| 12 | Sales / Administrator | 11 |

Take a user whose **explicit** groups are {12, 2}.

| Iteration | frontier | newly added | result |
|---|---|---|---|
| start | {12, 2} | — | {12, 2} |
| 1 | {12, 2} | 11 (from 12), 3 and 4 (from 2) | {12, 2, 11, 3, 4} |
| 2 | {11, 3, 4} | 10 (from 11), 1 (from 3) | {12, 2, 11, 3, 4, 10, 1} |
| 3 | {10, 1} | 5 (from 1); 10 → 1 already present | {12, 2, 11, 3, 4, 10, 1, 5} |
| 4 | {5} | — | unchanged |
| 5 | {} | — | terminate |

The closure has **8** groups: {1, 2, 3, 4, 5, 10, 11, 12}. Consequences:

- the *Share User* flag is **false**, because 1 (*Role / User*) is present;
- the *Role* field reads *Administrator*, because 2 is present;
- the group count shown on the user form is 8;
- the access-right count is the number of distinct access-right rows attached to any of the eight;
- the record-rule count is the number of distinct rules attached to any of the eight.

### 1.5 Searching through the closure

Searching for "users whose closure contains group *g*" is rewritten as "users whose explicit groups
are in `subset_closure({g})`", because a user reaches *g* exactly when one of their explicit groups
is *g* or implies *g*. Conversely, searching for "groups whose closure contains *g*" is rewritten as
"groups whose identifier is in `subset_closure({g})`", and "groups reached from *g*" as "identifiers
in `superset_closure({g})`".

Continuing the example: `subset_closure({1})` = {1, 3, 10, 11, 12, 2} — every group whose members
are internal users. A search for internal users is therefore a search for users whose explicit
groups intersect that set.

---

## 2. Password hashing

### 2.1 The hashing context

Passwords are stored as encoded hashes produced by a password-based key derivation function with a
keyed hash of digest length 512 bits. The context is configured as follows:

| Setting | Value |
|---|---|
| accepted schemes, in order of preference | the iterated derivation scheme first, then the plain-text scheme |
| deprecated schemes | automatic — every scheme except the first is deprecated |
| iteration count for the derivation scheme | `max(600000, the parameter password.hashing.rounds interpreted as a whole number, defaulting to 0)` |

The formula for the iteration count is therefore:

```formula
iterations = maximum( 600000 , configured_rounds )
```

with *configured_rounds* read from the system parameter `password.hashing.rounds` and taken as 0
when absent or unparseable. The lower bound of 600 000 cannot be reduced by configuration; only
raised.

The plain-text scheme exists solely so that a value written directly into the column by a
provisioning script is recognised as a password and upgraded on first use; it is deprecated, so any
stored plain value is replaced by a derived hash the first time it verifies.

### 2.2 Verification and silent upgrade

Verification returns two results: a verdict, and a **replacement hash** when the stored hash uses a
deprecated scheme or an iteration count below the configured one.

```formula
(verdict, replacement) = verify_and_update( supplied_password , stored_hash )
```

When *replacement* is produced, it is written to the account's row immediately, inside the same
transaction as the successful sign-in.

### 2.3 Upgrade at start-up

When the foundation package initialises, every row whose password does not match the shape of an
encoded hash — that is, does not match a dollar sign, a scheme name, a dollar sign, parameters, a
dollar sign and at least one further character — is re-written through the hashing context. This is
what turns provisioning-time plain values into hashes.

### 2.4 Worked example

The parameter `password.hashing.rounds` is absent. A password is set to `correct horse battery
staple`.

1. `configured_rounds = 0`.
2. `iterations = maximum(600000, 0) = 600000`.
3. The derivation runs with 600 000 iterations and a random salt, producing an encoded string of the
   form `$pbkdf2-sha512$600000$<salt>$<derived key>`.
4. That string is written to the column. Reading the password field afterwards yields the empty
   string, not the stored value.

Later an administrator sets the parameter to `1000000`.

5. The next successful sign-in of that account verifies the supplied password against the stored
   hash (600 000 iterations) — the verdict is positive — and the verification reports a replacement
   because 600 000 < 1 000 000.
6. The replacement (1 000 000 iterations, new salt) is written, the caches are cleared, and the
   session token is recomputed and stored so the user is not signed out by their own sign-in.

---

## 3. Application-key generation and verification

### 3.1 Generation

```formula
clear_key = hexadecimal_rendering( 20 random bytes )
index     = the first 8 characters of clear_key
stored    = key_hash( clear_key )
```

- The 20 bytes come from the operating system's cryptographic source. The rendering yields exactly
  40 hexadecimal characters, so the key has 160 bits of entropy.
- The index is 8 hexadecimal characters, that is 4 bytes, that is 20 % of the key. It is stored in
  clear so that verification can narrow to a handful of rows with one index lookup; knowing the
  index leaves 128 bits to guess.
- The key hash uses the same derivation function as passwords but with **6 000** iterations instead
  of at least 600 000, because a uniformly random 160-bit secret is not subject to dictionary
  attack and the cheaper factor keeps programmatic calls fast.

### 3.2 Verification

**Preconditions.** A purpose (called the *scope*) and a candidate key, both non-empty.

1. `index = the first 8 characters of the candidate`.
2. Select rows joined to their accounts where: the account is active; the row's index equals
   *index*; the row's scope is empty **or** equals the requested purpose; and the row's expiry is
   empty **or** is not earlier than the current moment in coordinated universal time.
3. For each selected row in turn, verify the candidate against the stored hash. Return the account
   identifier of the first match.
4. If no row matches, the result is "no account".

### 3.3 The maximum duration of a key

```formula
max_duration_days = maximum over the groups of the user's closure of ( group.api_key_duration )
if max_duration_days is 0 or absent then max_duration_days = 1.0
```

The validity requested for a new key must satisfy

```formula
current_moment < requested_expiry ≤ current_moment + max_duration_days × 86400 seconds
```

with the left inequality producing *You cannot set an expiration date in the past.* and the right
producing *You cannot exceed the maximum number of days days.*

### 3.4 Worked example

A user's closure contains *Role / User* (maximum duration 90.0 days) and *Sales / Administrator*
(maximum duration not set, i.e. 0.0).

1. `max_duration_days = maximum(90.0, 0.0) = 90.0`.
2. The choices offered are those base choices whose day count is at most 90: 1, 7, 30, 90 — plus the
   custom-date choice. The persistent choice is not offered because the user is not an
   administrator.
3. The user picks 90. Today is 11 September 2026, so the expiry becomes 10 December 2026.
4. Validation: the expiry is in the future, and `10 December 2026 ≤ 11 September 2026 + 90 days =
   10 December 2026` holds with equality, so it passes.
5. Generation produces, say, `9f2c1b7e4a0d6583cc21ff90ab3d7e5164280cb7`; the index is `9f2c1b7e`;
   the row stores the hash with 6 000 iterations.

If the same user typed a custom date of 1 January 2027, validation would compute
`1 January 2027 > 10 December 2026` and refuse with *You cannot exceed 90.0 days.*

---

## 4. Keyed message authentication

### 4.1 The primitive

Every signature in the domain is a keyed hash over a **scoped** message.

```formula
signature = keyed_hash_sha256( key = database_secret , message = repr( ( scope , message ) ) )
```

rendered as lower-case hexadecimal (64 characters).

- *database_secret* is the value of the system parameter `database.secret`, encoded as bytes. It is
  a random version-4 universally unique identifier written when the database is created. An empty
  secret is a programming error.
- *scope* is a non-empty text distinguishing one use from another, so that the same message signed
  for two purposes yields two different signatures. An empty scope is a programming error.
- `repr((scope, message))` is the canonical textual rendering of the pair as a two-element tuple.
  This exact rendering is part of the contract: a re-implementation must produce the identical byte
  string, otherwise no previously issued token verifies. The rendering is: an opening parenthesis;
  the scope rendered as a quoted literal; a comma; a space; the message rendered as a literal; a
  closing parenthesis.

### 4.2 A signed payload with an expiry

A signed payload packs a message, an expiry and a signature into one address-safe string.

**Producing.**

1. If a validity in hours *h* was given, `expiry = current_moment + h hours`. If a duration was
   given, `expiry = current_moment + duration`. If an absolute moment was given, that is the expiry.
   If nothing was given, there is no expiry.
2. `expiry_timestamp = 0` when there is no expiry, otherwise the whole number of seconds since the
   epoch of the expiry.
3. `message_text` = the message values serialised as a structured literal.
4. `signature = keyed_hash( scope , "1:" + message_text + ":" + expiry_timestamp )`.
5. The token bytes are: one octet with value 1 (the version); then the expiry timestamp as **8
   octets, least significant octet first**; then the 32 octets of the signature; then the bytes of
   the message text.
6. The token is the address-safe base-64 rendering of those bytes with trailing padding characters
   removed.

**Verifying.**

1. Re-add up to three padding characters and decode the address-safe base-64.
2. The first octet must be 1, otherwise the token is malformed (*Unknown token version*).
3. Octets 2 to 9 are the expiry timestamp, least significant first. Octets 10 to 41 are the
   signature. The remainder is the message text.
4. Recompute the signature over `"1:" + message_text + ":" + expiry_timestamp`.
5. Compare the two signatures **in constant time**. If they differ, the token is invalid.
6. If the expiry timestamp is 0, or the current moment is strictly before it, return the parsed
   message values; otherwise the token is invalid.

The total token length is therefore `ceil((41 + length of message_text) ÷ 3) × 4` characters before
padding removal.

### 4.3 Constant-time comparison

Every comparison of a secret with a supplied value uses a comparison whose duration does not depend
on the position of the first differing byte. This applies to: the session token, the signed payload
signature, the document access token, and the recipient signature.

---

## 5. Tokens of this domain

### 5.1 The document access token

```formula
access_token = a random universally unique identifier of version 4, rendered as text
```

That is: 128 random bits, of which 122 are free, rendered as 32 hexadecimal digits in five
hyphen-separated groups of 8, 4, 4, 4 and 12. It is stored on the document and compared in constant
time.

### 5.2 The sign-up token

The payload is the four-element list

```formula
payload = [ contact_identifier ,
            the identifiers of the contact's accounts ,
            most_recent_sign_in ,
            sign_up_type ]
```

where

```formula
most_recent_sign_in = the greatest of the sign-in moments of the contact's accounts, as a whole
                      number of seconds since the epoch, or absent when none of them has ever
                      signed in
```

and *sign_up_type* is the text `signup` or `reset`.

The validity in hours is:

| Sign-up type | Parameter | Default |
|---|---|---|
| `reset` | `auth_signup.reset_password.validity.hours` | 4 |
| `signup` | `auth_signup.signup.validity.hours` | 144 |

The token is the signed payload of section 4.2 with the scope `signup`.

**Resolving a token.**

1. Verify the signed payload with the scope `signup`. Failure yields "no contact".
2. Unpack the four elements.
3. Re-read the contact by identifier.
4. Accept only if **all three** of the following hold:
   - the recorded most recent sign-in equals the contact's current most recent sign-in;
   - the recorded list of account identifiers equals the contact's current list;
   - the recorded sign-up type equals the contact's current sign-up type.
5. Otherwise yield "no contact".

The three checks give the token four independent ways of dying: the signature expires; signing in
changes the most recent sign-in moment; creating an account for the contact changes the account
list; and clearing or changing the sign-up type invalidates it directly. Nothing usable is stored in
the database.

**Worked example.** Contact 4711 has one account, identifier 88, which has never signed in; the
sign-up type is `signup`; the validity parameter is at its default.

1. `most_recent_sign_in` is absent.
2. `payload = [4711, [88], absent, "signup"]`.
3. `expiry = current_moment + 144 hours`.
4. The message text is the serialised form of the payload, for example
   `[4711, [88], null, "signup"]`.
5. The signature is computed over `1:[4711, [88], null, "signup"]:<expiry timestamp>` with the
   scope `signup`.
6. The token bytes are `01`, the eight expiry octets, the 32 signature octets, and the message text;
   the token is their address-safe base-64 rendering without padding.

After account 88 signs in once at, say, timestamp 1 788 000 000, the current most recent sign-in
becomes 1 788 000 000 while the token still records "absent", so the token stops resolving even
though its signature is still valid.

### 5.3 The recipient signature on a shared document

```formula
recipient_signature = keyed_hash_sha256(
        key     = database_secret ,
        message = repr( ( database_name , document_access_token , recipient_contact_identifier ) ) )
```

rendered as lower-case hexadecimal. Note that this signature does **not** use the scoped primitive
of section 4.1: the message is a three-element tuple and there is no scope prefix. Including the
document's access token means the signature dies when the token is changed, and including the
database name prevents a signature from one database being replayed against another.

Producing it requires the document's entity to declare which field is the token field; an entity
that does not is a programming error:

> Model *the entity name* does not support token signature, as it does not have *the field name*
> field.

---

## 6. The session token

### 6.1 The key material

The **session-token fields** are read for the account in a single query. The base set is
{identifier, login, password, active}; packages add the second-factor secret, the delegated-sign-in
token, and — through a joined aggregate — the ordered list of the account's passkey identifiers.
The query also selects the value of the system parameter `database.secret` as a first column.

The query shape is:

```
select  ( the database secret ) as database secret ,
        the non-relational session-token columns, in ascending name order
from    the user table
        left join, for each relational extension, its table
where   the user identifier equals the account
group by the user identifier
```

Relational session-token fields are excluded from the plain column list precisely because they are
not columns; each contributes instead a joined aggregate expression. Passkeys contribute the array
of their identifiers, ordered by identifier descending, with rows where there is no passkey filtered
out.

### 6.2 The computation

```formula
key_pairs  = the ordered list of ( column name , value ) for every selected column
             whose value is not absent
key        = the bytes of repr( key_pairs as a tuple of pairs )
session_token = keyed_hash_sha256( key = key , message = the bytes of the session identifier )
```

rendered as lower-case hexadecimal.

Three properties are load-bearing:

- The **key** is the account's state and the **message** is the session identifier — the reverse of
  the usual arrangement. This means the token changes whenever any part of the account's state
  changes, which is exactly the intent.
- Values that are absent are **dropped** before the key is built. This is deliberate: installing a
  package that adds a new session-token field does not invalidate every session, because the new
  field is absent for accounts that have not used the feature.
- The column names are part of the key, so two fields swapping values do not produce the same token.

If the query returns anything other than exactly one row, the registry caches are cleared and the
computation yields "no token", which makes every session of that account invalid.

### 6.3 The older computation

An older construction exists and is accepted as a fallback during verification:

```formula
legacy_key = the bytes of repr( the tuple of the values only, in the same order )
legacy_token = keyed_hash_sha256( key = legacy_key , message = the session identifier )
```

It omits the column names and does not drop absent values. When a session's stored token matches the
older construction, the stored token is silently replaced by the current one and the session is
accepted.

### 6.4 The browser cache key

A separate value is handed to the client so that its own caches are invalidated by the same events:

```formula
browser_cache_secret = keyed_hash_sha256(
        key = database_secret ,
        message = repr( ( "browser_cache_key" , the ordered ( column name , value ) pairs ) ) )
```

It is computed with the scoped primitive of section 4.1, using the same key material as the session
token, and it is delivered only on the page that bootstraps the client, which is served with a
directive forbidding storage.

### 6.5 Worked example

Account 88: login `jean.martin@example.test`, password hash `$pbkdf2-sha512$600000$…`, active true,
no second-factor secret, no delegated-sign-in token, no passkeys. Session identifier
`c1f0…` (32 characters).

1. The selected columns are, in this order: the database secret; then the non-relational
   session-token columns in ascending name order — `active`, `id`, `login`, `password`; then, from
   the passkey extension, the aggregate `auth_passkey_key_ids`.
2. The values are: the secret; true; 88; the login; the hash; and, because the account has no
   passkey, the filtered aggregate is absent.
3. Absent values are dropped, so the passkey pair disappears.
4. The key is the rendering of the five surviving pairs as a tuple of pairs.
5. The token is the keyed hash of the session identifier under that key, rendered as 64 hexadecimal
   characters.

Now the user registers a passkey with identifier 3.

6. The aggregate becomes the one-element array [3], which is not absent, so a sixth pair joins the
   key.
7. The key changes, the token changes, and every other session of account 88 becomes invalid at its
   next request. The registering session is explicitly repaired: the new token is computed and
   written into that session in the same request.

---

## 7. One-time code arithmetic

### 7.1 Parameters

| Parameter | Value |
|---|---|
| secret size | 160 bits |
| hash function | keyed hash with a 160-bit digest |
| digits | 6 |
| time step | 30 seconds |
| acceptance window | ±30 seconds |

The secret is drawn as `160 ÷ 8 = 20` random bytes from the operating system's cryptographic source
and rendered in base 32 (32 characters), then grouped in blocks of four separated by spaces for
display. White space is removed before use, and the value is upper-cased.

### 7.2 The counter-based code

For a secret *K* (raw bytes) and a whole-number counter *C*:

```formula
C_bytes = C encoded as 8 octets, most significant octet first
mac     = keyed_hash_sha1( key = K , message = C_bytes )          (20 octets)
offset  = the low 4 bits of the last octet of mac                 (0 … 15)
truncated = the 4 octets of mac starting at offset, read most significant octet first
code_value = truncated and 0x7FFFFFFF                             (drop the top bit)
code = code_value mod 10^6
```

The masking with `0x7FFFFFFF` discards the highest bit so that the value is always non-negative; it
leaves 31 bits, which is why 9 digits is the practical maximum (each decimal digit is worth about
3.32 bits, and 9 digits need about 29.9 bits).

The code is compared **as a number**, not as text, so a code with leading zeros is accepted when the
client sends it as an integer.

### 7.3 The time-based code and the match

Given the secret *K*, a supplied code *c*, the current moment *t* in seconds, a window *w* = 30 and
a time step *s* = 30:

```formula
low  = floor( ( t − w ) ÷ s )
high = floor( ( t + w ) ÷ s ) + 1
match = the first counter n in low, low+1, …, high−1 such that code(K, n) = c
        (or "no match" when there is none)
```

The matched **counter** is returned, not merely a yes/no, because the counter is stored to prevent
replay.

Note that the range is half-open on the right — the loop runs from *low* up to but not including
*high* — so the counters tried are `floor((t−30)÷30)` through `floor((t+30)÷30)`, inclusive. That is
normally **three** counters: the previous step, the current step and the next step.

### 7.4 Replay prevention

```formula
accept if match is not "no match" and ( last_counter is absent or match > last_counter )
```

On acceptance, `last_counter = match`. A code accepted at counter *n* can therefore never be
accepted again, and neither can any older code.

### 7.5 Worked example

Let the current moment be *t* = 1 788 000 041 seconds.

```
low  = floor( (1788000041 − 30) ÷ 30 ) = floor( 1788000011 ÷ 30 ) = 59600000
high = floor( (1788000041 + 30) ÷ 30 ) + 1 = floor( 1788000071 ÷ 30 ) + 1 = 59600002 + 1 = 59600003
```

so the counters tried are 59 600 000, 59 600 001 and 59 600 002.

Suppose `code(K, 59600001) = 704318` and the user types `704318`. The match is 59 600 001.

- If the stored last counter is absent or is 59 600 000 or less, the code is accepted and the last
  counter becomes 59 600 001.
- If the user re-submits `704318` five seconds later, the window still contains 59 600 001, so the
  match is the same; but `59600001 ≤ 59600001`, so the code is refused as a replay with
  *Verification failed, please use the latest 6-digit code*.
- If the user types `123456` and no counter in the window produces it, the refusal is
  *Verification failed, please double-check the 6-digit code*.

### 7.6 The enrolment address

```
otpauth://totp/<issuer>:<login>?secret=<compressed secret>&issuer=<issuer>&algorithm=SHA1&digits=6&period=30
```

- The path component is the issuer, a colon and the login, address-encoded with the colon kept
  literal.
- The issuer is the request's host name with any port stripped; with no request, the account's
  company display name.
- The algorithm name is given in capitals; a lower-case name is rejected by common authenticator
  applications.
- The compressed secret is the base-32 secret with all white space removed.

The enrolment image is a quick response code of that address, drawn with a box size of 4 and
encoded as a portable network graphic, then rendered in base 64.

### 7.7 Rate limiting

Two independent limiters, each defined by a pair (limit, interval):

| Kind | Limit | Interval |
|---|---|---|
| mailing a code | 5 | 3600 seconds |
| verifying a code | 5 | 3600 seconds |

**Consuming.**

```formula
count = number of rows for ( this account , this kind ) whose creation moment
        is at or after ( current_moment − interval )
refuse if count ≥ limit
otherwise write one row ( this account , the remote address , this kind )
```

The messages are:

| Kind | Message |
|---|---|
| mailing | You reached the limit of authentication mails sent for your account, please try again later. |
| verifying | You reached the limit of code verifications for your account, please try again later. |

**Purging.** A successful verification deletes **all** rows of that account and that kind, so the
allowance is fully restored by a success.

The consumption is written **before** the code is checked, so a failed check consumes an attempt and
a successful one both consumes and then purges.

**Worked example.** Six wrong codes in quick succession for the same account:

| Attempt | Rows in window before | `count ≥ 5`? | Outcome |
|---|---|---|---|
| 1 | 0 | no | row written, code checked, wrong |
| 2 | 1 | no | row written, code checked, wrong |
| 3 | 2 | no | row written, code checked, wrong |
| 4 | 3 | no | row written, code checked, wrong |
| 5 | 4 | no | row written, code checked, wrong |
| 6 | 5 | yes | refused before checking; **no** row written |

The five rows expire 3600 seconds after each was written, so the first slot frees one hour after
attempt 1.

---

## 8. The sign-in cooldown

### 8.1 The parameters

| Parameter | Meaning | Fallback in the code | Value written at database creation |
|---|---|---|---|
| `base.login_cooldown_after` | failures before cooldown | 5 | 10 |
| `base.login_cooldown_duration` | cooldown length in seconds | 60 | 60 |

A threshold of 0 disables the feature entirely.

### 8.2 The predicate

```formula
on_cooldown = ( threshold ≠ 0 )
              and ( failures ≥ threshold )
              and ( current_moment − last_failure_moment < duration seconds )
```

### 8.3 The counter

The counters live in the process, keyed by the request's **remote network address**:

```formula
on failure : ( failures , last_failure_moment ) ← ( failures + 1 , current_moment )
on success : the entry for the address is removed
```

Counters are not shared between worker processes and are not synchronised between threads. They are
a rate limiter, not a quota.

### 8.4 Worked example

Threshold 10, duration 60 seconds, one source address.

| Failure number | failures after | on cooldown at the next attempt? |
|---|---|---|
| 1 … 9 | 1 … 9 | no (9 < 10) |
| 10 | 10 | yes, for 60 seconds after failure 10 |

At second 61 after failure 10, the elapsed time is 61 ≥ 60, so the attempt proceeds. If it fails,
failures becomes 11 and the last failure moment is reset, so a further 60 seconds of cooldown begin.
If it succeeds, the entry disappears and the source starts from zero.

With the code's fallback threshold of 5 (a database whose parameter has been deleted), the same
table reads 1 … 4 then cooldown from failure 5.

---

## 9. Default-value precedence

### 9.1 The query

For one entity name and one condition (or no condition), for the acting user and the acting company:

```
select the field name and the stored value
from   the default table joined to the field definitions
where  the field's entity is the requested entity
  and  ( the default's user is empty or equals the acting user )
  and  ( the default's company is empty or equals the acting company )
  and  ( the default's condition equals the requested condition ,
         or, when no condition is requested, the default's condition is empty )
order by the default's user , the default's company , the default's identifier
```

Rows are consumed in that order and, for each field, **the first row seen wins**; later rows for the
same field are discarded.

### 9.2 The consequence of the ordering

The ordering is ascending on the user column, then the company column, then the identifier. In the
ordering used by the storage layer, an **empty** value sorts **before** a set value. Therefore, among
the rows that survive the filter:

1. a row with **no user and no company** comes first;
2. then a row with **no user** and the acting company;
3. then a row with the acting user and **no company**;
4. then a row with the acting user and the acting company.

So the **least specific** default wins. This is the behaviour to reproduce; it is not what a reader
would assume, and it is therefore stated here explicitly rather than described as "most specific
wins".

### 9.3 Worked example

The entity is Sales Order, the field is the payment term, the acting user is 88 and the acting
company is 2. Four defaults exist:

| Identifier | User | Company | Value |
|---|---|---|---|
| 31 | — | — | payment term 1 (*Immediate*) |
| 32 | — | 2 | payment term 4 (*30 Days*) |
| 33 | 88 | — | payment term 7 (*45 Days*) |
| 34 | 88 | 2 | payment term 9 (*60 Days*) |

All four pass the filter. The ordering puts 31 first (empty user, empty company), then 32 (empty
user, company 2), then 33 (user 88, empty company), then 34. The first row for the payment-term
field is 31, so the applied default is **payment term 1**.

If row 31 is deleted, the winner becomes row 32 (payment term 4). If both 31 and 32 are deleted, the
winner becomes row 33 (payment term 7).

### 9.4 Setting a default

Setting resolves the two magic scope values first:

```formula
if user_scope is "the current user" then user_scope = the acting user identifier
if company_scope is "the current company" then company_scope = the acting company identifier
```

Then the value is validated by converting it to the field's type, a date or date-and-time value
given as a date object is rendered as text first, and the whole is serialised as a structured
literal without escaping non-ASCII characters. An existing row with the same (field, user, company,
condition) is rewritten **only when the serialised value differs**, so that setting a default to its
current value does not clear the caches.

### 9.5 Company-dependent fallbacks

For a company-dependent column, the fallback value of every company is precomputed once and cached:

```formula
fallbacks = a map from each company identifier to
            the column form of the default of ( entity , field ) read as the system account
            with that company active
```

This is why creating, writing or deleting any default invalidates the whole environment: it can
change the fallback of a company-dependent column in every record at once.

---

## 10. Session-lifetime summary per user

### 10.1 The computation

**Inputs.** The user's group closure.

For each of the two limit kinds — session timeout and inactivity timeout — with its companion
second-factor flag:

1. Collect the pairs `(minutes, demands_second_factor)` from every group of the closure whose
   minutes value is non-zero.
2. ```formula
   min_without = the smallest minutes among pairs whose flag is false, or absent
   min_with    = the smallest minutes among pairs whose flag is true,  or absent
   ```
3. Build the result list for this kind:
   - if *min_with* exists, append `( min_with × 60 , true )`;
   - if *min_without* exists **and** ( *min_with* does not exist **or** *min_without* < *min_with* ),
     append `( min_without × 60 , false )`;
4. Sort the list ascending by seconds.

The output is a map with one list per kind. The rule in step 3 means a non-second-factor limit is
kept only when it is **strictly shorter** than the second-factor one: a longer non-second-factor
limit would never fire before the stricter one and would only weaken it.

The whole computation is cached by the tuple of group identifiers and is computed with an empty
context.

### 10.2 Worked example

A user's closure contains four groups with limits:

| Group | Session timeout | Second factor on session timeout | Inactivity timeout | Second factor on inactivity |
|---|---|---|---|---|
| Role / User | — | — | — | — |
| Accounting / Administrator | 1440 minutes | yes | 15 minutes | no |
| Payroll / Officer | 720 minutes | no | 30 minutes | yes |
| Sales / User | 2880 minutes | no | — | — |

Session timeout:

- pairs: (1440, true), (720, false), (2880, false)
- `min_with = 1440`, `min_without = minimum(720, 2880) = 720`
- append (1440 × 60, true) = (86400, true)
- 720 exists and 720 < 1440, so append (720 × 60, false) = (43200, false)
- sorted: `[ (43200, false) , (86400, true) ]`

Inactivity timeout:

- pairs: (15, false), (30, true)
- `min_with = 30`, `min_without = 15`
- append (30 × 60, true) = (1800, true)
- 15 < 30, so append (15 × 60, false) = (900, false)
- sorted: `[ (900, false) , (1800, true) ]`

Reading: after 15 minutes of inactivity the user must re-authenticate with one factor; after 30
minutes, with two. After 12 hours of session age the user is signed out; after 24 hours the sign-out
demands two factors.

### 10.3 The inactivity offset

Only the **shortest** inactivity limit ever writes the *next identity check* moment into the session.
A longer inactivity limit must therefore measure from the same origin, which is done by subtracting
the shortest limit from the stored moment before comparing:

```formula
first_timeout = the seconds of the first entry of the inactivity list, or 0 when the list is empty
triggered( timeout ) = ( stored_moment − first_timeout ) ≤ ( current_moment − timeout )
```

The session timeout needs no such offset, because the session creation moment is written once when
the session is created.

**Worked example.** Using the lists above: `first_timeout = 900`. The client reports that the user
has been inactive for 900 seconds; the *next identity check* moment is written as
`current_moment + 900 − 900 = current_moment`. Thirty minutes of inactivity later, the current
moment is `stored + 1800`.

- For the entry (1800, true): `stored − 900 ≤ stored + 1800 − 1800 = stored` holds, so the
  second-factor re-authentication fires.
- For the entry (900, false): `stored − 900 ≤ stored + 1800 − 900 = stored + 900` also holds; but
  the pairs are examined in **descending** duration order, so (1800, true) is tested first and wins.

### 10.4 Rendering a number of minutes

```formula
if minutes is 0 or absent            then ( minutes , "minutes" )
else if minutes mod 1440 = 0         then ( minutes ÷ 1440 , "days" )
else if minutes mod 60 = 0           then ( minutes ÷ 60 , "hours" )
else                                      ( minutes , "minutes" )
```

and the inverse

```formula
minutes = value × 1440   when the unit is "days"
        = value × 60     when the unit is "hours"
        = value          otherwise
```

Examples: 1440 renders as 1 day; 2880 as 2 days; 720 as 12 hours; 90 as 90 minutes; 0 as 0 minutes.

---

## 11. The privacy search

### 11.1 Inputs and preparation

```formula
name             = the supplied name, trimmed
email_pattern    = "%" + the supplied address, trimmed + "%"
email_normalized = the normalised form of the supplied address
```

An address that does not normalise raises *Invalid email address "the value"*.

### 11.2 The query

The search is one union of selections, each producing three columns: the entity identifier, the
record identifier, and whether the record is active.

**Common table.** *indirect references* = the identifiers of contacts whose normalised address
equals *email_normalized* **or** whose name matches *name* case-insensitively.

**Part 1 — contacts.** Every contact in the indirect references, with its active flag.

**Part 2 — accounts.** Every account whose login matches *email_pattern* case-insensitively, or
whose contact's address matches *email_pattern* case-insensitively, or whose contact's name matches
*name* case-insensitively; with its active flag.

**Part 3 — messages.** Every message whose author is in the indirect references, marked active.

**Part 4 — every other entity.** For each entity in the registry, skipping the blacklist and
skipping transient entities and entities with no table of their own:

1. Build a list of conditions.
2. **Direct personal data.** Examine, in this order, the field names `email_normalized`, `email`,
   `email_from`, `company_email`. For the first of them that exists on the entity and is stored:
   - add the condition "that column equals *email_normalized*" when the field is the normalised one
     (or when the entity is the mailing-trace entity and the field is `email`); otherwise "that
     column matches *email_pattern* case-insensitively";
   - additionally, if the entity's display-name field exists, is stored, is a text field and is not
     translatable, add the condition "the display-name column matches *name* case-insensitively";
   - if the field examined was the normalised one, stop examining further address fields.
   Otherwise continue to the next name in the list.
3. **Indirect personal data.** For every stored single-link field of the entity that points at
   contacts and whose delete behaviour is **not** cascade, add the condition "that column is in the
   indirect references".
4. If there is at least one condition, add a selection over that entity's table whose filter is the
   **disjunction** of the conditions, selecting the entity identifier, the record identifier, and
   the active column when the entity has one (otherwise the constant true).

**The blacklist** contains: contacts and accounts (handled by parts 1 and 2); notifications,
followers and channel memberships (deleted by cascade anyway); and messages (handled by part 3).

### 11.3 Results

The rows are turned into one line per found record. The description groups them by entity:

```
<entity label> (<count>): #<identifier>, #<identifier>, …
```

with the entity's transport name appended after a hyphen when the acting user holds the *Technical
Features* group.

### 11.4 Worked example

Searching for *Marie Dubois*, address `marie.dubois@acme.test`, in a database containing the
accounting and sales packages.

1. `email_normalized = "marie.dubois@acme.test"`, `email_pattern = "%marie.dubois@acme.test%"`,
   `name = "Marie Dubois"`.
2. The indirect references resolve to one contact, identifier 1904.
3. Part 1 yields (contacts, 1904, active).
4. Part 2 yields nothing if no account has that login and the contact has no account.
5. Part 3 yields every message authored by contact 1904.
6. Part 4, for the Sales Order entity: it has no address field among the four, so step 2 adds
   nothing; step 3 finds the customer link, the invoice-address link and the delivery-address link
   (all single links to contacts that are not cascade-deleting) and adds three conditions; the
   selection therefore finds every order addressed to that contact.
7. Part 4, for the Customer Invoice entity: likewise through its contact link.
8. Part 4, for the mailing-trace entity: `email` exists and the entity is the mailing trace, so the
   condition is an **equality** against the normalised address rather than a pattern match.

The description might read:

```
Contact (1): #1904
Message (37): #90211, #90244, …
Sales Order (4): #6120, #6133, #6180, #6242
Customer Invoice (3): #44012, #44051, #44098
```

---

## 12. Recycling collection

### 12.1 The collected set

For one recycling rule:

```formula
rule_filter = the rule's stored filter, or the always-true filter when it is empty or the empty list

if the rule has a time field and a non-zero delta and a delta unit then
    now   = today, when the time field is a date
          = the current moment, when the time field is a date and time
    limit = now − delta × ( one unit of the delta unit )
    rule_filter = rule_filter and ( the time field ≤ limit )

candidates = the records of the rule's entity satisfying rule_filter
             ( with archived records included when the rule says so )
```

The subtraction uses **calendar** arithmetic for months and years: subtracting one month from 31
March yields 28 or 29 February, not 3 March.

Records that already have a candidate row for this rule — **active or discarded** — are excluded, so
a discarded candidate is never proposed again.

### 12.2 Batching

| Mode | Batch size | Behaviour |
|---|---|---|
| automatic | 5 000 | each batch is created and immediately validated; a commit follows each batch unless the run is a test |
| manual | 50 000 | candidates of every rule are accumulated, then created in batches; a commit follows each batch unless the run is a test |

### 12.3 Worked example — a rule collecting records older than ninety days

A rule is defined on the Lead entity: time field *Creation Date* (a date and time), delta 90, delta
unit *Days*, filter `[('active', '=', False)]` (that is, archived leads only), include archived set,
mode *manual*, action *Delete*, notify weekly.

Today is 11 September 2026 at 06:00 coordinated universal time.

1. `rule_filter` starts as "the archive flag is false".
2. The time field is a date and time, so `now = 2026-09-11 06:00:00`.
3. `limit = 2026-09-11 06:00:00 − 90 days = 2026-06-13 06:00:00`.
4. The filter becomes "the archive flag is false **and** the creation moment is at or before
   2026-06-13 06:00:00".
5. Because the rule includes archived records, the search is made with the archive filter disabled —
   which is necessary here, since the filter explicitly asks for archived records.
6. Suppose the search returns 61 342 records, of which 1 200 already have candidate rows. 60 142 new
   candidate rows are created, in two batches of 50 000 and 10 142, with a commit after each.
7. Because the mode is manual, nothing is deleted. The rule's outstanding candidate count becomes
   61 342 (the 1 200 pre-existing ones plus the new ones, less any that were discarded, since
   discarded rows are inactive and the count is over the active ones).

Had the mode been *automatic*, the 60 142 new rows would have been created and validated in batches
of 5 000: each batch reads the pointed-at records, deletes them with elevation, and then deletes the
candidate rows themselves.

### 12.4 The notification cadence

For each manual rule with at least one notified user and a non-zero frequency:

```formula
delta = frequency × ( one day | one week | one month , per the frequency period )
due   = ( last_notification is absent ) or ( last_notification + delta < current_moment )
```

When due, the last-notification moment is set to the current moment and a notice is sent. The notice
counts the candidates of that rule created **at or after** `today − delta`; when that count is zero,
no notice is sent at all (the recipient list becomes empty).

**Worked example.** Frequency 2, period *Weeks*, last notification 20 August 2026, current moment
11 September 2026.

```
delta = 2 weeks = 14 days
20 August 2026 + 14 days = 3 September 2026 < 11 September 2026  →  due
last_notification ← 11 September 2026
counted candidates = those created at or after 11 September 2026 − 14 days = 28 August 2026
```

---

## 13. Privacy-log masking

```formula
mask_word( w )   = the first character of w , followed by ( length(w) − 1 ) asterisks
mask_name( s )   = mask_email( s )                                  when s contains "@"
                 = the words of s, each masked with mask_word, rejoined with single spaces
mask_local( l )  = the full-stop-separated pieces of l, each masked with mask_word,
                   rejoined with full stops
mask_domain( d ) = d                                                when d is one of
                                                                    gmail.com, hotmail.com, yahoo.com
                 = the labels of d, all but the last masked with mask_word, the last kept,
                   rejoined with full stops
mask_email( a )  = mask_local( the part before "@" ) + "@" + mask_domain( the part after "@" )
```

Empty words and empty pieces are dropped rather than producing a bare asterisk.

**Worked examples.**

| Input | Output |
|---|---|
| `Marie Claire Dubois` | `M***** C****** D*****` |
| `marie.dubois@acme-corp.example` | `m****.d*****@a*********.e******` |
| `marie@gmail.com` | `m****@gmail.com` |
| `marie.dubois@acme.test` (as a name) | `m****.d*****@a***.t***` — because the name contains an at-sign, it is masked as an address |

---

## 14. Assorted derived values

### 14.1 The trusted-browser age

```formula
days = the parameter auth_totp.trusted_device_age interpreted as a whole number
if days ≤ 0 or the parameter is unparseable then days = 90 and a warning is logged
trusted_device_age_seconds = days × 86400
```

Default: 90 × 86 400 = 7 776 000 seconds.

### 14.2 The device-log write threshold

A trace already present in the session produces a new log row only when

```formula
current_moment_seconds − trace.last_activity_seconds ≥ 3600
```

so at most one row per hour per (session, platform, browser, network address).

### 14.3 The company colour

```formula
colour = the root company's contact colour , when it is set
       = the root company's identifier mod 12 , otherwise
```

### 14.4 The company logo for the web

The logo is resized to **180 points wide**, the height being derived from the aspect ratio (a height
of zero means "compute it"), and the result is stored in the row rather than as an attachment
because it is read directly by a low-level handler.

### 14.5 The identity-confirmation window

```formula
confirmation_is_fresh = ( session.identity_check_last > current_moment_seconds − 600 )
```

That is, ten minutes.

### 14.6 The unregistered-account reminder window

```formula
window_start = today − 5 days
window_end   = window_start + 1 day
```

Accounts whose creation moment lies in `[window_start, window_end)` and which have never signed in
are reported to their creator, in batches of 100 creators' worth of accounts.

### 14.7 The programmatic key limit

```formula
limit = the parameter base.programmatic_api_keys_limit interpreted as a whole number,
        falling back to 10 when absent or unparseable (a warning is logged on an unparseable value)
refuse a new programmatic key when the count of the user's unexpired keys ≥ limit
```

A key counts as unexpired when its expiry is empty **or** is at or after the current moment on the
database connection.

### 14.8 The presentation order of the groups of a privilege

```formula
rank( g ) = ( number of groups of this privilege in superset_closure( { g } ) , when g has a
              privilege ; 0 otherwise ,
              g.sequence ,
              g.identifier )
```

sorted ascending. Because a more powerful group implies the less powerful ones of the same
privilege, its closure contains more of them, so it sorts later — which puts the weakest choice
first in the selector.

**Worked example.** Privilege *Sales* with groups 10 (*User: own documents only*, sequence 10), 11
(*User: all documents*, sequence 20) and 12 (*Administrator*, sequence 30), implying one another as
in section 1.4.

| Group | Closure ∩ privilege groups | Count | Rank |
|---|---|---|---|
| 10 | {10} | 1 | (1, 10, 10) |
| 11 | {11, 10} | 2 | (2, 20, 11) |
| 12 | {12, 11, 10} | 3 | (3, 30, 12) |

Order: 10, 11, 12.

### 14.9 The default groups of a new internal account

```formula
default_groups = { Role / User } ∪ implied_ids( the group "Default access for new users" )
```

Note that this uses the **direct** implied groups of that group, not its closure, and that the group
itself is not included.

### 14.10 The multi-company group membership rule

```formula
count = the number of the account's permitted companies
if count ≤ 1 and the account explicitly holds "Multi Companies"      then remove it
if count > 1 and the account does not explicitly hold "Multi Companies" then add it
```

Applied at creation, on every write touching the permitted companies, and on the in-memory record
used by the form.
