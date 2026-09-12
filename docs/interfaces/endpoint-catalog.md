# Endpoint catalog

Every request endpoint the system exposes, grouped by business domain, with its path patterns, transport, authentication
level, allowed methods, site-page flag, parameters, the operation it performs and the failures it reports. The failures
of an endpoint are stated in two places, and the [Failures](#failures) section below says how to put the two together:
a refusal the endpoint makes for a reason of its own is in its row, and the failures that follow from its transport, its
authentication level, its path converters, its cross-site setting and the records it touches are stated once for all
endpoints that share them. The catalog
lists 855 endpoints bound to 1029 path patterns. An endpoint that one capability package declares and another package
extends is listed once, in the row of the endpoint it extends, with every contributing package named in the last column
and the added behaviour described in the purpose sentence; seven rows carry such extension layers, fourteen in total.
The condensed table of the same material, sorted by path and carrying the technical handler names, is
[`../references/routes.md`](../references/routes.md); the envelope, session and
error contracts are in [`remote-transport-contracts.md`](remote-transport-contracts.md); the generic entity operations
these endpoints call are in [`service-layer.md`](service-layer.md).

## How to read a row

| Column | Meaning |
|---|---|
| Path patterns | Every address pattern bound to the same handler. A request matching any of them runs the same operation. Patterns are matched after the optional language prefix is removed on site pages. |
| Operation | This specification's name for the handler, spelled without abbreviations. It is the name other documents of this repository cite; it is not a path, it is not sent on the wire, and a rebuild may name its own handler differently. The condensed catalog carries the same endpoints keyed by path. |
| Transport | How parameters arrive and how the answer is shaped: `page or file`, `remote call` or `structured call` (see below). |
| Authentication | The identity the request must carry before the handler runs (see below). |
| Methods | The request methods accepted. An endpoint declared without a method restriction accepts both reading and submitting methods, shown as `GET, POST`. |
| Site page | Whether the endpoint runs in site context: the site record of the requested host is resolved, the language prefix of the address is honoured, the visitor record is tracked, the time zone is taken from the network position, and the answer is wrapped in the page layout of that site. |
| Parameters | The named parameters the handler declares. `accepts further named parameters` means the handler also receives every other parameter of the request, which extension layers use to add their own inputs without changing the contract. |
| Purpose and effect | What the endpoint does, in one sentence, including the records it writes, the answer it returns and any failure the endpoint refuses on for a reason of its own, with the exact message where the system shows one. The failures that follow from the transport, the authentication level, the path converters, the cross-site setting and the records the endpoint touches are the same for every endpoint that shares them and are stated once under [Failures](#failures) below. |
| Capability package | The package that declares the endpoint, followed by the packages that extend it, in installation order. |

The complete set of failures of any row is the union of the refusals its own *Purpose and effect* sentence states and
the failures the [Failures](#failures) section derives from the other columns. No row is complete on its own; none is
meant to be.

### Path pattern notation

| Notation | Meaning |
|---|---|
| `<name:integer>` | A whole number segment, converted to an integer before the handler runs. A value that is not a whole number does not match the pattern and falls through to the not-found answer. |
| `<name:text>` | A single path segment of arbitrary characters, without a separator. |
| `<name:path>` | The remainder of the address, separators included. |
| `<name:record of Entity>` | A segment that identifies one record of the named entity. The segment carries the readable title and the record identifier; the record is loaded and access-checked before the handler runs, and a record the visitor may not read gives the not-found answer, never a different one, therefore the existence of a record is not revealed. |
| `<name:record of Entity>` with a stated restriction | The same, with an additional condition the record must satisfy, for example a talk that belongs to the event of the enclosing segment; a record that fails the condition gives the not-found answer. |
| `<name>` | A free segment passed to the handler as text. |
| `<ext:one of css or js>` | A segment restricted to the two listed values. |
| A literal entity name inside a path | Three addresses carry the technical entity name of the record they serve as a fixed path segment: the author picture of a thread message, the public form that creates a quotation request, and the image fields of a point of sale configuration. The segment is part of the contract and must be served exactly as written; the entity it names is stated in the purpose of the row. |
| `<key:text of length 16>` | A segment of exactly sixteen characters. |

Trailing plus signs and literal file suffixes are part of the pattern and must be matched exactly; the tables
below show each one in full.

### Transports

| Transport | Request | Answer | Errors |
|---|---|---|---|
| `page or file` | Parameters arrive in the query string, in a form-encoded body or as uploaded file parts. | A rendered page, a file stream with a media type and a disposition, or a redirection. | A rendered error page carrying the matching status: 400 for a malformed request, 403 for a refused one, 404 for an unknown record or address, 500 for an internal failure. A request that needs a signed-in user is redirected to the sign-in form with the requested address as the return target. |
| `remote call` | The body is a call envelope whose parameter block is a document of named values; path parameters are merged into it; a display context may be supplied in the envelope and replaces the session context for the call. | The envelope carries the result document. | The envelope carries an error block with a code, a message and a data block holding the error class, the arguments and the diagnostic trace. A session that expired is reported with its own code rather than as a redirection. |
| `structured call` | The body is the parameter document itself, with no envelope. | The raw result document. | A status code that matches the failure (400, 401, 403, 404, 422, 500) with a structured body naming the error class, the message and the arguments. |

Six endpoints use the structured call transport: the five reflection endpoints and the direct entity call endpoint.
Every other endpoint uses one of the first two transports: 446 page-or-file endpoints and 403 remote-call endpoints.
412 of the 855 endpoints run in site context and 443 do not.

### Authentication levels

| Level | Meaning | Failure |
|---|---|---|
| `none` | The handler runs without any user bound to the request. Used where no installation is selected yet, or where the answer must not depend on a user: the landing page, the sign-in form, the liveness probe, the installation administration endpoints and the crawler files. | None; the handler decides. |
| `public or signed in` | The signed-in user is bound when the session carries one; otherwise the anonymous public user of the installation is bound, and the handler runs with the reading rules granted to that public identity. | None; the handler decides what an anonymous visitor may see. |
| `signed in` | A real user must be bound: a request whose session holds no user, or holds the anonymous public user, is refused. | Page transports redirect to the sign-in form with the requested address as the return target; remote calls answer with the session-expired error. |
| `application key` | The request must carry an application key in the authorization header, or be an interactive navigation from the same origin proved by the browser-supplied request metadata (destination `document`, mode `navigate`, site `none` or `same-origin`, user-activated). The key is checked against the key registry; the key user becomes the request user, and the session is not saved. | 401 with the authentication challenge and one of the messages "Invalid apikey", "User not authenticated, use an API Key with a Bearer Authorization header." or "Missing \"Authorization\" or Sec-headers for interactive usage."; a key whose user differs from the session user is refused with an access-denied error and the message "Session user does not match the used apikey." |
| `invitation token` | The request must carry the invitation token of a meeting attendee in the query string; the attendee identified by the token becomes the subject of the call, and the anonymous public identity is bound. | 400 with the message "Invalid Invitation Token." when no attendee matches, or "Invitation cannot be forwarded via email. This event/meeting belongs to \<invited address\> and you are logged in as \<signed-in address\>. Please ask organizer to add you." when another user is signed in. |
| `plug-in key` | The request must carry, in the authorization header, an application key issued for the electronic mail plug-in scope; the key user becomes the request user and the display context of that user is applied. | 400 with the message "Access token missing" or "Access token invalid". |

The levels are distributed as follows: 482 endpoints are public or signed in, 314 require a signed-in user, 39 require no
identity, 11 use the plug-in key, 5 use the invitation token and 4 use an application key.

### Cross-site submission protection

On the page-or-file transport the anti-forgery token of a submitted form is verified on every method that is not a
reading method, unless the endpoint switches the check off. Sixty-five page-or-file endpoints switch it off, and they
are exactly the endpoints whose caller is not a browser of the current session: every payment provider notification
address and return address, the document exchange network callbacks, the tax identification verification callback, the
text message
delivery reports, the fulfilment notification of the print-on-demand partner, the mobile and terminal payment callbacks,
the inbound automation hook, the mailing list confirmation links, the periodic activity summary links, the module import
endpoint, and the installation administration endpoints, which run before any session exists.

The two remote call transports do not use the token at all, because their media type cannot be produced by a cross-site
form submission and a preflight is answered only for the endpoints that declare a cross-origin value. Two remote call
endpoints nevertheless switch the check on explicitly, because a site page submits them as an ordinary form: the mailing
feedback endpoint and the mailing list subscription update endpoint. One remote call endpoint, the terminal payment
notification of one point of sale provider, switches it off explicitly.

An endpoint that receives a browser back from an external
site additionally does not open a session on that request, because a cookie set on a cross-site submission is rejected by
the browser and a new empty session would replace the one the visitor already has.

### Access to a record without a session

Three token mechanisms grant access to one record without signing in, and they appear throughout the catalog:

1. **Document access token.** A random token stored on the document (order, invoice, task, transfer, subscription). A
   visitor holding it may read the document, its attachments and its discussion thread, and may perform the actions the
   document offers to its customer. It is the token embedded in every share link and in every message link.
2. **Signed link token.** A token computed over the record, the action and a scope, verified by recomputation. It carries
   no state, and it authorizes exactly one action on one record: approve a time off request, unsubscribe from the
   notifications of a journal, unsubscribe from a mailing, confirm a mailing list subscription.
3. **Guest identity.** A record that represents an unidentified visitor of a conversation, addressed by a guest token.
   It is the identity under which a visitor posts in a live conversation, joins a shared channel or opens a meeting room.

### Failures

Every endpoint of this catalogue reports failures, and the failures of an endpoint come from two places. A reader
assembles the complete set of an endpoint from both.

**Failures the row states itself.** Where an endpoint refuses on a condition of its own — a code that is not valid, a
fingerprint that does not match, a company without registered credentials, an amount below a stated minimum, a document
in a state that forbids the action — the refusal, and the exact message where the system shows one, is part of the
sentence in the *Purpose and effect* column of that row. Every endpoint that can fail for a reason no other endpoint
shares carries its refusal there; the endpoints that can only fail for the reasons of the table below carry none, and
that is what the absence of a refusal in a row means.

**Failures every endpoint of the same shape reports.** All the other failures follow from five properties that the row
already states — its path patterns, its methods, its transport, its authentication level and its site-page flag — plus
the records the endpoint touches. They are the same for every endpoint that shares those properties, so they are stated
once, here, rather than 855 times. An endpoint reports every failure in the table below whose condition its row
satisfies, and reports nothing beyond that and its own refusals.

| The row says | The condition that fails | What the caller receives |
|---|---|---|
| A path pattern contains `<name:integer>` | The segment is not a whole number | No pattern matches; the caller receives the not-found answer of its transport. |
| A path pattern contains `<name:record of Entity>` | No record of the entity has that identifier, or the caller may not read it, or it fails the restriction stated for that segment | The not-found answer, never a refusal, so that the existence of a record is not revealed. |
| A path pattern contains `<name:text>`, `<name:path>`, `<key:text of length 16>` or `<ext:one of css or js>` | The segment does not have the stated shape | No pattern matches; the not-found answer. |
| Methods | The request method is outside the listed set | Status `405`. |
| Transport | The body's media type does not belong to the transport family of the endpoint | Status `415`, with an `Accept` response header naming the media types the family accepts, and a body that names the families the request is compatible with and asks the caller to check the media-type header. |
| Transport | The body is larger than the effective maximum content length | Status `413`. |
| Transport `remote call` or `structured call` | The body is not a well-formed parameter document | Status `400` with the body `"Invalid JSON data"`; an envelope that parses but lacks its required members gives `400` with the body `"Invalid JSON-RPC data"`. |
| Authentication | The identity required by the level is absent or refused | The failure column of the level in the *Authentication levels* table above, which states the status and the exact message for each of the six levels. |
| Site page — yes | The address matches no page and no endpoint of the resolved site | The not-found answer rendered inside the page layout of that site rather than as the bare framework error page. A caller who holds the site-designer group is instead shown the site's own not-found page, which offers to create a page at that address; a page protected by a visibility password answers `403` with the site's password page. |
| Transport `page or file`, a method that is not a reading method, and the endpoint does not switch the check off | The anti-forgery token is absent, malformed, expired or does not recompute | Status `400` with the body `"Session expired (invalid CSRF token)"`. When no database is selected the request is redirected to the database selector instead. |
| The purpose names the document access token | The token is absent or does not equal the token stored on the document, and the caller has no reading right of its own | A page endpoint of the portal redirects the caller to `/my`; a remote-call endpoint of the portal answers with an error member, for example `"Invalid order."`. A document that does not exist at all gives the missing-record failure with the message `"This document does not exist."`. |
| The purpose names a signed link token | The signature does not recompute over the record, the action and the scope | No message is shown. A signed-in caller is redirected to the discussion screen; a caller with no session is redirected to the sign-in form carrying a link back to the record view. |
| The purpose names a guest identity | The guest token is absent or matches no guest record | The endpoint answers as it would to a visitor with no access to that conversation: the not-found answer for a page, an access refusal for a call. |
| Any endpoint that reads or writes records | Access rights, record rules or field permissions refuse the operation | The access refusal. |
| Any endpoint that invokes a business operation | A guard of that operation refuses | The business rule violation carrying that operation's own message; the guards and their messages are specified with the operation in [`service-layer.md`](service-layer.md) and in the domain folder named in the section heading. |
| Any endpoint that exports records | The caller does not hold the export permission and is not an administrator | The message `"You don't have the rights to export data. Please contact an Administrator."`; the structured browsing endpoints under `/json/2` answer instead with `"You need export permissions to use the /json route"`. |
| Any endpoint at all | The request raises a failure the handler does not catch | Status `500`. |
| Any endpoint at all | The database reports a serialization failure, a deadlock or a lock timeout | The whole request is rolled back and replayed, up to five attempts, before any failure is reported; a request carrying an uploaded file that cannot be rewound cannot be replayed and fails with `"Cannot retry request on input file '<name>' after serialization failure"`. |

**How a failure is rendered.** The mapping from a failure class to a status, an envelope code and a body is the same
for the whole platform and is tabulated once, per transport family, in
[`remote-transport-contracts.md`](remote-transport-contracts.md), section 13.2, which is the authoritative table; the
retry loop that precedes every reported failure is section 13.1 of the same document and the guidance on which failures
a client may retry unchanged is section 13.3. In short: on the `page or file` transport a failure becomes a rendered
error page carrying the matching status; on the `remote call` transport it becomes a `200` answer whose envelope carries
an error block, except that an expired session carries envelope code `100` and an unknown path or entity carries
envelope code `404`; on the `structured call` transport it becomes the matching status with a structured body naming the
error class, the message and the arguments. The three *Errors* cells of the *Transports* table above say the same thing
in one line each.

**A row that names no failure of its own.** Such a row has exactly the failure set that the table above gives it, and
nothing further. That is a statement about the endpoint, not a gap in the row.

## Endpoints by domain

Each section below groups the endpoints of one part of the system. The last column names the folder of this repository
that specifies the behaviour the endpoints invoke.

| Domain section | Endpoints | Groups | Behaviour specified in |
|---|---|---|---|
| Platform Foundation | 78 | 10 | [platform runtime](../runtime/request-lifecycle.md), [architecture](../overview/architecture.md) |
| Identity, Authentication and Access Control | 11 | 2 | [identity-and-access](../domains/identity-and-access/) |
| Customer Portal | 18 | 3 | [sales](../domains/sales/), [general-ledger](../domains/general-ledger/), [projects-and-tasks](../domains/projects-and-tasks/) |
| General Ledger and Invoicing Documents | 12 | 4 | [general-ledger](../domains/general-ledger/) |
| Accounts Receivable Payment Actions | 3 | 1 | [accounts-receivable](../domains/accounts-receivable/) |
| Electronic Document Interchange | 5 | 1 | [electronic-invoicing-and-document-exchange](../domains/electronic-invoicing-and-document-exchange/) |
| Fiscal Localizations | 15 | 2 | [fiscal localizations](../domains/README.md) |
| Calendar and Scheduling | 12 | 2 | [calendar and scheduling](../domains/README.md) |
| Attendance Recording | 13 | 2 | [attendances-and-working-time](../domains/attendances-and-working-time/) |
| Employee Services | 9 | 2 | [lunch-ordering](../domains/lunch-ordering/), [human-resources-core](../domains/human-resources-core/) |
| Time Off Approvals | 5 | 1 | [time-off](../domains/time-off/) |
| Employee Directory | 4 | 1 | [human-resources-core](../domains/human-resources-core/) |
| Online Job Positions | 6 | 1 | [recruitment](../domains/recruitment/) |
| Purchasing Portal | 5 | 1 | [purchasing](../domains/purchasing/) |
| Timesheet Portal | 2 | 1 | [timesheets](../domains/timesheets/) |
| Projects and Tasks | 12 | 3 | [projects-and-tasks](../domains/projects-and-tasks/) |
| Manufacturing Portal | 4 | 2 | [manufacturing](../domains/manufacturing/) |
| Inventory Operations | 3 | 1 | [inventory-operations](../domains/inventory-operations/) |
| Products and Catalogue | 4 | 1 | [products-and-catalog](../domains/products-and-catalog/) |
| Contacts and Organizations | 1 | 1 | [contacts-and-organizations](../domains/contacts-and-organizations/) |
| Spreadsheets and Dashboards | 6 | 1 | [spreadsheets-and-dashboards](../domains/spreadsheets-and-dashboards/) |
| Loyalty and Promotions | 5 | 1 | [loyalty-and-promotions](../domains/loyalty-and-promotions/) |
| Sales Quotations and Orders | 19 | 3 | [sales](../domains/sales/) |
| Customer Relationship Management | 9 | 3 | [customer-relationship-management](../domains/customer-relationship-management/) |
| Payment Providers | 62 | 2 | [payment providers](../domains/README.md) |
| Messaging and Collaboration | 141 | 12 | [messaging-and-activities](../domains/messaging-and-activities/) |
| Surveys, Certifications, Courses and Community Forum | 104 | 3 | [learning-surveys-and-gamification](../domains/learning-surveys-and-gamification/) |
| Site and Content Management | 90 | 9 | [website-and-storefront](../domains/website-and-storefront/) |
| Commerce Storefront | 66 | 4 | [website-and-storefront](../domains/website-and-storefront/) |
| Point of Sale | 41 | 3 | [point-of-sale](../domains/point-of-sale/) |
| Events | 38 | 3 | [events](../domains/events/) |
| Marketing Campaigns | 31 | 3 | [marketing and mass mailing](../domains/README.md) |
| Automation and Integration Services | 21 | 5 | [automation-and-integration](../domains/automation-and-integration/) |

## Platform Foundation

The platform foundation exposes the entry points of the desktop client, the session and authentication calls, the generic
record services the client uses to read and write any entity, the file and image delivery endpoints, the export services,
the report rendering endpoints, the notification bus, the installation administration endpoints and the diagnostic pages.
Every other domain builds on this group: a storefront page, a portal page and a point of sale screen all reload their data
through the generic record services listed here.

### Entry points and client bootstrap

These endpoints serve the shell of the desktop client and the resources it needs before any record is read. The landing
endpoint decides where a visitor goes: an internal user reaches the desktop client, an external (portal) user reaches the
external landing page, an anonymous visitor reaches the sign-in form.

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/` | `index` | page or file | none | GET, POST | no | `s_action`, `database` (accepts further named parameters) | Landing entry point: redirects a signed-in external user to the external landing page and every other visitor to the desktop client shell, forwarding all received query parameters. | Web + Customer Portal + Website |
| `/web` <br> `/app` <br> `/app/<subpath:path>` <br> `/scoped_app/<subpath:path>` | `web_client` | page or file | none | GET, POST | no | `s_action` (accepts further named parameters) | Serves the desktop client shell for a signed-in internal user, refreshes the session lifetime, and redirects to the sign-in form (with the requested address as the return target) when no valid session exists or to the external landing page when the signed-in user is external. | Web |
| `/web/bundle/<bundle_name:text>` | `bundle` | page or file | public or signed in | GET | no | `bundle_name` (accepts further named parameters) | Returns the definition of a named resource bundle (its script and style sheet members) as used to build the client shell. | Web + Website |
| `/web/assets/<unique:text>/<filename:text>` | `content_assets` | page or file | public or signed in | GET, POST | no | `filename`, `unique`, `nocache`, `assets_parameters` | Streams one generated resource bundle file; the version segment in the address makes the response immutable and cacheable for one year. | Web |
| `/web/webclient/load_menus` | `web_load_menus` | page or file | signed in | GET | no | `language` | Returns the complete menu tree the signed-in user may see, in the requested language, with the application icons inlined; the response is marked as not storable in any cache. | Web + Auth Timeout |
| `/web/webclient/translations` | `translations` | page or file | public or signed in | GET, POST | no | `hash`, `mods`, `language` | Returns the user-facing translated texts for the requested capability packages and language, and returns nothing when the supplied fingerprint of the translation set matches the current one. | Web |
| `/web/webclient/bootstrap_translations` | `bootstrap_translations` | remote call | none | POST | no | `mods` | Returns the minimal translated texts needed to render the sign-in form and the installation administration pages before a session exists, chosen from the language declared by the browser. | Web |
| `/web/webclient/version_info` | `version_info` | remote call | none | POST | no | none | Returns the platform version descriptor of the installation. | Web |
| `/web/session/get_lang_list` | `get_language_list` | remote call | none | POST | no | none | Returns the list of languages that can be selected on the sign-in and installation administration pages, as code and name pairs. | Web |
| `/web/login_successful` | `login_successful_external_user` | page or file | signed in | GET, POST | yes | accepts further named parameters | Renders the landing page shown to an external user right after a successful sign-in when no customer portal is installed. | Web |
| `/robots.txt` | `robots` | page or file | none | GET, POST | no | accepts further named parameters | Returns the crawler instruction file: crawling is denied for the whole address space except the explicitly allowed paths contributed by the installed capability packages. | Web + Website |

### Session and authentication calls

Session calls create, inspect and end the session that carries the user identity, the active company selection and the
display context. The detailed envelope, cookie and expiry semantics are in
[`remote-transport-contracts.md`](remote-transport-contracts.md).

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/web/session/authenticate` | `authenticate` | remote call | none | POST | no | `database`, `login_name`, `password`, `base_location` | Authenticates a login name and password against the named installation, opens a session, and returns the session descriptor; used by non-browser clients. | Web |
| `/web/session/get_session_info` | `get_session_info` | remote call | signed in | POST | no | none | Returns the descriptor of the current session: user identity, allowed companies and the active company selection, user groups, display context, server version and installation identifier. | Web |
| `/web/session/check` | `check` | remote call | signed in | POST | no | none | Confirms that the current session is still valid; returns a session-expired error otherwise. | Web |
| `/web/session/destroy` | `destroy` | remote call | signed in | POST | no | none | Ends the current session and clears its stored identity. | Web |
| `/web/session/logout` | `logout` | page or file | none | GET, POST | no | `redirect` | Ends the current session while keeping the installation selection, then redirects to the address given as the return target, defaulting to the desktop client shell. | Web + Web Routing + Website |
| `/web/session/modules` | `modules` | remote call | signed in | POST | no | none | Returns the list of installed capability packages that contribute client-side resources. | Web |
| `/web/session/account` | `account` | remote call | signed in | POST | no | none | Returns the address of the publisher account service sign-in flow, built from the installation identifier and the public base address, for opening account management in a new window. | Web |
| `/web/login` | `web_login` | page or file | none | GET, POST | no | `redirect` (accepts further named parameters) | Serves the sign-in form and processes its submission: it validates the credentials, applies the anti-robot verification when the failure history requires it, opens a session, and redirects to the requested return target or to the default landing page of the user. | Web + OAuth2 Authentication + Signup + Website |
| `/web/become` | `switch_to_admin` | page or file | signed in | GET, POST | no | none | Switches the session of a member of the system administration group to the technical superuser identity, recomputes the session token and redirects to the landing page. | Web |

### Generic record and action services

These are the services the desktop client uses for every entity: they resolve actions, run generic and business operations
on records, validate filter conditions and describe entities. The call and result structures are specified in
[`service-layer.md`](service-layer.md).

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/web/action/load` | `load` | remote call | signed in | POST | no | `action`, `context` | Resolves an action by its identifier or external identifier and returns its complete definition, including the views to display and the stored context. | Web |
| `/web/action/run` | `run` | remote call | signed in | POST | no | `action`, `context` | Runs a server-side action of the action registry with the supplied context and returns either the resulting action to display or a plain result. | Web |
| `/web/action/load_breadcrumbs` | `load_breadcrumbs` | remote call | signed in | POST | no | `actions` | Returns the display names for a list of action and record pairs, used to rebuild the navigation trail after a page reload. | Web |
| `/web/dataset/call_kw` <br> `/web/dataset/call_kw/<path:path>` | `call_kw` | remote call | signed in | POST | no | `model_name`, `method`, `args`, `kwargs`, `path` | Invokes any allowed operation of any entity with positional and named arguments and returns its result; this is the general-purpose record service of the client. | Web |
| `/web/dataset/call_button` <br> `/web/dataset/call_button/<path:path>` | `call_button` | remote call | signed in | POST | no | `model_name`, `method`, `args`, `kwargs`, `path` | Invokes a form button operation on selected records and returns either the action the operation produced or the value false when it produced none; the transaction is committed before the result is serialized. | Web |
| `/web/domain/validate` | `validate` | remote call | signed in | POST | no | `model_name`, `domain` | Checks that a filter condition can be evaluated against an entity and returns true or false; raises a validation error when the entity name is unknown. | Web |
| `/web/model/get_definitions` | `get_model_definitions` | page or file | signed in | POST | no | `model_names` (accepts further named parameters) | Returns the field definitions, relations and access flags of the requested entities for the client-side record cache. | Web |
| `/web/view/edit_custom` | `edit_custom` | remote call | signed in | POST | no | `custom`, `arch` | Stores the personal layout adjustment a user made on a view (the retained arrangement of the list columns) and acknowledges the change. | Web |
| `/json/<subpath:path>` | `web_structured_data` | page or file | signed in | GET, POST | no | `subpath` (accepts further named parameters) | Returns, as a structured document, the same records the desktop client would display for the given navigation path, for a signed-in user. | Web |
| `/json/1/<subpath:path>` | `web_structured_data_1` | page or file | application key | GET, POST | no | `subpath` (accepts further named parameters) | Returns, as a structured document authenticated by an application key, the records of the navigation path: a single record read for a record path, a grouped read when a grouping is requested, a search read otherwise; incomplete paths are answered with a redirect to the canonical path. | Web |

### Files, images and attachments

File endpoints resolve a stored value (an attachment, a binary field, an image field) and stream it with the correct media
type, caching headers and disposition. Access is checked with the reading rules of the owning record, or with an access
token when one is supplied.

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/web/content` <br> `/web/content/<xmlid:text>` <br> `/web/content/<xmlid:text>/<filename:text>` <br> `/web/content/<id:integer>` <br> `/web/content/<id:integer>/<filename:text>` <br> `/web/content/<model:text>/<id:integer>/<field:text>` <br> `/web/content/<model:text>/<id:integer>/<field:text>/<filename:text>` | `content_common` | page or file | public or signed in | GET, POST | no | `external_identifier`, `model_name`, `identifier`, `field`, `filename`, `filename_field`, `mimetype`, `unique`, `download`, `access_token`, `nocache` | Streams the binary content of an attachment or of a binary field of any record, addressed by attachment identifier, by external identifier or by entity, record and field, optionally as a download with a chosen file name and media type, optionally authorized by an access token. | Web |
| `/web/image` <br> `/web/image/<xmlid:text>` <br> `/web/image/<xmlid:text>/<filename:text>` <br> `/web/image/<xmlid:text>/<width:integer>x<height:integer>` <br> `/web/image/<xmlid:text>/<width:integer>x<height:integer>/<filename:text>` <br> `/web/image/<model:text>/<id:integer>/<field:text>` <br> `/web/image/<model:text>/<id:integer>/<field:text>/<filename:text>` <br> `/web/image/<model:text>/<id:integer>/<field:text>/<width:integer>x<height:integer>` <br> `/web/image/<model:text>/<id:integer>/<field:text>/<width:integer>x<height:integer>/<filename:text>` <br> `/web/image/<id:integer>` <br> `/web/image/<id:integer>/<filename:text>` <br> `/web/image/<id:integer>/<width:integer>x<height:integer>` <br> `/web/image/<id:integer>/<width:integer>x<height:integer>/<filename:text>` <br> `/web/image/<id:integer>-<unique:text>` <br> `/web/image/<id:integer>-<unique:text>/<filename:text>` <br> `/web/image/<id:integer>-<unique:text>/<width:integer>x<height:integer>` <br> `/web/image/<id:integer>-<unique:text>/<width:integer>x<height:integer>/<filename:text>` | `content_image` | page or file | public or signed in | GET, POST | no | `external_identifier`, `model_name`, `identifier`, `field`, `filename_field`, `filename`, `mimetype`, `unique`, `download`, `width`, `height`, `crop`, `access_token`, `nocache` | Streams an image field of any record with the same addressing options as the content endpoint, additionally resizing to the requested width and height, optionally cropping, and falling back to a placeholder image when the field is empty. | Web |
| `/web/binary/company_logo` <br> `/logo` <br> `/logo.png` | `company_logo` | page or file | none | GET, POST | no | `dbname` (accepts further named parameters) | Streams the logo of the company of the installation, or the default logo when none is set, without requiring a session. | Web |
| `/web/binary/upload_attachment` | `upload_attachment` | page or file | signed in | GET, POST | no | `model_name`, `identifier`, `ufile`, `callback` | Accepts one or more uploaded files, stores each as an attachment linked to the given entity and record, and returns the created attachment descriptors to the calling window. | Web |
| `/web/filestore/<_path:path>` | `content_filestore` | page or file | none | GET, POST | no | `path` | Refuses direct access to the stored file area and answers with the not-found status; it exists to make sure no stored file is ever served by path. | Web |
| `/web/sign/get_fonts` <br> `/web/sign/get_fonts/<fontname:text>` | `get_fonts` | remote call | none | POST | no | `fontname` | Returns the handwriting fonts offered when a signature is typed rather than drawn, as encoded font files, either all of them or the one named in the path. | Web |
| `/web_enterprise/partner/<partner:record of Contact>/vcard` <br> `/web/partner/vcard` | `download_vcard` | page or file | signed in | GET, POST | no | `partner`, `partner` (accepts further named parameters) | Returns the electronic business card of a contact as a downloadable contact card file. | Web |

### Export services

Export turns the current list selection, or a pivot table, into a downloadable file. The field selection and template
mechanics are described in [`report-and-export-documents.md`](report-and-export-documents.md).

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/web/export/formats` | `formats` | remote call | signed in | POST | no | none | Returns the available export file formats as identifier and display name pairs. | Web |
| `/web/export/get_fields` | `get_fields` | remote call | signed in | POST | no | `model_name`, `domain`, `prefix`, `parent_name`, `import_compat`, `parent_field_type`, `parent_field`, `exclude` | Returns the exportable fields of an entity at the requested nesting level, therefore a user can expand a relation and pick its fields; the import-compatible mode restricts the list to fields that can be imported back. | Web |
| `/web/export/namelist` | `namelist` | remote call | signed in | POST | no | `model_name`, `export` | Returns the display labels of the fields of a saved export template, in template order. | Web |
| `/web/export/csv` | `web_export_comma_separated_values` | page or file | signed in | GET, POST | no | `data` | Streams the selected records and fields as a comma-separated values file. | Web |
| `/web/export/xlsx` | `web_export_xlsx` | page or file | signed in | GET, POST | no | `data` | Streams the selected records and fields as a spreadsheet workbook file. | Web |
| `/web/pivot/export_xlsx` | `export_xlsx` | page or file | signed in | GET, POST | no | `data` (accepts further named parameters) | Streams the current pivot table, with its row groups, column groups, measures and totals, as a spreadsheet workbook file. | Web |

### Report rendering and download

Report endpoints render a document template for a set of records into the requested output form. The template model,
paper formats and translation rules are in [`report-and-export-documents.md`](report-and-export-documents.md).

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/report/<converter>/<reportname>` <br> `/report/<converter>/<reportname>/<docids>` | `report_routes` | page or file | signed in | GET, POST | yes | `reportname`, `docids`, `converter` (accepts further named parameters) | Renders the named report for the given record identifiers in the requested output form (formatted document, printable document or plain text) and returns it inline. | Web |
| `/report/download` | `report_download` | page or file | signed in | GET, POST | no | `data`, `context`, `token`, `readonly` | Renders a report from a stored request descriptor and returns it as a file download, setting the completion cookie the client waits for, and returning the error page when rendering fails. | Web |
| `/report/barcode` <br> `/report/barcode/<barcode_type>/<value:path>` | `report_barcode` | page or file | public or signed in | GET, POST | no | `barcode_type`, `value` (accepts further named parameters) | Renders a machine-readable code image (of the requested symbol family, value, width, height, human-readable line and quiet zone) for embedding in a document or a screen. | Web |
| `/report/check_wkhtmltopdf` | `check_document_renderer` | remote call | signed in | POST | no | none | Returns the availability state of the printable document renderer, which the client uses to decide whether printing is offered. | Web |

### Notification bus and live connection

The bus delivers server-initiated notifications to open clients over a long-lived connection, with a polling fallback.
The channel model, the delivery guarantees and the reconnection rules are in [`../runtime/notification-bus.md`](../runtime/notification-bus.md).

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/websocket` | `websocket` | page or file | public or signed in | GET, POST | no | `version` | Upgrades the connection to a long-lived two-way channel for server-initiated notifications; a client announcing an outdated worker version is closed immediately. | Instant Messaging Bus |
| `/websocket/health` | `health` | page or file | none | GET, POST | no | none | Returns the liveness state of the notification service. | Instant Messaging Bus |
| `/websocket/on_closed` | `on_websocket_closed` | remote call | public or signed in | POST | no | none | Registers that a long-lived connection was closed, for installations that terminate the connection outside the application. | Instant Messaging Bus |
| `/websocket/peek_notifications` | `peek_notifications` | remote call | public or signed in | POST | no | `channels`, `last`, `is_first_poll` | Returns the notifications pending on the requested channels since the last received sequence number, as the fallback used when a long-lived connection cannot be established. | Instant Messaging Bus + Discuss |
| `/bus/has_missed_notifications` | `has_missed_notifications` | remote call | public or signed in | POST | no | `last_notification` | Reports whether notifications were dropped since the given sequence number, which tells the client to resynchronize its state. | Instant Messaging Bus |
| `/bus/websocket_worker_bundle` | `get_websocket_worker_bundle` | page or file | public or signed in | GET, POST | no | `v` | Returns the background worker program that maintains the shared long-lived connection for all tabs of one browser, versioned by the requested version marker. | Instant Messaging Bus |
| `/bus/get_model_definitions` | `get_model_definitions` | page or file | signed in | POST | no | `model_names_to_fetch` (accepts further named parameters) | Returns the field definitions of the entities the client keeps in its local store, for the entities named in the request. | Instant Messaging Bus |

### Progressive application, offline page and service worker

These endpoints let the client be installed as a standalone application on a device and behave predictably when the
device is offline.

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/web/manifest.webmanifest` | `webmanifest` | page or file | public or signed in | GET | no | none | Returns the installation descriptor of the whole platform as an installable application: name, colours, icons, start address and display mode. | Web |
| `/web/manifest.scoped_app_manifest` | `scoped_app_manifest` | page or file | public or signed in | GET | no | `app`, `path`, `app_name` | Returns an installation descriptor limited to one application scope, with a start address and a scope that cannot overlap another installed scope. | Web |
| `/scoped_app` | `scoped_app` | page or file | public or signed in | GET | no | `app`, `path`, `app_name` | Serves the page that offers installation of one application scope as a standalone application. | Web |
| `/scoped_app_icon_png` | `scoped_app_icon_png` | page or file | public or signed in | GET | no | `app`, `add_padding` | Returns the icon of an application scope rendered at a fixed pixel size, optionally with padding, for devices that require a fixed-size raster icon. | Web + Point of Sale Self Order |
| `/web/service-worker.js` | `service_worker` | page or file | public or signed in | GET | no | none | Returns the background script that caches the shell and serves the offline page when the device has no connection. | Web |
| `/app/offline` | `offline` | page or file | public or signed in | GET | no | none | Serves the page displayed by the background script while the device has no connection. | Web |

### Installation administration

These endpoints manage whole installations (creating, copying, restoring, removing and backing up a data set). Every one
of them is protected by the administration passphrase of the server rather than by a user session, and none of them is
protected against cross-site submission, because they are used before a session exists.

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/web/database/manager` | `manager` | page or file | none | GET, POST | no | accepts further named parameters | Serves the installation administration page listing the available installations and the operations offered on them. | Web |
| `/web/database/selector` | `selector` | page or file | none | GET, POST | no | accepts further named parameters | Serves the installation selection page shown when several installations are reachable and none is selected. | Web |
| `/web/database/list` | `list` | remote call | none | POST | no | none | Returns the names of the installations that may be listed. | Web |
| `/web/database/create` | `create` | page or file | none | POST | no | `master_pwd`, `name`, `language`, `password` (accepts further named parameters) | Creates a new installation with the given name, base language and administrator password, after verifying the administration passphrase, then signs the caller in to it. | Web |
| `/web/database/duplicate` | `duplicate` | page or file | none | POST | no | `master_pwd`, `name`, `new_name`, `neutralize_database` | Copies an installation under a new name after verifying the administration passphrase, optionally neutralizing it (disabling outbound communication and scheduled work in the copy). | Web |
| `/web/database/drop` | `drop` | page or file | none | POST | no | `master_pwd`, `name` | Removes an installation and its stored files after verifying the administration passphrase. | Web |
| `/web/database/backup` | `backup` | page or file | none | POST | no | `master_pwd`, `name`, `backup_format`, `filestore` | Streams a backup of an installation, with or without its stored files, in the requested archive form, after verifying the administration passphrase. | Web |
| `/web/database/restore` | `restore` | page or file | none | POST | no | `master_pwd`, `backup_file`, `name`, `copy`, `neutralize_database` | Restores an uploaded backup as an installation under the given name, either as a copy (new identity) or as the same installation, after verifying the administration passphrase. | Web |
| `/web/database/change_password` | `change_password` | page or file | none | POST | no | `master_pwd`, `master_pwd_new` | Replaces the administration passphrase of the server after verifying the current one. | Web |
| `/web/health` | `health` | page or file | none | GET, POST | no | `database_server_status` | Returns the liveness of the application and, when requested, whether the data store accepts connections; answers with the failure status when the data store is unreachable. | Web |

### Setup assistance and diagnostics

These endpoints support the first-run assistance panel, the performance profiler and the automated client test suites.

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/base_setup/data` | `base_setup_data` | remote call | signed in | POST | no | accepts further named parameters | Returns the first-run assistance summary of the installation (company, users, applications installed) used by the setup panel, extended by the sign-up package with the invitation state. | Initial Setup Tools + Signup |
| `/base_setup/demo_active` | `base_setup_is_demo` | remote call | signed in | POST | no | accepts further named parameters | Reports whether the installation was created with sample data. | Initial Setup Tools |
| `/kpi/summary` | `key_performance_indicator_summary` | remote call | none | POST | no | `credentials` | Returns the activity indicator summaries of several installations hosted together, for each pair of installation name and application key that could be verified; installations that are absent, that fail key verification or that run another platform version are left out of the answer. | Initial Setup Tools |
| `/web/set_profiling` | `profile` | page or file | public or signed in | GET, POST | no | `profile`, `collectors` (accepts further named parameters) | Turns performance recording on or off for the current session and selects the collectors to activate. | Web |
| `/web/profile_config/<profile>` | `profile_configuration` | page or file | signed in | GET, POST | no | `profile`, `action` (accepts further named parameters) | Serves the configuration page of one stored performance recording and applies the requested change to it. | Web |
| `/web/speedscope/<profile>` | `speedscope` | page or file | signed in | GET, POST | no | `profile`, `action` (accepts further named parameters) | Serves the interactive flame view of one stored performance recording. | Web |
| `/web/tests` | `unit_tests_suite` | page or file | signed in | GET, POST | no | `mod` (accepts further named parameters) | Serves the client-side test suite page for the requested capability packages. | Web |
| `/web/tests/legacy` | `test_suite` | page or file | signed in | GET, POST | no | `mod` (accepts further named parameters) | Serves the second client-side test suite page, for the test files written against the earlier client test harness. | Web |

## Identity, Authentication and Access Control

These endpoints carry the sign-in variants that are not part of the plain password form: the second authentication factor,
the device-bound credential (passkey) exchange, the delegated sign-in through an external identity provider, the sign-up
and password reset flows, and the re-identification prompt raised when a sensitive operation is attempted after the
identity check has aged. The rules behind them (password policy, factor enrolment, trusted device window) belong to
[`../domains/identity-and-access/workflows.md`](../domains/identity-and-access/workflows.md).

### Second factor and device-bound credentials

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/web/login/totp` | `web_time_based_one_time_password` | page or file | public or signed in | GET, POST | yes | `redirect` (accepts further named parameters) | Serves and processes the second-factor form after a correct password: it accepts the time-based one-time code, optionally remembers the device for the configured trusted period, and continues to the return target; the invitation package adds the enrolment invitation path for users who have not yet configured a factor. | Two-Factor Authentication (Time Based One Time Password) + two factor authentication Invite mail |
| `/auth/passkey/start-auth` | `structured_data_start_authentication` | remote call | public or signed in | POST | no | none | Starts a device-bound credential (passkey) authentication by returning the challenge and the allowed credential descriptors; the identity-timeout package reuses the same call to start a re-identification challenge inside an open session. | Passkeys + Auth Timeout |
| `/.well-known/assetlinks.json` | `web_well_known_android` | page or file | public or signed in | GET, POST | no | none | Publishes the mobile application association document that lets a companion mobile application be accepted as an origin for device-bound credentials. | Passkeys |
| `/.well-known/change-password` | `well_known_change_password` | page or file | public or signed in | GET | no | none | Redirects to the password change form at the address password managers probe by convention. | Signup |
| `/auth-timeout/check-identity` | `check_identity` | page or file | signed in | GET, POST | yes | `redirect` | Serves the re-identification page shown when an operation requires a fresh identity check, carrying the address to return to after the check succeeds. | Auth Timeout |
| `/auth-timeout/session/check-identity` | `check_identity_session` | remote call | signed in | POST | no | accepts further named parameters | Receives the re-identification form (password, one-time code or device-bound credential assertion) and, when it verifies, refreshes the identity check timestamp of the session. | Auth Timeout |
| `/auth-timeout/send-totp-mail-code` | `send_time_based_one_time_password_mail_code` | remote call | signed in | POST | no | none | Sends the one-time code by electronic mail when the user chooses the mail delivery of the second factor during re-identification. | Auth Timeout |

### Delegated sign-in, sign-up and password reset

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/auth_oauth/signin` | `signin` | page or file | none | GET, POST | no | accepts further named parameters | Receives the redirection back from an external identity provider, verifies the returned token with that provider, matches or creates the local user, opens a session and continues to the requested target. | OAuth2 Authentication |
| `/auth_oauth/oea` | `oea` | page or file | none | GET, POST | no | accepts further named parameters | Starts the delegated sign-in flow with the publisher account service, which is the provider preconfigured for support access. | OAuth2 Authentication |
| `/web/signup` | `web_authentication_signup` | page or file | public or signed in | GET, POST | yes | accepts further named parameters | Serves and processes the self-registration form: it validates the invitation token when one is present, enforces the sign-up policy of the installation, creates the user and the linked contact, signs the new user in and continues to the landing page. | Signup |
| `/web/reset_password` | `web_authentication_reset_password` | page or file | public or signed in | GET, POST | yes | accepts further named parameters | Serves and processes the password reset request form: it sends the reset message to the address of a known user, and, when opened with a valid reset token, accepts the new password and signs the user in. | Signup |

## Customer Portal

The customer portal is the signed-in area where an external contact sees the documents addressed to it. Every portal page
is reachable in two ways: by a signed-in user with the reading rules of a portal user, or through a shared link whose
access token authorizes a single record for an anonymous visitor. Document-specific portal pages (invoices, orders,
tasks, tickets) are listed under their own domains; the endpoints below are the frame proper.

### Portal home, counters and account

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/my` <br> `/my/home` | `home` | page or file | signed in | GET, POST | yes | accepts further named parameters | Serves the portal home page with one summary card per document family the visitor may see. | Customer Portal |
| `/my/counters` | `counters` | remote call | signed in | POST | yes | `counters` (accepts further named parameters) | Returns the record counts for the requested document families and remembers them in the session, therefore the home cards can be refreshed without reloading the page. | Customer Portal |
| `/my/account` | `account` | page or file | signed in | GET, POST | yes | accepts further named parameters | Serves and processes the personal details form of the signed-in portal user (name, electronic mail address, telephone, address, tax identification number and company name), rejecting changes to fields the portal user may not alter. | Customer Portal |
| `/my/security` | `security` | page or file | signed in | GET, POST | yes | accepts further named parameters | Serves the security page of the portal user: password change, second-factor management, device-bound credentials, application keys and the account deletion request. | Customer Portal |
| `/my/deactivate_account` | `deactivate_account` | page or file | signed in | POST | yes | `validation`, `password` (accepts further named parameters) | Deactivates the account of the signed-in portal user after the confirmation word and the password are both verified, then ends the session. | Customer Portal |
| `/scoped_app/<subpath:path>` <br> `/app` <br> `/app/<subpath:path>` <br> `/web` | `web_client` | page or file | signed in | GET, POST | no | `s_action` (accepts further named parameters) | Extends the desktop client entry point in installations with a portal: a signed-in external user is sent to the portal home instead of the desktop client shell. | Customer Portal |

### Portal address book

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/my/addresses` | `my_addresses` | page or file | signed in | GET, POST | yes | accepts further named parameters | Lists the invoicing and delivery addresses attached to the contact of the portal user. | Customer Portal |
| `/my/address` | `portal_address` | page or file | signed in | GET | yes | `partner`, `address_type`, `use_delivery_as_billing` (accepts further named parameters) | Serves the address form for creating a new address or editing an existing one, for the requested address type, optionally marking the delivery address as identical to the invoicing address. | Customer Portal |
| `/my/address/submit` | `portal_address_submit` | page or file | signed in | POST | yes | `partner` (accepts further named parameters) | Validates and saves the address form, creating or updating the address record, and returns either the address to continue to or the list of invalid fields with their messages. | Customer Portal |
| `/my/address/archive` | `address_archive` | remote call | signed in | POST | yes | `partner` | Archives an address of the portal user, refusing when the address is the main contact itself or is still used by an open document. | Customer Portal |
| `/my/address/country_info/<country:record of Country>` | `portal_address_country_info` | remote call | public or signed in | POST | yes | `country`, `address_type` (accepts further named parameters) | Returns the address rules of a country (whether the state is required, the label and format of the postal code, the list of states) for the address form, for the requested address type. | Customer Portal |

### Discussion thread, attachments and ratings on portal pages

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/portal/chatter_init` | `portal_chatter_init` | remote call | public or signed in | POST | yes | `thread_model`, `thread` (accepts further named parameters) | Returns the initial state of the discussion thread of a portal document: the visible messages, the composition rights, and the identity of the current visitor. | Customer Portal |
| `/mail/chatter_fetch` | `portal_message_fetch` | remote call | public or signed in | POST | yes | `thread_model`, `thread`, `fetch_parameters` (accepts further named parameters) | Returns a further page of messages of the discussion thread of a portal document, honouring the access token when the visitor is anonymous. | Customer Portal |
| `/mail/update_is_internal` | `portal_message_update_is_internal` | remote call | signed in | POST | yes | `message`, `is_internal` | Marks a thread message as internal or external, which decides whether the portal visitor sees it. | Customer Portal |
| `/mail/unfollow` | `mail_action_unfollow` | page or file | signed in | GET, POST | yes | `model_name`, `related_record_identifier`, `pid`, `token` (accepts further named parameters) | Removes the signed link recipient from the followers of a document, from the unsubscribe link of a notification message. | Customer Portal + Discuss |
| `/mail/avatar/mail.message/<res_id:integer>/author_avatar/<width:integer>x<height:integer>` | `portal_avatar` | page or file | public or signed in | GET, POST | no | `related_record_identifier`, `height`, `width`, `access_token`, `hash`, `pid` | Streams the author picture shown next to a message (a Thread Message record) in a portal discussion thread, at the requested size, authorized by the access token of the document. | Customer Portal |
| `/portal/attachment/remove` | `attachment_remove` | remote call | public or signed in | POST | no | `attachment`, `access_token` | Deletes an attachment that is still in the pending state (uploaded but not yet sent with a message), for a visitor holding either the access rights or a valid access token. | Customer Portal |
| `/website/rating/comment` | `publish_rating_comment` | remote call | signed in | POST | yes | `rating`, `publisher_comment` | Saves the published reply of the company to a customer rating and returns the stored reply text. | Portal Rating |

## General Ledger and Invoicing Documents

The accounting endpoints serve the customer-facing invoice pages, the download of the legal document forms of an invoice,
the mail preference link carried by invoice notifications, and the product catalogue helper used when building order and
invoice lines from a product grid.

### Customer invoice portal

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/my/invoices` <br> `/my/invoices/page/<page:integer>` | `portal_my_invoices` | page or file | signed in | GET, POST | yes | `page`, `date_begin`, `date_end`, `sortby`, `filterby` (accepts further named parameters) | Lists the invoices and credit notes addressed to the contact of the visitor, with paging, a date range filter, sorting by date, due date, name or amount, and grouping by state. | Invoicing |
| `/my/invoices/<invoice_id:integer>` | `portal_my_invoice_detail` | page or file | public or signed in | GET, POST | yes | `invoice`, `access_token`, `report_type`, `download` (accepts further named parameters) | Serves one invoice as a portal page or streams its printable form when a download is requested; the payment package adds the pay action with the saved payment methods and the amount to pay, and the Indonesian localization adds the tax document attachment to the page. | Invoicing + Payment - Account + Indonesia Localization |
| `/my/journal/<journal_id:integer>/unsubscribe` | `portal_my_journal_unsubscribe` | page or file | public or signed in | GET, POST | yes | `journal` (accepts further named parameters) | Stops the electronic mail notifications a contact receives for the documents of one journal, verified by the signed link token, and renders the confirmation page; an invalid or mismatched token is answered with the forbidden status and the message "Invalid token". | Invoicing |
| `/terms` | `terms_conditions` | page or file | public or signed in | GET, POST | yes | accepts further named parameters | Serves the general terms and conditions page of the company, the target of the link printed on quotations and invoices. | Invoicing |

### Legal document downloads

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/account/download_invoice_documents/<invoices:record of Journal Entry>/<filetype:text>` | `download_invoice_documents_filetype` | page or file | signed in | GET, POST | no | `invoices`, `filetype`, `allow_fallback` | Streams the legal document forms of the selected invoices in the requested form (structured form, printable form or every available form), returning a single file for one document and a compressed archive for several; when the structured form cannot be produced for a single invoice the request fails with the message "Error while creating Markup Document:" followed by the list of reasons. | Invoicing |
| `/account/download_move_attachments/<moves:record of Journal Entry>` | `download_move_attachments` | page or file | signed in | GET, POST | no | `moves` | Streams the attachments of the selected journal entries as a single file or as a compressed archive, renaming duplicates, therefore no two members share a name. | Invoicing |
| `/account/download_invoice_attachments/<attachments:record of Attachment>` | `download_invoice_attachments` | page or file | signed in | GET, POST | no | `attachments` | Streams the selected invoice attachments, as a single file when one is selected and as a compressed archive otherwise. | Invoicing |

### Product catalogue sections on orders and invoices

The product catalogue is the grid used to add lines to an order or an invoice by picking products. These calls maintain
the section headings of the document being built.

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/product/catalog/get_sections` | `product_catalog_get_sections` | remote call | signed in | POST | no | `related_record_model`, `order`, `child_field` (accepts further named parameters) | Returns the section headings that already exist on the document being edited, with their sequence, therefore the catalogue can offer them as drop targets. | Invoicing |
| `/product/catalog/create_section` | `product_catalog_create_section` | remote call | signed in | POST | no | `related_record_model`, `order`, `child_field`, `name`, `position` (accepts further named parameters) | Creates a new section heading line on the document at the requested position and returns it. | Invoicing |
| `/product/catalog/resequence_sections` | `product_catalog_resequence_sections` | remote call | signed in | POST | no | `related_record_model`, `order`, `sections`, `child_field` (accepts further named parameters) | Rewrites the sequence of the section headings of the document after they were reordered in the catalogue. | Invoicing |

### Shared computation verification

These two endpoints exist only to compare the client-side and server-side implementations of the tax computation on the
same input, and they are used by the automated conformance suite.

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/account/init_tests_shared_js_python` | `route_init_tests_shared_js_python` | page or file | signed in | GET, POST | yes | none | Serves the page that runs the shared tax computation conformance suite in the client. | Invoicing |
| `/account/post_tests_shared_js_python` | `route_post_tests_shared_js_python` | remote call | signed in | POST | no | `results` | Stores the results the client produced for the shared tax computation conformance suite, for comparison with the server results. | Invoicing |

## Accounts Receivable Payment Actions

These endpoints let a customer pay one invoice, or every overdue invoice at once, from the portal. The transaction they
create is processed by the payment provider endpoints of the payment domain.

### Paying invoices from the portal

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/invoice/transaction/<invoice_id:integer>` | `invoice_transaction` | remote call | public or signed in | POST | no | `invoice`, `access_token` (accepts further named parameters) | Creates a draft payment transaction for one invoice and returns the values the client needs to continue the payment; the request is rejected with a validation error when the invoice reference or the access token does not match. | Payment - Account |
| `/invoice/transaction/overdue` | `overdue_invoices_transaction` | remote call | public or signed in | POST | no | `payment_reference` (accepts further named parameters) | Creates a single draft payment transaction covering every overdue invoice of the signed-in customer and returns the values needed to continue the payment; the request is rejected when no user is signed in or when the overdue invoices do not all share one currency and company. | Payment - Account |
| `/my/invoices/overdue` | `portal_my_overdue_invoices` | page or file | public or signed in | GET | yes | `access_token` (accepts further named parameters) | Serves the page that lists the overdue invoices of the customer with their total and offers a single payment for all of them. | Payment - Account |

## Electronic Document Interchange

The interchange endpoints are the inbound half of the document exchange network integration: the network access point
calls them to announce a new inbound document, a change in the delivery state of an outbound document, or a change in the
registration state of the participant. The outbound half, the registration states and the document forms are specified in
[`external-integrations.md`](external-integrations.md) and in
[`../domains/electronic-invoicing-and-document-exchange/workflows.md`](../domains/electronic-invoicing-and-document-exchange/workflows.md).

### Document exchange network callbacks

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/peppol/webhook/new-message` | `webhook_new_message` | page or file | public or signed in | POST | no | `token` | Announces that one or more inbound documents are waiting at the access point; the handler verifies the shared token of the registration, then schedules the retrieval that creates the draft vendor bills. | Peppol |
| `/peppol/webhook/message-state-update` | `webhook_message_update` | page or file | public or signed in | POST | no | `token` | Announces that the delivery state of previously sent documents changed; the handler verifies the shared token and updates the exchange state of the matching documents. | Peppol |
| `/peppol/webhook/user-state-update` | `webhook_user_update` | page or file | public or signed in | POST | no | `token` | Announces that the registration state of the participant changed at the access point; the handler verifies the shared token and updates the registration state of the company. | Peppol |
| `/peppol/authentication/callback` | `peppol_authentication_callback` | page or file | signed in | GET | no | `authentication_type`, `connect_token`, `authentication_token`, `state` | Receives the redirection back from the identity verification of the access point and stores the returned registration token against the pending registration. | Peppol |
| `/peppol/authentication/webhook` | `peppol_authentication_webhook` | page or file | public or signed in | POST | no | `authentication_type`, `connect_token`, `authentication_token` | Receives the positive identity verification decision from the access point and completes the registration of the participant without further user action. | Peppol |

## Fiscal Localizations

Country configuration packages add endpoints of two kinds: callbacks by which a national electronic invoicing platform or
its certified intermediary announces a decision, and storefront steps that collect the extra data a national invoice
requires before the order is confirmed.

### National platform callbacks and authorizations

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/l10n_ro_edi/authorize/<company_id:integer>` | `authorize` | page or file | signed in | GET, POST | no | `company` (accepts further named parameters) | Redirects a signed-in user to the authorization page of the Romanian tax administration with the registered client identity and return address, refusing with the message "Client ID and Client Secret field must be filled." when the company has no registered client credentials. | Romania Localization: Electronic Invoicing |
| `/l10n_ro_edi/callback/<company_id:integer>` | `callback` | page or file | signed in | GET, POST | no | `company` (accepts further named parameters) | Receives the authorization answer of the Romanian tax administration, exchanges the returned key for an access and refresh token pair, stores both on the company, and raises an error naming the received key when the key is missing or the exchange fails. | Romania Localization: Electronic Invoicing |
| `/api/signaturit_authentication_status/1/webhooks` | `notify_authentication_status` | page or file | public or signed in | POST | no | none | Receives the identity verification decision of the certified French exchange platform: it matches the company by the verification reference, refreshes the registration state when the decision is complete, answers the bad-request status when the reference or the decision is missing, and the not-found status when no company matches. | France Localization: Electronic Invoicing (Approved Platform) |
| `/peppol/webhook/new-regulatory-message` | `webhook_regulatory_message` | page or file | public or signed in | POST | no | `token` | Announces that a regulatory message addressed to the participant is waiting on the certified French exchange platform, verified by the shared token of the registration. | France Localization: Electronic Invoicing (Approved Platform) |
| `/nemhandel/webhook/new-message` | `webhook_nemhandel_new_message` | page or file | public or signed in | POST | no | `token` | Announces inbound documents waiting on the Danish national exchange network, verified by the shared token, and schedules their retrieval. | Denmark Localization: Nemhandel |
| `/nemhandel/webhook/message-state-update` | `webhook_nemhandel_message_update` | page or file | public or signed in | POST | no | `token` | Announces a change in the delivery state of documents sent over the Danish national exchange network, verified by the shared token. | Denmark Localization: Nemhandel |
| `/nemhandel/webhook/user-state-update` | `webhook_nemhandel_user_update` | page or file | public or signed in | POST | no | `token` | Announces a change in the registration state of the participant on the Danish national exchange network, verified by the shared token. | Denmark Localization: Nemhandel |

### Storefront and portal steps required by a national invoice

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/shop/l10n_tw_invoicing_info` | `localization_tw_invoicing_info_get` | page or file | public or signed in | GET | yes | accepts further named parameters | Serves the additional checkout step of the Taiwanese electronic invoice: the invoice carrier type, the carrier number, the donation code and the business tax identification number. | Taiwan Localization: Electronic Invoicing Electronic Commerce |
| `/shop/l10n_tw_invoicing_info/submit` | `localization_tw_invoicing_info_post` | page or file | public or signed in | POST | yes | accepts further named parameters | Validates and stores the Taiwanese electronic invoice data on the order, re-rendering the step with per-field messages when the carrier number, the donation code or the tax identification number is refused by the certified intermediary. | Taiwan Localization: Electronic Invoicing Electronic Commerce |
| `/payment/ecpay/check_mobile_barcode/<sale_order_id:integer>` | `check_mobile_barcode` | remote call | public or signed in | POST | no | `sale_order` (accepts further named parameters) | Asks the certified Taiwanese intermediary whether a mobile carrier code exists and returns the answer to the checkout step. | Taiwan Localization: Electronic Invoicing Electronic Commerce |
| `/payment/ecpay/check_love_code/<sale_order_id:integer>` | `check_love_code` | remote call | public or signed in | POST | no | `sale_order` (accepts further named parameters) | Asks the certified Taiwanese intermediary whether a donation code exists and returns the answer to the checkout step. | Taiwan Localization: Electronic Invoicing Electronic Commerce |
| `/invoice/ecpay/agreed_invoice_allowance/<invoice_id:integer>` | `agreed_invoice_allowance` | page or file | public or signed in | POST | no | `invoice`, `access_token` (accepts further named parameters) | Records the agreement of the customer to an invoice allowance (the Taiwanese credit note form) from the portal link, authorized by the access token of the invoice. | Taiwan Localization: Electronic Invoicing |
| `/l10n_id_efaktur_coretax/download_attachments/<attachments:record of Attachment>` | `download_invoice_attachments` | page or file | signed in | GET, POST | no | `attachments` | Streams the Indonesian tax document attachments of the selected invoices, as one file or as a compressed archive. | Indonesia Localization: E-faktur (Coretax) |
| `/portal/state_infos/<state:record of Country Subdivision>` | `state_infos` | remote call | public or signed in | POST | yes | `state` (accepts further named parameters) | Returns the districts and provinces that belong to a state, for the cascading address selection required by the Peruvian address format. | Peru Localization |
| `/portal/city_infos/<city:record of City>` | `city_infos` | remote call | public or signed in | POST | yes | `city` (accepts further named parameters) | Returns the districts that belong to a city, for the cascading address selection required by the Peruvian address format. | Peru Localization |

## Calendar and Scheduling

Calendar endpoints answer invitation links sent by electronic mail, deliver the reminder pop-ups to open clients, and
drive the synchronization with external calendar services. Invitation links authenticate with the invitation token of the
attendee record rather than with a session: the token identifies the attendee, and forwarding the message to another
person is refused.

### Invitation answers

All five invitation endpoints authenticate with the attendee invitation token. When the token matches no attendee the
request is refused with the message "Invalid Invitation Token."; when a different user is signed in, the request is
refused with the message "Invitation cannot be forwarded via email. This event/meeting belongs to <invited address> and
you are logged in as <signed-in address>. Please ask organizer to add you."

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/calendar/meeting/accept` | `accept_meeting` | page or file | invitation token | GET, POST | no | `token`, `identifier` (accepts further named parameters) | Sets the answer of the attendee of one meeting to accepted and shows the meeting page. | Calendar |
| `/calendar/meeting/decline` | `decline_meeting` | page or file | invitation token | GET, POST | no | `token`, `identifier` (accepts further named parameters) | Sets the answer of the attendee of one meeting to declined and shows the meeting page. | Calendar |
| `/calendar/recurrence/accept` | `accept_recurrence` | page or file | invitation token | GET, POST | no | `token`, `identifier` (accepts further named parameters) | Sets the answer of the attendee to accepted for every remaining occurrence of a repeating meeting. | Calendar |
| `/calendar/recurrence/decline` | `decline_recurrence` | page or file | invitation token | GET, POST | no | `token`, `identifier` (accepts further named parameters) | Sets the answer of the attendee to declined for every remaining occurrence of a repeating meeting. | Calendar |
| `/calendar/meeting/view` | `view_meeting` | page or file | invitation token | GET, POST | no | `token`, `identifier` (accepts further named parameters) | Shows the public page of one meeting to the invited attendee, with its time, place, description, attendee list and answer buttons. | Calendar |
| `/calendar/meeting/join` | `calendar_join_meeting` | page or file | signed in | GET, POST | yes | `token` (accepts further named parameters) | Adds the signed-in user as an attendee of a meeting that accepts self-registration, using the sharing token of the meeting. | Calendar |
| `/calendar/join_videocall/<access_token:text>` | `calendar_join_videocall` | page or file | public or signed in | GET, POST | no | `access_token` | Opens the video meeting room attached to a meeting from its sharing token, answering not found when the token matches no meeting. | Calendar |

### Reminders and external calendar synchronization

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/calendar/notify` | `notify` | remote call | signed in | POST | no | none | Returns the reminders that are due for the signed-in user, which the client shows as pop-ups. | Calendar |
| `/calendar/notify_ack` | `notify_ack` | remote call | signed in | POST | no | none | Records that the user acknowledged the reminders, which stops them from being offered again. | Calendar |
| `/calendar/check_credentials` | `check_calendar_credentials` | remote call | signed in | POST | no | none | Reports whether the external calendar synchronizations configured for the user still hold valid credentials; the calendar connector adds the state of its own stored authorization to the answer. | Calendar + Google Calendar |
| `/google_calendar/sync_data` | `google_calendar_sync_data` | remote call | signed in | POST | no | `model_name` (accepts further named parameters) | Starts or continues the two-way synchronization with the external calendar service of the first provider and returns the resulting state: configuration missing, authorization needed, refresh needed, synchronization stopped, or success. | Google Calendar |
| `/microsoft_calendar/sync_data` | `microsoft_calendar_sync_data` | remote call | signed in | POST | no | `model_name` (accepts further named parameters) | Starts or continues the two-way synchronization with the external calendar service of the second provider and returns the same set of states. | Outlook Calendar |

## Attendance Recording

The attendance endpoints serve the shared check-in station (a device left in the entrance hall of a site) and the personal
check-in control in the desktop client. The station has no session: it is opened with a station token that belongs to a
company, and every call repeats that token.

### Shared check-in station

Every call of this group carries the station token of the company. A call whose token does not match an active station
configuration is refused.

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/hr_attendance/<token>` | `open_kiosk_mode` | page or file | public or signed in | GET, POST | yes | `token`, `from_trial_mode` | Opens the check-in station page of the company that owns the token, in trial mode when requested, showing the identification method the company configured (badge scan, personal code, or manual selection). | Attendances |
| `/hr_attendance/kiosk_mode_menu/<company_id:integer>` | `kiosk_menu_item_action` | page or file | signed in | GET, POST | no | `company` | Opens the check-in station for a company from the desktop client menu, generating the station address that carries the token. | Attendances |
| `/hr_attendance/attendance_barcode_scanned` | `scan_barcode_with_geolocation` | remote call | public or signed in | POST | no | `token`, `barcode`, `latitude`, `longitude` | Records a check-in or check-out for the employee whose badge code was scanned, storing the reported position of the station, and returns the greeting data to display. | Attendances |
| `/hr_attendance/manual_selection` | `manual_selection` | remote call | public or signed in | POST | no | `token`, `employee`, `personal_identification_number_code`, `latitude`, `longitude` | Records a check-in or check-out for the employee picked from the station list, after the personal identification code is verified when one is required, storing the reported position. | Attendances |
| `/hr_attendance/employees_infos` | `employees_infos` | remote call | public or signed in | POST | no | `token`, `limit`, `offset`, `domain` | Returns a page of employees of the station company, filtered by the supplied condition, with their current presence state, for the selection list of the station. | Attendances |
| `/hr_attendance/attendance_employee_data` | `employee_attendance_data` | remote call | public or signed in | POST | no | `token`, `employee` | Returns the attendance summary of one employee (last check-in, worked hours today, worked hours this week, overtime balance) after an identification at the station. | Attendances |
| `/hr_attendance/get_employees_without_badge` | `get_employees_without_badge` | remote call | public or signed in | POST | no | `token`, `name`, `limit` | Returns the employees of the station company that have no badge code yet, matching the typed name, for the badge assignment step. | Attendances |
| `/hr_attendance/set_badge` | `set_badge` | remote call | public or signed in | POST | no | `employee`, `badge`, `token` | Stores a scanned badge code on an employee that had none, from the station. | Attendances |
| `/hr_attendance/create_employee` | `create_employee` | remote call | public or signed in | POST | no | `name`, `token` | Creates an employee record from the station with the typed name, for installations that let new arrivals enrol themselves. | Attendances |
| `/hr_attendance/set_settings` | `set_attendance_settings` | remote call | public or signed in | POST | no | `token`, `mode` | Changes the identification mode of the station (badge scan or manual selection) from the station device. | Attendances |
| `/hr_attendance/kiosk_keepalive` | `kiosk_keepalive` | remote call | signed in | POST | no | none | Refreshes the station session, therefore an idle station is not signed out. | Attendances |

### Personal check-in control

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/hr_attendance/systray_check_in_out` | `systray_attendance` | remote call | signed in | POST | no | `latitude`, `longitude` | Records a check-in or a check-out for the employee of the signed-in user from the control in the client header, storing the reported position, and returns the new presence state and the worked hours of the day. | Attendances |
| `/hr_attendance/attendance_user_data` | `user_attendance_data` | remote call | signed in | POST | no | none | Returns the presence state, the last check-in time, the hours worked today and the overtime balance of the employee of the signed-in user. | Attendances |

## Employee Services

These endpoints serve the periodic activity summary message and the meal ordering service.

### Periodic activity summary

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/digest/<digest_id:integer>/unsubscribe` | `digest_unsubscribe` | page or file | public or signed in | GET, POST | yes | `digest`, `token`, `user`, `one_click` | Removes the recipient from a periodic activity summary, verified by the signed link token, and shows the confirmation page. | Key Performance Indicator Digests |
| `/digest/<digest_id:integer>/unsubscribe_oneclik` | `digest_unsubscribe_oneclick` | page or file | public or signed in | POST | yes | `digest`, `token`, `user` | Removes the recipient from a periodic activity summary through the one-click unsubscribe header of the message; only submissions are accepted, which prevents automatic unsubscription by mail scanners, and form-encoded bodies are accepted. | Key Performance Indicator Digests |
| `/digest/<digest_id:integer>/set_periodicity` | `digest_set_periodicity` | page or file | signed in | GET, POST | yes | `digest`, `periodicity` | Changes the sending rhythm of a periodic activity summary (daily, weekly, monthly or quarterly) from the link in the message. | Key Performance Indicator Digests |

### Meal ordering

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/lunch/infos` | `infos` | remote call | signed in | POST | no | `user`, `context` | Returns the meal ordering panel of the user: account balance, the lines of the current order with product, options, quantity, price, state, date, location and note, and the paid and unpaid subtotals. | Lunch |
| `/lunch/pay` | `pay` | remote call | signed in | POST | no | `user`, `context` | Sends the new lines of the current meal order to the vendor, returning true when at least one line was sent and false when there was nothing to send. | Lunch |
| `/lunch/trash` | `trash` | remote call | signed in | POST | no | `user`, `context` | Cancels and deletes the lines of the current meal order that have not yet been sent or confirmed. | Lunch |
| `/lunch/payment_message` | `payment_message` | remote call | signed in | POST | no | none | Returns the payment instruction message shown when the balance of the account is not sufficient. | Lunch |
| `/lunch/user_location_get` | `get_user_location` | remote call | signed in | POST | no | `user`, `context` | Returns the delivery location last used by the user, or the first location available to the active companies when none applies. | Lunch |
| `/lunch/user_location_set` | `set_user_location` | remote call | signed in | POST | no | `source_location`, `user`, `context` | Stores the delivery location chosen by the user for the next meal orders. | Lunch |

## Time Off Approvals

Approval links let a manager approve or refuse a time off request or an allocation directly from the notification message,
without opening the desktop client. Each link carries the record reference and a signed token; an invalid token is
refused.

### Approval links

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/leave/validate` | `human_resources_holidays_request_validate` | page or file | signed in | GET | no | `related_record_identifier`, `token` | Approves a time off request from the notification link and shows the result page. | Time Off |
| `/leave/approve` | `human_resources_holidays_request_approve` | page or file | signed in | GET | no | `related_record_identifier`, `token` | Gives the first approval of a time off request that requires two approvals, from the notification link. | Time Off |
| `/leave/refuse` | `human_resources_holidays_request_refuse` | page or file | signed in | GET | no | `related_record_identifier`, `token` | Refuses a time off request from the notification link and shows the result page. | Time Off |
| `/allocation/validate` | `human_resources_holidays_allocation_validate` | page or file | signed in | GET | no | `related_record_identifier`, `token` | Approves an allocation of time off days from the notification link. | Time Off |
| `/allocation/refuse` | `human_resources_holidays_allocation_refuse` | page or file | signed in | GET | no | `related_record_identifier`, `token` | Refuses an allocation of time off days from the notification link. | Time Off |

## Employee Directory



### Organization chart and employee documents

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/hr/get_org_chart` | `get_org_chart` | remote call | signed in | POST | no | `employee`, `new_parent` (accepts further named parameters) | Returns the organization chart around one employee: the chain of managers up to the top, the direct reports with their own report counts, and the position the employee would take under a proposed new manager. | Human Resources Org Chart |
| `/hr/get_subordinates` | `get_subordinates` | remote call | signed in | POST | no | `employee`, `subordinates_type` (accepts further named parameters) | Returns the direct or the indirect subordinates of one employee. | Human Resources Org Chart |
| `/hr/get_redirect_model` | `get_redirect_model` | remote call | signed in | POST | no | none | Returns the entity the organization chart must open when a node is clicked, which differs for a user who may see the full employee file and one who may only see the public directory. | Human Resources Org Chart |
| `/print/cv` | `print_employee_cv` | page or file | signed in | GET, POST | no | `employee`, `color_primary`, `color_secondary` (accepts further named parameters) | Renders the skill summary sheet of one employee as a printable document, in the two chosen colours. | Skills Management |

## Online Job Positions



### Published job positions and applications

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/jobs` <br> `/jobs/page/<page:integer>` | `jobs` | page or file | public or signed in | GET, POST | yes | `country`, `all_countries`, `department`, `office`, `contract_type`, `is_remote`, `is_other_department`, `is_untyped`, `industry`, `is_industry_untyped`, `nofuzzy`, `page`, `search` (accepts further named parameters) | Lists the published job positions with the facet filters of the page (country, department, office, contract type, remote work and industry), where each facet shows the counts the other active facets allow, and a free-text search with an optional exact-match mode. | Online Jobs |
| `/jobs/<job:record of Job Position>` | `job` | page or file | public or signed in | GET, POST | yes | `job` (accepts further named parameters) | Serves the public page of one job position with its description, requirements and application button. | Online Jobs |
| `/jobs/detail/<job:record of Job Position>` | `jobs_detail` | page or file | public or signed in | GET, POST | yes | `job` (accepts further named parameters) | Serves the detail page of one job position from the listing, keeping the active filters in the navigation trail. | Online Jobs |
| `/jobs/apply/<job:record of Job Position>` | `jobs_apply` | page or file | public or signed in | GET, POST | yes | `job` (accepts further named parameters) | Serves the application form of a job position and accepts its submission, creating the application with the candidate name, electronic mail address, telephone, message, curriculum attachment and the answers of the recruitment questionnaire. | Online Jobs |
| `/jobs/add` | `jobs_add` | remote call | signed in | POST | yes | accepts further named parameters | Creates a new job position from the published listing for an editor of the site and returns the address of its page. | Online Jobs |
| `/website_hr_recruitment/check_recent_application` | `check_recent_application` | remote call | public or signed in | POST | yes | `field`, `value`, `job_position` | Reports whether the same electronic mail address or telephone number already applied to the same position recently, therefore the form can warn about a duplicate application. | Online Jobs |

## Purchasing Portal



### Vendor portal

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/my/purchase` <br> `/my/purchase/page/<page:integer>` | `portal_my_purchase_orders` | page or file | signed in | GET, POST | yes | `page`, `date_begin`, `date_end`, `sortby`, `filterby` (accepts further named parameters) | Lists the confirmed purchase orders addressed to the signed-in vendor contact, with paging, a date range filter, sorting and grouping. | Purchase |
| `/my/rfq` <br> `/my/rfq/page/<page:integer>` | `portal_my_requests_for_quotation` | page or file | signed in | GET, POST | yes | `page`, `date_begin`, `date_end`, `sortby`, `filterby` (accepts further named parameters) | Lists the requests for quotation sent to the signed-in vendor contact, with paging, a date range filter and sorting. | Purchase |
| `/my/purchase/<order_id:integer>` | `portal_my_purchase_order` | page or file | public or signed in | GET, POST | yes | `order`, `access_token` (accepts further named parameters) | Serves one purchase order or request for quotation as a portal page, with the confirm and reject actions when the document is a request for quotation, authorized by session rights or by the access token. | Purchase |
| `/my/purchase/<order_id:integer>/update` | `portal_my_purchase_order_update_dates` | remote call | public or signed in | POST | yes | `order`, `access_token` (accepts further named parameters) | Records the delivery date the vendor promises on one order line and returns the refreshed line. | Purchase |
| `/my/purchase/<order_id:integer>/download_edi` | `portal_my_purchase_order_download_electronic_data_interchange` | page or file | public or signed in | GET, POST | yes | `order`, `access_token` (accepts further named parameters) | Streams the structured interchange form of the purchase order as a downloadable file. | Purchase |

## Timesheet Portal



### Time entries in the portal

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/my/timesheets` <br> `/my/timesheets/page/<page:integer>` | `portal_my_timesheets` | page or file | signed in | GET, POST | yes | `page`, `sortby`, `filterby`, `search`, `search_inch`, `groupby` (accepts further named parameters) | Lists the time entries the signed-in portal contact may see, with paging, sorting, grouping by project, task, date or employee, and free-text search; with the billing package installed the list also shows the invoicing state of each entry. | Task Logs + Sales Timesheet |
| `/my/tasks/<task_id>/orders/invoices` <br> `/my/tasks/<task_id>/orders/invoices/page/<page:integer>` | `portal_my_tasks_invoices` | page or file | signed in | GET, POST | yes | `task`, `page`, `date_begin`, `date_end`, `sortby`, `filterby` (accepts further named parameters) | Lists the invoices produced from the time entries of one task, with paging, a date range filter, sorting and grouping. | Sales Timesheet |

## Projects and Tasks

The project endpoints serve the customer portal pages of projects and tasks, the shared project workspace where an
external collaborator edits tasks with the full form, and the mail plug-in operations that create projects and tasks from
a message.

### Project and task portal

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/my/projects` <br> `/my/projects/page/<page:integer>` | `portal_my_projects` | page or file | signed in | GET, POST | yes | `page`, `date_begin`, `date_end`, `sortby` (accepts further named parameters) | Lists the projects the signed-in portal contact follows, with paging, a date range filter and sorting. | Project |
| `/my/projects/<project_id:integer>` <br> `/my/projects/<project_id:integer>/page/<page:integer>` | `portal_my_project` | page or file | public or signed in | GET, POST | yes | `project`, `access_token`, `page`, `date_begin`, `date_end`, `sortby`, `search`, `search_inch`, `groupby`, `task` (accepts further named parameters) | Serves one project page with its tasks, paging, a date range filter, sorting, grouping and free-text search, authorized by session rights or by the access token. | Project |
| `/my/tasks` <br> `/my/tasks/page/<page:integer>` | `portal_my_tasks` | page or file | signed in | GET, POST | yes | `page`, `date_begin`, `date_end`, `sortby`, `filterby`, `search`, `search_inch`, `groupby` (accepts further named parameters) | Lists the tasks the signed-in portal contact may see across all projects, with paging, a date range filter, sorting, grouping and free-text search. | Project |
| `/my/tasks/<task_id:integer>` | `portal_my_task` | page or file | public or signed in | GET, POST | yes | `task`, `report_type`, `access_token`, `project_sharing` (accepts further named parameters) | Serves one task page, or streams its printable form when a download is requested, authorized by session rights or by the access token, and switches to the shared workspace layout when the visitor holds edit rights on the project. | Project |
| `/my/projects/<project_id:integer>/task/<task_id:integer>` | `portal_my_project_task` | page or file | public or signed in | GET, POST | yes | `project`, `task`, `access_token` (accepts further named parameters) | Serves one task page in the context of its project, keeping the project navigation trail. | Project |
| `/my/projects/<project_id:integer>/task/<task_id:integer>/subtasks` | `portal_my_project_subtasks` | page or file | signed in | GET | yes | `project`, `task`, `page`, `date_begin`, `date_end`, `sortby`, `filterby`, `search`, `search_inch`, `groupby` (accepts further named parameters) | Lists the sub tasks of one task in the portal with paging, filtering, sorting, grouping and free-text search. | Project |
| `/my/projects/<project_id:integer>/task/<task_id:integer>/recurrent_tasks` | `portal_my_project_recurrent_tasks` | page or file | signed in | GET | yes | `project`, `task`, `page`, `date_begin`, `date_end`, `sortby`, `filterby`, `search`, `search_inch`, `groupby` (accepts further named parameters) | Lists the other occurrences of a repeating task in the portal with paging, filtering, sorting, grouping and free-text search. | Project |

### Shared project workspace

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/my/projects/<project_id:integer>/project_sharing` <br> `/my/projects/<project_id:integer>/project_sharing/<subpath:path>` | `render_project_backend_view` | page or file | signed in | GET | no | `project`, `subpath` | Serves the shared project workspace, which runs the full task views for an external collaborator with the rights granted on that project only, at the requested inner path. | Project |
| `/project_sharing/attachment/add_image` | `add_image` | page or file | signed in | POST | yes | `name`, `data`, `related_record_identifier`, `access_token` (accepts further named parameters) | Stores an image pasted or dropped into a task description in the shared workspace and returns its address, authorized by the access token of the task. | Project |

### Mail plug-in operations

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/mail_plugin/project/search` | `projects_search` | remote call | plug-in key | POST | no | `search_term`, `limit` | Returns the projects whose name contains the typed text, limited to the requested count, for the project picker of the mail plug-in. | Project Mail Plugin |
| `/mail_plugin/project/create` | `project_create` | remote call | plug-in key | POST | no | `name` | Creates a project with the given name from the mail plug-in and returns its reference. | Project Mail Plugin |
| `/mail_plugin/task/create` | `task_create` | remote call | plug-in key | POST | no | `email_subject`, `email_body`, `project`, `partner` | Creates a task in the chosen project from the open message, using its subject as the title and its body as the description, linked to the chosen contact, and returns the task reference. | Project Mail Plugin |

## Manufacturing Portal



### Subcontracting portal

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/my/productions` <br> `/my/productions/page/<page:integer>` | `portal_my_productions` | page or file | signed in | GET, POST | yes | `page`, `date_begin`, `date_end`, `sortby`, `filterby` | Lists the subcontracting transfers assigned to the signed-in subcontractor, with paging, a date range filter, sorting and grouping. | Manufacturing Subcontracting |
| `/my/productions/<picking_id:integer>` | `portal_my_production` | page or file | signed in | GET | yes | `transfer` | Serves one subcontracting transfer as a portal page with its components and finished products. | Manufacturing Subcontracting |
| `/my/productions/<picking_id:integer>/subcontracting_portal` | `render_production_backend_view` | page or file | signed in | GET | no | `transfer` | Serves the subcontractor workspace, which runs the production recording form (component consumption, lot numbers and produced quantity) with the rights granted on that transfer only. | Manufacturing Subcontracting |

### Storefront availability of assembled products

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/website_sale_mrp/get_unavailable_qty_from_kits` | `get_unavailable_quantity_from_kits` | remote call | public or signed in | POST | yes | `product` (accepts further named parameters) | Returns, for the products of the current storefront page that are assembled from components, the quantity that cannot be promised because a component is short, therefore the page can show the true availability. | Kit Availability |

## Inventory Operations



### Pickup points and inventory reports

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/delivery/get_pickup_locations` | `delivery_get_pickup_locations` | remote call | signed in | POST | no | `order`, `zip_code` | Returns the pickup points of the selected carrier near a postal code, determining the country from the network position of the visitor and falling back to the country of the delivery address of the order. | Delivery Costs |
| `/delivery/set_pickup_location` | `delivery_set_pickup_location` | remote call | signed in | POST | no | `order`, `pickup_location_data` | Stores the chosen pickup point on the order, which fixes the delivery address of the shipment. | Delivery Costs |
| `/stock/<output_format:text>/<report_name:text>` | `report` | page or file | signed in | GET, POST | no | `output_format`, `report_name` (accepts further named parameters) | Streams an inventory analysis report (the forecast report or the valuation report) in the requested file form, built from the filter conditions of the current view. | Inventory |

## Products and Catalogue



### Product catalogue grid and product files

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/product/catalog/order_lines_info` | `product_catalog_get_order_lines_info` | remote call | signed in | POST | no | `related_record_model`, `order`, `product` (accepts further named parameters) | Returns, for the products displayed in the catalogue grid, the quantity already on the document, the unit price for the price list and quantity in force, the unit of measure label, the internal reference and whether the line may still be changed. | Products & Pricelists |
| `/product/catalog/update_order_line_info` | `product_catalog_update_order_line_info` | remote call | signed in | POST | no | `related_record_model`, `order`, `product`, `quantity` (accepts further named parameters) | Sets the quantity of one product on the document being built, creating, updating or removing the line, and returns the resulting unit price; the project purchasing package extends it to attach the created line to the project of the document. | Products & Pricelists + Project Purchase |
| `/product/document/upload` | `upload_document` | page or file | signed in | POST | no | `ufile`, `related_record_model`, `related_record_identifier` (accepts further named parameters) | Attaches one or more uploaded files to a product or a product variant as product documents. | Products & Pricelists |
| `/product/export/pricelist/` | `export_pricelist` | page or file | signed in | GET, POST | no | `report_data`, `export_format` | Streams the price list report of the selected products, price list and quantities in the requested file form. | Products & Pricelists |

## Contacts and Organizations



### Tax identification verification callback

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/base_vat/1/webhook_update_vies` | `webhook_update_vies` | page or file | public or signed in | GET, POST | no | `webhook_token`, `status` | Receives the delayed answer of the tax identification verification service for a number that was left pending, verified by a token derived from the installation identity, and stores the resulting validity state on the contact. | Value Added Tax Number Validation |

## Spreadsheets and Dashboards



### Dashboards and shared spreadsheets

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/board/add_to_dashboard` | `add_to_dashboard` | remote call | signed in | POST | no | `action`, `context_to_save`, `domain`, `view_mode`, `name` | Adds the current view, with its filter condition, grouping and display mode, as a new block on the personal dashboard under the given name. | Dashboards |
| `/spreadsheet/dashboard/data/<dashboard:record of Spreadsheet Dashboard>` | `get_dashboard_data` | page or file | signed in | GET, POST | no | `dashboard` | Returns the workbook definition and the data of one dashboard for a signed-in user. | Spreadsheet dashboard |
| `/dashboard/share/<share_id:integer>/<token>` | `share_portal` | page or file | public or signed in | GET, POST | no | `share`, `token` | Serves a dashboard that was shared by link, to any visitor holding the sharing token. | Spreadsheet dashboard |
| `/dashboard/data/<share_id:integer>/<token>` | `get_shared_dashboard_data` | page or file | public or signed in | GET | no | `share`, `token` | Returns the workbook definition and the frozen data of a dashboard shared by link, to any visitor holding the sharing token. | Spreadsheet dashboard |
| `/dashboard/download/<share_id:integer>/<token>` | `download` | page or file | signed in | GET, POST | no | `token`, `share` | Streams the shared dashboard as a spreadsheet workbook file. | Spreadsheet dashboard |
| `/spreadsheet/log` | `log_action` | remote call | signed in | POST | no | `action_type`, `datasources` (accepts further named parameters) | Records which spreadsheet feature and which data sources were used, for the usage statistics of the installation. | Spreadsheet |

## Loyalty and Promotions



### Coupons, rewards and wallet

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/coupon/<code:text>` | `activate_coupon` | page or file | public or signed in | GET, POST | yes | `code`, `r` (accepts further named parameters) | Applies a promotion or coupon code carried in a shared link to the current cart and continues to the requested page, showing the reason when the code is refused. | Coupons, Promotions, Gift Card and Loyalty for Electronic Commerce |
| `/shop/claimreward` | `claim_reward` | page or file | public or signed in | GET, POST | yes | `reward`, `code` (accepts further named parameters) | Claims one reward of a loyalty or promotion programme on the current cart, optionally with the code that unlocks it, and returns the refreshed cart. | Coupons, Promotions, Gift Card and Loyalty for Electronic Commerce |
| `/wallet/top_up` | `wallet_top_up` | page or file | signed in | GET, POST | yes | accepts further named parameters | Adds a top-up product for the chosen amount to the cart, which credits the customer wallet when the order is paid. | Coupons, Promotions, Gift Card and Loyalty for Electronic Commerce |
| `/my/loyalty_card/<card_id:integer>/history` <br> `/my/loyalty_card/<card_id:integer>/history/page/<page:integer>` | `portal_my_loyalty_card_history` | page or file | signed in | GET, POST | yes | `card`, `page`, `sortby` (accepts further named parameters) | Lists the point movements of one loyalty card of the signed-in customer, with paging and sorting. | Coupons & Loyalty |
| `/my/loyalty_card/<card_id:integer>/values` | `portal_get_card_history_values` | remote call | signed in | POST | no | `card` | Returns the balance and the point movements of one loyalty card for the portal dialog; the storefront package adds the rewards the balance can buy. | Coupons & Loyalty + Coupons, Promotions, Gift Card and Loyalty for Electronic Commerce |

## Sales Quotations and Orders

The sales endpoints serve the customer portal of quotations and orders, including online acceptance with a signature, the
product configurator used both in the desktop client and on the storefront, and the fulfilment callback of the
print-on-demand partner.

### Quotation and order portal

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/my/quotes` <br> `/my/quotes/page/<page:integer>` | `portal_my_quotes` | page or file | signed in | GET, POST | yes | accepts further named parameters | Lists the quotations addressed to the signed-in customer contact, with paging, a date range filter and sorting. | Sales |
| `/my/orders` <br> `/my/orders/page/<page:integer>` | `portal_my_orders` | page or file | signed in | GET, POST | yes | accepts further named parameters | Lists the confirmed sales orders addressed to the signed-in customer contact, with paging, a date range filter, sorting and grouping. | Sales |
| `/my/orders/<order_id:integer>` | `portal_order_page` | page or file | public or signed in | GET, POST | yes | `order`, `report_type`, `access_token`, `message`, `download`, `payment_amount`, `amount_selection` (accepts further named parameters) | Serves one quotation or order as a portal page, or streams its printable form when a download is requested, showing the acceptance, refusal and payment actions that the state of the document allows, authorized by session rights or by the access token. | Sales |
| `/my/orders/<order_id:integer>/accept` | `portal_quote_accept` | remote call | public or signed in | POST | yes | `order`, `access_token`, `name`, `signature` | Accepts a quotation online: it records the signer name and the drawn or typed signature on the order, confirms it, and returns the address to continue to (the payment step when a prepayment is required). | Sales |
| `/my/orders/<order_id:integer>/decline` | `portal_quote_decline` | page or file | public or signed in | POST | yes | `order`, `access_token`, `decline_message` (accepts further named parameters) | Refuses a quotation online with the reason typed by the customer, which cancels the document and posts the reason in its discussion thread. | Sales |
| `/my/orders/<order_id:integer>/update_line_dict` | `portal_quote_option_update` | remote call | public or signed in | POST | yes | `order`, `line`, `access_token`, `remove`, `input_quantity` (accepts further named parameters) | Changes the quantity of an optional line of a quotation from the portal, either by setting the typed quantity or by adding or removing one unit, and returns the refreshed totals. | Sales |
| `/my/orders/<order_id:integer>/transaction` | `portal_order_transaction` | remote call | public or signed in | POST | no | `order`, `access_token` (accepts further named parameters) | Creates a draft payment transaction for one order and returns the values needed to continue the payment; the request is refused when the order reference or the access token does not match. | Sales |
| `/my/orders/<order_id:integer>/document/<document_id:integer>` | `portal_quote_document` | page or file | public or signed in | GET, POST | no | `order`, `document`, `access_token` | Streams one of the documents attached to the quotation offer (a product sheet or a terms document) to the customer. | Sales |
| `/my/orders/<order_id:integer>/download_edi` | `portal_my_sale_order_download_electronic_data_interchange` | page or file | public or signed in | GET, POST | yes | `order`, `access_token` (accepts further named parameters) | Streams the structured interchange form of the order as a downloadable file. | Sales |
| `/my/picking/pdf/<picking_id:integer>` | `portal_my_picking_report` | page or file | public or signed in | GET, POST | yes | `transfer`, `access_token` (accepts further named parameters) | Streams the delivery slip of one shipment to the customer, authorized by access rights or by the access token. | Sales and Warehouse Management |
| `/my/picking/return/pdf/<picking_id:integer>` | `portal_my_picking_return_report` | page or file | public or signed in | GET, POST | yes | `transfer`, `access_token` (accepts further named parameters) | Streams the return label of one shipment to the customer, authorized by access rights or by the access token. | Sales and Warehouse Management |

### Product configurator

These calls are shared by the quotation form of the desktop client and by the storefront product page. They never write:
they compute prices and availability for a candidate combination of options.

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/sale/product_configurator/get_values` | `sale_product_configurator_get_values` | remote call | signed in | POST | no | `product_template`, `quantity`, `currency`, `sales_order_date`, `unit_of_measure`, `company`, `pricelist`, `product_template_attribute_value`, `only_main_product` (accepts further named parameters) | Returns everything the configurator needs for a product: its attribute lines and values, the exclusions between values, the currently selected combination, the unit price for the price list, quantity, unit of measure and date in force, and the optional products offered with it. | Sales |
| `/sale/product_configurator/update_combination` | `sale_product_configurator_update_combination` | remote call | signed in | POST | no | `product_template`, `product_template_attribute_value`, `currency`, `sales_order_date`, `quantity`, `unit_of_measure`, `company`, `pricelist` (accepts further named parameters) | Returns the refreshed name, reference, price, extra price and availability after the selected combination of attribute values changed. | Sales |
| `/sale/product_configurator/get_optional_products` | `sale_product_configurator_get_optional_products` | remote call | signed in | POST | no | `product_template`, `product_template_attribute_value`, `parent_product_template_attribute_value`, `currency`, `sales_order_date`, `company`, `pricelist` (accepts further named parameters) | Returns the optional products offered with a chosen combination, each with its own configuration data and price. | Sales |
| `/sale/product_configurator/create_product` | `sale_product_configurator_create_product` | remote call | signed in | POST | no | `product_template`, `product_template_attribute_value` | Creates the product variant of a combination that contains a dynamically created attribute value and returns its reference. | Sales |
| `/sale/combo_configurator/get_data` | `sale_combo_configurator_get_data` | remote call | signed in | POST | no | `product_template`, `quantity`, `date`, `currency`, `company`, `pricelist`, `selected_combo_items` (accepts further named parameters) | Returns the choices of a combination offer: each selection group, the items it contains, their prices and their own configuration needs, for the given quantity, currency, price list, company and date. | Sales |
| `/sale/combo_configurator/get_price` | `sale_combo_configurator_get_price` | remote call | signed in | POST | no | `product_template`, `quantity`, `date`, `currency`, `company`, `pricelist` (accepts further named parameters) | Returns the price of a combination offer for the given quantity, currency, price list, company and date. | Sales |

### Quotation documents and fulfilment callback

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/sale_pdf_quote_builder/quotation_document/upload` | `upload_document` | page or file | signed in | POST | no | `ufile`, `sale_order_template`, `allowed_company` | Stores an uploaded document (a cover page, a terms page or a product sheet) that the quotation builder merges into the printed offer, for the chosen quotation template and company. | Sales Portable Document Quotation Builder |
| `/gelato/webhook` | `gelato_webhook` | page or file | public or signed in | POST | no | none | Receives the fulfilment notifications of the print-on-demand partner for an order: a failure posts the reason in the discussion thread of the order, a cancellation cancels the order and posts the cancellation, a shipment and a delivery each send the status message with the tracking data to the customer; every notification is verified against the signature computed from the order, and the answer is empty. | Gelato |

## Customer Relationship Management

These endpoints carry the approval links of lead notifications, the operations of the electronic mail plug-in that turns a
message into a lead, and the reseller directory with the lead pages a reseller sees.

### Lead actions from notification links

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/lead/case_mark_won` | `customer_relationship_management_lead_case_mark_won` | page or file | signed in | GET | no | `related_record_identifier`, `token` | Marks a lead as won from the notification link, verified by the signed token. | Customer Relationship Management |
| `/lead/case_mark_lost` | `customer_relationship_management_lead_case_mark_lost` | page or file | signed in | GET | no | `related_record_identifier`, `token` | Marks a lead as lost from the notification link, verified by the signed token. | Customer Relationship Management |
| `/lead/convert` | `customer_relationship_management_lead_convert` | page or file | signed in | GET | no | `related_record_identifier`, `token` | Converts a lead into an opportunity from the notification link, verified by the signed token. | Customer Relationship Management |

### Electronic mail plug-in

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/mail_plugin/lead/create` | `customer_relationship_management_lead_create` | remote call | plug-in key | POST | no | `partner`, `email_body`, `email_subject` | Creates a lead from the open message for the chosen contact, using the subject as the title and the body as the description, and returns the lead reference. | Customer Relationship Management Mail Plugin |

### Reseller directory and reseller leads

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/partners` <br> `/partners/page/<page:integer>` <br> `/partners/grade/<grade:record of Partner Grade>` <br> `/partners/grade/<grade:record of Partner Grade>/page/<page:integer>` <br> `/partners/country/<country:record of Country>` <br> `/partners/country/<country:record of Country>/page/<page:integer>` <br> `/partners/grade/<grade:record of Partner Grade>/country/<country:record of Country>` <br> `/partners/grade/<grade:record of Partner Grade>/country/<country:record of Country>/page/<page:integer>` | `partners` | page or file | public or signed in | GET, POST | yes | `country`, `grade`, `page` (accepts further named parameters) | Lists the published resellers on the site, filtered by partnership level and by country, with paging, and shows them on a map. | Resellers |
| `/my/leads` <br> `/my/leads/page/<page:integer>` | `portal_my_leads` | page or file | signed in | GET, POST | yes | `page`, `date_begin`, `date_end`, `sortby` (accepts further named parameters) | Lists the leads assigned to the signed-in reseller, with paging, a date range filter and sorting. | Resellers |
| `/my/lead/<lead:record of Lead>` | `portal_my_lead` | page or file | signed in | GET, POST | yes | `lead` (accepts further named parameters) | Serves one assigned lead to the reseller with the accept and refuse actions. | Resellers |
| `/my/opportunities` <br> `/my/opportunities/page/<page:integer>` | `portal_my_opportunities` | page or file | signed in | GET, POST | yes | `page`, `date_begin`, `date_end`, `sortby`, `filterby` (accepts further named parameters) | Lists the opportunities assigned to the signed-in reseller, with paging, a date range filter, sorting and grouping by stage. | Resellers |
| `/my/opportunity/<opp:record of Lead>` | `portal_my_opportunity` | page or file | signed in | GET, POST | yes | `opp` (accepts further named parameters) | Serves one assigned opportunity to the reseller, where the reseller may update the stage, the expected revenue, the probability, the next activity and the contact details. | Resellers |

## Payment Providers

Every online payment follows the same three-endpoint shape, whatever the provider: the payer is sent to a page that
creates a transaction, the provider sends the payer back to a return address, and the provider also calls a notification
address (a webhook) from its own servers, therefore the outcome is recorded even when the payer closes the browser. The return
address and the notification address are handled by the same provider-specific code path: both parse the received data,
find the transaction by its reference, verify the authenticity of the message (signature, message digest or a second call
back to the provider), and then apply the outcome to the transaction. Notification endpoints are never protected against
cross-site submission, because the caller is the provider, and each answers with the exact acknowledgement string the
provider expects; a wrong or missing answer makes the provider retry. Notification handling is idempotent: applying the
same outcome twice leaves the transaction in the same state. The transaction state machine, the signature rules and the
error paths are specified by the payment providers domain, whose entry is carried by the domain index
[`../domains/README.md`](../domains/README.md).

### Shared payment flow

These endpoints belong to the payment engine and are provider-independent.

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/payment/pay` | `payment_pay` | page or file | public or signed in | GET | yes | `reference`, `amount`, `currency`, `partner`, `company`, `access_token` (accepts further named parameters) | Serves the payment page for an amount, a currency, a customer, a company and a reference, showing the payment methods and saved tokens allowed by that context; a malformed parameter is ignored rather than blocking the payment; the invoicing package fills the missing values from the invoice being paid and the site package from the site configuration. | Payment Engine + Payment - Account + Website Payment |
| `/payment/transaction` | `payment_transaction` | remote call | public or signed in | POST | no | `amount`, `currency`, `partner`, `access_token` (accepts further named parameters) | Creates a draft transaction for the payment page and returns the values needed to continue: the rendering form of the chosen method, the reference and the addresses to return to; an empty amount means a payment method validation instead of a payment. | Payment Engine |
| `/payment/status` | `display_status` | page or file | public or signed in | GET, POST | yes | accepts further named parameters | Serves the payment status page that polls until the transaction reaches a final state. | Payment Engine |
| `/payment/status/poll` | `poll_status` | remote call | public or signed in | POST | no | accepts further named parameters | Returns the current state of the transaction of the session and runs its post-processing (confirming the order, reconciling the invoice, sending the confirmation) when the outcome is final. | Payment Engine |
| `/payment/confirmation` | `payment_confirm` | page or file | public or signed in | GET | yes | `tx`, `access_token` (accepts further named parameters) | Serves the confirmation page of one transaction, verified by its access token, with the outcome message and the link to the paid document. | Payment Engine |
| `/my/payment_method` | `payment_method` | page or file | signed in | GET | yes | accepts further named parameters | Serves the page where a customer manages the payment methods saved for later use; the site package adds the storefront layout around it. | Payment Engine + Website Payment |
| `/payment/archive_token` | `archive_token` | remote call | signed in | POST | no | `token` | Archives a saved payment method after checking that the caller may write on it, which prevents its further use without deleting the history. | Payment Engine |

### Provider return and notification endpoints

One row per provider endpoint. The acknowledgement column of the purpose text states the exact answer body the provider
expects.

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/payment/adyen/notification` | `adyen_webhook` | page or file | public or signed in | POST | no | none | Receives the notifications of the first provider, applies the outcome named by the event code, and answers with the string "[accepted]"; the restaurant terminal package extends it to route terminal events to the open point of sale session. | Payment Provider: Adyen + Point of Sale Restaurant Adyen |
| `/payment/adyen/payment_methods` | `adyen_payment_methods` | remote call | public or signed in | POST | no | `provider`, `formatted_amount`, `partner` | Returns the payment methods the provider offers for the amount, currency, country and customer of the pending payment, for the inline payment form. | Payment Provider: Adyen |
| `/payment/adyen/payments` | `adyen_payments` | remote call | public or signed in | POST | no | `provider`, `reference`, `converted_amount`, `currency`, `partner`, `payment_method`, `access_token`, `browser_info` | Sends the payment request with the collected method details and the browser fingerprint data, then applies the returned outcome to the transaction. | Payment Provider: Adyen |
| `/payment/adyen/payments/details` | `adyen_payment_details` | remote call | public or signed in | POST | no | `provider`, `reference`, `payment_details` | Sends the result of an additional payer action (an inline challenge or a redirection) and applies the returned outcome to the transaction. | Payment Provider: Adyen |
| `/payment/adyen/return` | `adyen_return_from_3ds_authentication` | page or file | public or signed in | GET, POST | no | accepts further named parameters | Receives the payer back from the strong authentication page and applies the outcome; the endpoint does not open a session, because the return is a cross-site submission whose cookie would be rejected. | Payment Provider: Adyen |
| `/payment/aps/return` | `aps_return_from_checkout` | page or file | public or signed in | POST | no | accepts further named parameters | Receives the payer back from the hosted page of the Amazon Payment Services provider and applies the outcome, without opening a session. | Payment Provider: Amazon Payment Services |
| `/payment/aps/webhook` | `aps_webhook` | page or file | public or signed in | POST | no | accepts further named parameters | Receives the notifications of the Amazon Payment Services provider, verifies their signature, applies the outcome and answers with the string "SUCCESS". | Payment Provider: Amazon Payment Services |
| `/payment/asiapay/return` | `asiapay_return_from_checkout` | page or file | public or signed in | GET | no | accepts further named parameters | Receives the payer back from the hosted page of the AsiaPay provider and applies the outcome. | Payment Provider: AsiaPay |
| `/payment/asiapay/webhook` | `asiapay_webhook` | page or file | public or signed in | POST | no | accepts further named parameters | Receives the notifications of the AsiaPay provider, verifies them and answers with the string "Allowed". | Payment Provider: AsiaPay |
| `/payment/authorize/payment` | `authorize_payment` | remote call | public or signed in | POST | no | `reference`, `partner`, `access_token`, `opaque_data` | Sends the payment request of the Authorize.Net provider with the opaque card data collected by the inline form, verified by the access token of the transaction, and applies the outcome. | Payment Provider: Authorize.Net |
| `/payment/buckaroo/return` | `buckaroo_return_from_checkout` | page or file | public or signed in | POST | no | accepts further named parameters | Receives the payer back from the hosted page of the Buckaroo provider and applies the outcome, without opening a session. | Payment Provider: Buckaroo |
| `/payment/buckaroo/webhook` | `buckaroo_webhook` | page or file | public or signed in | POST | no | accepts further named parameters | Receives the notifications of the Buckaroo provider, verifies their signature, applies the outcome and answers with an empty body. | Payment Provider: Buckaroo |
| `/payment/custom/process` | `custom_process_transaction` | page or file | public or signed in | POST | no | accepts further named parameters | Records the intention to pay by a manual method (bank transfer or another offline arrangement) and moves the transaction to the pending state with the instructions shown to the payer. | Payment Provider: Custom Payment Modes |
| `/payment/demo/simulate_payment` | `demo_simulate_payment` | remote call | public or signed in | POST | no | accepts further named parameters | Applies a chosen simulated outcome to the transaction, for the demonstration provider used to exercise the payment flow without a real provider. | Payment Provider: Demo |
| `/payment/dpo/return` | `dpo_return_from_checkout` | page or file | public or signed in | GET | no | accepts further named parameters | Receives the payer back from the hosted page of the DPO provider and applies the outcome. | Payment Provider: DPO |
| `/payment/ecpay/return` | `ecpay_return_from_checkout` | page or file | public or signed in | GET, POST | no | accepts further named parameters | Receives the payer back from the hosted page of the ECPay provider and applies the outcome when the return carries it, without opening a session. | Payment Provider: ECPay |
| `/payment/ecpay/webhook` | `ecpay_webhook` | page or file | public or signed in | POST | no | accepts further named parameters | Receives the notifications of the ECPay provider, verifies their message digest, applies the outcome and answers with the string "1/Allowed". | Payment Provider: ECPay |
| `/payment/flutterwave/return` | `flutterwave_return_from_checkout` | page or file | public or signed in | GET | no | accepts further named parameters | Receives the payer back from the hosted page of the Flutterwave provider and applies the outcome. | Payment Provider: Flutterwave |
| `/payment/flutterwave/auth_return` | `flutterwave_return_from_authorization` | page or file | public or signed in | GET | no | `response` | Receives the merchant back from the account authorization of the Flutterwave provider and stores the returned credentials on the provider record. | Payment Provider: Flutterwave |
| `/payment/flutterwave/webhook` | `flutterwave_webhook` | page or file | public or signed in | POST | no | none | Receives the notifications of the Flutterwave provider, verifies their signature header, applies the outcome and answers with an empty body. | Payment Provider: Flutterwave |
| `/payment/iyzico/return` | `iyzico_return_from_payment` | page or file | public or signed in | POST | no | `tx_reference` (accepts further named parameters) | Receives the payer back from the hosted page of the Iyzico provider, finds the transaction by the returned reference and applies the outcome, without opening a session. | Payment Provider: Iyzico |
| `/payment/iyzico/webhook` | `iyzico_webhook` | page or file | public or signed in | POST | no | none | Receives the notifications of the Iyzico provider, verifies them by querying the provider again, and answers with an empty response. | Payment Provider: Iyzico |
| `/payment/mercado_pago/payments` | `mercado_pago_payment` | remote call | public or signed in | POST | no | `reference`, `transaction_amount`, `token`, `installments`, `payment_method_brand`, `issuer` | Sends the payment request of the Mercado Pago provider with the card token, the amount in minor units, the number of instalments, the card brand and the issuer, and applies the outcome. | Payment Provider: Mercado Pago |
| `/payment/mercado_pago/return` | `mercado_pago_return_from_checkout` | page or file | public or signed in | GET | no | accepts further named parameters | Receives the payer back from the hosted page of the Mercado Pago provider and applies the outcome. | Payment Provider: Mercado Pago |
| `/payment/mercado_pago/webhook/<reference:path>` | `mercado_pago_webhook` | page or file | public or signed in | POST | no | `reference` (accepts further named parameters) | Receives the notifications of the Mercado Pago provider for the transaction whose reference is part of the address, verifies them by querying the provider again, applies the outcome and answers with an empty body. | Payment Provider: Mercado Pago |
| `/payment/mercado_pago/oauth/return` | `mercado_pago_return_from_authorization` | page or file | signed in | GET | yes | accepts further named parameters | Receives the merchant back from the account authorization of the Mercado Pago provider, exchanges the authorization code for credentials through the publisher proxy, verifies the anti-forgery token and returns to the provider form. | Payment Provider: Mercado Pago |
| `/payment/mollie/return` | `mollie_return_from_checkout` | page or file | public or signed in | GET, POST | no | accepts further named parameters | Receives the payer back from the hosted page of the Mollie provider and applies the outcome, without opening a session. | Payment Provider: Mollie |
| `/payment/mollie/webhook` | `mollie_webhook` | page or file | public or signed in | POST | no | accepts further named parameters | Receives the notifications of the Mollie provider, which carry only a payment reference, fetches the payment from the provider, applies the outcome and answers with an empty body. | Payment Provider: Mollie |
| `/payment/nuvei/return` | `nuvei_return_from_checkout` | page or file | public or signed in | GET | no | `tx_reference`, `error_access_token` (accepts further named parameters) | Receives the payer back from the hosted page of the Nuvei provider, applies the outcome, and, for a cancelled or failed payment, verifies the supplied access token before marking the transaction. | Payment Provider: Nuvei |
| `/payment/nuvei/webhook` | `nuvei_webhook` | page or file | public or signed in | POST | no | accepts further named parameters | Receives the notifications of the Nuvei provider, verifies their message digest, applies the outcome and answers with the string "Allowed". | Payment Provider: Nuvei |
| `/payment/paymob/return` | `paymob_return_from_checkout` | page or file | public or signed in | GET | no | accepts further named parameters | Receives the payer back from the hosted page of the Paymob provider and applies the outcome. | Payment Provider: Paymob |
| `/payment/paymob/webhook` | `paymob_webhook` | page or file | public or signed in | POST | no | accepts further named parameters | Receives the notifications of the Paymob provider, verifies their message digest, applies the outcome and answers with an empty body. | Payment Provider: Paymob |
| `/payment/paypal/complete_order` | `paypal_complete_order` | remote call | public or signed in | POST | no | `order`, `reference` | Captures an authorized PayPal order and applies the outcome, using the transaction reference to build the repeat-safe key that prevents a double capture. | Payment Provider: Paypal |
| `/payment/paypal/webhook/` | `paypal_webhook` | page or file | public or signed in | POST | no | none | Receives the notifications of the PayPal provider, verifies them with the provider verification call, applies the outcome and answers with an empty body. | Payment Provider: Paypal |
| `/payment/payu/return` | `payu_return_from_checkout` | page or file | public or signed in | POST | no | accepts further named parameters | Receives the payer back from the hosted page of the PayU provider and applies the outcome, without opening a session. | Payment Provider: PayU |
| `/payment/payu/webhook` | `payu_webhook` | page or file | public or signed in | POST | no | accepts further named parameters | Receives the notifications of the PayU provider, verifies their signature, applies the outcome and answers with an empty response. | Payment Provider: PayU |
| `/payment/payu/oauth/return` | `payu_return_from_authorization` | page or file | signed in | GET | yes | accepts further named parameters | Receives the merchant back from the account authorization of the PayU provider, verifies the anti-forgery token, refuses with the forbidden status when it does not match, and stores the returned credentials. | Payment Provider: PayU |
| `/payment/razorpay/return` | `razorpay_return_from_checkout` | page or file | public or signed in | POST | no | `reference` (accepts further named parameters) | Receives the payer back from the hosted page of the Razorpay provider, finds the transaction by reference and applies the outcome, without opening a session. | Payment Provider: Razorpay |
| `/payment/razorpay/webhook` | `razorpay_webhook` | page or file | public or signed in | POST | no | none | Receives the notifications of the Razorpay provider, verifies their signature, applies the outcome and answers with an empty body. | Payment Provider: Razorpay |
| `/payment/razorpay/oauth/return` | `razorpay_return_from_authorization` | page or file | signed in | GET | yes | accepts further named parameters | Receives the merchant back from the account authorization of the Razorpay provider, exchanges the authorization code for credentials through the publisher proxy and returns to the provider form. | Payment Provider: Razorpay |
| `/payment/redsys/return` | `redsys_return_from_checkout` | page or file | public or signed in | GET | no | accepts further named parameters | Receives the payer back from the hosted page of the Redsys provider with the encoded payment data and applies the outcome. | Payment Provider: Redsys |
| `/payment/redsys/webhook` | `redsys_webhook` | page or file | public or signed in | POST | no | accepts further named parameters | Receives the encoded notifications of the Redsys provider, verifies their signature, applies the outcome and answers with the string "Allowed". | Payment Provider: Redsys |
| `/payment/stripe/return` | `stripe_return` | page or file | public or signed in | GET | no | accepts further named parameters | Receives the payer back from the Stripe provider, whether the payment was taken inline, on the hosted page or after strong authentication, and applies the outcome. | Payment Provider: Stripe |
| `/payment/stripe/webhook` | `stripe_webhook` | page or file | public or signed in | POST | no | none | Receives the notifications of the Stripe provider, verifies their signature header against the stored signing secret, applies the outcome and answers with an empty body. | Payment Provider: Stripe |
| `/payment/stripe/onboarding/return` | `stripe_return_from_onboarding` | page or file | signed in | GET | no | `provider`, `menu` | Receives the merchant back from the account enrolment of the Stripe provider and returns to the provider form with the refreshed account state. | Payment Provider: Stripe |
| `/payment/stripe/onboarding/refresh` | `stripe_refresh_onboarding` | page or file | signed in | GET | no | `provider`, `account`, `menu` | Issues a new enrolment link when the merchant returns with an expired one and redirects to it. | Payment Provider: Stripe |
| `/.well-known/apple-developer-merchantid-domain-association` | `stripe_apple_pay_get_domain_association_file` | page or file | public or signed in | GET, POST | no | none | Publishes the domain ownership file that the wallet provider and the payment provider both read to authorize the wallet button on this site. | Payment Provider: Stripe |
| `/payment/toss-payments/success` | `toss_payments_success_return` | page or file | public or signed in | GET | no | accepts further named parameters | Receives the payer back after a successful payment at the Toss Payments provider, with the order reference, the payment key and the amount, and confirms the payment with the provider before applying the outcome. | Payment Provider: Toss Payments |
| `/payment/toss-payments/failure` | `toss_payments_failure_return` | page or file | public or signed in | GET | no | accepts further named parameters | Receives the payer back after a failed payment at the Toss Payments provider and applies the failure, verified by the access token because the failure return carries no payment key. | Payment Provider: Toss Payments |
| `/payment/toss-payments/webhook` | `toss_payments_webhook` | page or file | public or signed in | POST | no | none | Receives the notifications of the Toss Payments provider, applies the outcome and answers with an empty body. | Payment Provider: Toss Payments |
| `/payment/worldline/return` | `worldline_return_from_checkout` | page or file | public or signed in | GET | no | accepts further named parameters | Receives the payer back from the hosted page of the Worldline provider, identified by the provider reference appended to the return address, and applies the outcome. | Payment Provider: Worldline |
| `/payment/worldline/webhook` | `worldline_webhook` | page or file | public or signed in | POST | no | none | Receives the notifications of the Worldline provider, verifies their signature, applies the outcome and answers with an empty body. | Payment Provider: Worldline |
| `/payment/xendit/payment` | `xendit_payment` | remote call | public or signed in | POST | no | `reference`, `token_reference`, `access_token`, `authentication` | Sends the payment request of the Xendit provider using a saved card token, verified by the access token of the transaction, and applies the outcome, including the strong authentication result when one was required. | Payment Provider: Xendit |
| `/payment/xendit/return` | `xendit_return` | page or file | public or signed in | GET | no | `tx_reference`, `success`, `access_token` (accepts further named parameters) | Receives the payer back from the hosted page of the Xendit provider and moves the transaction to pending until the notification confirms it. | Payment Provider: Xendit |
| `/payment/xendit/webhook` | `xendit_webhook` | page or file | public or signed in | POST | no | none | Receives the notifications of the Xendit provider, verifies their token header, applies the outcome and answers with the string "accepted". | Payment Provider: Xendit |

## Messaging and Collaboration

This is the largest endpoint family of the system. It covers the discussion threads attached to every document, the
channels of the messaging application, the live conversation service embedded on public sites, the scripted conversation
assistant, the mailing lists, the electronic mail client plug-in, the customer rating links and the delivery reports of
the text message gateway. Two access patterns recur. First, the visitor of a public page is identified by a guest record
and a guest token rather than by a user; endpoints whose path contains the cross-origin segment take that token as an
explicit parameter, because a conversation widget embedded on a foreign site cannot send cookies. Second, every call that
reads or writes a thread accepts a fetch specification that names which related sets to return (messages, followers,
attachments, reactions), therefore one call can fill a whole panel.

### Discussion data services

Two general services serve the messaging application: one that only reads and one that may write. Both take a fetch
specification naming the data sets to return and a display context.

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/mail/data` | `mail_data` | remote call | public or signed in | POST | no | `fetch_parameters`, `context` | Returns the requested messaging data sets (the initial state of the application, a channel, a thread, the settings of the user, the list of channels) without changing anything. | Discuss |
| `/mail/action` | `mail_action` | remote call | public or signed in | POST | no | `fetch_parameters`, `context` | Performs the requested messaging actions (marking as read, pinning, joining, leaving) and returns the same kind of data sets as the read-only service. | Discuss |
| `/discuss/search` | `search` | remote call | public or signed in | POST | no | `term`, `category`, `limit` | Searches across channels, contacts and messages for the typed text, limited to the requested categories and count, for the quick search of the messaging application. | Discuss |
| `/mail/inbox/messages` | `discuss_inbox_messages` | remote call | signed in | POST | no | `fetch_parameters` | Returns a page of the messages addressed to the user that are still waiting in the inbox. | Discuss |
| `/mail/starred/messages` | `discuss_starred_messages` | remote call | signed in | POST | no | `fetch_parameters` | Returns a page of the messages the user marked with a star. | Discuss |
| `/mail/history/messages` | `discuss_history_messages` | remote call | signed in | POST | no | `fetch_parameters` | Returns a page of the messages the user has already handled. | Discuss |
| `/mail/thread/messages` | `mail_thread_messages` | remote call | signed in | POST | no | `thread_model`, `thread`, `fetch_parameters` | Returns a page of the messages of the discussion thread of one document. | Discuss |
| `/discuss/channel/messages` | `discuss_channel_messages` | remote call | public or signed in | POST | no | `channel`, `fetch_parameters` | Returns a page of the messages of one channel with the related data named in the fetch specification. | Discuss |
| `/mail/set_manual_im_status` | `set_manual_instant_messaging_status` | remote call | signed in | POST | no | `status` | Sets the presence state the user chooses to show (available, away, busy or invisible), which overrides the state derived from activity. | Discuss |
| `/websocket/update_bus_presence` | `update_bus_presence` | remote call | public or signed in | POST | no | `inactivity_period` | Reports the inactivity period of the user, therefore the presence state can be recomputed, for clients that maintain the long-lived connection themselves. | Discuss |

### Posting, editing and reacting to messages

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/mail/message/post` | `mail_message_post` | remote call | public or signed in | POST | no | `thread_model`, `thread`, `post_data`, `context` (accepts further named parameters) | Posts a message on the discussion thread of any document or channel with its body, attachments, recipients, message type and subtype, and returns the stored message; the storefront package adds the order context, the live conversation package adds the guest identity, and the automated assistant package adds its own author identity. | Discuss + Electronic Commerce + Live Chat + the system robot |
| `/mail/message/update_content` | `mail_message_update_content` | remote call | public or signed in | POST | no | `message`, `update_data` (accepts further named parameters) | Replaces the body and the attachment set of an existing message, keeping its identity and its place in the thread. | Discuss |
| `/mail/message/reaction` | `mail_message_reaction` | remote call | public or signed in | POST | no | `message`, `content`, `action` (accepts further named parameters) | Adds or removes one reaction symbol of the current identity on a message. | Discuss |
| `/mail/message/translate` | `translate` | remote call | signed in | POST | no | `message` | Returns the translation of one message into the language of the reader, produced by the external translation service and cached on the message. | Discuss |
| `/mail/message/<message_id:integer>` | `mail_thread_message_redirect` | page or file | public or signed in | GET, POST | no | `message` (accepts further named parameters) | Redirects a message link to the page that shows the message in its context: the channel, the document page or the portal page. | Discuss |
| `/mail/view` | `mail_action_view` | page or file | public or signed in | GET, POST | no | `model_name`, `related_record_identifier`, `access_token` (accepts further named parameters) | Resolves the document link of a notification message: it sends a visitor holding a public address to that address, a user with read access to the document, and a user without read access to the messaging application with the message selected. | Discuss |
| `/mail/link_preview` | `mail_link_preview` | remote call | public or signed in | POST | no | `message` | Builds and returns the preview cards of the addresses contained in a message body. | Discuss |
| `/mail/link_preview/hide` | `mail_link_preview_hide` | remote call | public or signed in | POST | no | `message_link_preview` | Hides one preview card on one message for everyone. | Discuss |
| `/mail/partner/from_email` | `mail_thread_partner_from_email` | remote call | signed in | POST | no | `thread_model`, `thread`, `emails` | Finds or creates the contacts matching a list of electronic mail addresses, therefore recipients typed by hand become real contacts. | Discuss |
| `/mail/thread/recipients` | `mail_thread_recipients` | remote call | signed in | POST | no | `thread_model`, `thread`, `message` | Returns the recipients suggested for a reply on a thread, creating the missing contacts on the fly. | Discuss |
| `/mail/thread/recipients/get_suggested_recipients` | `mail_thread_recipients_get_suggested_recipients` | remote call | signed in | POST | no | `thread_model`, `thread`, `partner`, `main_email` | Returns the suggested recipients of a document, taking into account the recipients the user has already added and the main address of the document. | Discuss |
| `/mail/thread/recipients/fields` | `mail_thread_recipients_fields` | remote call | signed in | POST | no | `thread_model` | Returns the fields of an entity that hold electronic mail addresses, which drives the recipient suggestions. | Discuss |
| `/mail/thread/subscribe` | `mail_thread_subscribe` | remote call | signed in | POST | no | `related_record_model`, `related_record_identifier`, `partner` | Adds contacts as followers of a document. | Discuss |
| `/mail/thread/unsubscribe` | `mail_thread_unsubscribe` | remote call | signed in | POST | no | `related_record_model`, `related_record_identifier`, `partner` | Removes contacts from the followers of a document. | Discuss |
| `/mail/read_subscription_data` | `read_subscription_data` | remote call | signed in | POST | no | `follower` | Returns the notification categories available on a document and which of them one follower has subscribed to. | Discuss |
| `/website_mail/follow` | `website_message_subscribe` | remote call | public or signed in | POST | yes | `identifier`, `object`, `message_is_follower`, `email` (accepts further named parameters) | Subscribes a visitor, by electronic mail address, to the updates of a published record, and returns the new follower state. | Website Mail |
| `/website_mail/is_follower` | `is_follower` | remote call | public or signed in | POST | yes | `records` (accepts further named parameters) | Reports, for a set of records given by entity and reference, which ones the current visitor already follows, together with the display data of the follow button. | Website Mail |

### Attachments in threads

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/mail/attachment/upload` | `mail_attachment_upload` | page or file | public or signed in | POST | no | `ufile`, `thread`, `thread_model`, `is_pending` (accepts further named parameters) | Stores an uploaded file as an attachment of a thread, marked as pending until the message that carries it is posted; the cloud storage package diverts large files to the external store, and the live conversation package accepts the guest identity instead of a session. | Discuss + Cloud Storage + Live Chat |
| `/mail/attachment/delete` | `mail_attachment_delete` | remote call | public or signed in | POST | no | `attachment`, `access_token` | Deletes an attachment of a thread, authorized by access rights or by the access token of the attachment. | Discuss |
| `/mail/attachment/zip` | `mail_attachment_get_zip` | page or file | public or signed in | POST | no | `file`, `zip_name` (accepts further named parameters) | Streams the chosen attachments as one compressed archive under the requested name. | Discuss |
| `/mail/attachment/pdf_first_page/<attachment_id:integer>` | `mail_attachment_portable_document_first_page` | page or file | public or signed in | GET | no | `attachment`, `access_token` | Streams only the first page of a printable document attachment, for the preview card. | Discuss |
| `/mail/attachment/update_thumbnail` | `mail_attachement_update_thumbnail` | remote call | public or signed in | POST | no | `attachment`, `thumbnail`, `access_token` | Stores the preview image the client generated for an attachment. | Discuss |

### Channels

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/discuss/channel/<channel_id:integer>` | `discuss_channel` | page or file | public or signed in | GET | no | `channel` | Serves the standalone page of one channel, used when a channel link is opened outside the messaging application. | Discuss |
| `/discuss/channel/join` | `discuss_channel_join` | remote call | public or signed in | POST | no | `channel` | Adds the current identity as a member of a channel that allows self-joining. | Discuss |
| `/discuss/channel/members` | `discuss_channel_members` | remote call | public or signed in | POST | no | `channel`, `known_member` | Returns a page of the members of a channel, skipping the ones the client already knows. | Discuss |
| `/discuss/channel/mark_as_read` | `discuss_channel_mark_as_read` | remote call | public or signed in | POST | no | `channel`, `last_message` | Moves the read marker of the current member to the given message. | Discuss |
| `/discuss/channel/set_new_message_separator` | `discuss_channel_set_new_message_separator` | remote call | public or signed in | POST | no | `channel`, `message` | Places the unread separator line of the current member at a chosen message. | Discuss |
| `/discuss/channel/notify_typing` | `discuss_channel_notify_typing` | remote call | public or signed in | POST | no | `channel`, `is_typing` | Tells the other members that the current identity started or stopped typing. | Discuss |
| `/discuss/channel/ping` | `channel_ping` | remote call | public or signed in | POST | no | `channel`, `rtc_session`, `check_rtc_session` | Refreshes the presence of the current member in a channel and, when a live call session is given, confirms that the session is still alive. | Discuss |
| `/discuss/channel/pinned_messages` | `discuss_channel_pins` | remote call | public or signed in | POST | no | `channel` | Returns the pinned messages of a channel. | Discuss |
| `/discuss/channel/attachments` | `load_attachments` | remote call | public or signed in | POST | no | `channel`, `limit`, `before` | Returns a page of the attachments of a channel, older than a given attachment when one is supplied. | Discuss |
| `/discuss/channel/update_avatar` | `discuss_channel_avatar_update` | remote call | signed in | POST | no | `channel`, `data` | Replaces the picture of a channel with the uploaded image. | Discuss |
| `/discuss/channel/sub_channel/create` | `discuss_channel_sub_channel_create` | remote call | public or signed in | POST | no | `parent_channel`, `from_message`, `name` | Creates a thread inside a channel, optionally anchored at the message it answers, under the given name. | Discuss |
| `/discuss/channel/sub_channel/fetch` | `discuss_channel_sub_channel_fetch` | remote call | public or signed in | POST | no | `parent_channel`, `search_term`, `before`, `limit` | Returns a page of the threads of a channel, filtered by the typed text and older than a given thread. | Discuss |
| `/discuss/channel/sub_channel/delete` | `discuss_delete_sub_channel` | remote call | signed in | POST | no | `sub_channel` | Deletes a thread of a channel. | Discuss |
| `/discuss/settings/mute` | `discuss_mute` | remote call | signed in | POST | no | `minutes`, `channel` | Silences the notifications of one channel, or of every channel when no channel is given, for the requested number of minutes, where minus one means until the user cancels it. | Discuss |
| `/discuss/settings/custom_notifications` | `discuss_custom_notifications` | remote call | signed in | POST | no | `custom_notifications`, `channel` | Sets the notification rule of one channel, or the personal default when no channel is given, to all messages, mentions only or no notification. | Discuss |
| `/mail/guest/update_name` | `mail_guest_update_name` | remote call | public or signed in | POST | no | `guest`, `name` | Changes the display name a guest chose for itself in a conversation. | Discuss |
| `/chat/<create_token:text>` <br> `/chat/<create_token:text>/<channel_name:text>` | `discuss_channel_chat_from_token` | page or file | public or signed in | GET | no | `create_token`, `channel_name` | Opens, or creates on first use, the conversation attached to a shared link token, optionally naming it, and adds the visitor as a member. | Discuss |
| `/meet/<create_token:text>` <br> `/meet/<create_token:text>/<channel_name:text>` | `discuss_channel_meet_from_token` | page or file | public or signed in | GET | no | `create_token`, `channel_name` | Opens, or creates on first use, the meeting room attached to a shared link token and joins its live call. | Discuss |
| `/chat/<channel_id:integer>/<invitation_token:text>` | `discuss_channel_invitation` | page or file | public or signed in | GET | no | `channel`, `invitation_token`, `email_token` | Accepts an invitation to a channel: it verifies the invitation token, adds the visitor as a member (matching the invited address when the link carries one) and opens the channel. | Discuss |

### Live calls inside channels

A live call is a set of sessions, one per participant device, exchanging their connection data through the server. All
these calls require membership of the channel.

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/mail/rtc/channel/join_call` | `channel_call_join` | remote call | public or signed in | POST | no | `channel`, `check_rtc_session`, `camera` | Creates a call session for the current member of a channel, with or without camera, and returns the session data of all participants; the live conversation package allows a guest identity to join. | Discuss + Live Chat |
| `/mail/rtc/channel/leave_call` | `channel_call_leave` | remote call | public or signed in | POST | no | `channel`, `session` | Ends the call session of the current member and clears any pending invitation to that call. | Discuss |
| `/mail/rtc/channel/cancel_call_invitation` | `channel_call_cancel_invitation` | remote call | public or signed in | POST | no | `channel`, `member` | Cancels the pending call invitations of the named members, or of all invited members when none is named. | Discuss |
| `/mail/rtc/channel/upgrade_connection` | `channel_upgrade` | remote call | signed in | POST | no | `channel` | Switches the call of a channel from direct connections between participants to a relayed connection when the direct mode fails. | Discuss |
| `/mail/rtc/session/update_and_broadcast` | `session_update_and_broadcast` | remote call | public or signed in | POST | no | `session`, `values` | Updates the state of the own call session (microphone muted, camera on, screen shared, deafened) and broadcasts it to the other participants. | Discuss |
| `/mail/rtc/session/notify_call_members` | `session_call_notify` | remote call | public or signed in | POST | no | `peer_notifications` | Forwards connection negotiation messages from the own session to the named participant sessions. | Discuss |
| `/mail/rtc/audio_worklet_processor_v2` | `audio_worklet_processor` | page or file | public or signed in | GET | no | none | Returns the sound processing script the client must load as a separate file, because it runs in its own processing context. | Discuss |
| `/discuss/voice/worklet_processor` | `voice_worklet_processor` | page or file | public or signed in | GET | no | none | Returns the voice recording processing script, loaded as a separate file for the same reason. | Discuss |

### Animated image search

The message composer can insert an animated image from an external library. The platform proxies the calls, therefore the
library credentials stay on the server.

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/discuss/gif/search` | `search` | remote call | signed in | POST | no | `search_term`, `locale`, `country`, `position`, `readonly` | Returns a page of animated images matching the typed text for the given language and country, from the external image library. | Discuss |
| `/discuss/gif/categories` | `categories` | remote call | signed in | POST | no | `locale`, `country` | Returns the categories offered by the external image library for the given language and country. | Discuss |
| `/discuss/gif/favorites` | `get_favorites` | remote call | signed in | POST | no | `offset` | Returns a page of the animated images the user marked as favourite. | Discuss |
| `/discuss/gif/add_favorite` | `add_favorite` | remote call | signed in | POST | no | `tenor_gif` | Marks one animated image of the external library as a favourite of the user. | Discuss |
| `/discuss/gif/remove_favorite` | `remove_favorite` | remote call | signed in | POST | no | `tenor_gif` | Removes one animated image from the favourites of the user. | Discuss |

### Live conversation service, same-origin calls

These endpoints serve the conversation widget when it runs on a page of this platform. Their cross-origin twins are in the
next group.

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/im_livechat/loader/<channel_id:integer>` | `loader` | page or file | public or signed in | GET, POST | no | `channel` (accepts further named parameters) | Returns the loading script and the configuration of the conversation widget for one conversation channel, including the greeting, the button text and whether an operator is available. | Live Chat |
| `/im_livechat/support/<channel_id:integer>` | `support_page` | page or file | public or signed in | GET, POST | no | `channel` (accepts further named parameters) | Serves the standalone conversation page of one channel, for visitors who open the conversation in its own window. | Live Chat |
| `/im_livechat/get_session` | `get_session` | remote call | public or signed in | POST | no | `channel`, `previous_operator`, `chatbot_script`, `persisted` (accepts further named parameters) | Starts a conversation: it picks an available operator (preferring the previous one when asked), or starts the scripted assistant when one is configured, creates the conversation channel when the visitor persists it, and returns the channel data. | Live Chat |
| `/im_livechat/visitor_leave_session` | `visitor_leave_session` | remote call | public or signed in | POST | no | `channel` | Records that the visitor left the conversation, which clears the pending request and tells the operator that the conversation is over, therefore a new request may be sent later. | Live Chat |
| `/im_livechat/feedback` | `feedback` | remote call | public or signed in | POST | no | `channel`, `rate`, `reason` (accepts further named parameters) | Stores the rating and the written reason the visitor gave at the end of a conversation. | Live Chat |
| `/im_livechat/history` | `history_pages` | remote call | public or signed in | POST | no | `pid`, `channel`, `page_history` | Stores the list of pages the visitor browsed during the conversation, which the operator sees beside the conversation. | Live Chat |
| `/im_livechat/download_transcript/<channel_id:integer>` | `download_livechat_transcript` | page or file | public or signed in | GET, POST | no | `channel` | Streams the transcript of one conversation as a printable document. | Live Chat |
| `/im_livechat/email_livechat_transcript` | `email_livechat_transcript` | remote call | signed in | POST | no | `channel`, `email` | Sends the transcript of one conversation to the address the visitor typed. | Live Chat |
| `/im_livechat/session/update_note` | `livechat_session_update_note` | remote call | signed in | POST | no | `channel`, `notes` | Stores the internal note an operator writes about a conversation; only internal users who may read the conversation may call it. | Live Chat |
| `/im_livechat/session/update_status` | `livechat_session_update_status` | remote call | signed in | POST | no | `channel`, `livechat_status` | Sets the handling state of a conversation (in progress, waiting or looking for help); only internal users who may read the conversation may call it. | Live Chat |
| `/im_livechat/conversation/update_tags` | `livechat_conversation_update_tags` | remote call | signed in | POST | no | `channel`, `tags`, `method` | Adds or removes classification tags on a conversation. | Live Chat |
| `/im_livechat/conversation/write_expertises` | `livechat_conversation_write_expertises` | remote call | signed in | POST | no | `channel`, `orm_commands` | Sets the areas of expertise recorded on a conversation from the change list sent by the client. | Live Chat |
| `/im_livechat/conversation/create_and_link_expertise` | `livechat_conversation_create_and_link_expertise` | remote call | signed in | POST | no | `channel`, `expertise_name` | Creates an area of expertise under the typed name and links it to the conversation. | Live Chat |
| `/im_livechat/assets_embed.<ext:one of css or js>` | `assets_embed` | page or file | public or signed in | GET, POST | no | `ext` (accepts further named parameters) | Returns the style sheet or the script of the conversation widget for embedding in a foreign page. | Live Chat |
| `/im_livechat/external_lib.<ext:one of css or js>` | `external_lib` | page or file | public or signed in | GET, POST | no | `ext` (accepts further named parameters) | Returns the widget script under the address used by earlier embeddings; only the script is served, because the style sheet is fetched by the isolated rendering tree of the widget to avoid clashing with the host page. | Live Chat |
| `/im_livechat/emoji_bundle` | `get_emoji_bundle` | page or file | public or signed in | GET, POST | no | none | Returns the symbol set used by the conversation widget. | Live Chat |
| `/im_livechat/font-awesome` | `fontawesome` | page or file | none | GET, POST | no | accepts further named parameters | Returns the icon font used by the conversation widget. | Live Chat |
| `/im_livechat/system_ui_icons` | `system_user_interface_icons` | page or file | none | GET, POST | no | accepts further named parameters | Returns the icon set of the platform used by the conversation widget. | Live Chat |
| `/web/tests/livechat` | `test_external_livechat` | page or file | signed in | GET, POST | no | accepts further named parameters | Serves the page that exercises the conversation widget as if it were embedded on a foreign site. | Live Chat |

### Live conversation service, cross-origin calls

Each endpoint of this group mirrors one same-origin endpoint. The difference is the identity: the guest token is passed
as a parameter instead of a cookie, the responses carry the cross-origin headers, and cross-site submission protection is
disabled, because the caller is a foreign page by design.

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/im_livechat/cors/get_session` | `cors_get_session` | remote call | public or signed in | POST | no | `channel`, `previous_operator`, `chatbot_script`, `persisted` (accepts further named parameters) | Starts a conversation from a foreign page and returns the channel data together with the guest token to use for the following calls. | Live Chat |
| `/im_livechat/cors/init` | `cors_livechat_init` | remote call | public or signed in | POST | no | `channel`, `guest_token` | Returns the initial state of a conversation for a foreign page, for the given channel and guest token. | Live Chat |
| `/im_livechat/cors/data` | `livechat_data` | remote call | public or signed in | POST | no | `guest_token` (accepts further named parameters) | Returns the messaging data sets a foreign page needs, for the given guest token. | Live Chat |
| `/im_livechat/cors/action` | `livechat_action` | remote call | public or signed in | POST | no | `guest_token` (accepts further named parameters) | Performs the messaging actions requested by a foreign page, for the given guest token. | Live Chat |
| `/im_livechat/cors/channel/messages` | `livechat_channel_messages` | remote call | public or signed in | POST | no | `guest_token`, `channel`, `fetch_parameters` | Returns a page of messages of the conversation to a foreign page. | Live Chat |
| `/im_livechat/cors/channel/mark_as_read` | `livechat_channel_mark_as_read` | remote call | public or signed in | POST | no | `guest_token` (accepts further named parameters) | Moves the read marker of the guest to the latest message. | Live Chat |
| `/im_livechat/cors/channel/notify_typing` | `livechat_channel_notify_typing` | remote call | public or signed in | POST | no | `guest_token`, `channel`, `is_typing` | Tells the operator that the guest started or stopped typing. | Live Chat |
| `/im_livechat/cors/channel/ping` | `livechat_channel_ping` | remote call | public or signed in | POST | no | `guest_token`, `channel`, `rtc_session`, `check_rtc_session` | Refreshes the presence of the guest in the conversation and confirms its live call session when one is given. | Live Chat |
| `/im_livechat/cors/message/post` | `livechat_message_post` | remote call | public or signed in | POST | no | `guest_token`, `thread_model`, `thread`, `post_data`, `context` (accepts further named parameters) | Posts a message of the guest in the conversation. | Live Chat |
| `/im_livechat/cors/message/update_content` | `livechat_message_update_content` | remote call | public or signed in | POST | no | `guest_token`, `message`, `update_data` (accepts further named parameters) | Replaces the body and attachments of a message the guest posted. | Live Chat |
| `/im_livechat/cors/message/reaction` | `livechat_message_reaction` | remote call | public or signed in | POST | no | `guest_token`, `message`, `content`, `action` (accepts further named parameters) | Adds or removes a reaction of the guest on a message. | Live Chat |
| `/im_livechat/cors/link_preview` | `livechat_link_preview` | remote call | public or signed in | POST | no | `guest_token`, `message` | Builds the preview cards of the addresses in a message of the conversation. | Live Chat |
| `/im_livechat/cors/link_preview/hide` | `livechat_link_preview_hide` | remote call | public or signed in | POST | no | `guest_token`, `message_link_preview` | Hides one preview card in the conversation. | Live Chat |
| `/im_livechat/cors/attachment/upload` | `instant_messaging_livechat_attachment_upload` | page or file | public or signed in | GET, POST | no | `guest_token`, `ufile`, `thread`, `thread_model`, `is_pending` (accepts further named parameters) | Stores a file the guest uploads into the conversation, pending until the message is posted. | Live Chat |
| `/im_livechat/cors/attachment/delete` | `instant_messaging_livechat_attachment_delete` | remote call | public or signed in | POST | no | `guest_token`, `attachment`, `access_token` | Deletes an attachment the guest uploaded, authorized by its access token. | Live Chat |
| `/im_livechat/cors/feedback` | `cors_feedback` | remote call | public or signed in | POST | no | `guest_token`, `channel`, `rate`, `reason` | Stores the rating and reason the guest gave at the end of the conversation. | Live Chat |
| `/im_livechat/cors/history` | `cors_history_pages` | remote call | public or signed in | POST | no | `guest_token`, `pid`, `channel`, `page_history` | Stores the browsing history of the guest during the conversation. | Live Chat |
| `/im_livechat/cors/visitor_leave_session` | `cors_visitor_leave_session` | remote call | public or signed in | POST | no | `guest_token`, `channel` | Records that the guest left the conversation. | Live Chat |
| `/im_livechat/cors/download_transcript/<channel_id:integer>` | `cors_download_livechat_transcript` | page or file | public or signed in | GET, POST | no | `guest_token`, `channel` | Streams the transcript of the conversation to the guest as a printable document. | Live Chat |
| `/im_livechat/cors/rtc/channel/join_call` | `livechat_channel_call_join` | remote call | public or signed in | POST | no | `guest_token`, `channel`, `check_rtc_session` | Lets the guest join the live call of the conversation. | Live Chat |
| `/im_livechat/cors/rtc/channel/leave_call` | `livechat_channel_call_leave` | remote call | public or signed in | POST | no | `guest_token`, `channel` | Ends the live call session of the guest. | Live Chat |
| `/im_livechat/cors/rtc/session/update_and_broadcast` | `livechat_session_update_and_broadcast` | remote call | public or signed in | POST | no | `guest_token`, `session`, `values` | Updates the call session state of the guest and broadcasts it to the other participants. | Live Chat |
| `/im_livechat/cors/rtc/session/notify_call_members` | `livechat_session_call_notify` | remote call | public or signed in | POST | no | `guest_token`, `peer_notifications` | Forwards connection negotiation messages from the guest session to the other sessions. | Live Chat |

### Scripted conversation assistant

A scripted assistant answers first, following a script of steps; each step is a question, an answer choice, a free text
request or a handover to a human operator.

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/chatbot/step/trigger` | `chatbot_trigger_step` | remote call | public or signed in | POST | no | `channel`, `chatbot_script`, `data` | Runs the next step of the script in a conversation and returns the message the assistant posts. | Live Chat |
| `/chatbot/answer/save` | `chatbot_save_answer` | remote call | public or signed in | POST | no | `channel`, `message`, `selected_answer` | Stores the answer the visitor selected for a step, which decides the next step. | Live Chat |
| `/chatbot/step/validate_email` | `chatbot_validate_email` | remote call | public or signed in | POST | no | `channel` | Checks that the text the visitor gave at an address step is a valid electronic mail address and stores it on the conversation. | Live Chat |
| `/chatbot/restart` | `chatbot_restart` | remote call | public or signed in | POST | no | `channel`, `chatbot_script` | Restarts the script of an assistant from its first step in the same conversation. | Live Chat |
| `/chatbot/cors/step/trigger` | `cors_chatbot_trigger_step` | remote call | public or signed in | POST | no | `guest_token`, `channel`, `chatbot_script`, `data` | Runs the next step of the script for a conversation embedded on a foreign page, identified by the guest token. | Live Chat |
| `/chatbot/cors/answer/save` | `cors_chatbot_save_answer` | remote call | public or signed in | POST | no | `guest_token`, `channel`, `message`, `selected_answer` | Stores the selected answer for a conversation embedded on a foreign page. | Live Chat |
| `/chatbot/cors/step/validate_email` | `cors_chatbot_validate_email` | remote call | public or signed in | POST | no | `guest_token`, `channel` | Validates and stores the address given at an address step for a conversation embedded on a foreign page. | Live Chat |
| `/chatbot/cors/restart` | `cors_chatbot_restart` | remote call | public or signed in | POST | no | `guest_token`, `channel`, `chatbot_script` | Restarts the script for a conversation embedded on a foreign page. | Live Chat |
| `/chatbot/<chatbot_script:record of Chatbot Script>/test` | `chatbot_test_script` | page or file | signed in | GET, POST | yes | `chatbot_script` | Opens a private conversation that runs one assistant script for testing, creating the channel that holds the exchange because no public conversation channel is attached. | Website Live Chat |

### Mailing list groups

A mailing list group receives messages by electronic mail, publishes them on a page and forwards them to its members.
Subscription changes made by an anonymous visitor are confirmed by a token sent to the address.

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/groups` | `groups_index` | page or file | public or signed in | GET, POST | yes | `email` (accepts further named parameters) | Lists the public mailing list groups with the subscription state of the given address and the subscribe and unsubscribe actions. | Mail Group |
| `/groups/<group:record of Mail Group>` <br> `/groups/<group:record of Mail Group>/page/<page:integer>` | `group_view_messages` | page or file | public or signed in | GET, POST | yes | `group`, `page`, `mode`, `date_begin`, `date_end` (accepts further named parameters) | Shows the messages of one group, in thread or flat mode, over an optional date range, with paging. | Mail Group |
| `/groups/<group:record of Mail Group>/<message:record of Mailing List Message>` | `group_view_message` | page or file | public or signed in | GET, POST | yes | `group`, `message`, `mode`, `date_begin`, `date_end` (accepts further named parameters) | Shows one message of a group with its answers. | Mail Group |
| `/groups/<group:record of Mail Group>/<message:record of Mailing List Message>/get_replies` | `group_message_get_replies` | remote call | public or signed in | POST | yes | `group`, `message`, `last_displayed` (accepts further named parameters) | Returns the further answers of one message, after the ones already displayed. | Mail Group |
| `/group/subscribe` | `group_subscribe` | remote call | public or signed in | POST | yes | `group`, `email`, `token` (accepts further named parameters) | Subscribes the signed-in user, or the given address, to a group; an anonymous visitor is sent a confirmation message carrying a token instead of being subscribed at once. | Mail Group |
| `/group/unsubscribe` | `group_unsubscribe` | remote call | public or signed in | POST | yes | `group`, `email`, `token` (accepts further named parameters) | Unsubscribes the signed-in user, or the given address, from a group, with the same confirmation rule for anonymous visitors. | Mail Group |
| `/group/subscribe-confirm` | `group_subscribe_confirm` | page or file | public or signed in | GET, POST | yes | `group`, `email`, `token` (accepts further named parameters) | Completes a subscription from the token sent by electronic mail. | Mail Group |
| `/group/unsubscribe-confirm` | `group_unsubscribe_confirm` | page or file | public or signed in | GET, POST | yes | `group`, `email`, `token` (accepts further named parameters) | Completes an unsubscription from the token sent by electronic mail. | Mail Group |
| `/group/<group_id:integer>/unsubscribe_oneclick` | `group_unsubscribe_oneclick` | page or file | public or signed in | POST | yes | `group`, `token`, `email` | Unsubscribes an address through the one-click unsubscribe header of the message; only submissions are accepted, which prevents mail scanners from unsubscribing people by following links. | Mail Group |
| `/group/is_member` | `group_is_member` | remote call | public or signed in | POST | yes | `group`, `email` (accepts further named parameters) | Returns the member address of a group when the given address is subscribed, and nothing otherwise. | Website Mail Group |

### Electronic mail client plug-in

The plug-in runs inside an external mail client. It obtains an application key once, through a consent page, and then
calls the platform with that key in the authorization header.

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/mail_client_extension/auth` <br> `/mail_plugin/auth` | `authentication` | page or file | signed in | GET | yes | accepts further named parameters | Shows the consent page telling the signed-in user that an external application asks for access, with the allow and deny actions. | Mail Plugin |
| `/mail_client_extension/auth/confirm` <br> `/mail_plugin/auth/confirm` | `authentication_confirm` | page or file | signed in | POST | no | `scope`, `friendlyname`, `redirect`, `info`, `do` (accepts further named parameters) | Records the consent, generates a short-lived authorization code for the requested scope and friendly name, and redirects back to the external application with that code. | Mail Plugin |
| `/mail_client_extension/auth/access_token` <br> `/mail_plugin/auth/access_token` | `authentication_access_token` | remote call | none | POST, OPTIONS | no | `authentication_code` (accepts further named parameters) | Exchanges the short-lived authorization code for the lasting application key the plug-in sends in the authorization header of every later call. | Mail Plugin |
| `/mail_plugin/auth/check_version` | `authentication_check_version` | remote call | none | POST, OPTIONS | no | none | Reports that the plug-in support is installed and which plug-in version it serves. | Mail Plugin |
| `/mail_plugin/get_translations` | `get_translations` | remote call | plug-in key | POST | no | none | Returns the translated texts of the plug-in interface. | Mail Plugin |
| `/mail_client_extension/partner/get` <br> `/mail_plugin/partner/get` | `res_partner_get` | remote call | plug-in key | POST | no | `email`, `name`, `partner` (accepts further named parameters) | Returns the contact matching a reference, or an address and a name; when no contact exists it answers with a placeholder reference of minus one, looks for a matching company, and creates an enriched company when none is found. | Mail Plugin |
| `/mail_client_extension/partner/create` <br> `/mail_plugin/partner/create` | `res_partner_create` | remote call | plug-in key | POST | no | `email`, `name`, `company` | Creates a contact with the given address, name and parent company from the plug-in. | Mail Plugin |
| `/mail_plugin/partner/search` | `res_partners_search` | remote call | plug-in key | POST | no | `search_term`, `limit` (accepts further named parameters) | Returns the contacts whose name, internal reference or electronic mail address matches the typed text, limited to the requested count. | Mail Plugin |
| `/mail_plugin/partner/enrich_and_create_company` | `res_partner_enrich_and_create_company` | remote call | plug-in key | POST | no | `partner` | Looks up the company of a contact in the enrichment service and creates the enriched company record when one is found. | Mail Plugin |
| `/mail_plugin/partner/enrich_and_update_company` | `res_partner_enrich_and_update_company` | remote call | plug-in key | POST | no | `partner` | Refreshes an existing company record with the data of the enrichment service. | Mail Plugin |
| `/mail_plugin/log_mail_content` | `log_mail_content` | remote call | plug-in key | POST | no | `model_name`, `related_record_identifier`, `message`, `attachments` | Posts the open message, with its attachments, in the discussion thread of the chosen record. | Mail Plugin |

### Customer ratings and delivery reports

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/rate/<token:text>/<rate:integer>` | `action_open_rating` | page or file | public or signed in | GET, POST | yes | `token`, `rate` (accepts further named parameters) | Records the rating a customer chose in a rating request message (the link carries the rating value) and shows the page that asks for an optional comment. | Customer Rating |
| `/rate/<token:text>/submit_feedback` | `action_submit_rating` | page or file | public or signed in | post, get | yes | `token`, `rate` (accepts further named parameters) | Stores the written comment, and a changed rating value when the customer changed it, on the rating record. | Customer Rating |
| `/sms/status` | `update_text_message_status` | remote call | public or signed in | POST | no | `message_statuses` | Receives a batch of delivery reports of the text message service: each entry carries a delivery state and the list of message references it applies to, and the matching messages are updated. | Text Message gateway |
| `/sms_twilio/status/<uuid:text>` | `update_text_message_status` | page or file | public or signed in | POST | no | `universally_unique_identifier`, `smsstatus`, `errorcode`, `errormessage` (accepts further named parameters) | Receives one delivery report of the alternative text message service for the message named in the address, with its state, error code and error message. | Twilio Text Message |
| `/web_editor/font_to_img/<icon>` <br> `/web_editor/font_to_img/<icon>/<color>` <br> `/web_editor/font_to_img/<icon>/<color>/<size:integer>` <br> `/web_editor/font_to_img/<icon>/<color>/<width:integer>x<height:integer>` <br> `/web_editor/font_to_img/<icon>/<color>/<size:integer>/<alpha:integer>` <br> `/web_editor/font_to_img/<icon>/<color>/<width:integer>x<height:integer>/<alpha:integer>` <br> `/web_editor/font_to_img/<icon>/<color>/<bg>` <br> `/web_editor/font_to_img/<icon>/<color>/<bg>/<size:integer>` <br> `/web_editor/font_to_img/<icon>/<color>/<bg>/<width:integer>x<height:integer>` <br> `/web_editor/font_to_img/<icon>/<color>/<bg>/<width:integer>x<height:integer>/<alpha:integer>` <br> `/mail/font_to_img/<icon>` <br> `/mail/font_to_img/<icon>/<color>` <br> `/mail/font_to_img/<icon>/<color>/<size:integer>` <br> `/mail/font_to_img/<icon>/<color>/<width:integer>x<height:integer>` <br> `/mail/font_to_img/<icon>/<color>/<size:integer>/<alpha:integer>` <br> `/mail/font_to_img/<icon>/<color>/<width:integer>x<height:integer>/<alpha:integer>` <br> `/mail/font_to_img/<icon>/<color>/<bg>` <br> `/mail/font_to_img/<icon>/<color>/<bg>/<size:integer>` <br> `/mail/font_to_img/<icon>/<color>/<bg>/<width:integer>x<height:integer>` <br> `/mail/font_to_img/<icon>/<color>/<bg>/<width:integer>x<height:integer>/<alpha:integer>` | `export_icon_to_png` | page or file | none | GET, POST | no | `icon`, `color_index`, `bg`, `size`, `alpha`, `font`, `width`, `height` | Renders one icon character of an icon font as an image in the requested colour, background colour, size and transparency, for use in messages, where icon fonts are not supported. | Discuss |

## Surveys, Certifications, Courses and Community Forum

Three public applications share this domain: questionnaires (used for surveys, quizzes, certifications and live sessions),
online courses with their contents and quizzes, and the question and answer forum with its reputation rules. Every
questionnaire page is addressed by two tokens: the questionnaire token identifies which questionnaire is served, and the
answer token identifies one participation, which is what allows an anonymous participant to resume where it stopped.
Forum actions are guarded by the reputation score of the author, which is why most forum endpoints answer with a refusal
page when the score is too low.

### Questionnaires and certifications

Participation tokens replace authentication on these pages: possession of the answer token is the proof of identity for
an anonymous participant.

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/survey/start/<survey_token:text>` | `survey_start` | page or file | public or signed in | GET, POST | yes | `survey_token`, `answer_token`, `email` (accepts further named parameters) | Starts a participation: it accepts an existing answer token to resume, or creates one when the questionnaire allows open access, records the electronic mail address when one is asked for, and redirects to the first page. | Surveys |
| `/survey/begin/<survey_token:text>/<answer_token:text>` | `survey_begin` | remote call | public or signed in | POST | yes | `survey_token`, `answer_token` (accepts further named parameters) | Opens the first page of an existing participation and returns its content together with an empty correct-answer map. | Surveys |
| `/survey/<survey_token:text>` <br> `/survey/<survey_token:text>/<answer_token:text>` | `survey_display_page` | page or file | public or signed in | GET, POST | yes | `survey_token`, `answer_token` (accepts further named parameters) | Serves the current page of a participation, or the landing page of the questionnaire when no participation is named. | Surveys |
| `/survey/submit/<survey_token:text>/<answer_token:text>` | `survey_submit` | remote call | public or signed in | POST | yes | `survey_token`, `answer_token` (accepts further named parameters) | Stores the answers of one page and returns the next page: answers that fail validation are returned with their messages instead, and when the time limit has passed the answers are ignored and the participation is closed. | Surveys |
| `/survey/next_question/<survey_token:text>/<answer_token:text>` | `survey_next_question` | remote call | public or signed in | POST | yes | `survey_token`, `answer_token` (accepts further named parameters) | Returns the question the host of a live session has moved to, which every participant screen requests when the host advances. | Surveys |
| `/survey/retry/<survey_token:text>/<answer_token:text>` | `survey_retry` | page or file | public or signed in | GET, POST | yes | `survey_token`, `answer_token` (accepts further named parameters) | Creates a new participation for a participant who failed and still has attempts left, and redirects to its first page. | Surveys |
| `/survey/print/<survey_token:text>` | `survey_print` | page or file | public or signed in | GET, POST | yes | `survey_token`, `review`, `answer_token` (accepts further named parameters) | Serves the whole questionnaire in one printable page, filled with the answers of a participation when an answer token is given, and in review mode when requested. | Surveys |
| `/survey/results/<survey:record of Survey>` | `survey_report` | page or file | signed in | GET, POST | yes | `survey`, `answer_token` (accepts further named parameters) | Serves the statistics page of a questionnaire: per question the answer counts and charts, and the overall participation and success figures. | Surveys |
| `/survey/test/<survey_token:text>` | `survey_test` | page or file | signed in | GET, POST | yes | `survey_token` (accepts further named parameters) | Creates a throw-away participation, therefore a manager can walk through a questionnaire without recording a real answer. | Surveys |
| `/survey/<survey_id:integer>/get_certification` | `survey_get_certification` | page or file | signed in | GET | yes | `survey` (accepts further named parameters) | Streams the certification document of a participant who passed the certification. | Surveys |
| `/survey/<survey:record of Survey>/certification_preview` | `show_certification_portable_document` | page or file | signed in | GET, POST | yes | `survey` (accepts further named parameters) | Streams a sample certification document filled with placeholder data, for checking the layout. | Surveys |
| `/survey/<survey:record of Survey>/get_certification_preview` | `survey_get_certification_preview` | page or file | signed in | GET | yes | `survey` (accepts further named parameters) | Returns the address of the sample certification document for the preview panel. | Surveys |
| `/survey/<survey_token:text>/get_background_image` | `survey_get_background` | page or file | public or signed in | GET, POST | yes | `survey_token` | Streams the background image of a questionnaire. | Surveys |
| `/survey/<survey_token:text>/<section_id:integer>/get_background_image` | `survey_section_get_background` | page or file | public or signed in | GET, POST | yes | `survey_token`, `section` | Streams the background image of one section of a questionnaire, which overrides the questionnaire background. | Surveys |
| `/survey/get_question_image/<survey_token:text>/<answer_token:text>/<question_id:integer>/<suggested_answer_id:integer>` | `survey_get_question_image` | page or file | public or signed in | GET, POST | yes | `survey_token`, `answer_token`, `question`, `suggested_answer` | Streams the image attached to one suggested answer of one question. | Surveys |
| `/s` | `survey_session_code` | page or file | public or signed in | GET, POST | yes | accepts further named parameters | Serves the page where a participant of a live session types the session code. | Surveys |
| `/s/<session_code:text>` | `survey_start_short` | page or file | public or signed in | GET, POST | yes | `session_code` | Resolves a short session code to the questionnaire and starts the participation, showing an error message when the code matches no open session. | Surveys |
| `/survey/check_session_code/<session_code:text>` | `survey_check_session_code` | remote call | public or signed in | POST | yes | `session_code` | Checks a typed session code and either redirects to the short session address or returns the error that invites the participant to type it again. | Surveys |
| `/survey/session/manage/<survey_token:text>` | `survey_session_manage` | page or file | signed in | GET, POST | yes | `survey_token` (accepts further named parameters) | Serves the host console of a live session: the start screen with the session options when the session is ready, and the current question with its controls once it is running; a questionnaire without questions shows the empty session screen. | Surveys |
| `/survey/session/next_question/<survey_token:text>` | `survey_session_next_question` | remote call | signed in | POST | yes | `survey_token`, `go_back` (accepts further named parameters) | Advances the live session to the next question, or back to the previous one when asked, and returns the new question content for the transition. | Surveys |
| `/survey/session/results/<survey_token:text>` | `survey_session_results` | remote call | signed in | POST | yes | `survey_token` (accepts further named parameters) | Returns the answer counts of the current question of a live session for display beside the question. | Surveys |
| `/survey/session/leaderboard/<survey_token:text>` | `survey_session_leaderboard` | remote call | signed in | POST | yes | `survey_token` (accepts further named parameters) | Returns the ranking of the participants of a live session after the current question. | Surveys |

### Courses and course contents

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/slides` <br> `/slides/page/<page:integer>` <br> `/slides/tag/<slug_tags:text>` <br> `/slides/tag/<slug_tags:text>/page/<page:integer>` | `slides_channel` | page or file | public or signed in | GET, POST | yes | `slide_category`, `slug_tags`, `my`, `page` (accepts further named parameters) | Lists the published courses, filtered by content category and by tags, with paging and a personal view that shows only the courses the visitor follows. | Electronic Learning |
| `/slides/all` <br> `/slides/all/tag/<slug_tags:text>` | `slides_channel_all` | page or file | public or signed in | GET, POST | yes | `slug_tags` (accepts further named parameters) | Lists every published course, filtered by tags, as a single listing. | Electronic Learning |
| `/slides/<channel_id:integer>` <br> `/slides/<channel_id:integer>/category/<category_id:integer>` <br> `/slides/<channel_id:integer>/category/<category_id:integer>/page/<page:integer>` <br> `/slides/<channel:record of Course>` <br> `/slides/<channel:record of Course>/page/<page:integer>` <br> `/slides/<channel:record of Course>/tag/<tag:record of Content Tag>` <br> `/slides/<channel:record of Course>/tag/<tag:record of Content Tag>/page/<page:integer>` <br> `/slides/<channel:record of Course>/category/<category:record of Course Content>` <br> `/slides/<channel:record of Course>/category/<category:record of Course Content>/page/<page:integer>` | `channel` | page or file | public or signed in | GET, POST | yes | `channel`, `channel`, `category`, `category`, `tag`, `page`, `slide_category`, `uncategorized`, `sorting`, `search` (accepts further named parameters) | Serves one course: its description, its sections, its contents filtered by category, tag, content type and typed text, the progress of the visitor and the enrolment action. | Electronic Learning |
| `/slides/slide/<slide:record of Course Content>` | `slide_view` | page or file | public or signed in | GET, POST | yes | `slide` (accepts further named parameters) | Serves one course content (a document, a video, an image, a web page or a quiz) with the player, the completion controls and the discussion thread. | Electronic Learning |
| `/slides/slide/<slide_id:integer>/share` | `slide_shared_view` | page or file | public or signed in | GET, POST | yes | `slide` (accepts further named parameters) | Serves one course content through a shared link, with the sharing layout and without the course navigation. | Electronic Learning |
| `/slides/embed/<slide_id:integer>` | `slides_embed` | page or file | public or signed in | GET, POST | yes | `slide`, `page` (accepts further named parameters) | Serves one course content in an embeddable frame for a page of this platform, at the requested page number. | Electronic Learning |
| `/slides/embed_external/<slide_id:integer>` | `slides_embed_external` | page or file | public or signed in | GET, POST | yes | `slide`, `page` (accepts further named parameters) | Serves one course content in an embeddable frame for a foreign site, with the embedding permissions that the course allows. | Electronic Learning |
| `/slides/slide/<slide:record of Course Content>/pdf_content` | `slide_get_portable_document_content` | page or file | public or signed in | GET, POST | yes | `slide` | Streams the printable document of a document content for the viewer. | Electronic Learning |
| `/slides/slide/get_html_content` | `get_rich_text_content` | remote call | public or signed in | POST | yes | `slide` | Returns the formatted body of a web page content for the viewer. | Electronic Learning |
| `/slides/slide/<slide_id:integer>/get_image` | `slide_get_image` | page or file | public or signed in | GET, POST | yes | `slide`, `field`, `width`, `height`, `crop` | Streams an image field of a course content at the requested size, optionally cropped. | Electronic Learning |
| `/slides/channel/join` | `slide_channel_join` | remote call | public or signed in | POST | yes | `channel` | Enrols the visitor in a course that allows free enrolment and returns the refreshed course state. | Electronic Learning |
| `/slides/channel/leave` | `slide_channel_leave` | remote call | signed in | POST | yes | `channel` | Removes the enrolment of the visitor from a course. | Electronic Learning |
| `/slides/channel/subscribe` | `slide_channel_subscribe` | remote call | signed in | POST | yes | `channel` | Adds the visitor to the followers of a course, which sends the course notifications. | Electronic Learning |
| `/slides/channel/unsubscribe` | `slide_channel_unsubscribe` | remote call | signed in | POST | yes | `channel` | Removes the visitor from the followers of a course. | Electronic Learning |
| `/slides/<channel_id:integer>/invite` | `slide_channel_invite` | page or file | public or signed in | GET, POST | yes | `channel`, `invite_partner`, `invite_hash` | Opens a course from an invitation link: it verifies the invitation fingerprint, enrols the invited contact, and redirects to the course when the rights allow it or to the course listing otherwise. | Electronic Learning |
| `/slides/<channel_id:integer>/identify` | `slide_channel_identify_from_invite` | page or file | public or signed in | GET, POST | yes | `channel`, `invite_partner`, `invite_hash` | Sends an invited visitor that chooses to sign in or register to the correct identification page and back to the course afterwards. | Electronic Learning |
| `/slides/channel/send_share_email` | `slide_channel_send_share_email` | remote call | signed in | POST | yes | `channel`, `emails` | Sends the invitation message of a course to the given addresses. | Electronic Learning |
| `/slides/slide/send_share_email` | `slide_send_share_email` | remote call | signed in | POST | yes | `slide`, `emails`, `fullscreen` | Sends the sharing message of one course content to the given addresses, with the full-screen player link when requested. | Electronic Learning |
| `/slides/slide/set_completed` | `slide_set_completed` | remote call | public or signed in | POST | yes | `slide` | Marks one course content as completed for the visitor and returns the refreshed course progress. | Electronic Learning |
| `/slides/slide/set_uncompleted` | `slide_set_uncompleted` | remote call | public or signed in | POST | yes | `slide` | Marks one course content as not completed for the visitor and returns the refreshed progress. | Electronic Learning |
| `/slides/slide/<slide:record of Course Content>/set_completed` | `slide_set_completed_and_redirect` | page or file | signed in | GET, POST | yes | `slide`, `next_slide` | Marks one course content as completed and redirects to the next content. | Electronic Learning |
| `/slides/slide/<slide:record of Course Content>/set_uncompleted` | `slide_set_uncompleted_and_redirect` | page or file | signed in | GET, POST | yes | `slide` | Marks one course content as not completed and redirects back to it. | Electronic Learning |
| `/slides/slide/like` | `slide_like` | remote call | public or signed in | POST | yes | `slide`, `upvote` | Records an approving or disapproving vote of the visitor on one course content and returns the new counts. | Electronic Learning |
| `/slides/add_slide` | `create_slide` | remote call | signed in | POST | yes | accepts further named parameters | Creates a course content from the course page with its type, title, category, document or address and publication state; the certification package adds the certification content type to the same call. | Electronic Learning + Course Certifications |
| `/slides/category/add` | `slide_category_add` | page or file | signed in | POST | yes | `channel`, `name` | Adds a section to a course, placed at the end of the content list. | Electronic Learning |
| `/slides/slide/archive` | `slide_archive` | remote call | signed in | POST | yes | `slide` | Archives one course content, which removes it from the course without deleting the progress already recorded. | Electronic Learning |
| `/slides/slide/toggle_is_preview` | `slide_preview` | remote call | signed in | POST | yes | `slide` | Switches one course content between preview (visible without enrolment) and restricted. | Electronic Learning |
| `/slides/prepare_preview` | `prepare_preview` | remote call | signed in | POST | yes | `channel`, `slide_category`, `web_address` | Fetches the title, description, duration and thumbnail of an external document or video address from its source service, by creating a temporary content record and reading its derived fields. | Electronic Learning |
| `/slides/slide/quiz/get` | `slide_quiz_get` | remote call | public or signed in | POST | yes | `slide` | Returns the questions, the possible answers, the previous attempts and the remaining attempts of the quiz of a content. | Electronic Learning |
| `/slides/slide/quiz/submit` | `slide_quiz_submit` | remote call | public or signed in | POST | yes | `slide`, `answer` | Grades the submitted quiz answers, returns the correct answers, the earned points and the new rank, and marks the content completed when the quiz is passed. | Electronic Learning |
| `/slides/slide/quiz/reset` | `slide_quiz_reset` | remote call | signed in | POST | yes | `slide` | Clears the quiz attempts of the visitor on one content, therefore the quiz may be taken again. | Electronic Learning |
| `/slides/slide/quiz/question_add_or_update` | `slide_quiz_question_add_or_update` | remote call | signed in | POST | yes | `slide`, `question`, `sequence`, `answer`, `existing_question` | Adds a quiz question with its answers to a content, or replaces an existing question, and clears the completion of the author, therefore the quiz can be taken again. | Electronic Learning |
| `/slides/slide/quiz/save_to_session` | `slide_quiz_save_to_session` | remote call | public or signed in | POST | yes | `quiz_answers` | Keeps the answers of an anonymous visitor in the session, therefore they are graded after the visitor signs in. | Electronic Learning |
| `/slides/category/search_read` | `slide_category_search_read` | remote call | signed in | POST | yes | `fields`, `domain` | Returns the sections matching a filter condition, for the pickers of the course editor. | Electronic Learning |
| `/slides/tag/search_read` | `slide_tag_search_read` | remote call | signed in | POST | yes | `fields`, `domain` | Returns the content tags matching a filter condition. | Electronic Learning |
| `/slides/channel/tag/search_read` | `slide_channel_tag_search_read` | remote call | signed in | POST | yes | `fields`, `domain` | Returns the course tags matching a filter condition. | Electronic Learning |
| `/slides/channel/tag/group/search_read` | `slide_channel_tag_group_search_read` | remote call | signed in | POST | yes | `fields`, `domain` | Returns the course tag groups matching a filter condition. | Electronic Learning |
| `/slides/channel/tag/add` | `slide_channel_tag_add` | remote call | signed in | POST | yes | `channel`, `tag`, `group` | Adds a course tag to a course, creating the tag and its group when the request carries a new name instead of an existing reference. | Electronic Learning |
| `/slide_channel_tag/add` | `slide_channel_tag_create_or_get` | remote call | signed in | POST | yes | `tag`, `group` | Returns the reference of a course tag in a group, creating it when it does not exist yet. | Electronic Learning |
| `/slides_survey/certification/search_read` | `slides_certification_search_read` | remote call | signed in | POST | yes | `fields` | Returns the certifications that can be attached to a course content. | Course Certifications |
| `/slides_survey/slide/get_certification_url` | `slide_get_certification_web_address` | page or file | signed in | GET, POST | yes | `slide` (accepts further named parameters) | Returns the address at which the visitor takes the certification attached to a course content. | Course Certifications |
| `/slides/get_course_products` | `get_course_products` | remote call | signed in | POST | no | none | Returns the sellable products of the paid courses with their formatted prices, for the course listing. | Sell Courses |

### Community forum

Every write action of this group is guarded by the reputation score of the author; the score thresholds are configured per
forum and are specified in the forum rules of the content domain. Actions performed by a moderator bypass the thresholds.

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/forum` | `forum` | page or file | public or signed in | GET, POST | yes | accepts further named parameters | Lists the published forums with their description and question counts. | Forum |
| `/forum/all` <br> `/forum/all/page/<page:integer>` <br> `/forum/<forum:record of Forum>` <br> `/forum/<forum:record of Forum>/page/<page:integer>` <br> `/forum/<forum:record of Forum>/tag/<tag:record of Forum Tag>/questions` <br> `/forum/<forum:record of Forum>/tag/<tag:record of Forum Tag>/questions/page/<page:integer>` | `questions` | page or file | public or signed in | GET, POST | yes | `forum`, `tag`, `page`, `filters`, `my`, `sorting`, `search`, `created_by_user`, `include_answers` (accepts further named parameters) | Lists the questions of one forum or of all forums, filtered by tag, by state (unanswered, unsolved, followed, own posts), sorted by date, answers, votes or relevance, with free-text search and paging. | Forum |
| `/forum/<forum:record of Forum>/<question:record of Forum Post>` | `question` | page or file | public or signed in | GET, POST | yes | `forum`, `question` (accepts further named parameters) | Serves one question with its answers, comments, votes, tags and the moderation actions available to the reader. | Forum |
| `/forum/<forum:record of Forum>/question/<question:record of Forum Post>` | `old_question` | page or file | public or signed in | GET, POST | yes | `forum`, `question` (accepts further named parameters) | Redirects a question address of the earlier addressing scheme to the current question page. | Forum |
| `/forum/<forum:record of Forum>/ask` | `forum_post` | page or file | signed in | GET, POST | yes | `forum` (accepts further named parameters) | Serves the form for asking a new question in a forum. | Forum |
| `/forum/<forum:record of Forum>/new` <br> `/forum/<forum:record of Forum>/<post_parent:record of Forum Post>/reply` | `post_create` | page or file | signed in | POST | yes | `forum`, `post_parent` (accepts further named parameters) | Creates a question, or an answer when a parent post is named, from the submitted form, and redirects to the resulting page. | Forum |
| `/forum/<forum:record of Forum>/post/<post:record of Forum Post>/edit` | `post_edit` | page or file | signed in | GET, POST | yes | `forum`, `post` (accepts further named parameters) | Serves the edit form of a post to an author or moderator with sufficient reputation. | Forum |
| `/forum/<forum:record of Forum>/post/<post:record of Forum Post>/save` | `post_save` | page or file | signed in | POST | yes | `forum`, `post` (accepts further named parameters) | Saves the edited title, body and tags of a post and redirects to the question. | Forum |
| `/forum/<forum:record of Forum>/post/<post:record of Forum Post>/upvote` | `post_upvote` | remote call | signed in | POST | yes | `forum`, `post` (accepts further named parameters) | Records an approving vote on a post and returns the new vote count and the reputation change. | Forum |
| `/forum/<forum:record of Forum>/post/<post:record of Forum Post>/downvote` | `post_downvote` | remote call | signed in | POST | yes | `forum`, `post` (accepts further named parameters) | Records a disapproving vote on a post and returns the new vote count and the reputation change. | Forum |
| `/forum/<forum:record of Forum>/post/<post:record of Forum Post>/toggle_correct` | `post_toggle_correct` | remote call | signed in | POST | yes | `forum`, `post` (accepts further named parameters) | Marks an answer as the accepted one, or removes that mark, which awards or withdraws the reputation of an accepted answer. | Forum |
| `/forum/<forum:record of Forum>/post/<post:record of Forum Post>/comment` | `post_comment` | page or file | signed in | POST | yes | `forum`, `post` (accepts further named parameters) | Adds a comment under a question or an answer. | Forum |
| `/forum/<forum:record of Forum>/post/<post:record of Forum Post>/comment/<comment:record of Message>/delete` | `delete_comment` | remote call | signed in | POST | yes | `forum`, `post`, `comment` (accepts further named parameters) | Deletes one comment of a post. | Forum |
| `/forum/<forum:record of Forum>/post/<post:record of Forum Post>/comment/<comment:record of Message>/convert_to_answer` | `convert_comment_to_answer` | page or file | signed in | POST | yes | `forum`, `post`, `comment` (accepts further named parameters) | Turns a comment into a full answer of the question. | Forum |
| `/forum/<forum:record of Forum>/post/<post:record of Forum Post>/convert_to_comment` | `convert_answer_to_comment` | page or file | signed in | POST | yes | `forum`, `post` (accepts further named parameters) | Turns an answer into a comment on the question. | Forum |
| `/forum/<forum:record of Forum>/post/<post:record of Forum Post>/delete` | `post_delete` | page or file | signed in | POST | yes | `forum`, `post` (accepts further named parameters) | Deletes a post, which hides it from the forum and withdraws the reputation it earned. | Forum |
| `/forum/<forum:record of Forum>/post/<post:record of Forum Post>/validate` | `post_accept` | page or file | signed in | GET, POST | yes | `forum`, `post` (accepts further named parameters) | Accepts a post that waited in the moderation queue and publishes it. | Forum |
| `/forum/<forum:record of Forum>/post/<post:record of Forum Post>/refuse` | `post_refuse` | page or file | signed in | GET, POST | yes | `forum`, `post` (accepts further named parameters) | Refuses a post that waited in the moderation queue and notifies its author. | Forum |
| `/forum/<forum:record of Forum>/post/<post:record of Forum Post>/flag` | `post_flag` | remote call | signed in | POST | yes | `forum`, `post` (accepts further named parameters) | Flags a post for moderation and returns the new flag state. | Forum |
| `/forum/<forum:record of Forum>/post/<post:record of Forum Post>/mark_as_offensive` | `post_mark_as_offensive` | page or file | signed in | POST | yes | `forum`, `post` (accepts further named parameters) | Marks a flagged post as offensive with the chosen reason, which hides it and withdraws reputation from its author. | Forum |
| `/forum/<forum:record of Forum>/post/<post:record of Forum Post>/ask_for_mark_as_offensive` | `post_hypertext_transfer_protocol_ask_for_mark_as_offensive` | page or file | signed in | GET | yes | `forum`, `post` (accepts further named parameters) | Serves the page that asks a moderator to confirm marking a post as offensive and to choose the reason. | Forum |
| `/forum/<post:record of Forum Post>/ask_for_mark_as_offensive` | `post_structured_data_ask_for_mark_as_offensive` | remote call | signed in | POST | yes | `post` (accepts further named parameters) | Returns the same confirmation data as a structured answer, for the moderation panel. | Forum |
| `/forum/<forum:record of Forum>/question/<question:record of Forum Post>/close` | `question_close` | page or file | signed in | POST | yes | `forum`, `question` (accepts further named parameters) | Closes a question with the chosen reason, which keeps it visible but refuses new answers. | Forum |
| `/forum/<forum:record of Forum>/question/<question:record of Forum Post>/ask_for_close` | `question_ask_for_close` | page or file | signed in | POST | yes | `forum`, `question` (accepts further named parameters) | Serves the page that asks for the closing reason of a question. | Forum |
| `/forum/<forum:record of Forum>/question/<question:record of Forum Post>/reopen` | `question_reopen` | page or file | signed in | POST | yes | `forum`, `question` (accepts further named parameters) | Reopens a closed question. | Forum |
| `/forum/<forum:record of Forum>/question/<question:record of Forum Post>/delete` | `question_delete` | page or file | signed in | POST | yes | `forum`, `question` (accepts further named parameters) | Deletes a question and its answers. | Forum |
| `/forum/<forum:record of Forum>/question/<question:record of Forum Post>/undelete` | `question_undelete` | page or file | signed in | POST | yes | `forum`, `question` (accepts further named parameters) | Restores a deleted question. | Forum |
| `/forum/<forum:record of Forum>/question/<question:record of Forum Post>/edit_answer` | `question_edit_answer` | page or file | signed in | GET, POST | yes | `forum`, `question` (accepts further named parameters) | Opens the edit form of the own answer to a question. | Forum |
| `/forum/<forum:record of Forum>/question/<question:record of Forum Post>/toggle_favourite` | `question_toggle_favorite` | remote call | signed in | POST | yes | `forum`, `question` (accepts further named parameters) | Adds or removes a question from the favourites of the visitor. | Forum |
| `/forum/<forum:record of Forum>/validation_queue` | `validation_queue` | page or file | signed in | GET, POST | yes | `forum` (accepts further named parameters) | Lists the posts waiting for moderation in a forum. | Forum |
| `/forum/<forum:record of Forum>/flagged_queue` | `flagged_queue` | page or file | signed in | GET, POST | yes | `forum` (accepts further named parameters) | Lists the posts that visitors flagged in a forum. | Forum |
| `/forum/<forum:record of Forum>/offensive_posts` | `offensive_posts` | page or file | signed in | GET, POST | yes | `forum` (accepts further named parameters) | Lists the posts marked as offensive in a forum. | Forum |
| `/forum/<forum:record of Forum>/closed_posts` | `closed_posts` | page or file | signed in | GET, POST | yes | `forum` (accepts further named parameters) | Lists the closed questions of a forum. | Forum |
| `/forum/<forum:record of Forum>/tag` <br> `/forum/<forum:record of Forum>/tag/<tag_char:text>` | `tags` | page or file | public or signed in | GET, POST | yes | `forum`, `tag_char`, `filters`, `search` (accepts further named parameters) | Lists the tags of a forum, restricted to those beginning with one character when the address names it, filtered by all, followed, most used or unused, and by typed text. | Forum |
| `/forum/get_tags` | `tag_read` | page or file | public or signed in | GET | yes | `forum`, `query`, `limit` (accepts further named parameters) | Returns the tags of a forum matching the typed text, limited to the requested count, for the tag picker of the question form. | Forum |
| `/forum/<forum:record of Forum>/faq` | `forum_frequently_asked_questions` | page or file | public or signed in | GET, POST | yes | `forum` (accepts further named parameters) | Serves the guidelines page of a forum. | Forum |
| `/forum/<forum:record of Forum>/faq/karma` | `forum_frequently_asked_questions_karma` | page or file | public or signed in | GET, POST | yes | `forum` (accepts further named parameters) | Serves the page that explains the reputation thresholds of the forum and what each of them unlocks. | Forum |
| `/forum/<forum:record of Forum>/partner/<partner_id:integer>` | `open_partner` | page or file | public or signed in | GET, POST | yes | `forum`, `partner` (accepts further named parameters) | Redirects from a contact to the public profile of the user of that contact on the forum. | Forum |
| `/forum/user/<user_id:integer>` | `view_user_forum_profile` | page or file | public or signed in | GET, POST | yes | `user` | Serves the forum profile of one user: reputation, badges, questions, answers and votes. | Forum |
| `/forum/get_url_title` | `get_web_address_title` | remote call | signed in | POST | yes | accepts further named parameters | Returns the title of an external address pasted into a post, for building a readable link. | Forum |

## Site and Content Management

The site endpoints serve the public pages, the in-place editor that authors those pages, the media and shape services the
editor draws from, the site-wide search, the dynamic content blocks, the public form handler, the multi-site and
multi-language switches, and the public profile pages. Editing endpoints require the content editor group; the pages
themselves are public unless the page record says otherwise.

### Public pages, routing, languages and sites

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/pages` <br> `/pages/page/<page:integer>` | `pages_list` | page or file | public or signed in | GET, POST | yes | `page`, `search` (accepts further named parameters) | Lists the published pages of the site with a free-text filter and paging. | Website |
| `/model/<page_name_slugified:text>` <br> `/model/<page_name_slugified:text>/page/<page_number:integer>` <br> `/model/<page_name_slugified:text>/<record_slug:text>` | `generic_model` | page or file | public or signed in | GET, POST | yes | `page_name_slugified`, `page_number`, `record_slug` (accepts further named parameters) | Serves the generated listing and detail pages of an entity that was published as a page family, with paging and a record segment. | Website |
| `/@/` <br> `/@/<path:path>` | `client_action_redirect` | page or file | public or signed in | GET, POST | yes | `path` (accepts further named parameters) | Sends an internal user to the editing preview of the requested public path and every other visitor to the plain public page. | Website |
| `/website/force/<website_id:integer>` | `website_force` | page or file | signed in | GET, POST | yes | `website`, `path`, `isredir` (accepts further named parameters) | Switches the session to another site of the installation after the visitor has reached the domain of that site, then continues to the requested path. | Website |
| `/website/lang/<lang>` | `change_language` | page or file | public or signed in | GET, POST | yes | `language`, `r` (accepts further named parameters) | Switches the display language of the site for the visitor and redirects to the translated address of the current page; the storefront package additionally repricing the cart in the currency attached to the new language. | Website + Electronic Commerce |
| `/website/get_languages` | `website_languages` | remote call | signed in | POST | yes | accepts further named parameters | Returns the languages published on the current site with their codes and names. | Website |
| `/website/translations` | `get_website_translations` | page or file | public or signed in | GET, POST | no | `hash`, `language`, `mods` | Returns the translated texts needed by the public pages of the site, for the requested language, capability packages and fingerprint. | Web Routing |
| `/website/get_translated_elements` | `translated_elements` | remote call | signed in | POST | no | accepts further named parameters | Returns the translatable parts of the current page for the in-place translation mode. | Website |
| `/website/field/translation/update` | `update_field_translation` | remote call | signed in | POST | yes | `model_name`, `record`, `field_name`, `translations` | Stores the translations typed in place for one field of one record in the languages that were edited. | Website |
| `/website/country_infos/<country:record of Country>` | `country_infos` | remote call | public or signed in | POST | yes | `country` (accepts further named parameters) | Returns the address rules of a country (state list, state requirement, postal code format and field order) for the public address forms. | Website |
| `/website/social/<social:text>` | `social` | page or file | public or signed in | GET, POST | yes | `social` (accepts further named parameters) | Redirects to the address of one social network account configured on the site. | Website |
| `/website/info` | `website_info` | page or file | public or signed in | GET, POST | yes | accepts further named parameters | Serves the page that shows the version and the installed capability packages of the site. | Website |
| `/favicon.ico` | `favicon` | page or file | public or signed in | GET, POST | yes | accepts further named parameters | Streams the site icon. | Website |
| `/sitemap.xml` | `sitemap_markup_document_index` | page or file | public or signed in | GET, POST | yes | accepts further named parameters | Returns the page index for crawlers, generated from the published pages and regenerated when it is older than the configured age. | Website |
| `/google<key:text of length 16>.html` | `google_console_search` | page or file | public or signed in | GET, POST | yes | `key` (accepts further named parameters) | Serves the ownership verification file of the search console under the key configured on the site. | Website |
| `/web/assets/<website_id:integer>/<unique>/<filename:text>` | `content_assets_website` | page or file | public or signed in | GET, POST | no | `website` (accepts further named parameters) | Streams a generated resource bundle of one site, which differs between sites because each site has its own appearance settings. | Website |

### In-place page editing

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/website/add` <br> `/website/add/<path:path>` | `pagenew` | page or file | signed in | POST | yes | `path`, `add_menu`, `template`, `redirect` (accepts further named parameters) | Creates a page at the requested path from the chosen template, optionally adding it to the menu, and redirects to it in edit mode. | Website |
| `/website/get_new_page_templates` | `get_new_page_templates` | remote call | signed in | POST | yes | accepts further named parameters | Returns the page templates offered when a new page is created. | Website |
| `/website/check_new_content_access_rights` | `check_create_access_rights` | remote call | signed in | POST | no | `models` | Reports which content types of the creation menu the current user may create, therefore the menu only offers the allowed ones. | Website |
| `/website/check_can_modify_any` | `check_can_modify_any` | remote call | signed in | POST | yes | `records` | Reports whether the user may modify at least one of the given records, which decides whether the editing controls are shown. | Website |
| `/website/save_xml` | `save_markup_document` | remote call | signed in | POST | yes | `view`, `arch` | Saves the edited structure of one page template. | Website |
| `/website/reset_template` | `reset_template` | remote call | signed in | POST | no | `view`, `mode` (accepts further named parameters) | Restores a damaged page template: the soft mode returns to the previous saved structure, the hard mode returns to the structure the capability package shipped. | Website |
| `/website/get_switchable_related_views` | `get_switchable_related_views` | remote call | signed in | POST | yes | `key` | Returns the optional page parts that can be switched on or off for the current page. | Website |
| `/website/theme_customize_data` | `theme_customize_data` | remote call | signed in | POST | yes | `is_view_data`, `enable`, `disable`, `reset_view_arch` | Enables and disables the named page parts or resource entries of the appearance settings, optionally restoring their shipped structure; the storefront package adds its own storefront parts to the same call. | Website + Electronic Commerce |
| `/website/theme_customize_data_get` | `theme_customize_data_get` | remote call | signed in | POST | yes | `keys`, `is_view_data` | Returns which of the named page parts or resource entries are currently enabled. | Website |
| `/website/theme_customize_bundle_reload` | `theme_customize_bundle_reload` | remote call | signed in | POST | yes | none | Rebuilds the resource bundles after an appearance change and returns their new versioned addresses. | Website |
| `/website/update_footer_template` | `update_footer_template` | remote call | signed in | POST | yes | `template_key`, `possible_values` | Switches the page foot to another template and aligns the copyright band with it. | Website |
| `/website/get_assets_editor_resources` | `get_assets_editor_resources` | remote call | signed in | POST | yes | `key`, `get_views`, `get_scss`, `get_js`, `bundles`, `bundles_restriction`, `only_user_custom_files` | Returns the page templates, style sources and script sources related to one page, for the resource editor, restricted to the personal customizations when requested. | Website |
| `/website/save_session_layout_mode` | `save_session_layout_mode` | remote call | public or signed in | POST | yes | `layout_mode`, `view` | Remembers, for the session, whether a listing is displayed as a grid or as a list. | Website |
| `/website/iframefallback` | `get_iframe_fallback` | page or file | signed in | GET, POST | yes | none | Serves the placeholder page shown inside the editing frame while the edited page is reloading. | Website |
| `/website/get_seo_data` | `get_seo_data` | remote call | signed in | POST | yes | `related_record_identifier`, `related_record_model` | Returns the indexing data of a record or page (title, description, keywords, social preview image and the derived address) for the optimization panel. | Website |
| `/website/seo_suggest` | `seo_suggest` | remote call | signed in | POST | yes | `keywords`, `language` | Returns keyword suggestions for the typed text in the chosen language and region, obtained from the suggestion service of the external search engine. | Website |
| `/website/get_suggested_links` | `get_suggested_link` | remote call | signed in | POST | yes | `needle`, `limit` | Returns the internal pages and records matching the typed text, for the link picker of the editor. | Website |
| `/website/check_existing_link` | `check_existing_link` | remote call | signed in | POST | yes | `link` | Reports whether an address already exists on the site, which warns the author about a duplicate page. | Website |
| `/website/update_broken_links` | `update_broken_links` | remote call | signed in | POST | yes | `links` | Rewrites the links the editor detected as broken to the addresses supplied. | Website |
| `/website/get_alt_images` | `get_alt_images` | remote call | signed in | POST | yes | `models` | Returns the images of the given records that have no alternative text, for the accessibility panel. | Website |
| `/website/update_alt_images` | `update_alt_images` | remote call | signed in | POST | yes | `imgs` | Stores the alternative texts typed for those images. | Website |
| `/website/action/<path_or_xml_id_or_id>` <br> `/website/action/<path_or_xml_id_or_id>/<path:path>` | `actions_server` | page or file | public or signed in | GET, POST | yes | `path_or_markup_document_identifier_or` (accepts further named parameters) | Runs a published action from a public page address and returns the page or the redirection it produces. | Website |
| `/website/track_installing_modules` | `website_track_installing_modules` | remote call | signed in | POST | no | `selected_features`, `total_features` | Reports the progress of the feature installation started by the site configurator, as the count of installed features against the count selected. | Website |
| `/website/configurator` <br> `/website/configurator/<step:integer>` | `website_configurator` | page or file | signed in | GET, POST | yes | `step` (accepts further named parameters) | Serves the site configurator step by step: purpose, name, appearance palette, features and page generation, and starts the generation at the last step. | Website |
| `/website/fetch_dashboard_data` | `fetch_dashboard_data` | remote call | signed in | POST | no | `website` | Returns the visit and conversion figures of one site for the site dashboard. | Website |

### Editor media, images, shapes and video

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/web_editor/attachment/add_data` <br> `/html_editor/attachment/add_data` | `add_data` | remote call | signed in | POST | yes | `name`, `data`, `is_image`, `quality`, `width`, `height`, `related_record_identifier`, `related_record_model` (accepts further named parameters) | Stores an uploaded image or file as an attachment of the site or of a record, optionally re-encoding it to the requested quality, width and height, and returns its descriptor. | Rich Text Editor |
| `/web_editor/attachment/add_url` <br> `/html_editor/attachment/add_url` | `add_web_address` | remote call | signed in | POST | yes | `web_address`, `related_record_identifier`, `related_record_model` (accepts further named parameters) | Stores an external image address as an attachment reference on the site or on a record. | Rich Text Editor |
| `/html_editor/attachment/remove` | `remove` | remote call | signed in | POST | yes | `ids` (accepts further named parameters) | Deletes the given image attachments when no page template still uses them, and returns the attachments that were kept together with the templates that hold them. | Rich Text Editor |
| `/web_editor/get_image_info` <br> `/html_editor/get_image_info` | `get_image_info` | remote call | signed in | POST | yes | `source` | Returns the original source, the format and the transformation parameters of an image already placed on a page, therefore it can be cropped, filtered or re-optimized from its original. | Rich Text Editor |
| `/web_editor/modify_image/<attachment:record of Attachment>` <br> `/html_editor/modify_image/<attachment:record of Attachment>` | `modify_image` | remote call | signed in | POST | yes | `attachment`, `related_record_model`, `related_record_identifier`, `name`, `data`, `original`, `mimetype`, `alt_data` | Stores a transformed copy of an image attachment (cropped, filtered, resized or re-encoded) with its alternative text and returns the address to place in the page. | Rich Text Editor |
| `/web_editor/shape/<module>/<filename:path>` <br> `/html_editor/shape/<module>/<filename:path>` | `shape` | page or file | public or signed in | GET, POST | yes | `module`, `filename` (accepts further named parameters) | Returns a vector background shape or illustration with the colours of the current palette substituted into it. | Rich Text Editor |
| `/web_editor/image_shape/<img_key:text>/<module>/<filename:path>` <br> `/html_editor/image_shape/<img_key:text>/<module>/<filename:path>` | `image_shape` | page or file | public or signed in | GET, POST | yes | `module`, `filename`, `image_key` (accepts further named parameters) | Returns an image masked by a vector shape, combining the stored image with the named shape. | Rich Text Editor |
| `/web_editor/video_url/data` <br> `/html_editor/video_url/data` | `video_web_address_data` | remote call | signed in | POST | yes | `video_web_address`, `autoplay`, `loop`, `hide_controls`, `hide_fullscreen`, `hide_dm_logo`, `hide_dm_share`, `start_from` | Resolves a pasted video address to its embedding data (platform, identifier, thumbnail and the embedding address built from the chosen options: automatic start, repeat, hidden controls, hidden full screen button, hidden platform logo, hidden sharing controls and start time). | Rich Text Editor |
| `/html_editor/media_library_search` | `media_library_search` | remote call | signed in | POST | yes | accepts further named parameters | Searches the illustration library of the publisher and returns the matching items with their preview addresses. | Rich Text Editor |
| `/web_editor/save_library_media` <br> `/html_editor/save_library_media` | `save_library_media` | remote call | signed in | POST | no | `media` | Stores the chosen illustrations as attachments of the site, converting the ones that support colour substitution into recolourable vector images. | Rich Text Editor |
| `/web_editor/get_ice_servers` <br> `/html_editor/get_ice_servers` | `get_ice_servers` | remote call | signed in | POST | no | none | Returns the connection relay servers used by the collaborative editing session. | Rich Text Editor |
| `/web_editor/bus_broadcast` <br> `/html_editor/bus_broadcast` | `bus_broadcast` | remote call | signed in | POST | no | `model_name`, `field_name`, `related_record_identifier`, `bus_data` | Broadcasts the editing operations of one field of one record to the other editors of the same field, for collaborative editing. | Rich Text Editor |
| `/web_editor/generate_text` <br> `/html_editor/generate_text` | `generate_text` | remote call | signed in | POST | no | `prompt`, `conversation_history` | Returns text generated for the typed instruction and the previous exchanges, from the text generation service. | Rich Text Editor |
| `/html_editor/link_preview_internal` | `link_preview_metadata_internal` | remote call | signed in | POST | no | `preview_web_address` | Returns the title, description and image of an internal address, for the link preview card. | Rich Text Editor |
| `/html_editor/link_preview_external` | `link_preview_metadata` | remote call | public or signed in | POST | no | `preview_web_address` | Returns the title, description and image of an external address, fetched from that address. | Rich Text Editor |
| `/website/theme_upload_font` | `theme_upload_font` | remote call | signed in | POST | yes | `name`, `data` | Stores an uploaded font file and returns, for each face it contains, the name, the media type and the address at which it is served. | Website |
| `/website/google_font_metadata` | `google_font_metadata` | remote call | signed in | POST | yes | none | Returns the catalogue of the external font service, cached on the server, therefore the editor does not call that service from the browser. | Website |
| `/website/image` <br> `/website/image/<xmlid>` <br> `/website/image/<xmlid>/<width:integer>x<height:integer>` <br> `/website/image/<xmlid>/<field>` <br> `/website/image/<xmlid>/<field>/<width:integer>x<height:integer>` <br> `/website/image/<model>/<id>/<field>` <br> `/website/image/<model>/<id>/<field>/<width:integer>x<height:integer>` | `website_content_image` | page or file | public or signed in | GET, POST | no | `identifier`, `maximum_width`, `maximum_height` (accepts further named parameters) | Streams an image field of a record or an external identifier, bounded by a maximum width and height, for use inside page content. | Website |

### External image library

The image library of a stock photography service is offered inside the media picker. The platform proxies the search therefore
the service credentials stay on the server, and it must register each downloaded image with the service.

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/web_unsplash/fetch_images` | `fetch_unsplash_images` | remote call | signed in | POST | no | accepts further named parameters | Returns a page of photographs matching the typed text from the external photograph service, using the access key stored in the installation. | Unsplash Image Library |
| `/web_unsplash/attachment/add` | `save_unsplash_web_address` | remote call | signed in | POST | no | `unsplashurls` (accepts further named parameters) | Stores the chosen photographs as attachments and calls the download registration address of the service for each of them, as its terms require. | Unsplash Image Library |
| `/web_unsplash/get_app_id` | `get_unsplash_app` | remote call | public or signed in | POST | no | accepts further named parameters | Returns the registered application identity used when the media picker calls the service directly. | Unsplash Image Library |
| `/web_unsplash/save_unsplash` | `save_unsplash` | remote call | signed in | POST | no | accepts further named parameters | Stores the access key and the application identity of the photograph service in the installation settings. | Unsplash Image Library |

### Site search and dynamic content blocks

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/website/search` <br> `/website/search/page/<page:integer>` <br> `/website/search/<search_type:text>` <br> `/website/search/<search_type:text>/page/<page:integer>` | `hybrid_list` | page or file | public or signed in | GET, POST | yes | `page`, `search`, `search_type` (accepts further named parameters) | Serves the site-wide search results page, optionally restricted to one content type, with paging. | Website |
| `/website/snippet/autocomplete` | `autocomplete` | remote call | public or signed in | POST | yes | `search_type`, `term`, `order`, `limit`, `maximum_number_of_chars`, `options` | Returns the search suggestions for the typed text across the selected content types, with the requested ordering, result count, text truncation and display options; the storefront package adds products and their prices to the suggestions. | Website + Electronic Commerce |
| `/website/snippet/filters` | `get_dynamic_filter` | remote call | public or signed in | POST | yes | `filter` (accepts further named parameters) | Returns the records selected by one dynamic content filter, rendered with the chosen template, for a dynamic block. | Website |
| `/website/snippet/filter_templates` | `get_dynamic_snippet_templates` | remote call | public or signed in | POST | yes | `filter_name` | Returns the display templates available for one dynamic content filter. | Website |
| `/website/snippet/options_filters` | `get_dynamic_snippet_filters` | remote call | signed in | POST | yes | `model_name`, `search_domain` | Returns the dynamic content filters that can be used for an entity and a filter condition, for the block options panel. | Website |

### Public forms

The public form block writes a record of a chosen entity from a submitted form. Only entities explicitly marked as
writable from a public form are accepted, and only their fields marked as writable from a public form are taken.

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/website/form` | `website_form_empty` | page or file | public or signed in | POST | no | accepts further named parameters | Accepts a submission with no entity named, which is the shape used by the plain contact form, and sends the message to the configured address. | Website |
| `/website/form/<model_name:text>` | `website_form` | page or file | public or signed in | POST | yes | `model_name` (accepts further named parameters) | Creates a record of the named entity from the submitted fields and files, applies the field filter of the form, sends the notification message and returns either the identifier of the created record or the validation messages. | Website |

### Address autocomplete and maps

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/autocomplete/address` | `autocomplete_address` | remote call | public or signed in | POST | yes | `partial_address`, `session`, `use_employees_key` | Returns address suggestions for a partly typed address from the external address service, keeping the session reference that groups the calls of one lookup into one billed request. | Google Address Autocomplete |
| `/autocomplete/address_full` | `autocomplete_address_full` | remote call | public or signed in | POST | yes | `address`, `session`, `google_place`, `use_employees_key` (accepts further named parameters) | Returns the complete structured address (street, number, city, postal code, state and country) of a chosen suggestion. | Google Address Autocomplete |
| `/google_map` | `google_map` | page or file | public or signed in | GET, POST | yes | accepts further named parameters | Serves the map frame of the map content block with the configured pins. | Google Maps |
| `/website/google_maps_api_key` | `google_maps_application_programming_interface_key` | remote call | public or signed in | POST | yes | none | Returns the map service key configured for the site, which the map block needs to load the map. | Website |
| `/website/get_current_currency` | `get_current_currency` | remote call | public or signed in | POST | yes | accepts further named parameters | Returns the currency in force for the visitor with its symbol and position; the storefront package answers with the currency of the price list of the cart. | Website + Electronic Commerce |

### Blog

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/blog` <br> `/blog/page/<page:integer>` <br> `/blog/tag/<tag:text>` <br> `/blog/tag/<tag:text>/page/<page:integer>` <br> `/blog/<blog:record of Blog>` <br> `/blog/<blog:record of Blog>/page/<page:integer>` <br> `/blog/<blog:record of Blog>/tag/<tag:text>` <br> `/blog/<blog:record of Blog>/tag/<tag:text>/page/<page:integer>` | `blog` | page or file | public or signed in | GET, POST | yes | `blog`, `tag`, `page`, `search` (accepts further named parameters) | Lists the posts of one blog or of all blogs, filtered by tag and by typed text, with paging. | Blog |
| `/blog/<blog:record of Blog>/<blog_post:record of Blog Post>` | `blog_post` | page or file | public or signed in | GET, POST | yes | `blog`, `blog_post`, `tag`, `page`, `enable_editor` (accepts further named parameters) | Serves one blog post with its content, tags, the previous and next post, the related posts and the comment thread, and opens the editor when requested. | Blog |
| `/blog/<blog:record of Blog>/post/<blog_post:record of Blog Post>` | `old_blog_post` | page or file | public or signed in | GET, POST | yes | `blog`, `blog_post` (accepts further named parameters) | Redirects a post address of the earlier addressing scheme to the current post page. | Blog |
| `/blog/<blog:record of Blog>/feed` | `blog_feed` | page or file | public or signed in | GET, POST | yes | `blog`, `limit` (accepts further named parameters) | Returns the syndication feed of a blog, limited to the requested number of posts. | Blog |

### Published contact directories and public profiles

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/customers` <br> `/customers/page/<page:integer>` <br> `/customers/country/<country:record of Country>` <br> `/customers/country/<country:record of Country>/page/<page:integer>` <br> `/customers/industry/<industry:record of Industry>` <br> `/customers/industry/<industry:record of Industry>/page/<page:integer>` <br> `/customers/industry/<industry:record of Industry>/country/<country:record of Country>` <br> `/customers/industry/<industry:record of Industry>/country/<country:record of Country>/page/<page:integer>` | `customers` | page or file | public or signed in | GET, POST | yes | `country`, `industry`, `page` (accepts further named parameters) | Lists the published customer references, filtered by country and by industry, with paging. | Customer References |
| `/customers/<partner_id>` | `customers_detail` | page or file | public or signed in | GET, POST | yes | `partner` (accepts further named parameters) | Serves the published page of one customer reference with its description and its implemented solutions. | Customer References |
| `/partners/<partner_id>` | `partners_detail` | page or file | public or signed in | GET, POST | yes | `partner` (accepts further named parameters) | Serves the published page of one contact of a directory; the reseller package adds the partnership level, the assigned leads and the contact form. | Website Partner + Resellers |
| `/profile/users` <br> `/profile/users/page/<page:integer>` | `view_all_users_page` | page or file | public or signed in | GET, POST | yes | `page` (accepts further named parameters) | Lists the public profiles of the users ranked by reputation, with paging. | Website profile |
| `/profile/user/<user_id:integer>` | `view_user_profile` | page or file | public or signed in | GET, POST | yes | `user` (accepts further named parameters) | Serves the public profile of one user: biography, reputation, rank, badges and contributions. | Website profile |
| `/profile/user/save` | `save_edited_profile` | remote call | signed in | POST | yes | accepts further named parameters | Saves the public profile fields the user edited (name, biography, country, website address, city and visibility). | Website profile |
| `/profile/avatar/<user_id:integer>` | `get_user_profile_avatar` | page or file | public or signed in | GET, POST | yes | `user`, `field`, `width`, `height`, `crop` (accepts further named parameters) | Streams the picture of a public profile at the requested size, optionally cropped. | Website profile |
| `/profile/ranks_badges` | `view_ranks_badges` | page or file | public or signed in | GET, POST | yes | accepts further named parameters | Serves the page that explains every rank and badge and how they are earned. | Website profile |
| `/profile/send_validation_email` | `send_validation_email` | remote call | signed in | POST | yes | accepts further named parameters | Sends the address confirmation message to the signed-in user and returns the state of the request. | Website profile |
| `/profile/validate_email` | `validate_email` | page or file | public or signed in | GET, POST | yes | `token`, `user`, `email` (accepts further named parameters) | Confirms the electronic mail address of a user from the token in the confirmation message and awards the confirmation badge. | Website profile |
| `/profile/validate_email/close` | `validate_email_done` | remote call | public or signed in | POST | yes | accepts further named parameters | Serves the page shown after the address confirmation is complete. | Website profile |

## Commerce Storefront

The storefront endpoints carry the whole buying path: catalogue browsing, the product page and its configurator, the cart,
the checkout steps (addresses, delivery method, extra information, payment), the confirmation page, and the storefront
editing actions an editor performs on the catalogue. The cart is the sales order of the session: an anonymous visitor gets
a cart bound to the session, a signed-in customer gets the cart bound to the customer. Every price shown is computed with
the price list in force for the visitor. The checkout steps are configurable: a step that needs no input can be skipped
automatically.

### Catalogue and product pages

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/shop` <br> `/shop/page/<page:integer>` <br> `/shop/category/<category:record>` <br> `/shop/category/<category:record>/page/<page:integer>` | `shop` | page or file | public or signed in | GET, POST | yes | `page`, `category`, `search`, `minimum_price`, `maximum_price`, `tags` (accepts further named parameters) | Serves the catalogue listing: the published products of the site filtered by category, by tag, by attribute values and by a price range, sorted by the chosen order, with free-text search and paging. | Electronic Commerce |
| `/shop/<product:record>` <br> `/shop/<category:record>/<product:record>` | `product` | page or file | public or signed in | GET, POST | yes | `product`, `category`, `pricelist` (accepts further named parameters) | Serves the page of one product: its variants and options, the images, the price for the price list in force, the availability, the documents, the alternative products, the accessories and the add-to-cart action. | Electronic Commerce |
| `/shop/product/<product:record>` | `old_product` | page or file | public or signed in | GET, POST | yes | `product`, `category` (accepts further named parameters) | Redirects a product address of the earlier addressing scheme to the current product page. | Electronic Commerce |
| `/shop/<product_template:record of Product Template>/document/<document_id:integer>` | `product_document` | page or file | public or signed in | GET, POST | yes | `product_template`, `document` | Streams one published document of a product to the visitor. | Electronic Commerce |
| `/website_sale/get_combination_info` | `get_combination_info_website` | remote call | public or signed in | POST | yes | `product_template`, `product`, `combination`, `add_quantity`, `unit_of_measure` (accepts further named parameters) | Returns the name, reference, price, former price, availability, images and configuration state of a chosen combination of options for the requested quantity and unit of measure; the availability package adds the stock figures and the notification package adds whether the visitor may ask to be told when the product returns. | Electronic Commerce + Product Availability + Product Availability Notifications |
| `/shop/product/is_add_to_cart_allowed` | `is_add_to_cart_allowed` | remote call | public or signed in | POST | yes | `product` (accepts further named parameters) | Reports whether a product variant may be put in a cart on this site, which the page uses to enable the action. | Electronic Commerce |
| `/website_sale/should_show_product_configurator` | `website_sale_should_show_product_configurator` | remote call | public or signed in | POST | yes | `product_template`, `product_template_attribute_value`, `is_product_configured` | Reports whether the option dialogue must be opened before adding a product, given the selected combination and whether it is already fully configured. | Electronic Commerce |
| `/website_sale/product_configurator/get_values` | `website_sale_product_configurator_get_values` | remote call | public or signed in | POST | yes | accepts further named parameters | Returns the configurator data of a product for the storefront, with storefront pricing and publication rules applied. | Electronic Commerce |
| `/website_sale/product_configurator/update_combination` | `website_sale_product_configurator_update_combination` | remote call | public or signed in | POST | yes | accepts further named parameters | Returns the refreshed configurator data after the selected combination changed, for the storefront. | Electronic Commerce |
| `/website_sale/product_configurator/get_optional_products` | `website_sale_product_configurator_get_optional_products` | remote call | public or signed in | POST | yes | accepts further named parameters | Returns the optional products offered with a combination, for the storefront. | Electronic Commerce |
| `/website_sale/product_configurator/create_product` | `website_sale_product_configurator_create_product` | remote call | public or signed in | POST | yes | accepts further named parameters | Creates the product variant of a combination that contains a dynamically created value, for the storefront. | Electronic Commerce |
| `/website_sale/combo_configurator/get_data` | `website_sale_combo_configurator_get_data` | remote call | public or signed in | POST | yes | accepts further named parameters | Returns the selection groups and items of a combination offer, for the storefront. | Electronic Commerce |
| `/website_sale/combo_configurator/get_price` | `website_sale_combo_configurator_get_price` | remote call | public or signed in | POST | yes | accepts further named parameters | Returns the price of a combination offer, for the storefront. | Electronic Commerce |
| `/sale/create_product_variant` | `create_product_variant` | remote call | public or signed in | POST | no | `product_template`, `product_template_attribute_value` (accepts further named parameters) | Creates the product variant matching a chosen combination of attribute values and returns its reference, for the storefront option dialogue. | Electronic Commerce |
| `/shop/compare` | `product_compare` | page or file | public or signed in | GET, POST | yes | accepts further named parameters | Serves the comparison page of the products the visitor added to the comparison list, showing their attributes side by side. | Product Comparison |
| `/shop/compare/get_product_data` | `get_product_data` | remote call | public or signed in | POST | yes | `product` | Returns the comparable attribute values and prices of the products in the comparison list. | Product Comparison |
| `/shop/products/recently_viewed_update` | `products_recently_viewed_update` | remote call | public or signed in | POST | yes | `product` (accepts further named parameters) | Records that the visitor looked at a product, which feeds the recently viewed block. | Electronic Commerce |
| `/shop/products/recently_viewed_delete` | `products_recently_viewed_delete` | remote call | public or signed in | POST | yes | `product`, `product_template` (accepts further named parameters) | Removes a product from the recently viewed list of the visitor. | Electronic Commerce |
| `/shop/save_shop_layout_mode` | `save_shop_layout_mode` | remote call | public or signed in | POST | yes | `layout_mode` | Remembers whether the catalogue is shown as a grid or as a list for the session. | Electronic Commerce |
| `/shop/add/stock_notification` | `add_stock_email_notification` | remote call | public or signed in | POST | yes | `email`, `product` | Registers the address of a visitor to be told when an out-of-stock product becomes available again. | Product Availability |
| `/gmc.xml` | `gmc_feed` | page or file | public or signed in | GET, POST | yes | `feed`, `access_token` | Serves the product feed of the shopping advertisement service: one entry per published product with its title, description, availability, price, images and links, built from the feed configuration and authorized by the feed access token. | Electronic Commerce |

### Cart

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/shop/cart` | `cart` | page or file | public or signed in | GET, POST | yes | `identifier`, `access_token`, `revive_method` (accepts further named parameters) | Serves the cart page with its lines, totals, suggested products and the checkout action, and handles the recovery of an abandoned cart addressed by its reference, access token and recovery method; the promotions package adds the reward lines and the code entry. | Electronic Commerce + Coupons, Promotions, Gift Card and Loyalty for Electronic Commerce |
| `/shop/cart/add` | `add_to_cart` | remote call | public or signed in | POST | yes | `product_template`, `product`, `quantity`, `unit_of_measure`, `product_custom_attribute_values`, `no_variant_attribute_value`, `linked_products` (accepts further named parameters) | Adds a product, with its chosen options, custom values, quantity, unit of measure and linked products, to the cart and returns the refreshed cart summary. | Electronic Commerce |
| `/shop/cart/quick_add` | `quick_add` | remote call | signed in | POST | yes | `product_template`, `product`, `quantity` (accepts further named parameters) | Adds one product to the cart without the option dialogue, for the listing buttons. | Electronic Commerce |
| `/shop/cart/update` | `update_cart` | remote call | public or signed in | POST | yes | `line`, `quantity`, `product` (accepts further named parameters) | Sets the quantity of one cart line; a quantity of zero or less deletes the line, because a storefront order does not hold negative quantities. | Electronic Commerce |
| `/shop/cart/quantity` | `cart_quantity` | remote call | public or signed in | POST | yes | none | Returns the number of items in the cart, for the cart counter of the page header. | Electronic Commerce |
| `/shop/cart/clear` | `clear_cart` | remote call | public or signed in | POST | yes | none | Empties the cart of the visitor. | Electronic Commerce |
| `/my/orders/reorder` | `my_orders_reorder` | remote call | public or signed in | POST | yes | `order`, `access_token` | Adds the products of a previous order to the cart, authorized by the access token of that order, and returns which products could be added and which could not. | Electronic Commerce |
| `/shop/pricelist` | `pricelist` | page or file | public or signed in | GET, POST | yes | `promo` (accepts further named parameters) | Applies a price list code typed by the visitor and reprices the cart; the promotions package first tries the code as a coupon or promotion code. | Electronic Commerce + Coupons, Promotions, Gift Card and Loyalty for Electronic Commerce |
| `/shop/change_pricelist/<pricelist:record of Pricelist>` | `pricelist_change` | page or file | public or signed in | GET, POST | yes | `pricelist` (accepts further named parameters) | Switches the visitor to another price list that is offered on the site and reprices the cart. | Electronic Commerce |
| `/shop/wishlist` | `get_wishlist` | page or file | public or signed in | GET, POST | yes | accepts further named parameters | Serves the wish list page of the visitor. | Shopper's Wishlist |
| `/shop/wishlist/add` | `add_to_wishlist` | remote call | public or signed in | POST | yes | `product` (accepts further named parameters) | Adds a product to the wish list of the visitor. | Shopper's Wishlist |
| `/shop/wishlist/remove/<wish_id:integer>` | `remove_from_wishlist` | remote call | public or signed in | POST | yes | `wish` (accepts further named parameters) | Removes one entry from the wish list. | Shopper's Wishlist |
| `/shop/wishlist/get_product_ids` | `shop_wishlist_get_product` | remote call | public or signed in | POST | yes | none | Returns the products currently in the wish list, for marking them on the catalogue pages. | Shopper's Wishlist |

### Checkout, delivery and payment

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/shop/checkout` | `shop_checkout` | page or file | public or signed in | GET | yes | `try_skip_step` (accepts further named parameters) | Serves the checkout page with the addresses and the delivery choice, and, when asked and when no input is missing, forwards straight to the next step; the Taiwanese invoicing package forces the step to be shown when the national invoice data is still missing. | Electronic Commerce + Taiwan Localization: Electronic Invoicing Electronic Commerce |
| `/shop/address` | `shop_address` | page or file | public or signed in | GET | yes | `partner`, `address_type`, `use_delivery_as_billing` (accepts further named parameters) | Serves the address form of the checkout for a new or an existing address of the requested type, optionally marking the delivery address as identical to the invoicing address. | Electronic Commerce |
| `/shop/address/submit` | `shop_address_submit` | page or file | public or signed in | POST | yes | `partner`, `address_type`, `use_delivery_as_billing`, `callback` (accepts further named parameters) | Creates or updates the checkout address and returns either the address to continue to or the invalid fields with their messages. | Electronic Commerce |
| `/shop/update_address` | `shop_update_address` | remote call | public or signed in | POST | yes | `partner`, `address_type` (accepts further named parameters) | Selects one of the existing addresses of the customer as the invoicing or the delivery address of the order. | Electronic Commerce |
| `/shop/delivery_methods` | `shop_delivery_methods` | remote call | public or signed in | POST | yes | none | Returns the delivery methods available for the cart, rendered as the delivery choice block. | Electronic Commerce |
| `/shop/get_delivery_rate` | `shop_get_delivery_rate` | remote call | public or signed in | POST | yes | `dm` | Returns the price, the currency and the promised delay of one delivery method for the current cart, asking the carrier service when the method is a carrier connection. | Electronic Commerce |
| `/shop/set_delivery_method` | `shop_set_delivery_method` | remote call | public or signed in | POST | yes | `dm` (accepts further named parameters) | Sets the delivery method on the order and returns the refreshed order totals; when the method is already set the totals are returned without recomputing. | Electronic Commerce |
| `/website_sale/get_pickup_locations` | `website_sale_get_pickup_locations` | remote call | public or signed in | POST | yes | `zip_code` (accepts further named parameters) | Returns the pickup points near a postal code for the chosen carrier, with the country taken from the network position of the visitor or from the delivery address; the collection package adds the shops of the company. | Electronic Commerce + Click & Collect |
| `/website_sale/set_pickup_location` | `website_sale_set_pickup_location` | remote call | public or signed in | POST | yes | `pickup_location_data` | Stores the chosen pickup point on the order during checkout. | Electronic Commerce |
| `/shop/set_click_and_collect_location` | `shop_set_click_and_collect_location` | remote call | public or signed in | POST | yes | `pickup_location_data` | Sets a shop as the collection point and the in-store delivery method on the cart, creating the cart when the visitor has none; this is the product page variant of the pickup choice. | Click & Collect |
| `/website_sale_mondialrelay/update_shipping` | `mondial_relay_update_shipping` | remote call | public or signed in | POST | yes | accepts further named parameters | Stores the relay point chosen in the widget of the relay network as the delivery address of the order. | Electronic Commerce Mondialrelay Delivery |
| `/shop/extra_info` | `extra_info` | page or file | public or signed in | GET, POST | yes | accepts further named parameters | Serves the extra information step of the checkout, which collects the additional fields the site asks for before payment. | Electronic Commerce |
| `/shop/payment` | `shop_payment` | page or file | public or signed in | GET, POST | yes | accepts further named parameters | Serves the payment step with the payment methods available for the order; a cart with no line returns to the catalogue and the session is cleared. | Electronic Commerce |
| `/shop/payment/transaction/<order_id:integer>` | `shop_payment_transaction` | remote call | public or signed in | POST | yes | `order`, `access_token` (accepts further named parameters) | Creates the draft transaction for the order and returns the values needed to continue the payment, verified by the access token. | Electronic Commerce |
| `/shop/payment/validate` | `shop_payment_validate` | page or file | public or signed in | GET, POST | yes | `sale_order` (accepts further named parameters) | Concludes the payment: it reads the transaction outcome, confirms the order when the outcome allows it, and redirects to the confirmation page. | Electronic Commerce |
| `/shop/confirmation` | `shop_payment_confirmation` | page or file | public or signed in | GET, POST | yes | accepts further named parameters | Serves the confirmation page of the finished order, clearing the checkout data from the session and reading the order by its reference rather than from the session. | Electronic Commerce |
| `/shop/print` | `print_saleorder` | page or file | public or signed in | GET, POST | yes | accepts further named parameters | Streams the printable form of the confirmed order to the customer. | Electronic Commerce |
| `/shop/express_checkout` | `process_express_checkout` | remote call | public or signed in | POST | yes | `billing_address`, `shipping_address`, `shipping_option` (accepts further named parameters) | Records the invoicing address, the delivery address and the delivery choice received from a wallet button on the order, creating the customer record when the visitor is anonymous or reusing the matching one. | Electronic Commerce |
| `/shop/express/shipping_address_change` | `express_checkout_process_delivery_address` | remote call | public or signed in | POST | yes | `partial_delivery_address` | Receives the partial delivery address a wallet button supplies, stores it and returns the delivery methods available for it; the promotions package reapplies the programmes that depend on the address. | Electronic Commerce + Coupons, Promotions, Gift Card and Loyalty for Electronic Commerce |
| `/shop/express/shipping_address_change/compute_taxes` | `express_checkout_shipping_address_compute_taxes` | remote call | public or signed in | POST | yes | none | Returns the recomputed tax amounts and order total for the partial address supplied by the wallet button. | Electronic Commerce |
| `/website_payment/snippet/supported_payment_methods` | `get_supported_payment_methods` | page or file | public or signed in | GET | yes | `limit` | Returns the payment methods of the providers published on the site, replacing a primary method by its brands, limited to the requested count, for the payment badge block. | Website Payment |
| `/donation/pay` | `donation_pay` | page or file | public or signed in | GET, POST | yes | accepts further named parameters | Serves the donation payment page with the suggested amounts and descriptions configured in the donation block. | Website Payment |
| `/donation/transaction/<minimum_amount>` | `donation_transaction` | remote call | public or signed in | POST | yes | `amount`, `currency`, `partner`, `access_token`, `minimum_amount` (accepts further named parameters) | Creates the draft transaction of a donation for the typed amount, refusing an amount below the minimum stated in the address. | Website Payment |
| `/website/form/shop.sale.order` | `website_form_saleorder` | page or file | public or signed in | POST | yes | accepts further named parameters | Accepts the public form submission that creates a quotation request (a Sales Order record) from the site and returns its reference. | Electronic Commerce |

### Storefront editing actions

These calls are available to a site editor while browsing the storefront; each one writes catalogue data from the page.

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/shop/config/website` | `change_website_configuration` | remote call | signed in | POST | no | accepts further named parameters | Changes the storefront display settings of the site (products per row and per page, default sort order, and which product features are shown); the wish list package adds its own switch to the same call. | Electronic Commerce + Shopper's Wishlist |
| `/shop/config/product` | `change_product_configuration` | remote call | signed in | POST | no | `product` (accepts further named parameters) | Changes the storefront settings of one product from its page (publication, display size in the grid, sequence and the storefront-only fields). | Electronic Commerce |
| `/shop/config/category` | `change_category_configuration` | remote call | signed in | POST | no | `category` (accepts further named parameters) | Changes the storefront settings of one catalogue category from the listing. | Electronic Commerce |
| `/shop/config/attribute` | `change_attribute_configuration` | remote call | signed in | POST | no | `attribute` (accepts further named parameters) | Changes how one product attribute is displayed in the catalogue filters. | Electronic Commerce |
| `/shop/product/extra-media` | `add_product_media` | remote call | signed in | POST | yes | `media`, `type`, `product_product`, `product_template`, `combination` | Adds images or videos to a product, a variant or a combination, linking each stored file to the product. | Electronic Commerce |
| `/shop/product/clear-images` | `clear_product_images` | remote call | signed in | POST | yes | `product_product`, `product_template` | Removes every image from a product or a variant. | Electronic Commerce |
| `/shop/product/resequence-image` | `resequence_product_image` | remote call | signed in | POST | yes | `image_res_model`, `image_res`, `move` | Moves one product image one position earlier or later and rewrites the sequence of all images of that product. | Electronic Commerce |
| `/snippets/category/set_image` | `set_category_image` | remote call | signed in | POST | no | `category`, `attachment` | Sets the cover image of a catalogue category from a stored attachment; a user without site editing rights is refused. | Electronic Commerce |

## Point of Sale

The point of sale endpoints serve three surfaces: the cashier station, the self-ordering surface (a customer device or a
standing kiosk) and the customer-facing second screen. The station and the self-ordering surface are long-running pages
that keep their data locally and synchronize through the generic record services; the endpoints below open those
surfaces, carry the self-ordering calls that must not require a session, and receive the notifications of payment
terminals and mobile payment services. A self-ordering call is authorized by the access token of the point of sale
configuration, and an order-specific call additionally by the access token of that order.

### Cashier station and customer display

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/pos/ui/<config_id>` <br> `/pos/ui/<config_id>/<subpath:path>` | `point_of_sale_web` | page or file | signed in | GET, POST | no | `point_of_sale_configuration`, `from_backend`, `subpath` (accepts further named parameters) | Opens the cashier station for one configuration: it selects the open session of the cashier, creates one when none is open, and serves the station page at the requested inner path. | Point of Sale |
| `/pos/web` <br> `/pos/ui` | `old_point_of_sale_web` | page or file | signed in | GET, POST | no | `point_of_sale_configuration`, `from_backend` (accepts further named parameters) | Serves the cashier station under its alternative addresses, with the same session selection. | Point of Sale |
| `/pos/ping` | `point_of_sale_ping` | remote call | signed in | POST | no | none | Answers a liveness probe of the station, which is how the station decides whether it is online or must work from its local data. | Point of Sale |
| `/pos/service-worker.js` | `point_of_sale_web_service_worker` | page or file | signed in | GET, POST | no | none | Returns the background script that keeps the station usable without a connection. | Point of Sale |
| `/web/image/pos.config/<id>/<field:text>` <br> `/web/image/pos.config/<id>/<field:text>/<width:integer>x<height:integer>` | `point_of_sale_content_image` | page or file | public or signed in | GET, POST | no | `field` (accepts further named parameters) | Streams an image field of a Point of Sale Configuration record (the station background or logo) at the requested size, without requiring a session. | Point of Sale |
| `/pos_customer_display/<id_>/<device_uuid>` | `point_of_sale_customer_display` | page or file | public or signed in | GET, POST | yes | `identifier`, `device_universally_unique_identifier` (accepts further named parameters) | Serves the customer-facing second screen of one station, addressed by the station reference and the device identity, showing the running order, its total and the payment state. | Point of Sale |
| `/pos/sale_details_report` | `print_sale_details` | page or file | signed in | GET, POST | no | `date_start`, `date_stop` (accepts further named parameters) | Streams the sales detail report of the stations over a date range as a printable document. | Point of Sale |
| `/pos/ticket` | `invoice_request_screen` | page or file | public or signed in | GET, POST | yes | accepts further named parameters | Serves the page where a customer types the receipt reference to request an invoice for a past ticket. | Point of Sale |
| `/pos/ticket/validate` | `show_ticket_validation_screen` | page or file | public or signed in | GET, POST | yes | `access_token` (accepts further named parameters) | Validates the receipt reference and access token, shows the ticket and, when the customer confirms, creates the invoice for it. | Point of Sale |

### Self-ordering and kiosk

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/pos-self/<config_id>` <br> `/pos-self/<config_id>/<subpath:path>` | `start_self_ordering` | page or file | public or signed in | GET, POST | yes | `point_of_sale_configuration`, `access_token`, `table_identifier`, `subpath` | Opens the self-ordering surface of one configuration, verified by its access token, for the scanned table when one is named, at the requested inner path. | Point of Sale Self Order |
| `/pos-self/data/<config_id>` | `get_self_ordering_data` | remote call | public or signed in | POST | yes | `point_of_sale_configuration`, `access_token`, `table_identifier` | Returns the data the self-ordering surface needs: the configuration, the product catalogue with prices and taxes, the categories, the payment methods and the table. | Point of Sale Self Order |
| `/pos-self/relations/<config_id>` | `get_self_ordering_relations` | remote call | public or signed in | POST | no | `point_of_sale_configuration`, `access_token`, `table_identifier` | Returns the field and relation descriptions of the records the self-ordering surface stores locally. | Point of Sale Self Order |
| `/pos-self/ping` | `point_of_sale_ping` | remote call | public or signed in | POST | no | `access_token` | Answers a liveness probe of the self-ordering surface. | Point of Sale Self Order |
| `/pos-self-order/process-order/<device_type>/` | `process_order` | remote call | public or signed in | POST | yes | `order`, `access_token`, `table_identifier`, `device_type` | Creates or updates the self-ordering order from the lines held on the device, for the given device type (customer device or kiosk), and returns the stored order with its computed taxes and totals. | Point of Sale Self Order |
| `/pos-self-order/get-order/<order_id:integer>` | `get_order` | remote call | public or signed in | POST | yes | `access_token`, `order`, `order_access_token` | Returns one self-ordering order and its state, verified by the order access token. | Point of Sale Self Order |
| `/pos-self-order/get-user-data` | `get_orders_by_access_token` | remote call | public or signed in | POST | yes | `access_token`, `order_access_tokens`, `table_identifier` | Returns the orders that belong to the tokens held by the device, and the table data, therefore a customer sees the history of the visit. | Point of Sale Self Order |
| `/pos-self-order/remove-order` | `remove_order` | remote call | public or signed in | POST | yes | `access_token`, `order`, `order_access_token` | Cancels a self-ordering order that has not been sent to preparation. | Point of Sale Self Order |
| `/pos-self-order/get-slots` | `get_slots` | remote call | public or signed in | POST | yes | `access_token`, `preset` | Returns the collection or delivery time slots still open for the chosen preset, with their remaining capacity. | Point of Sale Self Order |
| `/pos-self-order/validate-partner` | `validate_partner` | remote call | public or signed in | POST | yes | `access_token`, `name`, `phone`, `street`, `postal_code`, `city`, `country`, `country_subdivision`, `partner`, `email` | Validates the contact details typed by a self-ordering customer (name, telephone, address, postal code, city, country, state and electronic mail address) and creates or updates the contact record. | Point of Sale Self Order |
| `/pos-self-order/send_self_order_receipt` | `send_self_order_receipt` | remote call | public or signed in | POST | yes | `access_token`, `order`, `order_access_token`, `fullticketimage`, `basicticketimage` | Sends the receipt of a self-ordering order to the address of the customer, with the full image and the plain image of the ticket. | Point of Sale Self Order |
| `/pos_self_order/kiosk/increment_nb_print/` | `point_of_sale_kiosk_increment_number_of_print` | remote call | public or signed in | POST | yes | `access_token`, `order`, `order_access_token` | Counts one more printed ticket for a kiosk order, which limits reprinting. | Point of Sale Self Order |
| `/pos-self-order/change-printer-status` | `change_printer_status` | remote call | public or signed in | POST | yes | `access_token`, `has_paper` | Records whether the kiosk printer still has paper, which switches the kiosk to screen-only receipts when it does not. | Point of Sale Self Order |
| `/kiosk/payment/<pos_config_id:integer>/<device_type>` | `point_of_sale_self_order_kiosk_payment` | remote call | public or signed in | POST | yes | `point_of_sale_configuration`, `order`, `payment_method`, `access_token`, `device_type` | Starts the payment of a kiosk order on the chosen payment method and device type and returns the data the payment device needs. | Point of Sale Self Order |

### Point of sale payments

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/pos/pay/<pos_order_id:integer>` | `point_of_sale_order_pay` | page or file | public or signed in | GET | yes | `point_of_sale_order`, `access_token`, `exit_route` | Serves the online payment page of a point of sale order, verified by its access token, with the address to return to when the payment ends; the self-ordering package adds the self-ordering layout. | Point of Sale online payment + Point of Sale Self-Order / Online Payment |
| `/pos/pay/transaction/<pos_order_id:integer>` | `point_of_sale_order_pay_transaction` | remote call | public or signed in | POST | yes | `point_of_sale_order`, `access_token` (accepts further named parameters) | Creates the draft transaction for a point of sale order and returns the values needed to continue the payment. | Point of Sale online payment |
| `/pos/pay/confirmation/<pos_order_id:integer>` | `point_of_sale_order_pay_confirmation` | page or file | public or signed in | GET | yes | `point_of_sale_order`, `tx`, `access_token`, `exit_route` (accepts further named parameters) | Serves the confirmation page of the online payment of a point of sale order and tells the station that the payment concluded. | Point of Sale online payment + Point of Sale Self-Order / Online Payment |
| `/pos_adyen/notification` | `notification` | remote call | public or signed in | POST | no | none | Receives the terminal notifications of the first terminal provider and routes the outcome to the station that owns the payment line. | Point of Sale Adyen |
| `/pos_mercado_pago/notification` | `notification` | page or file | none | POST | no | none | Receives the structured notifications of the Mercado Pago terminal service and routes the outcome to the station. | Point of Sale Mercado Pago |
| `/pos_mollie/webhook` | `mollie_webhook` | page or file | public or signed in | POST | no | `identifier`, `payload` | Receives the notifications of the Mollie terminal service for one payment and routes the outcome to the station. | Point of Sale Mollie |
| `/pos_viva_com/notification` | `notification` | page or file | none | GET, POST | no | `company`, `token` | Receives the notifications of the Viva.com terminal service for one company, verified by the stored token, and routes the outcome to the station. | Point of Sale Viva.com |
| `/pos_self_order_viva_com/poll_payment` | `poll_payment` | remote call | public or signed in | POST | no | `access_token`, `viva_session`, `payment_method` | Returns the current state of a Viva.com payment for a self-ordering device that polls instead of receiving notifications. | Point of Sale Self Order Viva.com |
| `/pos-self-order/stripe-connection-token/` | `get_stripe_creditentials` | remote call | public or signed in | POST | yes | `access_token`, `payment_method` | Returns the short-lived connection token a Stripe card reader needs to register with the service. | Point of Sale Self Order Stripe |
| `/pos-self-order/stripe-capture-payment/` | `stripe_capture_payment` | remote call | public or signed in | POST | yes | `access_token`, `order_access_token`, `payment_intent`, `payment_method` | Captures the authorized Stripe payment of a self-ordering order and applies the outcome. | Point of Sale Self Order Stripe |
| `/pos-self-order/razorpay-fetch-payment-status/` | `razorpay_payment_status` | remote call | public or signed in | POST | yes | `access_token`, `order`, `payment_data`, `payment_method` | Returns the current state of a Razorpay terminal payment for a self-ordering order. | Point of Sale Self Order Razorpay |
| `/pos-self-order/razorpay-cancel-transaction/` | `razorpay_cancel_status` | remote call | public or signed in | POST | yes | `access_token`, `order`, `payment_data`, `payment_method` | Cancels a pending Razorpay terminal payment of a self-ordering order. | Point of Sale Self Order Razorpay |
| `/pos-self-order/pine-labs-fetch-payment-status/` | `pine_labs_fetch_payment_status` | remote call | public or signed in | POST | no | `access_token`, `order`, `payment_data`, `payment_method` | Returns the current state of a Pine Labs terminal payment for a self-ordering order. | Point of Sale Self Order Pine Labs |
| `/pos-self-order/pine-labs-cancel-transaction/` | `pine_labs_cancel_transaction` | remote call | public or signed in | POST | no | `access_token`, `order`, `payment_data`, `payment_method` | Cancels a pending Pine Labs terminal payment of a self-ordering order. | Point of Sale Self Order Pine Labs |
| `/pos_safaricom/callback` | `safaricom_callback` | page or file | public or signed in | POST | no | `payload` | Receives the result of a mobile money payment request sent to the customer telephone and applies it to the payment line. | Point of Sale Safaricom |
| `/c2b/validation/callback` | `c2b_validation_callback` | page or file | public or signed in | POST | no | `payload` | Answers the validation request of the mobile money service before the customer is charged; every request is accepted, with the completion answer the service expects. | Point of Sale Safaricom |
| `/c2b/confirmation/callback` | `c2b_confirmation_callback` | page or file | public or signed in | POST | no | `payload` | Receives the confirmation of a customer-initiated mobile money payment and matches it to the open order. | Point of Sale Safaricom |
| `/qfpay/notify` | `qfpay_notify` | page or file | public or signed in | POST | no | accepts further named parameters | Receives the asynchronous payment and refund notifications of the QFPay service, verifies their signature, applies the outcome and answers with the string "SUCCESS"; an invalid signature is answered with the bad-request status. | Point of Sale QFPay |

## Events

The event endpoints serve the published event pages, the registration path with its slots and tickets, the attendee
badges and calendar files, the talk programme with its proposals, reminders and quizzes, the exhibitor pages, and the
booth sales path. A registration link sent by electronic mail is authorized by a fingerprint computed over the
registration set rather than by a session.

### Event pages and registration

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/event` <br> `/event/page/<page:integer>` <br> `/event/tags/<slug_tags:text>` <br> `/event/tags/<slug_tags:text>/page/<page:integer>` <br> `/events` <br> `/events/page/<page:integer>` <br> `/events/tags/<slug_tags:text>` <br> `/events/tags/<slug_tags:text>/page/<page:integer>` | `events` | page or file | public or signed in | GET, POST | yes | `page`, `slug_tags` (accepts further named parameters) | Lists the published events, filtered by tags, by country, by date range and by typed text, with paging, under both address spellings. | Events |
| `/event/<event:record of Event>` | `event` | page or file | public or signed in | GET, POST | yes | `event` (accepts further named parameters) | Serves the home page of one event with its description, dates, place, sponsors and the registration action. | Events |
| `/event/<event:record of Event>/page/<page:path>` | `event_page` | page or file | public or signed in | GET, POST | yes | `event`, `page` (accepts further named parameters) | Serves one of the additional pages an organizer authored for an event. | Events |
| `/event/<event:record of Event>/register` | `event_register` | page or file | public or signed in | GET, POST | yes | `event` (accepts further named parameters) | Serves the registration page of an event with its time slots and ticket types. | Events |
| `/event/<event:record of Event>/registration/slot/<slot_id:integer>/tickets` | `registration_tickets` | remote call | public or signed in | POST | yes | `event`, `slot` | Returns the ticket selection for a chosen time slot, limited by the seats still available in that slot and for each ticket type. | Events |
| `/event/<event:record of Event>/registration/new` | `registration_new` | remote call | public or signed in | POST | yes | `event` (accepts further named parameters) | Returns the attendee detail form for the chosen tickets, one block per seat. | Events |
| `/event/<event:record of Event>/registration/confirm` | `registration_confirm` | page or file | public or signed in | POST | yes | `event` (accepts further named parameters) | Creates the registrations after checking once more that enough seats remain, and returns the registration page with a formatted message when they do not; the ticketing package turns paid tickets into a cart instead of confirming at once. | Events + Online Event Ticketing |
| `/event/<event:record of Event>/registration/success` | `event_registration_success` | page or file | public or signed in | GET | yes | `event`, `registration` | Serves the page shown after a successful registration, with the attendee details and the links to the badge and the calendar file. | Events |
| `/event/<event_id:integer>/my_tickets` | `event_my_tickets` | page or file | public or signed in | GET, POST | no | `event`, `registration`, `tickets_hash`, `badge_mode`, `responsive_rich_text` | Streams the tickets of the named registrations of one event as a printable document, in badge layout or in ticket layout; a missing, invalid or mismatched fingerprint is refused with the forbidden status. | Events Organization |
| `/event/<event:record of Event>/ics` | `event_ics_file` | page or file | public or signed in | GET, POST | yes | `event` (accepts further named parameters) | Streams the calendar file of an event for adding it to a personal calendar. | Events Organization |
| `/event/init_barcode_interface` | `init_barcode_interface` | remote call | signed in | POST | no | `event` | Returns the data the attendance scanning surface needs for one event: the registration set, the ticket types and the scanning rules. | Events Organization |
| `/event/manifest.webmanifest` | `webmanifest` | page or file | public or signed in | GET | yes | none | Returns the installation descriptor that lets the event site be installed as a standalone application. | Advanced Events |
| `/event/service-worker.js` | `service_worker` | page or file | public or signed in | GET | yes | none | Returns the background script scoped to the event site. | Advanced Events |
| `/event/offline` | `offline` | page or file | public or signed in | GET | yes | none | Serves the page shown by that background script when the device has no connection. | Advanced Events |

### Talks, proposals, reminders and quizzes

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/event/<event:record of Event>/track` <br> `/event/<event:record of Event>/track/tag/<tag:record of Event Track Tag>` | `event_tracks` | page or file | public or signed in | GET, POST | yes | `event`, `tag` (accepts further named parameters) | Lists the talks of an event, filtered by tag, by typed text and by day, with the programme layout. | Advanced Events |
| `/event/<event:record of Event>/track/<track:record of Event Track>` | `event_track_page` | page or file | public or signed in | GET, POST | yes | `event`, `track` (accepts further named parameters) | Serves the page of one talk with its speakers, time, room, description, the live stream when one runs, and the discussion thread. | Advanced Events |
| `/event/<event:record of Event>/agenda` | `event_agenda` | page or file | public or signed in | GET, POST | yes | `event`, `tag` (accepts further named parameters) | Serves the programme grid of an event by day and room, filtered by tag. | Advanced Events |
| `/event/<event:record of Event>/track/<track:record of Event Track>/ics` | `event_track_ics_file` | page or file | public or signed in | GET, POST | yes | `event`, `track` | Streams the calendar file of one talk. | Advanced Events |
| `/event/track/toggle_reminder` | `track_reminder_toggle` | remote call | public or signed in | POST | yes | `track`, `set_reminder_on` | Sets or clears the reminder of one talk for the current visitor, creating the visitor record when it does not exist yet. | Advanced Events |
| `/event/track/send_email_reminder` | `send_email_reminder` | remote call | public or signed in | POST | yes | `track`, `email_to` | Sends the talk reminders with their calendar files to the address supplied by an anonymous visitor, or to the address of the signed-in user. | Advanced Events |
| `/event/<event:record of Event>/track_proposal` | `event_track_proposal` | page or file | public or signed in | GET, POST | yes | `event` (accepts further named parameters) | Serves the talk proposal form of an event. | Advanced Events |
| `/event/<event:record of Event>/track_proposal/post` | `event_track_proposal_post` | page or file | public or signed in | POST | yes | `event` (accepts further named parameters) | Creates the proposed talk with its speaker details, biography, title, description and tags, and returns the acknowledgement. | Advanced Events |
| `/event/track_tag/search_read` | `website_event_track_fetch_tags` | remote call | public or signed in | POST | yes | `domain`, `fields` | Returns the talk tags matching a filter condition, for the tag pickers. | Advanced Events |
| `/event_track/get_track_suggestion` | `get_next_track_suggestion` | remote call | public or signed in | POST | yes | `track` | Returns the next talk suggested to a visitor who is finishing one, used by the live programme. | Live Event Tracks |
| `/event/<event:record of Event>/community` | `community` | page or file | public or signed in | GET, POST | yes | `event`, `language` (accepts further named parameters) | Serves the community page of an event; with the quiz package installed it shows the quiz progress and the leaderboard entry point. | Events + Quizzes on Tracks |
| `/event/<event:record of Event>/community/leaderboard` | `community_leaderboard` | page or file | public or signed in | GET, POST | yes | `event` (accepts further named parameters) | Serves the leaderboard page of the talk quizzes of an event. | Quizzes on Tracks |
| `/event/<event:record of Event>/community/leaderboard/results` <br> `/event/<event:record of Event>/community/leaderboard/results/page/<page:integer>` | `leaderboard` | page or file | public or signed in | GET, POST | yes | `event`, `page`, `language` (accepts further named parameters) | Returns a page of the leaderboard ranking, in the requested language. | Quizzes on Tracks |
| `/event_track/quiz/submit` | `event_track_quiz_submit` | remote call | public or signed in | POST | yes | `event`, `track`, `answer` | Grades the quiz answers of a talk, returns the correct answers and the points earned, and updates the leaderboard position of the visitor. | Quizzes on Tracks |
| `/event_track/quiz/reset` | `quiz_reset` | remote call | public or signed in | POST | yes | `event`, `track` | Clears the quiz attempt of the visitor on one talk, therefore it can be taken again. | Quizzes on Tracks |

### Exhibitors and booths

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/event/<event:record of Event>/exhibitors` <br> `/event/<event:record of Event>/exhibitor` | `event_exhibitors` | page or file | public or signed in | GET, POST | yes | `event` (accepts further named parameters) | Lists the exhibitors of an event, filtered by country, by sponsorship level and by typed text. | Event Exhibitors |
| `/event/<event:record of Event>/exhibitor/<sponsor:record of Event Sponsor>` | `event_exhibitor` | page or file | public or signed in | GET, POST | yes | `event`, `sponsor` (accepts further named parameters) | Serves the page of one exhibitor with its description, its contact form and, during the event, its live conversation room. | Event Exhibitors |
| `/event_sponsor/<sponsor_id:integer>/read` | `event_sponsor_read` | remote call | public or signed in | POST | yes | `sponsor` | Returns the exhibitor data for the dialogue shown when the event has not started or the exhibitor is not available yet. | Event Exhibitors |
| `/event/<event:record of Event>/booth` | `event_booth_main` | page or file | public or signed in | GET, POST | yes | `event`, `booth_category`, `booth` | Serves the booth sales page of an event with the booth categories and the floor plan. | Online Event Booths |
| `/event/<event:record of Event>/booth/register` | `event_booth_register` | page or file | public or signed in | POST | yes | `event`, `booth_category`, `event_booth` | Serves the booth selection step for a chosen category with the booths still free. | Online Event Booths |
| `/event/<event:record of Event>/booth/register_form` | `event_booth_contact_form` | page or file | public or signed in | GET | yes | `event`, `booth`, `booth_category` | Returns the contact form for the chosen booths. | Online Event Booths |
| `/event/<event:record of Event>/booth/confirm` | `event_booth_registration_confirm` | page or file | public or signed in | POST | yes | `event`, `booth_category`, `event_booth` (accepts further named parameters) | Confirms the booth reservation with the submitted contact details; the booth sales package creates the quotation and continues to the payment instead of confirming directly. | Online Event Booths + Online Event Booth Sale |
| `/event/booth/check_availability` | `check_booths_availability` | remote call | public or signed in | POST | no | `event_booth` | Reports whether the chosen booths are still free, which the page checks before submitting. | Online Event Booths |
| `/event/booth_category/get_available_booths` | `get_booth_category_available_booths` | remote call | public or signed in | POST | no | `event`, `booth_category` | Returns the free booths of one category with their price and position. | Online Event Booths |

## Marketing Campaigns

The marketing endpoints are almost entirely public links that appear inside sent messages: the open tracker, the shortened
links that count clicks, the unsubscribe and block list actions, the browser view of a message, and the personalized
card images shared on social networks. Each of them is authorized by a fingerprint computed over the mailing, the
recipient address and the document, never by a session, because the reader of a message is not signed in.

### Message links, tracking and shortened addresses

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/mail/track/<mail_id:integer>/<token:text>/blank.gif` | `track_mail_open` | page or file | public or signed in | GET, POST | no | `mail`, `token` (accepts further named parameters) | Returns a one-pixel image and records that the message was opened, verified by the token embedded in the image address. | Email Marketing |
| `/r` | `shorten_web_address` | page or file | signed in | GET, POST | yes | accepts further named parameters | Creates a shortened address for a target address and returns its code. | Link Tracker |
| `/website_links/new` | `create_shorten_web_address` | remote call | signed in | POST | no | accepts further named parameters | Creates a shortened address with its campaign, source and medium labels from the tracking link editor. | Link Tracker |
| `/website_links/add_code` | `add_code` | remote call | signed in | POST | no | accepts further named parameters | Replaces the automatically generated code of a shortened address with a chosen one, refusing a code already in use. | Link Tracker |
| `/website_links/recent_links` | `recent_links` | remote call | signed in | POST | no | accepts further named parameters | Returns the shortened addresses created recently with their click counts, for the tracking link editor. | Link Tracker |
| `/r/<code:text>` | `full_web_address_redirect` | page or file | public or signed in | GET, POST | yes | `code` (accepts further named parameters) | Redirects a shortened address to its target and counts the click with the position and device of the visitor. | Link Tracker |
| `/r/<code:text>+` | `statistics_shorten_web_address` | page or file | signed in | GET, POST | yes | `code` (accepts further named parameters) | Serves the statistics page of one shortened address: total clicks, clicks over time and clicks by country. | Link Tracker |
| `/r/<code:text>/m/<mailing_trace_id:integer>` | `full_web_address_redirect` | page or file | public or signed in | GET, POST | no | `code`, `mailing_trace` (accepts further named parameters) | Redirects a shortened address that was placed in one electronic mail message and records the click against that message. | Email Marketing |
| `/r/<code:text>/s/<sms_id_int:integer>` | `text_message_short_link_redirect` | page or file | public or signed in | GET, POST | no | `code`, `text_message_identifier_int` (accepts further named parameters) | Redirects a shortened address that was placed in one text message and records the click against that message. | Text Message Marketing |

### Subscription, block list and feedback

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/mailing/<mailing_id:integer>/unsubscribe` | `mailing_unsubscribe` | page or file | public or signed in | GET, POST | yes | `mailing`, `document`, `email`, `hash_token` | Serves the unsubscribe page of a mailing for the recipient address, verified by the fingerprint, showing the mailing lists the address belongs to and the block list action. | Email Marketing |
| `/mail/mailing/<mailing_id:integer>/unsubscribe` | `mailing_unsubscribe` | page or file | public or signed in | GET, POST | yes | `mailing`, `email`, `related_record_identifier`, `token` (accepts further named parameters) | Serves the same unsubscribe page under the second address spelling, accepting the alternative parameter names. | Email Marketing |
| `/mailing/<mailing_id:integer>/confirm_unsubscribe` | `mailing_confirm_unsubscribe` | page or file | public or signed in | GET, POST | yes | `mailing`, `document`, `email`, `hash_token` | Performs the unsubscription for the recipient address after the confirmation step and shows the result. | Email Marketing |
| `/mailing/confirm_unsubscribe` | `mailing_confirm_unsubscribe_post` | page or file | public or signed in | POST | yes | `mailing`, `document`, `email`, `hash_token` | Performs the same confirmed unsubscription from a submitted form rather than a link. | Email Marketing |
| `/mailing/<mailing_id:integer>/unsubscribe_oneclick` | `mailing_unsubscribe_oneclick` | page or file | public or signed in | POST | yes | `mailing`, `document`, `email`, `hash_token` (accepts further named parameters) | Unsubscribes the recipient address through the one-click unsubscribe header of the message, accepting only submissions. | Email Marketing |
| `/mailing/list/update` | `mailing_update_list_subscription` | remote call | public or signed in | POST | no | `mailing`, `document`, `email`, `hash_token`, `lists_optin` (accepts further named parameters) | Stores the mailing lists the recipient chose to keep or to leave on the unsubscribe page. | Email Marketing |
| `/mailing/blocklist/add` | `mail_blocklist_add` | remote call | public or signed in | POST | no | `mailing`, `document`, `email`, `hash_token` | Adds the recipient address to the block list, which stops every future mailing to it. | Email Marketing |
| `/mailing/blocklist/remove` | `mail_blocklist_remove` | remote call | public or signed in | POST | no | `mailing`, `document`, `email`, `hash_token` | Removes the recipient address from the block list. | Email Marketing |
| `/mailing/feedback` | `mailing_send_feedback` | remote call | public or signed in | POST | no | `mailing`, `document`, `email`, `hash_token`, `last_action`, `opt_out_reason`, `feedback` (accepts further named parameters) | Stores the reason and the free text the recipient gave after unsubscribing or after being added to the block list, writing them on the most relevant record. | Email Marketing |
| `/mailing/my` | `mailing_my` | page or file | signed in | GET, POST | yes | none | Serves the page where a signed-in contact manages every mailing list subscription it holds. | Email Marketing |
| `/mailing/<mailing_id:integer>/view` | `mailing_view_inch_browser` | page or file | public or signed in | GET, POST | yes | `mailing`, `email`, `document`, `hash_token` (accepts further named parameters) | Renders the sent message as a page for the recipient who chose to view it in a browser. | Email Marketing |
| `/view` | `mailing_view_inch_browser_placeholder_link` | page or file | signed in | GET, POST | yes | none | Serves the sample page behind the browser-view placeholder link while the message is being edited. | Email Marketing |
| `/unsubscribe_from_list` | `mailing_unsubscribe_placeholder_link` | page or file | public or signed in | GET, POST | yes | accepts further named parameters | Serves the sample page behind the unsubscribe placeholder link while the message is being edited, with no language prefix added to the address. | Email Marketing |
| `/mailing/report/unsubscribe` | `mailing_report_deactivate` | page or file | public or signed in | GET, POST | yes | `token`, `user` | Stops the periodic mailing performance report for the recipient user, verified by the token. | Email Marketing |
| `/mailing/mobile/preview` | `mass_mailing_preview_mobile_content` | page or file | signed in | GET | yes | none | Returns the message rendered at telephone width for the preview panel of the editor. | Email Marketing |
| `/sms/<mailing_id:integer>/<trace_code:text>` | `blacklist_page` | page or file | public or signed in | GET, POST | yes | `mailing`, `trace_code` (accepts further named parameters) | Serves the unsubscribe page of a text message campaign after verifying that the trace code matches the campaign and the message, and that the number can be normalized. | Text Message Marketing |
| `/sms/<mailing_id:integer>/unsubscribe/<trace_code:text>` | `blacklist_number` | page or file | public or signed in | GET, POST | yes | `mailing`, `trace_code` (accepts further named parameters) | Adds the telephone number of the recipient to the block list and shows the confirmation. | Text Message Marketing |
| `/website_mass_mailing/is_subscriber` | `is_subscriber` | remote call | public or signed in | POST | yes | `list`, `subscription_type` (accepts further named parameters) | Reports whether the visitor is already subscribed to a mailing list, for the subscription block. | Newsletter Subscribe Button |
| `/website_mass_mailing/subscribe` | `subscribe` | remote call | public or signed in | POST | yes | `list`, `value`, `subscription_type` (accepts further named parameters) | Subscribes the typed electronic mail address or telephone number to a mailing list from the subscription block and returns the result state. | Newsletter Subscribe Button |

### Personalized campaign cards

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/cards/<card_slug:text>/card.jpg` <br> `/cards/<card_id:integer>/card.jpg` | `card_campaign_image` | page or file | public or signed in | GET, POST | yes | `card`, `card_slug` | Streams the personalized image of one campaign card, rendered from the card template and the data of its recipient. | Marketing Card |
| `/cards/<card_slug:text>/preview` <br> `/cards/<card_id:integer>/preview` | `card_campaign_preview` | page or file | public or signed in | GET, POST | yes | `card`, `card_slug` | Serves the page where the recipient sees its personalized card and shares it on social networks. | Marketing Card |
| `/cards/<card_slug:text>/redirect` <br> `/cards/<card_id:integer>/redirect` | `card_campaign_redirect` | page or file | public or signed in | GET, POST | yes | `card`, `card_slug` | Redirects a card link to the campaign target address for a person, and answers a link-preview crawler with the sharing metadata of the card instead. | Marketing Card |

## Automation and Integration Services

This group holds the entry points that belong to no single business domain: the alternative remote call protocols, the
reflection service that describes entities to an integrator, the inbound automation hook, the data import upload, the
authorization callbacks of the external account providers, and the hardware bridge of the point of sale.

### Alternative remote call protocols

These endpoints expose the same operations as the desktop client services under other envelopes, for integrations written
in other languages. The envelopes themselves are specified in
[`remote-transport-contracts.md`](remote-transport-contracts.md).

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/jsonrpc` | `remote_procedure_call` | remote call | none | POST | no | `service`, `method`, `args` | Runs one call of the named service and operation with the supplied arguments under the structured remote call envelope; the services are the authentication service, the record service and the installation administration service. | Remote Procedure Call endpoints |
| `/xmlrpc/2/<service>` | `remote_procedure_call_2` | page or file | none | POST | no | `service` | Runs one call of the named service under the markup remote call envelope, answering faults with a numeric fault code. | Remote Procedure Call endpoints |
| `/xmlrpc/<service>` | `remote_procedure_call_1` | page or file | none | POST | no | `service` | Runs one call of the named service under the markup remote call envelope, answering faults with a textual fault code. | Remote Procedure Call endpoints |
| `/json/2/<__model__>/<__method__>` | `web_structured_data_2_remote_procedure_call` | structured call | application key | POST | no | `model`, `method`, `ids`, `context` (accepts further named parameters) | Runs one operation of one entity over the direct structured protocol: the request body is the argument document itself, the answer is the raw result, the caller is identified by an application key, and the record selection and display context are part of the body. | Remote Procedure Call endpoints |
| `/json/2` <br> `/json/2/<subpath:path>` | `web_structured_data_2_404` | structured call | public or signed in | GET, POST, PUT, DELETE, PATCH | no | `subpath` | Answers every other path under the direct structured protocol with the not-found status and a structured error body, therefore a mistyped entity or operation gives a machine-readable answer. | Remote Procedure Call endpoints |
| `/web/version` <br> `/json/version` | `version` | page or file | none | GET, POST | no | none | Returns the version descriptor of the installation to an integration, under both address spellings. | Remote Procedure Call endpoints |

### Reflection service

The reflection service lets an integrator discover entities, fields and operations without reading any source. It is the
machine-readable counterpart of the entity catalogue.

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/doc` <br> `/doc/<model_name>` <br> `/doc/index.html` | `doc_client` | page or file | signed in | GET, POST | no | `mod` (accepts further named parameters) | Serves the browsable reference of the entities of the installation, optionally limited to one capability package or opened on one entity. | Application Programming Interface Documentation |
| `/doc/index.json` | `doc_index` | structured call | signed in | GET, POST | no | none | Returns the index of capability packages, entities, operations and fields with their technical and translated names. | Application Programming Interface Documentation |
| `/doc/<model_name>.json` | `doc_model` | structured call | signed in | GET, POST | no | `model_name` | Returns the complete description of one entity: its documentation text, the full field descriptions and, for every operation, its signature, parameters and documentation text. | Application Programming Interface Documentation |
| `/doc-bearer/index.json` | `doc_bearer_index` | structured call | application key | GET, POST | no | none | Returns the same index, authenticated by an application key instead of a session. | Application Programming Interface Documentation |
| `/doc-bearer/<model_name>.json` | `doc_bearer_modec` | structured call | application key | GET, POST | no | `model_name` | Returns the same entity description, authenticated by an application key instead of a session. | Application Programming Interface Documentation |

### Inbound automation and data import

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/web/hook/<rule_uuid:text>` | `call_webhook_hypertext_transfer_protocol` | page or file | public or signed in | GET, POST | no | `rule_universally_unique_identifier` (accepts further named parameters) | Runs the automation rule whose identity is in the address with the received body as its payload; this is the inbound hook by which a foreign system triggers work in the platform. | Automation Rules |
| `/base_import/set_file` | `set_file` | page or file | signed in | POST | no | `identifier` | Receives the uploaded file of an import and attaches it to the import session, which the mapping step then reads. | Base import |
| `/base_import_module/login_upload` | `login_upload` | page or file | none | POST | no | `login_name`, `password`, `force`, `mod_file` (accepts further named parameters) | Signs in with the supplied credentials and installs or updates the capability package contained in the uploaded archive, replacing an existing one when the replace flag is set. | Base import module |

### External account authorization callbacks

Each of these endpoints receives the browser back from an external provider after the account holder granted or refused
consent, and stores the resulting credentials on the record that requested them.

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/google_account/authentication` | `oauth2callback` | page or file | public or signed in | GET, POST | no | accepts further named parameters | Receives the consent decision of the first external account provider and stores the resulting credentials for the service that requested them. | Google Users |
| `/microsoft_account/authentication` | `oauth2callback` | page or file | public or signed in | GET, POST | no | accepts further named parameters | Receives the consent decision of the second external account provider and stores the resulting credentials. | Microsoft Users |
| `/google_gmail/confirm` | `google_gmail_callback` | page or file | signed in | GET, POST | no | `code`, `state`, `error` (accepts further named parameters) | Receives the authorization code for a mail server of the first provider, exchanges it for a refresh token and an access token, and stores both on that mail server. | Google Gmail |
| `/google_gmail/iap_confirm` | `google_gmail_in_application_purchase_callback` | page or file | signed in | GET, POST | no | `model_name`, `record`, `csrf_token`, `access_token`, `refresh_token`, `expiration` | Receives the refresh token, the access token and the expiry back from the publisher proxy for a mail server of the first provider, after verifying the anti-forgery token. | Google Gmail |
| `/microsoft_outlook/confirm` | `microsoft_outlook_callback` | page or file | signed in | GET, POST | no | `code`, `state`, `error_description` (accepts further named parameters) | Receives the authorization code for a mail server of the second provider, exchanges it for a refresh token and an access token, and stores both on that mail server. | Microsoft Outlook |
| `/microsoft_outlook/iap_confirm` | `microsoft_outlook_in_application_purchase_callback` | page or file | signed in | GET, POST | no | `model_name`, `record`, `csrf_token`, `access_token`, `refresh_token`, `expiration` | Receives the refresh token, the access token and the expiry back from the publisher proxy for a mail server of the second provider, after verifying the anti-forgery token. | Microsoft Outlook |

### Hardware bridge

| Path patterns | Operation | Transport | Authentication | Methods | Site page | Parameters | Purpose and effect | Capability package |
|---|---|---|---|---|---|---|---|---|
| `/hw_proxy/scale_read` | `scale_read` | remote call | none | POST | no | none | Returns the current reading of the connected weighing scale to the point of sale station. | Hardware Proxy |

## Reconciliation notes

The two drafts that were merged into this catalog disagreed on a small number of points. Each disagreement was settled
against the source of the system and against the condensed catalog
[`../references/routes.md`](../references/routes.md); the resolutions are recorded here.

1. **Endpoint and path counts.** One draft stated 906 handlers on 855 distinct endpoints with 51 folded extension
   layers, and elsewhere 448 page-or-file and 406 remote-call endpoints, which do not add up. The counts were recomputed
   from the rows of this catalog and from the condensed catalog: 855 endpoints, 1029 path patterns, 446 page-or-file,
   403 remote-call and 6 structured-call endpoints, and seven rows carrying fourteen extension layers in total.
2. **Authentication distribution.** The signed-in count was stated as 316 and the plug-in key count as 14. Both included
   endpoints that the system keeps only for superseded plug-in versions and that this specification therefore does not
   describe. The corrected counts are 314 signed in and 11 plug-in key.
3. **Cross-site submission protection.** One draft stated that 794 endpoints verify the anti-forgery token and 66 do
   not. That pair of numbers counted the endpoints that leave the setting at its default rather than the endpoints that
   perform the check. The rule is restated by transport: the check applies to the page-or-file transport on every method
   that is not a reading method, 65 page-or-file endpoints switch it off, the remote call transports do not use the
   token, two remote call endpoints switch it on and one switches it off.
4. **The desktop client entry path.** One draft wrote the desktop client entry path as `/system` and its offline page as
   `/system/offline`. The path published by the system, and the one carried by the condensed catalog and by
   [`remote-transport-contracts.md`](remote-transport-contracts.md), is `/app`, with `/app/<subpath:path>` and
   `/app/offline`. This catalog now uses `/app`.
5. **Endpoints kept only for superseded plug-in clients.** Five endpoints of the electronic mail plug-in
   (`/mail_client_extension/modules/get`, `/mail_client_extension/lead/get_by_partner_id`,
   `/mail_client_extension/lead/open`, `/mail_client_extension/lead/create_from_partner` and
   `/mail_client_extension/log_single_mail_content`) are marked in the system as superseded and exist only so that older
   plug-in builds keep working. They are not part of the current contract and are not catalogued here; their replacement
   endpoints under `/mail_plugin/` are.
6. **The operation column.** One draft presented the operation names as the handler identifiers a rebuild must
   reproduce. They are not: nothing outside the server ever sees them. The column now states that they are this
   specification's own names, spelled without abbreviations, and that the condensed catalog is keyed by path.
7. **The failures of an endpoint.** The plan for this document promises, for every endpoint, its path, method,
   authentication, request type, purpose, inputs, outputs and failures, and the opening sentence of the catalog
   repeated that promise; but the row schema had no failures column, the description of the purpose column mentioned
   only the records written and the answer returned, and only the endpoints with a refusal peculiar to themselves
   stated one. The failures of an endpoint are in fact determined by five properties the row already carries — its path
   patterns, its methods, its transport, its authentication level and its site-page flag — together with the records
   the endpoint touches, and they are identical for every endpoint sharing them. The [Failures](#failures) section now
   states that derivation completely, in one table, so that the failure set of any of the 855 rows can be read off; the
   description of the purpose column says that a refusal peculiar to an endpoint is stated in the row; and the section
   points at [`remote-transport-contracts.md`](remote-transport-contracts.md), section 13, for the authoritative
   mapping of every failure class to a status, an envelope code and a body, for the retry loop and for the client retry
   guidance.
8. **Links to four domain folders.** The behaviour-owner column of the domain table and two paragraphs of the payment
   sections referred to the calendar and scheduling, fiscal localizations, marketing and mass mailing, and payment
   providers domains by a direct folder link. They now go through the domain index
   [`../domains/README.md`](../domains/README.md), which carries the entry of every domain, so that every
   cross-reference in this file resolves to a file of this repository.
