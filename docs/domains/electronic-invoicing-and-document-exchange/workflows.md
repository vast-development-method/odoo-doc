# Workflows

Every operational procedure of the Electronic Invoicing and Document Exchange domain, with actors, preconditions, numbered steps, branches, the records written at each step, the notifications produced and the postconditions. Section 20 gives a condensed transition table for the five state fields that these procedures write most often; the complete specification of all eleven state machines of the domain, with every state, every guard in evaluation order and every refusal message, is in [state-machines.md](state-machines.md).

---

# 1. Register a format on a journal

**Actors:** accountant, system administrator.
**Preconditions:** at least one Electronic Document Format record exists.

1. The user opens a journal and reads the field `electronic_document_formats`. The selectable values are limited to `compatible_electronic_document_formats`, that is to the formats whose `is_compatible_with_journal` answered true for this journal. The default compatibility rule accepts only sale journals.
2. The user adds or removes formats and saves.
3. **Guard.** Before the write is accepted, the system determines the removed formats, then searches every Electronic Document whose accounting document belongs to one of the saved journals, whose format is one of the removed ones, and whose state is `to_send` or `to_cancel`.
   - **Branch A:** at least one of those records belongs to a format that needs a remote call. The write is refused with `Cannot deactivate (%s) on this journal because not all documents are synchronized`, the placeholder listing the display names of the affected formats. Nothing is written.
   - **Branch B:** none of them needs a remote call. The write is accepted and those records are deleted, because the framework no longer has to produce anything for them.
4. **Postcondition:** every accounting document posted in this journal from now on will produce one Electronic Document per remaining format for which the format declares applicability.

**Automatic recomputation.** Whenever the journal type, the company or the fiscal country of the company changes, `compatible_electronic_document_formats` and `electronic_document_formats` are recomputed. The recomputation never silently removes a format that still has pending work: the formats with a delivery record in `to_send` or `to_cancel` are added back as protected formats.

---

# 2. Post an accounting document with electronic documents

**Actors:** accountant.
**Preconditions:** the accounting document is in draft or is being posted; its journal declares at least one format.

1. The general ledger posting runs to completion. If it fails, nothing in this domain happens.
2. For each posted document, for each format declared on its journal:
   1. Ask the format for its applicability on this document. An empty answer means the format does not apply; move to the next format.
   2. Ask the format for its configuration errors.
      - **Branch A: errors exist.** Raise `Invalid invoice configuration:\n\n%s` where the placeholder is the error messages joined by a line break. The whole posting, including the general ledger part, is rolled back. The user is expected to fix the configuration and post again.
      - **Branch B: no error.** Continue.
   3. If a delivery record already exists for this document and this format, write it with `state = "to_send"` and `attachment` cleared. This is the case when the document was cancelled and posted again.
   4. Otherwise collect the values of a new delivery record with `state = "to_send"`.
3. Create all collected delivery records in one operation.
4. Process immediately every delivery record of the posted documents whose format does **not** need a remote call. See section 3.
5. Unless the calling context asks to skip it, schedule the sending scheduled action to run as soon as possible, so that the records that need a remote call are picked up without waiting a full day.

**Records written:** one Electronic Document per document per applicable format, in state `to_send`; possibly, in step 4, the same records in state `sent` with their attachment.
**Postcondition:** `electronic_document_state` of the accounting document is `to_send` when at least one format needs a remote call and is still pending, `sent` when every such format already succeeded, and empty when no format needs a remote call.

When the business response package is installed, posting also triggers the approval response flow of section 14.2.

---

# 3. Process documents that need no remote call

**Actors:** the posting flow, the cancellation flow.

1. Keep the delivery records whose format answers false to `needs_remote_call`.
2. Prepare jobs from them, using the batching rules of [entities.md](entities.md) section 1.8.
3. Run every job inside the current transaction, with no locking and no commit.

Because the work runs inside the transaction of the posting, a failure inside a synchronous format rolls the posting back. That is the intended behaviour: a format that produces a file locally must never leave a posted document without its file.

---

# 4. The sending scheduled action

**Actors:** scheduled job runner.
**Preconditions:** the scheduled action is active; it becomes active automatically the first time a format that needs a remote call is created.

1. Search every Electronic Document where `state` is `to_send` or `to_cancel`, the accounting document is posted, and `blocking_level` is not `error`.
2. Keep only the records whose format needs a remote call.
3. Prepare jobs from them. A format that declares a batching key groups several accounting documents of the same company and state into one job; a format that does not declares one job per accounting document.
4. Take the first `job_count` jobs, where `job_count` is the parameter of the scheduled action, twenty by default.
5. For each taken job:
   1. Collect the accounting documents of the job and the payload attachments of the job that are not linked to any business record.
   2. Take a row lock on the delivery records, on the accounting documents and on those attachments.
      - **Branch A:** the lock cannot be taken. The job is skipped and left for the next run, and a debug trace is recorded. When the caller asked for no commit, for example the manual "Process now" operation, the user error `This document is being sent by another process already. ` is raised instead.
      - **Branch B:** the lock is taken. Continue.
   3. Execute the job: for state `to_send`, run the `post` operation of the format inside the "send only when ready" guard, then apply the send post processing; for state `to_cancel`, run the `cancel` operation and apply the cancellation post processing.
   4. When commits are allowed and more than one job is being processed, commit. Each job therefore survives a later failure.
6. Compute the number of jobs left, that is the total number of prepared jobs minus the number taken.
7. If that number is greater than zero, schedule the same scheduled action to run again as soon as possible.

**Records written:** the delivery records of every processed job, and, in the cancellation flow, the accounting documents that become fully cancelled.
**Postcondition:** every document either reached `sent` or `cancelled`, or carries an error and a blocking level.

---

# 5. Retry a failed electronic document

**Actors:** accountant.
**Preconditions:** the accounting document shows an electronic invoicing error.

1. The user presses "Retry" on the accounting document, or "⇒ See errors" to open the list of failing delivery records first.
2. `error` and `blocking_level` are cleared on **every** delivery record of the document, not only the failing ones.
3. The delivery records in state `to_send` or `to_cancel` and not at blocking level `error`, which is now all of them, are processed with commits allowed.

**Postcondition:** either the records progressed to `sent` or `cancelled`, or they carry a fresh error.

A record whose blocking level was `warning` is *not* stuck: the scheduled action keeps picking it up on every run, because only `error` excludes a record from job preparation. A record at `info` is likewise picked up again, which is how a format implements a multiple step remote protocol: the first step returns an error text with level `info`, the applicability of the next run reads that text and returns the operation of the second step.

---

# 6. Request, call off and force a cancellation

**Actors:** accountant.
**Preconditions:** the accounting document is posted and at least one delivery record is in state `sent` for a format that needs a remote call and declares a `cancel` operation.

## 6.1 Request cancellation

1. The user presses "Request EDI Cancellation", shown because `electronic_document_show_cancel_button` is true; the label reads "Request Cancellation" with the acronym expanded.
2. For each selected document the fiscal lock dates are checked. A locked period refuses the operation with the general ledger message.
3. The qualifying delivery records are collected and the note `A cancellation of the electronic document has been requested.` is posted in the discussion thread of the document.
4. The collected records are written to `to_cancel` with `error` and `blocking_level` cleared.
5. The next run of the sending scheduled action executes the `cancel` operation.

## 6.2 Call off the cancellation

1. The user presses "Call off EDI Cancellation", shown because `electronic_document_show_call_off_button` is true.
2. The delivery records in `to_cancel` whose applicability declares a `cancel` operation are collected and the note `A request for cancellation of the EDI has been called off.` is posted.
3. The collected records are written back to `sent` with `error` and `blocking_level` cleared.

## 6.3 Force cancellation

1. The user presses "Force Cancel", shown because the general ledger allows force cancelling.
2. The note `This invoice was canceled while the EDIs %s still had a pending cancellation request.` is posted, with the acronym expanded and the placeholder listing the format names of the records still in `to_cancel`.
3. The ordinary cancellation runs, which writes every record not in `sent` to `cancelled` and every record in `sent` to `to_cancel`.

## 6.4 Automatic cancellation of the accounting document

When the remote service confirms the cancellation of a record, and the accounting document is posted, and every delivery record of the document is either `cancelled` or belongs to a format that needs no remote call, the accounting document itself is reset to draft and then cancelled. This is the only automatic state change of an accounting document in this domain.

---

# 7. Generate the structured file of an accounting document

**Actors:** billing clerk, through the sending service; accountant, through the download operation; the framework, through a format that produces a file.
**Preconditions:** the accounting document is posted, has a partner, and a format has been resolved for that partner.

1. **Resolve the format.** The chosen sending method decides: with the network method the format is `network_format()` of the partner; otherwise it is `structured_format()` of the partner. Both fall back to the suggested format of the country of the commercial partner.
2. **Resolve the builder** with `builder_for(format)`.
3. **Validate the tax structure.** Every tax used on the lines is asked to validate its repartition structure. A failing tax raises `Tax '%(tax_name)s' is invalid: %(error_message)s`.
4. **Collect the export values.** The builder assembles the document level values: document type, company, currency, supplier, customer, delivery address, and the rounded base lines and tax lines of the accounting document. The universal business language family additionally rounds the raw totals, the raw gross totals and the raw discounts to six digits, turns negative unit prices into negative quantities, and turns emptying taxes into extra lines. The cross industry invoice family additionally extracts the cash rounding lines and the early payment discount lines. The details are in [calculations.md](calculations.md).
5. **Build the document tree** by calling the node builders in the order declared by the profile. The complete element by element mapping is in [universal-business-language-mapping.md](universal-business-language-mapping.md) and [cross-industry-invoice-mapping.md](cross-industry-invoice-mapping.md).
6. **Run the constraints.** Each profile contributes its own checks; every non empty message is collected into a set.
7. **Serialise** the tree against the ordered element template of the syntax, with the namespace declarations of the profile, and prefix it with the declaration of the markup version and the character encoding, which is always the eight bit unicode transformation format.
8. **Return** the serialised bytes and the set of errors.

**Branch: errors were collected.** The sending service records `error_title` set to `Errors occurred while creating the EDI document (format: %s):` with the acronym expanded and the placeholder filled with the description of the builder, and `errors` set to the collected set, and marks the error as one that does not stop the rest of the sending. The file is *not* attached.

**Branch: no error.** The attachment values are stored for the later steps: the computed file name, the raw bytes, the markup media type, the accounting document as the owning record and the field `electronic_invoice_markup_file` as the owning field.

---

# 8. Send an invoice through the sending service

**Actors:** billing clerk.
**Preconditions:** the accounting document is posted; the sending wizard is open.

1. **Compute the sending methods.** The network method is added to the default methods when it is applicable to the document and at least one country of the commercial partner is in the list of countries where the network is the default. Section 8.6 gives the applicability rules.
2. **Refresh the partner verification.** For each partner and company pair of the selected documents, when the verification state is not `valid` and the network method is applicable to the company, the participant lookup runs again.
3. **Show the alerts** listed in [entities.md](entities.md) section 18.4.
4. **Pre send.** When the network method is selected:
   1. If the documents all belong to one company and that company cannot send, open the registration wizard instead of sending. The user registers, and the wizard returns to the sending action afterwards.
   2. Otherwise, for every document whose `network_document_state` is `ready` or empty, set `network_document_state` to `to_send`.
   3. For a single document, refuse with `Partner doesn't have a valid Peppol configuration.` when the commercial partner verification state is not `valid`.
5. **Before the printed document is rendered:** generate the structured file as in section 7.
6. **Render the printed document** through the accounts receivable report.
7. **After the printed document is rendered:**
   1. When the chosen format is not the cross industry invoice family, embed the printed document and the accepted user attachments into the structured file (section 8.2).
   2. Always generate a cross industry invoice file: the already generated one when the format is the cross industry invoice family, a freshly generated one otherwise. Attach it to the printed document under the name `factur-x` with the extension `xml`, declared as a markup media type with the relationship "alternative".
   3. When the format is the cross industry invoice family, or when the commercial partner country is France or Germany and the partner scheme is not the German routing identifier scheme `0204`, and the country of the accounting document is France or Germany, and the printed document is not yet in the archival profile, convert it to the archival profile and add the archival metadata built from the document name and the current date. A conversion failure is logged and does not stop the sending.
8. **Call the remote services** (section 9 for the network method).
9. **Link the documents:** create the structured file attachments with administrator rights and invalidate the cached values of the accounting documents.
10. **Send the message** with the printed document, the structured file and the extra attachments.

## 8.1 Which attachments may be embedded

A format declares whether it embeds attachments. Only the Peppol billing 3.0 profile does. Among the attachments selected in the wizard, only those whose media type is one of the supported types are embedded; the others are reported to the user. The supported media types are the portable document, the open document spreadsheet, the office open spreadsheet, the photographic image, the portable network image and the comma separated values text.

## 8.2 Embedding into the structured file

1. Parse the generated structured file.
2. Find the insertion anchor. For an invoice root: the first element whose local name is the project reference, the signature or the accounting supplier party. For a credit note root: the statement document reference, the originator document reference, the signature or the accounting supplier party. For a debit note root: the signature or the accounting supplier party. When no anchor is found the embedding is skipped rather than producing an invalid file.
3. Build the list of documents to embed: first the accepted user attachments, then the printed document.
4. For each, insert before the anchor an additional document reference node carrying the file name as its identifier, the document type code node supplied by the profile when it has one, and an attachment node holding the base sixty four encoded content with its media type and its file name.
5. Re-serialise the tree.

The printed document entry is the one that carries the document type code node, because it is the human readable rendition of the invoice; user attachments carry no type code.

## 8.3 Placeholder attachment

While the wizard is open and before anything is generated, a placeholder entry is shown with the file name the builder would compute and the markup media type, so that the user sees what will be produced.

## 8.4 Extra attachments of the outgoing message

The structured file of the document, and the payload of every delivery record that is linked to a business record, are added to the attachments of the message.

## 8.5 File names produced by each profile

| Profile | File name |
|---|---|
| Universal Business Language 2.0 | the document name with every slash replaced by an underscore, then `_ubl_20` with the extension `xml` |
| Universal Business Language 2.1 | the document name with slashes replaced, then `_ubl_21` with the extension `xml` |
| Peppol Billing 3.0 | the document name with slashes replaced, then `_ubl_bis3` with the extension `xml` |
| German Electronic Invoice | the document name with slashes replaced, then `_xrechnung` with the extension `xml` |
| Netherlands Standard Invoice | the document name with slashes replaced, then `_nlcius` with the extension `xml` |
| Australia and New Zealand Billing | the document name with slashes replaced, then `_a_nz` with the extension `xml` |
| Singapore Billing | the document name with slashes replaced, then `_sg` with the extension `xml` |
| Belgian Electronic Invoicing | `efff_`, the tax identification number of the commercial partner of the company (empty when absent), an underscore when that number is present, and the document name with every character that is not a letter, a digit or an underscore removed, then the extension `xml` |
| Cross Industry Invoice | the document name with slashes replaced, then `_zugferd` with the extension `xml` when the commercial partner country is Germany, otherwise `_factur_x` with the extension `xml` |

## 8.6 Applicability of the network sending method

**To a company:** the country of the company is in the eligible country list and the participant state is none of `not_registered`, `in_verification` and `rejected`.

**To an accounting document:** all of the following hold.

1. The country of the commercial partner is in the eligible country list.
2. The method is applicable to the company of the document.
3. The commercial partner verification state is `valid`.
4. The participant state of the company is not `rejected`.
5. The document still needs a structured file for the chosen format, or it already has one and is not yet sent on the network.

Before those checks, two corrections run:

- When the partner verification state is `not_verified`, the participant lookup runs.
- When the state is still not `valid`, the partner has an endpoint and the scheme is one of the two Belgian schemes, the alternative Belgian scheme is tried: the company registry scheme `0208` is converted to the tax identification scheme `9925` by prefixing the endpoint with the country prefix `BE`, and the tax identification scheme is converted to the company registry scheme by removing the first two characters. When the converted pair passes the endpoint validity rules and the lookup answers `valid`, the partner is rewritten with the converted pair and verified again. This silently repairs the very common confusion between the two Belgian identifiers.

---

# 9. Hand documents to the exchange network

**Actors:** the sending service, after the printed document has been rendered.

1. Group the documents to send by credential. For each document where the network method is selected and applicable:
   1. Build the payload parameters (section 9.1). A document for which no payload can be built is skipped, with its error already recorded.
   2. Append the payload to the parameter list of that credential and remember the document.
2. When nothing was collected, stop.
3. Take a row lock on all the collected accounting documents. When the lock cannot be taken, log an error and stop; the documents keep their state and can be sent again.
4. For each credential, call the send document endpoint with its list of payloads.
5. Commit when the caller allows it.

## 9.1 Building one payload

1. Resolve the credential of the company. When there is none, the document is skipped.
2. Take the structured file: the freshly generated attachment values when present; otherwise the stored structured file of the document when it exists and the document is not already sent; otherwise set `network_document_state` to `error`, record the error `Errors occurred while creating the EDI document (format: %s):` with the acronym expanded and the builder description, and skip.
3. When the document already has a printed document attached and the format is not the cross industry invoice family, re-run the embedding of section 8.2 so that the payload carries the printed document.
4. When the payload exceeds sixty four million bytes, record the error `Invoice %s exceeds the size limit of 64 MB to be sent via Peppol.` with the placeholder filled with the document name and the unit expanded to "megabytes", and skip.
5. Produce the payload: the file name, the receiver expressed as the scheme, a colon and the endpoint of the commercial partner, and the file content encoded in base sixty four.

## 9.2 Handling the answer

- **Branch A: the call raised.** For every document of that credential, set `network_document_state` to `error` and record the raised message as the error title.
- **Branch B: the answer carries an error.** The only error expected here is the one saying that the participant is not ready. For every document of that credential, set `network_document_state` to `error` and record the translated error message as the error title.
- **Branch C: success.** The answer carries one message entry per payload, **in the same order as the payloads were sent**. For each pair of message entry and document:
  1. Write `network_message_identifier` with the returned message identifier and `network_document_state` with `processing`.
  2. Split the wizard attachments into the ones the format embedded and the ones it could not. When some could not be embedded, log `Some attachments could not be sent with the markup file:` and those attachments linked to the log entry.
  3. Log `The invoice has been sent to the Peppol Access Point. The following attachments were sent with the markup file:`, attaching the embedded user attachments, the printed document and the structured file.
  4. Detach the attachments of that log entry from the accounting document and attach them to the log entry itself, clearing the main attachment of the document when it was one of them. This keeps the document attachment list clean while preserving the evidence of what was sent.
  5. Schedule the delivery state polling scheduled action to run in five minutes.

Because the correlation between payloads and answers relies on order, a replacement must preserve the order of the payload list and of the answer list.

---

# 10. Poll the delivery state of sent documents

**Actors:** scheduled job runner, or a user pressing "refresh outgoing electronic invoice status" on a journal.
**Preconditions:** at least one credential exists whose company state allows sending.

1. Select the credentials whose company participant state is `sender`, `smp_registration` or `receiver` and whose service is an exchange network service.
2. For each credential, in the context of its company:
   1. Collect the records awaiting a state: every accounting document of the company whose `network_document_state` is `processing`, limited to the batch size plus one so that an overflow can be detected; then, when the business response package is installed and the limit is not already reached, every Peppol Business Response of the company whose `delivery_state` is `processing`, limited to what is left plus one.
   2. When nothing is awaiting a state, move to the next credential.
   3. Note whether the collection exceeded the batch size; if so a retrigger will be needed.
   4. Index the first batch size records by their message identifier and call the get document endpoint with that list of identifiers.
   5. Process the answers (section 10.1).
   6. Call the acknowledge endpoint with the identifiers that were processed, so that the proxy stops returning them.
3. When a retrigger is needed, schedule the same scheduled action to run in five minutes.

The batch size is fifty by default and may be overridden by the calling context, which is how a deployment that handles very large documents lowers it.

## 10.1 Processing one delivery state answer

For a business response record:

| Condition | Effect |
|---|---|
| The answer carries an error with code 702, meaning the request is still being processed | Nothing is written and the identifier is **not** acknowledged, so the record is polled again. |
| The answer carries an error with code 207, meaning the recipient cannot receive this document type | `delivery_state` becomes `not_serviced`; the identifier is acknowledged. |
| The answer carries any other error | `delivery_state` becomes `error` and the accounting document receives the log `Peppol business response error: %s` with the message of the error; the identifier is acknowledged. |
| The answer carries a state | `delivery_state` takes that state; the identifier is acknowledged. |

For an accounting document:

| Condition | Effect |
|---|---|
| The answer carries an error with code 702 | Nothing is written and the identifier is not acknowledged. |
| The answer carries any other error | `network_document_state` becomes `error` and the document receives a log holding the translated error message; the identifier is acknowledged. |
| The answer carries a state | `network_document_state` takes that state and the document receives the log `Peppol status update: %s` with that state; the identifier is acknowledged. |

---

# 11. Poll the inbox for new documents

**Actors:** scheduled job runner, or a user pressing "fetch incoming electronic invoices" on the reception journal, or the inbox webhook.
**Preconditions:** at least one credential exists whose company participant state is `receiver`.

1. Select the credentials whose company participant state is `receiver` and whose service is an exchange network service.
2. For each credential, in the context of its company:
   1. **Journal check.** When the company has no reception journal, build the message `Please set a journal for Peppol invoices on %s before receiving documents.` with the company display name. The scheduled action logs it as a warning and continues; a manual call raises it.
   2. Call the list documents endpoint with the filter: direction incoming, no errors, receiver equal to the participant identification of the credential. A failure is logged and the credential is skipped.
   3. **Duplicate detection.** Collect the identifiers of the returned messages whose sender differs from the receiver. A message whose sender equals the receiver is a genuine self addressed document, and must not be treated as a duplicate, because the outgoing invoice already carries that identifier. Search for accounting documents of the company carrying one of those identifiers, and, when the business response package is installed, for business responses of the company carrying one of them. The union is the duplicate set.
   4. When the duplicate set is non empty, acknowledge those identifiers so the proxy stops returning them, and log which identifiers were discarded as duplicates.
   5. Build the list of identifiers to fetch by removing the duplicates. When the list is empty, move to the next credential.
   6. Note whether the list exceeds the job count, and truncate it to the job count.
   7. Call the get document endpoint with the truncated list.
   8. Process the messages (section 12).
   9. Commit, outside of tests, so that the imported documents survive a later failure.
   10. Acknowledge the identifiers that were processed and run the post processing of section 11.1.
3. When a retrigger is needed, schedule the same scheduled action to run again.

## 11.1 Post processing after an inbox batch

1. When the company has a reception journal, notify that journal that electronic invoices were received, which is the general ledger notification used to raise the counter on the journal card.
2. For every partner of the imported documents whose participant verification state is `not_verified` or empty, run the participant lookup, so that a response can be sent back to a partner that supports it.
3. When the business response package is installed, send an acknowledgement response for every imported document that may be answered (section 14.1).

---

# 12. Turn an inbound message into a record

**Actors:** the inbox polling flow.

For each message of the batch:

1. Determine the file extension and media type. The base rule returns the extension `xml` and the markup media type.
2. Take the file name from the message, defaulting to `attachment`.
3. Decrypt the payload: the symmetric key of the message is decrypted with the private key of the credential, then the payload is decrypted with that symmetric key.
4. Create an attachment with the name made of the file name, a full stop and the extension, the decrypted content, the binary kind and the media type. The attachment is not yet linked to a record.
5. **Branch by document kind.**
   - **A business response** (the message declares the response document type): extract the response information (section 14.4), find the accounting document of the company whose message identifier equals the origin message identifier of the response, and, when the document exists and the response code is one of the seven known codes, create a Peppol Business Response with the message identifier, the code, the delivery state of the message and the document. When the delivery state is `done`, log on the document: for a rejection, `The Peppol receiver of this document has rejected it with the following information:` followed by a line break and the formatted reasons and actions; for an acknowledgement, `The Peppol receiver of this document replied that he has received it.`; for an approval, `The Peppol receiver of this document replied that he has accepted it.` The identifier is marked processed.
   - **Any other document:** run section 13. When it produced a record, mark the identifier processed and collect the produced accounting document. A failure is logged with the message identifier and the identifier is **not** marked processed, so the message stays in the inbox.

---

# 13. Import a received document into an accounting document

**Actors:** the inbox polling flow; also the ordinary attachment import of the accounts receivable domain, for a file dropped by a user or received by electronic mail.

1. **Decide the document family.** Read the document type code of the file. The codes `389`, `527` and `261` mean a self billed document, that is a document the customer produced on behalf of this company.
   - **Self billed:** the target journal is the one supplied by the caller, otherwise the first sale journal of the company marked as self billing, otherwise the first sale journal of the company. The document type is a customer invoice. When no sale journal exists at all, stop and produce nothing.
   - **Not self billed:** the target journal is the one supplied by the caller, otherwise the reception journal of the company. The document type is a vendor bill. When there is no journal, stop and produce nothing.
   A failure while reading the type code is logged and treated as "not self billed".
2. **Create the accounting document** in that journal with that type, with `network_document_state` set to the state of the inbound message and `network_message_identifier` set to its identifier. When the extraction feature is present, mark the document as not extractable so that it is not sent to the extraction service.
3. **Extend the document with the attachment**, which runs the recognition of [entities.md](entities.md) section 12.10 and then the decoder of the matching profile. The decoder procedure is in [import-mapping.md](import-mapping.md).
4. **Run the automatic posting rule** of the accounts receivable domain, which posts the document when the same vendor sent the same shape of document three times in a row without a manual change.
5. **Link the attachment** to the created accounting document.
6. **Return** the message identifier and the created document.

**Guard:** a decoder refuses to run on an accounting document that already has lines, returning the standard "cannot decode, the document already has lines" reason of the accounts receivable domain. This prevents a second file from overwriting a document a user already worked on.

---

# 14. Business responses

**Actors:** billing clerk, accountant, the inbox polling flow.
**Preconditions:** the business response package is installed; the partner publishes the response transaction in its service list.

## 14.1 Acknowledge a received document

Immediately after an inbox batch has been imported and post processed, an acknowledgement response with the code `AB` is sent for every imported document whose `network_can_send_response` is true.

## 14.2 Approve a received document

When a vendor bill that may still be answered is posted, an approval response with the code identifier `AP` is sent, grouped by company.

## 14.3 Reject a received document

1. The user cancels a vendor bill, or presses the rejection action directly.
2. When at least one selected document may still be answered, the rejection wizard opens on those documents, carrying the result of the cancellation in its context.
3. The user picks at least one reason from the reason code list and optionally one or more suggested actions from the action code list. The default reason is the code `UNR`, commercial transaction not recognised.
4. On send, when no reason is selected, refuse with `At least one reason must be given when rejecting a Peppol invoice.`
5. The clarification list is built from the reasons followed by the actions, each entry carrying its list identifier, its code and its name.
6. Grouped by company, a rejection response with the code `RE` and that clarification list is sent.
7. The wizard returns the carried cancellation result, so the cancellation completes.

## 14.4 Sending a response

1. Keep the documents that have a message identifier and whose `network_can_send_response` is true. When none remain, stop.
2. Assert that the status is one of the code identifiers `AB`, `AP` and `RE`. A rejection without at least one reason from the reason list raises `At least one reason must be given when rejecting a Peppol invoice.`
3. Call the send response endpoint with the message identifiers of the referenced documents, the status and the clarification list.
4. **Branch A: the call raised.** Log on every referenced document `An error occurred while responding to this invoice's expeditor.` followed by a line break and `Status: <status> - <error text>`.
5. **Branch B: success.** Create one Peppol Business Response per returned message entry that carries a message identifier, pairing entries with documents by order, with `delivery_state` set to `processing`. Then log on the documents that got a response `A Peppol response was sent to the Peppol Access Point declaring you <status word> this document.` and on the documents that did not `A Peppol response declaring you <status word> this document could not be sent to the Peppol Access Point.` The status word is `received` for `AB`, `accepted` for `AP` and `rejected` for `RE`.

## 14.5 Reading an inbound response

1. Parse the response document.
2. Read the response code from the document response node.
3. For every status node under the response:
   - When both a status reason code and a status reason text are present, and the list identifier of the code is one of the two known lists, add the text to that list.
   - When only a code is present, and its list identifier is one of the two known lists, look the code up in the shipped clarifications of that list and add its name, or the raw code when no clarification matches.
   - When only a text is present, look for a shipped clarification whose name or description equals that text; add the text to the list of that clarification, or to a third "miscellaneous" list when nothing matches.
4. Build the formatted message: when reasons exist, the bold title `Reasons:` followed by a line break and the reasons formatted as a natural language list; when actions exist, a line break, the bold title `Suggested actions:` and the actions; when miscellaneous entries exist, a line break, the bold title `Miscellaneous:` and those entries.
5. Return the response code and the formatted message.

## 14.6 Keeping the published services in step

A scheduled action, shipped active with an interval so long that it effectively runs only when triggered, keeps the published document types of every receiver in step with what the platform supports.

1. Select the credentials whose service is the base exchange network service and whose company state is `receiver`.
2. Compute the full supported identifier set, and the same set without the response transaction identifier.
3. For each receiver, the target set is the full set when the company has a reception journal, and the reduced set otherwise. A participant that cannot file received documents must not advertise that it answers them.
4. Read the currently published services. When they already equal the target set, move on.
5. Remove the published identifiers that are not in the target set, then add the target identifiers that are not published.
6. A failure is logged per receiver and sets a retry flag; at the end, when the flag is set, the scheduled action is scheduled again in four hours.

The same scheduled action is triggered whenever the reception journal of a company is written, because adding or removing that journal changes the target set.

---

# 15. Register on the exchange network

**Actors:** system administrator, accountant.
**Preconditions:** the company has a country, a contact email and a mobile number.

## 15.1 Opening the wizard

The wizard is opened from the settings screen, from the sending flow when the company cannot send, or from the explanation dialogue. When it is opened from the sending flow the identifiers of the accounting documents are carried in the context, so that the sending action can be resumed afterwards.

## 15.2 Filling the form

1. The wizard proposes the participant parent company when the active company is a branch whose parent already has a connection. The user chooses between `use_parent` and `use_self`. Choosing the parent makes every configuration field of the wizard read and write the parent company.
2. The scheme defaults to the French electronic address scheme `0225` for a French company, otherwise to the scheme already on the selected company.
3. The endpoint is typed; every character that is not a letter or a digit is removed while typing.
4. The phone number is typed and rewritten in the international format using the country of the selected company.
5. The warnings of [entities.md](entities.md) section 9.2 are displayed as they become applicable.
6. The wizard asks the proxy whether a connection is possible for this identification (section 15.3), which decides whether the direct activation buttons or the identity login button is shown.

## 15.3 The connection enquiry

The enquiry is a public request, not signed by any credential, carrying the database unique identifier, the participant identification in lower case, the address the proxy must call back after an interactive authentication, the address the proxy must call when an asynchronous decision is taken, a signed connection token and the contact email. It returns whether authentication is required and which authentication methods are available.

The connection token is a signed structure holding the participant identification, the company identifier, the partner identifier of the current user and the creation instant. It expires after two weeks.

## 15.4 Activating

1. Re-assign the scheme from the selected company, because the derived field resets itself.
2. **Mandatory field checks**, in this order, each raising and stopping:
   - `Please select a country for your company.` when the fiscal country of the selected company has no code.
   - `Contact email and phone number are required.` when either is empty.
   - `Peppol Address should be provided.` when the scheme or the endpoint is empty.
   - `Peppol Identifier should be different from main company.`, when the user chose to register this branch itself and the scheme and endpoint equal the ones of the parent company.
   - `Cannot register a user with a %s application` where the placeholder is the translated label of the current participant state, when that state is not `not_registered`.
3. **Approved platform guard.** When the scheme is the French electronic address scheme `0225`, refuse with a redirect warning whose message is the approved platform installation message and whose button opens the package, because a French participant must use the approved platform service rather than the ordinary network service.
4. **Archive stale credentials.** Every credential of the company for the exchange network service with the same participant identification is archived.
5. **Conflicting service guard.** When the company already has a credential for another exchange network service, refuse with `A connection to '%s' already exists.` where the placeholder is the translated label of that service.
6. When the parent connection is used, copy the scheme, the endpoint, the contact email and the phone number onto the active company.
7. **Check the connection answer**, raising the first applicable message:
   - `Could not connect to Proxy Server.` when the answer is empty.
   - `Your identifier you entered is invalid for Peppol.` when the answer says the identifier is not on the network.
   - `Your identifier does not have a valid format.%s` where the placeholder is ` Expected format: %(expected_format)s.` when the answer carries an example, when the answer says the format is wrong.
   - `Your identifier is invalid.` for any other invalid identifier answer.
   - `The database you are trying to connect to is not suitable for Peppol.` when the answer says the database is not suitable.
   - `You need to authenticate to continue.` when no authentication was chosen and the answer requires one.
   - `Selected authentication method is not available.` when the chosen method is not among the available ones.
8. **Branch A: authentication is required.** Return an instruction to open the authorisation address of the chosen method in the same window. The flow continues in section 15.5.
9. **Branch B: no authentication is required.** Create the connection (section 15.6) and return a success notification whose message depends on the resulting state:
   - `sender`: `You can now send electronic invoices via Peppol.`
   - `smp_registration`: `Your Peppol registration will be activated soon. You can already send invoices.`
   - `receiver`: `You can now send and receive electronic invoices via Peppol`
   - `rejected`: title `Registration rejected.` and message `Your registration has been rejected. Please contact the support for further assistance.`
   The notification carries a follow up action: when the wizard was opened from the sending flow, the sending action of the carried accounting documents; otherwise closing the dialogue.

## 15.5 Returning from an interactive authentication

Two entry points exist.

**The browser callback** is requested by the proxy in the browser of the authenticated user. It carries the authentication kind, the connection token, an optional authentication token and an optional state.

1. Decode the connection token. When it cannot be decoded, log a warning with a truncated token and redirect to the settings screen with the result `failure`.
2. When the state is `canceled`, notify the partner of the user on the message bus with the result `canceled` and redirect to the settings screen.
3. When the state is `pending`, meaning a manual identity decision is still being taken, notify with the result `pending` and redirect to the root of the application rather than the settings screen, so that the user is not shown an unfinished registration form.
4. When no authentication token is present, log a warning and redirect with the result `failure`.
5. Create the connection with the authentication token. A user error is caught, logged, and redirected with the result `failure` and the error text.
6. Otherwise notify with the result `success` and redirect to the settings screen.

**The asynchronous webhook** is requested by the proxy, without a session, when a manual identity decision becomes positive.

1. Decode the connection token. When it cannot be decoded or no authentication token is present, log a warning and answer with the structured error `invalid_request` and the status code 400.
2. When the company already has a credential, answer with the status `already_connected`. The browser callback may have completed first.
3. Create the connection. A user error is caught, logged and answered with the status `failed`; the user can still finish from the link sent by electronic mail.
4. Otherwise notify the partner on the message bus with the result `success` and answer with the status `connected`.

## 15.6 Creating the connection

1. Determine the operating mode of the company.
2. Generate a two thousand and forty eight bit private key named after the service, the mode and the company identifier.
3. Collect the company details: display name, tax identification number, street, city, postal code, country code, phone number, contact email, migration key, webhook address, a freshly signed webhook token and the supported document type map.
4. Call the connect endpoint with the participant identification, the database unique identifier, the company identifier, the public key, the authentication token when there is one, and the company details. A failure is logged and re-raised.
5. Create the credential with the returned client identifier, the company, the exchange network service, the operating mode, the participant identification, the generated private key and the returned rotating token.
6. Write the returned participant state onto the company.
7. Commit when allowed.
8. When the resulting state is `sender`, send the welcome message.

The webhook token is a signed structure holding the company identifier and the webhook address; it expires after thirty days. It is verified by checking the signature and then checking that the address actually requested starts with the address inside the token, which prevents a token minted for one deployment being replayed against another.

## 15.7 Keeping the webhook alive

A scheduled action running every two weeks re-registers the webhook of every credential whose company state is `sender` or `receiver`, calling the set webhook endpoint with the webhook address and a freshly signed token. The token lifetime of thirty days is longer than the interval, so a missed run does not break the callbacks.

---

# 16. Upgrade, downgrade and deregistration

## 16.1 Sender becomes receiver

**Precondition:** the participant state is `sender`, otherwise refuse with `Cannot register a user with a %s application` and the translated state label.

1. Resolve the participant identification of the company.
2. Look the participant up on the network. When it already exists there, store the serving access point name on the company and refuse with the message described in [entities.md](entities.md) section 16.4, which tells the user to deregister from the other provider first.
3. Call the register sender as receiver endpoint with the migration key and the list of supported document type identifiers.
4. Clear the migration key, because it has been handed over; the field is kept for a future migration away.
5. Set the participant state to `smp_registration` and clear the serving access point name.
6. Schedule the participant state polling scheduled action to run in one hour.

## 16.2 Receiver becomes sender only

1. When the state is `receiver`, first poll the delivery states and the inbox, and commit, so that nothing is lost.
2. Call the unregister to sender endpoint.
3. Set the participant state to `sender`.

## 16.3 Deregistration

1. Ask the proxy for the participant state, calling the state endpoint directly rather than the tolerant polling operation. An error whose code says the client or the user no longer exists is swallowed; any other error is re-raised.
2. When the remote state is `sender`, `smp_registration` or `receiver`, first poll the delivery states and the inbox and commit, then call the cancel registration endpoint.
3. Reset the participant configuration of the company, which clears the state, the migration key, the serving access point, the scheme, the endpoint, the contact email and the phone number, and recomputes the scheme and the endpoint of the company partner.
4. Delete the credential.

A re-registration button performs the deregistration, or resets the configuration when there is no credential, and then opens the registration wizard.

## 16.4 Disconnecting a branch from its parent

The branch runs the ordinary deregistration and then shows the notification `Disconnected this branch company peppol configuration from %s.` with the previous parent company name.

---

# 17. Recover a connection whose credentials went out of step

**Actors:** system administrator.
**Preconditions:** a call failed with the invalid signature error, which happens after a database restore or an unneutralised copy.

## 17.1 Detection

When a call answers with the invalid signature code:

1. If the credential is already marked out of step, nothing more is done.
2. Otherwise the credential is written with `is_token_out_of_step` set to true and `refresh_token` cleared, and the change is flushed immediately, so that a concurrent transaction that renews the token cannot leave the database thinking it is still connected.
3. A call to the mark connection out of step endpoint is made, signed with the private key rather than the token, carrying the current step version.
4. When that call answers that the connection has been superseded, the database disconnects itself (section 17.3), commits, and raises `This connection has been superseded by another database. Register again.`
5. Every later call is refused locally with `Failed to connect to Peppol Access Point. This might happen if you restored a database from a backup or copied it without neutralization. To fix this, please go to Settings > Accounting > Peppol Settings and click on 'Reconnect this database'.`

## 17.2 Reconnecting this database

1. Increment the step version.
2. Call the resynchronise endpoint, signed with the private key, carrying the new step version.
3. When the answer carries an error saying the connection has been superseded, disconnect this database, commit, and raise the error; otherwise raise the error as is.
4. Otherwise store the returned rotating token and clear the out of step flag.
5. Schedule the participant state polling scheduled action, rather than calling it inline, because the proxy may confirm the token before this transaction commits, which would leave an unrecoverable state if the transaction then failed.

## 17.3 Disconnecting this database

Reset the participant configuration of the company in the soft form, which sets the state back to `not_registered` and clears the migration key while keeping the scheme, the endpoint, the contact email and the phone number so that the user can register again, then delete the credential. The remote connection is untouched: the other copy of the database keeps it.

---

# 18. Verify a trading partner

**Actors:** billing clerk, or any flow that needs to know whether a partner can receive.

1. When the partner has no scheme or no endpoint, stop and return false without changing anything.
2. Build the participant identification as the scheme, a colon and the endpoint, in lower case.
3. **Resolve the verification state:**
   1. When the scheme or the endpoint is missing, or the resolved format is not one the network accepts, the state is `not_verified`.
   2. Look the participant up through the proxy (section 18.1). A lookup that returns nothing gives the state `not_valid`.
   3. Check that the participant really exists: the returned participant identifier must equal the requested identification, compared without case, and the address of its first published service must not belong to the pre-registration directory of the Belgian federal service, because every Belgian company is pre-registered there without being a real participant.
   4. When the participant does not really exist, the state is `not_valid`.
   5. Otherwise check whether the participant publishes the customization identifier of the resolved format for the billing process. When it does, the state is `valid`; when it does not, the state is `not_valid_format`.
4. When the state changed, write it for the company in question and log the change in the discussion thread of the partner as an old value, an arrow, a new value, the field label and the company name. The change is logged rather than tracked as an ordinary field, because the value is per company and every company must see the history.
5. When the business response package is installed, also refresh the published document type list of the partner, so that `participant_supports_responses` becomes correct.

## 18.1 The participant lookup

The lookup is a public request to the proxy, carrying the participant identification in lower case as a query parameter. It is skipped entirely in demonstration mode. Failures, unreadable answers and answers whose error code is not "not found" are logged; every failure returns nothing, which the caller reads as "not on the network". A "not found" answer is logged only at debug level, because it is an ordinary outcome.

---

# 19. Export and import orders, and export point of sale receipts

## 19.1 Export a sales order

**Actors:** salesperson.

1. The order export operation of the sales domain asks every registered builder for a file. The ordering profile builder is one of them.
2. The builder assembles the values: the document type is an order; the supplier is the commercial partner of the company; the customer is the partner of the order; the delivery address is the shipping address of the order, otherwise the first delivery child of the customer, otherwise the customer.
3. The base lines are prepared from the order lines that are not display lines, the tax details are computed and rounded, negative unit prices are turned into negative quantities, emptying taxes are turned into extra lines, and the six digit rounding of the raw totals runs.
4. The tree is built: header, buyer customer party, seller supplier party, delivery, payment terms, order lines, allowances and charges, tax totals and the anticipated monetary total.
5. The file is serialised against the order template.

## 19.2 Export a purchase order

The same procedure, with the supplier being the partner of the order and the customer being the commercial partner of the company, with the delivery address being the destination address of the order, otherwise the first delivery child of the customer, otherwise the customer, and with an extra step that resolves, per line, the vendor catalogue entry of the product for that vendor so that the vendor item identifier and the vendor item name can be written.

## 19.3 Import an order

**Actors:** salesperson, buyer.

1. A structured file whose customization identifier is the ordering transaction identifier is recognised as an order of the matching profile.
2. The decoder reads the order level values: the order date from the validity end date, falling back to the issue date; the note from the document note; the payment term matched by name inside the company; the currency from the document currency code.
3. For a sales order, the note is dropped, because the terms and conditions of the sales order take precedence over the ones of the incoming purchase order; the customer is matched from the buyer customer party; the customer order reference is taken from the document identifier; the origin is taken from the quotation document reference; the shipping address is matched from the delivery party.
4. Document level allowances and charges become extra lines with sequence 0.
5. The order lines are read with the shared line reconstruction of [import-mapping.md](import-mapping.md). Deferred date fields are removed because an order has none. A line whose product could not be matched produces the message `Could not retrieve the product named: %(name)s`. A line discount is dropped, because the price on the order must come from the price list of the seller.
6. The order is written, the discussion thread receives `Format used to import the document: %s` with the description of the builder, and, when messages were collected, an activity is created carrying `Some information could not be imported:` followed by the list.
7. For a sales order, the unit price and the discount of every line that matched a product are recomputed from the price list.

## 19.4 Export a point of sale receipt

1. The document type is an invoice when the receipt total is not negative, otherwise a credit note.
2. The supplier is the commercial partner of the company and the customer is the partner of the receipt; the journal is the invoicing journal of the point of sale configuration; the document name is the receipt name.
3. The base lines come from the receipt tax base line preparation; the tax details are computed and rounded.
4. The tree is built with the universal business language 2.1 node builders, in the order: header, accounting supplier party, accounting customer party, payment means (empty), allowances and charges, tax totals, monetary total, lines. Note that the lines are built **after** the monetary total for this profile.
5. The monetary total is completed with a prepaid amount equal to the tax exclusive amount minus the amount paid, and a payable amount equal to the amount paid.
6. No constraint is contributed, so the export never fails on validation.

---

# 20. State machines, in summary

The tables below summarise the transitions that the procedures of this file fire. They are a reading aid, not the specification: [state-machines.md](state-machines.md) carries the eleven complete machines, with every state and its stored value and label, every guard in the order in which it is evaluated, the exact refusal of each guard, the records each transition creates or changes, and a diagram per machine. Where the two files appear to differ, the complete document governs.

## 20.1 Electronic Document `state`

| From | Trigger | Guard | To | Side effects |
|---|---|---|---|---|
| (none) | posting | the format applies and its configuration check passes | `to_send` | record created |
| `to_send` | send operation returns success | not at blocking level `error` | `sent` | attachment stored; `error` and `blocking_level` cleared |
| `to_send` | send operation returns a failure | not at blocking level `error` | `to_send` | `error` and `blocking_level` written |
| `to_send` | reset the accounting document to draft | reset is allowed | (deleted) | |
| `to_send` | cancel the accounting document | | `cancelled` | `error` and `blocking_level` cleared |
| `to_send` | remove the format from the journal | the format needs no remote call | (deleted) | |
| `sent` | request cancellation | the format needs a remote call, a `cancel` operation exists, the fiscal lock allows it | `to_cancel` | note posted |
| `sent` | cancel the accounting document | | `to_cancel` | `error` and `blocking_level` cleared |
| `sent` | post the accounting document again | the format still applies | `to_send` | attachment cleared |
| `to_cancel` | call off the cancellation | a `cancel` operation exists | `sent` | note posted |
| `to_cancel` | cancel operation returns success | not at blocking level `error` | `cancelled` | attachment cleared; the accounting document is cancelled when every record is settled |
| `to_cancel` | cancel operation returns a failure | not at blocking level `error` | `to_cancel` | `error` and `blocking_level` written |
| `to_cancel` | cancel the accounting document | | `cancelled` | `error` and `blocking_level` cleared |
| `cancelled` | post the accounting document again | the format still applies | `to_send` | attachment cleared |

## 20.2 Journal Entry `electronic_document_state`

This field is derived, not written. The table states which combination of delivery record states produces which value; only records whose format needs a remote call are considered.

| Combination | Value |
|---|---|
| every record is `sent` | `sent` |
| every record is `cancelled` | `cancelled` |
| at least one record is `to_send` | `to_send` |
| no record is `to_send` and at least one is `to_cancel` | `to_cancel` |
| no such record exists, or the combination matches none of the above | empty |

## 20.3 Company `participant_state`

| From | Trigger | Guard | To | Side effects |
|---|---|---|---|---|
| `not_registered` | activate the registration, no authentication required | every mandatory field check passes | the state returned by the proxy: `sender`, `smp_registration`, `receiver` or `rejected` | credential created; welcome message sent when the state is `sender` |
| `not_registered` | activate the registration, authentication required | the chosen method is available | unchanged until the callback or the webhook returns | the browser is sent to the authorisation address |
| `sender` | upgrade to receiver | the participant is not already published elsewhere | `smp_registration` | migration key cleared; state polling scheduled in one hour |
| `smp_registration` | the participant state poll returns `receiver` | | `receiver` | |
| `smp_registration` | the participant state poll returns `rejected` | | `rejected` | |
| `receiver` | downgrade to sender only | | `sender` | states and inbox flushed first |
| any | the participant state poll returns `draft` | | `not_registered` | the participant configuration is reset and the credential archived |
| any | the participant state poll fails with the client gone code | | `not_registered` | the participant configuration is reset and the credential archived |
| any | deregistration | | `not_registered` | states and inbox flushed first; registration cancelled remotely; configuration reset; credential deleted |
| any | disconnect this database | the credential is out of step | `not_registered` | soft configuration reset; credential deleted; the remote registration is untouched |

The poll maps the remote state names to the local ones: `draft` to `not_registered`, and `sender`, `smp_registration`, `receiver` and `rejected` to themselves. An unknown remote state is logged as a warning and changes nothing.

## 20.4 Journal Entry `network_document_state`

| From | Trigger | Guard | To | Side effects |
|---|---|---|---|---|
| empty | posting a sale document | the company can send and the commercial partner verification state is `valid` | `ready` | |
| `ready` or empty | the network pre send step | the network method is selected | `to_send` | |
| `to_send` | the payload could not be built, or the payload is too large, or the file generation failed | | `error` | the error is recorded on the sending result |
| `to_send` | the send call raised or answered with an error | | `error` | the error is recorded |
| `to_send` | the send call succeeded | | `processing` | message identifier stored; attachments logged; state polling scheduled in five minutes |
| `processing` | the state poll returns a state | | that state, typically `done` | a log entry records the new state |
| `processing` | the state poll returns an error other than "still processing" | | `error` | a log entry records the error |
| any before sending | the user cancels the network document | the document is not already sent | empty | the stored sending data is cleared |
| `done` | a received business response with delivery state `done` carries `AB` | | `AB` | |
| `done` or `AB` | a received business response carries `AP` or `PD` | | `AP` | |
| any | a received business response carries `RE` | | `RE` | a rejection log entry is written |
| any | the document goes back to draft and is not yet sent | the document is a sale document | empty | |

## 20.5 Peppol Business Response `delivery_state`

| From | Trigger | To | Side effects |
|---|---|---|---|
| (none) | a response is sent successfully | `processing` | record created with the returned message identifier |
| (none) | a response is received | the state carried by the inbound message | record created; the document is logged when the state is `done` |
| `processing` | the state poll returns a state | that state | |
| `processing` | the state poll returns the code 207, the recipient cannot receive this document type | `not_serviced` | the document can no longer be answered |
| `processing` | the state poll returns any other error | `error` | the document receives `Peppol business response error: %s` |
| `processing` | the state poll returns the code 702 | `processing` | nothing is written and the message is not acknowledged |

---

# 21. Upload a certificate

**Actors:** system administrator.

1. The user creates a Digital Certificate and uploads a file, optionally with a password.
2. The parsing step tries the binary encoding, then the container encoding, then the text bundle encoding, and the first that succeeds sets the original format, the normalised certificate, the subject common name, the serial number and the validity window. A failure with a password set stores `This certificate could not be loaded. Either the content or the password is erroneous.`
3. The private key inside the container or the bundle, when there is one, is normalised and stored as a Digital Key, reusing an existing key of the same company with the same content rather than creating a duplicate.
4. Every issuing certificate present in the upload that is not already in the database is created as an archived record named after its subject common name followed by ` (certificate authority)`.
5. The issuer link of the certificate is derived by matching the issuer common name, preferring the candidate with the furthest expiration date, and requiring either a cryptographic signature verification or, failing that, a match between the authority key identifier of the certificate and the subject key identifier of the candidate.
6. The validations refuse a certificate whose content is set but whose normalised form is empty, and a certificate whose linked private or public key does not match the key inside it.

**Postcondition:** the certificate can sign, provided it is inside its validity window and has a private key.
