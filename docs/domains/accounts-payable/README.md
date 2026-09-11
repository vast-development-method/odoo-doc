# Accounts Payable

## Scope

This domain specifies everything the accounting system does with **documents owed to suppliers**: vendor bills, vendor credit notes (refunds), purchase receipts, and the debit notes raised against them. It covers how such a document is created (by hand, by uploading a file, by electronic mail sent to a journal mailbox, by a decoder reading a structured or scanned file, or by a purchasing document), how its lines take their defaults from products, product categories and past behaviour, how the supplier's payable account and bank account are chosen, how payment terms turn one total into one or more due amounts and due dates, how the system warns that the same bill may already have been captured, how the document is reviewed, posted, reversed and cancelled, what journal items each transition produces, how a payment made by printed cheque is numbered, worded, printed, voided and reprinted, and how every posted line feeds the invoice analysis report.

Everything in this folder is derived from the behaviour of the following capability packages: the core **Accounting** package (the bill-specific parts of the journal entry and journal item entities, the business document import mixin, the invoice analysis report, the mass posting and review wizard, the autopost-bills wizard, payment terms, journals and their reception mailbox), the **Check Printing Base** package (cheque payment method, per-journal cheque number sequences, cheque layouts, the amount-in-words rendering, the stub, voiding and renumbering), the **Debit Notes** package (raising an additional charge linked to an existing bill), and the **Intercompany Payment Clearing** package (settling a bill of one company paid by another company of the same group).

The domain does **not** re-specify: the general posting mechanics, numbering grammar, lock dates, hash chain and reconciliation core (see `../general-ledger/`); the customer-facing mirror image of the same entity (see `../accounts-receivable/`); payments, the payment register wizard, bank statements and the reconciliation models (see `../payments-and-bank-reconciliation/`); tax computation and fiscal positions (see `../taxes/`); currency rates and exchange differences (see `../multi-currency/`); analytic distribution (see `../analytic-accounting/`); the purchase order and its receipt and billing policies (see `../purchasing/`); the structured electronic document formats and the network that carries them (see `../electronic-invoicing-and-document-exchange/`).

Where a behaviour is shared with customer invoicing — the dynamic line synchronization, the payment term line generation, cash rounding — this folder states the payable-side consequences and points at `../accounts-receivable/` for the shared algorithm. The **invoice analysis report**, however, is specified here **in full**, column by column, because it serves receivables and payables from a single definition.

## Capabilities covered

| Capability | Where specified |
|---|---|
| Vendor bill, vendor credit note and purchase receipt as document kinds, their direction, their sign and their reversal partner | `entities.md`, `state-machines.md` |
| Vendor reference (`ref`), its uniqueness expectations, its use as the payment term line label, and its index | `entities.md`, `business-rules.md` |
| Bill date, accounting date, delivery date, taxable supply date, due date and their interdependence | `entities.md`, `calculations.md`, `business-rules.md` |
| Supplier bank account selection, the archived-account and untrusted-account rules | `entities.md`, `business-rules.md`, `workflows.md` |
| Payable account selection for the term line: previous line, partner property, company partner property, first active payable account, fiscal position remapping | `calculations.md`, `business-rules.md` |
| The reviewed flag (`checked`), who may set it, and the "to check" filter | `entities.md`, `state-machines.md`, `workflows.md` |
| Automatic posting of bills per vendor, the three vendor policies, the learning wizard and its threshold | `workflows.md`, `configuration.md`, `business-rules.md` |
| Duplicate bill detection: the full matching predicate, exact versus probable duplicates, the two warning levels and their exact wording, the delete action, and the suppression of automatic posting | `calculations.md`, `business-rules.md`, `workflows.md`, `interfaces.md` |
| Bill upload and the electronic decoding contract: file grouping, embedded file extraction, decoder priority, roll-back on failure, the exact chatter messages | `workflows.md`, `interfaces.md`, `business-rules.md` |
| The incoming mailbox that creates bills: routing, the no-attachment bounce, sender identification, attachment splitting into several bills | `workflows.md`, `interfaces.md`, `configuration.md` |
| Line defaulting from products and product categories: description, unit, unit price, taxes, expense account, most-frequent-account fallback, neighbour-line fallback, journal default account | `calculations.md`, `workflows.md` |
| Partial deductibility of purchase lines and the private-part journal items | `entities.md`, `accounting-effects.md`, `calculations.md` |
| Payment terms applied to bills: percent and fixed lines, the four delay types, the balance-line rule, early payment discount in three modes, the due date algorithm | `calculations.md`, `entities.md` |
| Reversal of a bill: full reversal, cancelling reversal, partial reversal, the reversal of a partly paid bill, the reconciliation performed | `workflows.md`, `accounting-effects.md`, `acceptance-criteria.md` |
| Debit notes raised from a bill or from a vendor credit note, their dedicated sequence prefix | `entities.md`, `workflows.md`, `configuration.md` |
| Cheque printing in full: the cheque payment method, manual versus pre-printed numbering, the per-journal sequence, the number uniqueness constraint, the amount in words algorithm, the fill characters, the stub and its paging, layouts and margins, printing, voiding, unmarking as sent, renumbering | `calculations.md`, `workflows.md`, `entities.md`, `configuration.md`, `interfaces.md` |
| Purchase receipts: what distinguishes them from bills, their fiscal position default, their sequence | `entities.md`, `state-machines.md` |
| The purchase order matching hook and the origin field parsing | `workflows.md`, `interfaces.md` |
| Mass posting and the confirmation wizard: future dates, hashing, abnormal amount and date acknowledgements | `workflows.md`, `interfaces.md`, `business-rules.md` |
| Cancel and reset-to-draft rules, the audit trail protection, the exchange-difference and cash-basis prohibitions, lock dates | `state-machines.md`, `business-rules.md` |
| Intercompany clearing when another company of the group pays the bill | `accounting-effects.md`, `workflows.md`, `configuration.md` |
| The invoice analysis report entity with every column and its computation | `entities.md`, `calculations.md` |
| Every journal item produced by every transition, with account, side, amount and currency | `accounting-effects.md` |

## Entities

| Entity | Transport name | Storage name | Purpose |
|---|---|---|---|
| Journal Entry | `account.move` | `account_move` | The document itself; a vendor bill, vendor credit note or purchase receipt is a journal entry whose type is one of the purchase types |
| Journal Item | `account.move.line` | `account_move_line` | One line of the document: an expense line, a tax line, a payable term line, a rounding line or a non-deductible line |
| Journal | `account.journal` | `account_journal` | The purchase journal that numbers the document, and the bank journal that numbers cheques |
| Payment Term | `account.payment.term` | `account_payment_term` | The rule set turning a total into instalments with due dates |
| Payment Term Line | `account.payment.term.line` | `account_payment_term_line` | One instalment rule: a percentage or fixed amount and a delay |
| Payment | `account.payment` | `account_payment` | The outgoing payment; carries the cheque number and the amount in words |
| Payment Method | `account.payment.method` | `account_payment_method` | The `check_printing` method that makes a payment a cheque |
| Invoice Analysis Report | `account.invoice.report` | none — a database view named `account_invoice_report` | One row per product line of every invoice, bill, credit note and receipt, with signed quantities and amounts converted to the reporting currency |
| Business Document Import Mixin | `account.document.import.mixin` | none — an abstract mixin | The contract for turning attachments into documents and decoding them |
| Confirm Entries Wizard | `validate.account.move` | `validate_account_move` | The mass posting dialogue with its force-post, force-hash and acknowledge-abnormal switches |
| Autopost Bills Wizard | `account.autopost.bills.wizard` | `account_autopost_bills_wizard` | Offers to switch a vendor to automatic bill validation |
| Add Debit Note Wizard | `account.debit.note` | `account_debit_note` | Creates a debit note from one or more posted documents |
| Print Pre-numbered Cheques Wizard | `print.prenumbered.checks` | `print_prenumbered_checks` | Asks for the number printed on the first sheet and renumbers the selected payments |
| Company | `res.company` | `res_company` | Holds the cheque layout and margins, the automatic-bill-posting switch, the quick encoding mode, the purchase receipt fiscal position and the intercompany clearing accounts |
| Partner | `res.partner` | `res_partner` | Holds the supplier payment term, the automatic posting policy, the abnormal-amount and abnormal-date acknowledgements and the supplier rank |

Entities of other domains that carry payable-specific fields specified here: Account (`account.account`), Fiscal Position (`account.fiscal.position`), Product Template (`product.template`), Product Category (`product.category`), Partner Bank Account (`res.partner.bank`), Attachment (`ir.attachment`), Sequence (`ir.sequence`), Mail Alias (`mail.alias`).

## Reading order

1. `glossary.md` — the vocabulary (bill, refund, receipt, term line, stub, void, deductibility, exact duplicate, decoder).
2. `entities.md` — the data model, field by field.
3. `state-machines.md` — the three state fields (document status, payment status, reviewed) and their transitions.
4. `calculations.md` — every formula: due dates, instalment amounts, duplicate matching, amount in words, the analysis report columns.
5. `accounting-effects.md` — every journal item produced by every transition.
6. `workflows.md` — the operational sequences end to end.
7. `business-rules.md` — validations, messages, permissions, locking.
8. `configuration.md` — settings, sequences, groups, access rights, scheduled jobs.
9. `interfaces.md` — navigation, views, named operations, routes, printed documents, mail templates.
10. `acceptance-criteria.md` — numbered scenarios that an implementation must satisfy.

## Dependencies on other domains

| Domain | Dependency |
|---|---|
| `../general-ledger/` | Accounts and account types, journals, posting, numbering and sequence gaps, lock dates and lock exceptions, the hash chain, reconciliation core, the audit trail |
| `../accounts-receivable/` | The shared dynamic line synchronization, the shared payment term entity, cash rounding, the shared document sending flow; the mirror-image customer document |
| `../taxes/` | Tax computation on expense lines, tax groups, fiscal position tax and account mapping, cash basis |
| `../payments-and-bank-reconciliation/` | The payment entity and its state machine, the payment register wizard, outstanding accounts, reconciliation and unreconciliation, the bank journal dashboard |
| `../multi-currency/` | Currency rounding, the bill currency rate, conversion for the analysis report, exchange differences on reconciliation |
| `../analytic-accounting/` | Analytic distribution carried by expense lines and copied to tax lines |
| `../purchasing/` | The purchase order that the bill may be matched to; the origin field; the vendor price that seeds the unit price |
| `../electronic-invoicing-and-document-exchange/` | The concrete decoders that implement the decoding contract described here |
| `../products-and-catalog/` | Product purchase description, purchase unit, supplier taxes, expense account of the product and of its category |
| `../messaging-and-activities/` | The chatter, the mail alias that receives bills, the attachment lifecycle, the bounce message |

## Behaviour not present in the source set

The following capabilities that are commonly associated with accounts payable are **not** implemented by the packages covered here and are only described as industry-standard defaults where a companion package is expected to add them: three-way matching between a purchase order, a receipt and a bill (only the matching hook and the origin parsing exist here — see `workflows.md`), payment runs and remittance advice batches, supplier statement reconciliation, cheque layouts themselves (this package defines the layout **selection** and its margins, but ships no drawable layout: the selection contains only the value `disabled` until a country package adds one), and the optical character recognition of scanned bills (only the decoder contract exists — see `workflows.md`). These points are called out where relevant.

## How to read the specifications in this folder

**Formulas.** Every formula is written in a fenced block labelled `formula`, in plain mathematics, using named quantities in words and the symbols × ÷ + − =. Rounding is always explicit: `round_to(currency, x)` rounds to the smallest representable step of that currency using the *round half away from zero* rule. Each formula is followed by an explanation of every quantity and by at least one worked numeric example.

**Algorithms.** Numbered steps in prose, with preconditions, postconditions and failure conditions stated. Where the order of evaluation matters — and in this domain it almost always does, because the dynamic lines are rebuilt in a fixed order at every save — the order is stated explicitly.

**Messages.** Error and warning texts are reproduced exactly as the system produces them, with placeholders translated into words between guillemets, for example *«the document number»*.

**Identifiers.** Storage names, transport names and selection values appear in code font because external contracts depend on them; each is accompanied on first use by its full name in words.

## The shape of a purchase document

A purchase document is a single record that carries **two layers at once**:

1. A **commercial layer** — the supplier, the supplier's own reference, the dates, the payment terms, the bank account to pay to, and a list of what was bought at what price.
2. An **accounting layer** — a balanced set of journal items.

The two layers are kept in step by a **synchronizer** that runs inside every save. It reads the commercial layer and rebuilds the dynamic journal items — tax lines, payable term lines, cash rounding lines, early payment discount lines and private-share lines — so that they always match. Understanding that synchronizer is the key to the whole domain, and the order in which it rebuilds each kind of line is given in `workflows.md` §1 step 6.

Reading the domain in the order given above therefore means: first the vocabulary, then the two layers as data, then the states they move through, then the arithmetic that derives one layer from the other, then the ledger consequences, and only then the operational sequences.

## What makes the payable side different from the receivable side

The two sides share one entity and most of one algorithm. The differences that matter, all specified here, are:

| Aspect | Receivable | Payable |
|---|---|---|
| Direction sign | −1 for an invoice | **+1** for a bill |
| Term line account type | receivable | **payable** |
| Term line side | debit | **credit** |
| Document number | issued by the company | issued by the company, but the **supplier's own number** is recorded separately as the vendor reference |
| Bill date at posting | filled with today when missing | **required**; posting refuses without it |
| Bank account | the company's, and it must be trusted | the **supplier's**, and it must not be archived |
| Line description | the product's sale description | the product's **purchase description** |
| Line unit | the product's reference unit | the **supplier's purchase unit** when one is recorded |
| Line price | the product's sale price | the product's **purchase price** |
| Line taxes | the product's sale taxes | the product's **supplier taxes** |
| Line account | the income account | the **expense account** |
| Duplicate detection | same partner, same total, same date | **same reference within a calendar year**, or same partner, total and date |
| Deductibility | not allowed | allowed, and generates private-share lines |
| Capture | typed or generated from a sales order | typed, **uploaded, mailed in or decoded** |
| Automatic posting | not offered | offered per vendor, with a learning wizard |
| Abnormal detection | not offered | offered on amount and on billing frequency |
| Attachments on reset to draft | detached, so the printable document can be regenerated | **kept**, because the file belongs to the supplier |
| Payment | received | issued, possibly as a **printed cheque** |
| Analysis report sign | positive | **negative** |
