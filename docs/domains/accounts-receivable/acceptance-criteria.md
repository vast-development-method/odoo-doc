# Acceptance criteria of the Accounts Receivable domain

Numbered scenarios in Given / When / Then form, with concrete numbers. Unless a scenario says
otherwise, the shared fixture is:

| Item | Value |
| --- | --- |
| Company | Northwind Trading, currency the euro, calendar fiscal year, tax rounding "Round per Tax" |
| Sale journal | code `INV`, dedicated credit note sequence on, dedicated debit note sequence on, communication type "Based on Invoice", communication standard "Full Reference" |
| Customer | ACME Industries, receivable account `1200 Trade Receivables`, no credit limit, payment term "Immediate Payment" |
| Income account | `7000 Product Sales` |
| Tax account | `4510 Tax Received` |
| Tax "Sales 21 %" | percentage, 21 %, not price-included, exigible on invoice |
| Tax "Sales 6 %" | percentage, 6 %, not price-included, exigible on invoice |
| Product "Consulting hour" | income account `7000`, customer tax "Sales 21 %", sales price 85.00 |
| Product "Printed manual" | income account `7000`, customer tax "Sales 21 %", sales price 14.50 |
| Product "Digital subscription" | income account `7000`, customer tax "Sales 6 %", sales price 299.00 |
| Today | 2026-03-15 |

---

## Group A — Creating and posting a customer invoice

### A1. Happy path: a three-line invoice with two taxes

**Given** the shared fixture,
**and** a new draft customer invoice for ACME Industries dated 2026-03-15 with the payment term
"Immediate Payment",
**and** three lines: 12 × Consulting hour at 85.00 with no discount; 40 × Printed manual at 14.50
with a 10 % discount; 1 × Digital subscription at 299.00 with no discount,

**When** the document is saved,

**Then** the line subtotals are 1 020.00, 522.00 and 299.00,
**and** the derived lines are exactly two tax lines and one payment term line,
**and** the tax line for Sales 21 % carries 323.82 on a base of 1 542.00,
**and** the tax line for Sales 6 % carries 17.94 on a base of 299.00,
**and** the payment term line carries a debit of 2 182.76 on `1200 Trade Receivables` with maturity
date 2026-03-15,
**and** the document shows Untaxed Amount 1 841.00, Tax 341.76, Total 2 182.76,
**and** the document has no number and its status is draft.

**When** the document is posted,

**Then** the status is posted,
**and** the number is `INV/2026/00001` (the first of the journal in 2026),
**and** the payment reference is `INV/2026/00001`,
**and** the payment term line's label is `INV/2026/00001`,
**and** the payment status is `not_paid`,
**and** the amount due is 2 182.76,
**and** the journal items are exactly:

| Account | Debit | Credit |
| --- | --- | --- |
| 7000 Product Sales | | 1 020.00 |
| 7000 Product Sales | | 522.00 |
| 7000 Product Sales | | 299.00 |
| 4510 Tax Received | | 323.82 |
| 4510 Tax Received | | 17.94 |
| 1200 Trade Receivables | 2 182.76 | |

### A2. Product defaulting

**Given** a new draft customer invoice for ACME Industries,
**When** a line is created and the product "Consulting hour" is chosen,
**Then** the label becomes "Consulting hour" (plus the sales description on a second line when the
product has one, rendered in ACME Industries' language),
**and** the unit becomes the product's reference unit,
**and** the unit price becomes 85.00,
**and** the taxes become {Sales 21 %},
**and** the account becomes `7000 Product Sales`,
**and** the quantity defaults to 1.

### A3. Product defaulting through a fiscal position

**Given** a fiscal position "Intra-community" that maps Sales 21 % to Sales 0 % and `7000 Product
Sales` to `7001 Product Sales (EU)`,
**and** ACME Industries carries that fiscal position,
**When** a new draft customer invoice is created for ACME Industries and a line with the product
"Consulting hour" is added,
**Then** the line's taxes are {Sales 0 %},
**and** the line's account is `7001 Product Sales (EU)`,
**and** the payment term line's account is the mapped receivable account when the fiscal position
maps `1200 Trade Receivables`, and `1200 Trade Receivables` otherwise.

### A4. Posting without a customer

**Given** a draft customer invoice with lines but no customer,
**When** posting is attempted,
**Then** the operation fails with the message:
> The 'Customer' field is required to validate the invoice.
> You probably don't want to explain to your auditor that you invoiced an invisible man :)

### A5. Posting an empty document

**Given** a draft customer invoice with a customer and only one note line,
**When** posting is attempted,
**Then** the operation fails with:
> Even magicians can't post nothing!

### A6. Posting a negative invoice

**Given** a draft customer invoice with one line of quantity 1 and unit price −100.00,
**When** posting is attempted,
**Then** the operation fails with:
> You cannot validate an invoice with a negative total amount. You should create a credit note
> instead. Use the action menu to transform it into a credit note or refund.

### A7. Several failures are reported together

**Given** a draft customer invoice with no customer, no line, and an archived journal,
**When** posting is attempted,
**Then** the operation fails once, with the three messages joined by line breaks, in an
unspecified order:
> The 'Customer' field is required to validate the invoice.
> You probably don't want to explain to your auditor that you invoiced an invisible man :)
>
> Even magicians can't post nothing!
>
> You cannot post an entry in an archived journal (INV)

### A8. Posting fills in a missing document date

**Given** a draft customer invoice with lines, a customer and no document date,
**When** it is posted on 2026-03-15,
**Then** the document date becomes 2026-03-15,
**and** the accounting date becomes 2026-03-15,
**and** the currency rate is not recomputed if the user had overridden it.

### A9. Posting a future-dated document softly

**Given** a draft customer invoice dated 2026-06-01 whose auto-post is "No",
**When** a scheduled caller posts it in the soft mode on 2026-03-15,
**Then** the document stays draft,
**and** its auto-post becomes "At Date",
**and** a note is written on its thread:
> This move will be posted at the accounting date: 06/01/2026
(formatted according to the reader's language).

### A10. Posting a zero-total invoice

**Given** a draft customer invoice for ACME Industries with one line of quantity 1 and unit price
0.00 and no tax,
**When** it is posted,
**Then** the payment status is `paid` (the residual is zero and there is no counterpart at all),
**and** the "invoice paid" hook has fired.

### A11. The customer rank increases

**Given** ACME Industries with a customer rank of 3,
**When** one customer invoice for ACME Industries is posted,
**Then** ACME Industries' customer rank becomes 4,
**and** its commercial entity's customer rank also increases by one when it is a different record.

---

## Group B — Payment terms and instalments

### B1. Thirty percent immediately, the balance in forty-five days

**Given** a payment term "30 % now, balance in 45 days" with two lines in this order: percent 30,
"Days after invoice date", 0 days; percent 70, "Days after invoice date", 45 days,
**and** a customer invoice dated 2026-01-20 with a net total of 1 000.00 and one tax of 21 %,

**When** the document is saved,

**Then** there are two payment term lines,
**and** the first carries a debit of 363.00 with maturity date 2026-01-20,
**and** the second carries a debit of 847.00 with maturity date 2026-03-06,
**and** their sum is exactly 1 210.00,
**and** the document's due date is 2026-03-06,
**and** once posted with the payment reference `INV/2026/00007`, the labels are
`INV/2026/00007 installment #1` and `INV/2026/00007 installment #2`.

### B2. The balance rule absorbs the rounding

**Given** a payment term with three percent lines of 33.33, 33.33 and 33.34 at 0, 30 and 60 days,
**and** a customer invoice with a gross total of 100.00,
**When** the document is saved,
**Then** the three instalments are 33.33, 33.33 and 33.34,
**and** their sum is exactly 100.00.

**Given instead** a payment term with three percent lines of 33.33, 33.33 and 33.33 — which fails
the validation of B10 — the term cannot be saved at all.

**Given instead** a payment term with three percent lines of 33.34, 33.33 and 33.33,
**and** an invoice with a gross total of 100.00,
**Then** the instalments are 33.34, 33.33 and the balance 33.33, summing to 100.00.

### B3. A fixed line plus a balance line

**Given** a payment term with two lines: fixed 500.00 at 0 days; percent 100 at 30 days,
**and** a customer invoice in the company currency with a gross total of 1 210.00,
**When** the document is saved,
**Then** the first instalment is 500.00 due today,
**and** the second instalment is 710.00 due in thirty days,
**and** their sum is 1 210.00.

### B4. A fixed line larger than the total

**Given** the payment term of B3,
**and** a customer invoice with a gross total of 300.00,
**When** the document is saved,
**Then** the first instalment is 500.00,
**and** the second instalment is −200.00,
**and** no validation prevents this; the sum is still 300.00.

### B5. Two lines on the same date merge

**Given** a payment term with two percent lines of 40 and 60, both at 0 days,
**and** a customer invoice with a gross total of 1 000.00,
**When** the document is saved,
**Then** there is exactly **one** payment term line, of 1 000.00, due today,
because the two instalments share the same needed-terms key.

### B6. Every delay type

**Given** a document dated 2026-01-20,
**Then** a single-line payment term produces the following maturity dates:

| Delay type | Days | Day of next month | Maturity |
| --- | --- | --- | --- |
| Days after invoice date | 0 | — | 2026-01-20 |
| Days after invoice date | 45 | — | 2026-03-06 |
| Days after end of month | 0 | — | 2026-01-31 |
| Days after end of month | 15 | — | 2026-02-15 |
| Days after end of next month | 0 | — | 2026-02-28 |
| Days after end of next month | 10 | — | 2026-03-10 |
| Days end of month on the | 0 | 10 | 2026-02-10 |
| Days end of month on the | 15 | 10 | 2026-03-10 |
| Days end of month on the | 10 | 0 | 2026-01-31 |

### B7. No payment term at all

**Given** a customer invoice with no payment term, a gross total of 500.00 and a due date typed as
2026-04-30,
**When** the document is saved,
**Then** there is exactly one payment term line of 500.00 with maturity date 2026-04-30.

**When** the due date is then changed to 2026-05-31,
**Then** the same line's maturity date becomes 2026-05-31 (the line is recycled, not recreated, so
its identifier is unchanged).

### B8. Changing the payment term rebuilds the instalments

**Given** the posted-ready draft of B1 with two instalments,
**When** the payment term is changed to "Immediate Payment",
**Then** there is one payment term line of 1 210.00 due 2026-01-20,
**and** the document's due date becomes 2026-01-20,
**and** the manually typed due date, if any, has been cleared by the change of term.

### B9. Deleting a payment term line by hand

**Given** the saved draft of B1,
**When** the user tries to delete one of the two payment term lines directly,
**Then** the operation fails with:
> You cannot delete a payable/receivable line as it would not be consistent with the payment terms

### B10. Payment term validation

**Given** a payment term being created,
**Then** each of the following is refused with its message:

| Attempt | Message |
| --- | --- |
| percent lines summing to 99 | The Payment Term must have at least one percent line and the sum of the percent must be 100%. |
| no line at all | the same message |
| a percent line of 120 | Percentages on the Payment Terms lines must be between 0 and 100. |
| an early discount with two lines | The Early Payment Discount functionality can only be used with payment terms using a single 100% line. |
| an early discount of 0 % | The Early Payment Discount must be strictly positive. |
| an early discount of 2 % over 0 days | The Early Payment Discount days must be strictly positive. |
| a day-of-next-month of 45 | The days added must be between 0 and 31. |
| a day-of-next-month of `ab` | The days added must be a number and has to be between 0 and 31. |
| a day-of-next-month of `-5` | The days added must be between 0 and 31. |

### B11. Deleting a referenced payment term

**Given** a payment term referenced by at least one document,
**When** it is deleted,
**Then** the operation fails with:
> Uh-oh! Those payment terms are quite popular and can't be deleted since there are still some
> records referencing them. How about archiving them instead?

### B12. The preview

**Given** a payment term with two percent lines of 30 at 0 days and 70 at 45 days, an example amount
of 1 000 and an example date of 2026-01-20,
**Then** the preview shows two rows:
> **1#** Installment of €300.00 due on 01/20/2026
> **2#** Installment of €700.00 due on 03/06/2026

**And given** the same term with an early discount of 2 % within 10 days,
**Then** an additional row appears above:
> Early Payment Discount: **€980.00** if paid before **01/30/2026**

(the two-line restriction of B10 means the term must first be reduced to one line for the discount
to be saved; the preview formula itself is independent of the number of lines).

---

## Group C — Early payment discount

### C1. Mode "On early payment" — the invoice

**Given** a payment term "2/10 net 30" with one percent line of 100 at 30 days, an early discount of
2 % within 10 days, and the computation mode "On early payment",
**and** a customer invoice dated 2026-01-20 with one line of 1 × 1 000.00 taxed at 21 %,
**When** the document is saved and posted,
**Then** the journal items are: a credit of 1 000.00 on `7000`, a credit of 210.00 on `4510`, and a
debit of 1 210.00 on `1200`,
**and** the payment term line stores a discount deadline of 2026-01-30, a discount amount in
currency of 1 185.80 and a discount balance of 1 185.80,
**and** the printed document states "€1,185.80 due if paid before 01/30/2026".

### C2. Mode "On early payment" — paying early

**Given** the posted invoice of C1,
**When** a payment of 1 185.80 is registered on 2026-01-25 with the early discount applied,
**Then** the payment's entry carries a debit of 1 185.80 on the outstanding receipts account, a
debit of 20.00 on the cash discount write-off loss account, a debit of 4.20 on `4510`, and a credit
of 1 210.00 on `1200`,
**and** after reconciliation the invoice's residual is 0.00,
**and** the invoice's payment status is `paid` (or `in_payment` when the company distinguishes and
the payment is not yet matched with a statement),
**and** the tax declared for the period is 205.80 on a base of 980.00.

### C3. Mode "On early payment" — paying late

**Given** the posted invoice of C1,
**When** a payment of 1 210.00 is registered on 2026-02-19,
**Then** no write-off is produced,
**and** the invoice's residual is 0.00 and its payment status is `paid`,
**and** the tax declared stays 210.00.

### C4. Mode "Never"

**Given** the same setup as C1 but with the computation mode "Never",
**Then** the invoice is identical (net 1 000.00, tax 210.00, total 1 210.00),
**and** the payment term line stores a discount amount in currency of 1 190.00,
**and** paying 1 190.00 on 2026-01-25 produces a write-off of a debit of 20.00 on the cash discount
write-off loss account and **no** tax write-off,
**and** the tax declared stays 210.00 on a base of 1 000.00.

### C5. Mode "Always (upon invoice)" — the invoice

**Given** the same setup as C1 but with the computation mode "Always (upon invoice)",
**When** the document is saved,
**Then** two early payment discount lines appear: a debit of 20.00 on `7000` carrying the tax
Sales 21 %, and a credit of 20.00 on `7000` carrying no tax,
**and** the tax line carries 205.80 on a base of 980.00,
**and** the document shows Untaxed Amount 1 000.00, Tax 205.80, Total 1 205.80,
**and** the payment term line carries a debit of 1 205.80 and stores a discount amount in currency
of 1 185.80.

### C6. Mode "Always (upon invoice)" — paying early and late

**Given** the posted invoice of C5,
**When** 1 185.80 is paid on 2026-01-25 with the discount applied,
**Then** the write-off is a single debit of 20.00 on the cash discount write-off loss account, with
no tax line,
**and** the residual reaches zero.

**When instead** 1 205.80 is paid on 2026-02-19,
**Then** no write-off is produced, the residual reaches zero, and the tax reduction booked at
invoicing stands.

### C7. Eligibility

**Given** the posted invoice of C1,
**Then** the document is eligible for the early payment discount only when all of the following
hold:

1. the payment currency equals the document currency;
2. the document type is customer invoice, sales receipt, vendor bill or purchase receipt;
3. the payment term carries an early discount;
4. the reference date is empty, or the document date is empty, or the reference date is on or before
   2026-01-30;
5. no payment term line of the document is already reconciled.

**When** a partial payment of 100.00 is reconciled on 2026-01-22 and a second payment is prepared on
2026-01-25,
**Then** the document is **no longer** eligible, because condition 5 fails.

### C8. Non-discountable taxes are excluded

**Given** a customer invoice with one line of 1 000.00 carrying both Sales 21 % and a fixed tax of
5.00, under the mode "Always (upon invoice)" with a 2 % discount,
**When** the document is saved,
**Then** the early payment discount is computed on the base of the percentage tax only, so the
base-shift line carries 20.00 against Sales 21 % and the fixed tax of 5.00 is untouched.

### C9. Distribution over several lines

**Given** a customer invoice under the mode "Always (upon invoice)" with a 2 % discount and two
lines of 333.33 and 666.67, both on `7000` with Sales 21 %,
**When** the document is saved,
**Then** the group's discount is round(1 000.00 × 0.02) = 20.00,
**and** it is distributed over the two lines in proportion to their raw tax-excluded amounts,
giving 6.67 and 13.33 (the rounding remainder is allocated to the largest weight),
**and** the two produced lines carry 20.00 in total on each side.

---

## Group D — Cash rounding

### D1. Add a rounding line, nearest, 0.05

**Given** a cash rounding method "Swiss 0.05" with precision 0.05, method "Nearest", strategy "Add a
rounding line", profit account `7900 Cash Rounding Gain`, loss account `6900 Cash Rounding Loss`,
**and** the three-line invoice of A1 reduced to: 3 × 12.34 at 21 %, 1 × 45.67 at 21 %, 2 × 9.99 at
6 %, giving net 102.67, tax 18.56 and total 121.23,
**When** the cash rounding method is attached and the document is saved,
**Then** a rounding line appears with a credit of 0.02 on `7900 Cash Rounding Gain`,
**and** the document shows Untaxed Amount 102.69, Tax 18.56, Total 121.25,
**and** the payment term line carries a debit of 121.25.

### D2. Modify the biggest tax amount, nearest, 0.05

**Given** the same fixture with the strategy "Modify tax amount",
**When** the document is saved,
**Then** a rounding line appears with a credit of 0.02 on `4510 Tax Received`, labelled
"Sales 21 % (rounding)", carrying the same tax repartition line and tax grids as the Sales 21 % tax
line,
**and** the document shows Untaxed Amount 102.67, Tax 18.58, Total 121.25,
**and** the payment term line carries a debit of 121.25.

### D3. Rounding down

**Given** the fixture of D1 with the rounding method "Down",
**When** the document is saved,
**Then** the rounding difference is −0.03 for the customer, that is a **debit** of 0.03,
**and** because the balance is strictly positive the **loss** account `6900 Cash Rounding Loss` is
used,
**and** the document shows Untaxed Amount 102.64, Tax 18.56, Total 121.20.

### D4. Rounding up

**Given** the fixture of D1 with the rounding method "Up",
**When** the document is saved,
**Then** the result equals D1 because 121.25 is both the nearest and the next multiple of 0.05
above 121.23.

**And given** a total of 121.26,
**Then** "Up" gives 121.30 (a customer-facing difference of +0.04), "Nearest" gives 121.25 (−0.01)
and "Down" gives 121.25 (−0.01).

### D5. An already-rounded total

**Given** the fixture of D1 with a total of 121.25,
**When** the document is saved,
**Then** no rounding line exists,
**and** if one existed before, it is deleted.

### D6. Switching strategies

**Given** the saved document of D1 carrying a rounding line on `7900`,
**When** the cash rounding method's strategy is changed to "Modify tax amount" and the document is
saved again,
**Then** the old rounding line is deleted and a new one is created on `4510` with the tax
repartition of the biggest tax line.

### D7. Removing the cash rounding method

**Given** the saved document of D1,
**When** the cash rounding method is cleared and the document is saved,
**Then** the rounding line is deleted,
**and** the totals return to net 102.67, tax 18.56, total 121.23.

### D8. Biggest-tax strategy with no tax

**Given** a customer invoice with one untaxed line of 121.23 and a cash rounding method using the
"Modify tax amount" strategy,
**When** the document is saved,
**Then** no rounding line is produced and the total stays 121.23.

### D9. Cash rounding and instalments

**Given** the cash rounding method of D1 and a payment term of 30 % at 0 days and 70 % at 45 days,
**and** a customer invoice whose cash-rounded gross total is 1 207.35,
**When** the document is saved,
**Then** the first instalment is 30 % of 1 207.35 = 362.205, rounded to the currency as 362.21, then
cash-rounded by −0.01 to **362.20**,
**and** the second instalment is the balance 1 207.35 − 362.20 = **845.15**,
**and** both are multiples of 0.05,
**and** their sum equals the total exactly.

### D10. Validation

**Given** a cash rounding method being created with a precision of 0.00,
**When** it is saved,
**Then** the operation fails with:
> Please set a strictly positive rounding value.

**And given** the strategy "Add a rounding line" is chosen while the current company has no profit
account on the method,
**Then** a non-blocking warning appears:
> **Warning for Cash Rounding Method: Swiss 0.05**
> You must specify the Profit Account (company dependent)

---

## Group E — Discount allocation

### E1. Two lines, merged keys, weighted analytic distribution

**Given** the company's "Customer Invoices Discounts Account" set to `7050 Discounts Granted`,
**and** a customer invoice with two lines on `7000 Product Sales`: 10 × 100.00 with a 10 % discount
and an analytic distribution of Project Alpha 100 %; 5 × 200.00 with a 20 % discount and an analytic
distribution of Project Alpha 50 % and Project Beta 50 %,
**When** the document is saved,
**Then** two discount lines are produced:

| Account | Debit | Credit | Analytic distribution |
| --- | --- | --- | --- |
| 7000 Product Sales | 300.00 | | Alpha 66.67 %, Beta 33.33 % |
| 7050 Discounts Granted | | 300.00 | Alpha 66.67 %, Beta 33.33 % |

**and** the net total of the document is unchanged by the pair.

### E2. No discount allocation account

**Given** the same invoice with the company's discount account cleared,
**When** the document is saved,
**Then** no discount line is produced.

### E3. A line whose account is already the discount account

**Given** a line whose income account is `7050 Discounts Granted` and which carries a discount,
**When** the document is saved,
**Then** that line contributes no discount allocation entries (the rule requires the line's account
to differ from the allocation account).

### E4. A discount that rounds to zero

**Given** a line of 1 × 0.01 with a 10 % discount,
**Then** the discounted amount rounds to 0.00 and the line contributes nothing; no discount line
appears for it.

---

## Group F — Numbering

### F1. The first document of the year

**Given** a sale journal with code `INV`, a calendar fiscal year, and no posted document in 2026,
**When** a customer invoice dated 2026-03-15 is posted,
**Then** its number is `INV/2026/00001`.

### F2. The next document

**Given** `INV/2026/00001` exists and is posted,
**When** a second customer invoice dated 2026-03-20 is posted,
**Then** its number is `INV/2026/00002`.

### F3. A new year restarts the counter

**Given** `INV/2026/00042` is the highest number,
**When** a customer invoice dated 2027-01-04 is posted,
**Then** its number is `INV/2027/00001`.

### F4. A dedicated credit note sequence

**Given** the journal's dedicated credit note sequence is on and no credit note exists in 2026,
**When** a customer credit note dated 2026-03-20 is posted,
**Then** its number is `RINV/2026/00001`,
**and** the invoice counter is unaffected.

### F5. A dedicated debit note sequence

**Given** the journal's dedicated debit note sequence is on and no debit note exists in 2026,
**When** a customer debit note dated 2026-03-20 is posted,
**Then** its number is `DINV/2026/00001`.

### F6. A staggered fiscal year

**Given** the company's fiscal year ends on 30 June,
**When** a customer invoice dated 2026-03-15 is posted as the first of its range,
**Then** its number is `INV/25-26/0001`,
**and** a customer invoice dated 2026-07-01 posted afterwards is numbered `INV/26-27/0001`.

### F7. A draft has no number but shows a placeholder

**Given** `INV/2026/00007` is the highest number,
**When** a draft customer invoice dated 2026-03-20 is created,
**Then** its number is empty,
**and** its placeholder is `INV/2026/00008`.

### F8. Changing the date of a never-posted draft clears a stale number

**Given** a draft customer invoice whose number was typed as `INV/2026/00008` and whose date is
2026-03-20,
**When** the date is changed to 2027-01-04,
**Then** the number is cleared, because the document has never been posted and its number no longer
matches its period.

### F9. Duplicate numbers are impossible

**Given** `INV/2026/00007` exists and is posted,
**When** a second document in the same journal is posted with the same number,
**Then** the operation fails with:
> Another entry with the same name already exists.

### F10. Changing the journal of a numbered document

**Given** a posted document numbered `INV/2026/00007` that has been reset to draft,
**When** its journal is changed without clearing the number,
**Then** the operation fails with:
> You cannot edit the journal of an account move if it has been posted once, unless the name is
> removed or set to "/". This might create a gap in the sequence.

### F11. Deleting a document in the middle of a chain

**Given** posted documents `INV/2026/00005`, `INV/2026/00006` and `INV/2026/00007`, all reset to
draft,
**and** the current user is in the invoicing group but not the accountant group, and quick-encoding
mode is off,
**When** `INV/2026/00006` is deleted,
**Then** the operation fails with:
> You cannot delete this entry, as it has already consumed a sequence number and is not the last one
> in the chain. You should probably revert it instead.

**When** `INV/2026/00007` is deleted instead,
**Then** the deletion succeeds.

### F12. The accounting date is pushed past a lock date

**Given** a fiscal year lock date of 2026-02-28,
**and** a sale document dated 2026-02-10 whose numbering resets yearly,
**When** it is posted on 2026-03-15,
**Then** its accounting date becomes the earlier of today and 31 December 2026, that is 2026-03-15,
**and** its number is taken from the 2026 series.

---

## Group G — The payment reference

### G1. Full reference, based on invoice

**Given** the journal's communication standard "Full Reference" and type "Based on Invoice",
**When** `INV/2026/00042` is posted,
**Then** the payment reference is `INV/2026/00042`.

### G2. Full reference, based on customer

**Given** the standard "Full Reference" and type "Based on Customer",
**and** ACME Industries whose internal reference is `ACM-17`,
**When** the invoice is posted,
**Then** the payment reference is `CUST/ACM-17`.

### G3. Numbers only, based on invoice

**Given** the standard "Numbers only" and type "Based on Invoice",
**When** `INV/2026/00042` is posted,
**Then** the payment reference is `202600042`.

### G4. Numbers only, based on customer

**Given** the standard "Numbers only" and type "Based on Customer",
**and** a customer whose internal reference is `customer 97`,
**Then** the payment reference is `97`.

### G5. European, based on invoice

**Given** the standard "European" and type "Based on Invoice",
**and** a journal whose code is `INV`,
**and** an invoice whose internal identifier is 37,
**Then** the data is `INV000037`,
**and** the check number is 67,
**and** the payment reference is `RF67 INV0 0003 7`.

### G6. European, based on customer

**Given** the standard "European" and type "Based on Customer",
**and** a journal code of `INV` and a customer whose internal reference is `food buyer 654`,
**Then** the data is `INV654`,
**and** the check number is 16,
**and** the payment reference is `RF16 INV6 54`.

### G7. An unknown combination

**Given** a journal whose communication standard and type name no known generator,
**When** a customer invoice is posted,
**Then** the operation fails with:
> The combination of reference model and reference type on the journal is not implemented

### G8. The reference reaches the line labels

**Given** the posted invoice of G1 with a two-instalment payment term,
**Then** the two payment term lines are labelled `INV/2026/00042 installment #1` and
`INV/2026/00042 installment #2`.

**Given instead** a single-instalment term,
**Then** the single line is labelled `INV/2026/00042`.

### G9. The sanitised form

**Given** the payment reference `RF67 INV0 0003 7`,
**Then** the sanitised payment reference is `RF67INV000037`,
**and** a bank statement line whose free text contains that string can be matched against the
document.

---

## Group H — Payment status

### H1. A partial payment

**Given** the posted invoice of A1 with a residual of 2 182.76,
**When** a payment of 1 000.00 is registered and reconciled on 2026-03-20,
**Then** the receivable line's residual becomes 1 182.76 and its reconciled flag stays false,
**and** the document's amount due becomes 1 182.76,
**and** the payment status becomes `partial`,
**and** the amount paid shown in the portal is 1 000.00,
**and** the next amount to pay is 1 182.76,
**and** no journal item is added to the invoice.

### H2. Full payment

**Given** the situation after H1,
**When** a second payment of 1 182.76 is registered and reconciled,
**Then** the receivable line's residual becomes 0.00 and its reconciled flag becomes true,
**and** a full reconciliation record is created and its label is stamped on every participating line,
**and** the document's payment status becomes `paid`,
**or**, when the company distinguishes in-payment and the payments are not yet matched with a bank
statement, `in_payment`.

### H3. In payment becomes paid

**Given** an invoice whose payment status is `in_payment` because its settling payment is not
matched,
**When** the payment is matched with a bank statement line,
**Then** the payment status becomes `paid`.

### H4. Undoing a reconciliation

**Given** the fully paid invoice of H2,
**When** one of the two partial reconciliations is removed,
**Then** the residual becomes the removed amount,
**and** the payment status becomes `partial`.

**When** both are removed,
**Then** the residual becomes 2 182.76 and the payment status becomes `not_paid`.

### H5. Reversed rather than paid

**Given** a posted customer invoice of 2 182.76 and a posted customer credit note of 2 182.76 for
the same customer,
**When** their receivable lines are reconciled together,
**Then** both residuals reach zero,
**and** the invoice's payment status is `reversed`, not `paid`, because the only counterpart type is
a customer credit note and no payment is involved,
**and** the credit note's payment status is also `reversed`.

### H6. Blocking

**Given** the posted invoice of A1 with payment status `not_paid`,
**When** the payment block is toggled on,
**Then** the payment status becomes `blocked`,
**and** the computation no longer overwrites it.

**When** the block is toggled off,
**Then** the status is recomputed and becomes `not_paid`.

**Given instead** an invoice whose payment status is `paid`,
**When** the block is toggled on,
**Then** the operation fails with:
> You can't block a paid invoice.

### H7. A draft invoice with a non-zero total qualifies

**Given** a draft customer invoice with a total of 500.00 and no reconciliation,
**Then** its payment status is `not_paid` (it qualifies as an invoice because it is draft with a
non-zero total).

**Given instead** a draft customer invoice whose total is 0.00,
**Then** it does not qualify and its payment status is forced to `not_paid` by the fallback branch.

### H8. The imported-balance status is never overwritten

**Given** an imported document whose payment status is the imported-balance value (`invoicing_legacy`),
**When** any reconciliation changes,
**Then** the payment status stays the imported-balance value (`invoicing_legacy`).

---

## Group I — Credit notes and reversal

### I1. Credit note against a paid invoice

**Given** the posted invoice of A1, fully paid on 2026-03-20, payment status `paid`,
**When** the reversal wizard is opened with reversal date 2026-04-02 and reason "Manuals returned",
and the **Reverse** button is used,
**Then** a draft customer credit note is created with:

| Field | Value |
| --- | --- |
| Type | Customer Credit Note |
| Reversal of | the invoice |
| Reference | `Reversal of: INV/2026/00001, Manuals returned` |
| Accounting date, document date, due date | 2026-04-02 |
| Journal | `INV` |
| Payment term | empty |
| Salesperson, origin | copied |
| Auto-post | `no` |

**and** every line of the invoice has been copied,
**and** the invoice's thread carries "This entry has been **reversed**" with a link,
**and** the invoice's payment status is still `paid`.

**When** the user keeps only the "Printed manual" line and posts the credit note,
**Then** the credit note's number is `RINV/2026/00001`,
**and** its journal items are a debit of 522.00 on `7000`, a debit of 109.62 on `4510` and a credit
of 631.62 on `1200`,
**and** its payment status is `not_paid` with a residual of 631.62,
**and** the invoice is untouched.

**When** an outbound payment of 631.62 is registered against the credit note and reconciled,
**Then** the credit note's payment status becomes `paid`.

**When instead** the credit note is left unreconciled,
**Then** it appears in the outstanding credits block of the customer's next invoice,
**and** the customer's total receivable is reduced by 631.62.

### I2. Credit note against an unpaid invoice, fully reconciled

**Given** the posted invoice of A1, unpaid,
**When** it is fully reversed and the two receivable lines are reconciled,
**Then** both residuals reach zero,
**and** the invoice's payment status becomes `reversed`.

### I3. Reverse and create invoice

**Given** the posted invoice of A1, unpaid,
**When** the reversal wizard is used with **Reverse and Create Invoice** and a reversal date of
2026-04-02,
**Then** a credit note is created, posted immediately and reconciled with the invoice,
**and** the invoice's payment status becomes `reversed`,
**and** a third document is created: a draft customer invoice carrying only the three product lines,
dated 2026-04-02, with the invoice's origin copied,
**and** that draft has no tax lines yet until it is saved, after which the synchronisation rebuilds
them.

### I4. A future-dated reversal is not cancelling

**Given** the posted invoice of A1 and a reversal date of 2026-12-01 (in the future),
**When** **Reverse and Create Invoice** is used,
**Then** the credit note's auto-post is `at_date` and it is **not** posted or reconciled
immediately, because the cancelling behavior requires a non-future reversal.

### I5. The early-discount payment term survives

**Given** a posted invoice whose payment term uses the mode "Always (upon invoice)",
**When** it is reversed,
**Then** the credit note keeps the same payment term (unlike the general rule that clears it), so its
early payment discount lines mirror the invoice's.

### I6. Reversal validation

**Given** documents selected for reversal,
**Then** each of the following is refused with its message:

| Attempt | Message |
| --- | --- |
| documents of two companies | All selected moves for reversal must belong to the same company. |
| a draft document | To reverse a journal entry, it has to be posted first. |
| a journal whose kind differs from the sources' | Journal should be the same type as the reversed entry. |

---

## Group J — Debit notes

### J1. A debit note copying the lines

**Given** the posted invoice `INV/2026/00001` of A1,
**When** the debit note wizard is used with the reason "Price correction", the date 2026-04-02 and
the "copy lines" option on,
**Then** a draft **customer invoice** is created with:

| Field | Value |
| --- | --- |
| Type | Customer Invoice |
| Original invoice debited | `INV/2026/00001` |
| Reference | `INV/2026/00001, Price correction` |
| Accounting date and document date | 2026-04-02 |
| Payment term | empty |
| Lines | the three product lines copied |

**and** the source's thread carries "This debit note was created from:" with a link,
**and** posting it gives the number `DINV/2026/00001`.

### J2. A debit note without lines

**Given** the same source and the "copy lines" option off,
**Then** the debit note is created with no line at all.

### J3. A debit note from a credit note

**Given** a posted customer credit note,
**When** a debit note is created from it,
**Then** the debit note's type is **Customer Invoice**,
**and** its lines are always empty, whatever the "copy lines" option says.

### J4. Debit note validation

| Attempt | Message |
| --- | --- |
| a draft source | You can only debit posted moves. |
| a source that already has a debit note | You can't make a debit note for an invoice that is already linked to a debit note. |
| a source of type sales receipt | You can make a debit note only for a Customer Invoice, a Customer Credit Note, a Vendor Bill or a Vendor Credit Note. |

---

## Group K — Foreign currency

### K1. A foreign-currency invoice

**Given** a company whose currency is the euro,
**and** a rate of 1 euro = 1.0850 United States dollars on 2026-05-04,
**and** a customer invoice in dollars dated 2026-05-04 with one line of 4 × 250.00 taxed at 21 %,
**When** the document is saved and posted,
**Then** the document's currency rate is 1.0850,
**and** the journal items are:

| Account | Debit (euro) | Credit (euro) | Amount in currency (dollar) |
| --- | --- | --- | --- |
| 7000 Product Sales | | 921.66 | −1 000.00 |
| 4510 Tax Received | | 193.55 | −210.00 |
| 1200 Trade Receivables | 1 115.21 | | +1 210.00 |

**and** the document shows Untaxed Amount 1 000.00 dollars, Tax 210.00 dollars, Total 1 210.00
dollars, Total Signed 1 115.21 euros.

### K2. An exchange difference on settlement

**Given** the posted invoice of K1,
**and** a rate of 1 euro = 1.1000 dollars on 2026-06-10,
**When** a payment of 1 210.00 dollars is registered on 2026-06-10 into a dollar bank account and
reconciled with the invoice,
**Then** the dollar residual reaches zero,
**and** an exchange difference entry is created in the exchange journal dated 2026-06-10 with a
debit of 15.21 on the loss exchange account and a credit of 15.21 on `1200 Trade Receivables`,
**and** that counterpart line is reconciled with the invoice's receivable line,
**and** the invoice's payment status becomes `paid`.

**When** the reconciliation is later undone,
**Then** the exchange difference entry is reversed.

### K3. Changing the rate on a draft

**Given** the draft of K1 before posting,
**When** the currency rate is changed from 1.0850 to 1.1000,
**Then** the tax line keeps −210.00 dollars and its euro balance becomes
round(−210.00 ÷ 1.1000) = −190.91,
**and** the product line is recomputed from the unit price, giving −1 000.00 dollars and
round(−1 000.00 ÷ 1.1000) = −909.09 euros,
**and** the payment term line is rebuilt to +1 210.00 dollars and +1 100.00 euros.

### K4. Changing the currency on a draft

**Given** the draft of K1,
**When** the currency is changed from the dollar to the pound sterling,
**Then** the tax lines are recomputed from the bases rather than having their amounts preserved,
**and** every line's currency becomes the pound sterling,
**and** the rate is looked up for the pound at the document date.

### K5. A non-positive rate is refused

**Given** a draft customer invoice in dollars,
**When** the currency rate is set to 0,
**Then** the operation fails with:
> The currency rate must be strictly positive.

### K6. An archived currency cannot be posted

**Given** a draft customer invoice whose currency is archived,
**When** posting is attempted,
**Then** the operation fails with:
> You cannot validate a document with an inactive currency: USD

---

## Group L — The dynamic line synchronisation

### L1. Adding a line rebuilds the taxes and the instalments

**Given** the saved invoice of A1 with two tax lines and one payment term line,
**When** a fourth line of 1 × Digital subscription at 299.00 is added and the document is saved,
**Then** the Sales 6 % tax line becomes 35.88 on a base of 598.00,
**and** the payment term line becomes 2 517.64,
**and** no line has been created and deleted unnecessarily: the existing tax line and payment term
line were updated in place.

### L2. Removing every taxed line removes the tax lines

**Given** the saved invoice of A1,
**When** the two lines carrying Sales 21 % are deleted and the document is saved,
**Then** the Sales 21 % tax line is deleted,
**and** the Sales 6 % tax line remains,
**and** the payment term line becomes 316.94.

### L3. A manual tax correction is preserved

**Given** the saved invoice of A1,
**When** the user edits the Sales 21 % amount in the totals block from 323.82 to 323.80,
**Then** that tax line carries 323.80,
**and** the payment term line becomes 2 182.74,
**and** saving again without touching the lines leaves the manual amount alone.

**When** a product line's quantity is then changed,
**Then** the tax lines are recomputed from the bases, discarding the manual amount, because the
changed line carries taxes and the tax lines were not all protected.

### L4. A user-created derived line is respected

**Given** a caller that creates a document with its product lines, its tax lines **and** explicit
amounts on the base lines in one operation,
**When** the document is saved,
**Then** the synchroniser recomputes nothing: it detects that the changed base lines carry explicit
amounts and stops.

### L5. A deleted derived line is not resurrected

**Given** a saved document whose needed maps have not changed,
**When** a derived line is deleted through a path that bypasses the deletion guard,
**Then** the synchroniser does not recreate it, because keys naming line identifiers that no longer
exist are dropped from the before-state.

### L6. Recycling preserves identifiers

**Given** the saved invoice of B7 with one payment term line whose internal identifier is *n*,
**When** the due date is changed,
**Then** there is still exactly one payment term line and its internal identifier is still *n*.

### L7. The order of reconciliation

**Given** a document with a cash rounding method, a discount allocation account, taxes and a
two-instalment payment term,
**When** a product line's price is changed and the document is saved,
**Then** the derived families are reconciled in this order: partner propagation, early payment
discount lines, non-deductible lines, tax lines, discount allocation lines, cash rounding line,
automatic balancing line, payment term lines,
**and** therefore the payment term lines see the final totals including the cash rounding and the
tax changes.

### L8. A posted document is never synchronised

**Given** a posted customer invoice,
**When** any field is written that would normally trigger the synchronisation,
**Then** no derived line is created, updated or deleted,
**and** writing one of the readonly fields fails with:
> You cannot modify the following readonly fields on the posted move INV/2026/00001: *the field names*

### L9. The balance invariant holds after every save

**Given** any of the documents in this file,
**When** it is saved,
**Then** the sum of the balances of its lines, rounded to the company currency, is zero,
**and** if it were not, the save would fail with:
> The entry is not balanced.

---

## Group M — Sending

### M1. Sending one invoice by e-mail

**Given** the posted invoice of A1,
**and** ACME Industries whose invoice sending method is "by Email" and whose e-mail address is set,
**When** the user presses **Send**, keeps the defaults, and confirms,
**Then** the printable document is rendered with the resolved layout and stored as the document's
file attachment,
**and** that attachment becomes the thread's main attachment,
**and** the sent flag becomes true,
**and** a comment message is posted on the thread with the subject
`Northwind Trading Invoice (Ref INV/2026/00001)`, the template body, the customer as recipient and
the file attached,
**and** the journal's invoice subscribers receive a copy.

### M2. Sending a credit note picks the credit note template

**Given** a posted customer credit note,
**When** the send wizard is opened,
**Then** the chosen template is "Credit Note: Sending" with the subject
`Northwind Trading Credit Note (Ref RINV/2026/00001)`.

### M3. A recipient without an e-mail address blocks a single send

**Given** a posted invoice whose only recipient has no e-mail address,
**When** the send wizard is confirmed,
**Then** the operation fails with:
> Partner(s) should have an email address.

**Given instead** a batch of several documents in the same situation,
**Then** the alert is a warning, not a blocker, and offers the action "View Partner(s)".

### M4. Batch sending

**Given** five posted customer invoices and an active sending job,
**When** the batch wizard is confirmed,
**Then** each document's sending data is set,
**and** the job is triggered,
**and** the user sees the notification "Invoices are being sent in the background." titled "Sending
invoices",
**and** each document's "being sent" indicator is true until the job processes it.

### M5. Batch sending with the job archived

**Given** the sending job is archived,
**When** the batch wizard is confirmed by a system administrator,
**Then** the operation fails with a redirecting warning:
> Batch invoice sending is unavailable. Please, activate the cron to enable batch sending of
> invoices.
with the button "Go to cron configuration".

**Given instead** a user who is not a system administrator,
**Then** the operation fails with:
> Batch invoice sending is unavailable. Please, contact your system administrator to activate the
> cron to enable batch sending of invoices.

### M6. Sending an unposted document

**Given** a draft customer invoice,
**When** sending is attempted,
**Then** the operation fails with:
> You can't generate invoices that are not posted.

### M7. Sending a vendor bill

**Given** a posted vendor bill,
**When** sending is attempted through this flow,
**Then** the operation fails with:
> You can only generate sales documents.

### M8. Resetting to draft detaches the file

**Given** the sent invoice of M1,
**When** it is reset to draft,
**Then** the file attachment stops being the value of the document's file field,
**and** it is renamed to `INV_2026_00001 (detached by Jane Doe on 03/20/2026).pdf`
(the original name, the parenthetical, then the extension),
**and** the send button is highlighted again because no file is linked.

### M9. The job retries a retryable error

**Given** a document queued for batch sending whose external service fails with a retryable error,
**When** the job runs,
**Then** the error is written on the document's thread,
**and** the sending data is **kept**, so the next run tries again.

**Given instead** a non-retryable error,
**Then** the sending data is cleared and the document is no longer queued.

---

## Group N — The customer portal

### N1. The list page

**Given** ACME Industries' portal user,
**and** three documents: a posted invoice, a posted credit note and a draft invoice,
**When** the user opens the document list,
**Then** only the two posted documents appear (the record rule excludes draft and cancelled),
**and** they are sorted by document date descending by default.

### N2. Receipts are invisible in the portal

**Given** a posted sales receipt for ACME Industries,
**When** the portal user opens the document list,
**Then** the receipt does **not** appear, because the portal record rule covers only customer
invoices, customer credit notes, vendor bills and vendor credit notes.

### N3. The overdue count

**Given** two posted customer invoices for ACME Industries: one due 2026-02-01 with a residual of
500.00 and one due 2026-04-01 with a residual of 300.00,
**and** today is 2026-03-15,
**Then** the overdue count is 1.

### N4. Downloading

**Given** a posted invoice that has a generated file,
**When** the portal user asks for the printable form with the download flag,
**Then** the stored file is returned.

**Given instead** a posted invoice with no generated file,
**Then** the page is rendered in *pro forma* mode and the download returns the freshly rendered
*pro forma* document.

### N5. Paying online

**Given** the posted invoice of A1 with a residual of 2 182.76,
**and** the portal payment parameter on and at least one enabled provider in euros,
**When** the portal user opens the document page,
**Then** the payment form is shown with an amount of 2 182.76,
**and** the transaction address is `/invoice/transaction/<the identifier>`,
**and** the landing address is the document's portal address.

### N6. Paying is refused

**Given** the fully paid invoice of H2,
**When** the portal user opens the document page,
**Then** the payment form is not shown,
**and** the page explains, among the applicable sentences:
> There is no amount to be paid.
> This invoice has already been paid.

### N7. A pending transaction blocks a second payment

**Given** a posted invoice with a pending transaction from a real provider,
**Then** the payment form is not shown and the page states:
> There are pending transactions for this invoice.

### N8. A tampered custom amount

**Given** a portal address carrying an amount and a signed token that do not match,
**When** the page is requested,
**Then** the visitor is redirected to the portal home.

### N9. Batch payment of overdue invoices

**Given** three overdue customer invoices for ACME Industries, all in euros, of 500.00, 300.00 and
200.00,
**When** the portal user opens the overdue page,
**Then** the total offered is 1 000.00,
**and** the communication is the company's next batch payment communication,
**and** the transaction address is `/invoice/transaction/overdue`.

**Given instead** two overdue invoices in different currencies,
**Then** the transaction call fails with:
> Impossible to pay all the overdue invoices if they don't share the same currency.

**Given** an anonymous visitor,
**Then** the transaction call fails with:
> Please log in to pay your overdue invoices

### N10. Instalments in the portal

**Given** the two-instalment invoice of B1, posted, unpaid, today being 2026-02-15,
**Then** the instalment state is `overdue` (the first instalment was due 2026-01-20),
**and** the amount due shown is the document residual 1 210.00,
**and** the next amount to pay is 363.00,
**and** the next reference is `INV/2026/00007-1`,
**and** the next due date is 2026-01-20.

**Given instead** today being 2026-01-15,
**Then** the instalment state is `next`,
**and** the next amount to pay is 363.00,
**and** the next reference is `INV/2026/00007-1`.

### N11. The early payment discount in the portal

**Given** the posted invoice of C1 and today being 2026-01-25,
**Then** the instalment state is `epd`,
**and** the amount due shown is 1 185.80,
**and** the next amount to pay is 1 210.00,
**and** the message reads:
> Discount of €24.20 if paid within 5 days

**Given** today being 2026-01-30 (the deadline itself),
**Then** the message reads:
> Discount of €24.20 if paid today

---

## Group O — The credit limit

### O1. Below the limit

**Given** the credit limit feature on, ACME Industries with a limit of 10 000.00 and a receivable
balance of 7 500.00,
**and** a draft customer invoice of 2 000.00,
**Then** the total credit is 9 500.00 and no warning appears.

### O2. Above the limit

**Given** the same setup with a draft customer invoice of 3 000.00,
**Then** the total credit is 10 500.00 and the warning reads:
> ACME Industries has reached its credit limit of: €10,000.00
> Total amount due (including this document): €10,500.00

### O3. With orders awaiting invoicing

**Given** the same setup, a draft invoice of 1 000.00 and 3 000.00 of confirmed orders not yet
invoiced,
**Then** the total credit is 11 500.00 and the warning's second line reads:
> Total amount due (including sales orders and this document): €11,500.00

### O4. On a posted document

**Given** a posted customer invoice,
**Then** no credit warning is computed at all, because the warning applies only to draft customer
invoices.

### O5. The partner limit toggle

**Given** a company default credit limit of 5 000.00,
**When** ACME Industries' "Partner Limit" toggle is switched off,
**Then** its credit limit becomes 5 000.00,
**and** the toggle reads as off because the limit equals the company default.

**When** the limit is set to 10 000.00,
**Then** the toggle reads as on.

---

## Group P — Duplicate detection

### P1. Two identical customer invoices

**Given** a posted customer invoice for ACME Industries dated 2026-03-15 with a total of 2 182.76 in
euros,
**When** a second draft customer invoice for the same customer, the same date, the same total and
the same currency is created,
**Then** the second document lists the first as a duplicate,
**and** the first lists the second as a duplicate,
**and** the second's "has draft duplicates" flag is false while the first's is true.

### P2. A different total is not a duplicate

**Given** the same setup with a total of 2 182.75,
**Then** neither document lists the other.

### P3. A different currency is not a duplicate

**Given** the same amounts but one document in dollars,
**Then** neither lists the other.

### P4. A cancelled document is not a duplicate

**Given** the first document cancelled,
**Then** it no longer appears as a duplicate of the second.

### P5. Deleting duplicates

**Given** the situation of P1 with the second document selected,
**When** the delete-duplicates action is run,
**Then** the first document is deleted if the deletion rules allow it.

---

## Group Q — Locking, hashing and the audit trail

### Q1. A lock date blocks a change of date

**Given** a posted customer invoice dated 2026-01-15 and a fiscal year lock date of 2026-02-28,
**When** its accounting date is changed,
**Then** the operation fails with:
> You cannot add/modify entries prior to and inclusive of: *the lock date names and values*.

### Q2. A hashed document cannot be reset

**Given** a posted customer invoice in a journal with hashing enabled, whose hash is set,
**When** a reset to draft is attempted,
**Then** the operation fails with:
> You cannot reset to draft a locked journal entry.

### Q3. A hashed document cannot be edited

**Given** the same document,
**When** one of the hashed fields is written,
**Then** the operation fails with:
> This document is protected by a hash. Therefore, you cannot edit the following fields: *the field
> labels*.

### Q4. The restrictive audit trail forbids deletion

**Given** a company with the restrictive audit trail on,
**and** a customer invoice that has been posted and then reset to draft,
**When** it is deleted,
**Then** the operation fails with:
> To keep the restrictive audit trail, you can not delete journal entries once they have been
> posted.
> Instead, you can cancel the journal entry.

### Q5. The get-rid-of routine picks the right path

**Given** three documents: one draft never posted; one posted inside a locked period; one posted
under a restrictive audit trail,
**When** the routine that removes documents is invoked on all three,
**Then** the first is deleted,
**and** the second is reversed with a cancelling reversal,
**and** the third is cancelled.

---

## Group R — Line-level rules

### R1. A receivable account requires a due date

**Given** a draft customer invoice,
**When** a line is created on `1200 Trade Receivables` with a display type of `product`,
**Then** the operation fails with:
> Any journal item on a receivable account must have a due date and vice versa.

### R2. A payable account on a sale document

**Given** a draft customer invoice,
**When** a line is created on `4400 Trade Payables`,
**Then** the operation fails with:
> Account 4400 is of payable type, but is used in a sale operation.

### R3. An archived account

**Given** a draft customer invoice with a line on an archived account,
**When** posting is attempted,
**Then** the operation fails with:
> A line of this move is using a archived account, you cannot post it.

**And when** an archived account is written onto a line at any time,
**Then** the operation fails with:
> You cannot use an archived account.

### R4. An account from another company

**Given** a draft customer invoice of company Northwind Trading with a line on an account belonging
only to company Southwind Trading,
**When** posting is attempted,
**Then** the operation fails with:
> The entry is using accounts (5000 Other Account) from a different company.

### R5. Off-balance accounts

**Given** a draft document with one line on an off-balance account and one on a normal account,
**Then** the operation fails with:
> If you want to use "Off-Balance Sheet" accounts, all the accounts of the journal entry must be of
> this type

### R6. Deleting a line of a posted document

**Given** a posted customer invoice,
**When** one of its non-zero lines is deleted,
**Then** the operation fails with:
> You can't delete a posted journal item. Don’t play games with your accounting records; reset the
> journal entry to draft before deleting it.

### R7. Changing the taxes of a posted line

**Given** a posted customer invoice,
**When** a line's taxes are changed,
**Then** the operation fails with:
> You cannot modify the taxes related to a posted journal item, you should reset the journal entry
> to draft to do so.

### R8. A section line with an amount

**Given** a line whose display type is `line_section`,
**When** an amount or an account is written on it,
**Then** the stored check refuses it:
> Forbidden balance or account on non-accountable line

### R9. Deductibility on a customer invoice

**Given** a draft customer invoice,
**When** a line's deductibility is set to 80,
**Then** the operation fails with:
> Only vendor bills allow for deductibility of product/services.

### R10. A reconciliation-breaking edit

**Given** a posted customer invoice whose receivable line is reconciled,
**When** that line's account is changed while the counterpart line is **not** part of the same write,
**Then** the reconciliation is removed automatically before the write proceeds.

**When instead** every line of the reconciliation group is written together and the only changing
field is the account,
**Then** the reconciliation is preserved.

---

## Group S — Scheduled jobs

### S1. The auto-post job posts due documents

**Given** three draft documents: one dated 2026-03-10 with auto-post "At Date", one dated 2026-03-20
with auto-post "At Date", one dated 2026-03-10 with auto-post "No",
**and** today is 2026-03-15,
**When** the auto-post job runs,
**Then** only the first is posted.

### S2. The auto-post job isolates a failure

**Given** two draft documents due for auto-posting, the second of which has no customer,
**When** the job runs,
**Then** the batch attempt fails and is rolled back,
**and** the job retries one at a time: the first is posted,
**and** the second stays draft with a note on its thread:
> The move could not be posted for the following reason: The 'Customer' field is required to
> validate the invoice.
> You probably don't want to explain to your auditor that you invoiced an invisible man :)

### S3. A recurring invoice

**Given** a draft customer invoice dated 2026-01-31 with auto-post "Monthly" and an end date of
2026-04-30,
**When** it is posted on 2026-01-31,
**Then** a draft copy dated 2026-02-28 is created with the same auto-post setting,
**and** each posting produces the next copy, until the copy dated 2026-04-30 is posted and no
further copy is produced.

**When** the copy dated 2026-02-28 is reset to draft after the copy dated 2026-03-31 has been
created but is still draft,
**Then** the copy dated 2026-03-31 is deleted.

### S4. The sending job processes ten at a time

**Given** twenty-five documents queued for batch sending,
**When** the job runs,
**Then** at most ten are processed per run, ordered by accounting date, then document date, then
sequence position, then identifier,
**and** the job reports fifteen remaining so the scheduler runs again.

---

## Group T — Multi-company and access

### T1. Company scoping of documents

**Given** a user whose allowed companies are Northwind Trading only,
**When** the customer document list is opened,
**Then** only documents of Northwind Trading are visible, by the company record rule.

### T2. A branch company may use a parent's accounts

**Given** Northwind Trading with a branch Northwind North,
**and** a receivable account shared by the parent,
**When** a customer invoice of Northwind North uses it,
**Then** posting succeeds, because the account's companies include an ancestor of the document's
company.

### T3. Posting without the invoicing group

**Given** a user who is only in the read-only accounting group,
**When** posting is attempted,
**Then** the operation fails with:
> You don't have the access rights to post an invoice.

### T4. The portal user sees only their own documents

**Given** two customers with posted invoices,
**When** one customer's portal user requests the other's document page without a valid access token,
**Then** the visitor is redirected to the portal home.

### T5. Reading the credit figures

**Given** an internal user who is in neither the invoicing group nor the read-only accounting group,
**When** the customer form is opened,
**Then** the total receivable, the total payable, the credit limit and the total invoiced are not
readable.

---

## Group U — Numeric edge cases

### U1. The rounding delta on price-included taxes

**Given** two lines of 1 × 21.53 each, with a price-included tax of 21 %, and global tax rounding,
**When** the document is saved,
**Then** both lines display a subtotal of 17.79,
**and** their stored balances are 17.80 and 17.79,
**and** the single tax line carries 7.47,
**and** the totals are Untaxed 35.59, Tax 7.47, Total 43.06.

### U2. A very small amount

**Given** a line of 1 × 0.01 with Sales 21 %,
**Then** the tax is round(0.01 × 0.21) = round(0.0021) = 0.00,
**and** no tax line is produced for that grouping key because its amount is zero in both currencies,
**and** the total is 0.01.

### U3. A three-decimal currency

**Given** a currency with three decimal places and a line of 1 × 1.2345,
**Then** the line subtotal is 1.235 (half away from zero at the third decimal),
**and** the payment term line carries 1.235.

### U4. A zero-decimal currency

**Given** a currency with zero decimal places and a line of 3 × 1 000 with a 21 % tax,
**Then** the subtotal is 3 000, the tax is 630 and the total is 3 630, all whole numbers.

### U5. An instalment on a leap day

**Given** a document dated 2028-01-29 and a payment term line of "Days after end of next month",
0 days,
**Then** the maturity date is 2028-02-29 (2028 is a leap year).

**Given** the same term on a document dated 2027-01-29,
**Then** the maturity date is 2027-02-28.

### U6. "Days end of month on the" with a short target month

**Given** a document dated 2026-01-20 and a line of "Days end of month on the", 10 days, day of next
month 31,
**Then** the intermediate date is 2026-01-30, moving one month forward gives February 2026, and
setting the day to 31 clamps to 2026-02-28.

---

## Group V — End-to-end scenarios

### V1. Quote to cash with instalments and an early discount

1. **Given** the fixture, a payment term "2/10 net 45" with one line of 100 % at 45 days, an early
   discount of 2 % within 10 days, mode "On early payment".
2. **When** a customer invoice dated 2026-03-15 is created for ACME Industries with 12 × Consulting
   hour at 85.00 and posted,
   **Then** the number is `INV/2026/00001`, the total is 1 234.20 (1 020.00 net plus 214.20 of tax),
   the single instalment is due 2026-04-29, the discount deadline is 2026-03-25 and the discounted
   amount is round(1 234.20 × 0.98) = 1 209.52.
3. **When** the document is sent by e-mail,
   **Then** the printable document states "€1,209.52 due if paid before 03/25/2026", the sent flag
   is true and the thread carries the message.
4. **When** the customer pays 1 209.52 on 2026-03-20 through the portal,
   **Then** a transaction is created, a payment of 1 209.52 is booked, a write-off of 20.40 on the
   cash discount write-off loss account and 4.28 on `4510` is added, the receivable is cleared, and
   the payment status becomes `paid`.

   *Check:* the discount of 24.68 splits as a net part of round(1 020.00 × 0.02) = 20.40 and a tax
   part of round(214.20 × 0.02) = 4.28, and 20.40 + 4.28 = 24.68 = 1 234.20 − 1 209.52. ✓

### V2. Invoice, partial payment, credit note, refund

1. **When** the invoice of A1 is posted (2 182.76) and a payment of 1 000.00 is reconciled,
   **Then** the payment status is `partial` and the residual is 1 182.76.
2. **When** a credit note of 631.62 is issued for the returned manuals and reconciled against the
   invoice,
   **Then** the invoice's residual becomes 551.14 and the payment status stays `partial`,
   **and** the credit note's residual becomes 0.00 and its payment status becomes `paid`.
3. **When** a final payment of 551.14 is reconciled,
   **Then** the invoice's residual becomes 0.00 and the payment status becomes `paid` — not
   `reversed`, because a payment counterpart is involved.

### V3. Foreign currency with instalments and cash rounding

1. **Given** a company in euros, a customer invoice in Swiss francs dated 2026-05-04 with a rate of
   1 euro = 0.9500 francs, a cash rounding method of 0.05 nearest with the "add a rounding line"
   strategy, and a payment term of 40 % at 0 days and 60 % at 30 days.
2. **And** one line of 1 × 1 000.00 francs taxed at 21 %: net 1 000.00, tax 210.00, total 1 210.00
   francs, which is already a multiple of 0.05 so no rounding line is produced.
3. **Then** the euro amounts are: net round(1 000.00 ÷ 0.95) = 1 052.63, tax
   round(210.00 ÷ 0.95) = 221.05, total 1 273.68.
4. **And** the effective rate for the distribution is |1 210.00 ÷ 1 273.68| = 0.95000…, so:
   - instalment 1: 40 % of 1 210.00 francs = 484.00, and 40 % of 1 273.68 euros = 509.47; the cash
     rounding difference of 484.00 at 0.05 is zero, so no correction;
   - instalment 2 (the balance): 1 210.00 − 484.00 = 726.00 francs and 1 273.68 − 509.47 = 764.21
     euros.
5. **Then** the journal items balance in both columns: credits 1 052.63 + 221.05 = 1 273.68 euros and
   1 000.00 + 210.00 = 1 210.00 francs; debits 509.47 + 764.21 = 1 273.68 euros and
   484.00 + 726.00 = 1 210.00 francs. ✓

### V4. A fully worked cash rounding and instalment interaction

1. **Given** a cash rounding method of 0.05 nearest, "add a rounding line", and a payment term of
   30 % at 0 days and 70 % at 30 days.
2. **And** a customer invoice with one line of 1 × 998.00 taxed at 21 %: net 998.00, tax
   round(998.00 × 0.21) = 209.58, total 1 207.58.
3. **Then** the cash rounding difference for the customer is 1 207.60 − 1 207.58 = +0.02, so a
   rounding line of a credit of 0.02 is created on the profit account, and the rounded total is
   1 207.60.
4. **And** the first instalment is 30 % of 1 207.60 = 362.28, whose cash rounding difference is
   362.30 − 362.28 = +0.02, so it becomes **362.30**.
5. **And** the second instalment is the balance 1 207.60 − 362.30 = **845.30**, which is a multiple
   of 0.05.
6. **Then** the two instalments sum to 1 207.60 exactly, and both are payable in the available
   coinage. ✓
