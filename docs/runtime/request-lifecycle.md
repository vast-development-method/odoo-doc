# Request lifecycle

A request arrives at the server process as a network message and leaves it as a response. Between those two points the platform performs a fixed, ordered sequence of steps: it prepares per-request bookkeeping, rewrites the client address when a trusted reverse proxy is in front, wraps the raw message, resolves a session and a database, decides between three serving branches (a static file, a database-free endpoint, a database-backed endpoint), acquires an entity registry and a database transaction, resolves the path to an endpoint, applies the authentication level declared by that endpoint, builds the execution context (acting user, language, time zone, company), dispatches to the endpoint, and turns the return value or the raised exception into a response. This document specifies that sequence completely, including the read-only optimization, the retry behaviour, the error envelopes and the limits enforced on the way in. The wire spellings of paths, headers and envelope members are the contract of [`../interfaces/remote-transport-contracts.md`](../interfaces/remote-transport-contracts.md); this document specifies the behaviour around them.

## 1. Vocabulary

| Term | Meaning |
|---|---|
| Request layer | The component that turns an incoming network message into a call on an endpoint and back into a response. |
| Endpoint | A callable bound to one or more path patterns with a declared transport family, authentication level and options. |
| Serving branch | One of the three mutually exclusive ways a request is handled: static asset, database-free, database-backed. |
| Entity registry | The per-database in-memory description of every entity, field and relation, plus the per-database cache containers. See [`caching.md`](caching.md). |
| Transaction | One database transaction, opened for a request and closed when the request ends. See [`transactions-and-concurrency.md`](transactions-and-concurrency.md). |
| Cursor | The handle on the open transaction. A cursor is either read-only (opened against a read replica) or read/write (opened against the primary). |
| Deferred response | A holder of headers and cookies accumulated while the request runs, merged into the final response at the post-dispatch step. |
| Execution context | The acting user, the language, the time zone, the active companies and the free key-value pairs that record operations read. |
| Acting user | The user whose permissions apply to the request. |
| Read-only intent | The declaration, made by an endpoint, that serving it needs no write to the database. |
| Persistent socket | The long-lived bidirectional connection used by the notification bus. See [`notification-bus.md`](notification-bus.md). |

## 2. Per-request bookkeeping

Before anything else the request layer resets, on the handling thread, the following counters and markers. They are observable through the logs and through the profiling records, and a replacement must maintain equivalents because several later steps read them.

| Marker | Initial value | Purpose |
|---|---|---|
| Query count | `0` | Number of statements issued on behalf of this request. |
| Query time | `0` | Accumulated statement duration. |
| Start instant | Current monotonic instant | Basis for the request duration and for the runtime watchdog. |
| Cursor mode | unset | Set later to one of `rw` (read/write throughout), `ro` (read-only throughout) or `ro->rw` (escalated from read-only to read/write). See section 7. |
| Database name | cleared | Set once a database has been resolved. |
| Acting user key | cleared | Set once an execution context exists. |
| Remote operation name | empty text | Set by the generic dispatch endpoint to the entity transport name, a full stop, and the operation name. |

The three cursor-mode markers are reproduced values: they appear verbatim in log lines and in the performance profiles of [`logging-and-audit.md`](logging-and-audit.md).

## 3. Trusted proxy handling

When the deployment declares that a reverse proxy is in front and the incoming message carries a forwarded-host indication, the request layer rewrites, in place and before the request object is built, the client address, the scheme and the host from the first entry of the corresponding forwarded headers, counting exactly one proxy hop for each of those three values. When the deployment does not declare a proxy, the forwarded headers are ignored entirely and the direct peer address is used.

The client address obtained in this way is the value used for: the geographic resolution (section 12), the login failure counter (section 15), the device trace of the session (see [`sessions-and-authentication.md`](sessions-and-authentication.md), section 7) and the login log lines.

## 4. Request object and inbound limits

The raw message is wrapped into a request object that exposes the method, the path, the query arguments, the form fields, the uploaded files, the headers, the cookies and the body. Two limits are applied at this point.

| Limit | Default | Overridable by |
|---|---|---|
| Maximum content length of the whole body | 134 217 728 bytes (128 mebibytes) | The database-wide system parameter `web.max_file_upload_size` (maximum upload size for the web client), read at entity-level pre-dispatch (section 10.2), then the per-endpoint declaration. |
| Maximum size of a form part held in memory | 10 485 760 bytes (10 mebibytes) | Not overridable. A part larger than this is spooled to a temporary file rather than refused. |

A body larger than the effective maximum content length is rejected with the payload-too-large status and the endpoint is never called.

The request object also carries an empty deferred response. Every header or cookie set while the request runs is written on the deferred response and merged into the real response during post-dispatch (section 13). This exists because the endpoint may return a response object built long after those headers were decided.

## 5. Session and database resolution

This step runs before any branch decision, because the branch depends on whether a database could be resolved.

1. Read the session cookie. If it is absent, or if its value does not match the session identifier format (84 characters from the web-safe alphabet consisting of the letters, the digits, the hyphen and the underscore), create a brand-new empty session with a freshly generated identifier. Otherwise load the stored session with that identifier; when nothing is stored under it, an empty session carrying that same identifier is used, therefore an unknown but well-formed cookie is not an error.
2. Fill every missing key of the session with its default (see [`sessions-and-authentication.md`](sessions-and-authentication.md), section 2).
3. If the session context has no language, set it to the request language preference resolved from the accepted-languages header, falling back to `en_US` (English as used in the United States). See section 11.1.
4. Resolve the database with the following precedence.

| Order | Condition | Result |
|---|---|---|
| 1 | The session names a database **and** that name passes the deployment's database filter for the requested host | The database is the one named by the session. If the request also carries the database-selection header with a different name, the request is refused with the forbidden status and the reproduced message `Cannot use both the session_id cookie and the x-odoo-database header.`, which names the session cookie and the database-selection header by their contractual spellings. |
| 2 | The session names no usable database **and** the request carries the database-selection header | The session is marked non-persistable (it will never be written back to storage, and no session cookie is emitted), and the database is the header value if it passes the filter, otherwise none. |
| 3 | Neither of the above | The list of databases visible to the process is read; if it contains exactly one entry, that entry is the database, otherwise no database is resolved. |

5. If the resolved database differs from the one recorded in the session and the session recorded one, the platform logs the warning `Logged into database 'alpha', but dbfilter rejects it; logging session out.`, in which `alpha` stands for the database name recorded in the session, and performs a full logout of the session (clearing the acting user, the login, the session token and the context) without keeping the database. The resolved database is then written on the session.
6. The session is marked clean at the end of this step, therefore the mere act of filling defaults never causes a write to session storage.

**The database filter.** The deployment may declare a pattern in which the placeholder for the full host name and the placeholder for the first label of the host name are substituted, after stripping any port and any leading `www.` from the host. Only databases whose name matches the resulting pattern from the start are visible. When no pattern is declared but an explicit list of database names is declared, the visible set is the intersection of the existing databases with that list, sorted. When neither is declared, every existing database is visible. When the database server cannot be reached at all, the visible list is empty rather than an error.

## 6. Branch decision

| Order | Condition | Branch |
|---|---|---|
| 1 | The path resolves to a file inside the static directory of an installed capability package | Static asset serving (section 6.1). |
| 2 | A database was resolved | Database-backed serving (section 7). |
| 3 | Otherwise | Database-free serving (section 6.2). |

If the database-backed branch fails to obtain a usable registry, it degrades to the database-free branch (section 7.1).

### 6.1 Static asset serving

The path has the shape of a package name, then `/static/`, then a resource path. The package name is mapped to the absolute static directory of that package; when no such package exists the request is answered with the not-found status and the message `Module "<package>" not found.`, in which the placeholder is the package name taken from the path. The resource part is joined to that directory with a safe join that refuses any traversal outside it. The file is then streamed (see [`attachments-and-file-store.md`](attachments-and-file-store.md), section 9) with:

- a cache lifetime of 604 800 seconds (one week), unless the session carries the asset development flag and the requesting agent is not the document-rendering engine, in which case the cache lifetime is zero;
- no content security policy of its own, but the generic response hardening of section 13.3 still applies;
- the resource advertised as cacheable by shared caches.

A missing file or a permission problem while opening it is answered with the not-found status and the message `File "<resource>" not found in module <package>.`, in which the first placeholder is the resource path and the second is the package name.

Static asset serving never opens a database transaction, never touches the session and never emits a session cookie.

### 6.2 Database-free serving

Only endpoints whose declared authentication level is `none` are published in the database-free routing table, and only from capability packages loaded process-wide. The steps are:

1. Bind the database-free routing table to the request and match the path. On no match, answer with the not-found status and a small page whose text is `Not Found` as a heading, then `No database is selected and the requested address was not found in the server-wide controllers.`, then an invitation to verify the host name and to log in.
2. Select the dispatcher for the transport family declared by the matched endpoint, checking compatibility with the request media type (section 9.1).
3. Run the generic pre-dispatch step (section 10.1). The entity-level pre-dispatch step of section 10.2 is skipped, because there is no registry.
4. Call the endpoint with the deserialized parameters.
5. Run the post-dispatch step (section 13).

## 7. Database-backed serving

### 7.1 Acquiring the registry

The registry for the resolved database is obtained. If it is not resident, it is built (every installed capability package is loaded into it). The registry keeps a last-used instant that is refreshed on every acquisition. A bounded number of registries is kept resident; the least recently used is evicted when the bound is exceeded.

A read-only cursor is opened (section 7.2) and the registry's signalling check runs on it, which may reload the registry or clear cache containers (see [`caching.md`](caching.md), section 10).

If acquiring the registry or the cursor fails because the database is unreachable, does not exist or is not a platform database, the request layer logs `Database or registry unusable, trying without`, drops the database from the request, logs the session out, and falls back to database-free serving. When the requested path is one of the entry paths that require a database (the desktop client entry path and its sub-paths, the login path, and the bare client path), the request is additionally rewritten in place to the same path with the database query argument removed, in order that the database-free page does not loop on a database that cannot be opened.

### 7.2 Choosing the cursor

The cursor for the whole request is chosen as follows.

1. If the deployment declares a read replica, a read-only cursor is attempted on it. A failure to connect records a failure instant and the replica is not attempted again for 1 200 seconds (20 minutes); during that window every cursor request returns a read/write cursor on the primary and the cursor mode marker is set to `ro->rw`.
2. If no replica is declared, a read/write cursor on the primary is returned and the cursor mode marker is set to `ro->rw`.

The isolation level of every cursor is repeatable read (see [`transactions-and-concurrency.md`](transactions-and-concurrency.md), section 3).

### 7.3 Resolving the path

An empty execution context is created on the cursor with the acting user taken from the session (possibly none) and the context taken from the session. The routing table of the database is then matched against the path.

- The routing table is built from every installed capability package plus the process-wide packages, and is cached in the registry's routing cache container keyed by an optional discriminator (used by multi-site deployments).
- Matching is not sensitive to a trailing slash, and consecutive slashes in the path are **not** merged.
- Every declared method of a rule is augmented with the preflight method, therefore a cross-origin preflight request always reaches the rule.
- Path converters may turn a path segment into a record of a named entity; those records are resolved lazily against the request's execution context and are access-checked later, at pre-dispatch (section 10.2).

**On a match**, the dispatcher for the declared transport family is selected and compatibility with the request media type is verified (section 9.1). The serving function is the matched-endpoint function, and the read-only intent is the endpoint's declared read-only flag; when that flag is a rule rather than a constant, it is evaluated now with the controller instance, the matched rule and the path arguments, and yields a true or false value.

**On no match**, the serving function is the fallback function (section 8) and the read-only intent is true.

### 7.4 Running under the chosen cursor

The chosen serving function is executed by the following procedure. Every step is observable.

1. If the read-only intent is true **and** the cursor obtained in section 7.2 is a read-only cursor, set the cursor mode marker to `ro` and go to step 2. Otherwise set the cursor mode marker to `rw` and go to step 6.
2. Run the serving function inside the retry wrapper of section 7.5.
3. If the run succeeded, its result is the result of the request. The procedure ends.
4. If the run failed with a read-only-transaction violation, log a warning whose text is the violation's own message followed by `, retrying with a read/write cursor`, set the cursor mode marker to `ro->rw`, and go to step 6.
5. If the run failed with any other error, attach an error response to that error (section 14.1) and re-raise it. The procedure ends.
6. Prepare a read/write transaction. If the current cursor is read-only, close it and open a new read/write cursor on the primary. If the current cursor is already read/write, roll it back, which starts a fresh transaction on the same connection.
7. Rebind the execution context to the read/write cursor, so that the acting user, the language and the company carry over.
8. Run the serving function inside the retry wrapper of section 7.5. If it succeeds, its result is the result of the request. If it fails, attach an error response to the error and re-raise it.
9. Whichever branch was taken, and whether or not an error was raised, close the cursor exactly once and drop the execution context in a final step.

Three consequences are observable and must be reproduced.

1. An endpoint declared read-only that nevertheless writes is **not** an error for the client: the whole request is replayed from the start of the serving function on a read/write cursor. Everything the first attempt read is discarded. The endpoint must therefore be free of side effects outside the database, or it will perform them twice.
2. The rollback in step 6, when the cursor was already read/write, exists because the signalling check of section 7.1 ran in the same transaction. Starting a new transaction there prevents an avoidable serialization failure that the retry loop would only mask.
3. The cursor is closed exactly once, in step 9.

### 7.5 The retry wrapper

Both attempts of section 7.4 run inside the retry wrapper specified in [`transactions-and-concurrency.md`](transactions-and-concurrency.md), section 8. For the request lifecycle the salient points are:

- up to five attempts; between attempt *n* and attempt *n+1* the process waits a uniformly distributed random delay in the interval from zero to two raised to the power *n*, expressed in seconds;
- before each new attempt the transaction is rolled back, the transaction state is reset, the registry invalidations of the failed attempt are cancelled, **the session is re-read from storage**, and every uploaded file of the request is rewound to its beginning;
- if an uploaded file cannot be rewound, the retry is abandoned with the error `Cannot retry request on input file '<name>' after serialization failure`, in which the placeholder is the name of the uploaded file;
- after the successful attempt the transaction is flushed, committed, and the registry invalidations are signalled to the other processes.

## 8. Fallback serving

When no endpoint matched the path:

1. The request parameters are set to the merged query arguments, form fields and uploaded files.
2. The public authentication level is applied, therefore the acting user becomes the public user when the session carries no user.
3. The fallback resolution is invoked. The platform's own fallback looks for an attachment of type `binary` whose declared address equals the request path; when such an attachment exists and has content (either stored in the file store or stored in the database), it is streamed as the response.
4. Capability packages may extend the fallback resolution to serve content managed by them, for example site pages.
5. If the fallback produced a response, the post-dispatch step runs on it and it is returned.
6. If no fallback produced a response, a not-found error is raised whose cause is recorded as the original no-match error, and an error response is attached to it.

Fallback serving always runs with the read-only intent.

## 9. Transport families and parameter deserialization

Three transport families exist. Their envelopes are specified in [`../interfaces/remote-transport-contracts.md`](../interfaces/remote-transport-contracts.md). The lifecycle-relevant behaviour is:

| Family | Where the parameters come from | What the return value becomes |
|---|---|---|
| Page transport | The query arguments, the form fields and the uploaded files, merged in that order, then overlaid with the path arguments. | Text, bytes or nothing become a response body; a response object is passed through; a deferred template response is rendered at dispatch time. |
| Enveloped remote call | The `params` (parameters) member of the request body object, overlaid with the path arguments. The `method` (operation name) member is ignored because the path already selected the endpoint. Only by-name parameters are supported. | Wrapped into a success envelope carrying the same correlation value as the request. |
| Direct remote call | The whole request body object, overlaid with the path arguments; an empty body yields only the path arguments. | Serialized directly as the body, unless the endpoint already returned a response object. |

### 9.1 Compatibility check

Each family declares the request media types it accepts. The page family accepts every media type. The enveloped family accepts only the two structured-data media types. The direct family accepts the structured-data media type, and also accepts a request with no body at all.

If the matched endpoint's family is not compatible with the request, the request is refused with the unsupported-media-type status, an `Accept` response header listing the family's media types, and a body whose first line is `Request inferred type is compatible with [<compatible families>] but '<path pattern>' is type='<declared family>'.`, followed by a blank line, followed by `Please verify the Content-Type request header and try again.`. The first placeholder lists the families the request body would suit, the second is the matched path pattern and the third is the family the endpoint declares.

A cross-origin preflight request skips this check entirely.

## 10. Pre-dispatch

### 10.1 Generic pre-dispatch

Performed for every matched endpoint, in both the database-free and the database-backed branch, in this order:

1. **Session persistence flag.** The session's persistable flag is combined by logical conjunction with the endpoint's save-session declaration. An endpoint that declares it does not save the session therefore makes the whole request stateless even when the session was loaded from a cookie.
2. **Cross-origin headers.** If the endpoint declares a cross-origin value, the allow-origin header of the deferred response is set to that value, and the allowed-methods header is set to `POST` for the enveloped family, or to the endpoint's declared methods (defaulting to the list `GET`, `POST`) for the other families.
3. **Preflight short-circuit.** If a cross-origin value is declared and the request method is the preflight method, the deferred response additionally receives a maximum age of 86 400 seconds (one day) and an allowed-headers list whose reproduced value is `Origin, X-Requested-With, Content-Type, Accept, Authorization, Range`, and the request is terminated immediately with a no-content response. The endpoint is not called.
4. **Per-endpoint body limit.** If the endpoint declares a maximum content length, it replaces the effective limit of section 4. When that declaration is a rule rather than a constant, it is evaluated with the controller instance.

### 10.2 Entity-level pre-dispatch

Performed only in the database-backed branch, after authentication, in this order:

1. **Database-wide upload limit.** The system parameter `web.max_file_upload_size` is read as the acting superuser. When present, its integer value replaces the request's maximum content length. A non-integer value is logged as `invalid web.max_file_upload_size: <value>, using <effective> instead`, in which the first placeholder is the stored text and the second the limit actually applied, and is otherwise ignored, never failing the request. This happens **before** the generic pre-dispatch, therefore a per-endpoint declaration takes precedence over the database-wide parameter.
2. The generic pre-dispatch of section 10.1 runs.
3. **Language validation.** The language currently in the execution context is validated and replaced by a valid one following the rule of section 11.2. When the request has no acting user yet, the validation is performed with superuser rights on a neutral entity.
4. **Path record rebinding.** Every path argument that resolved to a record is rebound to the request's execution context, therefore its access checks and its derived values use the final acting user, language and company.
5. **Path record access check.** Every path argument that is a record is checked for read access. On an access failure or a missing-record failure:
   - if the endpoint declares a handler for parameter access errors, that handler is called with the error and the path arguments, and if it returns a response the request terminates with it;
   - otherwise, when the acting user is the public user, or when the failure is a missing record, the request is answered with the not-found status (deliberately not disclosing whether the record exists);
   - otherwise the access error propagates.

### 10.3 The four authentication levels

The level is declared by the endpoint and applied between the generic pre-dispatch and the dispatch. The levels and their effects are specified in full in [`sessions-and-authentication.md`](sessions-and-authentication.md), section 8; the lifecycle sees them as follows.

| Level | Stored value | Effect on the request |
|---|---|---|
| No authentication | `none` | No session token is checked and no acting user is set. The endpoint is published in both routing tables. |
| Public | `public` | The session's user is used when present; otherwise the acting user becomes the public user of the database. Never refuses. |
| Any user | `user` | The session must carry a user and a matching session token. Otherwise a session-expired error is raised and handled by section 14.3. |
| Bearer credential | `bearer` | An authorization header carrying an application key is accepted in place of a session; a valid key sets the acting user. Failure raises a session-expired error. |

## 11. Language and time zone resolution

### 11.1 The request language preference

The accepted-languages header is parsed and the best entry is selected. The entry is normalized as follows: if it names a territory, the result is the language code, an underscore, and the territory code in capital letters; if it names only a language, the result is the canonical locale alias for that language. If parsing fails or no entry exists, there is no request preference and the fallback `en_US` is used.

### 11.2 The effective language of a request

The effective language is resolved in this order and the **first installed** candidate wins:

| Order | Candidate |
|---|---|
| 1 | The language present in the execution context, when it names an installed language. |
| 2 | The language of the contact record of the acting user's current company, when it names an installed language. |
| 3 | `en_US`, when it is installed. |
| 4 | The first installed language, in the platform's own ordering of language records. |

The resolved value is written back into the execution context of the request.

A second, related rule applies when a session is finalized at login: the user's own language and time zone are read and placed in the session context. That rule resolves the language as follows and the first installed candidate wins: the user's own language; the request language preference of section 11.1; the language of the contact record of the user's company; `en_US`; the first installed language. The same operation also writes the acting user key into the session context.

The complete fallback chain used when a term has no translation in the effective language is specified in [`translation.md`](translation.md), section 4.

### 11.3 The time zone

The time zone of a request is the value of the `tz` (time zone) key of the execution context when present, otherwise the time zone recorded on the acting user. A value that is not a recognized zone name is discarded with a debug-level note and coordinated universal time is used. When neither is set, coordinated universal time is used.

Every date-and-time value is stored and manipulated in coordinated universal time. Conversion to the request time zone happens only for display and for the few computations that are explicitly defined in local terms, namely the rescheduling of a scheduled action across a daylight-saving boundary (see [`scheduled-jobs.md`](scheduled-jobs.md), section 6.3) and the resolution of date placeholders in a numbering sequence (see [`transactions-and-concurrency.md`](transactions-and-concurrency.md), section 7.4).

At the first interactive login of a user, and at any login of a user who has no time zone recorded, the time zone advertised by the client in a dedicated cookie is written onto the user record when it names a recognized zone.

### 11.4 Context construction

The execution context of a database-backed request is built in three layers.

| Layer | Contents |
|---|---|
| Session context | Written at login: the user's language, the user's time zone, the acting user key. Persisted in the session and reused by every later request of that session. |
| Request overlay | The effective language of section 11.2; the debug flags extracted from the query string by the desktop client entry endpoints; any key an endpoint chooses to add before dispatching. |
| Call overlay | For the enveloped and direct remote call families, the caller may supply a context member that **replaces** the session context for the duration of the call. It is removed from the parameters before the endpoint is called. |

The context is immutable during a call; changing it produces a new execution context bound to the same transaction and the same cursor.

### 11.5 The active companies

The set of companies whose records the request may see is taken from the companies cookie when present and intersected with the companies the acting user is allowed to use; the first entry of the result is the current company. When the cookie is absent or its intersection is empty, the user's own default company is the only active company. The rules that govern the allowed set are specified in [`../overview/security-model.md`](../overview/security-model.md).

## 12. Geographic resolution

The client address of section 3 may be resolved to a country and a city when the deployment supplies the corresponding databases. The resolution is lazy: nothing is read until a consumer asks for a country, a city, a subdivision, coordinates or a zone name. A failure of any kind, including an address that is absent from the data, yields an empty result whose every attribute reads as unset rather than an error. The country name falls back to the continent name and the country code falls back to the continent code when the country itself is unknown.

The resolution is consumed by: the device trace of the session (country and city), the localization defaults of public sign-up pages, and any capability package that keys behaviour on the visitor's country.

## 13. Post-dispatch

The post-dispatch step runs for every branch, on the response produced by the endpoint, in this order:

1. **Save the session** (section 13.1).
2. **Merge the deferred response**: every header accumulated on the deferred response is appended to the real response.
3. **Harden the response** (section 13.3).

### 13.1 Saving the session

The first matching row of the following table decides what happens; rows are evaluated top to bottom.

| Order | Condition | Action |
|---|---|---|
| 1 | The session is not persistable | Nothing is written. |
| 2 | The session is marked for rotation | Hard rotation, then store. |
| 3 | The session has an acting user, the current instant is at or after the session creation instant plus 10 800 seconds (three hours), the path is not one of the rotation-excluded paths, and the skip-rotation header is absent | Soft rotation, then store. |
| 4 | The session is dirty | Store. |
| 5 | Otherwise | Nothing is written. |

Then, if the session is dirty or the cookie value differs from the current session identifier, a session cookie is written on the deferred response with the current identifier, marked inaccessible to client scripts, and with a maximum age equal to the effective session inactivity limit (section 13.2).

The rotation-excluded paths are the three persistent-socket lifecycle paths: the close notification path, the notification polling fallback path and the presence update path. Rotation mechanics are specified in [`sessions-and-authentication.md`](sessions-and-authentication.md), section 5.

### 13.2 The effective session inactivity limit

The limit is the integer value of the system parameter `sessions.max_inactivity_seconds` (maximum session inactivity in seconds), defaulting to 604 800 seconds (seven days) when the parameter is absent, when no usable execution context exists, or when the stored value is not an integer. A non-integer value is logged as `Invalid value for 'sessions.max_inactivity_seconds', using default value.`.

### 13.3 Response hardening

Every response, including static files and streamed attachments, receives the header that disables content-type sniffing. In addition, a response whose media type starts with the image family and that does not already carry a content security policy receives the most restrictive policy, which forbids the resource from issuing any request of its own.

Streamed responses receive their own policy at creation time (see [`attachments-and-file-store.md`](attachments-and-file-store.md), section 9).

## 14. Error paths

### 14.1 Where an error is turned into a response

An error raised inside the serving function is caught at the boundary of section 7.4 and an error response is attached to it, produced by the error handler of the transport family in force. The attached response is then invoked at the outermost level. An error raised outside the serving function (for example while resolving the session) reaches the outermost level without an attached response, and the handler of the default transport family (the page family, which is the family in force until an endpoint has matched) produces one there.

A protocol-level exception that carries no status code is not an error at all: it is the mechanism by which an endpoint aborts with a ready-made response (the preflight short-circuit of section 10.1 uses it). Such an exception is unwrapped into its response and the post-dispatch step runs on it.

### 14.2 Logging of errors

| Error kind | Log level | Traceback |
|---|---|---|
| An error carrying its own declared level | the declared level | only when the error carries one |
| A protocol-level exception with a status | not logged | no |
| A session-expired error | informational | no |
| An access error | warning | only in access-debugging mode |
| A business rule violation | warning | no |
| Anything else | error | yes |

### 14.3 Error envelopes by transport family

**Page family.**

| Error | Response |
|---|---|
| Session expired | The session is logged out while keeping the database. A redirect to the login path is produced, carrying the original path and query as the return address. If the session had an acting user, the session is hard-rotated and a fresh session cookie is written on the redirect with the effective inactivity limit as its maximum age. |
| A protocol-level error with a status | Returned unchanged. |
| A business rule violation | Translated into the protocol error whose status the violation declares; when the violation declares no usable status, the unprocessable-content status is used, with the violation message as the body. |
| Anything else | The internal-error status with no detail. |

**Enveloped remote call family.** The response always has the success status; the failure is described inside the envelope, whose error member carries a numeric code, a message and a data object. The data object always contains the fully qualified error name, the message, the error arguments, the error context and the formatted traceback.

| Error | Code | Message |
|---|---|---|
| Not found | `404` | `404: Not Found` |
| Session expired | `100` | a session-expiry message |
| Anything else | `0` | a generic server-error message |

**Direct remote call family.** The response carries a real status and the body is the serialized error description.

| Error | Status | Body |
|---|---|---|
| A protocol-level error that already carries a response | that response, unchanged | unchanged |
| A business rule violation, or a session-expired error | the status declared by the error (the forbidden status for session expiry) | the error description |
| Another protocol-level error | its own status | the error description, with the description text as both message and first argument, and the status as the second argument; the media-type header of the original error is replaced, its other headers are kept |
| Anything else | internal error | the error description |

### 14.4 Body parse failures

For the enveloped family, a body that cannot be parsed produces the bad-request status and a plain-text body stating that the structured data is invalid, and a body that parses but is not an object produces the bad-request status and a plain-text body stating that the enveloped remote call data is invalid. Both bypass the error envelope entirely. For the direct family, an unparseable body produces the bad-request status and a plain-text body stating that the body could not be parsed as structured data, followed by the reason reported by the parser.

## 15. Rate limiting

The platform applies rate limiting in exactly three places. There is no general per-address request quota.

### 15.1 Interactive login cooldown

Keyed by client address, held in the memory of the worker process and therefore **not** shared between workers. A map from client address to a pair (failure count, last failure instant) is maintained.

- Before each authentication attempt, the cooldown rule is evaluated. Its default form reads two system parameters: `base.login_cooldown_after` (the number of failures after which the cooldown applies, default 5 in the rule and seeded to 10 at database creation) and `base.login_cooldown_duration` (the cooldown length in seconds, default 60 and seeded to 60). A configured threshold of `0` disables the feature entirely.
- The cooldown is in force when the recorded failure count is greater than or equal to the threshold **and** the elapsed time since the last failure is shorter than the duration.
- While the cooldown is in force, the attempt is not even evaluated: it is refused with the message `Too many login failures, please wait a bit before trying again.` and a warning is logged that names the address, the attempted login, the database, the failure count and the last failure instant. When the address is in a private range, an additional warning notes that the deployment may be misconfigured behind a proxy.
- A failed attempt increments the count and records the instant. A successful attempt removes the entry entirely.
- The same counter guards the creation of application keys and the verification of a second authentication factor.

**Worked example.** The threshold is 5 and the duration is 60 seconds. An address makes five failed attempts, the last at instant `T`. A sixth attempt at `T` plus 30 seconds is refused without evaluating the credentials. An attempt at `T` plus 61 seconds is evaluated; if it fails, the count becomes 6 and the instant becomes `T` plus 61 seconds, therefore the next window ends at `T` plus 121 seconds; if it succeeds, the entry is removed and the next failure starts counting from 1.

### 15.2 Persistent socket frame rate limiting

Each open persistent socket connection keeps the receive instants of its last *burst* incoming frames. When the buffer is full and the elapsed time between the oldest retained instant and now is shorter than the burst size multiplied by the delay, expressed in seconds, the connection is closed with the try-later close code. The burst size and the delay are deployment settings; see [`notification-bus.md`](notification-bus.md), section 6.

### 15.3 Personal outgoing mail server throttling

Outgoing messages sent through a mail server owned by an individual user are capped per rolling window; see [`mail-gateway.md`](mail-gateway.md), section 6.

## 16. Profiling hook

If the session carries a profiling session marker and the request has a database, the request body runs inside a profiler that records statement traces and stack traces. The profiler is not started when any of the following holds:

| Condition | Behaviour |
|---|---|
| The recorded profiling expiration is in the past | The marker is cleared and a warning `Profiling expiration reached, disabling profiling` is logged. |
| The path is the profiling-toggle path | Skipped silently at debug level. |
| The path belongs to the persistent socket family | Skipped silently at debug level. |
| The process is the event-driven worker | Skipped silently at debug level. |
| Creating the profiler raises | The failure is logged with its traceback and the marker is cleared. |

The resulting record is specified in [`logging-and-audit.md`](logging-and-audit.md), section 7.

## 17. Worked examples

### 17.1 A read-only listing that stays on the replica

Given a deployment with a read replica, a session authenticated as an internal user, and an endpoint declared read-only.

| Step | Observable effect |
|---|---|
| 1 | Bookkeeping reset; cursor mode unset. |
| 2 | Session loaded from the cookie; database taken from the session and accepted by the filter. |
| 3 | Registry acquired; read-only cursor opened on the replica; signalling check finds no change. |
| 4 | Path matches the endpoint; read-only intent is true; cursor is read-only, therefore cursor mode becomes `ro`. |
| 5 | Authentication level `user` passes because the session token matches. |
| 6 | Pre-dispatch validates the language, finds `fr_BE` (French as used in Belgium) installed, keeps it. |
| 7 | The endpoint reads records and returns them. |
| 8 | The retry wrapper flushes (nothing dirty), commits (a read-only transaction commit releases the snapshot and writes nothing) and signals no invalidation. |
| 9 | Post-dispatch: the session is clean and its identifier is unchanged, therefore no cookie is written; the sniffing-protection header is added. |
| 10 | The cursor is closed and returned to the replica pool. |

### 17.2 A read-only endpoint that writes

Same setup, but the endpoint records a visit counter.

| Step | Observable effect |
|---|---|
| 1 to 6 | Identical to 17.1; cursor mode is `ro`. |
| 7 | The write statement is refused by the read-only transaction. |
| 8 | A warning is logged, ending with `retrying with a read/write cursor`; cursor mode becomes `ro->rw`. |
| 9 | The read-only cursor is closed; a read/write cursor is opened on the primary; the execution context is rebound. |
| 10 | The whole serving function runs again from its first line, including authentication and pre-dispatch. |
| 11 | The counter is written, flushed and committed. |
| 12 | Post-dispatch and cursor close as in 17.1. |

The client sees one response and one status. The endpoint body ran twice.

### 17.3 A concurrent write that is retried

Two requests update the same record. The second request's flush fails with a serialization failure at attempt 1.

| Attempt | Wait before the attempt | Behaviour |
|---|---|---|
| 1 | none | Fails with a serialization failure; transaction rolled back; transaction state reset; registry invalidations cancelled; session re-read; uploaded files rewound. |
| 2 | a random duration between zero and two seconds | Fails again. |
| 3 | a random duration between zero and four seconds | Succeeds; flushed and committed. |

The informational log line of each failed attempt has the form `<error name>, <n> tries left, try again in <seconds> sec...`. If the fifth attempt also fails, the line `<error name>, maximum number of tries reached!` is logged and the failure is turned into a response by the transport family's error handler.

### 17.4 A request to an unknown path that an attachment serves

| Step | Observable effect |
|---|---|
| 1 | No routing rule matches the path `/brochure-2026`. |
| 2 | The read-only cursor is kept; the serving function is the fallback. |
| 3 | The public authentication level is applied; the acting user becomes the public user. |
| 4 | An attachment of type `binary` whose declared address equals `/brochure-2026` is found. |
| 5 | It is streamed with its own entity tag (its content checksum), its own media type and the download name taken from its name. |
| 6 | Post-dispatch runs on that response. |

If no attachment matched, a not-found error is raised and turned into a not-found page.

### 17.5 A cross-origin preflight

| Step | Observable effect |
|---|---|
| 1 | The path matches an endpoint that declares a cross-origin value; the preflight method was added to the rule automatically. |
| 2 | The media-type compatibility check is skipped. |
| 3 | Generic pre-dispatch sets the allow-origin header, the allowed-methods header, the maximum age of 86 400 seconds and the allowed-headers list. |
| 4 | The request terminates with the no-content status. The endpoint body never runs and no transaction work is done beyond acquiring the registry. |

## 18. Acceptance criteria

1. **Given** a request with a well-formed session cookie for which no stored session exists, **when** the request is served, **then** an empty session carrying that identifier is used and the request is not refused.
2. **Given** a session naming database `alpha` and a request carrying the database-selection header naming `beta`, **when** `alpha` passes the filter, **then** the request is refused with the forbidden status and the message `Cannot use both the session_id cookie and the x-odoo-database header.`.
3. **Given** a session naming database `alpha` that no longer passes the filter, **when** the request is served, **then** the session is logged out, the warning `Logged into database 'alpha', but dbfilter rejects it; logging session out.` is logged, and the database is resolved again from scratch.
4. **Given** exactly one visible database and a request with no session and no database header, **when** the request is served, **then** that database is used.
5. **Given** two visible databases and a request with no session and no database header, **when** the request is served to a path that is not static, **then** the database-free branch is used.
6. **Given** an endpoint declared read-only that performs a write, **when** a read replica exists, **then** the endpoint body executes exactly twice and the client receives exactly one successful response.
7. **Given** an endpoint declared read-only that performs a write, **when** no read replica exists, **then** the endpoint body executes exactly once because the cursor was already read/write.
8. **Given** a request whose body exceeds 128 mebibytes and an endpoint with no declared limit and no database-wide limit, **when** the body is read, **then** the request is refused with the payload-too-large status.
9. **Given** the system parameter `web.max_file_upload_size` set to `2000000` and an endpoint declaring a limit of `50000000`, **when** a body of 30 000 000 bytes is posted, **then** the request succeeds, because the per-endpoint declaration overrides the database-wide parameter.
10. **Given** an endpoint declaring a cross-origin value, **when** a preflight request arrives, **then** the response has the no-content status, carries the allow-origin, allowed-methods, maximum-age and allowed-headers headers, and the endpoint body is not executed.
11. **Given** an endpoint of the enveloped family, **when** the request body is not parseable, **then** the response has the bad-request status and a plain-text body stating that the structured data is invalid, without an envelope.
12. **Given** an endpoint of the enveloped family, **when** the endpoint raises a business rule violation, **then** the response has the success status and the envelope carries an error member whose data object contains the error name, the message, the arguments, the context and the traceback.
13. **Given** an authenticated page request, **when** the session token no longer matches, **then** the session is logged out keeping the database, the session identifier is rotated, a fresh session cookie is written, and the client is redirected to the login path with the original path and query as the return address.
14. **Given** a session created 3 hours and 1 second ago with an acting user, **when** any request other than the three persistent-socket lifecycle paths is served and the skip-rotation header is absent, **then** the session identifier changes while the session content is preserved, and a new cookie is written.
15. **Given** the same session, **when** the request path is the notification polling fallback path, **then** the session identifier does not change.
16. **Given** a path argument that resolves to a record the acting user may not read, **when** the acting user is the public user, **then** the response is not-found; **when** the acting user is an internal user, **then** the response reports the access error.
17. **Given** a request whose accepted-languages header names `fr-BE` and a database where `fr_BE` is installed, **when** the session carries no language, **then** the effective language of the request is `fr_BE`.
18. **Given** the same request and a database where `fr_BE` is not installed but `en_US` is, **when** the request is served, **then** the effective language is `en_US`.
19. **Given** five consecutive failed login attempts from one address with the threshold at 5 and the duration at 60 seconds, **when** a sixth attempt is made 30 seconds later, **then** it is refused with `Too many login failures, please wait a bit before trying again.` without evaluating the credentials.
20. **Given** a static asset request, **when** the session carries the asset development flag and the agent is an ordinary browser, **then** the response cache lifetime is zero; **when** the flag is absent, **then** it is 604 800 seconds.
21. **Given** a request that reaches a database whose registry cannot be built, **when** the path is the desktop client entry path with a database query argument, **then** the session is logged out, the request is rewritten to the same path without that argument, and the database-free branch answers it.
22. **Given** a static asset request for a package that is not installed in the process, **when** the request is served, **then** the response has the not-found status and the body `Module "<package>" not found.`.
23. **Given** a request whose path contains two consecutive slashes, **when** the routing table is matched, **then** the slashes are not merged and the path matches only a rule that spells them the same way.
24. **Given** an endpoint of the page family that declares it does not save the session, **when** the request completes, **then** no session cookie is written even though the session was loaded from one.
25. **Given** a request with no acting user reaching an endpoint whose authentication level is `public`, **when** the endpoint runs, **then** the acting user is the public user of the database and the request is not refused.

## 19. Reconciliation notes

1. The refusal raised when a session names one database and the database-selection header names another was stated in two different ways: as a paraphrase in the resolution procedure and as the exact message in the acceptance criteria. The exact message is the observable behaviour and is now used in both places; it reproduces the contractual spellings of the session cookie and of the database-selection header, which is why those two strings appear in code font.
2. The escalation from a read-only to a read/write cursor was expressed as a code-shaped sketch. It is restated as the numbered procedure of section 7.4 with no change of behaviour: the same two attempts, the same cursor-mode markers, the same single close.
3. The record cache, the unit of work and the recomputation engine were described in a separate overview document and in a separate runtime document. Both are now in [`caching.md`](caching.md), and every reference from this document points there.
4. The outgoing mail queue and the generic queue semantics were described in one document. They are now split between [`mail-gateway.md`](mail-gateway.md) and [`background-workers.md`](background-workers.md); the personal mail server throttling referred to from section 15.3 lives in the first of the two.
