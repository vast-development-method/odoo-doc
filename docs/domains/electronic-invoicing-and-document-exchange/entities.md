# Entities

Complete field level specification of every entity owned by the Electronic Invoicing and Document Interchange domain, followed by every field this domain adds to entities owned by other domains.

## Conventions used in this file

Every persistent entity carries the shared fields `identifier` (surrogate integer primary key), `created_on`, `created_by_user`, `last_updated_on` and `last_updated_by_user`. Entities that support archiving additionally carry `active` (boolean, default true; archived records are excluded from default queries). These shared fields are not repeated in the per entity tables below; only entities where `active` has a specific business meaning mention it explicitly.

Field names are full word snake_case. Where the reference field name is an abbreviation, this file uses an expanded name and states the expansion in the "Notes" column of the first table that mentions it. The most important renamings are:

| Expanded canonical name used here | Meaning |
|---|---|
| `electronic_document_format` | The registered format on an Electronic Document. |
| `electronic_documents` | The list of Electronic Document records of a Journal Entry. |
| `electronic_document_state` | The aggregated delivery state of a Journal Entry. |
| `electronic_invoice_markup_file` | The stored structured markup payload of a Journal Entry. |
| `electronic_invoicing_tax_category_code` | The tax category code configured on a Tax. |
| `peppol_electronic_address_scheme` | The code that says which kind of identifier the participant endpoint holds. |
| `participant_state` | The registration state of a company on the exchange network. |
| `network_document_state` | The exchange state of one Journal Entry on the network. |

Relations are described with their deletion behaviour: `restrict` blocks deletion of the target, `cascade` deletes the dependent record, `set null` clears the link.

---

# 1. Electronic Document

**Canonical name:** Electronic Document. **Identifier:** `electronic_document`. **Kind:** stored record.

One record per accounting document per registered format. It is the unit of work of the delivery framework: it carries the produced payload, its delivery state, the last error and the severity of that error.

## 1.1 Fields

| Field | Type | Required | Default | Stored or derived | Copied on duplicate | Tracked | Visibility | Meaning |
|---|---|---|---|---|---|---|---|---|
| `journal_entry` | many_to_one to Journal Entry, deletion `cascade`, indexed | yes | none | stored | no | no | all users with read access to the model | The accounting document this delivery record belongs to. |
| `electronic_document_format` | many_to_one to Electronic Document Format, deletion `restrict` | yes | none | stored | no | no | all users with read access to the model | The format that produces and transmits the payload. |
| `attachment` | many_to_one to Attachment, deletion `set null` | no | none | stored | no | no | system administrators only | The file produced by the format when the accounting document was posted and this record was processed. Restricting visibility to administrators is deliberate: the payload may contain a signature or a state assigned by an authority, and users must reach it only through the accounting document. |
| `state` | selection: `to_send`, `sent`, `to_cancel`, `cancelled` | no | empty | stored | no | no | all users with read access to the model | The delivery state. Section 1.4 gives the transition table. |
| `error` | rich_text | no | empty | stored | no | no | all users with read access to the model | The text of the last error raised while processing this record. Cleared on every successful processing, on retry and on reset to draft. |
| `blocking_level` | selection: `info`, `warning`, `error` | no | empty | stored | no | no | all users with read access to the model | Severity of `error`. `info` means the record is not blocked and the current step succeeded. `warning` means an error occurred that does not prevent the current operation from succeeding. `error` means the error blocks the current operation and the record is skipped by the scheduled action until it is retried. |
| `name` | text, derived | no | none | derived from `attachment.name` | not applicable | no | all users with read access to the model | Display name of the produced file. |
| `electronic_document_format_name` | text, derived | no | none | derived from `electronic_document_format.name` | not applicable | no | all users with read access to the model | Display name of the format. |
| `electronic_document_content` | binary, derived | no | none | derived, recomputed when `journal_entry`, `error` or `state` changes | not applicable | no | all users with read access to the model | A preview of the payload that would be produced right now. Section 1.3 gives the derivation. |

## 1.2 Identity, ordering and uniqueness

- **Uniqueness constraint** `unique_document_per_entry_per_format`: `UNIQUE(electronic_document_format, journal_entry)`. Violation message: `Only one edi document by move by format`. A replacement must reproduce a uniqueness failure with an equivalent message; the message is shown only when a second record would be created for the same pair.
- **Index:** `journal_entry` is indexed, because the framework reads all records of one accounting document on every state recomputation and reads all records of a set of journals when a format is switched off.
- **Default ordering:** by `identifier` ascending. No business ordering is defined; batches are grouped by key, not by order.
- **Display name rule:** the value of `name`, that is the file name of the attachment, or empty when no attachment exists yet.
- **Company scoping:** indirect, through `journal_entry.company`. Record level filtering follows the Journal Entry rules.
- **Archiving:** not supported. Records are deleted when the accounting document is deleted (`cascade`), when a format is switched off on a journal and the removed format needs no remote call, and when an accounting document that still had records in `to_send` is reset to draft.

## 1.3 Derivation of `electronic_document_content`

```
for each electronic_document:
    result = empty
    if state IN ("to_send", "to_cancel"):
        configuration_errors = format.check_document_configuration(journal_entry)
        if configuration_errors is not empty:
            result = the configuration error messages joined with a line break, encoded as binary
        else:
            applicability = format.get_document_applicability(journal_entry)
            if applicability exists and applicability has a "content_preview" operation:
                result = applicability.content_preview(journal_entry)
    electronic_document_content = result
```

The preview is therefore empty for records in `sent` or `cancelled`, and it shows the blocking configuration errors instead of a payload when the accounting document cannot be exported.

## 1.4 State machine

| From state | Trigger | Guard | To state | Side effects |
|---|---|---|---|---|
| (none) | Posting an accounting document whose journal declares an applicable format | `format.get_document_applicability(entry)` returns a non empty result, and `format.check_document_configuration(entry)` returns no error | `to_send` | A new record is created with `state = "to_send"`. |
| `sent`, `to_cancel`, `cancelled` | Posting again the same accounting document with the same format | same guard | `to_send` | The existing record is reused: `state` is set to `to_send` and `attachment` is cleared. |
| `to_send` | Processing without a remote call (`format.needs_remote_call()` is false) | the record is not at blocking level `error` | `sent` or stays `to_send` | Section 1.5. |
| `to_send` | The sending scheduled action, or the manual "Process now" operation | the record is not at blocking level `error` and the accounting document is posted | `sent` on success, stays `to_send` on failure | Section 1.5. |
| `sent` | "Request Cancellation" on the accounting document | the format needs a remote call and the applicability declares a `cancel` operation and the accounting document passes its fiscal lock date check | `to_cancel` | `error` and `blocking_level` are cleared; a note "A cancellation of the EDI has been requested." is posted on the accounting document with the acronym expanded to "electronic document". |
| `to_cancel` | "Call off Cancellation" on the accounting document | the applicability declares a `cancel` operation | `sent` | `error` and `blocking_level` are cleared; a note "A request for cancellation of the EDI has been called off." is posted with the acronym expanded. |
| `to_cancel` | The sending scheduled action, or the manual "Process now" operation | the record is not at blocking level `error` and the accounting document is posted | `cancelled` on success, stays `to_cancel` on failure | Section 1.6. |
| `to_send`, `to_cancel`, `sent` | Cancelling the accounting document | the record is not in `sent` | `cancelled` | `error` and `blocking_level` are cleared. |
| `sent` | Cancelling the accounting document | the record is in `sent` | `to_cancel` | `error` and `blocking_level` are cleared, then records that need no remote call are processed immediately. |
| `to_send` | Resetting the accounting document to draft | the accounting document is allowed to be reset (section 12.6) | (record deleted) | `error` and `blocking_level` are first cleared on all records of the document, then records in `to_send` are deleted. |

## 1.5 Post processing of a send batch

For every record of the batch, the operation returns a per accounting document result:

1. If the result carries an attachment, the previous `attachment` is remembered, the new one is stored, and the previous one is scheduled for deletion when it is not linked to any business record (no model name and no record identifier). Such attachments have no traceability for the user and must not accumulate.
2. If the result says success, the record is written with `state = "sent"`, `error` cleared and `blocking_level` cleared.
3. Otherwise the record keeps its state, `error` receives the returned error text (or is cleared when the result carries none), and `blocking_level` receives the returned level, defaulting to `error` when the result carries an error text but no level, and cleared when the result carries no error text.
4. The collected orphan attachments are deleted.

## 1.6 Post processing of a cancellation batch

1. On success, the record is written with `state = "cancelled"`, `error` cleared, `attachment` cleared and `blocking_level` cleared; the previous attachment is collected for deletion when it is not linked to a business record.
2. On success, if the accounting document is posted and every one of its electronic documents is either `cancelled` or belongs to a format that needs no remote call, the accounting document is reset to draft and then cancelled. This is the only place where the framework changes the state of the accounting document itself.
3. On failure, the record keeps its state and receives the returned error text and blocking level, defaulting to `error` when an error text is present.
4. The collected orphan attachments are deleted.

## 1.7 Operations

| Operation | Inputs | Behaviour | Errors |
|---|---|---|---|
| `download_payload` | one record | Returns a download instruction pointing at the `electronic_document_content` of the record. | none |
| `prepare_jobs` | a set of records | Groups records into jobs. Section 1.8. | none |
| `process_job` | one job | Executes the job. Section 1.9. | Raises `All electronic documents of a job should have the same state` when the job mixes states; a replacement must treat this as an internal consistency failure. |
| `process_documents_without_remote_call` | a set of records | Keeps only records whose format needs no remote call, prepares jobs from them and runs every job immediately, inside the current transaction. | none |
| `process_documents_with_remote_call` | a set of records, an optional maximum job count, a commit flag | Keeps only records whose format needs a remote call, prepares jobs from them, takes the first `job_count` jobs, locks each job, processes it, optionally commits between jobs, and returns the number of jobs left unprocessed. | Raises `This document is being sent by another process already. ` when the lock cannot be taken and the commit flag is false. When the commit flag is true the job is silently skipped and left for the next run. |
| `run_sending_scheduled_action` | an optional maximum job count | Section 1.10. | none |
| `attachments_for_mailing` | one record | Returns the information needed to attach the payload to an outgoing message. Section 1.11. | none |

## 1.8 Job preparation and batching

```
to_process = empty map
for (state, flow) in [("to_send", "post"), ("to_cancel", "cancel")]:
    candidates = records where state = state AND blocking_level <> "error"
    for document in candidates:
        applicability = document.format.get_document_applicability(document.journal_entry) or empty
        key = [document.format, state, document.journal_entry.company]
        if applicability has an operation named flow + "_batching":
            key = key + list(applicability[flow + "_batching"](document.journal_entry))
        else:
            key = key + [document.journal_entry.identifier]
        batch = to_process.setdefault(tuple(key), {documents: empty set,
                                                   operation: applicability[flow]})
        batch.documents = batch.documents + document
return the values of to_process
```

Consequences a replacement must preserve:

- A format that does not declare a batching key produces **one job per accounting document**, because the accounting document identifier is part of the key.
- A format that declares a batching key produces **one job per distinct key value**, so several accounting documents of the same company and the same state travel in one remote call.
- Records at blocking level `error` are never put into a job. They are only reachable through the retry operation, which clears the error and the level.
- A job never mixes formats, never mixes states and never mixes companies.

## 1.9 Job execution

1. If the job carries no operation, the operation used is a stub that returns success for every accounting document. This is how formats that only need the file to exist behave.
2. The job asserts that all records share one format and one company, and that all records share one state.
3. The journal items of the accounting documents are flushed, so the tax details are visible to the export.
4. When the state is `to_send`, the accounting documents are put in the "send only when ready" guard of the accounts receivable domain while the operation runs, then the post processing of section 1.5 runs.
5. When the state is `to_cancel`, the operation runs without that guard, then the post processing of section 1.6 runs.

## 1.10 The sending scheduled action

```
documents = all electronic documents where
    state IN ("to_send", "to_cancel")
    AND journal_entry.state = "posted"
    AND blocking_level <> "error"
remaining = documents.process_documents_with_remote_call(job_count)
if remaining > 0:
    schedule the sending scheduled action to run again as soon as possible
```

The scheduled action is shipped inactive and is switched on automatically the first time a format that needs a remote call is created. Its default job count is 20 and its default interval is one day; the self retrigger is what makes a large queue drain quickly.

## 1.11 Attachment handling for outgoing messages

```
attachment = the payload of this record, read with administrator rights
if attachment is empty: return empty
if attachment has no model name or no record identifier: return empty
if more than one accounting document is active in the context:
    return { attachments: [ (attachment.name, attachment.content) ] }
return { attachment_identifiers: [ attachment.identifier ] }
```

Returning the identifier links the existing attachment to the sending wizard; returning the content makes the wizard create a new attachment. In mass sending mode the identifier form is not usable because the wizard removes attachment identifiers from template values, so the content form is used.

---

# 2. Electronic Document Format

**Canonical name:** Electronic Document Format. **Identifier:** `electronic_document_format`. **Kind:** stored record.

A registered format. The record itself is little more than a name and a code; the behaviour is attached to the code by the implementation. A replacement may store the behaviour selector in the record or in a registry, provided every operation below can be resolved from the record.

## 2.1 Fields

| Field | Type | Required | Default | Stored or derived | Copied on duplicate | Tracked | Visibility | Meaning |
|---|---|---|---|---|---|---|---|---|
| `name` | text | no | empty | stored | yes | no | all users | Display name shown on journals and in error messages. |
| `code` | text | yes | none | stored | yes | no | all users | Stable technical selector of the behaviour. |

## 2.2 Constraints

- **Uniqueness constraint** `unique_format_code`: `UNIQUE(code)`. Violation message: `This code already exists`.
- **Default ordering:** by `identifier` ascending.
- **Display name rule:** `name`.
- **Company scoping:** none. Formats are global.
- **Archiving:** not supported.

## 2.3 Creation side effects

On creation of one or more formats:

1. If the application registry is not fully loaded, a flag is set so that the journal recomputation is deferred to the end of the loading sequence. This matters because the helper operations of a format are only complete once every localisation package has been loaded.
2. Otherwise every journal is reloaded and its `electronic_document_formats` and `compatible_electronic_document_formats` are recomputed.
3. If any created format needs a remote call, the sending scheduled action is switched on.

At the end of the loading sequence, if the deferral flag is set, it is cleared and every journal is recomputed.

## 2.4 Behaviour operations (extension points)

Each operation is described by what a concrete format must return. Default results are the ones used when a format does not override the operation.

| Operation | Inputs | Default result | Purpose |
|---|---|---|---|
| `get_document_applicability(entry)` | one accounting document | empty | Returns empty when the format does not apply to this document. Otherwise returns a structure with any of: `post` (the operation called for records in `to_send`), `cancel` (the operation called for records in `to_cancel`), `post_batching` (an operation returning the batching key of the sending flow), `cancel_batching` (an operation returning the batching key of the cancellation flow), `content_preview` (the operation used by the payload preview). |
| `needs_remote_call()` | none | false | True when the format must be produced asynchronously through a remote service. Decides which processing path is used, whether the format contributes to `electronic_document_state`, whether the accounting document may be reset to draft or resequenced, and whether its attachment may be deleted. |
| `is_compatible_with_journal(journal)` | one journal | `journal.type = "sale"` | Decides whether the format may be switched on for that journal at all. |
| `is_enabled_by_default_on_journal(journal)` | one journal | true | Decides whether a compatible format is switched on automatically. |
| `check_document_configuration(entry)` | one accounting document | empty list | Returns the list of blocking configuration error messages. A non empty list blocks posting. |
| `prepare_printed_report(writer, document)` | a portable document writer and one electronic document | nothing | Lets the format embed its payload inside the rendered portable document of a customer facing document. |
| `format_error_message(title, errors)` | a title and a list of messages | `title` followed by an unordered list of the escaped messages | Builds the rich text stored in `error`. |

`post` and `cancel` operations receive the set of accounting documents of the job and return, per accounting document, a result with the optional keys `success` (boolean), `error` (text), `blocking_level` (`info`, `warning` or `error`) and `attachment`.

---

# 3. Electronic Interchange Proxy User

**Canonical name:** Electronic Interchange Proxy User. **Identifier:** `electronic_interchange_proxy_user`. **Kind:** stored record.

The credential of one company on one proxy service in one operating mode. A proxy user has a unique participant identification on that service, which is how an inbound document addressed to the company is routed back. It also owns the private key with which inbound payloads are decrypted, because the proxy encrypts every stored payload with the matching public key.

## 3.1 Fields

| Field | Type | Required | Default | Stored or derived | Copied on duplicate | Tracked | Visibility | Meaning |
|---|---|---|---|---|---|---|---|---|
| `active` | boolean | no | true | stored | no | no | all users with read access | Archiving flag. An archived proxy user is ignored by every scheduled action and by every lookup. |
| `client_identifier` | text | yes | none | stored | no | no | all users with read access | The identifier assigned by the proxy service when the user was created. It is sent in a header on every request. |
| `company` | many_to_one to Company, deletion `restrict`, indexed | yes | the active company | stored | no | no | all users with read access | Owner of the credential. |
| `participant_identification` | text | yes | none | stored | no | no | all users with read access | The unique value that identifies this user on the proxy service. For the document exchange network it is the electronic address scheme code, a colon, and the endpoint value, for example `0208:0477472701`. For other services it is typically the tax identification number. |
| `private_key` | many_to_one to Digital Key, deletion `restrict`, limited to private keys | yes | none | stored | no | no | all users with read access | The key whose public half was handed to the proxy at registration. Used to decrypt the symmetric key of every inbound payload and, in the asymmetric authentication mode, to sign requests. |
| `refresh_token` | text | no | empty | stored | no | no | system administrators only | The rotating shared secret used to sign requests in the default authentication mode. It expires after twenty four hours on the proxy side so that two copies of the same database cannot both keep using it. |
| `is_token_out_of_step` | boolean | no | false | stored | no | no | all users with read access | Set when the proxy has told this database that its token no longer matches, which happens after a database restore or an unneutralised copy. While it is true every call is refused locally with an explanatory message. |
| `token_step_version` | integer | no | 0 | stored | no | no | all users with read access | Monotonic counter sent to the proxy when marking the connection out of step and when resynchronising, so that the proxy can tell which database asked last. |
| `proxy_type` | selection, extended by each package that adds a service; the exchange network package adds `peppol` with deletion behaviour `cascade` | yes | none | stored | no | no | all users with read access | Which proxy service this credential belongs to. |
| `operating_mode` | selection: `prod` (Production mode), `test` (Test mode), `demo` (Demo mode) | no | empty | stored | no | no | all users with read access | Which environment the credential points at. In demonstration mode no request ever leaves the platform. |

## 3.2 Constraints and indexes

- **Uniqueness constraint** `unique_client_identifier`: `UNIQUE(client_identifier)`. Violation message: `This id_client is already used on another user.` with the abbreviation expanded to "client identifier".
- **Partial unique index** `unique_active_company_proxy`: `UNIQUE(company, proxy_type, operating_mode) WHERE active IS TRUE`. Violation message: `This company has an active user already created for this EDI type` with the acronym expanded to "electronic data interchange". A company therefore has at most one live credential per service per environment, and archiving an old credential frees the slot.
- **Default ordering:** by `identifier` ascending.
- **Display name rule:** `participant_identification`.
- **Company scoping:** a record rule restricts visibility to `company` being the active company or one of its parents.
- **Archiving:** supported through `active`. Deregistration deletes the record; loss of the remote user archives it.

## 3.3 Operations

| Operation | Inputs | Behaviour | Errors |
|---|---|---|---|
| `proxy_addresses()` | none | Returns the map from service to environment to base web address. Each package that adds a service adds its own entry. The exchange network package adds the production address of the exchange network proxy, its acceptance address, and the literal value `demo` for demonstration mode. | none |
| `server_address(proxy_type, operating_mode)` | optional service and environment, defaulting to the ones on the record | Looks the address up in that map. A missing entry is an internal error, not a user error. | none |
| `proxy_users_of(company, proxy_type)` | a company and a service | Returns the credentials of that company for that service. | none |
| `participant_identification_for(company, proxy_type)` | a company and a service | Returns the value that will identify the company on that service, raising a user error when the company is not configured. For the exchange network it returns the scheme code, a colon and the endpoint. | `Please fill in the electronic address scheme code and the Participant Identifier code.` |
| `call(address, parameters, authentication_mode)` | a full web address, a parameter structure, and either the shared secret mode or the private key mode | Section 3.4. | Section 3.4. |
| `registration_parameters(company, proxy_type, private_key)` | a company, a service and a private key | Returns the database unique identifier, the company identifier, the participant identification, the public key in text form and the service name. | none |
| `register(company, proxy_type, operating_mode)` | a company, a service and an environment | Generates a two thousand and forty eight bit private key named after the service, the environment and the company identifier, resolves the participant identification, and, outside demonstration mode, refuses when a credential already exists for that triple, then calls the create user endpoint and creates the record with the returned client identifier and rotating token. In demonstration mode the response is simulated with the client identifier `demo` followed by the company identifier and the service name, and the token `demo`. | `A user already exists with this identification.`; `A user already exists with theses credentials on our server. Please check your information.` when the remote service reports an existing user; otherwise the message returned by the remote service. |
| `renew_token()` | one record | Takes a row lock; if the lock cannot be taken the operation returns without doing anything, because another transaction is already renewing. Otherwise calls the renew token endpoint and stores the new rotating token. A remote error is logged and not raised, because it means another copy of the database renewed first and this copy must not keep querying. | none raised to the user |
| `decrypt(payload, encrypted_symmetric_key)` | a payload and the symmetric key encrypted with the public half of `private_key` | Decrypts the symmetric key with the private key, then decrypts the payload with that symmetric key using an authenticated symmetric scheme. | Errors of the key operations. |

## 3.4 The proxy call contract

Every call is a structured remote procedure request with the fields `jsonrpc` set to `2.0`, `method` set to `call`, `params` set to the parameter structure, and `id` set to a fresh random hexadecimal value. The call is sent by an unencrypted or encrypted transport with a thirty second timeout and the content type header of structured data.

Authentication is added by a signer:

1. The client identifier is put in a header named `exchange-client-identifier`.
2. The current epoch second is put in a header named `exchange-timestamp`.
3. The signed message is the timestamp, a vertical bar, the path of the address, a vertical bar, the client identifier, a vertical bar, the query parameters of the address serialised as a key sorted structure, a vertical bar, and the request body serialised as a key sorted structure.
4. In the default mode the signature is the hexadecimal keyed hash of that message with the rotating token, decoded from its text form, as the key and a two hundred and fifty six bit digest. The header `exchange-signature-type` is set to `hmac`.
5. In the private key mode the signature is the base sixty four encoded signature of that message by `private_key`. The header `exchange-signature-type` is set to `asymmetric`. This mode exists only to recover a connection whose rotating token is out of step.
6. When the record has no client identifier the request is sent unsigned.

Response handling:

1. When the record is in demonstration mode, the call is refused before any network access with the internal code `block_demo_mode` and the message `Can't access the proxy in demo mode`. This is the last barrier; callers are expected to intercept demonstration mode earlier.
2. A transport failure, an unreadable body, a missing scheme, a timeout or a failing status code raises the internal code `connection_error` with the message `The url that this service requested returned an error. The url it tried to contact was %s` where the placeholder is the address, with the abbreviation expanded to "web address".
3. A transport level error object with code 404 raises `connection_error` with the message `The url that this service tried to contact does not exist. The url was “%s”` with the abbreviation expanded.
4. Any other transport level error object raises `connection_error` with the message `The url that this service requested returned an error. The url it tried to contact was %(url)s. %(error_message)s` with the abbreviation expanded.
5. A business level error inside the result is inspected. Code `refresh_token_expired` triggers a token renewal, commits it so that it is not lost, and retries the same call once in the default authentication mode. Code `no_such_user` archives the credential, because the participant identification was claimed by somebody else. Code `invalid_signature` raises the message `Failed to connect to the access point server. This might be due to another connection to the access point server. It can occur if you have duplicated your database. \n\nIf you are not sure how to fix this, please contact our support.` Any other code raises with its own message.
6. Otherwise the result structure is returned to the caller.

## 3.5 Exchange network specific operations

These operations exist on the credential once the exchange network package is installed. They are specified in [peppol-network.md](peppol-network.md); the list here is the contract surface.

`network_proxy_types()`, `network_endpoint(path, proxy_type)`, `call_network(path, parameters)`, `mark_connection_out_of_step()`, `reconnect_this_database()`, `disconnect_this_database()`, `sendable_states()`, `poll_new_documents(skip_when_no_journal)`, `poll_message_states()`, `poll_participant_state()`, `reset_webhook()`, `import_received_document(attachment, state, message_identifier, journal)`, `duplicate_message_identifiers(identifiers)`, `process_new_messages(messages)`, `post_process_new_messages(entries)`, `documents_awaiting_state(batch_size)`, `process_message_states(messages, index)`, `register_sender_as_receiver()`, `deregister_participant()`, `downgrade_to_sender()`, `deregister_services(package)`, `services()`, `generate_webhook_token(company)`, `user_from_token(token, address)`, `send_response(entries, status, clarifications)`, `extract_response_information(document)`.

---

# 4. Digital Certificate

**Canonical name:** Digital Certificate. **Identifier:** `digital_certificate`. **Kind:** stored record.

A loaded certificate. The record accepts three input encodings and normalises them to one text encoding, extracts the identifying attributes, discovers the issuing certificates in the same upload and stores them as archived records, and exposes signing operations.

## 4.1 Fields

| Field | Type | Required | Default | Stored or derived | Copied on duplicate | Tracked | Visibility | Meaning |
|---|---|---|---|---|---|---|---|---|
| `name` | text | no | empty | stored | yes | no | system administrators | Display name. |
| `content` | binary | yes | none | stored, writable | yes | no | system administrators | The uploaded file, in the binary encoding, the certificate container encoding, or the text bundle encoding. |
| `container_password` | text | no | empty | stored | yes | no | system administrators | Password used to open a certificate container or an encrypted text bundle. |
| `private_key` | many_to_one to Digital Key, deletion `restrict`, limited to private keys, same company | no | derived, writable | derived from `normalised_certificate`, stored, user overridable | yes | no | system administrators | The private key found inside the uploaded container or bundle, or the one chosen by the user. |
| `public_key` | many_to_one to Digital Key, deletion `restrict`, limited to public keys, same company | no | empty | stored | yes | no | system administrators | Used only when the public key inside the certificate is wrong. When set it is used instead of the one inside the certificate. |
| `scope` | selection, base value `general` (General), extended by localisation packages | no | empty | stored | yes | no | system administrators | Which regime the certificate may be used for. |
| `original_format` | selection: `der` (binary encoding), `pem` (text encoding), `pkcs12` (container encoding) | no | derived | derived from `content` and `container_password`, stored | not applicable | no | system administrators | Which encoding the upload used. |
| `normalised_certificate` | binary | no | derived | derived from `content` and `container_password`, stored | not applicable | no | system administrators | The leaf certificate re-encoded in the text encoding. Every other derivation reads this field. |
| `subject_common_name` | text | no | derived | derived, stored | not applicable | no | system administrators | The common name of the subject, falling back to the serial number when the subject has none. |
| `serial_number` | text | no | derived | derived, stored | not applicable | no | system administrators | The serial number, used in signed documents and in duplicate detection. |
| `valid_from` | datetime | no | derived | derived, stored | not applicable | no | system administrators | Start of the validity window, in coordinated universal time. |
| `valid_until` | datetime | no | derived | derived, stored | not applicable | no | system administrators | End of the validity window, in coordinated universal time. |
| `loading_error` | long_text | no | derived | derived, stored | not applicable | no | system administrators | Empty when the certificate loaded. Otherwise the message explaining why it did not. |
| `is_valid` | boolean, derived | no | derived | derived from `valid_from`, `valid_until` and `loading_error`, not stored, searchable | not applicable | no | system administrators | True when both dates exist, `loading_error` is empty, and the current instant lies inside the window. |
| `active` | boolean | no | true | stored | yes | no | system administrators | Set to false to archive. Discovered issuing certificates are created archived. |
| `company` | many_to_one to Company, deletion `cascade` | yes | the active company | stored | yes | no | system administrators | Owner. |
| `country_code` | text, derived | no | derived from `company.country_code` | derived | not applicable | no | system administrators | Convenience field used by localisation rules. |
| `issuer_certificate` | many_to_one to Digital Certificate, same company | no | derived | derived from `normalised_certificate`, `subject_common_name` and `company`, not stored | not applicable | no | system administrators | The certificate that issued this one, when it is present in the database. |

## 4.2 Ordering, identity and company rules

- **Default ordering:** `valid_until` descending, so the freshest certificate comes first.
- **Display name rule:** `name`.
- **Company consistency:** the company of `private_key`, `public_key` and `issuer_certificate` must match the company of the certificate. A replacement must enforce this on write.
- **Company scoping:** a record rule allows records with no company or with a company equal to the active company or one of its parents.
- **Archiving:** supported. Searching for issuing certificates deliberately includes archived records.

## 4.3 Derivations

**`original_format`, `normalised_certificate`, `subject_common_name`, `serial_number`, `valid_from`, `valid_until`, `loading_error`** are produced by one parsing step:

```
if content is empty:
    clear normalised_certificate, subject_common_name, original_format,
          valid_from, valid_until, serial_number, loading_error
    stop
password = container_password encoded as text bytes, or none
leaf, additional, original_format = parse(content, password)
if leaf is empty:
    clear the same fields
    if container_password is set:
        loading_error = "This certificate could not be loaded. Either the content or the password is erroneous."
    stop
certificate = load leaf
loading_error = empty
normalised_certificate = leaf
serial_number = the serial number of certificate
subject_common_name = common name of the subject of certificate, or the serial number when absent
valid_from = the not-before instant of certificate, expressed in coordinated universal time
valid_until = the not-after instant of certificate, expressed in coordinated universal time
```

The parsing step tries the encodings in a fixed order, and the first that succeeds decides `original_format`:

1. The binary encoding. Result: the leaf, no additional certificates, format `der`.
2. The container encoding, opened with the password. Result: the leaf, the additional certificates found in the container, format `pkcs12`.
3. The text bundle encoding. Result: the first block of the ordered bundle as the leaf, the remaining blocks as additional certificates, format `pem`.
4. None of them: empty leaf, no additional certificates, no format.

Ordering a text bundle: every certificate block is extracted in file order; a private key is looked for in the same bundle; when one is found, the block whose public half equals the public half of that private key is moved to the front, and every other block keeps its relative order; when no private key is found, the blocks keep their file order.

**`private_key`** is derived from `normalised_certificate`:

```
for each certificate with a normalised certificate:
    password = container_password encoded as text bytes, or none
    key = none
    if original_format = "pkcs12": key = the private key inside the container
    if original_format = "pem":    key = the private key inside the bundle
    if key exists:
        text_key = key re-encoded in the text encoding, unencrypted, in the standard private key structure
        existing = a Digital Key of the same company whose stored content equals text_key
        if existing is empty:
            existing = create a Digital Key named (subject_common_name or name) + ".key"
                       with that content and that company
        private_key = existing
```

**`is_valid`** is true when `valid_from` and `valid_until` are set, `loading_error` is empty and `valid_from <= now <= valid_until`. Searching on it is translated into the condition `normalised_certificate IS NOT NULL AND valid_from <= now AND valid_until >= now AND loading_error = ""`.

**`issuer_certificate`** is derived as follows:

1. Load every certificate of the set that has a normalised certificate and whose issuer common name can be read. Certificates that fail to load are left without an issuer.
2. Search once for candidate certificates: same company family, `subject_common_name` in the set of issuer common names collected, and a non empty normalised certificate. Archived candidates are included.
3. For each certificate, filter the candidates to the same company family, to the exact issuer common name, and exclude the certificate itself so that a self signed certificate is not its own issuer. Sort the remaining candidates by `valid_until` descending, so that the most recently renewed authority wins.
4. Take the first candidate whose key cryptographically signed this certificate. The check requires that the issuer distinguished name of this certificate equals the subject distinguished name of the candidate, and that the signature verifies against the public key of the candidate. The verification branches on the key type: for an elliptic curve key the signature scheme uses the certificate digest algorithm; for a digital signature algorithm key the digest algorithm is used directly; for a key of the ordinary factoring family the padding depends on the declared signature algorithm, using the deterministic padding by default and the probabilistic padding with a mask generation function and two candidate salt lengths, the digest size and the maximum, when the declared algorithm says so; for an Edwards curve key no digest algorithm is used. A key type that is not supported, or a probabilistic padding that fails both salt lengths, yields "could not be checked" rather than "disproven".
5. If no candidate is cryptographically proven, fall back to a candidate whose subject key identifier equals the authority key identifier of this certificate.
6. Store the identifier of the winner, or nothing.

## 4.4 Validation rules

| Rule | Condition | Message |
|---|---|---|
| Certificate and private key compatibility | The public half of `private_key` differs from the public key inside the certificate. | `The certificate and private key are not compatible.` |
| Private key loadable | `private_key` has a loading error. | The loading error of the key. |
| Certificate and public key compatibility | The public half of `public_key` differs from the public key inside the certificate. | `The certificate and public key are not compatible.` |
| Public key loadable | `public_key` has a loading error. | The loading error of the key. |
| Certificate loadable | `content` is set and `normalised_certificate` is empty. | `loading_error` when present, otherwise `This certificate could not be loaded. Please provide the certificate password.` |

The comparison of public halves is done in constant time, so that the check cannot be used as a timing oracle.

## 4.5 Automatic creation of issuing certificates

On creation, for each set of values, the chain of the uploaded content is parsed and every issuing certificate that is not already in the database is created as an extra record. The extra records are created in the same operation as the requested ones and are then dropped from the returned set, so that the caller only sees the certificates it asked for.

On write, when `content` or `container_password` changed, the same discovery runs for every record that has content and no loading error.

The chain is built by walking upward: the leaf is loaded, every additional certificate is indexed by its subject key identifier, and from the leaf the authority key identifier is followed to the parent, repeatedly, stopping when no parent is found or when a certificate would repeat, which prevents an endless loop on a self signed root.

Each missing issuing certificate is created with `name` set to the subject common name (or the serial number when absent) followed by ` (CA)` with the acronym expanded to " (certificate authority)", the same company, the serial number, the subject common name, the certificate content and `active = false`. Duplicates are detected on the pair of serial number and subject common name, including duplicates inside the same chain.

## 4.6 Operations

| Operation | Inputs | Result | Errors |
|---|---|---|---|
| `binary_bytes(formatting)` | an output shape: line wrapped text, plain text, or raw | The certificate in the binary encoding. | none |
| `fingerprint_bytes(digest, formatting)` | a digest name (`sha1` or `sha256`) and an output shape | The fingerprint. | `Unsupported hashing algorithm '<name>'. Currently supported: sha1 and sha256.` |
| `signature_bytes(formatting)` | an output shape | The signature of the certificate itself. | none |
| `public_key_numbers(formatting)` | an output shape | The two public numbers of the public key. Uses `public_key`, else `private_key`, else the key inside the certificate. | Errors of the key operation. |
| `public_key_bytes(encoding, formatting)` | an encoding (binary or text) and an output shape | The public key. Uses `public_key`, else `private_key`, else the key inside the certificate. | `The public key from the certificate could not be loaded.` |
| `sign(message, digest, formatting)` | a message, a digest name and an output shape | The signature of the message by `private_key`. | `This certificate is not valid, its validity has expired.` when `is_valid` is false and there is no loading error, the loading error when there is one, and `No private key linked to the certificate, it is required to sign documents.` when `private_key` is empty. |
| `chain()` | one record | The list starting with this certificate and walking up `issuer_certificate`, stopping when a certificate would repeat. | none |

The three output shapes are: line wrapped base sixty four text in seventy six character lines, plain base sixty four text, and the raw bytes.

---

# 5. Digital Key

**Canonical name:** Digital Key. **Identifier:** `digital_key`. **Kind:** stored record.

A public or private key. The record normalises the upload to the text encoding, detects whether it is public or private, and exposes signing, verification and decryption.

## 5.1 Fields

| Field | Type | Required | Default | Stored or derived | Copied on duplicate | Tracked | Visibility | Meaning |
|---|---|---|---|---|---|---|---|---|
| `name` | text | no | `New key` | stored | yes | no | system administrators | Display name. |
| `content` | binary | yes | none | stored | yes | no | system administrators | The uploaded key, in the binary or text encoding, public or private. |
| `password` | text | no | empty | stored | yes | no | system administrators | Password of an encrypted private key. |
| `normalised_key` | binary | no | derived | derived from `content` and `password`, stored | not applicable | no | system administrators | The key re-encoded in the text encoding: a public key in the subject public key structure, a private key in the standard private key structure, re-encrypted with the same password when one is set. |
| `is_public` | boolean | no | derived | derived from `content` and `password`, stored | not applicable | no | system administrators | True for a public key, false for a private key. |
| `loading_error` | long_text | no | derived | derived, stored | not applicable | no | system administrators | Empty when the key loaded. |
| `active` | boolean | no | true | stored | yes | no | system administrators | Set to false to archive. |
| `company` | many_to_one to Company, deletion `cascade` | yes | the active company | stored | yes | no | system administrators | Owner. |

## 5.2 Loading

Four loaders are tried in order: private key in the binary encoding, private key in the text encoding, public key in the binary encoding, public key in the text encoding. The first that succeeds decides `is_public` and produces the normalised form.

A loader that fails because a password was supplied for an unencrypted key, or because no password was supplied for an encrypted key, is skipped; when no password was supplied at all the pending error message is cleared, so that a user who has not yet typed the password of an encrypted key does not see an error. If every loader fails, `loading_error` is set to `This key could not be loaded. Either its content or its password is erroneous.`

- **Default ordering:** by `identifier` ascending.
- **Display name rule:** `name`.
- **Company scoping:** a record rule allows records with no company or with a company equal to the active company or one of its parents.

## 5.3 Operations

| Operation | Inputs | Result | Errors |
|---|---|---|---|
| `sign(message, digest, formatting)` | a message, a digest name, an output shape | The signature. Elliptic curve keys sign with the curve signature scheme and the digest; ordinary factoring keys sign with the deterministic padding and the digest; Edwards curve keys sign without a digest. | `Make sure to use a private key to sign documents.`; `The private key could not be loaded.`; the record name, a space, a hyphen, a space and `loading_error` when a loading error exists; `Unsupported hashing algorithm '<name>'. Currently supported: sha1 and sha256.`; `Unsupported asymmetric cryptography algorithm '%s'. Currently supported for signature: ED25519, EC and RSA.` |
| `verify(message, signature, digest)` | a message, a signature and a digest name | True when the signature verifies, false when it does not. | `Make sure to use a public key to verify the signature of documents.`; the record name, a space, a hyphen, a space and `loading_error`; `The public key could not be loaded.`; `Unsupported signature algorithm '<name>'. Currently supported: sha1 and sha256.`; `Unsupported asymmetric cryptography algorithm '%s'. Currently supported for signature: EC and RSA.` |
| `public_key_bytes(encoding, formatting)` | an encoding and an output shape | The public half. For a private key the public half is derived after opening it with `password`. | none |
| `public_key_numbers(formatting)` | an output shape | For an elliptic curve key the two coordinates of the public point; for an ordinary factoring key the public exponent and the modulus, each as big endian bytes of the minimal length. | `The public key could not be loaded.`; `Unsupported asymmetric cryptography algorithm '%s'. Currently supported: EC, RSA.` |
| `decrypt(message, digest)` | a payload and a digest name | The decrypted text, using the optimal asymmetric encryption padding with a mask generation function based on the same digest and no label. | `A private key is required to decrypt data.`; `Unsupported hashing algorithm '<name>'. Currently supported: sha1 and sha256.`; `Unsupported asymmetric cryptography algorithm '%s'. Currently supported for decryption: RSA.` |
| `symmetric_decrypt(key, message)` | a symmetric key and a payload | The decrypted payload, using an authenticated symmetric scheme with a timestamped token format. Used to open inbound exchange network payloads. | Errors of the scheme. |
| `generate_elliptic_curve_key(company, name, curve, password)` | a company, an optional name defaulting to `id_ec`, a curve defaulting to the two hundred and fifty six bit prime curve, an optional password | A new record. | `Unsupported curve algorithm '<name>'. Currently supported: SECP256R1.` |
| `generate_factoring_key(company, name, public_exponent, key_size, password)` | a company, an optional name defaulting to `id_rsa`, a public exponent defaulting to sixty five thousand five hundred and thirty seven, a key size defaulting to two thousand and forty eight bits, an optional password | A new record. | `The public exponent should be 65537 (or 3 for legacy purposes).`; `The key size should be at least 512 bytes.` |
| `generate_edwards_curve_key(company, name, password)` | a company, an optional name defaulting to `id_ed25519`, an optional password | A new record. | none |

Generated keys are stored in the text encoding with the standard private key structure, encrypted with the best available algorithm when a password is given and unencrypted otherwise.

---

# 6. Peppol Business Response

**Canonical name:** Peppol Business Response. **Identifier:** `peppol_business_response`. **Kind:** stored record.

One business level response about one document: either a response this company sent about a received vendor bill, or a response a customer sent about an invoice this company issued.

## 6.1 Fields

| Field | Type | Required | Default | Stored or derived | Copied on duplicate | Tracked | Visibility | Meaning |
|---|---|---|---|---|---|---|---|---|
| `network_message_identifier` | text, indexed when not null | no | empty | stored | no | no | invoicing users | The message identifier of the response itself on the exchange network. |
| `response_code` | selection, see section 6.2 | yes | none | stored | no | no | invoicing users | The business level response code carried by the response. |
| `delivery_state` | selection: `processing` (Pending Reception), `done` (Done), `error` (Error), `not_serviced` (Not Serviced) | no | empty | stored | no | no | invoicing users | Where the response stands on the network. |
| `journal_entry` | many_to_one to Journal Entry, deletion `cascade`, indexed when not null | no | none | stored | no | no | invoicing users | The document the response is about. |
| `company` | many_to_one to Company, derived from `journal_entry.company` | no | derived | derived | not applicable | no | invoicing users | Used to scope polling by company. |

## 6.2 Response code values

| Code | Name | Full word alias | Meaning |
|---|---|---|---|
| `AB` | Acknowledgement | `acknowledgement` | The receiver confirms technical receipt of the document. |
| `IP` | In Process | `in_process` | The receiver is processing the document. |
| `UQ` | Under query | `under_query` | The receiver has a question about the document. |
| `CA` | Conditionally accepted | `conditionally_accepted` | The receiver accepts the document under a condition. |
| `RE` | Rejection | `rejection` | The receiver refuses the document; reasons and suggested actions are attached. |
| `AP` | Approval | `approval` | The receiver approves the document for payment. |
| `PD` | Paid | `paid` | The receiver states the document has been paid. |

The platform only produces the acknowledgement, approval and rejection codes. All seven are stored when received, and only the rejection, approval and paid code identifiers `RE`, `AP` and `PD` influence the aggregated document state.

- **Default ordering:** by `identifier` ascending.
- **Display name rule:** the label of `response_code`.
- **Company scoping:** through `journal_entry.company`.
- **Archiving:** not supported.

---

# 7. Peppol Clarification

**Canonical name:** Peppol Clarification. **Identifier:** `peppol_clarification`. **Kind:** stored record.

A shipped code that may be attached to a rejection: either a reason why the document was refused, or an action suggested to the sender so that a corrected document is accepted.

## 7.1 Fields

| Field | Type | Required | Default | Stored or derived | Copied on duplicate | Tracked | Visibility | Meaning |
|---|---|---|---|---|---|---|---|---|
| `list_identifier` | selection: `OPStatusReason` (reason list), `OPStatusAction` (action list) | no | empty | stored | yes | no | invoicing users | Which of the two code lists the record belongs to. |
| `code` | text | no | empty | stored | yes | no | invoicing users | The code value carried in the response. |
| `name` | text | no | empty | stored | yes | no | invoicing users | Short label. |
| `description` | text | no | empty | stored | yes | no | invoicing users | Full explanation. |

The shipped records are listed in [configuration.md](configuration.md).

- **Default ordering:** by `identifier` ascending.
- **Display name rule:** `name`.
- **Company scoping:** none; the code lists are global.

---

# 8. Peppol Rejection Wizard

**Canonical name:** Peppol Rejection Wizard. **Identifier:** `peppol_rejection_wizard`. **Kind:** transient record.

## 8.1 Fields

| Field | Type | Required | Default | Meaning |
|---|---|---|---|---|
| `journal_entries` | many_to_many to Journal Entry | yes | the documents from which the wizard was opened | The received documents to refuse. |
| `reasons` | many_to_many to Peppol Clarification, restricted to the reason list | yes | the shipped record with code `UNR` (commercial transaction not recognised) | The reasons sent to the sender of the document. |
| `suggested_actions` | many_to_many to Peppol Clarification, restricted to the action list | no | empty | The actions suggested to the sender so that a resent document is accepted. Not mandatory. |

## 8.2 Operation

`send_rejection()`:

1. If `reasons` is empty, raise `At least one reason must be given when rejecting a Peppol invoice.`
2. Build the clarification list from `reasons` followed by `suggested_actions`, each entry carrying the list identifier, the code and the name.
3. Group the journal entries by company and, for each company, call `send_response` on the participant credential of that company with the status `RE` and that clarification list.
4. Return the value stored under `cancel_result` in the calling context when present, otherwise true. This is how the wizard, opened from the cancellation of a received document, lets that cancellation finish.

---

# 9. Peppol Registration Wizard

**Canonical name:** Peppol Registration Wizard. **Identifier:** `peppol_registration_wizard`. **Kind:** transient record.

## 9.1 Fields

| Field | Type | Required | Default or derivation | Meaning |
|---|---|---|---|---|
| `company` | many_to_one to Company | yes | the active company | The company being registered. |
| `parent_company` | many_to_one to Company, derived | no | `company.participant_parent_company` when set, otherwise `company` itself | The closest ancestor whose participant configuration may be reused. |
| `parent_company_name` | text, derived from `parent_company.name` | no | derived | Display helper. |
| `selected_company` | many_to_one to Company, derived from `use_parent_connection` | no | `parent_company` when the parent connection is used, otherwise `company` | The company whose configuration fields the wizard reads and writes. |
| `show_parent_connection_choice` | boolean, derived from `parent_company` | no | true when `company` differs from `parent_company` | Whether the choice between registering this company and sending from the parent is offered. |
| `parent_connection_choice` | selection: `use_parent` (Send from parent company), `use_self` (Register this company on peppol) | no | derived from `show_parent_connection_choice`: `use_parent` when the choice is shown and the parent can already send, otherwise `use_self`; writable | The user choice. |
| `use_parent_connection` | boolean, derived | no | true when the choice is shown and the choice is `use_parent` | Convenience flag. |
| `operating_mode` | selection: `demo` (Demo), `test` (Test), `prod` (Live), derived | no | `company.operating_mode(temporary_scheme = peppol_electronic_address_scheme)` | Which environment the registration will use. |
| `proxy_user` | many_to_one to Electronic Interchange Proxy User, derived | no | the first exchange network credential of `company` | Existing credential, if any. |
| `participant_state` | selection, derived from `company.participant_state` | no | derived | Current registration state. |
| `peppol_electronic_address_scheme` | selection, values from the company scheme list minus the retired schemes, plus the currently selected one | yes | `0225` when the company is a French company, otherwise `selected_company.peppol_electronic_address_scheme`; writing it writes the same field on `selected_company` | Which kind of identifier the endpoint holds. |
| `warnings` | structured_data, derived | no | section 9.2 | Non blocking advice shown in the form. |
| `contact_email` | text, derived from `selected_company.participant_contact_email`, writable | yes | derived | Primary contact email of the registration. |
| `phone_number` | text, derived from `selected_company.participant_phone_number`, writable | no | derived | Mobile number, used for identification only. |
| `peppol_endpoint` | text, derived from `selected_company.peppol_endpoint`, writable | yes | derived | The participant endpoint value. |
| `register_as_receiver` | boolean, derived | no | true when the participant is not already published on the network | Whether the registration will also publish the company in the service directory. |
| `external_access_point_provider` | text, derived | no | the service description of the access point that already serves this participant, when one exists | Shown when the participant is already registered elsewhere. |
| `connection_options` | structured_data, derived from the scheme and the endpoint | no | the answer of the "can connect" enquiry | Whether authentication is required and which authentication methods are available. |
| `show_identity_login` | boolean, derived | no | true when a generic or an identity based authentication method is available | Controls the identity login button. |
| `show_direct_buttons` | boolean, derived | no | true when no authentication is required | Controls the direct activation buttons. |

## 9.2 Warning derivation

The warning structure may contain any of these entries:

| Key | Level | Condition | Message |
|---|---|---|---|
| `company_french_warning` | warning | The company is a French company and the approved platform package is not installed. | `To use the Approved Platform for French E-Invoicing install the module '%s'.` where the placeholder is `France - E-Invoicing (Approved Platform)`. When the package is not even present in the catalogue, the message continues with `\nThe module was not found. Please update the app list first.` |
| `company_peppol_endpoint_warning` | warning | A scheme and an endpoint are set and the endpoint fails the advisory check of its scheme. | `The endpoint number might not be correct. Please check if you entered the right identification number.` |
| `company_already_on_directory` | information | The parent connection is not used, a scheme and an endpoint are set, and the participant is already published elsewhere. | `Your company is already registered on an Access Point (%s) for receiving invoices. We will register you with us as a sender only.` |
| `belgian_tax_number_warning` | warning | The scheme is the Belgian value-added tax scheme `9925`. | `You are about to register with your VAT number. Make sure you register with your Company Registry (BCE/KBO) first to be compliant with the new regulation.` with the acronym expanded to "value-added tax number" and the registry names kept. |

## 9.3 On change behaviours

| Field edited | Effect |
|---|---|
| `peppol_endpoint` | Every character that is not a letter or a digit is removed from the typed value. |
| `phone_number` | The typed value is parsed using the country of `selected_company` as the default region and rewritten in the international format. A value that cannot be parsed is left untouched. |

## 9.4 Operations

Full procedures are in [peppol-network.md](peppol-network.md). The surface is: `mandatory_fields_check()`, `forbid_approved_platform_through_network()`, `open_form(reopen)`, `send_notification(title, message)`, `check_connection_answer(answer, chosen_authentication)`, `generate_connect_token(identifier, company)`, `decode_connect_token(token)`, `can_connect()`, `create_connection(identifier, database_identifier, company, authentication_token)`, `company_details(company)`, `register_with_identity_provider()`, `register_participant(chosen_authentication)`.

---

# 10. Peppol Configuration Wizard

**Canonical name:** Peppol Configuration Wizard. **Identifier:** `peppol_configuration_wizard`. **Kind:** transient record.

Advanced maintenance of an existing registration.

## 10.1 Fields

| Field | Type | Required | Default or derivation | Meaning |
|---|---|---|---|---|
| `company` | many_to_one to Company | yes | the active company | The company being maintained. |
| `participant_proxy_user` | many_to_one to Electronic Interchange Proxy User, derived from `company.participant_proxy_user` | no | derived | The credential. |
| `participant_identification` | text, derived from `participant_proxy_user.participant_identification` | no | derived | Read only display of the identification. |
| `participant_state` | selection, derived from `company.participant_state`, writable | no | derived | Current registration state. |
| `participant_contact_email` | text | yes | `company.participant_contact_email` | Editable primary contact email. |
| `participant_migration_key` | text, derived from `company.participant_migration_key`, writable | no | derived | The key handed to the network when taking over a participant from another access point. |
| `service_data` | structured_data, derived, stored on the transient record, writable | no | the services of the credential when the state is `receiver`, otherwise empty | The document types currently published for this participant, as returned by the proxy. |
| `service_information` | rich_text, derived | no | section 10.2 | Explains services that exist remotely but cannot be configured here. |
| `services` | one_to_many to Peppol Service, derived, writable | no | section 10.3 | The selectable document types. |

## 10.2 Service information derivation

When the state is `receiver`, every published document type identifier that is not part of the supported document types of the company is collected. When that collection is non empty the field holds the message `The following services are listed on your participant but cannot be configured here. If you wish to configure them differently, please contact support.` followed by an unordered list of the names of those document types. Otherwise the field is empty.

## 10.3 Services derivation

When the state is `receiver` and `service_data` is non empty, one Peppol Service line is created per supported document type of the company, with the document type identifier, the document type name, and `enabled` set to whether that identifier is present in `service_data`. Otherwise the list is empty.

## 10.4 Operations

| Operation | Behaviour |
|---|---|
| `synchronise_with_proxy()` | When `participant_contact_email` differs from the value on the company, the company value is updated and the update user endpoint is called with the new contact email. Returns true. The service selection is not sent: removing a published service can make the participant non compliant, and every published service is expected to keep working. |
| `remove_from_network()` | When a credential exists, run the full deregistration. Otherwise reset the participant configuration of the company. Returns true. |
| `downgrade_to_sender()` | When a credential exists, unpublish the participant from the service directory and set the state back to `sender`. |
| `upgrade_to_receiver()` | When a credential exists, run the registration of the sender as a receiver, then poll the participant state. When the state became `smp_registration`, return a success notification with the title `Registered to receive documents via Peppol.` and the message `Your registration on Peppol network should be activated within a day. The updated status will be visible in Settings.` |

---

# 11. Peppol Service

**Canonical name:** Peppol Service. **Identifier:** `peppol_service`. **Kind:** transient record.

| Field | Type | Required | Default | Meaning |
|---|---|---|---|---|
| `wizard` | many_to_one to Peppol Configuration Wizard | no | none | Owner of the line. |
| `document_identifier` | text | no | empty | The document type identifier published on the network. |
| `document_name` | text | no | empty | Human readable name of the document type. |
| `enabled` | boolean | no | false | Whether the document type is currently published. |

**Default ordering:** `document_name` ascending, then `identifier` ascending.

---

# 12. Fields added to Journal Entry

The Journal Entry is owned by the [general ledger](../general-ledger/entities.md) domain. This domain adds the following fields.

## 12.1 Framework fields

| Field | Type | Required | Default | Stored or derived | Copied on duplicate | Tracked | Meaning |
|---|---|---|---|---|---|---|---|
| `electronic_documents` | one_to_many to Electronic Document | no | empty | stored inverse | no | no | The delivery records of this accounting document. |
| `electronic_document_state` | selection: `to_send` (To Send), `sent` (Sent), `to_cancel` (To Cancel), `cancelled` (Cancelled) | no | empty | derived from `electronic_documents.state`, stored | no | no | The aggregated state of the delivery records that need a remote call. |
| `electronic_document_error_count` | integer | no | 0 | derived from `electronic_documents.error`, not stored | not applicable | no | How many delivery records currently carry an error. |
| `electronic_document_blocking_level` | selection: `info`, `warning`, `error` | no | empty | derived, not stored | not applicable | no | The worst severity across the delivery records. |
| `electronic_document_error_message` | rich_text | no | empty | derived, not stored | not applicable | no | The message shown on the accounting document. |
| `electronic_document_pending_remote_formats` | long_text | no | empty | derived, not stored | not applicable | no | The comma separated names of the formats that still have work for the scheduled action. |
| `electronic_document_show_cancel_button` | boolean | no | false | derived, not stored | not applicable | no | Whether "Request Cancellation" is offered. |
| `electronic_document_show_call_off_button` | boolean | no | false | derived, not stored | not applicable | no | Whether "Call off Cancellation" is offered. |
| `electronic_document_show_force_cancel_button` | boolean | no | false | derived, not stored | not applicable | no | Whether "Force Cancel" is offered. |

**Derivation of `electronic_document_state`:**

```
states = the distinct states of the delivery records whose format needs a remote call
if states = {"sent"}:        electronic_document_state = "sent"
else if states = {"cancelled"}: electronic_document_state = "cancelled"
else if "to_send" in states:    electronic_document_state = "to_send"
else if "to_cancel" in states:  electronic_document_state = "to_cancel"
else:                           electronic_document_state = empty
```

**Derivation of `electronic_document_error_message` and `electronic_document_blocking_level`:**

```
if error_count = 0:
    message = empty ; level = empty
else if error_count = 1:
    the single record in error supplies both its error text and its blocking level
else:
    levels = the blocking levels of all delivery records
    if "error" in levels:
        message = "<count> Electronic invoicing error(s)" ; level = "error"
    else if "warning" in levels:
        message = "<count> Electronic invoicing warning(s)" ; level = "warning"
    else:
        message = "<count> Electronic invoicing info(s)" ; level = "info"
```

**Derivation of `electronic_document_pending_remote_formats`:** the names, joined by a comma and a space, of the distinct formats that need a remote call among the delivery records whose state is `to_send` or `to_cancel` and whose blocking level is not `error`.

**Derivation of the three button flags:**

- `electronic_document_show_cancel_button`: false when the accounting document is not posted. Otherwise true as soon as one delivery record has a format that needs a remote call, is in state `sent`, and whose applicability declares a `cancel` operation.
- `electronic_document_show_call_off_button`: true as soon as one delivery record has a format that needs a remote call, is in state `to_cancel`, and whose applicability declares a `cancel` operation. The records are read with administrator rights, because the flag must be correct even for a user who cannot read the delivery records.
- `electronic_document_show_force_cancel_button`: delegated to the general ledger rule that decides whether an accounting document may be force cancelled.

## 12.2 Structured file fields

| Field | Type | Required | Default | Stored or derived | Copied on duplicate | Meaning |
|---|---|---|---|---|---|---|
| `electronic_invoice_markup_file` | binary, stored as an attachment | no | empty | stored | **no** | The structured markup payload attached to this accounting document: the exported file for a customer document, or the received file for a vendor document. |
| `electronic_invoice_markup_attachment` | many_to_one to Attachment, derived from `electronic_invoice_markup_file` | no | empty | derived | not applicable | The attachment record that holds the payload, so that its name and raw content can be read. |
| `electronic_invoice_markup_filename` | text, derived from `electronic_invoice_markup_file` | no | empty | derived | not applicable | The name of that attachment. |

`electronic_invoice_markup_file` is added to the list of fields detached when an accounting document is duplicated or when its attachments are detached, so that a copy never inherits the payload of the original.

## 12.3 Exchange network fields

| Field | Type | Required | Default | Stored or derived | Copied on duplicate | Meaning |
|---|---|---|---|---|---|---|
| `network_message_identifier` | text, indexed when not null | no | empty | stored | **no** | The message identifier assigned by the proxy when the document was sent, or the identifier of the inbound message that produced this document. |
| `network_document_state` | selection: `ready` (Ready to send), `to_send` (Queued), `skipped` (Skipped), `processing` (Pending Reception), `done` (Done), `error` (Error); extended with `AB` (Received), `AP` (Approved), `RE` (Rejected) when the business response package is installed | no | empty | derived from `state`, `network_responses` and `network_responses.delivery_state`, stored | **no** | Where the document stands on the network. |
| `network_is_sent` | boolean | no | false | derived from `network_document_state`, not stored | not applicable | True when the document has left the platform, that is when `network_document_state` is none of empty, `ready`, `to_send`, `error` and `skipped`. |
| `network_responses` | one_to_many to Peppol Business Response | no | empty | stored inverse | no | The business responses exchanged about this document. |
| `network_can_send_response` | boolean | no | false | derived, not stored | not applicable | Whether a business response may still be sent about this document. |

**Derivation of `network_document_state`:**

```
step 1 (base):
    if company.participant_can_send
       AND commercial_partner.participant_verification_state = "valid"
       AND state = "posted"
       AND the document is a sale document including receipts
       AND network_document_state is empty:
        network_document_state = "ready"
    else if state = "draft"
            AND the document is a sale document including receipts
            AND NOT network_is_sent:
        network_document_state = empty
    else:
        network_document_state keeps its stored value

step 2 (only with the business response package):
    completed = the response codes of the responses whose delivery_state = "done"
    if completed is empty: keep the result of step 1
    else if "RE" in completed:                    network_document_state = "RE"
    else if "AP" in completed or "PD" in completed: network_document_state = "AP"   # the approval code identifier
    else:                                          network_document_state = "AB"
```

**Derivation of `network_can_send_response`:** true when `network_message_identifier` is set, the document type is a vendor bill or a vendor credit note, no existing response is in state `not_serviced` and no existing response that is not in state `error` carries the code `AP` or `RE`, and the partner supports the response service.

## 12.4 Posting

The posting of an accounting document is extended as follows, after the general ledger posting has completed:

1. For every posted document and for every format declared on its journal, ask the format for its applicability.
2. When applicable, ask the format for its configuration errors. If there are any, raise `Invalid invoice configuration:\n\n%s` where the placeholder is the error messages joined by a line break. The whole posting is rolled back.
3. When applicable and valid, reuse an existing delivery record for that format by setting its state to `to_send` and clearing its attachment, or collect the values to create a new one with state `to_send`.
4. Create all collected delivery records.
5. Process the delivery records that need no remote call immediately.
6. Unless the calling context asks to skip it, schedule the sending scheduled action to run.

When the business response package is installed, posting a document additionally sends an approval response for every document that may still be answered (section 12.7).

## 12.5 Cancellation

1. Run the general ledger cancellation.
2. Delivery records not in `sent` are written to `cancelled`, with `error` and `blocking_level` cleared.
3. Delivery records in `sent` are written to `to_cancel`, with `error` and `blocking_level` cleared.
4. Delivery records that need no remote call are processed immediately.
5. The sending scheduled action is scheduled to run.

When the business response package is installed, cancelling a document that may still be answered opens the rejection wizard instead of returning immediately, and the result of the cancellation is carried in the wizard context so that it can be returned after the rejection is sent.

## 12.6 Reset to draft

1. For each document, if `electronic_document_show_cancel_button` is true, raise `You can't edit the following journal entry %s because an electronic document has already been sent. Please use the 'Request EDI Cancellation' button instead.` with the acronym expanded to "electronic document".
2. Run the general ledger reset to draft.
3. Clear `error` and `blocking_level` on every delivery record.
4. Delete the delivery records in state `to_send`.

Independently, the general ledger flag that shows the reset to draft button is switched off when any delivery record belongs to a format that needs a remote call, is in state `sent` or `to_cancel`, and whose applicability declares a `cancel` operation. It is also switched off for a sale document that is already sent on the network.

## 12.7 Operations added to Journal Entry

| Operation | Behaviour |
|---|---|
| `request_cancellation()` | For each document: check the fiscal lock dates; collect the delivery records whose format needs a remote call, that are in state `sent`, and whose applicability declares a `cancel` operation; post the note `A cancellation of the EDI has been requested.` with the acronym expanded when at least one record was collected. Write the collected records to `to_cancel` with `error` and `blocking_level` cleared. |
| `call_off_cancellation()` | For each document: collect the delivery records in state `to_cancel` whose applicability declares a `cancel` operation; post the note `A request for cancellation of the EDI has been called off.` with the acronym expanded when at least one record was collected. Write the collected records back to `sent` with `error` and `blocking_level` cleared. |
| `force_cancel()` | Post the note `This invoice was canceled while the EDIs %s still had a pending cancellation request.` with the acronym expanded to "electronic documents" and the placeholder filled with the names of the formats of the records still in `to_cancel`. Then run the ordinary cancellation. |
| `process_remote_services_now()` | Process the delivery records in state `to_send` or `to_cancel` and not at blocking level `error`, inside the current transaction, without committing between jobs. |
| `retry_failed_documents()` | Clear `error` and `blocking_level` on every delivery record, then process them with commits between jobs. |
| `delivery_record_of(format)` | Returns the delivery record of that format. |
| `payload_of(format)` | Returns the attachment of that delivery record, read with administrator rights. |
| `download_structured_file()` | Returns a download instruction for the structured file of the selected documents, allowing the fallback described in section 12.8. |
| `group_or_ungroup_lines_by_tax()` | Section 12.9. |
| `send_approval_response()` | Groups the documents that may still be answered by company and sends an approval response for each group. |
| `open_rejection_wizard()` | When at least one document may still be answered, opens the rejection wizard on those documents. |
| `open_responses()` | Opens the list of business responses of the selected documents. |
| `cancel_network_documents()` | When any selected document is already sent on the network, raise `Cannot cancel an entry that has already been sent to PEPPOL`. Otherwise clear `network_document_state` and clear the stored sending data. |
| `cancel_and_release_number()` | Cancel the document and set its number back to the placeholder `/`, releasing the sequence number. |
| `reset_network_documents(identifiers_to_delete)` | Draft documents are cancelled and their numbers released; documents that are neither draft nor cancelled and that carry no inalterable hash are reset to draft; the identifiers explicitly listed are deleted. Documents carrying an inalterable hash and already cancelled documents are left untouched. |

## 12.8 Legal document retrieval

When the caller asks for the structured representation of an accounting document:

1. If `electronic_invoice_markup_attachment` exists, return its name, the file type `xml` and its raw content.
2. Otherwise, when fallback is allowed, if the document has a partner, and the commercial partner resolves a structured format for the company of the document, and the document still needs a structured file, then build the file with that format now and return its computed file name, the file type `xml`, the content and the set of validation errors.
3. Otherwise defer to the general ledger behaviour.

The extra print entry `Export markup file` is offered when at least one selected posted document either already has a structured file or has a commercial partner that resolves a format for which the document still needs a file.

## 12.9 Grouping and ungrouping lines by tax

This operation lets a user of an imported document collapse the imported lines into one line per tax combination, and expand them again from the original file.

```
check that the document is in state "draft", otherwise raise
    "You can only (un)group lines of a draft invoice"
check (with the purchasing package installed) that no line is linked to a purchase order,
    otherwise raise "You can only (un)group lines of an invoice not linked to a purchase order"
if the lines look grouped:
    ungroup
else:
    group
```

**Detection of grouped lines:** a line looks grouped when its label matches the pattern made of the partner name (or `Unknown partner` when the partner has none), a space, a hyphen, a space, one or more digits, a space, a hyphen, a space, and any text.

**Grouping:**

1. Refuse when the document is not an invoice including receipts, with the message `You can only group lines of an invoice`.
2. Read the rounded base lines of the document.
3. Reduce them with a grouping key made of the standard base line grouping key of the tax engine, aggregating the quantities.
4. Fix the tax details of the reduced base lines when manual tax amounts are present.
5. Replace every line by one line per group, whose label is the partner name (or `Unknown partner`), the account code and the tax names joined by a space, a slash and a space (or `Untaxed` when no tax applies), joined by a space, a hyphen and a space; whose quantity and unit price come from the reduced base line; and whose extra tax data is exported from that base line.
6. Log `Grouped lines by tax` in the discussion thread.

**Ungrouping:**

1. Refuse with `Cannot find the origin file, try by importing it again` when the document has no structured file.
2. Rebuild the file data from the attachment, unwrap any nested files, and take the first group of file data.
3. Refuse with `Cannot decode origin file, try by importing it again` when no decoder is registered for that file type.
4. Clear all lines, run the decoder, and log `Ungrouped lines from <attachment name>` on success. A decoder that returns a reason instead of nothing raises `Cannot find the origin file, try by importing it again`.

**Automatic grouping after a purchase order link:** when a received document is linked to a purchase order and the document is eligible for grouping, and the last posted document of the same partner, company and document family has grouped lines, the received document is grouped as well. This keeps the presentation of a vendor consistent across bills.

## 12.10 File type recognition and decoding

The import file type of a structured markup file is decided in this order:

| Condition on the parsed file | Resulting type |
|---|---|
| The local name of the root element is `AttachedDocument` | the wrapper type, which is unwrapped rather than decoded |
| The root element is the cross industry invoice root in the cross industry invoice namespace | the cross industry invoice profile |
| The customization identifier contains `xrechnung` | the German electronic invoice profile |
| The customization identifier is the Netherlands standard invoice identifier | the Netherlands standard invoice profile |
| The customization identifier is the Australia and New Zealand conformant billing identifier | the Australia and New Zealand billing profile |
| The customization identifier is the Singapore conformant billing identifier | the Singapore billing profile |
| The customization identifier is the European compliant billing identifier | the Peppol billing 3.0 profile |
| The syntax version element says `2.0` | the universal business language 2.0 profile |
| The syntax version element says `2.1`, `2.2` or `2.3` | the universal business language 2.1 profile |
| The customization identifier contains the European semantic standard prefix | the Peppol billing 3.0 profile |
| none of the above | the general ledger recognition rules decide |

The exact identifier strings are listed in [universal-business-language-mapping.md](universal-business-language-mapping.md).

**Unwrapping a wrapper file:** the top most attachment node is read. When it holds an embedded binary object whose declared media type is a markup media type, that object is decoded from base sixty four and parsed. Otherwise, when it holds an external reference with a description and a markup media type, the description is used as the file content and parsed. When neither parses, the decoded binary object is returned as content with no parsed tree. The unwrapped file is given its own file type by the same recognition rules and the unwrapping repeats, so that a wrapper inside a wrapper is resolved.

**Decoder selection:** when the recognised type is one of the universal business language builders or one of the cross industry invoice builders (including every profile that inherits from them), the decoder is the import operation of that builder, registered at priority 20.

## 12.11 Helper operations used by import

| Operation | Behaviour |
|---|---|
| `needs_structured_file(format)` | True when the document has no structured file yet, and the document is a sale document or is exportable as a self billed invoice, and the format is one of the shipped structured formats. |
| `is_exportable_as_self_invoice()` | True when the document is posted, is a purchase document, its journal is a self billing journal, its commercial partner resolves a network format, that format has a builder, and that builder supports the self billing process. |
| `build_extra_lines(values)` | Turns a list of label, quantity, unit price and tax identifiers into line values with sequence 0, so that generated lines sort above the real lines. |
| `predict_specific_tax(label, amount_type, amount, usage)` | Returns the tax most often used on lines with a similar label for this partner, when the prediction feature is available. |

---

# 13. Fields added to Journal

| Field | Type | Required | Default | Stored or derived | Meaning |
|---|---|---|---|---|---|
| `electronic_document_formats` | many_to_many to Electronic Document Format, restricted to `compatible_electronic_document_formats` | no | derived | derived from `type`, `company` and the fiscal country of the company, stored, writable | The formats produced for documents posted in this journal. |
| `compatible_electronic_document_formats` | many_to_many to Electronic Document Format | no | derived | derived from `type`, `company` and the fiscal country of the company, not stored | The formats that accept this journal. |
| `participant_state` | selection, derived from `company.participant_state` | no | derived | derived | Convenience for the view. |
| `is_exchange_reception_journal` | boolean | no | false | stored | Marks the single purchase journal that receives documents from the exchange network. |

**Derivation of `compatible_electronic_document_formats`:** every registered format for which `is_compatible_with_journal(journal)` is true.

**Derivation of `electronic_document_formats`:**

1. Read, in one query, for every journal of the set, the distinct formats that have at least one delivery record in state `to_cancel` or `to_send` on a document of that journal. These are the protected formats.
2. For each journal, the enabled formats are the compatible formats that are either enabled by default on the journal or already selected on the journal.
3. The result is the enabled formats plus the protected formats that are already selected.

**Write guard:** when a write changes `electronic_document_formats`, the formats that were removed are computed after the write. Every delivery record of the journals of the set, belonging to a removed format, in state `to_cancel` or `to_send`, is collected. If any of them belongs to a format that needs a remote call, the write is refused with `Cannot deactivate (%s) on this journal because not all documents are synchronized` where the placeholder lists the display names of the affected formats. Otherwise the collected records are deleted.

**Validation:** changing the type of a journal marked as the exchange reception journal to anything other than a purchase journal raises `You can't change the type of a journal used for Peppol invoice reception to a type different than 'Purchase'.\nPlease change the journal used for Peppol reception before changing the type of this journal.`

**Derived view flags:** the flag that shows the "fetch incoming electronic invoices" button is true when the journal is the exchange reception journal, the company state is `receiver`, the journal is a purchase journal and it is not a self billing journal. The flag that shows the "refresh outgoing electronic invoice status" button is true when the company state allows sending and the journal is a sale journal, or is a purchase journal marked as self billing.

**Operations:** `fetch_incoming_electronic_invoices()` polls the inbox for every receiver credential of the companies of the selected journals; `refresh_outgoing_electronic_invoice_state()` polls the delivery states for every sendable credential of those companies.

---

# 14. Fields added to Tax

| Field | Type | Required | Default | Stored or derived | Meaning |
|---|---|---|---|---|---|
| `electronic_invoicing_tax_category_code` | selection of ten values, section 14.1 | no | empty | stored | The tax category code written in the structured file for this tax. When empty the code is predicted (see [calculations.md](calculations.md)). |
| `electronic_invoicing_tax_exemption_reason_code` | selection of the exemption reason code list | no | empty | stored | The exemption reason code written in the structured file. When empty the reason is predicted. |
| `electronic_invoicing_requires_exemption_reason` | boolean | no | derived | derived from `electronic_invoicing_tax_category_code`, not stored | True when the chosen category is one of `AE`, `E`, `G`, `O`, `K`. |

## 14.1 Tax category codes

| Code | Label, reproduced verbatim | Full word alias | Meaning |
|---|---|---|---|
| `AE` | "Vat Reverse Charge" | `reverse_charge` | The liability for the tax is shifted to the buyer. |
| `E` | "Exempt from Tax" | `exempt` | The supply is exempt. |
| `S` | "Standard rate" | `standard_rate` | Ordinary taxed supply. |
| `Z` | "Zero rated goods" | `zero_rated` | Taxed at a zero rate. |
| `G` | "Free export item, VAT not charged" (reproduced label; the abbreviation inside it expands to value-added tax) | `export_outside_union` | Export outside the union. |
| `O` | "Services outside scope of tax" | `outside_scope` | The supply is outside the scope of the tax. |
| `K` | "VAT exempt for EEA intra-community supply of goods and services" (reproduced label; the abbreviations inside it expand to value-added tax and European economic area) | `intra_community_supply` | Intra community supply inside the European economic area. |
| `L` | "Canary Islands general indirect tax" | `canary_islands` | The general indirect tax of the Canary Islands. |
| `M` | "Tax for production, services and importation in Ceuta and Melilla" | `ceuta_and_melilla` | The production, services and importation tax of Ceuta and Melilla. |
| `B` | "Transferred (VAT), In Italy" (reproduced label; the abbreviation inside it expands to value-added tax) | `transferred_italy` | Transferred tax in Italy. |

## 14.2 On change behaviour

When `electronic_invoicing_requires_exemption_reason` becomes false, `electronic_invoicing_tax_exemption_reason_code` is cleared. A user therefore cannot leave an exemption reason attached to a category that does not accept one.

## 14.3 Exemption reason code list

The selectable exemption reason codes and the full sentence each expands to are reproduced in [universal-business-language-mapping.md](universal-business-language-mapping.md), section "Tax exemption reason codes". The list has ninety one entries: sixty two European codes and twenty nine French codes.

---

# 15. Fields added to Contact

| Field | Type | Required | Default | Stored or derived | Tracked | Meaning |
|---|---|---|---|---|---|---|
| `invoice_electronic_format` | selection, extended with seven values | no | empty | stored | no | The structured format used for this contact. Added values: `facturx` (France (FacturX)), `ubl_bis3` (EU Standard (Peppol Bis 3.0)), `zugferd` (Germany (ZUGFeRD)), `xrechnung` (Germany (XRechnung)), `nlcius` (Netherlands (NLCIUS)), `ubl_a_nz` (Australia (BIS Billing 3.0 A-NZ)), `ubl_sg` (Singapore (BIS Billing 3.0 SG)). |
| `is_universal_business_language_format` | boolean | no | derived | derived from `invoice_electronic_format` and the active company, not stored | no | True when the chosen format is one of the shipped structured formats. |
| `peppol_endpoint` | text | no | derived | derived from `peppol_electronic_address_scheme`, stored, writable | yes | The participant endpoint value, also called the endpoint identifier. |
| `peppol_electronic_address_scheme` | selection of ninety values, listed in [configuration.md](configuration.md) | no | derived | derived from the country code, the tax identification number and the company registry, stored, writable | yes | Which kind of identifier `peppol_endpoint` holds. |
| `available_peppol_electronic_address_schemes` | structured_data | no | derived | derived from the company and the current scheme, not stored | no | The schemes offered in the interface: every scheme except the retired ones, plus the currently selected scheme even when retired. |
| `available_network_sending_methods` | structured_data | no | derived | derived from the company, not stored | no | The sending methods offered: every method, minus the network method when the country of the active company is not an eligible country. |
| `available_network_formats` | structured_data | no | derived | derived from `invoice_sending_method`, not stored | no | When the sending method is the network method, the formats that the network accepts; otherwise every structured format. |
| `participant_verification_state` | selection: `not_verified` (Unchecked), `not_valid` (Partner is not on Peppol), `not_valid_format` (Partner cannot receive format), `valid` (Partner is on Peppol); one value per company | no | `not_verified` | stored per company | no | The result of the last participant lookup for this contact. |
| `participant_supported_documents` | structured_data | no | empty | stored | no | The document type identifiers the participant publishes. |
| `participant_supports_responses` | boolean | no | derived | derived from `participant_supported_documents` and `participant_verification_state`, not stored | no | True when the verification state is `valid` and the response transaction identifier is among the published document types. |

## 15.1 Derivation of `peppol_electronic_address_scheme`

```
keep the current value
country = the deduced country code of the contact
if country is in the scheme map:
    schemes = the schemes of that country, in declaration order
    if the current value is not one of them:
        candidates = the schemes of that country that are not retired,
                     or all of them when every scheme of that country is retired
        new_value = the first candidate
        for each (scheme, source_field) in candidates:
            if source_field exists on the contact:
                value = endpoint_value(country, source_field, scheme)
                if value is set and passes the endpoint validity rules of that scheme:
                    new_value = scheme
                    stop
        peppol_electronic_address_scheme = new_value
```

## 15.2 Derivation of `peppol_endpoint`

```
peppol_endpoint = sanitise(peppol_endpoint, peppol_electronic_address_scheme)
country = the deduced country code of the contact
if country is in the scheme map:
    source_field = the field mapped to the current scheme for that country
    value = endpoint_value(country, source_field, current scheme)
    if source_field is set and value is set and value passes the endpoint validity rules:
        peppol_endpoint = value
```

`endpoint_value(country, field, scheme)` returns nothing when the field is the endpoint field itself, which exists as a hook for country specific logic; otherwise it reads that field from the contact; for a Belgian contact whose company registry is empty it falls back to the tax identification number with the country prefix removed when the value is alphanumeric; and it finally sanitises the value for the scheme.

**Sanitising** removes every character that is not allowed for the scheme: for the Belgian company registry scheme `0208` everything that is not a digit; for the Belgian tax identification scheme `9925` everything that is not a digit or one of the letters of the country prefix in either case; for the electronic mail scheme `EM` everything that is not a letter, a digit, a hyphen, a full stop, an underscore or an at sign; for every other scheme everything that is not a letter, a digit, a hyphen, a full stop, an underscore or a tilde.

**Endpoint validity rules** (the first failing rule wins):

| Scheme | Condition | Message |
|---|---|---|
| `0208` | The endpoint is not exactly ten digits. | `The Peppol endpoint is not valid. The expected format is: 0239843188` |
| `0009` | The endpoint is not a valid French establishment identification number. | `The Peppol endpoint is not valid. The expected format is: 73282932000074` |
| `0007` | The endpoint is not exactly ten digits. | `The Peppol endpoint is not valid. It should contain exactly 10 digits (Company Registry number).The expected format is: 1234567890` |
| `EM` | The endpoint is not a single valid electronic mail address. | `The Peppol endpoint is not valid. A valid email is required` |
| any | The endpoint still contains a character the scheme forbids, or its length is not between one and fifty. | `The Peppol endpoint (%s) is not valid. It should contain only letters and digit.` |

These rules are enforced as a validation whenever `peppol_endpoint` changes and both the scheme and the endpoint are set.

## 15.3 Format resolution

| Operation | Behaviour |
|---|---|
| `structured_formats()` | The list of shipped structured format keys. |
| `structured_format_information()` | For each key: the eligible countries, whether the network accepts it, an ordering sequence and whether it embeds attachments. The table is in [configuration.md](configuration.md). |
| `structured_formats_by_country()` | Inverts the previous table into a country to format list map. |
| `suggested_structured_format()` | Take the deduced country code of the commercial partner. When the country has exactly one format, return it. When the country has several, return the German electronic invoice profile if the scheme is the German routing identifier scheme `0204`, otherwise the format with the smallest sequence, defaulting to one hundred when a format declares none. When the country has no format, return nothing. |
| `structured_format()` | `invoice_electronic_format` when set, otherwise the suggested format. |
| `network_formats()` | The shipped formats whose information says the network accepts them. |
| `suggested_network_format()` | The suggested structured format of the commercial partner when the network accepts it, otherwise the Peppol billing 3.0 profile. |
| `network_format()` | `invoice_electronic_format` when set, otherwise the suggested network format. |
| `builder_for(format)` | The builder of a format key: the German electronic invoice profile for `xrechnung`; the cross industry invoice profile for both `facturx` and `zugferd`; the Australia and New Zealand profile for `ubl_a_nz`; the Netherlands profile for `nlcius`; the Peppol billing 3.0 profile for `ubl_bis3`; the Singapore profile for `ubl_sg`. Any other key resolves to nothing. |

## 15.4 Creation and recomputation guards

- On creation of contacts, the participant verification state is recomputed per company for the new contacts.
- The endpoint and the scheme are **not** recomputed for contacts that are the partner of a company whose participant state allows sending. A registered company must keep exactly the identification it registered with.
- The verification recomputation runs for contacts that have a scheme, an endpoint, a structured format and a country in the eligible country list. When a write touched the scheme, the endpoint or the format, it runs for every contact of an eligible country. A contact that belongs to a specific company is checked for that company only; a shared contact is checked once per company whose participant state allows sending.

## 15.5 On change behaviour

Editing `invoice_electronic_format`, `peppol_endpoint` or `peppol_electronic_address_scheme` triggers the participant verification immediately, so that the interface can warn before the document is sent. When the commercial partner link is not yet computed it is computed first.

---

# 16. Fields added to Company

| Field | Type | Required | Default | Stored or derived | Tracked | Meaning |
|---|---|---|---|---|---|---|
| `electronic_interchange_proxy_users` | one_to_many to Electronic Interchange Proxy User, only active records | no | empty | stored inverse | no | The credentials of this company. |
| `participant_contact_email` | text | no | derived | derived from `email`, stored, writable | no | Primary contact email for connection related communication. It is the address used to reconnect the account after a database change. |
| `participant_phone_number` | text | no | derived | derived from `phone`, stored, writable | no | Mobile number used for identification only. |
| `participant_migration_key` | text | no | empty | stored, system administrators only | no | The key that lets the network move an existing participant from another access point to this one. Cleared once handed over. |
| `participant_state` | selection: `not_registered` (Not registered), `sender` (Can send but not receive), `smp_registration` (Can send, pending registration to receive), `receiver` (Can send and receive), `rejected` (Rejected) | yes | `not_registered` | stored | no | The registration state. |
| `participant_proxy_user` | many_to_one to Electronic Interchange Proxy User | no | derived | derived from `electronic_interchange_proxy_users`, not stored | no | The credential whose service is one of the exchange network services. |
| `peppol_electronic_address_scheme` | selection, derived from `partner.peppol_electronic_address_scheme`, writable | no | derived | derived | no | The scheme of the company itself. |
| `peppol_endpoint` | text, derived from `partner.peppol_endpoint`, writable | no | derived | derived | no | The endpoint of the company itself. |
| `reception_journal` | many_to_one to Journal, restricted to purchase journals | no | derived | derived from `participant_state`, stored, writable | no | The purchase journal in which received documents are created. |
| `external_access_point_provider` | text | no | empty | stored | yes | The name of the access point that already serves this participant, when the participant is published elsewhere. |
| `participant_can_send` | boolean | no | derived | derived from `participant_state`, not stored | no | True when the state is one of `sender`, `smp_registration`, `receiver`. |
| `participant_parent_company` | many_to_one to Company | no | derived | derived from `peppol_electronic_address_scheme` and `peppol_endpoint`, not stored | no | The closest ancestor whose participant identification this company shares, or whose identification it may borrow because it has none of its own. |
| `participant_metadata` | structured_data | no | empty | stored | no | Additive metadata supplied by the proxy. |
| `participant_metadata_updated_on` | datetime | no | empty | stored | no | When that metadata was last refreshed. |

## 16.1 Derivations

- `participant_contact_email` takes the value of `email` when it is empty.
- `participant_phone_number` takes the value of `phone` when it is empty **and** that value passes the phone number check; a failing value is silently skipped.
- `participant_proxy_user` is the credential whose service is one of the exchange network services.
- `participant_can_send` is true when the state is one of `sender`, `smp_registration`, `receiver`.
- `participant_parent_company` walks the ancestors from the closest outward and takes the first one for which either this company has a scheme and an endpoint equal to the ancestor scheme and endpoint, or this company has no endpoint while the ancestor has both.
- `reception_journal` is filled, when it is empty and the company can send, with the first purchase journal of the company, and that journal is marked as the exchange reception journal. Writing the field clears the mark on every other purchase journal of the company and sets it on the new one, which keeps exactly one marked journal.

## 16.2 Validations

| Rule | Condition | Message |
|---|---|---|
| Phone number format | `participant_phone_number` is set and cannot be parsed as a valid international number. | `Please enter the mobile number in the correct international format.\nFor example: +32123456789, where +32 is the country code.` |
| Phone number library | The phone number library is unavailable. | `Please install the phonenumbers library.` |
| Endpoint check | `peppol_endpoint` is set and fails the blocking endpoint rule of its scheme. | `The Peppol endpoint identification number is not correct.` |
| Reception journal type | `reception_journal` is set and is not a purchase journal. | `A purchase journal must be used to receive Peppol documents.` |

The blocking endpoint rules per scheme are: `0007` must be a valid Swedish organisation number; `0088` must be a valid international article number; `0184` must be a valid Danish central business register number; `0192` must be a valid Norwegian organisation number; `0208` must be a valid Belgian tax identification number. The advisory rules, which only produce a warning in the registration wizard, are: `0151` must be a valid Australian business number; `0201` must be exactly six alphanumeric characters; `0210` must be a valid Italian fiscal code; `0211` and `9906` must be a valid Italian tax identification number; `9907` must be a valid Italian fiscal code.

## 16.3 Endpoint sanitising on write

Before a company is created or written, when both a scheme and an endpoint are present in the values, a per scheme extractor rewrites the endpoint: scheme `0007` keeps the first run of ten digits; `0184` keeps the first run of eight digits; `0192` keeps the first run of nine digits; `0208` keeps the first run of ten digits. When the extractor finds nothing the typed value is kept.

On creation, the default value of the per company participant verification state of contacts is set to `not_verified` for the new company.

## 16.4 Operations

| Operation | Behaviour |
|---|---|
| `active_participant_ancestor()` | The closest ancestor, walking outward, whose `participant_can_send` is true. |
| `reset_participant_configuration(soft)` | Sets `participant_state` to `not_registered` and clears `participant_migration_key`. When not soft, also clears `external_access_point_provider`, the scheme, the endpoint, the contact email and the phone number, and recomputes the contact email and the phone number. In both cases the scheme and the endpoint of the company partner are recomputed, so that a branch which was borrowing the parent identification gets its own default back. |
| `supported_document_types()` | Flattens the per package document type map into one identifier to name map. The base map and the map added by the business response package are in [configuration.md](configuration.md). |
| `operating_mode(temporary_scheme)` | Returns `demo` when the scheme in force (the temporary one when given, otherwise the stored one) is the demonstration scheme `odemo`; otherwise the mode of the existing credential; otherwise the value of the configuration parameter that names the environment; otherwise `prod`. |
| `webhook_address()` | The base web address of the deployment followed by `/peppol/webhook`. |
| `information_on_network(identification)` | Looks the participant up. Returns whether the participant exists, the name of the serving access point when it can be read, and, when the participant exists, the message `A participant with these details has already been registered on the network. If you have previously registered to a Peppol service, please deregister.` extended with `The Peppol service that is used is %s.` when the serving access point name is known and is not this platform. The access point name is read by fetching the first published service address of the participant and reading its service description; failures are swallowed. |
| `send_welcome_message()` | When the state is `sender` or `receiver`, send the registration message template to the participant contact email. |
| `proxy_type()` | The service of the existing exchange network credential, defaulting to the base exchange network service. |
| `is_french_company()` | True when the fiscal country is France, Guadeloupe, Martinique or Réunion, or when the scheme is one of the French schemes. |
| `allows_document_reception()` | True. An extension point for localisations that forbid reception. |

---

# 17. Fields added to Configuration Settings

Every field here is a view over the company fields of section 16, so that the settings screen can present them. They are transient.

| Field | Derivation |
|---|---|
| `participant_proxy_user` | `company.participant_proxy_user` |
| `participant_operating_mode` | `participant_proxy_user.operating_mode` |
| `participant_contact_email` | read from and written to `company.participant_contact_email`, with a proxy synchronisation on write (section 17.1) |
| `peppol_electronic_address_scheme` | `company.peppol_electronic_address_scheme`, writable |
| `participant_identification` | `participant_proxy_user.participant_identification` |
| `peppol_endpoint` | `company.peppol_endpoint`, writable |
| `participant_migration_key` | `company.participant_migration_key`, writable |
| `participant_phone_number` | `company.participant_phone_number`, writable |
| `participant_state` | `company.participant_state`, writable |
| `reception_journal` | `company.reception_journal`, writable |
| `external_access_point_provider` | `company.external_access_point_provider`, writable |
| `reception_journal_required` | true when `participation_role` is `sending_and_receiving` |
| `use_parent_participant` | true when the company differs from its participant parent company, the company can send and the parent can send |
| `parent_participant_name` | the name of that parent when the previous flag is true, otherwise empty |
| `is_token_out_of_step` | `participant_proxy_user.is_token_out_of_step`, writable |
| `participation_role` | selection `sending_and_receiving` (Sending & Receiving), `sending_only` (Sending Only); derived from `participant_state`: `sending_only` when the state is `sender`, `sending_and_receiving` when the state is `smp_registration` or `receiver`, empty otherwise |

## 17.1 Writing the contact email

When the typed value equals the company value nothing happens. Otherwise the company value is updated. When no credential exists yet the new value is kept without contacting the proxy. Otherwise the update user endpoint is called with the new contact email.

## 17.2 Writing the participation role

- Choosing `sending_only` while the state is not `sender` unpublishes the participant from the service directory and sets the state back to `sender`.
- Choosing `sending_and_receiving` while the state is neither `smp_registration` nor `receiver` registers the sender as a receiver and then polls the participant state.

---

# 18. Fields and behaviour added to other entities

## 18.1 Attachment

A deletion guard forbids deleting an attachment that is the payload of an electronic document whose format needs a remote call, with the message `You can't unlink an attachment being an electronic document sent to the government.`. The guard reads the delivery records with administrator rights, because it must apply whatever the rights of the caller.

## 18.2 Report Action

Two extensions of the printed document rendering:

1. When exactly one accounting document is rendered with the invoice report, and that document is a sale document that is not draft, every delivery record of the document is asked to embed its payload inside the rendered portable document. The rendered stream is read, cloned into a writer, each format embeds what it wants, and the stream is replaced.
2. When exactly one accounting document is rendered with a report whose name appears in the comma separated list of the configuration parameter that names custom templates, and the document is a posted sale document, the cross industry invoice file is generated and embedded into the rendered portable document by the same post processing step that the sending service uses.

## 18.3 Document Sending Service

The sending service of the accounts receivable domain is extended at seven points. The full procedure is in [workflows.md](workflows.md).

| Extension point | Behaviour added |
|---|---|
| document constraints | The constraint that refuses a non sale document is lifted for a document that is exportable as a self billed invoice. |
| alerts | Section 18.4. |
| placeholder attachments | When the document still needs a structured file for the chosen format, a placeholder entry is added with the computed file name and the markup media type. |
| extra attachments | The structured file of the document, and the payload of every delivery record that is linked to a business record, are added to the attachments of the outgoing message. |
| before the printed document is rendered | The structured file is generated, and either an error is recorded or the attachment values and the format options are stored for the next steps. |
| after the printed document is rendered | The printed document and the user attachments are embedded into the structured file; a cross industry invoice file is always generated and embedded into the printed document; the printed document is converted to the archival profile when the country rules ask for it. |
| linking documents | The collected structured file attachment values are created with administrator rights and the cached values of the accounting documents are invalidated. |

## 18.4 Alerts added to the sending wizard

| Key | Level | Condition | Message and action |
|---|---|---|---|
| `configure_company` | information | At least one document uses a network format and the company partner has no scheme or no endpoint. | `Please fill in your company's VAT or Peppol Address to generate a complete XML file.` with the acronyms expanded to "value-added tax number" and "markup file"; action text `Configure`, opening the company partners concerned. |
| `configure_partner` | information | At least one document uses a network format and the commercial partner has no scheme or no endpoint. | `Please fill in partner's VAT or Peppol Address.` with the acronym expanded; action text `View Partner(s)`, opening the partners under the title `Check Partner(s)`. |
| `chorus_pro_install` | information | At least one commercial partner is the French public sector portal participant and the corresponding package is not installed. | `Please install the french Chorus pro module to have all the specific rules.` with action text `Install Chorus Pro`. |
| `warning_partner` | default | At least one document is sent over the network and its commercial partner is in verification state `not_valid_format`, and the partner configuration alert is not already present. | `Customer is on Peppol but did not enable receiving documents.` with action text `View Partner(s)`. |
| `what_is_the_network` | information | Every selected document belongs to a company in one of the countries where the information is always shown, at least one document is not yet sent, and no partner is in verification state `not_valid` or `not_verified`. Replaces the company configuration alert. | `You can send this invoice electronically via Peppol.`, or for a French company `To use the Approved Platform for French E-Invoicing, install the module`; action opens the explanation dialogue or the package installation. |
| `partner_wants_the_network` | default | Exactly one partner of the documents not sent over the network is in verification state `valid`, at least one document is not yet sent, and at least one partner country is an eligible default country. | `%s has requested electronic invoices reception on Peppol.` with the partner display name. |
| `approved_platform` | default | The warning is not switched off by the configuration parameter, and at least one company is a French company whose service is not the approved platform service. | `To use the Approved Platform for French E-Invoicing, install the module` |

The countries where the information is always shown are Belgium, Finland, Luxembourg, Latvia, the Netherlands, Norway, Sweden and France.

## 18.5 Sending wizard checkbox behaviour

The label of the network sending checkbox is extended with:

- ` (Customer not on Peppol)` when the partner verification state is `not_valid`, or is `not_verified` and the scheme and the endpoint could not be recomputed;
- ` (no VAT)` with the acronym expanded to "no value-added tax number", when the state is `not_verified`, the scheme or the endpoint is missing and the partner has no tax identification number;
- ` (Missing <scheme label>)` when the state is `not_verified`, the scheme or the endpoint is missing and the partner has a tax identification number;
- ` (Test)` when the operating mode is the test environment;
- ` (Demo)` when the operating mode is the demonstration environment.

When any of the first three suffixes applies, the checkbox is made read only and unchecked.

The wizard also computes, per attachment, the message `Unsupported file type via <format key>` for every attachment the chosen format cannot embed.

Sending one document over the network refuses with `Partner doesn't have a valid Peppol configuration.` when the commercial partner verification state is not `valid`.

## 18.6 Product Variant

Two extra matching strategies are added to the import product search, used only when the ordinary identifier strategies fail:

- Match on `default_code` against the extended seller item identifier, at ordering rank 12.
- Match on `barcode` against the extended standard item identifier, at ordering rank 14.

The ordinary strategies use the plain item identifiers, which carry the template level values, while the extended ones carry the variant level values.

## 18.7 Sales Order and Purchase Order

Each of the two adds: registration of its export builder, recognition of a structured file whose customization identifier is the ordering transaction identifier, registration of the matching decoder at priority 20, an activity creation used when part of an incoming order could not be interpreted, and a helper that turns a list of label, quantity, unit price and tax identifiers into order line values with sequence 0.

The activity is of the "to do" kind, is assigned to the current user, and its note is `Some information could not be imported:` followed by an unordered list of the individual messages.

## 18.8 Resequencing Wizard

Before resequencing, the delivery records of the selected documents whose format needs a remote call and whose state is `sent` are collected. When the collection is non empty the operation is refused with `The following documents have already been sent and cannot be resequenced: %s` where the placeholder lists the distinct names of the affected accounting documents.
