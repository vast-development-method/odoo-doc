# Business rules

The complete rule catalogue of the Electronic Invoicing and Document Exchange domain. Each rule has a stable number so that other documents can cite it. Messages are reproduced exactly, with product names removed and acronyms expanded where the surrounding text made them ambiguous; where an acronym is part of the literal message the expansion is stated next to it.

Rule groups:

| Range | Group |
|---|---|
| 001 to 029 | Format registration and journal configuration |
| 030 to 059 | Electronic document lifecycle |
| 060 to 089 | Export: shared validation |
| 090 to 129 | Export: European semantic standard and Peppol validation |
| 130 to 149 | Export: country profile validation |
| 150 to 179 | Export: cross industry invoice validation |
| 180 to 209 | Import |
| 210 to 259 | Exchange network: identification, registration and connection |
| 260 to 289 | Exchange network: sending, receiving and responses |
| 290 to 309 | Certificates and keys |
| 310 to 329 | Rounding, currency and company consistency |
| 330 to 349 | Permissions and visibility |

---

## Format registration and journal configuration

**EIDI-RULE-001.** The code of an Electronic Document Format is unique across the whole installation. Violation message: `This code already exists`.

**EIDI-RULE-002.** A format may only be switched on for a journal for which `is_compatible_with_journal` answers true. The default compatibility rule accepts a journal only when `journal.type = "sale"`.

**EIDI-RULE-003.** A format that is compatible with a journal and answers true to `is_enabled_by_default_on_journal` is switched on automatically when the list of formats of that journal is recomputed.

**EIDI-RULE-004.** A format may not be removed from a journal while at least one Electronic Document of that journal, for that format, is in state `to_send` or `to_cancel` **and** that format needs a remote call. Message: `Cannot deactivate (%s) on this journal because not all documents are synchronized`, the placeholder listing the display names of the affected formats.

**EIDI-RULE-005.** When a format that needs no remote call is removed from a journal, the Electronic Document records of that journal for that format in state `to_send` or `to_cancel` are deleted. Nothing else is deleted.

**EIDI-RULE-006.** The recomputation of the formats of a journal never removes a format that still has an Electronic Document in state `to_send` or `to_cancel` on a document of that journal, even when that format would no longer be enabled by default.

**EIDI-RULE-007.** Creating a format whose `needs_remote_call` answers true switches the sending scheduled action on. The scheduled action is shipped inactive.

**EIDI-RULE-008.** While the application registry is still loading, the recomputation of the journal format lists is deferred to the end of the loading sequence, because the helper operations of a format are only complete once every localisation package is loaded.

**EIDI-RULE-009.** A journal marked as the reception journal of the exchange network must be a purchase journal. Changing its type is refused with `You can't change the type of a journal used for Peppol invoice reception to a type different than 'Purchase'.\nPlease change the journal used for Peppol reception before changing the type of this journal.`

**EIDI-RULE-010.** At most one purchase journal of a company carries the reception mark. Writing the reception journal of a company clears the mark on every other purchase journal of that company and sets it on the new one.

---

## Electronic document lifecycle

**EIDI-RULE-030.** At most one Electronic Document exists per pair of accounting document and format. Violation message: `Only one electronic document by accounting document by format`.

**EIDI-RULE-031.** Posting an accounting document is refused when any applicable format reports configuration errors. Message: `Invalid invoice configuration:\n\n%s`, the placeholder holding the error messages joined by a line break. The whole posting is rolled back.

**EIDI-RULE-032.** Posting an accounting document that already has an Electronic Document for an applicable format reuses that record, setting its state to `to_send` and clearing its attachment, rather than creating a second one.

**EIDI-RULE-033.** An Electronic Document at blocking level `error` is never included in a job. It is only reachable through the retry operation, which clears `error` and `blocking_level` on **every** record of the accounting document.

**EIDI-RULE-034.** An Electronic Document at blocking level `warning` or `info` is included in every later run of the sending scheduled action. A format therefore implements a multiple step remote protocol by returning an error text with blocking level `info` and reading that text back in its applicability on the next run.

**EIDI-RULE-035.** A job never mixes formats, never mixes states and never mixes companies. A job that mixes states is an internal consistency failure, reported as `All electronic documents of a job should have the same state`.

**EIDI-RULE-036.** A format that declares no batching key produces one job per accounting document, because the accounting document identifier is part of the batching key.

**EIDI-RULE-037.** When a send returns success and carries an attachment, the previous attachment is replaced. The previous attachment is deleted when it is linked to no business record, that is when it has neither a model name nor a record identifier.

**EIDI-RULE-038.** When a send returns no success and carries an error text without a blocking level, the level defaults to `error`. When it carries no error text, the level is cleared.

**EIDI-RULE-039.** When a cancellation returns success, the accounting document is reset to draft and cancelled if and only if it is posted and every one of its Electronic Document records is either `cancelled` or belongs to a format that needs no remote call.

**EIDI-RULE-040.** A cancellation may only be requested for an Electronic Document whose format needs a remote call, whose state is `sent`, and whose applicability declares a `cancel` operation. The fiscal lock dates of the accounting document are checked first.

**EIDI-RULE-041.** An accounting document may not be reset to draft while `electronic_document_show_cancel_button` is true. Message: `You can't edit the following journal entry %s because an electronic document has already been sent. Please use the 'Request EDI Cancellation' button instead.`, with the acronym expanded to "electronic document".

**EIDI-RULE-042.** Resetting an accounting document to draft clears the error and the blocking level of every Electronic Document of that document and deletes the records in state `to_send`.

**EIDI-RULE-043.** Accounting documents that already have an Electronic Document in state `sent` for a format that needs a remote call may not be resequenced. Message: `The following documents have already been sent and cannot be resequenced: %s`, the placeholder listing the distinct names of the affected documents.

**EIDI-RULE-044.** An attachment that is the payload of an Electronic Document whose format needs a remote call may not be deleted. Message: `You can't unlink an attachment being an electronic document sent to the government.` The check runs with administrator rights so that it applies to every caller.

**EIDI-RULE-045.** An outgoing message is not sent to the customer while any Electronic Document of the accounting document is in state `to_send`. The readiness check of the accounts receivable domain answers false in that case.

**EIDI-RULE-046.** The payload preview of an Electronic Document shows the configuration errors instead of a payload when the format reports any, and is empty for a record whose state is `sent` or `cancelled`.

**EIDI-RULE-047.** The structured file field of an accounting document is not copied when the document is duplicated, and is detached when the attachments of the document are detached.

**EIDI-RULE-048.** The structured markup file is never made the main attachment of an accounting document when another attachment is available. When the current main attachment is the payload of an Electronic Document and more than one attachment is present, the main attachment is replaced.

**EIDI-RULE-049.** Processing an Electronic Document requires a row lock on the record, on its accounting document and on the payload attachments that are linked to no business record. A job whose lock cannot be taken is skipped by the scheduled action and refused by the manual operation with `This document is being sent by another process already. `

---

## Export: shared validation

**EIDI-RULE-060.** Every tax used on the lines of the accounting document must have a valid repartition structure. Message: `Tax '%(tax_name)s' is invalid: %(error_message)s`.

**EIDI-RULE-061.** Every line that is not a section, a subsection or a note, and for which the general ledger says a tax is required, must carry at least one tax. Message: `Each invoice line should have at least one tax.`

**EIDI-RULE-062.** The supplier must have a name. The required field check produces `The field 'Name' is required on %(record)s.` with the display name of the supplier, or, when several field names are checked at once, `At least one of the following fields %(field_list)s is required on %(record)s.`

**EIDI-RULE-063.** The commercial partner of the customer must have a name. Same message shape as EIDI-RULE-062.

**EIDI-RULE-064.** The accounting document must have a name and an invoice date. Same message shape.

**EIDI-RULE-065.** When a required field check is run against a structure rather than a record, or when the caller supplies its own message, the message is the supplied one, or `The element %(record)s is required on %(field_list)s.` when none is supplied.

**EIDI-RULE-066.** A unit price that is negative is turned into a positive unit price and a negative quantity before the file is built, so that the item net price and the item gross price are never negative.

**EIDI-RULE-067.** A tax whose amount type is neither a percentage nor one of the recognised fixed shapes is not reported as a tax category at all; a recycling contribution tax and an excise tax are reported as line level allowances or charges instead.

**EIDI-RULE-068.** A recycling contribution tax is a tax whose amount type is a fixed amount and whose flag "include in base amount" is true. An excise tax is a tax whose amount type is computed by code and whose flag "include in base amount" is true. An emptying tax is a tax whose amount type is a fixed amount or is computed by code and whose flag "include in base amount" is false; it is reported as an extra document line.

---

## Export: European semantic standard and Peppol validation

These rules apply to every profile that inherits the European semantic standard layer, which is every profile of the universal business language family except the plain 2.0 and 2.1 syntaxes and the Belgian profile.

**EIDI-RULE-090.** An invoice must carry either a buyer reference or a purchase order reference. Message: `A buyer reference or purchase order reference must be provided.`

**EIDI-RULE-091.** Every document line must carry an item name. Message: `Each invoice line should have a product or a label.`

**EIDI-RULE-092.** Every document line must carry exactly one classified tax category. Message: `Each invoice line shall have one and only one tax.`

**EIDI-RULE-093.** Lines whose tax category is "services outside scope of tax" may not be mixed with lines of any other tax category. Message: `Taxes of category 'Service outside scope of tax' shall not be mixed with tax from other categories. You should split your invoice in two`

**EIDI-RULE-094.** The postal address of the supplier must carry a country code. Message: `The country is required for the supplier.`

**EIDI-RULE-095.** The postal address of the customer must carry a country code. Message: `The country is required for the customer.`

**EIDI-RULE-096.** A tax identification number written under the value-added tax scheme must start with two letters, the country prefix. Message for the supplier: `The tax identification number of the supplier should be prefixed with its country code.` Message for the customer: `The tax identification number of the customer should be prefixed with its country code.` The acronym expands to "tax identification number".

**EIDI-RULE-097.** When a delivery address is present, it must carry a country code. The required field check on the country of the delivery partner produces the message of EIDI-RULE-062.

**EIDI-RULE-098.** When the payment means code is 30, credit transfer, or 58, instructed credit transfer, a payee financial account must be present. The check is the required field check on the bank account of the accounting document.

**EIDI-RULE-099.** For an intra community supply, that is when the customer country and the supplier country are both in the European union and differ, the delivery location must be present. Message: `For intracommunity supply, the delivery address should be included.`

**EIDI-RULE-100.** For an intra community supply, either the actual delivery date or the invoicing period must be present. Message: `For intracommunity supply, the actual delivery date or the invoicing period should be included.`

**EIDI-RULE-101.** A tax breakdown entry whose category is "exempt from tax" and that carries no exemption reason receives the default reason text `Exempt from tax`.

**EIDI-RULE-102.** For a Norwegian supplier, the tax identification number must be the country prefix `NO`, a valid nine digit Norwegian organisation number and the letters `MVA`, giving a total length of fourteen characters. Message: `The tax identification number of the supplier does not seem to be valid. It should be of the form: NO179728982MVA.`, with the acronym expanded to "tax identification number".

**EIDI-RULE-103.** For a Belgian supplier that has a company registry number, that number must be a valid Belgian tax identification number. Message: `%s should have a valid company registry number in the Company Identifier field` with the display name of the supplier; the two national registry short names of the original sentence are replaced by the words "company registry".

**EIDI-RULE-104.** The same rule applies to a Belgian customer with a company registry number, with the display name of the customer.

**EIDI-RULE-105.** When the document is sent over the exchange network, the customer must have an electronic address. Message: `[PEPPOL-EN16931-R010] An electronic address scheme must be provided on the customer '%s'.`

**EIDI-RULE-106.** When the document is sent over the exchange network, the company must have an electronic address. Message: `[PEPPOL-EN16931-R020] An electronic address scheme must be provided on the company '%s'.`

Rules 105 and 106 apply only when the sending flow marked the generation as bound for the network, because the same file may be produced for a consumer who only needs it for an accountant.

---

## Export: country profile validation

**EIDI-RULE-130.** For the German electronic invoice profile, the supplier must have a telephone number. The required field check produces the message of EIDI-RULE-062 with the field label of the telephone number.

**EIDI-RULE-131.** For the German electronic invoice profile, the supplier must have an electronic mail address. Same message shape.

**EIDI-RULE-132.** For a Netherlands supplier, the postal address must carry a street name, a postal code and a city name. Three separate required field checks.

**EIDI-RULE-133.** For a Netherlands supplier whose legal entity identifier is neither the chamber of commerce scheme `0106` nor the organisation identification scheme `0190`, a tax identification number is required.

**EIDI-RULE-134.** For a Netherlands supplier, a payment means must be present; when none is, a bank account on the accounting document is required.

**EIDI-RULE-135.** For a Netherlands supplier invoicing a Netherlands customer, the customer postal address must carry a street name, a city name and a postal code.

**EIDI-RULE-136.** For a Netherlands supplier invoicing a Netherlands customer whose legal entity identifier is neither `0106` nor `0190`, a tax identification number is required on the customer.

**EIDI-RULE-137.** For a Netherlands supplier issuing a credit note, a preceding invoice reference must be present; when none is, the customer reference field of the accounting document is required.

**EIDI-RULE-138.** The German electronic invoice profile always writes a buyer reference: the endpoint of the customer when its scheme is the German routing identifier scheme `0204`, otherwise the reference of the commercial partner, otherwise the literal `N/A`.

---

## Export: cross industry invoice validation

**EIDI-RULE-150.** For a customer invoice, the accounting document must carry a bank account, and that bank account must carry a sanitised account number. Messages: the required field check on the bank account, and `The field 'Sanitized Account Number' is required on the Recipient Bank.`

**EIDI-RULE-151.** The commercial partner of the supplier must have a country.

**EIDI-RULE-152.** The supplier must have a tax identification number.

**EIDI-RULE-153.** The commercial partner of the supplier must have a telephone number.

**EIDI-RULE-154.** The supplier must have an electronic mail address.

**EIDI-RULE-155.** The commercial partner of the customer must have a country.

**EIDI-RULE-156.** For an intra community supply, the supplier must have a tax identification number and the commercial partner of the customer must have a tax identification number.

**EIDI-RULE-157.** When the customer is in Spain and the first two characters of the postal code are `35` or `38`, that is the Canary Islands, every line must carry at least one tax whose rate is greater than zero. Message: `When the Canary Island General Indirect Tax (IGIC) applies, the tax rate on each invoice line should be greater than 0.`

**EIDI-RULE-158.** Every line must carry at least one tax. Message: `You should include at least one tax per invoice line. [BR-CO-04]-Each Invoice line (BG-25) shall be categorized with an Invoiced item tax category code (BT-151).`, with the acronym expanded to "tax category code".

---

## Import

**EIDI-RULE-180.** A decoder refuses to run on an accounting document that already has lines, returning the "cannot decode, the document already has lines" reason of the accounts receivable domain.

**EIDI-RULE-181.** The document family of an inbound file is decided by the root element and, for the universal business language family, by the sign of the tax inclusive amount: an invoice root with a negative tax inclusive amount is a credit note whose quantities must be negated; a credit note root is a credit note with unchanged quantities. For the cross industry invoice family the type code decides: `381` and `261` are credit notes with unchanged quantities; `380`, `389` and `527` are invoices, except when the grand total amount is negative, in which case they are credit notes whose quantities must be negated.

**EIDI-RULE-182.** The document type of the created accounting document is the direction of its journal, `out` for a sale journal and `in` for a purchase journal, combined with the family decided by EIDI-RULE-181. When the decided type differs from the type the document already had, the type is changed only when the two types belong to the same family, that is when the pair is customer invoice and customer credit note, or vendor bill and vendor credit note; otherwise the import stops without touching the document. A change to a credit note adds the message `The invoice has been converted into a credit note and the quantities have been reverted.`

**EIDI-RULE-183.** A journal that is neither a sale journal nor a purchase journal cannot receive an import; the import stops.

**EIDI-RULE-184.** The partner search order for the universal business language family is: tax identification number, then electronic address scheme and endpoint, then bank account number, then electronic mail address, then telephone number, then name. For the cross industry invoice family the bank account step is absent.

**EIDI-RULE-185.** A partner is created during an import only when the file supplies both a name and a tax identification number. Message when a partner was created because none matched: `Could not retrieve a partner corresponding to '%s'. A new partner was created.` Message when a partner matched but its stored tax identification number differs from the one in the file: `Could not retrieve a partner corresponding to '%s' with the same tax identification number. A new partner was created.`, with the acronym expanded to "tax identification number".

**EIDI-RULE-186.** When a matched partner has no tax identification number and the file supplies one, the number is written on the matched partner after the country specific validation, rather than creating a second partner.

**EIDI-RULE-187.** Comparison of tax identification numbers during import ignores spaces and case on both sides, ignores full stops on the side coming from the file, and, for a Swiss country, additionally removes hyphens and full stops on both sides and removes a trailing tax regime suffix in either of the three national languages.

**EIDI-RULE-188.** When the currency named in the file does not exist, the currency of the company is used and the message `Could not retrieve currency: %s. Did you enable the multicurrency option and activate the currency?` is added. When the currency exists but is archived, it is used and the message `The currency '%s' is not active.` is added.

**EIDI-RULE-189.** The exchange rate used for the imported document is the rate between the company currency and the document currency at the invoice date of the document, or at today when the file supplies no invoice date.

**EIDI-RULE-190.** A bank account named in the file is attached to the partner when the document is a customer credit note or a vendor bill, and to the company partner when the document is a customer invoice or a vendor credit note. A failure to create the bank account adds the message `The bank account couldn't be fetched: %s`.

**EIDI-RULE-191.** The product search order is the shipped order of the product matching strategies, extended by the variant level identifier strategies at ranks 12 and 14, and finally the prediction strategy based on the label of the line, which is tried last.

**EIDI-RULE-192.** A unit of measure resolved from the unit code of the file is discarded, and the unit of the line is left empty, when it has no common reference with the unit of the matched product. Message: `The Unit of Measure '%(unit)s' (from unit code '%(code)s') was ignored on the line for product '%(product)s' because it is not compatible with the product's Unit of Measure '%(product_unit)s'. The unit of measure was left empty.`, with the acronym expanded to "unit of measure".

**EIDI-RULE-193.** A tax that cannot be matched adds the message `Could not retrieve the tax: %(tax_percentage)s %% for line '%(line)s'.` when the line has a label, and `Could not retrieve the tax: %s for the document level allowance/charge.` when it has none.

**EIDI-RULE-194.** Taxes are matched first among taxes whose price is excluded, then among taxes whose price is included. When a tax whose price is included is matched by the legacy import path, the unit price of the line is multiplied by one plus the rate divided by one hundred, so that the resulting line still totals the amount stated in the file.

**EIDI-RULE-195.** A line level charge whose reason code is `AEO` is first offered to the fixed tax matching, because that reason code is how this platform writes a fixed tax. The fixed tax search order is: not price included and matching both the name and the amount; not price included and matching the amount; price included and matching both; price included and matching the amount.

**EIDI-RULE-196.** A charge that could not be matched to a fixed tax is folded into the unit price and the discount of its line, so that the line still totals the amount stated in the file. The formula is in [calculations.md](calculations.md).

**EIDI-RULE-197.** After the lines have been written, the tax amounts are corrected against the tax totals stated in the file, but only when every tax stated in the file could be matched and the absolute difference between the computed tax total and the stated tax total is at most three hundredths of the document currency. Outside that tolerance nothing is corrected, because the discrepancy means the error is elsewhere.

**EIDI-RULE-198.** After the tax correction, the untaxed total is corrected against the tax exclusive amount plus the payable rounding amount stated in the file, minus the amounts of the line charges that matched a fixed tax. When a difference remains, an extra line labelled `Rounding` with quantity one, that difference as its unit price and no tax is added.

**EIDI-RULE-199.** A payable rounding amount stated in the file that does not come from a cash rounding produces an extra line labelled `Rounding` with the message `A rounding amount of %s was detected.`

**EIDI-RULE-200.** A prepaid amount stated in the file produces the message `A payment of %s was detected.` and is not written as a line.

**EIDI-RULE-201.** After the lines have been written, lines whose tax inclusive total is zero in the document currency and whose discount is zero are removed, because they carry no information.

**EIDI-RULE-202.** The file itself is stored in `electronic_invoice_markup_file` only for a purchase document including receipts. A customer invoice keeps its exported file in the same field, written by the export flow.

**EIDI-RULE-203.** The discussion thread of the imported document receives the title `Format used to import the invoice: %s` with the display name of the builder, followed by the list of collected messages with duplicates removed while preserving their order. When the document carries a network message identifier, the title becomes `Peppol invoice received` and the first message becomes `Peppol document UUID: %s` with the acronym expanded to "universally unique identifier".

**EIDI-RULE-204.** When the file embeds documents in supported media types, each one is created as an attachment of the accounting document. The file name is normalised: any directory path, in either separator convention, is stripped, the extension is replaced by the one of the declared media type, and a missing name becomes `invoice`. Base sixty four padding is repaired before decoding.

**EIDI-RULE-205.** When the file embeds no document, the accounting document has no main attachment, the document is a purchase document, and the configuration parameter that disables the substitute printed document is false, a printed document is rendered from the dedicated report and attached, and it becomes the main attachment. Its name is the display name of the document with the word `Draft` removed, followed by ` - Generated by the system`. A failure is logged and does not stop the import.

**EIDI-RULE-206.** When the accounting document received a structured file by electronic mail and that file became the main attachment, a decoded portable document replaces it as the main attachment.

**EIDI-RULE-207.** Grouping or ungrouping the lines of an imported document is only allowed while the document is in draft. Message: `You can only (un)group lines of a draft invoice`. Grouping is only allowed on an invoice including receipts. Message: `You can only group lines of an invoice`. With the purchasing package installed, neither is allowed on a document whose lines are linked to a purchase order. Message: `You can only (un)group lines of an invoice not linked to a purchase order`.

**EIDI-RULE-208.** Ungrouping requires the original file. Message when it is missing: `Cannot find the origin file, try by importing it again`. Message when no decoder can be resolved for it: `Cannot decode origin file, try by importing it again`.

---

## Exchange network: identification, registration and connection

**EIDI-RULE-210.** The participant identification of a company on the exchange network is the electronic address scheme code, a colon and the endpoint value. When either is missing the operation is refused with `Please fill in the electronic address scheme code and the Participant Identifier code.`

**EIDI-RULE-211.** A company has at most one active credential per service and per operating mode. Violation message: `This company has an active user already created for this electronic interchange type`, with the acronym expanded to "electronic data interchange".

**EIDI-RULE-212.** The client identifier of a credential is unique across the installation. Violation message: `This id_client is already used on another user.`, with the abbreviation expanded to "client identifier".

**EIDI-RULE-213.** A company cannot hold credentials for two different exchange network services at once. Message: `A connection to '%s' already exists.` with the translated label of the conflicting service. The database level counterpart of this rule is an exclusion constraint on the pair of company and operating mode for the active credentials of the exchange network services.

**EIDI-RULE-214.** Registration is refused when the fiscal country of the selected company has no code. Message: `Please select a country for your company.`

**EIDI-RULE-215.** Registration is refused when the contact email or the phone number is empty. Message: `Contact email and phone number are required.`

**EIDI-RULE-216.** Registration is refused when the scheme or the endpoint is empty. Message: `Peppol Address should be provided.`

**EIDI-RULE-217.** A branch company that chooses to register itself must use a participant identification that differs from the one of its parent company. Message: `Peppol Identifier should be different from main company.`

**EIDI-RULE-218.** Registration is refused when the participant state of the company is not `not_registered`. Message: `Cannot register a user with a %s application` with the translated label of the current state.

**EIDI-RULE-219.** A participant whose scheme is the French electronic address scheme `0225` may not register on the ordinary exchange network service; the approved platform package must be installed instead. The refusal is a redirect warning whose message is `To use the Approved Platform for French E-Invoicing install the module '%s'.` and whose button opens the package, or, when the package is not in the catalogue, the same message followed by `\nThe module was not found. Please update the app list first.` and a button that refreshes the catalogue.

**EIDI-RULE-220.** The connection enquiry answer is checked before any connection is created. The messages, in order of precedence, are: `Could not connect to Proxy Server.`; `Your identifier you entered is invalid for Peppol.`; `Your identifier does not have a valid format.%s` where the placeholder is ` Expected format: %(expected_format)s.` when an example is supplied; `Your identifier is invalid.`; `The database you are trying to connect to is not suitable for Peppol.`; `You need to authenticate to continue.`; `Selected authentication method is not available.`

**EIDI-RULE-221.** The blocking endpoint rules per scheme are enforced on the company: scheme `0007` requires a valid Swedish organisation number; `0088` requires a valid international article number; `0184` requires a valid Danish central business register number; `0192` requires a valid Norwegian organisation number; `0208` requires a valid Belgian tax identification number. Message: `The Peppol endpoint identification number is not correct.`

**EIDI-RULE-222.** The advisory endpoint rules per scheme produce a warning in the registration wizard and never block: scheme `0151` expects a valid Australian business number; `0201` expects exactly six alphanumeric characters; `0210` and `9907` expect a valid Italian fiscal code; `0211` and `9906` expect a valid Italian tax identification number. Message: `The endpoint number might not be correct. Please check if you entered the right identification number.`

**EIDI-RULE-223.** The endpoint validity rules per scheme are enforced on every contact whenever the endpoint changes. They are listed in [entities.md](entities.md) section 15.2 with their exact messages.

**EIDI-RULE-224.** Before a company is created or written, the endpoint is rewritten by the per scheme extractor: scheme `0007` keeps the first run of ten digits; `0184` the first run of eight; `0192` the first run of nine; `0208` the first run of ten. When the extractor finds nothing the typed value is kept.

**EIDI-RULE-225.** The scheme and the endpoint of the partner of a company whose participant state allows sending are never recomputed. A registered company keeps exactly the identification it registered with.

**EIDI-RULE-226.** A phone number stored as the participant phone number must parse as a valid international number. Message: `Please enter the mobile number in the correct international format.\nFor example: +32123456789, where +32 is the country code.` When the phone number library is unavailable, the message is `Please install the phonenumbers library.`

**EIDI-RULE-227.** The precomputation of the participant phone number from the company phone number only happens when the company phone number passes the same validation; a failing value is silently skipped rather than stored.

**EIDI-RULE-228.** A participant may only be upgraded from sender to receiver when it is not already published on the network under another access point. Message: `A participant with these details has already been registered on the network. If you have previously registered to a Peppol service, please deregister.`, extended with `The Peppol service that is used is %s.` when the serving access point can be named and is not this platform.

**EIDI-RULE-229.** Every Belgian company is pre-registered in the federal pre-registration directory without being a real participant. A lookup whose first published service address belongs to that directory is therefore treated as "participant does not exist".

**EIDI-RULE-230.** A participant identification is compared without case in every lookup.

**EIDI-RULE-231.** When the operating mode of the company resolves to the demonstration mode, no request leaves the platform. The mode resolves to demonstration when the scheme in force is the demonstration scheme `odemo`, otherwise to the mode of the existing credential, otherwise to the value of the configuration parameter that names the environment, otherwise to production.

**EIDI-RULE-232.** The demonstration scheme is offered in the scheme list of a contact only while the company operates in demonstration mode.

**EIDI-RULE-233.** A retired scheme is not offered in the scheme list of a contact unless it is the scheme currently stored on that contact. The retired schemes are `0037`, `0213`, `9955` and `0193`.

**EIDI-RULE-234.** A request to the proxy is signed. The default signature is a keyed hash of the canonical message with the rotating token as the key; the alternative signature is a private key signature of the same message, used only to recover a connection whose token is out of step.

**EIDI-RULE-235.** A rotating token is renewed on demand when the proxy answers that it expired, and the renewal is committed before the original call is retried, so that a later failure cannot lose the new token.

**EIDI-RULE-236.** Only one transaction renews a rotating token at a time. A transaction that cannot take the row lock returns without renewing, because another one is already doing it.

**EIDI-RULE-237.** A proxy answer saying the user does not exist archives the credential, because the participant identification was claimed by somebody else.

**EIDI-RULE-238.** A proxy answer saying the signature is invalid marks the credential out of step and refuses every later call with `Failed to connect to Peppol Access Point. This might happen if you restored a database from a backup or copied it without neutralization. To fix this, please go to Settings > Accounting > Peppol Settings and click on 'Reconnect this database'.`

**EIDI-RULE-239.** A connection that the proxy declares superseded is removed locally: the participant configuration is reset in its soft form, the credential is deleted, and the operation raises `This connection has been superseded by another database. Register again.` The remote registration is untouched.

**EIDI-RULE-240.** A deregistration first flushes the delivery states and the inbox and commits, so that no document is lost, and only then cancels the registration remotely.

**EIDI-RULE-241.** A call reaching a credential that is not of an exchange network service is refused with `Interchange user should be of one of the following types: %s`, the acronym expanded to "electronic data interchange" and the placeholder listing the translated labels of the accepted services joined with "or".

**EIDI-RULE-242.** A webhook token is accepted only when its signature verifies and the address that was actually requested starts with the address recorded inside the token. A token minted for one deployment can therefore not be replayed against another.

**EIDI-RULE-243.** A connection token expires two weeks after it was minted; a webhook token expires thirty days after it was minted. The keep alive scheduled action runs every two weeks, which is shorter than the webhook token lifetime.

---

## Exchange network: sending, receiving and responses

**EIDI-RULE-260.** An accounting document is eligible for sending over the network only when the country of the commercial partner is an eligible country, the method is applicable to the company, the verification state of the commercial partner is `valid`, the participant state of the company is not `rejected`, and the document either still needs a structured file or already has one and is not yet sent.

**EIDI-RULE-261.** Before that check, a partner whose verification state is `not_verified` is verified, and a Belgian partner whose verification failed is retried with the other Belgian scheme, converting the endpoint by adding or removing the country prefix. A successful retry rewrites the scheme and the endpoint of the partner.

**EIDI-RULE-262.** Sending a single document over the network is refused when the verification state of the commercial partner is not `valid`. Message: `Partner doesn't have a valid Peppol configuration.`

**EIDI-RULE-263.** A payload larger than sixty four million bytes is refused with `Invoice %s exceeds the size limit of 64 MB to be sent via Peppol.`, the unit expanded to "megabytes".

**EIDI-RULE-264.** The correlation between the payloads sent in one call and the message identifiers returned by that call relies on order. A replacement must preserve the order of both lists.

**EIDI-RULE-265.** A document whose file generation failed is marked in state `error` and is not handed to the proxy, so that the user sees why nothing was sent.

**EIDI-RULE-266.** A document that has already left the platform may not be cancelled locally. Message: `Cannot cancel an entry that has already been sent to PEPPOL`. A document has left the platform when its network state is none of empty, `ready`, `to_send`, `error` and `skipped`.

**EIDI-RULE-267.** A sale document that has already left the platform may not be reset to draft; the reset button is hidden.

**EIDI-RULE-268.** Reception requires a reception journal on the company. The scheduled action logs `Please set a journal for Peppol invoices on %s before receiving documents.` and continues; a manual call raises it.

**EIDI-RULE-269.** An inbound message whose identifier already exists on an accounting document or on a business response of the same company is acknowledged and discarded as a duplicate, except when its sender equals its receiver, which is a genuine self addressed document.

**EIDI-RULE-270.** An inbound message is acknowledged only after the platform has saved it somewhere. A processing failure leaves the message in the inbox.

**EIDI-RULE-271.** A self billed document, recognised by the type codes `389`, `527` and `261`, is created as a customer invoice in the first sale journal of the company marked as self billing, falling back to the first sale journal. Every other document is created as a vendor bill in the reception journal.

**EIDI-RULE-272.** An imported document is passed through the automatic posting rule of the accounts receivable domain.

**EIDI-RULE-273.** A business response may only be sent about a document that carries a message identifier, that is a vendor bill or a vendor credit note, for which no existing response is in state `not_serviced` and no existing response that is not in state `error` carries the code `AP` or `RE`, and whose partner publishes the response transaction.

**EIDI-RULE-274.** A rejection requires at least one reason from the reason code list. Message: `At least one reason must be given when rejecting a Peppol invoice.` The check is enforced twice, once in the wizard and once in the sending operation.

**EIDI-RULE-275.** Only the code identifiers `AB`, `AP` and `RE` may be sent. Every one of the seven codes may be received and is stored.

**EIDI-RULE-276.** A delivery state poll that answers with the code 702, meaning the request is still being processed, writes nothing and does not acknowledge the message, so that the record is polled again.

**EIDI-RULE-277.** A delivery state poll of a business response that answers with the code 207, meaning the recipient cannot receive this document type, sets the response to `not_serviced`, which permanently stops further responses about that document.

**EIDI-RULE-278.** A receiver that has no reception journal does not advertise the response transaction among its published document types, because it could not file the documents it would be answering.

**EIDI-RULE-279.** The published document types of a receiver are reconciled with the supported document types by removing what is published and not supported and then adding what is supported and not published. A failure is logged per receiver and the reconciliation is retried four hours later.

---

## Certificates and keys

**EIDI-RULE-290.** A certificate whose content is set but that could not be normalised is refused. Message: the loading error when there is one, otherwise `This certificate could not be loaded. Please provide the certificate password.`

**EIDI-RULE-291.** A certificate that could not be opened while a password was supplied stores the loading error `This certificate could not be loaded. Either the content or the password is erroneous.`

**EIDI-RULE-292.** The private key linked to a certificate must have the same public half as the certificate. Message: `The certificate and private key are not compatible.` The comparison is done in constant time.

**EIDI-RULE-293.** The public key linked to a certificate must have the same public half as the certificate. Message: `The certificate and public key are not compatible.`

**EIDI-RULE-294.** A key linked to a certificate must itself have loaded. Message: the loading error of the key.

**EIDI-RULE-295.** A certificate may only sign while it is inside its validity window. Message: the loading error when there is one, otherwise `This certificate is not valid, its validity has expired.`

**EIDI-RULE-296.** A certificate may only sign when it has a private key. Message: `No private key linked to the certificate, it is required to sign documents.`

**EIDI-RULE-297.** A key may only sign when it is a private key. Message: `Make sure to use a private key to sign documents.` A key may only verify when it is a public key. Message: `Make sure to use a public key to verify the signature of documents.` A key may only decrypt when it is a private key. Message: `A private key is required to decrypt data.`

**EIDI-RULE-298.** Only two digest algorithms are supported. Message: `Unsupported hashing algorithm '<name>'. Currently supported: sha1 and sha256.` and, for verification, `Unsupported signature algorithm '<name>'. Currently supported: sha1 and sha256.`

**EIDI-RULE-299.** Signing supports the Edwards curve family, the elliptic curve family and the ordinary factoring family. Message: `Unsupported asymmetric cryptography algorithm '%s'. Currently supported for signature: ED25519, EC and RSA.` Verification supports the elliptic curve and ordinary factoring families. Message: `Unsupported asymmetric cryptography algorithm '%s'. Currently supported for signature: EC and RSA.` Decryption supports only the ordinary factoring family. Message: `Unsupported asymmetric cryptography algorithm '%s'. Currently supported for decryption: RSA.` Public number extraction supports the elliptic curve and ordinary factoring families. Message: `Unsupported asymmetric cryptography algorithm '%s'. Currently supported: EC, RSA.`

**EIDI-RULE-300.** A generated factoring key must use the public exponent sixty five thousand five hundred and thirty seven, or three, which is also accepted. Message: `The public exponent should be 65537 (or 3 for legacy purposes).` Its size must be at least five hundred and twelve. Message: `The key size should be at least 512 bytes.`

**EIDI-RULE-301.** A generated elliptic curve key must use the two hundred and fifty six bit prime curve. Message: `Unsupported curve algorithm '<name>'. Currently supported: SECP256R1.`

**EIDI-RULE-302.** A key that could not be loaded stores the error `This key could not be loaded. Either its content or its password is erroneous.` A key whose loader failed only because no password was supplied for an encrypted key stores no error, so that a half filled form does not show a red message.

**EIDI-RULE-303.** Issuing certificates discovered inside an upload are created archived, so that they appear in searches that include archived records but never in ordinary lists.

**EIDI-RULE-304.** An issuing certificate is never created twice for the same pair of serial number and subject common name, including inside one chain.

**EIDI-RULE-305.** The issuer of a certificate is resolved by preferring, among the candidates with the right subject common name, the one whose validity end is the furthest in the future, and by requiring either a cryptographic verification of the signature or a match between the authority key identifier of the certificate and the subject key identifier of the candidate. A self signed certificate is never its own issuer.

**EIDI-RULE-306.** The company of the private key, of the public key and of the issuing certificate of a certificate must equal the company of that certificate.

---

## Rounding, currency and company consistency

**EIDI-RULE-310.** Every monetary amount written in an exported file is produced by the tax engine of the [taxes](../taxes/calculations.md) domain. This domain only chooses the grouping keys, the sign conventions and the presentation precision.

**EIDI-RULE-311.** An amount presented in a file is formatted with a minimum number of decimal places and, optionally, a maximum. The value is first rounded to the maximum when one is given, or to the minimum otherwise, and then rendered with as many decimal places as needed between the minimum and the maximum, dropping trailing zeros beyond the minimum.

**EIDI-RULE-312.** Monetary amounts of the universal business language family are presented with the number of decimal places of the currency as the minimum and no maximum, except the unit price, which uses a minimum of one and a maximum of ten.

**EIDI-RULE-313.** Monetary amounts of the cross industry invoice family are presented with a minimum of two and a maximum of two, except the line and document allowance and charge amounts, which use a minimum of two and a maximum equal to the number of decimal places of the currency.

**EIDI-RULE-314.** The universal business language family, before building any node, rounds the raw totals excluding tax, the raw gross totals excluding tax and the raw discounts to six digits, in both the document currency and the company currency. The cross industry invoice family does the same in the document currency only.

**EIDI-RULE-315.** When the document currency differs from the company currency, the universal business language family writes a second tax total in the company currency and names that currency in the tax currency code element, except in the German, Netherlands, Singapore and Australia and New Zealand profiles, which suppress both.

**EIDI-RULE-316.** The tax currency code is only written when it differs from the document currency code.

**EIDI-RULE-317.** Every record created by this domain belongs to exactly one company: the Electronic Document through its accounting document, the credential, the certificate and the key directly.

**EIDI-RULE-318.** A batch of Electronic Document records processed together belongs to one company, because the company is part of the batching key.

**EIDI-RULE-319.** A business response and the document it refers to belong to the same company, because the poll and the inbox always run in the context of one credential and therefore of one company.

---

## Permissions and visibility

**EIDI-RULE-330.** Every internal user may read Electronic Document Format and Electronic Document. Only members of the invoicing group may create, write or delete them.

**EIDI-RULE-331.** Only system administrators may create, write or delete a credential. Members of the invoicing group may read credentials.

**EIDI-RULE-332.** Only system administrators may read, create, write or delete a Digital Certificate or a Digital Key.

**EIDI-RULE-333.** Members of the invoicing group may read, create, write and delete the registration wizard, the configuration wizard, the service line, the rejection wizard, the clarification codes and the business responses.

**EIDI-RULE-334.** A credential is visible only when its company is the active company or one of its ancestors.

**EIDI-RULE-335.** A certificate and a key are visible when they have no company, or when their company is the active company or one of its ancestors.

**EIDI-RULE-336.** The payload attachment of an Electronic Document is readable only by system administrators. Users reach it through the accounting document, never directly.

**EIDI-RULE-337.** The rotating token of a credential and the migration key of a company are readable only by system administrators.

**EIDI-RULE-338.** The menu entry that lists credentials is visible only to users who have switched the technical features on.

**EIDI-RULE-339.** The participant endpoint and the participant scheme are writable from the customer portal, because a customer must be able to declare how it wants to be invoiced.
