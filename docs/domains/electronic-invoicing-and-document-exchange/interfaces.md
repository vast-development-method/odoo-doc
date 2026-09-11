# Interfaces

The interface surface of the Electronic Invoicing and Document Interchange domain: the service operations a client or an integration invokes, the request endpoints the outside world calls, the outbound calls this domain makes, the reports and printed documents, the exported files, the notifications, the scheduled jobs, and the screens described as workflows on views. The records are specified in [entities.md](entities.md), the procedures in [workflows.md](workflows.md), the settings in [configuration.md](configuration.md), and the call contract with the exchange network proxy in [peppol-network.md](peppol-network.md).

---

# 1. Service operations

Every operation below is named with a stable full word identifier. Unless stated otherwise, an operation acts on a set of records and the permissions of section 7 of [configuration.md](configuration.md) apply.

## 1.1 Electronic document framework

| Operation | Inputs | Outputs | Side effects | Errors |
|---|---|---|---|---|
| `export_electronic_document_file` | one Electronic Document | a download of the produced file, named after the format and the accounting document | none | none |
| `process_documents_without_remote_call` | a set of Electronic Documents | none | posts or cancels every selected delivery record whose format needs no remote call; writes the produced attachment, the state, the error text and the blocking level | none |
| `process_documents_with_remote_call` | a set of Electronic Documents, an optional job limit, a flag saying whether to commit between jobs | the number of jobs that were left over | takes a row lock per job, calls the remote service, writes the state, the attachment, the error and the blocking level, commits between jobs when asked | `This document is being sent by another process already.` when the lock cannot be taken |
| `run_sending_scheduled_action` | an optional job limit, twenty by default | none | runs the two operations above and schedules itself again when jobs were left over | none |
| `retry_failed_electronic_documents` | one accounting document | none | clears the error and the blocking level of every delivery record of that document that is in error, then processes the ones that need no remote call | none |
| `request_cancellation` | a set of accounting documents | none | sets the state of the applicable delivery records to `to_cancel` | none |
| `call_off_cancellation` | a set of accounting documents | none | sets the state of those delivery records back to `sent` | none |
| `force_cancellation` | a set of accounting documents | none | cancels the accounting document without waiting for the remote service | none |

## 1.2 Structured file generation and interpretation

| Operation | Inputs | Outputs | Side effects | Errors |
|---|---|---|---|---|
| `export_invoice_file` | one accounting document, one profile | the produced file content and the set of validation messages | none | raises when a tax has an invalid repartition structure |
| `export_invoice_file_name` | one accounting document, one profile | the file name of section 2 of [universal-business-language-mapping.md](universal-business-language-mapping.md) | none | none |
| `export_order_file` | one sales order or one purchase order | the produced file content | none | none |
| `export_point_of_sale_receipt_file` | one receipt | the produced file content and the set of validation messages, which is always empty | none | none |
| `recognise_file_type` | a parsed file | the profile that should decode it | none | none |
| `import_invoice_file` | one accounting document, the file, a flag saying whether the document was just created | none | fills the accounting document, logs the result, stores the file and extracts the embedded documents | refuses with the standard reason when the document already has lines |
| `import_order_file` | one order, the file, a flag saying whether the order was just created | none | fills the order, logs the result and creates an activity when something could not be interpreted | none |
| `group_or_ungroup_lines_by_tax` | a set of accounting documents | none | merges the lines that share a tax into one line per tax, or splits them back | none |

## 1.3 Exchange network

| Operation | Inputs | Outputs | Side effects | Errors |
|---|---|---|---|---|
| `check_partner_endpoint` | one contact, optionally a company | none | runs the participant lookup and writes the verification state for that company, logging the change | none |
| `fill_participant_supported_documents` | a set of contacts | none | writes the published document type identifiers of each participant | none |
| `register_participant` | the registration wizard, optionally the chosen identity verification method | either a redirection to the identity verification service or a notification | creates the credential and sets the participant state | the six refusals of section 7.6 of [peppol-network.md](peppol-network.md) and the seven of section 7.3 |
| `register_sender_as_receiver` | one credential | none | asks the proxy to publish the participant, clears the migration key, sets the state to pending publication and schedules the state poll | `Cannot register a user with a <state> application` |
| `deregister_participant` | one credential | none | drains the pending states and the inbox, cancels the registration, resets the configuration and deletes the credential | none |
| `deregister_to_sender` | one credential | none | drains the pending states and the inbox, asks the proxy to withdraw the publication and sets the state to sender | none |
| `send_documents_to_network` | a credential and a list of payloads | none | writes the message identifier and the state on every document, logs what was sent, moves the sent attachments onto the log entry and schedules the delivery state poll | every proxy error of section 15 of [peppol-network.md](peppol-network.md) |
| `poll_message_states` | a set of credentials | none | writes the delivery state or the error on every document and response awaiting one, and acknowledges the processed identifiers | none |
| `poll_new_documents` | a set of credentials, a flag saying whether a missing reception journal is skipped or refused | none | imports every new message, acknowledges the processed identifiers, notifies the reception journal, verifies the new partners and acknowledges the imported documents | `Please set a journal for Peppol invoices on <company> before receiving documents.` when the flag says refuse |
| `poll_participant_states` | a set of credentials | none | writes the participant state, and archives the credential when the participant no longer exists | none |
| `reconnect_this_database` | one credential | none | takes the connection back, stores the new token and clears the out of step mark | `This connection has been superseded by another database. Register again.` |
| `disconnect_this_database` | one credential | none | soft resets the configuration and deletes the credential | none |
| `send_business_response` | a credential, a set of accounting documents, a status code, a clarification list | none | creates one business response record per accepted entry and logs the result on every document | `At least one reason must be given when rejecting a Peppol invoice.` |
| `reset_webhook` | a set of credentials | none | registers the callback address and a fresh token | none |
| `refresh_published_services` | none | none | brings the published document type set of every receiver in line with the target set | none |
| `fetch_incoming_electronic_invoices` | a set of journals | none | polls the inbox of every receiver credential of the companies of those journals | none |
| `refresh_outgoing_electronic_invoice_state` | a set of journals | none | polls the delivery states of every sendable credential of those companies | none |

## 1.4 Certificates and keys

| Operation | Inputs | Outputs | Side effects | Errors |
|---|---|---|---|---|
| `generate_private_key` | a company, a name, a key kind and size | the created key | creates the key record | none |
| `sign` | a key or a certificate, the payload, the requested formatting | the signature | none | refuses when the certificate has no private key or is outside its validity window |
| `verify` | a key, the payload, the signature | whether the signature is valid | none | none |
| `decrypt` | a private key, the payload | the decrypted payload | none | none |
| `public_key_bytes` | a key, an encoding | the public key in that encoding | none | none |
| `discover_certificate_chain` | a certificate | the chain | creates the issuing certificates that were not yet known, archived | none |

---

# 2. Request endpoints

Every path below is served by the deployment itself.

| Path | Method | Authentication | Purpose | Request | Response |
|---|---|---|---|---|---|
| `/peppol/webhook/new-message` | send | public | the proxy tells the deployment that a message is waiting | a signed callback token as a parameter | empty, with the no content status |
| `/peppol/webhook/message-state-update` | send | public | the proxy tells the deployment that a delivery state changed | a signed callback token as a parameter | empty, with the no content status |
| `/peppol/webhook/user-state-update` | send | public | the proxy tells the deployment that the participant state changed | a signed callback token as a parameter | empty, with the no content status |
| `/peppol/authentication/callback` | fetch | a signed in user | the identity verification service sends the browser back | the kind of verification, the connection request token, an optional verification token and a state | a redirection to the settings screen, or to the main screen when the decision is pending |
| `/peppol/authentication/webhook` | send | public | the identity verification service reports a positive decision taken asynchronously | the kind of verification, the connection request token and a verification token | a structured document holding a status of `connected`, `already_connected`, `failed`, or an error with the bad request status |

The three callbacks never do the work inside the request; each one verifies its token, resolves the credential and triggers the corresponding scheduled action. All three answer with an empty success even when the token fails verification, so that a caller cannot probe which tokens are valid.

**Customer portal.** The account page of the customer portal is extended: when the active company can send over the network, the sending method list offers the network method and the page offers the electronic address scheme list restricted to the schemes available to that contact. When a customer selects the network method, the billing address form additionally requires the scheme, the endpoint and the structured format, and refuses the submission with `That country is not available for Peppol.`, with the endpoint message of the failing validity rule, or with `If you want to be invoiced by Peppol, your configuration must be valid.`

**Notifications pushed to the browser.** During the registration, a message is pushed on a private channel of the contact of the requesting user, carrying a result of `success`, `pending`, `canceled` or `failure` and, on a failure, the refusal message. The screen that is waiting for the registration to finish listens on that channel.

---

# 3. Outbound calls

Every outbound call of this domain goes to the exchange network proxy, and every one of them is specified in section 6.6 of [peppol-network.md](peppol-network.md) with its parameters, its answer and its purpose. Three further outbound calls exist:

| Call | Purpose | Failure handling |
|---|---|---|
| a plain fetch of the participant lookup endpoint of the proxy, unsigned | resolve a participant | logged, produces no answer |
| a plain fetch of the service description address returned by the lookup | read the name of the access point that serves a participant | suppressed, produces no provider name |
| a plain fetch of the service metadata register of the participant | the older way of resolving a participant, kept for a register that answers with a markup document rather than a structured one | logged, produces no answer |

All three time out after ten seconds; the signed calls time out after thirty seconds.

---

# 4. Reports and printed documents

| Report | Content | Where it is used |
|---|---|---|
| the ordinary printed invoice | unchanged; this domain only post processes it | every customer document that is sent |
| the preview printed invoice | the ordinary invoice layout rendered from an imported vendor bill, with the trading partner shown as the issuer, with the header block replaced by a fixed height spacer, with the payment communication shown only when the document carries a payment reference, and with the company address block holding either the company details or the address and the name of the company contact, followed by the tax identification label of the company country and the tax identification number | rendered when an imported vendor bill arrived without a printed document |

The preview report is stamped with the word `Preview` on a coloured panel, followed by the two sentences `This is a visual representation of the markup content.` and `It is not an official document.`

**Post processing of the printed document.**

1. Every delivery record of a posted customer document contributes its payload to the printed document, each format deciding how it embeds it. The payload attachments are readable only by a system administrator, but the embedding runs with the rights of the framework, so a user who may see the accounting document sees the embedded files.
2. A cross industry invoice file is always embedded, under the name `factur-x` with the extension `xml`, with the relationship `/Alternative`.
3. The archival conversion and the archival metadata of section 10 of [cross-industry-invoice-mapping.md](cross-industry-invoice-mapping.md).

---

# 5. Exported files

| File | Produced by | Name | Media type |
|---|---|---|---|
| the structured invoice file | the sending flow, or the download action of a delivery record | the file name of the profile, listed in section 2 of [universal-business-language-mapping.md](universal-business-language-mapping.md) | the markup media type |
| the structured order file | the order download action | the name of the order, a hyphen, and `ubl_bis3` with the extension `xml` | the markup media type |
| the printed document | the ordinary report flow | the ordinary invoice file name | the portable document media type |
| the payload of a delivery record | the framework | whatever the format produced | whatever the format produced |

The structured invoice file is stored on the accounting document in a dedicated binary field and is offered as an extra attachment of the outgoing electronic mail message.

---

# 6. Notifications

| Notification | Trigger | Recipients | Content |
|---|---|---|---|
| welcome message | the company becomes a sender or a receiver | the company | the shipped welcome template |
| electronic invoices received | an inbox batch produced at least one document | the reception journal, through the counter of the general ledger | the number of new documents |
| discussion thread entry on import | a file was imported | the followers of the accounting document | the title of section 10 of [import-mapping.md](import-mapping.md) and the list of collected messages |
| discussion thread entry on sending | a document was handed to the network | the followers of the accounting document | `The invoice has been sent to the Peppol Access Point. The following attachments were sent with the markup file:` and, when some attachments could not be embedded, `Some attachments could not be sent with the markup file:` |
| discussion thread entry on a delivery state change | the delivery state poll returned a state | the followers of the accounting document | `Peppol status update: <state>` |
| discussion thread entry on a response | a response was sent or received | the followers of the accounting document | the seven messages of section 13.7 of [peppol-network.md](peppol-network.md) |
| discussion thread entry on a verification state change | the participant lookup changed the state | the followers of the contact | the previous label, an arrow, the new label, the field name and the company name |
| activity on a partially imported order | an order import collected messages | the user who ran the import | `Some information could not be imported:` followed by the list |
| electronic mail footnote | an invoice is sent by electronic mail to a partner in an eligible country | the recipient of the message | the network footnote described in section 3.3 of [peppol-network.md](peppol-network.md) |

---

# 7. Scheduled jobs

The six scheduled actions, their intervals, their batch sizes and their retrigger rules are in section 6 of [configuration.md](configuration.md). Their procedures are in sections 4, 10, 11 and 14.6 of [workflows.md](workflows.md).

---

# 8. Screens

Every screen is described as the fields it shows, the buttons it offers with their guards, and the filters and groupings it provides. No client technology is implied.

## 8.1 Journal form

Adds one group, visible only when at least one compatible format exists:

| Element | Behaviour |
|---|---|
| the compatible format list | hidden; it only drives the visibility of the group |
| the format list | a multiple choice list of check boxes restricted to the compatible formats |

Two buttons of the journal, both owned by the general ledger and made visible by this domain: "fetch incoming electronic invoices", visible when the journal is the reception journal, the company is a receiver, the journal is a purchase journal and it is not a self billing journal; and "refresh outgoing electronic invoice status", visible when the company can send and the journal is a sale journal, or is a purchase journal marked as self billing.

## 8.2 Accounting document form

| Element | Behaviour |
|---|---|
| the delivery state field | shown next to the journal, hidden while the document is a draft and when there is no state |
| the network state field | shown in the other information page, inside a group visible only to a user who has switched the technical features on and only when the document carries a message identifier; the value is a link that opens the business responses of the document |
| the queued notice | an informational banner listing the formats that will be sent, shown when the document is not a draft and at least one format is pending a remote call, with a button that processes them now |
| the error banner | a danger banner holding the aggregated error message, shown when at least one delivery record is in error at the blocking level, with a button that forces the cancellation when that is allowed, a link that opens the error list when there is more than one error, and a link that retries |
| the warning banner | the same banner in the warning colour, without the force cancellation button |
| the information banner | the same banner in the information colour |
| the network error banner | a danger banner holding `There was an error while sending this invoice via Peppol.`, shown when the network state is `error` |
| the electronic documents page | a read only list of the delivery records showing the file name, the format name and the state, with a download button per row shown when the row has an error whose blocking level is not informational; the page is visible only to a user who has switched the technical features on and only when the document has at least one delivery record |

**Buttons in the status bar**

| Button | Guard |
|---|---|
| "Request EDI Cancellation" (reproduced label; the abbreviation inside it expands to electronic data interchange) | at least one delivery record of a format that needs a remote call is in state `sent` and the format allows cancellation |
| "Call off EDI Cancellation" (reproduced label) | at least one delivery record is in state `to_cancel` |
| "Force Cancel" (reproduced label) | at least one delivery record is in state `to_cancel` and its format allows a forced cancellation |
| "Cancel PEPPOL" (reproduced label) | the network state is `to_send` and the document is not a draft |

## 8.3 Accounting document list and filters

The list offers four optional columns: the delivery state, which is emphasised and coloured by blocking level; the blocking level; the aggregated error message; and the network state.

The search view adds: a grouping by delivery state and a grouping by network state, both restricted to a user who has switched the technical features on for the first one; a filter "Electronic invoicing processing needed" selecting the documents whose delivery state is `to_send` or `to_cancel`; and a filter "Peppol Ready" selecting the posted customer documents whose network state is `ready`.

## 8.4 Electronic document list

A read only list, no creation, no deletion, no editing, showing the format name and the error, with the row coloured by the blocking level: informational in the information colour, warning in the warning colour, error in the danger colour.

## 8.5 The participant settings block

The fields and buttons are in section 3.1 of [configuration.md](configuration.md). The block is inside the accounting settings and is company dependent.

## 8.6 The registration form

A modal form that collects the participant identification, the contact electronic mail address and the mobile number, shows the warnings derived in [entities.md](entities.md) section 9.2, offers the choice between registering for this company and registering under the connection of a parent company when that is possible, and ends with either a button that registers directly or a button that starts the identity verification.

## 8.7 The advanced participant form

A modal form showing the participant identification, the contact electronic mail address and the migration key, together with the list of selectable document types and, when the participant publishes services that this deployment does not know, the notice `The following services are listed on your participant but cannot be configured here. If you wish to configure them differently, please contact support.` followed by their names. Its buttons synchronise the contact electronic mail address with the proxy, withdraw the publication, publish the participant, and deregister.

Publishing answers with the notification whose title is `Registered to receive documents via Peppol.` and whose message is `Your registration on Peppol network should be activated within a day. The updated status will be visible in Settings.`

## 8.8 The rejection form

A modal form collecting the rejection reasons, restricted to the reason list and defaulting to the code `UNR`, and the suggested actions, restricted to the action list and optional. Its single button sends the rejection and returns the result of the cancellation that opened it.

## 8.9 Business response list and form

Both show the message identifier, the response code, the delivery state and the accounting document. They are reached from the network state link of an accounting document.

## 8.10 Contact form

Adds the electronic address scheme, presented as a filtered selection restricted to the schemes available to that contact, and the participant endpoint, both inside the invoice sending settings group. Editing either one, or the structured format, triggers the participant lookup immediately.

## 8.11 Tax form

Adds the tax category code and, only when the chosen category demands one, the tax exemption reason code, both after the country of the tax.

## 8.12 Certificate and key lists

Two simple lists reached from the general settings, each with a form view. The empty state of the certificate list says `Create a first certificate` and the empty state of the key list says `Create a first key`.

## 8.13 The sending wizard

The wizard is owned by the accounts receivable domain. This domain adds:

| Element | Behaviour |
|---|---|
| the network sending method check box | offered when the company can send and the method applies to the document; its label carries ` (Test)` or ` (Demo)` when the operating mode is not the live one |
| the disabled reason on that check box | when the partner is not a valid participant the check box is read only and unchecked, and its label carries ` (Customer not on Peppol)`, or ` (no tax identification number)` when the partner has no tax identification number, or ` (Missing <the scheme label>)` when the scheme is known but the endpoint is missing |
| the attachment widget | shown for a format that embeds attachments; an attachment whose media type is not supported is marked with `Unsupported file type via <the format key>` |
| the placeholder attachment | a placeholder row named after the file the profile will produce, so that the user sees it before it exists |
| the alerts | the six alerts listed below |

**Alerts of the sending wizard**

| Alert | Condition | Message |
|---|---|---|
| configure the company | a network format is selected and the contact of the company has no scheme or no endpoint | `Please fill in your company's tax identification number or Peppol Address to generate a complete markup file.` |
| configure the partner | a network format is selected and the commercial partner has no scheme or no endpoint | `Please fill in partner's tax identification number or Peppol Address.` |
| install the national portal package | a network format is selected and the customer is behind the French public sector portal, whose identification is the scheme `0009` and the endpoint `11000201100044`, and that package is not installed | `Please install the french Chorus pro module to have all the specific rules.` |
| the partner cannot receive | the network method is selected and the verification state of the partner is `not_valid_format` | `Customer is on Peppol but did not enable receiving documents.` |
| what the network is | the company is established in one of Belgium, Finland, Luxembourg, Latvia, the Netherlands, Norway, Sweden or France, cannot send yet, at least one document has not been sent over the network, and no partner is unverified or absent from the network | `You can send this invoice electronically via Peppol.` |
| the partner asked for it | exactly one partner of the documents not selected for the network is a valid participant, at least one document has not been sent, and at least one partner is in a country where the network is the default | `<partner> has requested electronic invoices reception on Peppol.` |

**Before sending.** When the network method is selected and the partner is not a valid participant, the wizard refuses with `Partner doesn't have a valid Peppol configuration.` When every selected document belongs to one company and that company cannot send, the registration form is opened instead of sending. Otherwise every selected document whose network state is `ready` or empty moves to `to_send`.

**Automatic correction of a Belgian identification.** When the partner is not a valid participant, has an endpoint, and its scheme is the Belgian company registry scheme `0208` or the Belgian tax identification scheme `9925`, the other of those two is tried: the value gains the country prefix when moving to the tax identification scheme and loses its first two characters when moving to the company registry scheme. When the alternative passes the endpoint validity rules and resolves to a valid participant, the partner is rewritten with it and re-verified.

## 8.14 The order form

Both the sales order and the purchase order gain a download action that produces the structured order file, and an activity is created on them when an import could not interpret everything.
