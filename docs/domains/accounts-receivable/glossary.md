# Glossary of the Accounts Receivable domain

Every term this domain uses, defined in full. Reproduced identifiers — storage names, transport
names and selection values that an external contract depends on — are given in code font with their
full name in words.

---

## A

**Accountable line.** A journal item that carries an account and an amount, as opposed to a section,
a subsection or a note. Only accountable lines participate in the balance invariant and in the
ledger.

**Accounting date** (`date`). The date at which the document enters the ledger and therefore the
period it belongs to. For a customer document it follows the document date, pushed forward past any
violated lock date. Distinct from the document date, which is the commercial date printed on the
document.

**Accounting sign.** The sign convention of the ledger: a debit is positive, a credit is negative. A
customer invoice's receivable line therefore carries a positive balance and its revenue lines
negative balances.

**Allocation account.** See *discount allocation account*.

**Amount due** (`amount_residual`). The customer-facing amount still owed on a document: the sum of
the residuals of its payment term lines, re-signed so that a customer invoice shows a positive
figure.

**Amount in currency** (`amount_currency`). A journal item's amount expressed in the line's own
currency, carrying the accounting sign. Equal to the balance when the line currency is the company
currency.

**Analytic distribution** (`analytic_distribution`). A map from analytic account to percentage,
carried by a product line, saying how the line's cost or revenue is attributed to internal
dimensions. See [`../analytic-accounting/README.md`](../analytic-accounting/README.md).

**Auto-post** (`auto_post`). A per-document setting that makes the document post itself on its
accounting date, and optionally copy itself forward every month, quarter or year.

---

## B

**Balance** (`balance`). A journal item's signed amount in the company currency. Positive is a debit,
negative is a credit.

**Balance line rule.** The rule that the **last** payment term line of a payment term always takes
the whole remaining residual, whatever its declared kind or amount. It is what makes the instalments
add up to the document total exactly, in both currencies, with no rounding drift.

**Balance invariant.** The rule that the sum of the balances of a document's lines, rounded to the
company currency, is zero. Enforced around every write.

**Base line.** In the tax engine's vocabulary, a line on which tax is computed: a product line, an
early payment discount line, a cash rounding line produced by the "add a rounding line" strategy, or
a non-deductible base line.

**Bank partner** (`bank_partner_id`). The party whose bank account is shown on the document: the
company's own partner for an inbound document (the money comes to us), the customer's commercial
entity for an outbound one.

**Blocked.** A payment status set by hand to stop follow-up on a document. It cannot be set on a paid
or in-payment document and is never overwritten by the automatic computation.

---

## C

**Cash-basis tax.** A tax whose exigibility is "on payment": it becomes due to the authority only
when the customer pays, not when the invoice is issued. Reconciling such an invoice produces a
cash-basis entry.

**Cash rounding.** The adjustment of a document total to a multiple of the smallest circulating coin,
because the coins below that value have been withdrawn. See *cash rounding method*, *rounding
precision*, *rounding strategy*.

**Cash rounding difference.** The amount by which a total must move to reach the nearest allowed coin
multiple, computed as: round the amount to the currency first, then subtract it from the result of
rounding it to the coin multiple, then round the difference to the currency.

**Cash rounding method** (`account.cash.rounding`). The configuration record holding the rounding
precision, the rounding method, the strategy and the two absorbing accounts.

**Commercial entity** (`commercial_partner_id`). The invoicing party behind a contact: the topmost
company-flagged ancestor of the partner, or the partner itself. Every accountable line of a document
carries the commercial entity, not the contact.

**Company currency.** The currency in which the company keeps its books. Every balance is expressed
in it.

**Credit.** A journal item side. On a customer invoice the revenue and the tax are credits.

**Credit limit** (`credit_limit`). The maximum total credit a company is willing to extend to a
customer. Company-dependent, with a company-wide fallback. Exceeding it produces a warning, never a
block.

**Credit note.** See *customer credit note*.

**Currency rate** (`invoice_currency_rate`). The number of units of the document currency that one
unit of the company currency buys, stored on the document so that a later change of the rate table
does not restate a posted document.

**Customer credit note** (document type `out_refund`). A document that reduces what a customer owes:
the reverse of a customer invoice. Its direction sign is `+1`, so its revenue lines are debits and
its receivable line is a credit.

**Customer debit note.** A document that increases what a customer owes for an already-invoiced
transaction. It is a **customer invoice** linked to its source, not a distinct document type.

**Customer invoice** (document type `out_invoice`). The primary document of this domain: a claim on a
customer. Its direction sign is `−1`, so its revenue lines are credits and its receivable line is a
debit.

---

## D

**Days sales outstanding.** A measure of how long a customer takes to pay, computed as the customer's
total receivable divided by the lifetime tax-included invoiced amount, multiplied by the number of
days since the oldest document.

**Debit.** A journal item side. On a customer invoice the receivable line is a debit.

**Deductibility** (`deductible_amount`). The percentage of a purchase line that may be deducted. Not
used on the receivable side; a non-hundred value on a sale document is refused.

**Derived line.** A journal item produced and maintained by the dynamic line synchronisation rather
than typed by the user: a tax line, a discount allocation line, a cash rounding line, an early
payment discount line, a payment term line or a non-deductible line.

**Direction sign** (`direction_sign`). `+1` for an outbound document or a plain entry, `−1` for an
inbound one. It converts a customer-facing price into an accounting balance.

**Discount** (`discount`). A percentage subtracted from a product line's amount before tax.

**Discount allocation account.** A company-level account into which the discounted part of every
product line is moved, so that gross revenue and discounts granted can be read separately. On the
sale side it is the company's "separate account for expense discount".

**Discount allocation line** (display type `discount`). One of the pair of derived lines that moves a
discounted amount from a revenue account to the discount allocation account.

**Display type** (`display_type`). The kind of a journal item. Thirteen values; see
[`entities.md`](entities.md) section 2.1.

**Document currency.** The currency in which a document is expressed and in which the customer pays.

**Document date** (`invoice_date`). The commercial date printed on the document and the date from
which a payment term counts.

**Document type** (`move_type`). The discriminator that turns one journal entry table into seven
kinds of document. See [`entities.md`](entities.md) section 1.3.

**Due date** (`invoice_date_due`). The date by which the whole document is expected to be settled:
the latest maturity date among its payment term lines.

**Dynamic line synchronisation.** The machinery that keeps the derived lines of a document consistent
with the typed lines on every write, running a fixed stack of reconcilers in a fixed order. The
central mechanism of this domain; see [`calculations.md`](calculations.md) section 1.

---

## E

**Early payment discount.** A reduction offered to a customer who pays within a stated number of days
of the document date. Declared on a payment term as a percentage and a number of days, plus one of
three computation modes.

**Early payment discount line** (display type `epd`). A derived line produced only in the "always
(upon invoice)" mode, in pairs: one line removes the discounted amount from the taxed base, the other
puts it back untaxed, so the tax is computed on the reduced base while the net total is unchanged.

**Effective rate.** In the payment term distribution, the ratio of the document-currency total to the
company-currency total, used instead of the stored currency rate so that the instalments add up in
both currencies.

**Exchange difference.** The company-currency gap that appears when a foreign-currency document and
its payment were converted at different rates. It is booked as its own entry and reconciled against
the document.

**Exigibility.** When a tax becomes due to the authority: on invoice, or on payment (cash basis).

---

## F

**Fiscal position.** A mapping that rewrites the accounts and the taxes of a document's lines
according to the customer's tax situation. Resolved from the customer and the delivery address.

**Full reconciliation.** The record created when a group of reconciled journal items reaches zero
residual on every member. Its label becomes the matching number stamped on each line.

---

## G

**Gap (numbering).** A missing number in a journal's numbering chain. A flag marks the first document
that breaks the natural order, and the deletion rules discourage creating gaps.

**Grouping key (tax).** The tuple that decides how many tax lines a document carries: partner,
currency, analytic distribution, account, taxes, tax grids, tax repartition line and grouped tax.
Contributions sharing a key are summed into one line.

---

## I

**In payment.** A payment status meaning the residual is zero but at least one settling payment has
not yet been matched with a bank statement, so the money is not confirmed in the bank. Used only when
the company asks for the distinction.

**Inbound document.** A document for which money is expected to come in: a customer invoice, a sales
receipt or a vendor credit note.

**Instalment.** One payment term line of a document: an amount and a maturity date. Also, in the
portal, one row of the instalment list with its state (overdue, next, before a boundary, early
payment discount, other).

**Invoice.** Loosely, any of the six document types that are not plain journal entries. Strictly in
this domain, a customer invoice.

**Invoice line.** A line the user types: a product line, a section, a subsection or a note. The
filtered view of the document's lines restricted to those four display types.

---

## J

**Journal.** The book in which a document is recorded. A customer document requires a journal of the
sale kind.

**Journal entry** (`account.move`). The record that is at once the commercial document and the
accounting entry.

**Journal item** (`account.move.line`). One line of a journal entry.

---

## L

**Line subtotal** (`price_subtotal`). A product line's tax-excluded amount after discount, computed
for display on every keystroke by running the tax engine on that line alone.

**Line total** (`price_total`). The same with taxes included.

**Lock date.** A date on or before which entries may no longer be added or modified. Five kinds:
fiscal year, sale, purchase, tax and hard.

---

## M

**Matching number** (`matching_number`). The label stamped on every line of a reconciliation group.

**Maturity date** (`date_maturity`). The date at which one payment term line falls due.

**Mixed mode.** The early payment discount computation mode labelled "Always (upon invoice)": the tax
is computed on the already-discounted base at invoicing time.

---

## N

**Needed map.** For a derived line family, the map from key to required amounts that the
synchronisation compares against the lines actually present. Aggregated across contributors, dropping
any key whose monetary values all cancel to zero.

**Needed terms** (`needed_terms`). The needed map of the payment term family: one entry per distinct
(document, maturity date, discount deadline) triple, holding the balance, the amount in currency and
the discount fields.

**Net total.** See *untaxed amount*.

**Non-deductible line.** A derived line of the purchase side that isolates the non-deductible part of
a bill. Never produced on the receivable side.

**Number** (`name`). The document's identifier in its journal's numbering chain. Assigned at posting
unless typed by hand.

---

## O

**Outbound document.** A document for which money is expected to go out: a vendor bill, a customer
credit note or a purchase receipt.

**Outstanding credits.** Unreconciled counterpart lines on the same receivable account, of the
opposite sign to the document, which the user may attach to the document in one click.

**Overdue.** A posted customer invoice or sales receipt, not draft or cancelled, whose payment status
is none of in payment, paid, reversed, blocked or the imported-balance value, and whose due date is strictly before
today.

---

## P

**Partial reconciliation.** The record linking one debit line to one credit line with an amount,
reducing both residuals.

**Payment reference** (`payment_reference`). The string the customer is asked to quote when paying,
so the incoming bank line can be matched automatically. Computed at posting from the journal's
reference model and type.

**Payment status** (`payment_state`). The settlement state of a document: not paid, partially paid,
in payment, paid, reversed, blocked, or the frozen imported-balance value.

**Payment term** (`account.payment.term`). A named instalment plan, optionally carrying one early
payment discount.

**Payment term line** — two meanings, always distinguished by context:
1. **On a payment term** (`account.payment.term.line`): one rule saying how much and when.
2. **On a document** (display type `payment_term`): one receivable journal item carrying one
   instalment and its maturity date.

**Percent line.** A payment term line whose kind is `percent`: it takes a share of the document
total.

**Fixed line.** A payment term line whose kind is `fixed`: it takes an absolute amount expressed in
the document currency.

**Portal.** The customer-facing web area where a customer reads, downloads and pays their documents.

**Posting.** Moving a document from draft to posted: giving it a number, making its journal items
count, creating the analytic lines and making the receivable line reconcilable.

**Pro forma.** A rendering of a document that is explicitly not a legal invoice, used when the
document has no generated file or is not posted. Its title is prefixed with "Proforma".

**Product line** (display type `product`). A line the user types: what is billed, in what quantity, at
what price, with what discount and what taxes.

---

## Q

**Quick encoding.** A company mode in which the user types the gross total and the platform creates a
line to match it. It also relaxes the rule that forbids deleting a numbered document in the middle of
a chain.

**Quick response code.** The two-dimensional barcode printed on a document. Two kinds: the *payment*
code, which encodes bank payment data, and the *portal-link* code, which encodes the online payment
address.

---

## R

**Rate.** See *currency rate* and *effective rate*.

**Receivable account.** An account of the receivable kind, on which customer claims are recorded.
Only payment term lines of a sale document may use one, and they must.

**Recycling.** In the synchronisation, rewriting a line that was going to be deleted with the key and
values of a line that was going to be created, so that its internal identifier survives.

**Reference** (`ref`). The free customer reference of a document, distinct from its number and from
its payment reference.

**Reconciliation.** Linking a debit line and a credit line on the same reconcilable account so their
residuals offset.

**Residual** (`amount_residual`, `amount_residual_currency`). What is left of a journal item after
the reconciled parts are subtracted, in the company currency and in the line currency respectively.

**Reversal.** Creating a document of the opposite type that mirrors the source. Three paths: a plain
reversal, a cancelling reversal, and a cancelling reversal plus a fresh draft copy.

**Reversed.** A payment status meaning the residual reached zero purely through reversing documents,
with no payment involved.

**Rounding delta.** The cent the tax engine shifts onto one base line so that the sum of the rounded
per-line amounts equals the rounded per-tax total.

**Rounding method** (`rounding_method`). The tie-breaking rule of a cash rounding method: `UP` (Up),
`DOWN` (Down) or `HALF-UP` (Nearest).

**Rounding precision** (`rounding`). The smallest circulating coin value of a cash rounding method,
for example 0.05.

**Rounding strategy** (`strategy`). How a cash rounding difference is booked: `add_invoice_line` (Add
a rounding line), which changes the net total, or `biggest_tax` (Modify tax amount), which changes
the tax total.

---

## S

**Sales receipt** (document type `out_receipt`). A customer document meaning "paid at the counter".
Booked exactly like a customer invoice; its reverse is a customer credit note. It is not visible in
the customer portal.

**Section, subsection, note** (display types `line_section`, `line_subsection`, `line_note`).
Non-accountable lines that structure the printed document. A section may hide the composition or the
prices of the lines under it.

**Sending data** (`sending_data`). The transient structure that marks a document as queued for
background sending and carries the author for the job.

**Signed amount.** An amount carrying the accounting sign, as opposed to the customer-facing sign.
Every total exists in both forms.

**Storno accounting.** A company mode in which a reversal is booked as a negative amount on the
original side rather than as an amount on the opposite side.

**Structured creditor reference.** An international payment reference of the form `RF`, two check
digits, then the data, grouped in fours. Its check digits are computed with modulo ninety-seven
arithmetic over the data with the letters replaced by two-digit numbers.

**Suspense account.** The company account used as the last resort by the automatic balancing line.

---

## T

**Tax grid** (`tax_tag_ids`). A reporting bucket a line contributes to, with a sign. Base grids sit on
product lines, tax grids on tax lines.

**Tax line** (display type `tax`). A derived line, one per tax repartition grouping key.

**Tax repartition line.** The rule that says what share of a tax goes to which account and which
grids, separately for invoices and for refunds.

**Total** (`amount_total`). The customer-facing gross total of a document.

**Totals structure** (`tax_totals`). The computed structure the form and the printed document use to
render the totals block, with an inverse so a user may correct a tax amount there.

**Trust (bank account).** The flag that says a bank account number has been verified and may be used
to send money. An untrusted company account on an inbound document blocks posting.

---

## U

**Untaxed amount** (`amount_untaxed`). The customer-facing net total: the sum of the product,
rounding and non-deductible base lines, re-signed.

---

## Symbols and conventions used in the formulas

| Symbol or phrase | Meaning |
| --- | --- |
| `round( x , C )` | Round *x* to the number of decimal places of currency *C*, half away from zero on the currency's rounding step. |
| `is_zero( x , C )` | *x* rounded to currency *C* equals zero. |
| `× ÷ + − =` | Multiply, divide, add, subtract, equals. |
| `\| x \|` | The absolute value of *x*. |
| *s* | The direction sign of the document, or the inbound sign where the text says so. |
| *p* | A discount rate expressed as a fraction, that is the percentage divided by one hundred. |
| *G*, *X* | In the early payment discount formulas, the gross total and the tax total. |
| "the document currency" | The currency of the document being described. |
| "the company currency" | The currency of the company that owns the document. |
| "industry-standard default" | A behavior the source leaves implicit and that this specification states explicitly. |

---

## Cross-domain terms used without redefinition

These belong to other domains and are defined there; they appear here only as dependencies.

| Term | Defined in |
| --- | --- |
| Account, account kind, account group | [`../general-ledger/glossary.md`](../general-ledger/glossary.md) |
| Journal, journal kind, journal group | [`../general-ledger/glossary.md`](../general-ledger/glossary.md) |
| Sequence mixin, numbering grammar, reset rule | [`../general-ledger/glossary.md`](../general-ledger/glossary.md) |
| Inalterability hash, audit trail, lock date exception | [`../general-ledger/glossary.md`](../general-ledger/glossary.md) |
| Tax, tax group, tax repartition, price-included tax, tax engine | [`../taxes/glossary.md`](../taxes/glossary.md) |
| Currency rounding, rate lookup, conversion | [`../multi-currency/glossary.md`](../multi-currency/glossary.md) |
| Analytic plan, analytic account, distribution model | [`../analytic-accounting/glossary.md`](../analytic-accounting/glossary.md) |
| Payment, payment method, outstanding account, bank statement | [`../payments-and-bank-reconciliation/glossary.md`](../payments-and-bank-reconciliation/glossary.md) |
| Payment provider, transaction, token | [`../payment-providers/glossary.md`](../payment-providers/glossary.md) |
| Product, product template, product variant, unit of measure | [`../products-and-catalog/glossary.md`](../products-and-catalog/glossary.md) |
| Message thread, follower, mail template, activity | [`../messaging-and-activities/glossary.md`](../messaging-and-activities/glossary.md) |
| Sales order, order line, invoicing policy | [`../sales/glossary.md`](../sales/glossary.md) |
| Structured electronic document, document exchange network | [`../electronic-invoicing-and-document-exchange/glossary.md`](../electronic-invoicing-and-document-exchange/glossary.md) |
