# Accounts Receivable

This domain specifies everything the platform does between the moment a company decides to
charge a customer and the moment the resulting claim disappears from the books: the customer
invoice, the customer credit note, the sales receipt, the customer debit note, the payment
terms that split the claim into instalments, the early payment discount, the cash rounding of
the total, the numbering of the documents, the delivery of the documents to the customer, the
customer portal where the document is read and paid, the quick response code payment data
printed on the document, and the customer credit limit that warns the salesperson before the
claim is even created.

The domain is written so that an engineering team can rebuild it in any language with
behavioral equivalence. Every formula carries its rounding rule, every state transition carries
the journal items it produces, and every validation carries the exact message text the system
shows.

## Scope

**Covered here**

- Customer invoice, customer credit note and sales receipt as a single entity with a document
  type discriminator, and the complete field set of that entity in its receivable role.
- The dynamic line synchronisation: how product lines, discount allocation lines, cash rounding
  lines, tax lines, early payment discount lines and payment term lines are kept consistent with
  one another on every write, in a fixed order, with a fixed key per line family.
- Payment terms: percent lines and fixed-amount lines, the four delay types, the balance rule on
  the last line, the amount distribution algorithm in two currencies, the due date algorithm, and
  the early payment discount in its three computation modes.
- Cash rounding: both strategies (add a rounding line, modify the biggest tax amount) and the
  three rounding methods, including the interaction with payment terms and with the early payment
  discount.
- Numbering: how a customer invoice receives its number, how credit notes may receive a separate
  sequence, how the number is reserved and how gaps are detected.
- Payment state: the six payment statuses, their computation from reconciliation data, the
  in-payment versus paid distinction, and the blocked status.
- Amount totals in every variant: untaxed, tax, total, residual, and each of their signed and
  document-currency-signed counterparts.
- Invoice sending: the send-and-print flow, its checkboxes, its asynchronous mode, the generated
  document file, the mail template and the sending log.
- The printable customer invoice, section by section.
- The customer portal: the document list, the document page, the access token, the download links
  and the online payment entry point.
- Debit notes: the wizard, the copy modes and the resulting document.
- Reversal: the credit note wizard, the three reversal methods and the reconciliation they cause.
- Quick response code payment data: the two shipped generators and every field of their payload.
- Partner credit limit, the receivable and payable aggregates on the partner, and the warning text.
- Invoice line defaulting from a product, including fiscal position account and tax mapping.
- Duplicate detection on customer documents.

**Not covered here** (see the referenced domains)

- The chart of accounts, the journals, the generic journal entry mechanics, the hash chain, the
  lock dates, reconciliation internals and the reversal wizard's generic behavior:
  [`../general-ledger/README.md`](../general-ledger/README.md).
- Vendor bills, vendor credit notes, purchase receipts and check printing:
  [`../accounts-payable/README.md`](../accounts-payable/README.md).
- Payments, the register payment wizard, bank statements and the reconciliation models:
  [`../payments-and-bank-reconciliation/README.md`](../payments-and-bank-reconciliation/README.md).
- Tax definition, tax computation and the tax engine used by the line synchronisation:
  [`../taxes/README.md`](../taxes/README.md).
- Currency rates, rounding arithmetic and exchange differences:
  [`../multi-currency/README.md`](../multi-currency/README.md).
- Analytic distribution on lines: [`../analytic-accounting/README.md`](../analytic-accounting/README.md).
- Structured electronic invoice formats and the document exchange network:
  [`../electronic-invoicing-and-document-exchange/README.md`](../electronic-invoicing-and-document-exchange/README.md).
- Sales orders and the preparation of invoices from them:
  [`../sales/README.md`](../sales/README.md).
- Payment providers and online payment transactions:
  [`../payment-providers/README.md`](../payment-providers/README.md).

## Entities

| Entity | Transport name | Table | One-line purpose |
| --- | --- | --- | --- |
| Journal Entry (in its invoice role) | `account.move` | `account_move` | The customer invoice, credit note or sales receipt, together with the accounting entry it is. |
| Journal Item (in its invoice-line role) | `account.move.line` | `account_move_line` | Every line of the document: product lines, tax lines, discount allocation lines, cash rounding lines, early payment discount lines, payment term lines, sections and notes. |
| Payment Term | `account.payment.term` | `account_payment_term` | A named instalment plan with an optional early payment discount. |
| Payment Term Line | `account.payment.term.line` | `account_payment_term_line` | One instalment of a payment term: how much and when. |
| Cash Rounding Method | `account.cash.rounding` | `account_cash_rounding` | A rule that rounds the document total to the smallest circulating coin. |
| Partner (in its credit role) | `res.partner` | `res_partner` | The customer, carrying the receivable account, the payment term, the credit limit and the receivable aggregates. |
| Partner Bank Account | `res.partner.bank` | `res_partner_bank` | The account into which the customer is asked to pay; source of the quick response code payload. |
| Invoice Send Wizard | `account.move.send.wizard` | not stored | Single-document sending: which channels, which template, which attachments. |
| Invoice Batch Send Wizard | `account.move.send.batch.wizard` | not stored | Multi-document sending with the same choices. |
| Invoice Reversal Wizard | `account.move.reversal` | not stored | Creates credit notes from posted customer invoices in one of three methods. |
| Debit Note Wizard | `account.debit.note` | not stored | Creates a customer debit note from a posted customer invoice. |
| Invoice Analysis Line | `account.invoice.report` | database view `account_invoice_report` | Read-only reporting projection over posted invoice lines. |
| Payment Provider (receivable hook) | `payment.provider` | `payment_provider` | Extended here with one field only: the bank journal in which a successful online payment is posted. The provider itself belongs to [`../payment-providers/`](../payment-providers/README.md); the switch that allows portal payment is not a field on it but the system parameter documented in [`configuration.md`](configuration.md) section 3. |
| Payment Link Wizard (receivable hook) | `payment.link.wizard` | not stored | Extended here with the amount due, the open instalments and their preview, and the early payment discount notice; it composes the portal payment address and is the producer of the portal-link quick response code. |

## Reading order

1. [`glossary.md`](glossary.md) — read the vocabulary first; the rest of the domain uses it
   without re-explaining it.
2. [`entities.md`](entities.md) — the field-by-field definition of every entity, ending with the
   relations this domain has with the other domains.
3. [`state-machines.md`](state-machines.md) — the document status, the payment status and the
   sending status, with their transitions.
4. [`calculations.md`](calculations.md) — the dynamic line synchronisation, the payment term
   distribution, the early payment discount, the cash rounding, the totals and the worked
   examples. This is the heart of the domain.
5. [`accounting-effects.md`](accounting-effects.md) — the journal items produced by every event.
6. [`workflows.md`](workflows.md) — the end-to-end operational sequences.
7. [`business-rules.md`](business-rules.md) — every validation, constraint and message.
8. [`configuration.md`](configuration.md) — settings, sequences, groups, access rights, record
   rules and scheduled jobs.
9. [`interfaces.md`](interfaces.md) — menus, views, named operations, routes, reports, templates.
10. [`acceptance-criteria.md`](acceptance-criteria.md) — numbered scenarios with concrete numbers.

## Dependencies on other domains

| Depends on | What is consumed |
| --- | --- |
| General Ledger | Account, Journal, the journal entry state machine, the sequence mixin, reconciliation, lock dates, the hash chain, the audit trail. |
| Taxes | The tax computation engine: base line preparation, tax detail rounding, tax line preparation, the repartition lines and the tax grid tags. |
| Multi-currency | Currency rounding, the conversion rate lookup, and the exchange difference entries created when a foreign-currency invoice is reconciled. |
| Products and catalog | The product from which a line defaults its description, unit, price, taxes and income account. |
| Units of measure | The unit on a line and the price conversion when the line unit differs from the product's reference unit. |
| Analytic accounting | The analytic distribution carried by product lines and propagated to discount allocation lines. |
| Payments and bank reconciliation | The payments that reconcile against the receivable lines and therefore drive the payment status. |
| Payment providers | The online payment of an invoice from the portal. |
| Messaging and activities | The message thread on the document, the mail template used to send it, the follower list and the scheduled activities. |
| Website and storefront | The portal layout in which the customer document page is rendered. |

## What this domain contributes to other domains

- It creates the receivable journal items that the ledger reconciles and that the aged receivable
  report ages.
- It publishes the payment status that the sales domain reads to decide whether an order is paid.
- It publishes the receivable aggregates on the partner that the credit limit warning uses and
  that the sales domain shows on a quotation.
- It produces the source document from which the electronic invoicing domain builds structured
  documents.
