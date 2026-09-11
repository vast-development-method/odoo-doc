# Accounts Payable — Glossary

Every term used in this folder, defined in full. Reproduced identifiers appear in code font with their full name in words.

---

**Abnormal amount warning** — A warning text computed for a draft purchase document whose total falls outside the interval *mean plus or minus twice the sample standard deviation* of the totals of the vendor's ten to thirty most recent comparable posted documents. It also prevents automatic posting. See `calculations.md` §12.3.

**Abnormal date warning** — A warning text computed for a draft purchase document that arrives sooner than the vendor's historical billing rhythm predicts. See `calculations.md` §12.2.

**Accounting date** (`date`) — The date at which a journal entry hits the ledger, and therefore the period it belongs to and the numbering series it consumes. Distinct from the bill date. Derived from the bill date by the algorithm of `calculations.md` §2.3.

**Alias** — The local part of an electronic mail address attached to a journal, at which supplier documents may be received. Only sale and purchase journals have one.

**Allow outgoing payment** — A flag on a bank account meaning that the company considers the account trusted enough to send money to it. Also called *trusted*. It blocks the posting of an inbound document whose recipient bank account is not trusted.

**Amount in words** (`check_amount_in_words`) — The textual rendering of a payment amount in the user's language, title-cased, made of the integral part, the currency's unit label, the word *and*, the fractional part and the currency's subunit label. Printed on a cheque, padded with asterisks to two hundred characters. See `calculations.md` §7.

**Analysis report** — See *Invoice analysis report*.

**Attachment** — A stored file linked to a record. Only certain media types stay visible on a document; the rest are detached. See `entities.md` §9.3.

**Audit trail** — The set of notification messages recording every tracked field change on a document. Under the **restrictive** audit trail those messages cannot be deleted, and therefore neither can a document that has ever been posted.

**Automatic posting mode** (`auto_post`) — A scheduling state on a journal entry: `no`, `at_date`, `monthly`, `quarterly`, `yearly`. A scheduled job posts due entries; a periodic mode also copies the next occurrence at posting.

**Automatic bill validation** — The mechanism that posts a vendor bill without human review when the company's switch is on and the vendor's policy is *Always*. See `workflows.md` §8.2.

**Balance** — The signed amount of a journal item in the **company** currency. Positive is a debit, negative is a credit.

**Balance line** — The **last** payment term line, in the term's own line order. Whatever its declared type, it takes the whole remaining amount, so that the instalments sum exactly to the document total.

**Bank account** (`res.partner.bank`) — A partner's account number. On a purchase document the supplier's account is the one the bill will be paid to.

**Bill** — A vendor bill: a journal entry whose type is `in_invoice`. The company owes the amount.

**Bill date** (`invoice_date`) — The date the supplier issued the document. The reference date for payment terms, duplicate detection and the currency rate. Required to post a purchase document.

**Bill upload** — The flow that turns dropped files into draft bills, one per group of files. See `workflows.md` §2.

**Cancelled** (`cancel`) — The end state of a document that has been voided. Its journal items remain but are excluded from the ledger; its number is kept so that the numbering series has no hole.

**Cash rounding** — A rule adjusting a document total to the smallest coin still in circulation, either by adding a rounding line or by modifying the largest tax. See `calculations.md` §8.

**Cheque** — An outgoing payment whose payment method code is `check_printing`. It carries a number, an amount in words and a stub.

**Cheque layout** — The printable-document definition that draws a cheque on a given stationery. Selected on the company and optionally overridden per journal. The base package ships only the value `disabled`.

**Cheque number** (`check_number`) — The digits identifying one cheque. Unique per bank journal among **posted** payments, compared as integers. Drawn from the journal's cheque sequence when the stationery is blank, recorded afterwards when it is pre-printed.

**Cheque sequence** — A per-journal, no-gap, padding-five numbering sequence used for cheque numbers.

**Commercial partner** — The top of a contact's hierarchy: the legal entity. Duplicate detection compares commercial partners, and posting forces every accountable line's partner to the document's commercial partner.

**Company currency** — The currency of the company that owns the document. Balances are expressed in it.

**Credit** — A negative balance on a journal item. On a vendor bill the payable term line is a credit.

**Debit** — A positive balance on a journal item. On a vendor bill the expense and tax lines are debits.

**Debit note** — An additional charge raised against an existing posted document and linked to it. Its type is the source's type, except that a credit note yields an invoice. Optionally numbered from a dedicated series prefixed with `D`.

**Decoder** — An operation that reads an attachment and writes its content onto a document. Declared with a priority; only the highest-priority decoder runs. It answers nothing on success and a refusal reason otherwise. See `entities.md` §9.4.

**Deductibility** (`deductible_amount`) — The percentage of a purchase line that is a business expense, default 100. Values below 100 generate the private-share lines. Only valid on purchase documents.

**Direction sign** (`direction_sign`) — The multiplier converting a positive commercial amount into a signed accounting balance: +1 for a miscellaneous entry and for every outbound document (vendor bill, customer credit note, purchase receipt), −1 otherwise.

**Display type** — The classification of a journal item: `product`, `tax`, `payment_term`, `rounding`, `discount`, `epd`, `cogs`, the three non-deductible kinds, and the three presentation kinds (`line_section`, `line_subsection`, `line_note`).

**Document currency** — The currency written on the document. The amount in currency of every journal item is expressed in it.

**Document rate** (`invoice_currency_rate`) — The number of document-currency units per company-currency unit, as used by this document. Must be strictly positive when the two currencies differ.

**Due date** (`invoice_date_due`) — The latest maturity date among the document's payable term lines. Writable; overriding it collapses the instalments to one.

**Duplicate, exact** — A detected duplicate that matches the examined purchase document (a bill or a vendor credit note, not a receipt) on the vendor reference, a compatible type, the **partner**, the bill date and the total. Produces the red warning.

**Duplicate, probable** — Any detected duplicate that is not exact. Produces the amber warning while the document is draft.

**Duplicate detection** — The continuous comparison of the document being edited against draft and posted documents of the same company. See `calculations.md` §4.

**Early payment discount** — A reduction granted when a document is paid within a stated number of days of the bill date. Computed in one of three tax-reduction modes: `included` (*On early payment*), `excluded` (*Never*), `mixed` (*Always (upon invoice)*).

**Expense line** — A journal item of display type `product` on a purchase document: what was bought, at what price, with which taxes and to which expense account.

**File-data record** — The intermediate structure the import framework works on: name, bytes, media type, origin attachment, stored attachment, parsed tree, format label and decoder information. See `entities.md` §9.2.

**Fiscal position** — A rule set substituting taxes and accounts for a given counterpart or territory. On a purchase receipt the company's default purchase receipt fiscal position wins outright.

**Grouping, by origin attachment** — The default rule for turning uploaded files into documents: an extracted file joins its container's group; every other file forms its own group.

**Grouping, into groups of mixed types** — The rule used for files arriving by electronic mail: files of the same format label go to different groups, files of different format labels to the same group, ties broken by filename similarity. See `entities.md` §9.6.

**Hash chain** — The inalterability mechanism that seals posted entries of a journal. A hashed entry can never be reset to draft and its key fields can never be edited.

**Imported line** (`is_imported`) — A journal item captured automatically by an import or a decoder. Its unit price and taxes are never silently recomputed, and it may keep an archived account.

**Inbound document** — A document on which money comes in: `out_invoice`, `in_refund`, and, when receipts are included, `out_receipt`.

**Instalment** — One *(due date, amount)* pair produced by a payment term, materialised as one payable term line.

**Intercompany clearing** — The pair of journal entries created when a document of one company is paid by another company of the same group, so that both are left with mirrored intercompany balances. See `accounting-effects.md` §8.

**Invoice analysis report** (`account.invoice.report`) — A read-only database view holding one row per product line of every invoice, bill, credit note and receipt, with signed quantities and amounts and a currency conversion to the active company's currency. Fully specified in `entities.md` §3.

**Journal, purchase** — A journal of type `purchase`. It numbers bills, vendor credit notes and purchase receipts, holds the default expense account and the private share account, and may carry a reception mail alias.

**Journal entry** (`account.move`) — The single entity representing both an accounting entry and a commercial document.

**Journal item** (`account.move.line`) — One line of a journal entry.

**Lock date** — A date before which entries may not be added or modified. Several kinds exist (fiscal year, sale, purchase, tax, hard and soft). Posting moves an offending accounting date forward rather than refusing.

**Manual numbering** (`check_manual_sequencing`) — A journal setting meaning that the cheque stationery is blank and the system assigns the numbers.

**Maturity date** (`date_maturity`) — The due date carried by one payable term line.

**Most frequent account** — The account most often used with a given partner over the last two years, restricted by document direction, used as a line default and by quick encoding.

**Non-deductible tax line** — The journal item carrying the input tax on the private share, booked to the journal's private share account.

**Number** (`name`) — The document's identifier within its journal's numbering series. The placeholder `/` while the document has never been posted.

**Outbound document** — A document on which money goes out: `in_invoice`, `out_refund`, and, when receipts are included, `in_receipt`.

**Outstanding payments account** — The bridging account on which an outgoing payment sits between its posting and its appearance on a bank statement.

**Payable account** — An account whose type is *payable*. Exactly the payable term lines of a purchase document sit on one.

**Payable term line** — A journal item of display type `payment_term` on a purchase document: one instalment of the amount owed, on the payable account, with a maturity date.

**Payment method line** — The instantiation of a payment method on a journal. The cheque method is instantiated on bank journals.

**Payment reference** — The reference the supplier asked to be quoted when paying. When set, it becomes the label of the payable term line, alone or joined to the vendor reference.

**Payment status** (`payment_state`) — How much of a document has been settled: `not_paid`, `partial`, `in_payment`, `paid`, `reversed`, `blocked`, `invoicing_legacy`.

**Payment term** (`account.payment.term`) — The rule set turning one total into instalments with due dates, optionally with a single early payment discount.

**Posted** (`posted`) — The state of a document that has a definitive number and whose journal items are part of the ledger.

**Presentation line** — A section, subsection or note line. It carries no account and no amount and is not part of the entry in the accounting sense.

**Private share** — The non-deductible fraction of a mixed expense, moved out of the expense account and out of the deductible tax account into the journal's private share account. See `calculations.md` §10.

**Private share account** (`non_deductible_account_id`) — The journal account receiving the private share. Falls back to the journal's default account.

**Purchase document** — Any of the three types `in_invoice`, `in_refund`, `in_receipt`.

**Purchase receipt** (`in_receipt`) — A purchase document used where no formal supplier invoice exists. Distinguished from a bill by its fiscal position default and by its exclusion from the exact-duplicate predicate.

**Quick encoding** — A capture mode in which the user types only the tax-inclusive total and the system proposes one matching line. Enabled per company for sale documents, purchase documents, or both.

**Recipient bank account** (`partner_bank_id`) — The bank account the document will be paid to. On a purchase document it belongs to the supplier. Selected by the ranking of `calculations.md` §3.

**Reference date** — The date from which a payment term computes its due dates: the bill date, falling back to the accounting date, falling back to today.

**Residual** (`amount_residual`) — What is left unsettled on a document, in the document currency, positive while a bill is unpaid.

**Reversal** — A mirror-type document created from a posted one. A plain reversal is independent; a cancelling reversal is posted and reconciled against the original; a modifying reversal adds a draft replacement.

**Reviewed** (`checked`) — A two-valued flag saying whether someone with review authority has confirmed a posted document. The negation of *to check*.

**Roll-back-able transaction** — The guard around a decoder: commit before, run, commit after, roll back on failure. See `entities.md` §9.7.

**Rounding line** — The journal item carrying a cash rounding adjustment.

**Self billing** (`is_self_billing`) — A journal setting meaning that the company issues the supplier's documents on their behalf. Numbering then uses a separate series per partner and the self-billing mail templates are used.

**Sent** (`is_sent`) — On a payment, the flag meaning the cheque has been printed. Clearing it puts the cheque back in the print queue with the same number.

**Sequence gap** — A hole in a journal's numbering series, caused by deleting or leaving draft a document that had consumed a number. Flagged on the following document and surfaced on the journal dashboard.

**Stub** — The detachable summary of the documents a cheque pays. Nine lines per page; grouped into *Bills* and *Refunds*; cropped or paged depending on the company setting.

**Supplier rank** — A counter on a partner, incremented by one per posted purchase document, used to order vendor pickers.

**Tax line** — A journal item generated by the tax engine from the expense lines, booked to the account named by the tax's distribution line (the invoice distribution for a bill, the refund distribution for a vendor credit note).

**To check** — The informal name of the queue of documents whose reviewed flag is false and whose status is not draft.

**Trusted bank account** — See *Allow outgoing payment*.

**Unit factor** — The ratio between the factor of a line's unit and the factor of the product template's reference unit, used by the analysis report to restate quantities.

**Vendor** — The partner of a purchase document.

**Vendor credit note** (`in_refund`) — A purchase document reversing all or part of a bill. The supplier owes the amount.

**Vendor payment term** (`property_supplier_payment_term_id`) — The partner property that becomes the default payment term of a purchase document.

**Vendor reference** (`ref`) — The number the supplier printed on their own document. The primary key of duplicate detection and, absent a payment reference, the label of the payable term line.

**Void** — The operation that cancels an issued cheque: the payment is reset to draft and then cancelled; the number is kept for audit.
