# Remote transport contracts

Every capability of the system is reachable from outside the server process through one uniform request layer: a client sends a request to a path, the layer resolves a session or an application key, opens a database transaction, dispatches the request to the endpoint bound to that path, and serializes the result. This document specifies that layer completely: the declaration of an endpoint, the ordered request lifecycle, the three transport families and their envelopes, the two authentication schemes, the generic dispatch that exposes entity operations, every endpoint the desktop client relies on with its inputs and outputs, the request context, the batching limits, the retry semantics, the per-endpoint security checks, and the cross-site request forgery rules. The operations reachable through this layer are specified in [`service-layer.md`](service-layer.md); the exhaustive list of every path in the system is in [`endpoint-catalog.md`](endpoint-catalog.md) and [`../references/routes.md`](../references/routes.md).

## 1. Notation and vocabulary

### 1.1 Wire literals

Path patterns, header names, media types, cookie names, envelope member names and status codes are reproduced **verbatim** as data values, in code spans. They are the wire contract: a replacement that changes them is not interoperable with existing clients, therefore they are not translated into prose names. Everything else, including entity names, operation names and field names, follows the canonical naming of the specification.

Two families of literal are **deployment-chosen** rather than universal, because the reference behavior spells them with a product name: the desktop client entry path, written `/app` (with its sub-paths `/app/<path>`) throughout this specification, and the request header that selects a database, written `X-Database`. A deployment that must keep serving clients built against another spelling keeps that spelling; everything else about those two is specified here exactly.

Entity names appear inside request payloads as data. In worked examples the canonical entity identifier is used (`contact`, `sales_order`, `attachment`); in a real deployment the value carried on the wire is the technical registry name of the entity in the target system, whatever spelling that system uses. The same holds for field names inside payloads.

### 1.2 Terms

| Term | Meaning |
|---|---|
| Request layer | The component that turns an incoming network request into a call on an endpoint and turns the result into a response. |
| Endpoint | A callable bound to one or more path patterns, with a declared transport family, authentication level and options. |
| Controller | A named group of endpoints; a controller of one capability package may extend a controller of another, and their endpoint declarations then merge (section 2.4). |
| Transport family | One of three ways of carrying parameters and results: the page transport, the enveloped remote call transport, the direct remote call transport (section 3). |
| Envelope | The outer structure of a request or response body for the remote call transports. |
| Session | Server-side state identified by a cookie, holding the authenticated user, the database, the context and auxiliary flags (section 5). |
| Application key | A bearer credential that authenticates a user without a session and without a password (section 6.4). |
| Generic dispatch | The endpoint that invokes an arbitrary public operation of an arbitrary entity (section 8). |
| Read-only request | A request declared as not writing, served on a read-only database connection (section 4.6). |
| Deferred response | Headers and cookies accumulated during the request and merged into the final response at post-dispatch. |

### 1.3 Status codes used by the layer

| Code | Name | Emitted when |
|---|---|---|
| `200` | success | Any successful request, including an enveloped remote call that carries a business error inside its envelope. |
| `101` | protocol switch | A successful web socket handshake. |
| `204` | no content | A successful cross-origin preflight request on an endpoint that declares a cross-origin value. |
| `301`, `302`, `303`, `307` | redirection | Content served from an external address (`301`, or `302` when a maximum age is set), the database-selection redirect (`302`), the login redirect and the logout redirect (`303`), the canonical-parameters redirect of the structured browsing endpoints (`307`). |
| `304` | not modified | A conditional request whose validator matches the current entity tag or modification time. |
| `400` | bad request | An unparseable enveloped body, an invalid cross-site token, a malformed breadcrumb request. |
| `401` | unauthorized | A missing, invalid or expired application key on an endpoint whose authentication level is the application key. |
| `403` | forbidden | An access error, an authentication denial, an expired session on a remote call, a conflicting database header. |
| `404` | not found | An unknown path, an unknown entity, an unknown operation, a missing record on a public request. |
| `405` | method not allowed | A method that the endpoint does not declare. |
| `409` | conflict | A record lock that could not be taken. |
| `413` | payload too large | A body larger than the effective maximum content length. |
| `415` | unsupported media type | A body whose media type does not match the endpoint's transport family. |
| `422` | unprocessable content | A business rule violation (the generic user error), a wrong or missing operation argument, a validation failure. |
| `500` | internal error | Any other failure. |

## 2. Endpoint declaration

### 2.1 Declared attributes

An endpoint declaration binds a callable to paths and options. Every attribute below is part of the contract of the endpoint and is observable.

| Attribute | Type | Default | Effect |
|---|---|---|---|
| Paths | One path pattern or a list of them | Required; an endpoint without any path is not published, and the layer logs `"<name> is a controller endpoint without any route, skipping."` | The patterns that route to this endpoint. Patterns may contain converters (section 2.2). |
| Transport family | `http`, `jsonrpc`, `json2` | `http` | How parameters are read from the request and how the result is serialized (section 3). |
| Authentication level | `none`, `public`, `user`, `bearer` | `user` | Who may call it and which user the call runs as (section 6.1). |
| Methods | List of methods | All methods | The methods the endpoint answers. `OPTIONS` is always added to the published rule when the list is not empty. |
| Cross-origin value | Text or absent | Absent | When set, the value of the allowed-origin response header, and preflight handling is enabled (section 7.5). |
| Cross-site token requirement | Boolean | `true` for the page transport, `false` for the remote call transports | Whether an unsafe method requires a valid cross-site request forgery token (section 7). |
| Read-only | Boolean, or a rule evaluated per request | `true` when the authentication level is `none`, otherwise `false` | Whether the request may be served on a read-only database connection (section 4.6). |
| Maximum content length | Byte count, or a rule evaluated per request | The database-wide limit (section 12) | Upper bound on the request body size. |
| Save session | Boolean | `false` when the authentication level is `bearer`, otherwise `true` | Whether the session may be persisted and the session cookie re-sent. A `false` value makes the request stateless. |
| Captcha action | Action name or absent | Absent | When set, a human-verification token is validated before the endpoint runs, for every method that is not safe. |
| Parameter access error handler | Rule or absent | Absent | Invoked when a record named in the path cannot be read; may return a response that replaces the error (section 4.8). |
| Web socket | Boolean | Absent | Marks the endpoint as answering a protocol upgrade rather than a normal request. |
| Host, subdomain, alias, strict slashes, redirect target, build only | Routing options | Absent | Forwarded to the path matcher. Trailing-slash strictness is disabled globally and slash merging is disabled per rule, therefore `/a/b` and `/a/b/` match the same endpoint but `/a//b` does not. |

Safe methods, which never require a cross-site token and never trigger captcha validation, are `GET`, `HEAD`, `OPTIONS` and `TRACE`.

### 2.2 Path converters

| Converter | Pattern | Produces |
|---|---|---|
| `<name>` | Any text without a slash | Text. |
| `<string:name>` | Same as above | Text. |
| `<int:name>` | An optionally signed sequence of digits | Integer. Negative values are accepted. |
| `<path:name>` | Any text including slashes | Text. |
| `<model("entity"):name>` | A sequence of digits | The single record of that entity with that key, resolved before the endpoint runs. |
| `<models("entity"):name>` | Digits separated by commas | The record set of that entity for those keys. |

Records produced by the two record converters are re-bound to the request environment during pre-dispatch and their read access is checked there, not inside the endpoint (section 4.8).

### 2.3 Publication of a rule

Two routing tables exist:

1. The **database-free table**, built from the endpoints whose authentication level is `none`, taken from the framework-wide packages only. It serves requests for which no database could be selected.
2. The **database table**, built per database from every installed capability package, cached, and rebuilt when the registry changes.

Both tables disable trailing-slash strictness and disable slash merging on each rule.

### 2.4 Merging of overriding declarations

A capability package may redefine an endpoint declared by another package. The merge is deterministic:

1. The layer walks the extension chain from the most general definition to the most specific override.
2. For each level, the declared attributes replace the accumulated ones; attributes not declared at that level are inherited.
3. Paths do not accumulate across levels: the last declared path list wins. An override that declares no path keeps the inherited path list.
4. The transport family may not be changed by an override. When an override declares a different family, the original family is kept and the layer logs `"The endpoint <name> changes the route type, using the original type: <type>."`.
5. The read-only attribute may not be widened. When an override declares a read/write endpoint under a read-only parent, or the reverse, the merged endpoint becomes read/write and the layer logs `"The endpoint <name> made the route <readonly|read/write> altough its parent was defined as <readonly|read/write>. Setting the route read/write."`. A rule-valued read-only attribute is exempt from this check.
6. A method that is overridden without being re-declared as an endpoint is still published, and the layer logs that the endpoint carries no declaration of its own and that a default one is applied.

### 2.5 Argument filtering

Before the endpoint runs, the collected parameters are filtered against the parameters the endpoint accepts. Parameters that the endpoint does not accept and that it does not collect through a catch-all are dropped, and the layer logs `"<name> called ignoring args <names>"`. The endpoint therefore never fails because a client sent an extra parameter on the page transport or the enveloped transport. The direct remote call transport does the opposite: it validates the argument list and rejects a mismatch with status `422` (section 3.3).

## 3. The three transport families

### 3.1 Selection and compatibility

Each endpoint declares exactly one family. On each request the layer determines which families the request is *compatible* with, from the request media type:

| Family | Accepted request media types | Compatible when |
|---|---|---|
| Page transport (`http`) | `application/x-www-form-urlencoded`, `multipart/form-data`, `*/*` | Always. |
| Enveloped remote call (`jsonrpc`) | `application/json`, `application/json-rpc` | The request media type is one of those two. |
| Direct remote call (`json2`) | `application/json` | The request media type is `application/json`, or the request has no body. |

When the matched endpoint's family is not compatible with the request, the layer answers `415` with the body `"Request inferred type is compatible with <families> but '<path>' is type='<family>'.\n\nPlease verify the Content-Type request header and try again."` and an `Accept` response header listing the media types of the endpoint's family. A cross-origin preflight request skips this check.

### 3.2 The enveloped remote call transport

**Request body** is one structured-data object:

| Member | Type | Required | Meaning |
|---|---|---|---|
| `jsonrpc` | text | no | Protocol marker, by convention `"2.0"`. It is not validated. |
| `method` | text | no | Ignored: the path already selects the operation. Clients send `"call"`. |
| `params` | object | no | The named parameters of the endpoint. It must be an object; an array is not supported. Absent means no parameter. |
| `id` | any | no | Correlation value; echoed verbatim in the response. |

The parameters given to the endpoint are the members of `params`, overlaid by the values extracted from the path pattern (path values win on a name clash).

**Successful response body**:

```
{ "jsonrpc": "2.0", "id": <the request id, or null>, "result": <the endpoint result> }
```

The `result` member is omitted when the endpoint returned nothing. The response status is `200` and the media type is `application/json; charset=utf-8`.

**Failure response body** (status is still `200`):

```
{
  "jsonrpc": "2.0",
  "id": <the request id, or null>,
  "error": {
    "code": <integer>,
    "message": <text>,
    "data": {
      "name": <text>,
      "message": <text>,
      "arguments": <array>,
      "context": <object>,
      "debug": <text>
    }
  }
}
```

| Member | Content |
|---|---|
| `error.code` | `404` when the path, the entity or the operation was not found; `100` when the session expired; `0` for every other failure. |
| `error.message` | `"404: Not Found"`, `"Session Expired"` and `"Server Error"` respectively. A replacement uses its own product-neutral wording for the last two, for example `"Session Expired"` and `"Server Error"`, and keeps the codes. |
| `data.name` | The fully qualified name of the failure class, as `<namespace>.<class>`, for example `builtins.ValueError`, `core.exceptions.AccessError`, `transport.exceptions.NotFound`. Clients branch on this value, therefore a replacement must publish a stable, documented name per failure kind. |
| `data.message` | The end-user message of the failure. |
| `data.arguments` | The failure's arguments, as an array. For a business failure this is usually a one-element array holding the message. For a transport failure it is `[message, status_code]`. |
| `data.context` | The context attached to the failure, an empty object when there is none. The redirect warning carries the follow-up action here. |
| `data.debug` | The formatted call stack of the failure, for diagnosis. It is empty for an authentication denial, whose stack is deliberately suppressed. |

The error envelope keys are always exactly those five inside `data`, and exactly `code`, `message`, `data` inside `error`.

**Body parse failures** bypass the error envelope entirely and are answered as plain text: an unparseable body gives status `400` with body `"Invalid JSON data"`, and a body that is not an object gives status `400` with body `"Invalid JSON-RPC data"`.

Worked example, reading two fields of three contacts through the generic dispatch:

```
POST /web/dataset/call_kw/contact/read
Content-Type: application/json
Cookie: session_id=<84 characters>

{"jsonrpc": "2.0", "method": "call", "id": 17,
 "params": {"model": "contact", "method": "read",
            "args": [[3, 7, 9], ["display_name", "email"]],
            "kwargs": {"context": {"lang": "en_US"}}}}
```

```
200 OK
Content-Type: application/json; charset=utf-8

{"jsonrpc": "2.0", "id": 17, "result": [
  {"id": 3, "display_name": "Deco Addict", "email": "deco@example.com"},
  {"id": 7, "display_name": "Gemini Furniture", "email": false},
  {"id": 9, "display_name": "Ready Mat", "email": "ready@example.com"}]}
```

Worked example of a business failure on the same transport:

```
200 OK
Content-Type: application/json; charset=utf-8

{"jsonrpc": "2.0", "id": 17, "error": {
  "code": 0, "message": "Server Error",
  "data": {"name": "core.exceptions.UserError",
           "message": "You cannot delete a contact that has journal entries.",
           "arguments": ["You cannot delete a contact that has journal entries."],
           "context": {}, "debug": "Traceback (most recent call last): ..."}}}
```

### 3.3 The direct remote call transport

This transport carries the payload without an envelope and reports failures with the status code, which makes it the one to use for integrations.

**Request body**: one structured-data object whose members are the named parameters. An empty body is allowed and means no parameter. The members are overlaid by the values extracted from the path (path values win). A body that is not an object is treated as no parameter at all.

**Response**: the operation result serialized directly as the body, with status `200` and media type `application/json; charset=utf-8`. When the operation returns a record set, the body is the array of its keys. When the endpoint produces a file stream, that stream is returned unchanged with its own media type and headers.

**Failure response**: the body is the *bare* failure object, with the same five members as `data` in section 3.2, and the status code carries the kind:

| Failure | Status |
|---|---|
| Business rule violation | `422` |
| Validation failure | `422` |
| Wrong or missing operation argument | `422` |
| Access right or record rule refusal | `403` |
| Authentication denial | `403` |
| Missing record | `404` |
| Unknown entity, unknown operation, unknown path | `404` |
| Record lock failure | `409` |
| Expired session | `403` |
| Missing or invalid application key | `401`, with a `WWW-Authenticate: bearer` response header |
| Unparseable body | `400` |
| Any other failure | `500` |

For a failure that originates in the transport layer rather than in the business logic, `arguments` is the pair `[message, status]`, and the response keeps the headers of the transport failure except its media type.

Worked example:

```
POST /json/2/contact/search_read
Host: books.example.com
Authorization: bearer 6578616d706c65206170706c69636174696f6e206b6579
X-Database: books
Content-Type: application/json; charset=utf-8
User-Agent: ledger-sync/2.1

{"context": {"lang": "en_US"},
 "domain": [["name", "ilike", "deco"], ["is_company", "=", true]],
 "fields": ["name"]}
```

```
200 OK
Content-Type: application/json; charset=utf-8

[{"id": 25, "name": "Deco Addict"}]
```

Worked failure example, a missing required argument:

```
422 Unprocessable Content
Content-Type: application/json; charset=utf-8

{"name": "transport.exceptions.UnprocessableEntity",
 "message": "missing a required argument: 'domain'",
 "arguments": ["missing a required argument: 'domain'", 422],
 "context": {}, "debug": "Traceback (most recent call last): ..."}
```

### 3.4 The page transport

Used for anything that a browser navigates to or downloads, and for file uploads.

**Parameters** are the merge, in this order, of: the query-string pairs, the form-body pairs (both the urlencoded and the multipart forms), the uploaded files, and finally the values extracted from the path pattern. Later sources win on a name clash. A body whose media type is structured data is *not* parsed: such a request reaches a page endpoint with the path and query-string parameters only.

**Result coercion**, applied to whatever the endpoint returned:

| Returned | Response |
|---|---|
| A response object | Used as is. |
| A framework response of the underlying server library | Adopted and completed with the default template fields. |
| Text or bytes | A `200` response with that body and media type `text/html`. |
| Nothing | A `200` response with an empty body. |
| A transport failure object | Raised instead of returned; the layer logs `"<name> returns an HTTPException instead of raising it."`. |
| Anything else | The request fails with `"<name> returns an invalid value: <value>"`. |

**Failure handling**:

| Failure | Response |
|---|---|
| Expired session | The session is logged out while keeping the database, the session is rotated when a user was connected, and the client is redirected with `303` to `/web/login` carrying `redirect=<the full path that failed>`; the new session cookie is set on that redirect. |
| Transport failure | Returned as is, with its own status and its rich-text body. |
| Business rule violation | The status declared by the failure (`403` for an access refusal or an authentication denial, `404` for a missing record, `409` for a lock failure, `422` otherwise), with the message as body. |
| Any other failure | `500` with the generic internal-error body. |

## 4. The request lifecycle

The steps below are executed in this exact order for every request.

### 4.1 Entry

1. The per-request counters are reset: query count, query time, start instant, connection mode, database name, user key, and the recorded entity-and-operation label.
2. When the server is configured to trust a reverse proxy and the request carries a forwarded-host header, the forwarded host, protocol and address replace the direct ones.
3. The request object is created and published for the duration of the request.

### 4.2 Session and database resolution

1. Read the `session_id` cookie. When it is absent or does not match the expected shape (exactly 84 characters of the base64 web-safe alphabet), create a new empty session; otherwise load the stored session and keep the presented identifier.
2. Fill every missing session key with its default (section 5.1), and set the session language from the request's accepted languages when the session has none.
3. Determine the database:
   - If the session carries a database and that database passes the exposure filter, use it. If the request *also* carries an `X-Database` header naming a different database, fail immediately with `403` and the message `"Cannot use both the session_id cookie and the x-database header."`.
   - Else, if the request carries an `X-Database` header: mark the session as not persistable (the request is stateless) and use the named database when it passes the exposure filter; when it does not, no database is selected.
   - Else, when exactly one database is exposed, use it.
   - Else, no database is selected.
4. When the resolved database differs from the session's database and the session had one, log the session out (the message logged is `"Logged into database <name>, but dbfilter rejects it; logging session out."`) and store the resolved database on the session.
5. Reset the session's dirty flag; the resolution itself never causes a session write.

The exposure filter is a deployment configuration: a pattern matched against the database name, optionally substituting the request host and its first label, or an explicit list of names. When neither is configured every database is exposed.

### 4.3 Branching

1. When the path resolves to a static asset of a capability package (`/<package>/static/<relative path>`), serve the file (section 4.10) and stop.
2. When a database was selected, serve through the registry (section 4.4).
3. Otherwise serve from the database-free routing table: match, dispatch, post-dispatch. When no rule matches, answer `404` with a short page stating that no database is selected and that the requested address was not found in the server-wide controllers.

When serving through the registry fails because the registry cannot be built or the database is unusable, the request falls back to the database-free table: the database is dropped from the request, the session is logged out, and for the client entry paths the `db` query parameter is stripped before re-routing.

### 4.4 Serving with a database

1. Build or fetch the registry for the database and open a **read-only** connection. Check the registry signal on that connection, which reloads the registry when another worker changed it.
2. Create the environment from the connection, the session user and the session context.
3. Match the path against the database routing table.
   - No match: prepare the fallback path (section 4.9) and treat the request as read-only.
   - Match: select the dispatcher for the endpoint's family (section 3.1), and evaluate the endpoint's read-only attribute, calling it with the controller, the rule and the path values when it is a rule.
4. When the request is read-only and the connection is read-only, run the request on it. When the endpoint nevertheless attempts a write, the database refuses it; the layer logs `"<message>, retrying with a read/write cursor"` and continues to step 5 with the same request.
5. Otherwise, close the read-only connection and open a read/write connection (or, when no read-only replica exists, roll back the current one to start a fresh transaction), then run the request on it.
6. Both runs go through the retry loop of section 13.
7. The connection is closed and the environment released in every case, success or failure.

### 4.5 Running a matched request

1. **Authenticate** according to the endpoint's authentication level (section 6).
2. **Pre-dispatch**, in this order:
   1. Set the maximum content length from the database-wide parameter when it is defined; an unparseable value is ignored and logged as `"invalid web.max_file_upload_size: <value>, using <limit> instead"`.
   2. Apply the family's own preparation: mark the session as not persistable when the endpoint declares that it does not save the session; set the allowed-origin and allowed-methods headers when the endpoint declares a cross-origin value; answer a preflight request immediately (section 7.5); apply the endpoint's own maximum content length, overriding the database-wide one.
   3. Store the debug flags carried by the `debug` query parameter into the session (section 11.4).
   4. Resolve the effective language: when the context language is not an installed language, fall back to the company language, then to the default language, then to the first installed language.
   5. Re-bind every record produced by a path converter to the request environment and check read access on it. On refusal: when the endpoint declares a parameter access error handler and that handler returns a response, answer with it; else, when the caller is the public user or the record does not exist, answer `404`; else propagate the access failure.
3. **Dispatch**: validate the human-verification token when the endpoint declares a captcha action and the method is not safe, then call the endpoint with the collected parameters, then force the rendering of a deferred template result.
4. **Post-dispatch**: save the session when required (section 5.5), merge the deferred headers and cookies into the response, and set the content-security headers (section 4.11).

### 4.6 Read-only requests

The read-only attribute of an endpoint is a *promise* that the request performs no write. It is used to route the request to a read-only database connection, which allows a deployment to serve reads from a replica. The promise is enforced by the database, not by the layer: a write attempted on a read-only connection fails, and the layer transparently replays the whole request on a read/write connection (section 4.4, step 4). The replay is observable only through the log line and the doubled work; the client sees a single successful response.

The default value of the attribute is `true` for endpoints whose authentication level is `none` and `false` otherwise. The generic dispatch endpoints use a rule instead: the entity and the operation named in the body are read, the operation is looked up in the entity's extension chain, and the request is read-only when the operation is marked read-only (section 8.6). When the entity is unknown, the rule answers read/write for the generic dispatch and read-only for the direct remote call (a `404` needs no write).

### 4.7 Fallback serving

When no rule matches the path and a database is selected, the layer:

1. Collects the page-transport parameters.
2. Applies the public authentication level.
3. Asks the routing service for an alternative: the built-in one serves a published attachment whose stored web address equals the request path, streaming it with the rules of section 4.10. Capability packages that publish pages extend this hook.
4. When nothing is produced, answers `404` through the failure handler of the request's family.

### 4.8 Records named in the path

A record converter resolves its value *before* authentication, using a placeholder user, and the record is re-bound to the authenticated environment during pre-dispatch, where its read access is checked. Consequences that a replacement must reproduce:

- A record that does not exist yields `404` for every caller.
- A record that exists but is not readable yields `404` for the public user and an access failure (`403`) for an authenticated user.
- An endpoint may declare a handler that turns either case into a redirect or a custom page.

### 4.9 Static assets

`/<package>/static/<relative path>` is served directly from the package's static directory. The relative path is joined safely, therefore traversal outside the directory is impossible; a missing package answers `404` with `"Module \"<package>\" not found.\n"`, and a missing or unreadable file answers `404` with `"File \"<path>\" not found in module <package>.\n"`. The response carries an entity tag built from the file's modification time, size and a checksum of its path, a last-modified validator, a maximum age of one week (or zero when the session has the asset debug flag and the caller is not the document renderer), and `X-Content-Type-Options: nosniff`.

### 4.10 File streaming

Every binary answer, whatever the endpoint, is produced by one streaming component with a uniform contract:

| Source | Behavior |
|---|---|
| A filesystem path | Streamed from disk. When the deployment delegates file sending to the front-end server and the file lies in the file store, the response carries the delegation headers and an empty body with `Content-Length: 0`. |
| In-memory bytes | Streamed from memory. |
| An external web address | A `301` redirect, or a `302` redirect carrying `Cache-Control: max-age=<seconds>` when a maximum age is set. |

Common response rules: `X-Content-Type-Options: nosniff` is always set; `Content-Security-Policy: default-src 'none'` is set unless the caller asked for none; conditional requests are honored through the entity tag and the last-modified validator, answering `304` when they match; `Content-Disposition` is set to `attachment` or `inline` with the file name encoded for the transport; when the content is public and the maximum age is positive the cache is marked public, otherwise it is marked private; an immutable answer sets a maximum age of one year plus the immutable directive.

### 4.11 Response hardening

Every response receives `X-Content-Type-Options: nosniff`. A response whose media type starts with `image/` and that has no explicit policy receives `Content-Security-Policy: default-src 'none'`. The desktop client page additionally receives `X-Frame-Options: DENY` and `Cache-Control: no-store`; the login page receives `Cache-Control: no-cache`, `X-Frame-Options: SAMEORIGIN` and `Content-Security-Policy: frame-ancestors 'self'`.

### 4.12 Headers and cookies the layer reads and writes

**Read from the request**

| Header or cookie | Used for |
|---|---|
| `Cookie: session_id` | The session identifier (section 5.2). |
| `Cookie: tz` | The browser time zone, stored on the user at first login when the user has none. |
| `Cookie: cids` | The client's selected company set. Commas are normalized to dashes before use. |
| `X-Database` | Selecting the database without a session (section 4.2). |
| `X-Skip-Session-Rotation-Interval` | Suppressing the soft rotation for one request (section 5.4). |
| `Authorization` | The application key, with the `bearer` scheme (section 6.2). |
| `Content-Type` | Choosing the transport family (section 3.1) and parsing the body. |
| `Content-Length` | Enforcing the maximum body size, and deciding that a body-less direct call carries no parameter. |
| `Accept-Language` | The default language of a session that has none. |
| `Sec-Fetch-Dest`, `Sec-Fetch-Mode`, `Sec-Fetch-Site`, `Sec-Fetch-User` | The interactive-navigation check of the application-key level (section 6.2). |
| `If-None-Match`, `If-Modified-Since` | Conditional requests on streamed content and on the discovery documents. |
| `Cache-Control` | A `no-cache` directive bypasses the server-side caches of the discovery documents. |
| `Origin` | The cross-origin decision, and the downgrade check of the notification socket. |
| `Host` | Database exposure filtering, static-file resolution and address building. |
| `User-Agent` | The device trace, the browser-specific file-name normalization, and the asset debug exception for the document renderer. |
| `Connection`, `Upgrade`, `Sec-WebSocket-Key`, `Sec-WebSocket-Version` | The notification socket handshake (section 9.11). |
| The forwarded-host, forwarded-protocol and forwarded-address headers | Only when the deployment declares that it trusts a front-end server; they then replace the direct values. |

**Written on the response**

| Header or cookie | When |
|---|---|
| `Set-Cookie: session_id` | The session is dirty or the presented identifier differs (section 5.5); always http-only, with the inactivity maximum age. |
| `Set-Cookie: cids` with a zero maximum age | On logout. |
| `Content-Type` | Always; `application/json; charset=utf-8` for both remote call transports, `text/html` by default for the page transport, the file's own type for streamed content. |
| `Content-Length` | On serialized payloads and on streamed content; forced to `0` when file sending is delegated to the front-end server. |
| `Content-Disposition` | On every download, as `attachment` or `inline` with the file name encoded for the transport. |
| `Content-Language` | On the discovery documents. |
| `Cache-Control` | `no-store` on the client page and the menu payload; `no-cache` on the login page; `public, max-age=<one year>` on translations and immutable content; `private` on non-public streamed content; `public, max-age=<one week>` on static files. |
| `ETag`, `Last-Modified` | On streamed content and on the discovery documents. |
| `X-Content-Type-Options: nosniff` | On every response. |
| `Content-Security-Policy` | `default-src 'none'` on images and on streamed content that does not opt out; `frame-ancestors 'self'` on the login page. |
| `X-Frame-Options` | `DENY` on the client page, `SAMEORIGIN` on the login page, `deny` on the documentation page. |
| `Access-Control-Allow-Origin`, `Access-Control-Allow-Methods`, `Access-Control-Max-Age`, `Access-Control-Allow-Headers` | On endpoints declaring a cross-origin value (section 7.5). |
| `WWW-Authenticate: bearer` | On a `401` from the application-key level. |
| `Accept` | On a `415` caused by a transport-family mismatch. |
| `Location` | On every redirect. |
| The file-delegation headers | When the deployment delegates file sending and the file lies in the file store. |

## 5. Sessions

### 5.1 Content

| Key | Type | Default | Meaning |
|---|---|---|---|
| `uid` | integer or absent | `null` | The authenticated user's key. `null` means not authenticated. |
| `login` | text | `null` | The authenticated user's login. |
| `db` | text | `null` | The selected database. |
| `context` | object | `{}` | The session context merged into every environment (section 11). Always carries at least a language. |
| `debug` | text | `""` | The active debug flags, comma separated (section 11.4). |
| `session_token` | text | `null` | The binding between this session and the user's credentials (section 5.3). |
| `create_time` | number | The creation instant | Used for rotation and for the deletion timer. |
| `_trace` | array | `[]` | The device trace entries (section 5.6). |
| `_trace_disable` | boolean | absent | When true, no device trace is recorded. It cannot be set by a non-administrator. |
| `pre_login`, `pre_uid` | text, integer | absent | A partial authentication awaiting a second factor (section 6.3). |
| `next_sid`, `deletion_time`, `gc_previous_sessions` | text, number, boolean | absent | Rotation bookkeeping (section 5.4). |
| `profile_session`, `profile_collectors`, `profile_params`, `profile_expiration` | mixed | absent | The performance recording state (section 11.5). |
| `is_websocket_session` | boolean | absent | Set on the first notification poll, used to detect an expired session on later polls. |
| `auth_login` | text | absent | The login to pre-fill on the login page. |

Every value written into a session is normalized through a structured-data round trip, therefore only structured-data types can be stored, and writing a value equal to the current one does not mark the session dirty.

### 5.2 Identifier and cookie

- The identifier is 84 characters of the base64 web-safe alphabet, produced from the current instant plus 64 random bytes hashed with a 512-bit digest, which yields roughly 218 bits of entropy.
- The **first 42 characters** are the *stable part*. They identify the session across rotations, they are what the cross-site token is computed from (section 7.1), and they are what a device log stores in order that a session can be revoked without storing the full identifier.
- Sessions are stored on the server, scattered over 4096 directories keyed by the first two characters of the identifier.
- The cookie is named `session_id`, is marked http-only, and its maximum age is the inactivity limit: the database parameter `sessions.max_inactivity_seconds`, defaulting to 7 days. An unparseable parameter value is ignored and logged.

### 5.3 Session token

The session token binds a session to the *credential state* of its user. It is the hexadecimal message authentication code, using a 256-bit digest, of the session identifier, keyed by the tuple of column names and values of: the database secret, and the user's key, login, password hash and active flag, in a stable order, dropping entries whose value is empty.

Consequences, all of which a replacement must reproduce:

- Changing the user's password, deactivating the user, or rotating the database secret invalidates **every** session of that user immediately.
- On every request whose session carries a user, the token is recomputed and compared in constant time. On mismatch the session is logged out while keeping the database, the environment drops to no user, and the endpoint's authentication level decides what happens next (an authenticated endpoint then reports an expired session).
- The token is recomputed and stored whenever the session is rotated.

### 5.4 Rotation

| Kind | Trigger | Effect |
|---|---|---|
| Soft | The session carries a user, more than 3 hours have elapsed since `create_time`, the path is not one of the notification paths (`/websocket/on_closed`, `/websocket/peek_notifications`, `/websocket/update_bus_presence`), and the request does not carry the header `X-Skip-Session-Rotation-Interval`. | A new identifier is produced that **keeps the first 42 characters** and replaces the remaining 42. The old session file is kept for 120 seconds, holding a pointer to the new identifier, in order that requests already in flight with the old cookie still resolve. The new session records that the old one must be collected. `create_time` is reset and the token recomputed. |
| Hard | Completing an authentication, logging out, or switching to the administrator identity. | The old session is deleted and a completely new identifier is produced. Cross-site tokens computed before the rotation become invalid. |

Collection of the previous session happens on a later request: when the session is flagged for collection and 120 seconds have elapsed since its creation, every stored session whose identifier starts with the same 42 characters is deleted, and the flag is cleared.

Because a soft rotation preserves the stable part, a page rendered before the rotation can still post its cross-site token afterwards.

### 5.5 Persistence rules

At post-dispatch:

1. When the session was marked as not persistable (a stateless request: a database named in a header, or an endpoint that declares that it does not save the session), nothing is written and no cookie is set.
2. Else, when a rotation is pending, rotate (which writes the session).
3. Else, when a soft rotation is due (section 5.4), rotate softly.
4. Else, when the session is dirty, write it.
5. When the session is dirty or the presented cookie differs from the current identifier, set the `session_id` cookie with the inactivity maximum age and the http-only flag.

Sessions older than the inactivity limit are deleted by the daily maintenance job.

### 5.6 Device trace

For every request of an authenticated session whose trace is not disabled, the layer looks for an entry matching the request's platform, browser family and network address:

- No entry: append `{platform, browser, ip_address, first_activity, last_activity}` with both instants set to now, mark the session dirty, and record a device log.
- An entry exists and its last activity is at least 3600 seconds old: update the last activity, mark the session dirty, and record a device log.
- Otherwise: do nothing.

The device log lets a user list and revoke their active sessions; revocation deletes every stored session whose identifier starts with the stored 42-character stable part. A stored identifier that does not match the 42-character shape is refused with `"Identifier format incorrect, did you pass in a string instead of a list?"`, which prevents a crafted device log from deleting sessions of another database.

## 6. Authentication

### 6.1 The four levels

| Level | Precondition | Runs as | Failure |
|---|---|---|---|
| `none` | None. The endpoint works even without a database. | No user at all; the record layer must not be used outside abstract entities. | Never fails. |
| `public` | None. | The authenticated user when the session carries one; otherwise the dedicated public user. | Never fails. |
| `user` | The session carries a user that is neither absent nor the public user. | That user. | The session-expired failure, rendered per family (section 3). |
| `bearer` | An `Authorization: bearer <key>` header carrying a valid application key, **or** an authenticated session accompanied by browser navigation headers. | The key's user, or the session's user. | `401` with a `WWW-Authenticate: bearer` header. |

Before the level is applied, and for every level, a session carrying a user is validated against its session token (section 5.3); a mismatch logs the session out first.

A cross-origin preflight request is treated as level `none` whatever the endpoint declares.

Any failure during authentication that is not already an access denial, an expired session or a transport failure is logged as `"Exception during request Authentication."` and converted into an authentication denial, in order that authentication never leaks internal details.

### 6.2 The application-key level in detail

1. Extract the key from the `Authorization` header when it matches `bearer <key>`, case-insensitively on the scheme.
2. With a key:
   1. Verify it against the key store with the scope `rpc` (section 6.4). An invalid key gives `401` with the message `"Invalid apikey"` and the `WWW-Authenticate: bearer` header.
   2. When the request *also* carries a session with a different user, fail with the authentication denial `"Session user does not match the used apikey."`.
   3. Adopt the key's user and mark the session as not persistable.
3. Without a key:
   - When the request carries no authenticated session, fail with `401` and `"User not authenticated, use an API Key with a Bearer Authorization header."`.
   - When it does carry one, require that the request be a genuine top-level browser navigation: `Sec-Fetch-Dest: document`, `Sec-Fetch-Mode: navigate`, `Sec-Fetch-Site` equal to `none` or `same-origin`, and `Sec-Fetch-User: ?1`. When any of the four does not hold, fail with `401` and `"Missing \"Authorization\" or Sec-headers for interactive usage."`. This is what replaces the cross-site token for these endpoints: a cross-site request cannot forge those four headers.
4. Finally apply the `user` level, therefore the public user is still refused.

### 6.3 Interactive login

The login flow, from the login form or from the session authentication endpoint:

1. Build the credential: `{login, password, type: "password"}`. Other credential types exist (a second-factor response, a passkey response); the type member selects the verification method.
2. Enter the login-attempt guard (section 6.5).
3. Find the single active user whose login matches, case-insensitively in the configured order. No match: authentication denial.
4. Verify the credential. The password check is refused outright when the password is empty. A successful check returns `{uid, auth_method, mfa}` where `mfa` is `skip`, `default` or `enforce`.
5. When the browser sent a time-zone cookie and the user has no time zone or has never logged in, store that time zone on the user.
6. Update the user's last-login instant.
7. Record the attempt outcome in the guard: success clears the failure counter for the source address; a denial increments it.
8. When the result asks to skip the second factor, or the user has no second factor configured, **finalize** the session; otherwise store `pre_login` and `pre_uid` on the session and leave the session unauthenticated until the second factor is validated, which finalizes it then.
9. Finalizing writes `db`, `login`, `uid`, the user's own context, and the session token; it marks the session for a hard rotation.
10. When the caller is a system user and the request carries a base location, the stored public base address is updated unless it is frozen.

Login failure messages: `"Wrong login/password"` when the denial carries no specific message, otherwise the denial's own message; `"Only employees can access this database. Please contact the administrator."` when an internal-only page refused an external user.

### 6.4 Application keys

| Aspect | Rule |
|---|---|
| Shape | 20 random bytes rendered as 40 hexadecimal characters. |
| Storage | The first 8 characters are stored in clear as a lookup index; the whole key is stored as a salted derived hash (6000 iterations of a 512-bit password-based derivation). The clear key is never stored and cannot be recovered. |
| Scope | `null` (global) or a text. A key with a `null` scope matches every scope; a scoped key matches only its own scope. The scope required by the application-key authentication level is `rpc`. |
| Expiry | Optional instant. A key without an expiry is permanent and may only be created by a system user. A non-system user must give an expiry, at most the largest duration granted by their access groups (and at least one day when none is granted), never in the past. Messages: `"The API key must have an expiration date"`, `"You cannot exceed %(duration)s days."`, `"You cannot set an expiration date in the past."`. |
| Verification | Select the rows whose index equals the key's first 8 characters, whose user is active, whose scope is `null` or equals the requested scope, and whose expiry is absent or not passed; then verify the hash. The first match yields the user's key. |
| Deletion | A user may delete their own keys; a system user may delete any. Otherwise: `"You can not remove API keys unless they're yours or you are a system user"`. Deletion clears the registry caches. |
| Collection | Expired keys are deleted by the daily maintenance job. |

Programmatic management is disabled by default. It is enabled for system users always, and for other users only when the parameter `base.enable_programmatic_api_keys` is true; otherwise `"Programmatic API keys are not enabled"`.

**Generate operation** (`generate`): inputs are an existing valid key, a scope, a name and an expiry (a date or a date and time). It enters the login-attempt guard keyed by the presented key's index, counts the caller's live keys and refuses beyond the limit of the parameter `base.programmatic_api_keys_limit` (default 10) with `"Limit of %s API keys is reached for programmatic creation"` (status `422`), verifies the presented key against the requested scope (a global key may mint any scope, a scoped key only its own), refuses a key that is invalid or belongs to another user with `"The provided API key is invalid or does not belong to the current user."`, and returns the new clear key as a text.

**Revoke operation** (`revoke`): input is a key. It enters the guard, finds a live row with that index whose hash verifies, deletes it and returns true; otherwise it fails with `"The provided API key is invalid."` (status `403`). The key being revoked need not be the key used to authenticate the call, which is what makes rotation possible: authenticate with the new key and revoke the old one.

### 6.5 Login-attempt guard

A per-worker counter keyed by the source network address:

```
on_cooldown(failures, previous_failure_instant):
    minimum := parameter base.login_cooldown_after   (default 5, 0 disables)
    if minimum = 0: return false
    delay := parameter base.login_cooldown_duration  (default 60 seconds)
    return failures ≥ minimum AND (now − previous_failure_instant) < delay
```

While on cooldown, the attempt is not even evaluated: it fails with `"Too many login failures, please wait a bit before trying again."` and is logged. A successful attempt clears the counter; a failed one increments it and records the instant. The counter is not shared between workers; it is a brute-force dampener, not a distributed lock.

### 6.6 Logging out

- Session destruction clears every session key, restores the defaults, keeps the debug flags, keeps the database only when asked, resets the context language to the request's preferred language, marks the session for a hard rotation, and lets capability packages drop their own cookies (the desktop client drops its selected-companies cookie).
- The page logout endpoint keeps the database and redirects with `303` to the given target, `/app` by default.

## 7. Cross-site request forgery rules

### 7.1 Token computation

```
expiry     := now_in_seconds + (requested_lifetime or 31 536 000)     # one year by default
message    := first_42_characters_of(session_identifier) + decimal(expiry)
signature  := hexadecimal_hmac_sha1(key = database_secret, message)
token      := signature + "o" + decimal(expiry)
```

The database secret is a stored parameter; when it is missing, token generation and validation both fail with `"CSRF protection requires a configured database secret"`. The far expiry acts as a salt against compression-based attacks even when no lifetime is requested.

### 7.2 Validation

```
validate(token):
    if token is empty: reject
    signature, _, expiry := rpartition(token, "o")
    if expiry is present:
        if expiry is not an integer: reject
        if expiry < now_in_seconds: reject
    expected := hexadecimal_hmac_sha1(database_secret, first_42_characters_of(session_identifier) + expiry)
    accept when constant_time_equal(signature, expected)
```

Because only the stable 42 characters take part, a soft rotation does not invalidate outstanding tokens; a hard rotation does.

### 7.3 When a token is required

A token is required when **all** of the following hold: the endpoint uses the page transport, the request method is not one of `GET`, `HEAD`, `OPTIONS`, `TRACE`, and the endpoint does not declare the requirement as disabled. Then:

1. When no database is selected, the request is redirected to `/web/database/selector` instead.
2. The parameter named `csrf_token` is removed from the parameters and validated.
3. On failure the request is answered with `400` and the body `"Session expired (invalid CSRF token)"`. A token that was present but invalid is logged as `"CSRF validation failed on path '<path>'"`; a missing token is logged with the long explanatory warning that names the three remedies (embed the token in the form, read it from the client bundle, or disable the requirement for a genuinely external caller and protect the endpoint otherwise).

Endpoints that disable the requirement on purpose are exactly those called by third parties that cannot hold a token: the database-management endpoints and the legacy markup remote call endpoints (section 9).

### 7.4 Why the remote call transports do not use a token

The remote call transports require the media type `application/json`, which a cross-origin form post cannot produce without a preflight, and a preflight is only answered for endpoints that explicitly declare a cross-origin value. The application-key level adds the browser-navigation header check of section 6.2 for the session-authenticated case. Together these replace the token.

### 7.5 Cross-origin requests and preflight

When an endpoint declares a cross-origin value:

1. Every response to it carries `Access-Control-Allow-Origin: <the declared value>` and `Access-Control-Allow-Methods` equal to `POST` for the enveloped remote call transport, or the endpoint's declared method list (defaulting to `GET, POST`) otherwise.
2. An `OPTIONS` request is answered immediately, before authentication runs, with `204`, `Access-Control-Max-Age: 86400` and `Access-Control-Allow-Headers: Origin, X-Requested-With, Content-Type, Accept, Authorization, Range`.

Endpoints declaring a cross-origin value of `*` in the framework packages: the translation bundle, the company logo, the notification socket and its three helper endpoints, and the socket worker bundle.

## 8. Generic dispatch of entity operations

### 8.1 The call

Two endpoints expose the whole service layer: `/web/dataset/call_kw` (and `/web/dataset/call_kw/<path>`) for an operation that returns data, and `/web/dataset/call_button` (and `/web/dataset/call_button/<path>`) for a button operation that returns an action. Both use the enveloped remote call transport and the `user` authentication level.

Parameters inside `params`:

| Name | Type | Meaning |
|---|---|---|
| `model` | text | The entity whose operation is invoked. |
| `method` | text | The operation name. |
| `args` | array | The positional arguments (section 8.3). |
| `kwargs` | object | The named arguments, including the special `context` member (section 8.4). |
| `path` | text | Optional decoration equal to `<entity>.<operation>`, used only for logging and diagnostics; it never changes what is called. When it differs from the real pair, the real pair is recorded. |

The optional trailing path segment exists only to make server logs and browser network panels readable; the entity and the operation are always taken from the body.

### 8.2 Which operations may be called

An operation is callable remotely only when **all** of the following hold:

1. Its name does not start with an underscore. Otherwise: `"Private methods (such as '<entity>.<operation>') cannot be called remotely."` (an access failure).
2. Its name is not one of the reserved language-runtime attribute names (frame, code-object, traceback and method-resolution-order attributes). Same message.
3. It exists on the entity. Otherwise: `"The method '<entity>.<operation>' does not exist"`, reported as `404` on the direct transport.
4. It is bound to a record set, not to the entity definition itself. A definition-level helper that takes no record set is refused with `"The method '<entity>.<operation>' cannot be called remotely."`.
5. No definition of that name anywhere in the entity's extension chain is marked as private. Same message as rule 4. Marking one level private makes the operation unreachable even when a later level does not repeat the mark.

The complete list of generic operations that are marked private, and therefore not reachable, is in [`service-layer.md`](service-layer.md), section 2.4.

### 8.3 Binding the record set

- When the operation is declared as entity-level (it does not act on specific records), the record set is empty and `args` is passed unchanged.
- Otherwise the **first** element of `args` is the key or the list of keys the operation acts on; it is removed from `args` and the remaining elements become the positional arguments.

Example: `write` on three records is `args = [[3, 7, 9], {"name": "New name"}]`, which binds records 3, 7 and 9 and passes one positional argument, the values mapping.

### 8.4 The context

The member named `context` is removed from `kwargs` before the call and becomes the context of the bound record set (section 11.2). It is never passed to the operation as an argument. The named arguments are copied before the extraction, in order that a retry after a serialization failure still carries the original context.

### 8.5 Adapting the result

| Operation result | Serialized as |
|---|---|
| The creation operation called with a single values mapping | The integer key of the created record. |
| The creation operation called with a list of values mappings | The array of created keys, in the order of the input list. |
| Any record set | The array of keys. |
| Anything else | Serialized as is. |

A deferred value inside the result is forced before the connection closes. An operation that returns nothing is logged as `"The method <operation> of the object <entity> cannot return \`None\`!"` and serializes as the null value; clients must not rely on it.

### 8.6 Read-only resolution

Before the transaction opens, the layer reads `model` and `method` from the body, looks the operation up along the entity's extension chain, and takes the read-only mark of the first definition that carries one. Unknown entity: the generic dispatch treats the request as read/write (and then answers `404`), the direct remote call treats it as read-only. The complete list of read-only generic operations is in [`service-layer.md`](service-layer.md), section 2.

### 8.7 The button variant

`/web/dataset/call_button` performs the same call, then post-processes the result:

1. When the result is a mapping whose `type` member is present and not the empty text, it is cleaned (section 9.7) and returned.
2. Otherwise `false` is returned.

This is how a button that merely changes state answers "nothing more to do", while a button that opens a wizard answers with the action to run.

### 8.8 The direct remote call form

`/json/2/<entity>/<operation>`, method `POST`, application-key level, stateless. The body members are the named arguments, plus:

| Member | Type | Default | Meaning |
|---|---|---|---|
| `ids` | array of integers | `[]` | The records the operation acts on. |
| `context` | object | `{}` | The context. |

Differences from the enveloped generic dispatch, all observable:

1. There are **no positional arguments**: every argument is named.
2. The entity and the operation come from the path, never from the body. A body member named `__model__` or `__method__` is ignored: the path always wins.
3. The argument list is validated against the operation's signature *before* the call; a mismatch answers `422` with the underlying message, for example `"missing a required argument: 'domain'"`.
4. Calling an entity-level operation with a non-empty `ids` answers `422` with `"cannot call <entity>.<operation> with ids"`.
5. An unknown entity answers `404` with `"the model '<entity>' does not exist"`; an unknown operation answers `404` with the not-found message of section 8.2.
6. A record-set result is serialized as the array of keys; there is no special case for the creation operation, which already returns a record set.
7. `POST /json/2` and `POST /json/2/<anything else>` answer `404` with `"Did you mean POST /json/2/<model>/<method>?"`.

### 8.9 One call, one transaction

Each generic dispatch call runs in its own transaction, committed on success and discarded on failure. Several calls cannot be chained inside one transaction. An integration that needs several operations to be atomic must call a single operation that performs them; the generic operations that already combine steps are documented as such in [`service-layer.md`](service-layer.md) (for example the combined search and read).

## 9. The client endpoints, one by one

Unless stated otherwise, the transport is the enveloped remote call, the authentication level is `user`, and the parameters listed are the members of `params`.

### 9.1 Authenticate

`/web/session/authenticate`, level `none`, read/write.

| Input | Type | Required | Meaning |
|---|---|---|---|
| `db` | text | yes | The database to authenticate against. |
| `login` | text | yes | The login. |
| `password` | text | yes | The password. |
| `base_location` | text | no | The public address the client used; when the authenticated user is a system user and the stored base address is not frozen, it is stored. |

Behavior:

1. Refuse a database that is not exposed: access failure with `"Database not found."`.
2. When the request has no database, or a different one, open a separate environment on the named database; otherwise reuse the request's.
3. Run the login flow of section 6.3.
4. When the resulting user differs from the session's user (which happens when a partial session awaits a second factor), answer `{"uid": null}` and nothing else.
5. Otherwise store the database on the session, persist the session (which performs the pending hard rotation and sets the new cookie), and answer the session information payload of section 9.2 computed for the authenticated user.

### 9.2 Session information

`/web/session/get_session_info`, read-only. No input. It first marks the session as touched, in order that its lifetime is refreshed. The answer:

| Member | Meaning |
|---|---|
| `uid` | The authenticated user's key, or null. |
| `is_system`, `is_admin`, `is_public`, `is_internal_user` | Role flags of the caller. |
| `user_context` | The user's own context (language, time zone, allowed companies). It is also written back into the session when it differs. |
| `db` | The database name. |
| `registry_hash` | A signed value derived from the registry sequence; the client uses it as the cache key for everything it caches across reloads. |
| `user_settings` | The caller's stored client preferences record. |
| `server_version`, `server_version_info` | The version as text and as a structured tuple. |
| `support_url` | The support address to show in the interface. |
| `name`, `username` | The user's display name and login. |
| `quick_login` | Whether the quick-login affordance is offered; from the parameter `web.quick_login`, default true. |
| `partner_write_date`, `partner_display_name`, `partner_id` | The caller's contact record summary. |
| `web.base.url` | The configured public base address. |
| `active_ids_limit` | The maximum number of record keys a client may carry in a context; parameter `web.active_ids_limit`, default 20000. |
| `profile_session`, `profile_collectors`, `profile_params` | The performance recording state. |
| `max_file_upload_size` | The effective upload limit in bytes; parameter `web.max_file_upload_size`, default 134 217 728 (128 mebibytes). |
| `home_action_id` | The action to open at startup, or null. |
| `currencies` | Every currency with its symbol, position and decimal places. |
| `bundle_params` | The language, plus the debug flags when set; used to build asset addresses. |
| `test_mode` | Whether the server runs with test support enabled. |
| `view_info` | The catalogue of presentation kinds the client can render. |
| `groups` | A mapping of the access groups the client must know about, currently the export permission. |
| `user_companies` | For an internal user: the selected company, the allowed companies (each with key, name, sequence, children restricted to the visible hierarchy, parent and currency), and the disallowed ancestor companies (same shape without the currency). |
| `show_effect` | For an internal user: whether celebratory effects are enabled. |

The same payload minus the company block and plus `is_frontend` and `is_website_user` is produced for public pages.

### 9.3 Session housekeeping

| Endpoint | Level | Read-only | Input | Output |
|---|---|---|---|---|
| `/web/session/check` | `user` | yes | none | Nothing. Its only purpose is to let the authentication step decide; a valid session answers `200` with a null result, an expired one answers the session-expired error envelope (code `100`). |
| `/web/session/modules` | `user` | yes | none | The array of installed capability package names. |
| `/web/session/get_lang_list` | `none` | yes | none | The array of `[code, name]` pairs of the installable languages, or `{"error": <failure>, "title": "Languages"}`. |
| `/web/session/account` | `user` | yes | none | A web address to the account portal carrying the database identity and the base address as state. |
| `/web/session/destroy` | `user` | yes | none | Logs the session out (hard rotation) and answers nothing. |
| `/web/session/logout` | `none` | yes, page transport | `redirect` (default `/app`) | Logs out keeping the database and redirects with `303`. |

### 9.4 Load menus

`/web/webclient/load_menus`, page transport, level `user`, method `GET` only, read-only.

| Input | Meaning |
|---|---|
| `lang` | Optional language; applied to the request context when given. |

The answer is a structured-data body with `Cache-Control: no-store`, mapping each menu key to an entry, plus the entry `root`:

| Member | Meaning |
|---|---|
| `id` | The menu key, or the text `root`. |
| `name` | The label. |
| `children` | The array of child menu keys, in menu order. |
| `appID` | The key of the application (top-level menu) this entry belongs to. |
| `xmlid` | The stable external name of the menu, or the empty text. |
| `actionID` | The action to run, or false. For an application entry with no action of its own, it is the action of the first descendant that has one, found depth-first through the first child. |
| `actionModel` | The action's kind. |
| `actionPath` | The action's stable path segment, or false. |
| `webIcon` | For an application: either the stored icon specification `"<icon class>,<colour>,<background colour>"`, or false. |
| `webIconData` | The icon image inlined as a data address, or the default application icon address `"/web/static/img/default_icon_app.png"`, or null. |
| `webIconDataMimetype` | The media type of the inlined icon, or false. |
| `backgroundImage` | Only on `root`, when one is configured. |

Only menus the caller may see are returned: the blacklist of menus of uninstalled features is applied first, then visibility filtering by access group and by the entity the menu leads to, then menus whose whole chain of parents is not visible are dropped. The result is cached per user, per debug flag and per language.

### 9.5 Load views and fields

`get_views` on the target entity, through the generic dispatch, read-only, entity-level.

| Input | Type | Meaning |
|---|---|---|
| `views` | array of `[view key or false, kind]` | The presentations to load. `false` means "the default presentation of that kind". |
| `options.toolbar` | boolean | Also return the actions bound to the entity. |
| `options.load_filters` | boolean | Also return the saved filters for the search presentation. |
| `options.action_id` | integer | The action whose filters are wanted. |
| `options.mobile` | boolean | Ask for the narrow-screen variants of nested presentations. |

Output:

```
{
  "views": { <kind>: { "id": <key>, "arch": <presentation description>,
                       "toolbar": {"action": [...], "print": [...]},
                       "filters": [...] } },
  "models": { <entity>: { "fields": { <field>: { <attribute>: <value> } } } }
}
```

Rules a replacement must reproduce:

- The result depends only on: the requested kinds, the caller's access rights, the presentation access rules, the options, the context language, and the per-kind presentation override keys in the context. No other context value may influence it, because the result is cached on those inputs.
- Each presentation description lists the entities and fields it uses; those are merged across the requested presentations and described once, under `models`.
- The field attributes returned are exactly: `change_default`, `context`, `currency_field`, `definition_record`, `definition_record_field`, `digits`, `min_display_digits`, `domain`, `aggregator`, `groups`, `help`, `model_field`, `name`, `readonly`, `related`, `relation`, `relation_field`, `required`, `searchable`, `selection`, `size`, `sortable`, `store`, `string`, `translate`, `trim`, `type`, `groupable`, `falsy_value_label`.
- Additional fields are forced per kind: the record key and the last-write instant for the list, card and form kinds; every field of the entity for the search kind; the numeric fields for the chart kind; the groupable fields for the cross-table kind.
- A field the caller may not read is absent from the description, and the `readonly` attribute is forced to true when the caller may read but not write it.
- `get_view` is the single-presentation variant, returning one description without the `models` block merged across kinds.

### 9.6 The data operations

The four data operations of the desktop client are entity operations reached through the generic dispatch. Their transport form is given here; their full contract, including the read specification grammar, is in [`service-layer.md`](service-layer.md).

| Operation | Positional arguments | Named arguments | Result |
|---|---|---|---|
| `web_search_read` | none (entity-level) | `domain`, `specification`, `offset` (default 0), `limit`, `order`, `count_limit` | `{"length": <integer>, "records": [<record payload>]}` |
| `web_read` | the record keys | `specification` | An array of record payloads, one per existing record, in the order of the keys. |
| `web_save` | the record keys (empty for a creation) | `vals`, `specification`, `next_id` | An array with one record payload. |
| `web_read_group` | none (entity-level) | `domain`, `groupby`, `aggregates`, `limit`, `offset`, `order`, `auto_unfold`, `opening_info`, `unfold_read_specification`, `unfold_read_default_limit`, `groupby_read_specification` | `{"groups": [<group payload>], "length": <integer>}` |
| `onchange` | the record keys (empty for a new record) | `values`, `field_names`, `fields_spec` | `{"value": {...}, "warning": {...}}` |
| `web_name_search` | none (entity-level) | `name`, `specification`, `domain`, `operator` (default `ilike`), `limit` (default 100) | An array of record payloads. |

`length` in the search result is the total count matching the filter, computed as follows: when a limit was given and exactly `limit` records came back and the optional count ceiling was not reached, a separate count is performed, bounded by the ceiling; when the context asks for a forced count, the count is always performed; otherwise the length is `offset + number of returned records`. An empty result answers `{"length": 0, "records": []}` without counting.

### 9.7 Actions

| Endpoint | Level | Read-only | Input | Output |
|---|---|---|---|---|
| `/web/action/load` | `user` | yes | `action_id` (integer key, complete external name, or stable path segment), `context` | The action's readable members, or `false` when it resolves to nothing. |
| `/web/action/run` | `user` | no | `action_id` (integer key of a server action), `context` | The follow-up action produced by the server action, cleaned, or `false`. |
| `/web/action/load_breadcrumbs` | `user` | yes | `actions`: an array of `{action, resId}` entries | An array of `{display_name}` or `{error}` entries. |

**Resolution of `action_id`** in the load endpoint:

1. An integer, or text that parses as an integer: the action with that key.
2. Text containing a dot: the record registered under that external name; it must be an action.
3. Other text: the action whose stable path segment equals it.
4. Nothing found: the business failure `"The action “%s” does not exist."`.

Then the action's kind is read, and:

- For a report action, the context flag that asks for file sizes instead of file contents is set.
- For a window action, the complete window payload is produced, which additionally expands the embedded actions.
- For any other kind, the record is read.

**Cleaning an action** keeps only the members that are safe for a client:

| Kind | Members kept |
|---|---|
| Every kind | `binding_model_id`, `binding_type`, `binding_view_types`, `display_name`, `help`, `id`, `name`, `type`, `xml_id`, `path`. |
| Window action | plus `context`, `cache`, `mobile_view_mode`, `domain`, `filter`, `group_ids`, `limit`, `res_id`, `res_model`, `search_view_id`, `target`, `view_id`, `view_mode`, `views`, `embedded_action_ids`, `close_on_report_download`. |
| Close action | plus `effect`, `infos`. |
| Web address action | plus `target`, `url`, `close`. |
| Server action | plus `group_ids`, `model_name`. |
| Client action | plus `context`, `params`, `res_model`, `tag`, `target`. |
| Report action | plus `report_name`, `report_type`, `target`, `context`, `data`, `close_on_report_download`, `domain`. |

Members that are not fields of the action kind at all are kept as custom properties, with the warning `"Action <name> contains custom properties <names>. Passing them via the \`params\` or \`context\` properties is recommended instead"`. When a window action carries no presentation list, one is generated from its presentation modes: several modes give `[[false, mode], ...]`; a single mode with an optional presentation key gives `[[key or false, mode]]`; several modes together with a presentation key is a contradiction and fails.

**Breadcrumb resolution**, per entry, in order:

1. The entry names an action: load it. A server action is executed when it has a stable path, otherwise the entry answers `{"error": "A server action must have a path to be restored"}`.
2. A client action repeated as the next entry answers `{"error": "Client actions don't have multi-record views"}`.
3. With a record key equal to `new`, the label is the translated `"New"`; with a numeric record key and an entity, the label is that record's display name; without an entity, the action's display name.
4. Without a record key, read access on the action's entity is checked and the label is the action's display name, but only when the action has at least one presentation that is neither the form nor the search kind; otherwise the label is null.
5. The entry names an entity instead of an action: the label is the record's display name; a record key is mandatory, otherwise the request fails with `400` and `"Actions with a model should also have a resId"`.
6. An entry that is neither fails with `400` and `"Actions should have either an action (id or path) or a model"`.
7. A missing action, a missing record or an access refusal answers `{"error": <message>}` for that entry only; the other entries are still resolved.

### 9.8 Export

| Endpoint | Transport | Level | Read-only | Purpose |
|---|---|---|---|---|
| `/web/export/formats` | enveloped | `user` | yes | The available export formats. |
| `/web/export/get_fields` | enveloped | `user` | yes | The exportable field tree of an entity, one level at a time. |
| `/web/export/namelist` | enveloped | `user` | yes | The resolved labels of a saved export template. |
| `/web/export/csv` | page | `user` | no | Produces the comma-separated values file. |
| `/web/export/xlsx` | page | `user` | no | Produces the spreadsheet file. |
| `/web/pivot/export_xlsx` | page | `user` | yes | Produces a spreadsheet from an already computed cross-table. |

`formats` answers `[{"tag": "xlsx", "label": "XLSX", "error": <text or null>}, {"tag": "csv", "label": "CSV"}]`, where the error member is filled when the spreadsheet writer is unavailable in the deployment.

`get_fields` inputs: `model`, `domain`, `prefix`, `parent_name`, `import_compat` (default true), `parent_field_type`, `parent_field`, `exclude`. Its output and the whole export contract are specified in [`service-layer.md`](service-layer.md), section 8.

The two file endpoints take one parameter named `data` carrying a structured-data text with the members `model`, `fields` (each `{name, label}`), `ids`, `domain`, `groupby`, `import_compat` and `context`. They answer a file with `Content-Disposition: attachment` and a name built as `"<entity description> (<entity name>)"` with the format's extension, cleaned of characters that are invalid in a file name. Any failure inside them is answered with status `500` whose body is the structured-data error object `{"code": 0, "message": "Server Error", "data": {...}}`, because the browser downloads them outside the remote call transport and cannot read an envelope.

### 9.9 Binary content

| Path family | Level | Read-only | Purpose |
|---|---|---|---|
| `/web/content`, `/web/content/<external name>`, `/web/content/<key>`, `/web/content/<entity>/<key>/<field>`, each optionally followed by `/<file name>` | `public` | yes | Download or display the content of a binary field. |
| `/web/image/...` with the same shapes plus `/<width>x<height>` and `/<key>-<unique>` | `public` | yes, stateless | The same for an image field, with resizing. |
| `/web/assets/<unique>/<file name>` | `public` | yes | A built asset bundle. |
| `/web/binary/company_logo`, `/logo`, `/logo.png` | `none` | yes | The company logo, with cross-origin `*`. |
| `/web/binary/upload_attachment` | `user` | no | Upload one or more files as attachments. |
| `/web/filestore/<path>` | `none` | yes | Always `404`; it exists to detect a front-end server that is not configured to serve the file store itself. |

**Content parameters**: `xmlid`, `model` (default `ir.attachment`), `id`, `field` (default `raw`), `filename`, `filename_field` (default `name`), `mimetype`, `unique`, `download`, `access_token`, `nocache`.

Resolution and access, in order:

1. Resolve the record: by external name when given, else by entity and key. Nothing found: `404`.
2. When an access token is given and it validates as a **field access token** for that record and field, serve with elevated rights.
3. Else, when the record itself allows returning its content (an attachment that is public, an attachment whose token matches, or a portal caller who may read the record the attachment belongs to), serve with elevated rights.
4. Else check read access normally; a refusal is `403`, a missing record `404`.
5. Any business failure during resolution is converted into `404`, in order that the endpoint never reveals whether a record exists.

A field access token is the hexadecimal signature of the record, the field and an expiry, in a named scope, with the expiry appended after the letter `o`. Its lifetime is at least 14 days and at most 42 days, deterministic per record and field within a 14-day window in order that browsers can cache, and jittered per record in order that tokens do not all expire at once. Presenting a valid token in the query string also marks the response as publicly cacheable.

Response shaping: `download=1` sets `Content-Disposition: attachment`; `unique` sets the immutable directive and a maximum age of one year; `nocache` removes the maximum age; the file name is taken from the explicit parameter, else from the named field, else built as `"<table>-<key>-<field>"`, newlines replaced by underscores, truncated to 100 characters before the extension, with the extension guessed from the media type when missing.

**Image variants**: `width` and `height` (both default 0), `crop` (default false), `quality` (default 0). When both dimensions are 0 they are guessed from the field name. The entity tag is extended with `-<width>x<height>-crop=<crop>-quality=<quality>`, therefore each variant is cached separately. The image is only re-processed when the conditional request indicates the client does not already hold it. When the field cannot be read, or is empty, a placeholder image is served instead, resized the same way, and marked private; with `download=1` a missing image answers `404` rather than a placeholder. A non-image media type is forced to the generic binary media type.

**Attachment upload**: page transport, parameters `model`, `id`, `ufile` (one or more files), `callback`. Each file becomes an attachment `{name: <file name>, raw: <bytes>, res_model: <entity>, res_id: <key>}` followed by the post-upload hook. The answer is an array with one entry per file: `{"filename", "mimetype", "id", "size"}` on success, `{"error": "You are not allowed to upload an attachment here."}` on an access refusal, `{"error": "Something horrible happened"}` on any other failure. When `callback` is given, the array is wrapped in a small script that notifies the parent frame instead, and every returned name has the character `<` removed. File names sent by one browser family are normalized to the decomposed unicode form, in order that the name the client sees matches the name it sent.

**Company logo**: with no database, the packaged logo file. With a database, the stored logo of the company named by the `company` parameter, or of the session user's company, or the packaged "no logo" image; the media type is guessed from the bytes and the file name extension follows it. Any failure falls back to the packaged logo.

### 9.10 Reports

| Endpoint | Transport | Level | Read-only | Purpose |
|---|---|---|---|---|
| `/report/<converter>/<report name>` and `/report/<converter>/<report name>/<keys>` | page | `user` | yes | Render a report. |
| `/report/download` | page | `user` | no | Render a report and force a download with the business file name. |
| `/report/barcode` and `/report/barcode/<kind>/<value>` | page | `public` | yes | Render a barcode image. |
| `/report/check_wkhtmltopdf` | enveloped | `user` | yes | The state of the document renderer. |

The converter is one of `html`, `pdf`, `text`; anything else fails with `"Converter <name> not implemented."`. The record keys are the comma-separated digits in the path; non-numeric entries are ignored. Two optional parameters carry structured-data texts: `options`, whose members are merged into the render data, and `context`, whose members are merged into the request context. Answers: the rendered markup with the default media type; the rendered document with `Content-Type: application/pdf` and its length; the rendered text with `Content-Type: text/plain` and its length.

`/report/download` takes `data`, a structured-data text holding `[<report address>, <report kind>]`, plus an optional `context` and an ignored `token`. It accepts only the kinds `qweb-pdf` and `qweb-text`; anything else answers nothing. It extracts the report name and the record keys from the address, re-enters the render endpoint, and sets `Content-Disposition` with the file name `"<report name>.<extension>"`, replaced by the report's own computed name expression when the report defines one and a single record is printed. Any failure answers `500` whose body is the escaped structured-data error object, for the same reason as the export endpoints.

Barcode parameters: `barcode_type` (one of `Codabar`, `Code11`, `Code128`, `EAN13`, `EAN8`, `Extended39`, `Extended93`, `FIM`, `I2of5`, `MSI`, `POSTNET`, `QR`, `Standard39`, `Standard93`, `UPCA`, `USPS_4State`), `value`, `width`, `height`, `humanreadable` (0 or 1), `quiet` (0 or 1, default 1), `mask`, `barLevel` (default `L`). The answer is `Content-Type: image/png` with `Cache-Control: public, max-age=31536000, immutable`. An unrenderable input fails with `"Cannot convert into barcode."`.

### 9.11 Notifications: polling and the socket

Two interchangeable mechanisms deliver server-pushed notifications. Both work on **channels**; a channel is either a text or a record reference, and is always qualified with the database name before use.

**Polling**: `/websocket/peek_notifications`, enveloped transport, level `public`, cross-origin `*`.

| Input | Meaning |
|---|---|
| `channels` | The array of channel names the client wants. Every entry must be a text, otherwise `"bus.Bus only string channels are allowed."`. |
| `last` | The highest notification key the client already has. |
| `is_first_poll` | On the first poll, marks the session as a socket session; on a later poll, a session that does not carry that mark is reported as expired. |

The server expands the requested channels with the implicit ones: the broadcast channel, every access group of the caller, and, for an authenticated caller, the caller's own contact record. It clamps `last` to zero when it is greater than the highest existing notification key (which happens after a database restore). The answer is `{"channels": [<qualified channel>], "notifications": [{"id": <integer>, "message": {"type": <text>, "payload": <any>}}]}`. Selection of notifications: when `last` is zero, every notification created in the last 50 seconds on those channels; otherwise every notification with a key greater than `last` on those channels.

**Socket**: `/websocket`, page transport, level `public`, cross-origin `*`, marked as a protocol upgrade.

Handshake requirements: the headers `connection`, `host`, `sec-websocket-key`, `sec-websocket-version`, `upgrade` and `origin` must all be present; `upgrade` must equal `websocket`; `connection` must contain `upgrade`; the version must be `13`, otherwise the answer is `426` listing the supported versions; the key must be valid base64 decoding to exactly 16 bytes, otherwise `400` with `"Sec-WebSocket-Key should be b64 encoded"` or `"Sec-WebSocket-Key should be of length 16 once decoded"`. The answer is `101` with `Upgrade: websocket`, `Connection: Upgrade` and the accept value computed as the base64 of the 160-bit digest of the key concatenated with the protocol's fixed identifier. The session is forced to persist before the upgrade, because the socket authenticates from the stored session.

Client messages are structured-data objects `{"event_name": <text>, "data": <any>}`; `event_name` is mandatory, otherwise the connection reports `"Key 'event_name' is missing from request"`; an unparseable message reports `"Invalid JSON data, <detail>"`. Each message is authenticated exactly like a request (session token check, public user otherwise) and runs in its own transaction with the retry loop of section 13. The event `subscribe` carries `{"channels": [...], "last": <integer>}` and registers the connection with the dispatcher; other events are delegated to the extensible socket handler.

Server-pushed frames carry the array of notifications, in the same shape as the polling answer. The dispatcher listens to a database-level notification channel; each committed batch of notifications wakes the connections subscribed to the affected channels, which then read and send the notifications with a key greater than the last one sent, excluding those already sent and still inside the replay window.

Close codes: `1000` clean, `1001` going away, `1002` protocol error, `1003` incorrect data, `1006` abnormal, `1007` inconsistent data, `1008` policy violation, `1009` message too big, `1010` extension negotiation failed, `1011` server error, `1012` restart, `1013` try later, `1014` bad gateway, `4001` session expired, `4002` keep-alive timeout, `4003` immediate kill. A connection from an outdated client worker is closed cleanly with the reason `OUTDATED_VERSION`, in order that the client does not reconnect in a loop.

Helpers: `/websocket/health`, level `none`, answers `{"status": "pass"}` with `Cache-Control: no-store`; `/websocket/on_closed`, enveloped, level `public`, lets a client announce a closure; `/bus/websocket_worker_bundle`, page transport, level `public`, serves the client worker; `/bus/has_missed_notifications`, enveloped, level `public`, takes `last_notification_id` and answers true when that notification no longer exists, which tells the client it missed messages and must resynchronize.

### 9.12 Client bootstrap and assets

| Endpoint | Transport | Level | Read-only | Input | Output |
|---|---|---|---|---|---|
| `/`, `/web`, `/app`, `/app/<path>`, `/scoped_app/<path>` | page | `none` | no | any | The client page, or a redirect (section 9.13). |
| `/web/login` | page | `none` | no | `redirect`, `login`, `password`, and the sign-up parameters | The login page, or a redirect after a successful post. |
| `/web/login_successful` | page | `user` | no | none | The landing page for external users. |
| `/web/become` | page | `user` | yes | none | Switches a system user to the administrator identity, recomputes the session token, clears the registry caches, and redirects. |
| `/web/webclient/translations` | page | `public` | yes, cross-origin `*` | `hash`, `mods`, `lang` | `{"lang", "hash"}` and, when the digest differs, `{"lang_parameters", "modules", "multi_lang"}`; sent with `Cache-Control: public, max-age=31536000`. |
| `/web/webclient/bootstrap_translations` | enveloped | `none` | yes | `mods` | The minimal translations needed before a session exists. |
| `/web/webclient/version_info` | enveloped | `none` | yes | none | `{"server_version", "server_version_info", "server_serie", "protocol_version": 1}`. |
| `/web/bundle/<bundle name>` | page | `public` | yes, method `GET` | bundle parameters including `lang` and `debug` | The array of `{"type": <script or style>, "src": <address>}` composing the bundle. |
| `/web/health` | page | `none` | yes, stateless | `db_server_status` | `{"status": "pass"}` with `200`, or additionally `{"db_server_status": false, "status": "fail"}` with `500` when asked to probe the database server and the probe fails. |
| `/web/version`, `/json/version` | page | `none` | yes | none | `{"version_info": [...], "version": "<text>"}`. |
| `/robots.txt` | page | `none` | no | none | `User-agent: *`, `Disallow: /`, plus one `Allow:` line per path a capability package publishes. |

The language parameters block of the translation answer carries: `name`, `code`, `direction`, `date_format`, `time_format`, `grouping`, `decimal_point`, `thousands_sep` and `week_start`. The digest is computed over the whole translation payload including those parameters and the multi-language flag, therefore any change in either invalidates the client cache.

### 9.13 The client entry page

`/web`, `/app`, `/app/<path>` and `/scoped_app/<path>`, page transport, level `none` (in order to be able to redirect when no database is selected), read/write.

1. Ensure a database: take it from the `db` parameter when it passes the exposure filter, else from the session, else the single exposed database. When a database was named in the query string and the session had none, the session records it and the client is redirected (`302`) to the same address in order to acquire the cookie. When the resolved database differs from the session's, a brand new session is created for it and the client is redirected (`302`) to the same address. When no database can be resolved, redirect (`303`) to `/web/database/selector`.
2. When the session has no user, redirect (`303`) to `/web/login` carrying `redirect=<the full path>`.
3. When a `redirect` parameter is present, honor it with `303`.
4. Validate the session token; a mismatch raises the session-expired failure, which the page transport turns into the login redirect.
5. When the user is not internal, redirect (`303`) to `/web/login_successful`.
6. Touch the session, restore the user on the environment, run the per-user bootstrap hook, build the rendering context (colour scheme and the session information payload), add to it a per-user browser cache secret derived from the same credential values as the session token, render the client page with `X-Frame-Options: DENY` and `Cache-Control: no-store`.
7. An access failure anywhere in step 6 redirects to `/web/login?error=access`.

### 9.14 Structured browsing endpoints

`/json/<path>` (level `user`) redirects with `307` to `/json/1/<path>` keeping the query string. `/json/1/<path>` uses the page transport with the application-key level and is read-only. Both are disabled unless the database carries demonstration data or the parameter `web.json.enabled` is set; otherwise `404`.

The caller must hold the export permission, otherwise the access failure `"You need export permissions to use the /json route"`.

The path is the same segment grammar the client uses in its own addresses: a sequence of action segments, each optionally followed by a record key or the word `new`. The last segment selects the action; a server action with a stable path is executed in a read-only transaction to obtain the real action, and a server action that attempts a write fails with `"Unsupported server action"`. Only window actions are supported; anything else fails with `400` and `"<kind> are not supported server-side"`.

Parameters: `view_type`, `domain`, `offset`, `limit`, `groupby`, `fields`, `start_date`, `end_date`. The endpoint **completes** the parameters and, whenever the completed set differs from the received set, answers a `307` redirect to the canonical address carrying all of them. Then:

- With a record key, or the form presentation kind: read that one record with the presentation's field specification and answer the record payload.
- With a grouping: answer the grouped read result, with the per-group filter member removed.
- Otherwise: answer the combined search and read result.
- For the calendar, timeline and cohort kinds, a date window is added to the filter: the explicit `start_date` and `end_date` when both parse as dates, otherwise the first day of the current month and the first day of the next month; the date field is the one the presentation declares, otherwise the request fails.
- For the activity kind, the filter is narrowed to records that have activities and the activity fields are added to the specification.
- The default filter is the caller's default saved filter for that action when there is one, otherwise the filters named by the context's search defaults, resolved against the search presentation.

### 9.15 Entity and field discovery

| Endpoint | Transport | Level | Input | Output |
|---|---|---|---|---|
| `/web/model/get_definitions` | page, method `POST` | `user` | `model_names`: a structured-data array of entity names | A mapping from entity name to its definition (see [`service-layer.md`](service-layer.md), section 9). |
| `/bus/get_model_definitions` | page, method `POST` | `user` | `model_names_to_fetch` | The same shape, extended with the messaging flags. |
| `/doc`, `/doc/<entity>`, `/doc/index.html` | page | `user` | none | The documentation browser page; requires the documentation access group, otherwise the access failure `"This page is only accessible to %s users."`. Sent with `X-Frame-Options: deny`. |
| `/doc/index.json`, `/doc-bearer/index.json` | direct | `user`, `bearer` | none | `{"modules": [<package names in dependency order>], "models": [{"model", "name", "fields": {<field>: {"string"}}, "methods": [<names>]}]}`. |
| `/doc/<entity>.json`, `/doc-bearer/<entity>.json` | direct | `user`, `bearer` | none | `{"model", "name", "doc", "fields": {<field>: <full description plus the introducing package>}, "methods": {<operation>: <signature description>}}`. |

The index answer is cached twice: a signed value derived from the registry sequence, the language and the caller's access groups is sent as the entity tag, and a conditional request that matches answers `304`; the generated document is also stored as an attachment named `documentation-index-<sequence>-<signature>.json` and re-served from there. A request carrying `Cache-Control: no-cache` bypasses both caches and answers with `Cache-Control: no-store` and a download disposition. Both answers carry `Content-Language`.

A signature description holds `signature`, `parameters` (each with `name`, and optionally `kind`, `default`, `annotation`, `doc`), `return`, `raise`, `doc`, `api` (which contains `model` for an entity-level operation and `readonly` for a read-only one), `model` and `module`. Only `signature`, `parameters`, `model` and `module` are guaranteed; an absent `kind` means an ordinary positional-or-named parameter. Operations that are deprecated are omitted from both documents.

### 9.16 Remaining client endpoints

| Endpoint | Transport | Level | Read-only | Input | Output |
|---|---|---|---|---|---|
| `/web/domain/validate` | enveloped | `user` | no | `model`, `domain` | True when the filter can be used to search that entity, false otherwise. An unknown entity fails with `"Invalid model: %s"`. The check builds the query and asks the database to parse it without executing it. |
| `/web/view/edit_custom` | enveloped | `user` | no | `custom_id`, `arch` | `{"result": true}`. Refuses a personalisation belonging to another user with `"Custom view %(view)s does not belong to user %(user)s"`. |
| `/web/sign/get_fonts`, `/web/sign/get_fonts/<name>` | enveloped | `none` | yes | `fontname` | The array of transport-encoded font files, sorted by file name, restricted to the four web font extensions. |
| `/web/set_profiling` | page | `public` | no | `profile`, `collectors`, and the recorder parameters | Starts or stops the performance recording for the session. |
| `/web/speedscope/<recording>`, `/web/profile_config/<recording>` | page | `user` | yes | `profile`, `action` | The recording viewer pages. |
| `/web/manifest.webmanifest`, `/web/service-worker.js`, `/app/offline`, `/scoped_app`, `/scoped_app_icon_png`, `/web/manifest.scoped_app_manifest` | page | `public` | yes for the first three | installable-application parameters | The installable-application description, its worker, its offline page and its icons. |
| `/web/partner/vcard`, `/web_enterprise/partner/<record>/vcard` | page | `user` | no | `partner_ids` or the record in the path | A contact card file. |
| `/web/tests`, `/web/tests/legacy` | page | `user` | yes | `mod` | The automated-test runner pages. |

### 9.17 Database management endpoints

These endpoints are answered by the database-free routing table, therefore they work without any database. They are all method `POST`, they all disable the cross-site token requirement, and they all require the master password, which is verified against the configured hash; a wrong password fails with an authentication denial. When database listing is disabled in the deployment, every one of them fails with an authentication denial and the log line `"Database management functions blocked, admin disabled database listing"`.

| Endpoint | Media type | Inputs |
|---|---|---|
| `/web/database/create` | form | `master_pwd`, `name`, `login`, `password`, `demo`, `lang`, `phone`, `country_code` |
| `/web/database/duplicate` | form | `master_pwd`, `name`, `new_name`, `neutralize_database` (default false) |
| `/web/database/drop` | form | `master_pwd`, `name` |
| `/web/database/backup` | form | `master_pwd`, `name`, `backup_format` (default the archive format), `filestore` |
| `/web/database/restore` | multipart | `master_pwd`, `backup_file`, `name`, `copy` (default false), `neutralize_database` (default false) |
| `/web/database/change_password` | form | `master_pwd`, `master_pwd_new` |
| `/web/database/list` | enveloped | none; answers the array of exposed database names |
| `/web/database/selector`, `/web/database/manager` | page, `GET` | none; the selection and management pages |

## 10. The alternative remote call protocols

Two additional entry points expose the same service layer with a stateless, password-based scheme. They exist for integrations written against the older conventions; new integrations use the direct remote call transport of section 3.3.

### 10.1 Shape

| Endpoint | Encoding | Level | Body |
|---|---|---|---|
| `/jsonrpc` | The enveloped remote call transport | `none`, stateless | `params` carries `service`, `method` and `args`. |
| `/xmlrpc/<service>` | Markup-encoded remote call, method `POST`, cross-site token disabled | `none`, stateless | A markup method call whose name is the method and whose parameters are the arguments. |
| `/xmlrpc/2/<service>` | Same | Same | Same. |

The only difference between the two markup endpoints is the fault reporting (section 10.5). Both markup endpoints and the enveloped one close the database connection before dispatching, because the services open their own.

`service` is one of `common`, `db`, `object`.

### 10.2 The common service

| Operation | Arguments | Result |
|---|---|---|
| `version` | none | `{"server_version": <text>, "server_version_info": <tuple>, "server_serie": <text>, "protocol_version": 1}`. |
| `login` | `db`, `login`, `password` | The user key on success, `false` on an authentication denial. Equivalent to `authenticate` with no environment description. |
| `authenticate` | `db`, `login`, `password`, `user_agent_env` | The user key on success, `false` on an authentication denial. The environment description is passed to the login flow; the flow is marked as non-interactive. |
| `about` | `extended` | A short text, or the pair of that text and the version when extended. |
| `set_loglevel` | `loglevel`, `logger` | `true`. |

The version operation is also available without any protocol wrapper at `/web/version` and `/json/version` (section 9.12).

### 10.3 The object service

| Operation | Arguments |
|---|---|
| `execute` | `db`, `uid`, `password`, `model`, `method`, then as many positional arguments as the operation takes. |
| `execute_kw` | `db`, `uid`, `password`, `model`, `method`, `args` (an array of positional arguments), `kw` (an object of named arguments, optional). |

Execution, in order:

1. Refuse an empty password outright with an authentication denial.
2. Record the database and user on the worker.
3. Fetch the registry and check its signal.
4. Open a transaction and verify the pair of user key and password; the verification is cached per pair and enters the login-attempt guard. An inactive user is refused.
5. Reset the transaction, build the environment for that user with an empty context, and refuse an unknown entity with `"Object <entity> doesn't exist"`.
6. Call the operation through the very same generic dispatch as section 8: the first positional argument is the record keys unless the operation is entity-level, the named argument `context` becomes the context, the result is adapted the same way.
7. Run inside the retry loop of section 13, commit, and signal registry changes; on failure, discard the registry changes and propagate.
8. Force deferred values before the connection closes.

`execute` cannot carry a context, because the context is taken from the named arguments and `execute` has none. Any operation callable through the generic dispatch is callable here.

### 10.4 The database service

`create_database`, `duplicate_database`, `drop`, `dump`, `restore`, `change_admin_password`, `rename`, `migrate_databases`, `db_exist`, `list`, `list_lang`, `list_countries`, `server_version`. Every one except `db_exist`, `list`, `list_lang` and `server_version` takes the master password as its first argument and is refused with an authentication denial when it is wrong. The listing operations are additionally refused, with the same denial, when database listing is disabled in the deployment. The equivalent request endpoints are in section 9.17.

### 10.5 Fault reporting

The enveloped endpoint reports failures with the error envelope of section 3.2.

The markup endpoints report a fault. The two differ:

| Endpoint | Fault code | Fault text |
|---|---|---|
| `/xmlrpc/2/<service>` | `1` for an application failure and for a business rule violation, `2` for a warning or a redirect warning, `3` for an authentication denial, `4` for an access refusal | The failure message, or the formatted call stack for an unexpected failure. |
| `/xmlrpc/<service>` | The prefixed text `"warning -- <kind>\n\n<message>"` for a warning, a missing record, an access refusal or a business rule violation; the text `"AccessDenied"` for an authentication denial; the failure message otherwise | The empty text for the prefixed kinds, the formatted call stack otherwise. |

Value encoding on the markup endpoints: byte strings are sent as text after transport decoding rather than as binary; a date and time is sent as the text `YYYY-MM-DD HH:MM:SS`; a date is sent as the text `YYYY-MM-DD`; relational commands are sent as integers; markup is sent as text; control characters below code point 32, except tab, line feed and carriage return, are removed from every text, because they are not representable.

## 11. The request context

### 11.1 What it is

The context is a mapping carried on every environment. It influences defaults, language, time zone, company scoping, and the behavior of many operations. It is never a security boundary: it cannot grant access.

### 11.2 How it is assembled, in order

1. The **session context**, which is the user's own context as computed at authentication (at least the language, the time zone and the allowed companies) plus whatever the session accumulated.
2. The **request adjustments** made during pre-dispatch: the resolved effective language always replaces the language.
3. The **call context**: for the generic dispatch, the `context` member of the named arguments **replaces** nothing and is *added on top of* the record set's context; for the direct remote call, the `context` body member is applied the same way; for other endpoints, whatever the endpoint applies explicitly.
4. Any narrowing the operation itself performs.

### 11.3 Keys the transport layer itself reads or writes

| Key | Written by | Effect |
|---|---|---|
| `lang` | The session, then pre-dispatch | The language of translations and of formatting. |
| `tz` | The user's context | The time zone used to group and format instants. |
| `allowed_company_ids` | The client, from the company switcher | The companies in scope. |
| `bin_size` | The action loader for report actions, and the save operation | Makes binary fields read as a human-readable size instead of their content. |
| `active_id`, `active_ids`, `active_model` | The client, when running an action or a button | The records the action applies to. Bounded by the active-keys limit of section 12. |
| `default_<field>` | The client, from an action's context | Seeds the default value of that field. |
| `search_default_<filter>` | The client, from an action's context | Pre-activates a named filter. |
| `force_search_count` | The client | Forces the exact count in a combined search and read. |
| `fill_temporal` | The client | Fills gaps in a grouped read over a date field. |
| `read_group_expand` | The client | Asks for empty groups to be returned. |
| `max_number_opened_groups` | The client | Overrides the limit of automatically opened groups (default 10). |
| `active_test` | Many operations | Turns the implicit "archived records are hidden" filter off. |
| `download_attachments` | The binary endpoints | Forces a missing image to fail rather than fall back to a placeholder. |
| `import_compat` | The export endpoints | Selects the re-importable export shape. |

### 11.4 Debug flags

The query parameter `debug` sets the session's debug flags. Each comma-separated entry is normalized: `assets` and `tests` are kept as they are; any other value that reads as true becomes `1`; any value that reads as false becomes the empty text. The resulting comma-separated text is stored on the session and influences asset serving (unminified bundles, no caching), the client's technical affordances and the presentation post-processing.

### 11.5 Performance recording

When the session holds a recording configuration and a database is selected, every request is recorded, except: after the configured expiry (the recording is then stopped and `"Profiling expiration reached, disabling profiling"` is logged), on the endpoint that configures recording itself, on the socket paths, and on a server running in evented mode. A failure while starting the recorder is logged as `"Failure during Profiler creation"` and disables it.

## 12. Batching and limits

| Limit | Value | Scope | Behavior at the limit |
|---|---|---|---|
| Request body size | The parameter `web.max_file_upload_size`, default 134 217 728 bytes (128 mebibytes); overridden per endpoint | Every request | `413`. |
| Session cookie lifetime | The parameter `sessions.max_inactivity_seconds`, default 604 800 seconds (7 days) | Sessions | The cookie expires; the stored session is collected. |
| Session soft rotation interval | 10 800 seconds (3 hours) | Sessions | A new identifier with the same stable part. |
| Previous-session retention | 120 seconds | Sessions | The old identifier stops resolving. |
| Active record keys in a context | The parameter `web.active_ids_limit`, default 20 000 | Client actions | The client stops passing the explicit list and passes the filter instead. |
| Export read batch | 1000 records per batch (the prefetch batch size) | Export endpoints | Records are read in batches and the cache is dropped between batches, therefore memory stays bounded whatever the export size. |
| Spreadsheet rows | The format's own maximum | Spreadsheet export | The business failure `"There are too many rows (%(count)s rows, limit: %(limit)s) to export as Excel 2007-2013 (.xlsx) format. Consider splitting the export."`. |
| Spreadsheet cell text | The format's own maximum | Spreadsheet export | The cell is replaced by `"The content of this cell is too long for an XLSX file (more than %s characters). Please use the CSV format for this export."`. |
| Cross-table export measures and widths | 100 000 | Cross-table export | Values above are clamped. |
| Automatically opened groups | 10, overridable by context | Grouped read | Further groups come back folded. |
| Records read per opened group | 80 by default, overridable per call | Grouped read | The group carries only the first records; the client pages with an offset. |
| Name search results | 100 by default | Name search | Truncated. |
| Notification payload | 8000 bytes, overridable by deployment | Notification dispatch | The batch is split recursively in halves until each payload fits; the split is logged. |
| Notification replay window | The first poll returns the last 50 seconds | Polling | Older notifications are not replayed. |
| Concurrency retries | 5 attempts | Every transaction | The failure is propagated after the fifth. |
| Programmatic application keys | The parameter `base.programmatic_api_keys_limit`, default 10 | Key generation | `422` with the limit message. |
| Application key maximum lifetime | The largest duration granted by the caller's access groups, at least one day; unlimited for a system user | Key generation | The validation messages of section 6.4. |
| Login attempts | The parameters `base.login_cooldown_after` (default 5) and `base.login_cooldown_duration` (default 60 seconds) | Login | The cooldown message. |

## 13. Error and retry semantics

### 13.1 The retry loop

Every database-backed request and every remote call runs inside this loop:

```
for attempt in 1 .. 5:
    try:
        result := run()
        flush pending writes to the database
        break
    except integrity violation, operational failure, concurrency failure:
        if the connection is already closed: propagate
        roll back the transaction
        reset the in-memory state and the pending registry changes
        reload the session from storage
        rewind every uploaded file to its start
        if the failure is an integrity violation:
            translate it to a validation failure and propagate
        if the failure is not a serialization failure, a deadlock or a lock timeout:
            propagate
        if attempt = 5:
            propagate
        sleep a random duration uniformly drawn from [0, 2^attempt) seconds
commit
signal registry changes
```

Details a replacement must reproduce:

- The three retryable database conditions are: serialization failure, deadlock detected, lock not available. Everything else is propagated on the first occurrence.
- An **integrity violation** is never retried. It is translated into the validation failure `"The operation cannot be completed: <explanation>"`, where the explanation is produced by the entity owning the violated constraint, looked up by table name.
- An uploaded file that cannot be rewound makes the retry impossible; the request then fails with `"Cannot retry request on input file '<name>' after serialization failure"`.
- The session object is re-read between attempts, because the first attempt may have modified it.
- The waiting time grows exponentially with jitter: attempt 1 waits within 2 seconds, attempt 2 within 4, attempt 3 within 8, attempt 4 within 16.
- On success the transaction is committed and the post-commit work runs; the registry change signal is emitted after the commit.
- Whatever the outcome, the in-memory state and the pending registry changes are reset when the loop exits with a failure.

Worked example: two clients confirm two different orders that both draw the next number from the same sequence. The second transaction fails with a serialization failure; the loop rolls back, waits a random duration below 2 seconds, and replays the whole request, which now reads the committed counter and succeeds. The client sees one successful response and one slightly slower successful response, never a failure.

### 13.2 Failure rendering per family

The same failure is rendered differently per transport family. This table is the authoritative mapping.

| Failure | Page transport | Enveloped remote call | Direct remote call |
|---|---|---|---|
| Business rule violation | Status `422`, message as body | `200`, envelope code `0` | `422`, bare failure object |
| Validation failure | `422` | `200`, code `0` | `422` |
| Access refusal | `403` | `200`, code `0` | `403` |
| Authentication denial | `403`, without a call stack | `200`, code `0`, empty `debug` | `403`, empty `debug` |
| Missing record | `404` | `200`, code `0` | `404` |
| Lock failure | `409` | `200`, code `0` | `409` |
| Redirect warning | `422`, with the follow-up action in the failure context | `200`, code `0`, action in `data.context` | `422`, action in `context` |
| Expired session | `303` to the login page | `200`, code `100` | `403` |
| Unknown path or entity | `404` | `200`, code `404` | `404` |
| Unparseable body | not applicable | `400`, plain text | `400`, bare failure object |
| Media type mismatch | `415` | `415` | `415` |
| Body too large | `413` | `413` | `413` |
| Anything else | `500` | `200`, code `0` | `500` |

### 13.3 Client retry guidance

- A `409` may be retried immediately once, then with a growing delay.
- A `503` or a transport-level disconnection may be retried with a growing delay; the operation may or may not have been applied, therefore only operations that are idempotent by construction may be retried blindly.
- A `422`, `403`, `404` or `400` must never be retried unchanged.
- An enveloped answer carrying code `100` means the session must be re-established before retrying.
- Because each call is its own transaction (section 8.9), a retry of a non-idempotent call can duplicate work; the way to avoid that is to call a single operation that performs the whole unit of work.

## 14. Security checks per endpoint

The table lists, for each endpoint family, every check performed before the endpoint body runs. Checks are in the order applied.

| Endpoint family | Checks |
|---|---|
| Static assets | Path containment inside the package's static directory. |
| Database-free endpoints | None beyond routing; the record layer is not available. |
| `public` endpoints | Session token validation when a session carries a user; fall back to the public user. |
| `user` endpoints | Session token validation; the user must exist and not be the public user; cross-site token on unsafe methods for the page transport. |
| `bearer` endpoints | Application key verification with scope `rpc` and expiry, or session plus the four browser-navigation headers; the session user and the key user must agree; then the `user` checks. |
| Generic dispatch | The `user` checks; the operation must be public, bound, existing and not marked private; then the entity's own access rights, record rules and field permissions inside the operation. |
| Direct generic dispatch | The `bearer` checks; the same operation checks; plus signature validation and the entity-level-with-keys refusal. |
| Binary content | Record resolution, then in order: field access token, record-specific content rule (public attachment, matching attachment token, portal caller who may read the owner record), normal read access. |
| Export files | The export permission (`base.group_allow_export`) checked inside the export operation, plus normal read access on every record and field exported. |
| Structured browsing | The export permission, the feature switch, and only window actions. |
| Report rendering | Normal read access on the records rendered, through the report's own entity. |
| Notification polling and socket | Session token validation; channels are expanded server-side, and a client may only ever ask for text channels, therefore it cannot subscribe to another user's private channel by guessing a record reference. |
| Documentation endpoints | The documentation access group, plus read access on the entity being documented. |
| Database management | The master password, and the deployment switch that enables database listing. |
| Personalisation update | Ownership of the personalisation record. |
| Attachment upload | Create access on the attachment entity, evaluated per file. |

Two rules apply everywhere and must not be weakened:

1. **The context never grants access.** Anything a client can put in the context is untrusted input.
2. **Elevated execution is explicit and narrow.** The places where the layer deliberately runs with elevated rights are: reading the display name of a related record (because visibility follows the *referring* record, not the referenced one), reading an action definition in order to clean it, reading a menu's icon attachment, streaming content that a field access token or a content rule has authorized, reading the notification rows, and computing the session token. Nothing else runs elevated on behalf of a request.

## 15. Acceptance criteria

**AC-TRANSPORT-001 — Enveloped success envelope.** Given an authenticated session, when the client posts `{"jsonrpc":"2.0","method":"call","id":42,"params":{"model":"contact","method":"search_count","args":[[]],"kwargs":{}}}` to `/web/dataset/call_kw`, then the response status is `200`, the media type is `application/json; charset=utf-8`, and the body is `{"jsonrpc":"2.0","id":42,"result":<integer>}` with no `error` member.

**AC-TRANSPORT-002 — Enveloped failure envelope shape.** Given an authenticated session, when an operation raises a business rule violation, then the status is `200` and the body has exactly the members `jsonrpc`, `id`, `error`, where `error` has exactly `code`, `message`, `data`, and `data` has exactly `name`, `message`, `arguments`, `context`, `debug`.

**AC-TRANSPORT-003 — Unknown entity through the generic dispatch.** Given an authenticated session, when the client calls the generic dispatch with the entity name `lorem.ipsum`, then the status is `200` and `error.code` is `404`.

**AC-TRANSPORT-004 — Private operation refusal.** Given an authenticated session, when the client calls the generic dispatch with an operation name starting with an underscore, then the answer is an access refusal whose message is `"Private methods (such as '<entity>.<operation>') cannot be called remotely."`.

**AC-TRANSPORT-005 — Unparseable enveloped body.** Given any caller, when the client posts the text `not json` with media type `application/json` to an enveloped endpoint, then the status is `400` and the body is the plain text `"Invalid JSON data"`.

**AC-TRANSPORT-006 — Media type mismatch.** Given a valid application key, when the client posts to `/json/2/<entity>/search` with the form media type, then the status is `415`, the response carries `Accept: application/json`, and the body explains that the inferred type is not the endpoint's type and asks to verify the media-type header.

**AC-TRANSPORT-007 — Missing application key.** Given no credentials, when the client posts `{"domain": []}` to `/json/2/<entity>/search` with media type `application/json`, then the status is `401`, the response carries `WWW-Authenticate: bearer`, and the body is `{"name":"transport.exceptions.Unauthorized","message":"User not authenticated, use an API Key with a Bearer Authorization header.","arguments":["User not authenticated, use an API Key with a Bearer Authorization header.",401],"context":{},"debug":<text>}`.

**AC-TRANSPORT-008 — Revoked application key.** Given an application key that has just been revoked, when the client uses it on the direct transport, then the status is `401` and the message is `"Invalid apikey"`.

**AC-TRANSPORT-009 — Missing argument on the direct transport.** Given a valid application key, when the client posts `{}` to `/json/2/<entity>/search`, then the status is `422` and the message is `"missing a required argument: 'domain'"`.

**AC-TRANSPORT-010 — Entity-level operation called with keys.** Given a valid application key, when the client posts `{"ids":[0]}` to `/json/2/<entity>/create`, then the status is `422` and the message is `"cannot call <entity>.create with ids"`.

**AC-TRANSPORT-011 — Path wins over body.** Given a valid application key, when the client posts `{"__model__":"<other entity>","__method__":"search","domain":[["id","=",1]]}` to `/json/2/<entity>/search`, then the operation runs on `<entity>` and never on `<other entity>`.

**AC-TRANSPORT-012 — Access refusal on the direct transport.** Given a valid application key for a user without access to an administrative entity, when the client posts `{"domain": []}` to that entity's search, then the status is `403`, the failure name is the access refusal name, and the message lists the groups that would grant access.

**AC-TRANSPORT-013 — Database header without a session.** Given a server exposing two databases and a request that carries no session cookie, when the client posts to the direct transport with `X-Database` naming an exposed database, then the request is served against that database and the response sets no session cookie.

**AC-TRANSPORT-014 — Database header naming an unknown database.** Given the same server, when the header names a database that is not exposed, then the status is `404` and the body is the database-free not-found page.

**AC-TRANSPORT-015 — Conflicting database header.** Given an authenticated session on database A, when the client sends a request carrying `X-Database: B`, then the status is `403` and the body contains `"Cannot use both the session_id cookie and the x-database header."`.

**AC-TRANSPORT-016 — Cross-site token required.** Given an authenticated session, when the client posts a form to a page endpoint that requires the token without sending `csrf_token`, then the status is `400` and the body contains `"Session expired (invalid CSRF token)"`.

**AC-TRANSPORT-017 — Cross-site token invalid.** Same as above with `csrf_token=bad token`: the status is `400` and the same message is returned.

**AC-TRANSPORT-018 — Cross-site token survives a soft rotation.** Given a page that obtained a token, when the session is soft-rotated by another request and the page then posts with the original token, then the request succeeds.

**AC-TRANSPORT-019 — Soft rotation timing.** Given an authenticated session created more than 3 hours ago, when any request is made, then the response sets a new `session_id` cookie whose first 42 characters equal the previous one's, and both stored sessions exist. When a further request is made more than 120 seconds after the rotation, then only one stored session remains.

**AC-TRANSPORT-020 — Rotation skip header.** Given the same aged session, when the request carries `X-Skip-Session-Rotation-Interval`, then the cookie is unchanged; when the next request omits the header, then the cookie changes.

**AC-TRANSPORT-021 — Credential change invalidates sessions.** Given two active sessions of one user, when that user's password is changed, then both sessions fail their next authenticated request: the page transport redirects to the login page and the enveloped transport answers code `100`.

**AC-TRANSPORT-022 — Login cooldown.** Given the parameters at their defaults, when six wrong passwords are posted from the same address within one minute, then the sixth answers `"Too many login failures, please wait a bit before trying again."` without evaluating the credential, and after 60 seconds a correct password succeeds.

**AC-TRANSPORT-023 — Second factor.** Given a user with a second factor configured, when the session authentication endpoint is called with the correct password, then the answer is `{"uid": null}`, the session is not authenticated, and the session information endpoint still reports no user.

**AC-TRANSPORT-024 — Read-only replay.** Given an endpoint declared read-only that nevertheless writes, when it is called, then the client receives the normal successful response and the write is persisted, and the server log records one read-only refusal followed by a replay.

**AC-TRANSPORT-025 — Serialization retry.** Given two concurrent calls that write the same record, when both are dispatched, then both answer `200`, the second having been replayed at most four times with an exponential random delay.

**AC-TRANSPORT-026 — Integrity violation is not retried.** Given a call that violates a uniqueness constraint, when it is dispatched, then it fails immediately with the validation failure `"The operation cannot be completed: <explanation>"`, and no retry occurs.

**AC-TRANSPORT-027 — Upload limit.** Given the parameter `web.max_file_upload_size` set to 1 048 576, when a client uploads a 2-mebibyte file, then the status is `413` and no attachment is created.

**AC-TRANSPORT-028 — Binary access token.** Given an attachment that the caller may not read and a valid field access token for it, when the client requests `/web/content/<entity>/<key>/<field>?access_token=<token>`, then the file is served and the response is marked publicly cacheable; with a token that has expired, the answer is `404`.

**AC-TRANSPORT-029 — Image placeholder.** Given a record whose image field is empty, when the client requests `/web/image/<entity>/<key>/<field>/64x64`, then a 64 by 64 placeholder image is served with a private cache directive and status `200`; with `download=1`, the answer is `404`.

**AC-TRANSPORT-030 — Image variant caching.** Given a request for `/web/image/<entity>/<key>/<field>/128x128` that returned an entity tag, when the same address is requested with that validator, then the answer is `304` and no image processing occurs; when `256x256` is requested instead, then a different entity tag is returned.

**AC-TRANSPORT-031 — Cross-origin preflight.** Given an endpoint declaring a cross-origin value, when an `OPTIONS` request is sent to it without credentials, then the status is `204`, `Access-Control-Max-Age` is `86400`, and the allowed-headers list is exactly `Origin, X-Requested-With, Content-Type, Accept, Authorization, Range`.

**AC-TRANSPORT-032 — Notification first poll.** Given a client that never polled, when it polls with `last` equal to 0 and `is_first_poll` true, then it receives the notifications of the last 50 seconds on the requested channels plus the implicit channels, and the answer lists the qualified channels.

**AC-TRANSPORT-033 — Notification session expiry.** Given a client that polls without having set the first-poll marker and whose session is not marked as a socket session, when it polls, then the answer is the expired-session envelope (code `100`).

**AC-TRANSPORT-034 — Socket version.** Given a handshake carrying `sec-websocket-version: 8`, when it is sent, then the status is `426` and the answer names the supported version `13`.

**AC-TRANSPORT-035 — Button with no action.** Given a button operation that returns nothing, when it is called through `/web/dataset/call_button`, then the result is `false`.

**AC-TRANSPORT-036 — Button returning an action.** Given a button operation that returns a window action, when it is called through `/web/dataset/call_button`, then the result contains only the members allowed for a window action, and a custom member is preserved while a non-readable field of the action is removed.

**AC-TRANSPORT-037 — Action not found.** Given a client, when `/web/action/load` is called with an unknown stable path, then the failure message is `"The action “<name>” does not exist."`.

**AC-TRANSPORT-038 — Export permission.** Given a user without the export permission, when the comma-separated values export endpoint is called, then the status is `500`, the body is the structured-data error object, and its message is `"You don't have the rights to export data. Please contact an Administrator."`.

**AC-TRANSPORT-039 — Grouped export to the flat format.** Given an export request that carries a grouping and targets the comma-separated values format, then it fails with `"Exporting grouped data to csv is not supported."`.

**AC-TRANSPORT-040 — Menu payload.** Given an internal user, when `/web/webclient/load_menus` is called, then the answer carries `Cache-Control: no-store`, contains a `root` entry whose `children` are the visible application keys, and every application entry whose own action is empty carries the action of its first descendant that has one.

**AC-TRANSPORT-041 — Session information company block.** Given an internal user belonging to two companies, when the session information is requested, then `user_companies.allowed_companies` has two entries, each carrying key, name, sequence, children, parent and currency, and `current_company` is the user's selected company.

**AC-TRANSPORT-042 — Public session information.** Given no session, when the page-level session information of a public page is produced, then `uid` is null, `is_public` is true, and no company block is present.

**AC-TRANSPORT-043 — Structured browsing redirect.** Given the feature enabled and a caller with the export permission, when `/json/1/<action path>` is requested without `offset` and `limit`, then the answer is a `307` redirect to the same path carrying the completed parameters.

**AC-TRANSPORT-044 — Structured browsing disabled.** Given the feature switch off and a database without demonstration data, when `/json/1/<anything>` is requested, then the status is `404`.

**AC-TRANSPORT-045 — Documentation cache.** Given a caller in the documentation group, when the index document is requested twice with the validator returned the first time, then the second answer is `304`; when the request carries `Cache-Control: no-cache`, then a freshly generated document is returned with `Cache-Control: no-store`.

**AC-TRANSPORT-046 — Object service password check.** Given the object service, when `execute_kw` is called with an empty password, then the call fails with an authentication denial before any entity is touched.

**AC-TRANSPORT-047 — Object service context.** Given the object service, when `execute_kw` is called with `kw` containing `context`, then the operation runs with that context; when `execute` is used, then no context can be supplied and the operation runs with an empty one.

**AC-TRANSPORT-048 — Markup fault codes.** Given the markup remote call endpoints, when an access refusal occurs, then `/xmlrpc/2/<service>` reports fault code `4` while `/xmlrpc/<service>` reports a fault whose text starts with `"warning -- AccessError"`.

**AC-TRANSPORT-049 — Creation result adaptation.** Given the generic dispatch, when `create` is called with a single values mapping, then the result is an integer; when it is called with a list of two mappings, then the result is an array of two integers in the same order.

**AC-TRANSPORT-050 — Extra parameters are ignored.** Given a page endpoint that accepts one parameter, when the client sends that parameter plus two unknown ones, then the endpoint runs normally and the unknown parameters are dropped.
