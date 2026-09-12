# External integrations

Every touchpoint between the platform and a system outside it, written as an integration contract: the direction of the
exchange, what triggers it, the data that crosses the boundary, how each side authenticates, what happens when the
exchange fails, and whether repeating the exchange is safe. A replacement must reproduce these contracts, because the
external systems are not under the control of the implementer: their request shapes, their acknowledgement strings and
their retry behavior are fixed.

## How to read a contract

Each integration is described with the same eight headings.

| Heading | What it states |
|---|---|
| Direction | Outbound (the platform calls the external system), inbound (the external system calls the platform), or two-way. |
| Trigger | The event or schedule that starts the exchange. |
| Data exchanged | The fields that cross the boundary, in each direction. |
| Authentication | How each side proves its identity, and where the secret is stored. |
| Failure handling | What the platform does on timeout, refusal, malformed answer or service outage, including the exact user-facing message where one exists. |
| Idempotency | Whether repeating the same exchange produces the same result, and the key that makes repetition safe. |
| Configuration | The settings that must exist before the integration works. |
| Records written | The records created or updated when the exchange succeeds. |

## Recurring patterns

Six patterns recur across the integrations. They are described once here and referenced afterwards.

### The purchased-service protocol

Several services are bought from the publisher of the platform rather than contracted directly with the underlying
provider (address enrichment, contact autocompletion, lead generation, visitor identification, text messages, postal
mail, document digitization, text generation, tax identification verification). They all speak the same protocol.

1. Each service is identified by a technical service name and is represented locally by a **service account** record
   holding an **account token** (a forty-three character random value), the service, the companies it serves, an alert
   threshold and the users to alert.
2. Every call is a structured remote call to the service address, whose parameter block always carries the account
   token and the installation identifier, plus the service-specific parameters. The default service address is
   configurable per service through a configuration parameter, which is how a test or self-hosted service is
   substituted.
3. The answer is either a result document or an error block. An error whose class name ends in insufficient credit is
   converted into a dedicated exception carrying the balance, the service name and the address at which credit is
   bought; every other error becomes a generic service error.
4. A timeout (fifteen seconds by default, three hundred seconds for enrichment and lead generation, thirty seconds for
   text generation and postal mail) raises an access error with the message "The request to the service timed out.
   Please contact the author of the app. The Web Address it tried to contact was &lt;address&gt;." A transport failure raises an
   access error with the message "An error occurred while reaching &lt;address&gt;. Please contact support if this
   error persists." with the vendor name removed in a replacement.
5. The balance of an account is read on demand from the balance address; when the read fails the balance is reported as
   minus one, therefore the interface can show that it is unknown rather than zero.
6. When a call fails for lack of credit, the platform posts an internal notification naming the service and linking to
   the credit page, and it records a configuration parameter, therefore the same notification is not repeated until a
   successful call resets it.
7. An installation marked as neutralized (a copy made for testing) has the suffix `+disabled` appended to every account
   token on creation, which makes every call fail at the service instead of consuming credit.

### The signed proxy protocol

The electronic document exchange integrations do not call the network directly; they call a proxy operated by the
publisher, with a stronger authentication scheme:

1. Registration generates a private key locally, sends the matching public key and the participant identification to the
   proxy, and stores the returned client identifier and refresh token.
2. Every request carries the client identifier and a timestamp in its headers, and a signature computed over the string
   `timestamp|path|client identifier|sorted query parameters|body`. The signature is a keyed digest using the refresh
   token, or, when the token is out of synchronization, a signature made with the private key.
3. The refresh token expires after twenty-four hours; a request refused for an expired token triggers a token renewal
   call, which is serialized by locking the registration record, therefore two workers cannot renew at once. A renewal that
   fails because another copy of the installation renewed first is logged and left alone, which is what prevents two
   copies of the same installation from talking to the proxy with one identity.
4. Documents are encrypted with the participant key, therefore the proxy stores content it cannot read.
5. Three operating modes exist: production, test and demonstration. In demonstration mode every call is refused locally
   with the error "Can't access the proxy in demo mode", and registration is simulated.

### The delegated authorization pattern

Every integration that acts on behalf of a person at an external provider (calendars, mail servers, merchant accounts)
follows the authorization-code exchange:

1. The platform sends the browser to the authorization address of the provider with the registered client identity, the
   requested scope, the return address and a state value that carries the record being connected and an anti-forgery
   token.
2. The provider returns the browser to the return address with a single-use code.
3. The platform exchanges that code, server to server, for an access token and a refresh token, and stores both on the
   record; the access token has a stated expiry.
4. Before each later call, an expired access token is renewed with the refresh token. A refresh that is refused clears
   the stored credentials and marks the connection as needing authorization again.
5. A variant exists where the client secret belongs to the publisher rather than to the customer: the exchange is
   performed by the publisher proxy and the resulting tokens are posted back to the installation on a dedicated return
   address that verifies the anti-forgery token before storing them.

### The inbound notification pattern

An external system that must report an outcome calls a notification address of the platform. Every such address obeys
the same rules:

1. It is public, because the caller holds no session, and it is exempt from cross-site submission protection.
2. It authenticates the message content: a signature header verified against a shared secret, a message digest over the
   payload, a shared token stored at registration time, or a second call back to the provider that re-reads the object
   by its reference.
3. It answers with the exact body the provider expects; a different body or status makes the provider retry.
4. It is idempotent: applying an outcome that is already applied changes nothing.
5. It does not open a session, therefore no session cookie is set on a cross-site request.

### Outbound call reliability

| Rule | Statement |
|---|---|
| Timeouts | Every outbound call states a timeout. No call is made without one. |
| Transaction boundary | A call whose effect must not be undone by a rollback is deferred to after the commit; a call made during a transaction that then rolls back is logged as cancelled. |
| Logging | The request and the answer of a carrier or provider call are written to the technical log when debug logging is enabled on that provider record. |
| Test isolation | During automated tests every purchased-service call is refused locally with "Unavailable during tests.", therefore no test consumes credit or reaches a live service. |
| Installation start | Calls are refused while a capability package is being installed, with "Unavailable during module installation." |

### Data minimization

Only the fields the external service needs cross the boundary. Two values accompany almost every purchased-service call
and are part of the contract: the installation identifier (a stable random value that identifies the installation to the
service) and the account token of the service.

## Electronic mail, outbound

**Direction** Outbound. **Trigger** Any message queued for sending: a notification of a discussion thread, a mass
mailing, a document sent to a customer, a password reset, a scheduled reminder. The queue is drained by the scheduled
mail sending job and by immediate sends.

**Configuration.** An **outgoing mail server** record holds: name; priority (lower number wins, default ten); sender
filter (a comma-separated list of addresses or domains this server may send for); host; port (default twenty-five,
conventionally four hundred sixty-five for implicit encryption and five hundred eighty-seven for negotiated
encryption); authentication kind (user name and password, certificate, or the credentials of the server process);
user name; password; encryption (none, negotiated with validation, negotiated without validation, implicit with
validation, implicit without validation); client certificate and private key for certificate authentication; debug
logging flag; maximum message size; and the archived flag.

**Server selection.** For a given sender address the following order decides which server sends, and which sender
address is finally used:

1. An active server whose sender filter contains the exact normalized sender address; the message keeps its sender.
2. An active server whose sender filter contains the domain of the sender address; the message keeps its sender.
3. An active server whose sender filter contains the exact notification address of the installation; the sender becomes
   the notification address.
4. An active server whose sender filter contains the domain of the notification address; the sender becomes the
   notification address.
5. The first active server that declares no sender filter; the sender becomes the notification address when one exists,
   otherwise it stays the original sender.
6. The first active server, whatever its filter, with a warning in the technical log.
7. No server at all: the process-level mail settings are used.

**Sender rewriting.** When the envelope sender resolves to the notification address of the installation but the message
header sender is a different address, the header sender is encapsulated: the display name keeps the original address and
the address part becomes the notification address, which is what keeps the message aligned with the sender policy of the
domain while still showing the real author. When the bounce address matches the sender filter of the chosen server, the
envelope sender is set to the bounce address, therefore delivery failures return to the platform.

**Data exchanged.** Outbound: the envelope sender, the envelope recipients (the union of the visible, copied and blind
copied recipients), and the message itself: sender header, recipients, subject, alternative plain and formatted bodies,
attachments, message identifier, answer references, answer-to address, and any extra headers the caller supplies.
Inbound: the delivery acknowledgement of the server, or a refusal code.

**Failure handling.** A refusal raises a delivery exception and the message is marked as failed with the reason, which
is shown in the discussion thread of the document. The scheduled sending job retries a failed message according to the
retry policy of the queue. Identification headers (message identifier, answer references) are never folded across lines,
because relays that rewrite them break the conversation grouping. A message larger than the maximum size of the server
is refused before sending.

**Connection test.** Testing a server simulates a send without delivering: it opens the connection, announces the
sender, announces a test recipient, and begins the data phase, then abandons. Each step has its own message: "The server
refused the sender address (&lt;sender&gt;) with error &lt;reply&gt;", "The server refused the test recipient
(&lt;recipient&gt;) with error &lt;reply&gt;", "The server refused the test connection with error &lt;reply&gt;",
"Invalid server name!", "No response received. Check server address and port number.", "The server has closed the
connection unexpectedly." The same test optionally reads the maximum message size the server announces and stores it,
refusing with "The server \"&lt;name&gt;\" doesn't return the maximum email size." when the server does not announce one.

**Idempotency.** Sending is not idempotent: the transport has no repeat-safe key. The platform protects against
duplicates by marking the message as sent inside the same transaction that performed the send and by never retrying a
message whose state is already sent.

## Electronic mail, inbound

**Direction** Inbound. **Trigger** The scheduled mail fetching job, or a manual fetch from the server record; also a
direct delivery from a local mail transfer agent that pipes the message to the platform.

**Configuration.** An **incoming mail server** record holds: name; state (not confirmed, confirmed); server type
(internet message access protocol, post office protocol, or local delivery); host; port; encryption flag; user name;
password; the entity to create for new conversations; keep attachments flag; keep original message flag; priority (lower
first, default five); last fetch date; last error date and message; and the script path used for local delivery.

**Fetching order and batching.** The job processes confirmed, non-local servers ordered by priority ascending, then by
last fetch date ascending (oldest first), then by identifier, which rotates the servers between runs. Each server is
locked for update; a server that cannot be locked is skipped with a log line. Messages are processed one at a time in a
separate transaction, such that one failing message does not roll back the others, and the transaction is committed after
each message. At most fifty messages are taken from one server in one run, and the run also stops when the time budget
of the job is exhausted.

**Data exchanged.** Inbound: the complete message, including headers, bodies and attachments. Outbound: the read
acknowledgement that marks the message as handled on the server.

**Routing.** Each fetched message is routed to a record by the following order:

1. **Bounce detection.** When the message is a bounce (see the next section), bounce processing runs and routing stops.
2. **Answer to an existing message.** The answer references of the message are matched, newest first and limited to the
   last thirty-two references, against the stored message identifiers. A match gives the entity and record of the
   original message. When the recipient address is the alias of a different entity, the message is treated as a forward
   rather than an answer and the match is discarded.
3. **Alias match.** The recipient addresses are matched against the aliases, either by complete address or, when the
   alias accepts local matching, by local part. Every matching alias produces one route, therefore one message can update or
   create several records.
4. **Fallback.** The entity and record supplied by the caller of the gateway.
5. **Refusal.** When the recipients contain the catch-all address and nothing routable, a bounce message is generated
   back to the sender.

Each candidate route is then verified: the record must exist and accept updates, otherwise the route falls back to
creation; the entity must accept creation; and the alias contact policy must be satisfied (everyone, authenticated
partners only, or followers only). A route refused by the alias policy produces a bounce message to the sender and the
route is dropped. Refusals are logged as "Mailbox unavailable - &lt;reason&gt;".

**Loop protection.** Two guards prevent a message storm. First, a message whose references contain the loop-detection
marker is discarded, which stops an automatic reply to a bounce from generating another bounce. Second, when more than
the loop threshold of records was created from the same sender address through the same alias within the loop window,
further messages are refused; the threshold defaults to twenty records and the window to one hundred and twenty minutes,
both configurable.

**Failure handling.** A message that raises during processing is rolled back, counted as failed and still marked as
handled on the server, therefore one poisoned message cannot block the queue forever. A connection failure on a server records
the error date and message on that server; when the error persists for five days the server is set back to not confirmed
and the administrators are notified with "Deactivating incoming mail &lt;type&gt; server &lt;name&gt; (too many failures)".

**Idempotency.** The message identifier of an incoming message is stored on the created thread message. A message
delivered twice, therefore produces one thread message; the second delivery is recognized as an answer to the first.

## Delivery failure (bounce) handling

**Direction** Inbound. **Trigger** A delivery failure notification arriving on the fetching servers.

**Detection.** A message is a bounce when any of the following holds: a recipient address equals a configured bounce
address of an alias domain; the local part of the sender is the mail daemon convention; the content type is a
multipart report or declares a delivery-status report type.

**Processing.** The bounced address, the bounced contact and the identifiers of the bounced message are extracted. Then:

1. Every entity that supports blocking by address is searched for records whose normalized address equals the bounced
   address, and each receives the bounce, which increments its bounce counter and may add the address to the block list.
2. When the original message pointed to a record that was not already handled by the previous step, that record receives
   the bounce as well.
3. The notification rows of the original message that match the bounced contact or the bounced address are set to the
   bounce state, with the failure kind set to bounce and the failure reason set to the plain text of the bounce body.

**Reset.** Conversely, when a normal message is received from an address that had a bounce counter, the counter of every
record with that address is reset, because the address demonstrably works again.

**Idempotency.** Applying the same bounce twice increments the counter twice; the platform relies on the mail server
marking the bounce as handled to avoid reprocessing.

## Text messages

Two providers exist. The first is the purchased service of the publisher; the second is a direct contract with a
telephony provider. Both implement the same internal contract: send a batch grouped by content, and report per message a
state and a failure kind.

### Purchased text message service

**Direction** Outbound for sending, inbound for delivery reports. **Trigger** Any queued text message: a marketing
campaign, a document notification, a reminder, a one-time code.

**Data exchanged.** Outbound, per batch: a list of contents, each with the list of destination numbers and, for each
number, a generated message identifier; the address to which delivery reports must be sent; the account token and the
installation identifier. Inbound, per message identifier: a state among success, processing, server error, unregistered,
insufficient credit, wrong number format, duplicate message, country not supported; and, when the price is known, the
credit consumed.

**Authentication.** The account token of the text message service.

**Failure handling.** Each returned state maps to a stored failure kind and a message shown beside the message:
"You don't have an eligible In Application Purchase account." (unregistered), "You don't have enough credits on your In Application Purchase account." followed
by a link to buy credit (insufficient credit), "The number you're trying to reach is not correctly formatted."
(wrong format), "This Text Message has been removed as the number was already used." (duplicate), "The destination country is not
supported." (country not supported), "The content of the message violates rules applied by our providers."
(incompatible content).

**Delivery reports.** The service calls the report address of the installation with batches shaped as a list of
`{state, list of message identifiers}` entries; each matching message is updated. The address is public and exempt from
cross-site protection.

**Account registration.** Registering the sender requires three calls: send a verification code to a telephone number,
verify the code, and set the sender name. Their refusals have their own messages: "Your sender name must be between 3
and 11 characters long and only contain alphanumeric characters.", "Your text message account has not been activated yet.",
"This account already has an existing sender name and it cannot be changed.", "Invalid phone number. Please make sure to
follow the international format, i.e. a plus sign (+), then country code, city code, and local phone number. For example:
+1 555-555-555", "We were not able to reach you via your phone number. If you have requested multiple codes recently,
please retry later.", "The Text Message Service is currently unavailable for new users and new accounts registrations are
suspended.", "This phone number/account has been banned from our service.", "Your country is not supported due to sender
registration legislation", "Your database is not activated", "The verification code is incorrect.", "We were not able to
find your account in our database.", "You tried too many times. Please retry later.", "An unknown error occurred. Please
contact support if this error persists."

**Idempotency.** The generated message identifier is the repeat-safe key: the service rejects a second send of the same
identifier as a duplicate message, and a delivery report names the identifier, therefore reports may arrive more than
once without harm.

### Direct telephony provider

**Direction** Outbound for sending, inbound for delivery reports. **Trigger** The same queue.

**Data exchanged.** Outbound, one call per message: the sending number chosen for the destination, the destination
number, the body, and the address at which the provider must report the status, which carries the message identifier in
its path. Inbound: the provider reference of the accepted message, or an error code and message.

**Authentication.** Outbound: the account identifier and the authentication token of the company, sent as basic
credentials. Inbound: a signature header computed by the provider over the report address and its parameters, verified
before the report is applied; a report whose message identifier is not thirty-two hexadecimal characters, whose state is
unknown, or whose signature does not verify is answered with the not-found status.

**State mapping.** The provider states map as follows: queued and accepted and scheduled to outgoing; sending and
receiving to in process; sent and received to pending; delivered to sent; canceled to cancelled; failed and undelivered
to error.

**Error mapping.** Provider error codes map to failure kinds: invalid destination number, unreachable number and
unsupported number map to wrong number format; missing destination maps to missing number; identical sender and
destination maps to sender equals destination; missing sender maps to missing sender; unverified recipient on a trial
account maps to unverified account; an incorrect report address maps to a report address error. Each failure kind has a
message: "Trial Account Limitation", "A 'To' phone number is required", "Unverified recipient on Trial Account",
"Twilio Authentication Error", "Twilio StatusCallback Web Address is incorrect", "A 'From' number is required to send a
message", "'To' and 'From' numbers cannot be the same", "The number you're trying to reach is not correctly formatted",
"Unknown error, please contact support".

**Failure handling.** A transport failure or timeout (five seconds) leaves the message in the server error state with
the reason "Unknown failure at sending, please contact support".

**Idempotency.** The provider reference returned on acceptance is stored on the message; a report carries the message
identifier of the platform, therefore repeated reports are applied to the same message and produce the same state.

## Postal mail

**Direction** Outbound. **Trigger** A user chooses to send a document by post, or a scheduled job sends the pending
letters.

**Data exchanged.** Outbound, per batch: the account token; for each letter, the letter identifier, the entity and
record it belongs to, the rendered document, the page count, the recipient address (name, street, second street line,
postal code, city, state code, country code) and the return address in the same shape; and the print options (colour,
double sided, cover page, currency name). Inbound: a request code, a total cost, a credit error flag, and, per document,
whether it was sent and the tracking identifier, or an error code.

**Authentication.** The account token of the postal service.

**Failure handling.** Error codes are: missing required fields, credit error, trial error, no price available, format
error, unknown error, attachment error, and too many pages. Each produces a message posted on the letter and on the
notification: "You don't have enough credits to perform this operation." with a link to the account, "You don't have an
In Application Purchase account registered for this service." with a link to claim free credit, "The country of the partner is not covered by
the postal mail service.", "One or more required fields are empty.", "The attachment of the letter could not be sent. Please check its
content and contact the support if the problem persists.", "The document to be sent exceeds the maximum allowed limit of
8 pages.", "An unknown error happened. Please contact the support." A letter with an unusable address is not sent at all
and is set to the error state with "Invalid recipient name." A transport failure sets every letter of the batch to the
error state with the unknown error code and re-raises.

**Idempotency.** Each letter carries its own identifier in the request and the answer names it, therefore a repeated
answer updates the same letter. The platform commits after each letter, therefore a failure halfway through does not resend the
letters already accepted.

**Records written.** The letter state (in queue, sent, error, cancelled), its information message, its error code, the
tracking identifier, and the notification rows of the underlying message.

## Payment providers

The payment integrations are specified as a domain of their own; see
[`../domains/payment-providers/README.md`](../domains/payment-providers/README.md) for the transaction state machine,
the tokenization rules, the refund and capture flows and the per-provider peculiarities, and
[`endpoint-catalog.md`](endpoint-catalog.md) for the return and notification addresses. The integration contract common
to all of them is summarized here.

| Heading | Contract |
|---|---|
| Direction | Two-way. Outbound to create, capture, refund and query payments; inbound for the payer return and for provider notifications. |
| Trigger | A payer confirms a payment on the payment page, a portal page, a storefront checkout or a point of sale screen; or the provider reports an outcome. |
| Data exchanged | Outbound: the transaction reference, the amount in the units the provider expects, the currency, the payer contact and address, the return address, the notification address, the payment method, and the saved token when one is used. Inbound: the provider reference, the outcome, the amount actually taken, the payment method details, the error code and the signature of the message. |
| Authentication | Outbound: the credentials of the provider record (identifier and secret, or key pair, or a token obtained by delegated authorization). Inbound: a signature header, a message digest, or a second call that re-reads the payment at the provider. |
| Failure handling | A refused or failed outcome moves the transaction to error and the reason is shown to the payer; a transport failure leaves the transaction in its state and the payer sees the pending page, which polls until an outcome arrives. |
| Idempotency | The transaction reference is the repeat-safe key. Applying an outcome that is already applied is a no-operation; notifications are expected to arrive more than once. |
| Configuration | One provider record per company, with its state (disabled, test, enabled), its credentials, its allowed currencies and countries, its payment methods and its notification secret. |

## Shipping carriers and pickup networks

**Direction** Two-way. **Trigger** A price is requested for a cart or an order; a shipment is validated; a shipment is
cancelled; a customer opens a tracking link; a customer chooses a pickup point.

**The carrier contract.** A **delivery method** record declares its provider kind. Two provider kinds are built in
(fixed price and price rules) and every connected carrier adds its own. A connected carrier implements up to six
operations:

| Operation | Inputs | Output | Notes |
|---|---|---|---|
| `rate_shipment` | The order, with its delivery address, its lines, their weights and volumes | `{success, price, error_message, warning_message}` | The returned price is converted to the currency of the order, tax handling is applied according to the fiscal position, then the configured percentage margin and fixed margin are added, and the free-over rule may set the shown price to zero while the real price is kept. |
| `send_shipping` | The transfers to ship | One entry per transfer: `{exact_price, tracking_number}` | Called when the transfer is validated; the returned labels are attached to the transfer. |
| `get_return_label` | The transfers, an optional tracking number, an optional original date | The return label, stored as an attachment | Only when the method declares that it can generate returns; when the method publishes return labels on the portal, an access token is generated on the label. |
| `cancel_shipment` | The transfers | None | Called when a shipped transfer is cancelled. |
| `get_tracking_link` | The transfer | The tracking address | Also derivable from the tracking address template of the method, where the placeholder for the tracking number is substituted. |
| `_get_close_locations` | The delivery address | A list of pickup points | Only when the method declares that it uses pickup points. |

**Availability filters.** Before a method is offered it must pass every filter: the delivery country and state must be
in its lists, the postal code must match one of its prefixes when prefixes are configured, the total weight must not
exceed its maximum weight, the total volume must not exceed its maximum volume, every product must carry at least one
required tag when required tags are configured, and no product may carry an excluded tag.

**Data exchanged.** Outbound: the shipper address, the recipient address, the package list with dimensions, weights and
declared values, the service level, the insurance percentage, the payment arrangement and the label format. Inbound: the
price with its currency, the promised delay, the tracking numbers, the labels as documents, and the error text.

**Authentication.** The credentials stored on the delivery method record, with a production flag that selects between
the test and the production endpoints of the carrier.

**Failure handling.** A carrier error is returned as an unsuccessful rate with its message, which the storefront shows
beside the method; a shipping error raises and blocks the validation of the transfer. Every request and answer is
written to the technical log when the debug logging flag is set on the method.

**Idempotency.** Rating is read-only and may be repeated freely. Shipping is not idempotent at the carrier; the platform
protects against double shipment by refusing to ship a transfer that already carries a tracking number.

**Pickup networks.** A pickup network is a carrier that returns a list of points, each with a name, an address, a set of
opening hours and an identifier. The storefront and the portal ask for the points near a postal code; the country is
taken from the network position of the visitor, falling back to the country of the delivery address. The chosen point is
stored on the order as structured data and fixes the delivery address of the shipment. When no point can be offered the
answer is the message "No pick-up points are available for this delivery address." One network is integrated through a
browser widget of the carrier instead of a server call: the widget returns the chosen relay point and the storefront
stores it through its own endpoint. In-store collection is modelled as a pickup network whose points are the warehouses
of the company, which requires no external call.

## Calendar synchronization

Two external calendar services are supported. Both are two-way synchronizations of the meetings of one user, and both
follow the delegated authorization pattern.

**Direction** Two-way. **Trigger** The user opens the calendar or presses the synchronization action; a scheduled job
for users whose synchronization is active; and any local change of a synchronized meeting, which marks it as needing
synchronization.

**Outbound, per changed meeting.** Create, update or delete at the provider, carrying: title, description, start and
end (or the all-day dates), time zone, recurrence rule, place, attendee list with their answers, reminders, privacy and
the video meeting request. The video meeting room is requested at creation when the meeting asks for one.

**Inbound.** The events changed since the last synchronization token. The first synchronization has no token and is
therefore restricted to a window around today, three hundred and sixty-five days in each direction by default and
configurable. The provider returns pages; the platform follows the page token until the last page, which carries the
next synchronization token, and stores that token for the following run. The second provider uses a change-feed address
and reads the change token out of the returned feed address.

**Conflict resolution.** Each meeting carries the identifier it has at the provider and a needs-synchronization flag.
Local changes set the flag; the outbound pass sends them and clears it. The inbound pass writes provider changes with
the flag cleared, therefore an inbound write does not immediately bounce back. A meeting deleted at the provider is archived
locally; a meeting deleted locally is deleted at the provider, and a deletion the provider reports as already gone
(status gone or forbidden) is treated as success.

**Failure handling.** A synchronization token the provider refuses as no longer valid (status gone, with a full
synchronization required indication) restarts the synchronization from a full window. An expired authorization returns
the state that asks the user to authorize again. The states reported to the client are: configuration missing,
authorization needed, refresh needed, synchronization stopped, and success.

**Idempotency.** The provider identifier stored on each meeting is the repeat-safe key: a meeting received twice
updates the same record; a meeting created locally is sent with a client-generated identifier, therefore a retried creation does
not duplicate.

**Records written.** The meeting, its attendees and their answers, its recurrence, its reminders, and the
synchronization token and stop flag on the user settings.

## Delegated sign-in providers

**Direction** Two-way. **Trigger** A visitor presses a provider button on the sign-in form.

**Configuration.** A **delegated sign-in provider** record holds: provider name; client identity; authorization
address; scope (default `openid profile email`); user information address; optional extra data address; enabled flag;
button label; icon class; and sequence. Three providers are shipped preconfigured: the publisher account service, a
social network and a search company. The client identity of the publisher account service is set to the installation
identifier at installation time.

**Flow.** The button sends the browser to the authorization address with the client identity, the scope, the return
address and a state document that carries the installation name, the return target and, for a sign-up, the invitation
token. The provider returns to the sign-in endpoint with an access token. The platform then validates the token against
the user information address, adding the extra data address when one is configured; the request carries the token either
as a bearer header or as a query parameter, chosen by a configuration parameter. The subject identity is read from the
answer under the standard subject key, falling back to the two other conventional keys; a missing subject is refused
with "Missing subject identity".

**Matching.** A user whose stored provider and subject match is signed in and its stored access token is refreshed.
When no user matches, the sign-up values are generated (name, address, provider, subject, access token) and the
self-registration flow runs with the invitation token from the state; a refused sign-up produces an access-denied
refusal.

**Failure handling.** A validation answer carrying an error raises; a refused token answer is converted to the
authentication challenge the provider returned or to the generic invalid-request error.

**Idempotency.** The pair of provider and subject is unique per user, therefore repeated sign-ins never create a second
user.

## Directory servers

**Direction** Outbound. **Trigger** A sign-in attempt whose credentials do not match a local password, and a password
change.

**Configuration.** A **directory server** record per company holds: sequence; company; server address (default the
local host); port (default three hundred eighty-nine); binding account and password (empty for anonymous binding);
search filter (with one placeholder for the login, repeated as many times as the filter contains placeholders); search
base; template user to copy; create-user flag; and the negotiated encryption flag. Configurations are tried in sequence
order. Following referrals is disabled by default and controlled by a configuration parameter.

**Authentication flow.** For each configuration in order:

1. The directory is searched with the binding account for the entry whose filter matches the login. Exactly one entry
   must match; zero or several means no match.
2. A second connection binds with the distinguished name of that entry and the supplied password. An empty password is
   refused before any call, because an empty password with a valid distinguished name would be accepted as an anonymous
   binding.
3. On success the local user is found by lower-cased login. When none exists and the create-user flag is set, the
   template user is copied (or a plain user created) with the name taken from the common name attribute, the login, the
   company of the configuration, and the address when the login is itself an address.
4. When no local user exists and creation is not allowed, the sign-in is refused with "No local user found for directory
   login and not configured to create one".

**Password change.** Changing the password of a directory-backed user binds with the old password and issues the
password modification operation; a refusal leaves the local password unchanged.

**Failure handling.** An invalid binding logs "Directory bind failed."; any other directory error is logged and treated as no
match, therefore one unreachable directory does not block the other configurations. The search timeout is sixty seconds.

**Idempotency.** Reading is idempotent. User creation is guarded by the uniqueness of the login.

## Purchased data services

All of these follow the purchased-service protocol. They differ in the address they call, the parameters and the answer.

### Contact autocompletion

**Direction** Outbound. **Trigger** A user types a company name or a tax identification number in a contact form, or
asks to complete an existing contact.

**Data exchanged.** Outbound: the typed text or the number, plus the installation identifier, the platform version, the
display language, the account token, the country code of the company and its postal code. Inbound: candidate companies
with name, address, country, state, city, postal code, website, logo, tax identification number, registry number,
industry and company size.

**Operations.** Search by name, search by tax identification number, complete by registry number, complete by national
tax number, complete by web domain.

**Failure handling.** Every failure is converted to a pair of an empty result and a reason: "Insufficient Credit",
"No account token", or the text of the transport error. The form then simply offers no suggestion; nothing blocks.

**Idempotency.** Read-only.

### Lead enrichment

**Direction** Outbound. **Trigger** A lead is created with an electronic mail address and the enrichment setting is on,
or a user presses the enrich action. The scheduled enrichment job processes the leads waiting for enrichment.

**Data exchanged.** Outbound: a map of lead identifier to address domain. Inbound, per lead: the company data (name,
address, country, telephone, website, social profiles, employee count, revenue, industry, logo) or nothing when the
domain is unknown.

**Failure handling.** Insufficient credit raises the dedicated exception carrying the balance, the service name and the
credit address, and the platform posts the no-credit notification once. Domains that belong to generic mail providers
are never sent, because an address at a generic provider identifies no company; the list of generic providers is part of
the configuration data.

**Idempotency.** Read-only. A lead already enriched is not enriched again, which is recorded by a flag on the lead.

### Lead generation

**Direction** Outbound. **Trigger** A user runs a lead generation request.

**Data exchanged.** Outbound: the search criteria (countries, states, industries, company size range, number of results,
whether contact people are requested and their seniority and function filters), plus the account token and the
installation identifier. Inbound: a list of companies with their data and, when requested, contact people with name,
function, address and telephone.

**Failure handling.** A credit error sets the request to the error state with the credit error kind; an empty answer
sets the no-result error kind; any other failure raises "Your request could not be executed: &lt;reason&gt;".

**Idempotency.** Each returned company carries a stable external identifier; a lead is not created when a lead with the
same external identifier already exists.

### Visitor identification

**Direction** Outbound. **Trigger** The scheduled job that processes the recorded page views of anonymous visitors.

**Data exchanged.** Outbound: the network addresses of the visitors and the matching rules with their criteria. Inbound:
per network address, the identified company and, optionally, contact people; or a not-found marker.

**Failure handling.** A credit error notifies once and stops the run. Addresses that produced no answer are marked as
not found, therefore they are not retried in a loop.

**Idempotency.** A lead is not created when one already exists with the same external identifier of the identified
company; the recorded views of an identified address are deleted once a lead is created.

### Document digitization

**Direction** Outbound, then polling. **Trigger** A vendor bill, a receipt or a bank statement is uploaded as a
document and digitization is enabled.

**Data exchanged.** Outbound: the document and the account token. Inbound: the extracted fields with their positions and
confidence: supplier identity, supplier tax identification number, invoice date, due date, document reference, total
excluding tax, total tax, total including tax, currency, payment reference, and the line items with description,
quantity, unit price and taxes.

**Contract shape.** Submission returns a document reference; the platform polls for the result and applies it when the
extraction is complete, leaving the document in a waiting state in the meantime. A user may correct any extracted value,
and the corrections are sent back to the service, therefore it learns from them.

**Failure handling.** A failed extraction leaves the document in the error state with the reason and lets the user fill
the document by hand; no document is ever blocked by the service being unavailable.

**Idempotency.** The document reference returned by the service is the repeat-safe key; polling the same reference many
times is safe, and a document already applied is not applied twice.

*This subsection is an industry-standard completion for the parts of the flow that the shipped configuration switches
name but whose implementation is delivered separately: the settings "Document Digitization", "Invoice Digitization" and
"Bank Statement Digitization" exist and select the capability, and the submit-then-poll shape, the correction feedback
and the confidence handling follow the same purchased-service protocol as the other services documented above.*

### Text generation

**Direction** Outbound. **Trigger** A user asks the rich text editor to write or rewrite a passage.

**Data exchanged.** Outbound: the instruction, the previous exchanges of the same conversation and the installation
identifier. Inbound: a status and the generated text.

**Failure handling.** Status `error_prompt_too_long` produces "Sorry, your prompt is too long. Try to say it in fewer
words."; status `limit_call_reached` produces "You have reached the maximum number of requests for this service. Try
again later."; any other non-success status produces "Sorry, we could not generate a response. Please try again later.";
an access failure produces "Oops, it looks like our AI is unreachable!". The timeout is thirty seconds.

**Idempotency.** Not idempotent by nature; the platform never retries automatically.

### Tax identification verification

**Direction** Outbound with an inbound callback. **Trigger** A tax identification number is entered or changed on a
contact while the company has the verification setting on.

**Data exchanged.** Outbound: the number, the installation identifier, a client identifier and a client token generated
once per installation and stored as configuration parameters, the callback address of the installation, and a callback
token signed over the number with a validity of seven days. Inbound, immediately: a status among valid, invalid,
unassigned, pending and fault. Inbound, later: a call to the callback address carrying the callback token and the final
status.

**Authentication.** The client identifier and client token pair identifies the installation for update requests; the
callback token is verified by recomputation, which is what prevents anybody else from calling the callback.

**Failure handling.** A transport failure or an answer without a status is treated as the fault status. The statuses
produce log entries on the contact: "The VIES check is pending. The status will be updated soon.", "The VIES check
failed. Please check the Tax Identification manually.", "The Intra-Community validity has been updated to: &lt;status&gt;."

**Polling fallback.** A scheduled job calls the update address with the client identifier and token, receives a map of
numbers to statuses, and applies each status to every contact carrying that number. This covers the case where the
callback could not reach the installation.

**Idempotency.** The status is written by number, therefore repeated callbacks and repeated polls converge to the same
value.

**Special cases.** A child contact whose number equals the number of its parent inherits the validity of the parent
without any call.

## Electronic document exchange networks

Three national or international document exchange networks are integrated, and they share the signed proxy protocol.
The document formats, the participant registration states and the business rules are specified in
[`../domains/electronic-invoicing-and-document-exchange/README.md`](../domains/electronic-invoicing-and-document-exchange/README.md);
the inbound addresses are catalogued in [`endpoint-catalog.md`](endpoint-catalog.md).

| Heading | Contract |
|---|---|
| Direction | Two-way. Outbound to register a participant, to send documents and to query delivery states; inbound for the three notifications (new document waiting, delivery state changed, participant state changed) and for the registration decision. |
| Trigger | Outbound: a document is confirmed and its exchange is requested, or the scheduled job flushes the queue. Inbound: the network has something to report. |
| Data exchanged | Outbound: the encrypted document, the sender and receiver participant identifiers, the document kind and the request identifier. Inbound: the encrypted document, the delivery state per request identifier, and the participant state. |
| Authentication | Outbound: the client identifier, a timestamp and a signature over the canonical request string, using the refresh token or the private key. Inbound: a shared token stored at registration, carried as a parameter of the notification. |
| Failure handling | A refused request raises a proxy error with the message "The Web Address that this service requested returned an error. The Web Address it tried to contact was &lt;address&gt;." or, for a missing address, "The Web Address that this service tried to contact does not exist. The Web Address was &lt;address&gt;."; an expired refresh token triggers a renewal and one retry; a failure of the renewal is logged and the exchange is retried on the next run. |
| Idempotency | The request identifier of each sent document is the repeat-safe key; a notification about a request identifier already in its final state changes nothing. |
| Configuration | The participant identification, the operating mode (production, test, demonstration), the registration record with its key pair, and the contact details registered with the network. |

## External image and media libraries

### Photograph library

**Direction** Outbound. **Trigger** A user searches the media picker of the editor; a user inserts a chosen photograph.

**Data exchanged.** Outbound for search: the typed text, the page and the registered application key. Inbound: the
matching photographs with their preview addresses, their download addresses and their author credit. Outbound on
insertion: a download-registration call to the address supplied by the service, which the terms of the service require
for every image actually used.

**Authentication.** An access key and an application identity stored in the installation settings.

**Failure handling.** A failure of the download registration is logged and the insertion continues, because the image is
already stored; a search failure returns no result. A download-registration address that does not belong to the service
raises "ERROR: Unknown Unsplash notify Web Address!".

**Idempotency.** Search is read-only; the download registration is a counter increment at the service and repeating it
only inflates that counter.

### Illustration library

**Direction** Outbound. **Trigger** A user searches the illustration library inside the editor, then saves the chosen
illustrations.

**Data exchanged.** Outbound: the search terms. Inbound: the matching illustrations with their preview addresses.
Saving stores each chosen illustration as an attachment; illustrations that support colour substitution are stored as
recolourable vector images with their colour map.

### Animated image library

**Direction** Outbound. **Trigger** A user opens the animated image picker of the message composer or types a search
term in it.

**Data exchanged.** Outbound: the search term or the category request, the service key stored in the installation
settings, the installation name as the client key, a result limit of eight, the content filter level `medium`, the
display language, the country and the media variant `tinygif`; for paging, the position cursor returned by the previous
answer. Inbound: the matching images with their preview and full addresses and their identifiers.

**Failure handling.** The call has a three second timeout; a timeout or a refusal logs "Exceeded the request's maximum
size for a searching term." and answers the client with the bad-request status.

**Idempotency.** Read-only. Favourites are stored locally by the image identifier of the service, with one favourite per
user and image.

## Anti-robot verification

Two services are supported, and both plug into the same verification point: any endpoint that accepts a public
submission may require a verification token before it acts.

| | First service | Second service |
|---|---|---|
| Site key setting | `recaptcha_public_key` | `cf.turnstile_site_key` |
| Secret setting | `recaptcha_private_key` | `cf.turnstile_secret_key` |
| Enabled switch | `enable_recaptcha`, default true | Presence of the keys |
| Token parameter | `recaptcha_token_response` | `turnstile_captcha` |
| Extra rule | A score is returned; a score below `recaptcha_min_score` is treated as a robot. The action name obtained with the token must equal the action being verified. | No score; the action name is still compared. |
| Timeout | Two seconds | Two seconds |

**Direction** Outbound. **Trigger** Submission of a public form: the sign-in form after repeated failures, the contact
form, the job application form, the mailing list subscription, the forum post form, and any site form that declares
verification.

**Data exchanged.** Outbound: the secret, the token the browser obtained, and the network address of the visitor.
Inbound: a success flag, a score (first service only), the action name, and error codes.

**Failure handling.** The verification result maps to an outcome: a trusted visitor and a missing secret both let the
request continue; an invalid secret raises the validation message "The reCaptcha private key is invalid." or "The
Cloudflare turnstile private key is invalid."; an invalid token raises "The reCaptcha token is invalid." or "The
CloudFlare human validation failed."; a timeout raises "Your request has timed out, please retry."; a malformed request
raises "The request is invalid or malformed."; anything else raises "Suspicious activity detected by google reCAPTCHA."
or "Suspicious activity detected by Turnstile CAPTCHA."

**Idempotency.** Tokens are single use at the service; a replayed token is answered with the timeout-or-duplicate error
code, which the platform maps to the timeout outcome.

## Print on demand

**Direction** Two-way. **Trigger** Outbound: a product catalogue is imported, a price is quoted for a cart, an order is
confirmed, an order is cancelled, an order is queried. Inbound: the partner reports a fulfilment event.

**Data exchanged.** Outbound: for a quote, the delivery address and the ordered items with their product references and
quantities, answered with the available shipment methods and their prices; for an order, the order reference of the
platform, the recipient address, the items with their print files and product references, and the chosen shipment
method, answered with the order reference of the partner; for a product, the template reference, answered with the
variants, their attributes and their print file requirements. Inbound: the event name, the order reference of the
platform, the fulfilment status, the items with their tracking codes and tracking addresses, and a comment.

**Authentication.** Outbound: a key header carrying the account key of the company. Inbound: a signature header
verified against the value computed from the order, which is what proves the notification belongs to that order.

**Failure handling.** A refused call raises a user error carrying the message of the partner; a connection failure or
timeout (ten seconds) raises "Could not establish the connection to the Gelato service."

**Inbound effects.** A failed fulfilment posts the reason in the discussion thread of the order. A cancellation cancels
the order and posts the notice. A shipment and a delivery each send the status message with the tracking data to the
customer. The answer to every notification is empty.

**Idempotency.** The order reference of the platform is the repeat-safe key; a duplicate notification re-applies the
same state and posts the same message, therefore the platform checks the current state before acting.

## Cloud storage of attachments

**Direction** Two-way, but the bytes never pass through the platform: the browser uploads to, and downloads from, the
storage service directly, using addresses that the platform signs.

**Trigger.** An attachment is created with the cloud storage flag and the installation has a storage provider
configured; a visitor requests such an attachment.

**Flow.** On creation, the attachment is stored with the type cloud storage, an empty local content, its original media
type preserved, and an address built from a blob name of the shape `identifier/random value/file name`. The client then
asks for upload instructions and receives a signed address, the request method, the expected success status and the
headers to add. On download, the attachment stream is a redirection to a signed download address, cached until ten
seconds before that address expires.

**Signature lifetimes.** Upload addresses and download addresses are valid for three hundred seconds each.

**Authentication.** First provider: a service account document stored in the settings, from which the signing
credentials are derived and cached. Second provider: an account name, a tenant, a client identity and a client secret,
from which a delegation key valid for seven days is obtained and cached, refreshed when it is within one day of expiry;
an authentication failure is cached as well, therefore a broken configuration does not produce a request storm.

**Excluded entities.** Attachments of entities whose business logic reads the attachment content are never moved to the
cloud; the exclusion list is computed from the entities that declare a main attachment and from the document management
entities.

**Failure handling.** Creating a cloud attachment without a configured provider raises "Cloud Storage is not enabled".
Migrating an attachment back to local storage that cannot be downloaded raises "Failed to download attachment
(&lt;identifier&gt;) from cloud: &lt;status&gt; - &lt;reason&gt;".

**Idempotency.** The blob name contains a random value, therefore two uploads of the same file never collide;
re-signing an address is free and produces an equivalent address.

## Translation platform

**Direction** Outbound, as links only. No data crosses the boundary.

**Trigger.** A user opens the translation dialogue of a translatable field on a record that belongs to a capability
package.

**Behavior.** For each translation row the platform computes a deep link into the external translation platform, built
from the configured project address, the project of the package, the language code and the source text. The mapping
from package to project is read once from the packaging configuration files and cached. When no project address is
configured, no link is produced and the dialogue behaves as it does without the integration.

**Failure handling.** None needed; the integration produces addresses and never calls the platform.

## Currency rates

**Direction** Outbound. **Trigger** The scheduled currency rate update, or the manual update action on the company.

**Contract.** The company names a rate provider and a rate update interval (daily, weekly, monthly). The job asks the
provider for the rates of the day for the currencies that are active in the installation, expressed against the base
currency of the provider, converts them to rates against the currency of the company, and writes one **currency rate**
record per currency and date, replacing the record of the same date when one exists.

**Data exchanged.** Outbound: the base currency, the date and the list of currency codes. Inbound: a rate per currency
code for that date.

**Failure handling.** A provider that cannot be reached leaves the existing rates untouched and records the failure on
the company; the user is told that the rates could not be updated. A currency the provider does not quote is skipped
rather than set to zero, because a zero rate would make every conversion fail.

**Idempotency.** The pair of currency and date is unique, therefore repeating the update for the same day overwrites the
same rows and changes nothing else.

*The provider catalogue and the parsing of each provider answer are delivered by a separate capability package; the
setting that selects it is "Automatic Currency Rates" on the accounting configuration. The contract above is an
industry-standard completion: one rate per currency and date, rates stored against the currency of the company, no
negative or zero rates, and the last rate on or before a date being the one used by every conversion (see
[`../domains/multi-currency/calculations.md`](../domains/multi-currency/calculations.md)).*

## Geolocation

Two distinct services are involved: turning an address into coordinates, and turning the network address of a visitor
into a country and city.

### Address to coordinates

**Direction** Outbound. **Trigger** The geolocation action on a contact, on a lead or on a set of them; and any flow
that needs a distance (reseller search, map display).

**Providers.** Two provider records are shipped: an open mapping service (the default) and a commercial mapping
service. The provider is selected by a configuration parameter; when none is selected the first provider record is used.

**Data exchanged.** Outbound: the address assembled as one string. For the open service: the string as a query, with a
descriptive agent header and the answer format requested as structured data. For the commercial service: the string, the
registered key, and optionally a country restriction. Inbound: the coordinates of the best match.

**Fallback.** When the complete address yields nothing, the lookup is repeated with the city, state and country only.

**Failure handling.** The commercial service without a key raises "Key for GeoCoding (Places) required."; an answer
with no result returns nothing; an answer with an error status raises "Unable to geolocate, received the error: &lt;error
message&gt;" with the instruction to enable billing and the required capabilities. A provider name that has no
implementation raises "Provider &lt;name&gt; is not implemented for geolocation service." When one or more records could
not be located, the user receives the notification "No match found for &lt;names&gt; address(es)." Reverse lookup is
disabled during automated tests with "OpenStreetMap calls disabled in testing environment."

**Records written.** Latitude, longitude and the geolocation date on the record. Changing any address field of a record
clears its coordinates, therefore a stale position is never kept.

**Idempotency.** Read-only at the service; writing the same coordinates twice is harmless.

### Network position of a visitor

**Direction** None; the lookup is local against a database file the installation ships.

**Behavior.** Every request exposes the country, the region, the city, the coordinates and the time zone derived from
the network address of the visitor. The values are used to choose a country in address forms, to restrict pickup point
searches, to set the time zone of a public page, and to record where a page view came from. When the lookup database is
absent, every value is empty and each consumer falls back to its own default.

## Maps

**Direction** Outbound, from the browser. **Trigger** A page containing a map block is displayed.

**Behavior.** The map block is rendered as an embedded frame pointing at the map service with the configured pins; the
service key of the site is delivered to the page by a dedicated endpoint. Without a key the block shows the
configuration prompt instead of a map.

**Authentication.** A site-level key stored in the settings.

**Failure handling.** Entirely client side; a failure shows the placeholder of the service.

## Video embedding

**Direction** Outbound, from the browser for playback; outbound from the server for the thumbnail.

**Trigger.** A user pastes a video address into the editor, or a page containing a video block is displayed.

**Recognition.** The pasted address is matched, in order, against five platform patterns: a video sharing platform
(watch, short, live and shortened forms), a second video platform (with an optional privacy hash), a third video
platform (including its geographic player form), a photograph network (post addresses) and a social network (video,
watch, reel and plug-in forms). An address that matches none produces the message "The provided web address is invalid".

**Embedding address.** The embedding address is rebuilt from the platform, the video identifier and the options:
automatic start, repeat, hidden controls, hidden full screen control, hidden platform logo, hidden sharing controls and
start time. The per-platform rules are:

| Platform | Rules applied |
|---|---|
| First video platform | Related videos off; automatic start adds muted playback and enables the player interface needed for automatic start on telephones; start time in seconds; hidden controls; repeat requires the video to be listed as its own playlist; hidden full screen control. |
| Second video platform | Automatic start adds muted playback and disables automatic pause; do-not-track always on; hidden controls; repeat; the privacy hash is carried over from the pasted address or its query; start time is appended as a fragment. |
| Third video platform | Start time in seconds; the geographic player address is used. |
| Photograph network | The post embedding address is used, with no options. |
| Social network | The video plug-in address is used, with the video identifier. |

**Thumbnail.** The server fetches the thumbnail from the platform (a fixed thumbnail address, an embedding description
document, a thumbnail service or the media address of the post), with a ten second timeout, and stores the processed
image. A failure leaves the video without a thumbnail.

**Idempotency.** Read-only.

## Internet of things devices

**Direction** Two-way, between the browser and the device box on the local network. The platform server does not talk to
the devices.

**Trigger.** A screen that uses a device (a point of sale station, a weighing counter, a quality control step) registers
a listener for that device; a user action sends a command.

**Protocol.** The client keeps one long poll per device box at the address `/iot_drivers/event` of that box, carrying a
session identifier, the list of device identifiers it listens to and the sequence number of the last event it received.
Commands are sent to `/iot_drivers/action` of the same box with the session identifier, the device identifier and the
command data. The poll is restarted whenever the listener set changes, and it is aborted and reopened on each change.

**Failure handling.** A failed call is retried with an increasing delay starting at one and a half seconds and capped at
fifteen seconds; when the caller asked for silent handling no notification is shown, otherwise the user is told the
device is unreachable.

**Idempotency.** Events carry a sequence number, therefore a repeated poll returns the events after that number and no
event is processed twice. Commands are not idempotent; the device decides.

**Server-side bridge.** One server endpoint exists for the simplest case, the weighing scale connected to the machine
that runs the station: it returns the current reading.

## Live call relay

**Direction** Outbound. **Trigger** A participant joins a live call in a channel.

**Behavior.** The platform returns to the client the list of connectivity servers to use. When the telephony credentials
are configured, it asks the telephony provider for short-lived credentials and returns the server list of the provider;
otherwise it returns up to five locally configured servers, each with its kind (connectivity discovery or relay), its
address and, when needed, a user name and a credential. The limit of five exists because browsers refuse longer lists.

**Failure handling.** A refused or failed request to the telephony provider is logged with its status and content, and
the locally configured servers are returned instead, therefore a call still connects on the local network.

**Idempotency.** Read-only; credentials are short-lived by design.

## Inbound webhooks

**Direction** Inbound. **Trigger** A foreign system calls the hook address of an automation rule.

**Address.** Each automation rule whose trigger is the inbound hook owns a random identity, and its address is the base
address of the installation followed by `/web/hook/` and that identity. The identity can be rotated, which invalidates
the previous address immediately.

**Authentication.** Possession of the address. The address is public, exempt from cross-site protection and does not
open a session. Both reading and submitting methods are accepted.

**Payload and record resolution.** The body is read as a structured document; when it cannot be parsed, the query
parameters are used instead. The rule then evaluates its record expression against the payload to decide which record to
run on. The default expression reads the entity name from the payload key `_model` and the record identifier from the
payload key `_id`.

**Failure handling.** An unknown hook identity is answered with the not-found status and a body whose `status` field carries the value `error`.
A record expression that raises, or a record that does not exist, is answered with the server-error status and the same
body, and the platform raises "No record to run the automation on was found." internally. A successful run is answered
with the success status and a body whose `status` field carries the value `ok`. When the rule has call logging enabled, the payload, the
resolution failures and the run failures are each written to the technical log.

**Idempotency.** Not guaranteed: each call runs the actions of the rule again. A caller that must not act twice has to
make the rule itself idempotent, for example by having the actions check the current state of the record first.

## Outbound webhooks

**Direction** Outbound. **Trigger** A server action of the notification kind runs, whether from a button, an automation
rule or a scheduled action.

**Data exchanged.** A submission whose body is a structured document containing: the entity name under `_model`, the
record identifier under `_id`, the action name and identifier under `_action`, and the value of every field the action
selects, read from the record. Values that have no direct structured representation (dates, date and time values, binary
values) are serialized as text. Keys are sorted, therefore the body is stable.

**Timing and reliability.** The call is deferred to after the commit of the transaction: a transaction that rolls back
logs "Webhook call to &lt;address&gt; - cancelled due to a rollback" and sends nothing. The timeout is one second and
the strategy is send and forget: a read timeout is logged as possibly delivered ("Webhook call timed out after 1s - it
may or may not have failed."), any other transport failure or error status is logged as failed, and neither is retried.
An action with no address raises a user error when it runs.

**Authentication.** None by default; the receiving system is expected to treat the address as the secret.

**Idempotency.** The receiver must be idempotent on the pair of entity and record identifier, because the platform does
not retry but may be triggered again by the same business event.

## Publisher account services

Beyond the purchased data services, three small exchanges exist with the account service of the publisher.

| Exchange | Direction | Trigger | Data | Failure handling |
|---|---|---|---|---|
| Account sign-in address | Outbound, as a link | A user opens account management | The installation identifier, the public base address and the requested scope, encoded into the authorization address | None; the address is produced locally |
| Credit purchase address | Outbound, as a link | A user presses the credit action on a service account | The installation identifier, the service name and the hashed account token | None; the address is produced locally |
| Activity indicator summary | Inbound | A management system polls the summary endpoint with a list of installation and key pairs | Per installation: the indicator values it publishes | Installations that are absent, that fail key verification or that run another platform version are left out of the answer |

## Summary of touchpoints

| Integration | Direction | Authentication | Repeat-safe key |
|---|---|---|---|
| Electronic mail sending | Outbound | Server credentials, certificate, or the process credentials | None; guarded by the sent state |
| Electronic mail receiving | Inbound | Mailbox credentials | Message identifier |
| Delivery failure handling | Inbound | None (message content) | Bounce notification handled once per fetched message |
| Purchased text messages | Outbound and inbound reports | Account token | Generated message identifier |
| Direct telephony text messages | Outbound and inbound reports | Account identifier and token; signature on reports | Provider message reference |
| Postal mail | Outbound | Account token | Letter identifier |
| Payment providers | Two-way | Provider credentials; signature on notifications | Transaction reference |
| Shipping carriers | Two-way | Carrier credentials | Tracking number presence |
| Pickup networks | Outbound | Carrier credentials | Read-only |
| Calendar synchronization | Two-way | Delegated authorization | Provider event identifier |
| Delegated sign-in | Two-way | Delegated authorization | Provider and subject pair |
| Directory servers | Outbound | Binding account | Login |
| Contact autocompletion | Outbound | Account token | Read-only |
| Lead enrichment | Outbound | Account token | Enrichment flag on the lead |
| Lead generation | Outbound | Account token | External company identifier |
| Visitor identification | Outbound | Account token | External company identifier |
| Document digitization | Outbound and polling | Account token | Service document reference |
| Text generation | Outbound | Installation identifier | None |
| Tax identification verification | Outbound with callback | Client identifier and token; signed callback token | The number itself |
| Document exchange networks | Two-way | Signed requests; shared notification token | Request identifier |
| Photograph library | Outbound | Access key | Read-only plus a download counter |
| Illustration library | Outbound | Publisher service | Read-only |
| Animated image library | Outbound | Service key | Read-only |
| Anti-robot verification | Outbound | Secret key | Single-use token |
| Print on demand | Two-way | Account key; signature on notifications | Order reference |
| Cloud storage | Browser to service, signed by the platform | Signed addresses | Random blob name |
| Translation platform | Links only | None | Not applicable |
| Currency rates | Outbound | Provider credentials where required | Currency and date |
| Address geolocation | Outbound | Key for the commercial provider | Read-only |
| Maps | Browser to service | Site key | Read-only |
| Video embedding | Browser to service; server for thumbnails | None | Read-only |
| Internet of things devices | Browser to device box | Local network | Event sequence number |
| Live call relay | Outbound | Telephony credentials | Read-only |
| Inbound webhooks | Inbound | Secret address | None |
| Outbound webhooks | Outbound | None | Entity and record identifier |
