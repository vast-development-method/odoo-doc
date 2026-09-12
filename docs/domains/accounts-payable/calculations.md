# Accounts Payable — Calculations and Algorithms

Every formula in this file is written with named quantities. Unless stated otherwise:

- `round_to(currency, x)` means: round `x` to the smallest representable step of that currency — its **rounding factor** — using the *round half away from zero* rule. For a currency whose rounding factor is 0.01 this is ordinary two-decimal rounding. The full arithmetic is in `../multi-currency/calculations.md`.
- `is_zero(currency, x)` means `round_to(currency, x) = 0`.
- **company currency** is the currency of the company that owns the document; **document currency** is the currency written on the document.
- The **document rate** is the number of document-currency units per company-currency unit (`invoice_currency_rate`). Multiplying a company-currency amount by the rate gives the document-currency amount; dividing does the reverse.

---

## 1. Sign conventions on a purchase document

Every commercial quantity on a document is positive. The ledger needs signed balances. The bridge is the **direction sign**:

```formula
direction_sign = +1   when the document type is a miscellaneous entry, or an outbound type (in_invoice, out_refund, in_receipt)
direction_sign = −1   otherwise (out_invoice, in_refund, out_receipt)
```

For a **vendor bill** the direction sign is +1. Therefore:

```formula
line_balance = direction_sign × line_amount_in_document_currency ÷ document_rate
```

so an expense line of a bill has a **positive** balance (a debit) and the payable term line, being the counterpart, has a **negative** balance (a credit).

For a **vendor credit note** the direction sign is −1, so the expense line is a credit and the payable line is a debit.

The document totals are derived from the line balances and then re-signed for display:

```formula
amount_untaxed        = direction_sign × Σ amount_in_currency over product, rounding and non-deductible-product lines
amount_tax            = direction_sign × Σ amount_in_currency over tax, non-deductible-tax and tax-bearing rounding lines
amount_total          = direction_sign × Σ amount_in_currency over all of the above
amount_residual       = − direction_sign × Σ residual_in_currency over payment-term lines
amount_untaxed_signed = − Σ balance over product, rounding and non-deductible-product lines
amount_tax_signed     = − Σ balance over tax lines
amount_total_signed   = − Σ balance over all non-term lines      (for a miscellaneous entry: the absolute value)
amount_residual_signed=   Σ residual over payment-term lines
```

Consequence: on a vendor bill `amount_total` is positive (it is what the company owes) while `amount_total_signed` is **negative** (the ledger sees a credit on the payable side).

---

## 2. Dates

### 2.1 The three dates and their roles

| Date | Role |
|---|---|
| bill date (`invoice_date`) | The date the supplier issued the document. The reference date for payment terms, for duplicate detection and for the currency rate. |
| accounting date (`date`) | The date the entry hits the ledger and the period it belongs to. Determines the numbering period and whether a lock date is violated. |
| due date (`invoice_date_due`) | The last maturity date among the generated instalments. |

### 2.2 Deriving the accounting date from the bill date

The accounting date is computed from the **accounting date source**, which is the bill date when there is one and the current accounting date otherwise.

1. If the source is empty, or the document is not invoice-like: if the accounting date is empty, set it to today; stop.
2. If the document is a **sale** document: the accounting date is the source unchanged (the shifting rules below apply to sale documents only when a lock date is violated).
3. Otherwise — and every purchase document takes this path — run the algorithm of §2.3.
4. If the result differs from the current accounting date, write it, and schedule the recomputation of the line dates and of the document number.

### 2.3 The accounting date algorithm

Inputs: the candidate date *d*, a flag saying whether the document affects the tax report, and the list of **violated lock dates** for that candidate, the journal and the company, ordered chronologically.

1. Let *today* be the current date in the user's time zone.
2. Let *highest name* be the highest number already used in this numbering series, or, when that is unknown, the last number of the series computed with the relaxed rule.
3. Deduce the **number reset period** from that name: `month`, `year`, `year_range_month`, `year_range`, or `never`. This is the grammar of the numbering series (see `../general-ledger/`).
4. If any lock date is violated, replace *d* by the **latest violated lock date plus one day**.
5. If the document is a sale document:
   - if no lock date was violated → answer *d*;
   - if the series has no name yet, or it resets monthly → answer `min(today, last day of the month of d)`;
   - if it resets yearly → answer `min(today, last day of the year of d)`;
   - otherwise → answer *d*.
6. Otherwise (**every purchase document**):
   - if the series has no name yet, or it resets monthly, or it resets monthly within a year range:
     - if *today* is in a later month than *d* → answer **the last day of the month of *d***;
     - else → answer `max(d, today)`;
   - else if it resets yearly:
     - if *today* is in a later year than *d* → answer **31 December of the year of *d***;
     - else → answer `max(d, today)`;
   - otherwise → answer *d*.

The purpose is that a bill captured late still lands in the period it belongs to, while the numbering stays monotonically increasing.

**Worked example.** Today is 11 September 2026. A vendor bill dated 20 August 2026 is captured in a purchase journal whose numbers reset monthly and whose highest number is `BILL/2026/08/0007`. No lock date is violated. Today's month (September) is later than the bill's month (August), so the accounting date becomes **31 August 2026**, and the bill is numbered in the August series.

**Second worked example.** Same journal, a bill dated 2 September 2026 captured today, 11 September 2026. Today's month equals the bill's month, so the accounting date is `max(2 September, 11 September)` = **11 September 2026**.

### 2.4 The lock-date banner

When at least one lock date is violated, the document shows

> The date is being set prior to: «the formatted list of lock dates». The Journal Entry will be accounted on «the resulting accounting date» upon posting.

as a warning-level alert, visible only to users holding the read-only accounting group or the invoicing group.

---

## 3. Choosing the supplier bank account

The recipient bank account is recomputed whenever the bank partner, the currency or the preferred payment method line changes.

1. If the document is **inbound** (for payables that means a vendor credit note) **and** a payment method line is available — the document's preferred one, or failing that the bank partner's inbound payment method line property — **and** that method line has a journal, then the account is **that journal's bank account**; stop.
2. Otherwise, take the bank accounts of the **bank partner**: the company partner for an inbound document, the commercial partner (the vendor) for an outbound document.
3. Keep only the accounts that belong to the document's company (or to no company) and that are **active** — the active filter is applied explicitly, because some contexts disable it implicitly.
4. Sort them by the key

```formula
sort_key(account) = ( currency_priority , not allowed_out_payment )
```

   where `currency_priority` is **0** when the account has no currency or its currency equals the document currency, and **1** otherwise; and `not allowed_out_payment` is **false** (sorting first) when the account is trusted for outgoing payments.

5. Take the **first** account of that ordering, or none when the list is empty.

So: an account in the document's currency beats one in another currency; among equals, a trusted account beats an untrusted one.

---

## 4. Duplicate bill detection

### 4.1 Purpose

When the same supplier document is captured twice — once by electronic mail, once by hand, once again from a scan — the system must say so before the bill is posted or paid twice.

### 4.2 When it runs

The set of probable duplicates is recomputed whenever the vendor reference, the document type, the partner, the bill date, the tax totals or the currency changes. It runs on documents that are sale documents or purchase documents (receipts included); anything else yields an empty set.

Because the comparison must work on a document **being typed and not yet saved**, the algorithm injects the in-memory values of the candidate document into the comparison rather than reading them from storage. The fields injected are: company, partner, commercial partner, vendor reference, type, bill date, status, total and currency; plus the identifier, which is the stored identifier for an existing document and **0** for a brand-new one. The total is taken from the live tax totals structure rather than from the stored total, because the stored total is only computed at save time.

### 4.3 The matching predicate

A stored document *D* is a duplicate candidate of the document under examination *M* when **all** of the following hold. Unless said otherwise, `matching states` is the pair (`draft`, `posted`).

**Common conditions (both directions):**

1. `D.company = M.company`
2. `D.identifier ≠ M.identifier`
3. `D.status ∈ matching states`
4. the types are compatible:
   - `D.type = M.type`, **or**
   - both types are in {`in_invoice`, `in_receipt`}, **or**
   - both types are in {`out_invoice`, `out_receipt`}
5. `D.currency = M.currency`
6. the partners match: `D.commercial_partner = M.commercial_partner`, **or** `M` has no commercial partner **and** `D` is a draft

**Purchase-specific condition** (applied when *M* is of type `in_invoice`, `in_refund` or `in_receipt`; *D* must also be one of those three):

7. Either
   - **case 1 — same reference, same year:** `D.reference = M.reference` **and** ( `M` has no bill date **or** `D` has no bill date **or** the two bill dates fall in the same calendar year );
   - **or case 2 — different references but everything else identical:** `D.commercial_partner = M.commercial_partner` **and** `D.total = M.total` **and** `M.total ≠ 0` **and** `D.bill_date = M.bill_date`.

**Sale-specific condition** (for completeness, since the same routine serves both sides): `D.total = M.total` **and** `D.bill_date = M.bill_date`.

The result is grouped per examined document and then filtered down to the records the reading user is allowed to read.

### 4.4 Exact versus probable duplicates

A detected duplicate *D* makes the examined purchase document an **exact duplicate** when **all** of:

- `M.reference` is non-empty and `D.reference = M.reference`;
- the types are compatible (equal, or both in {`in_invoice`, `in_receipt`}, or both in {`out_invoice`, `out_receipt`});
- `D.partner = M.partner` — note this compares the **partner**, not the commercial partner, so a different contact of the same company is not an exact duplicate;
- `D.bill_date = M.bill_date`;
- `D.total = M.total`;
- `M` is a purchase document (a bill or a vendor credit note; **not** a receipt, because this predicate excludes receipts).

A separate flag records whether **any** detected duplicate is still a draft — this is what enables the delete action.

### 4.5 The two warnings

| Condition | Level | Text |
|---|---|---|
| the exact-duplicate flag is set | **danger** (red) | *This document might be a duplicate of* followed by a button naming the first duplicate, showing its total; plus, when at least one duplicate is a draft, a red action reading *Delete duplicate* when there is one duplicate and *Delete all duplicates* when there are several |
| duplicates exist, the exact-duplicate flag is **not** set, and the document is still a draft | **warning** (amber) | the same wording and the same delete action |

In the bill upload list, the vendor reference cell is shaded: red when the exact-duplicate flag is set, amber when duplicates exist and the document is a draft.

The delete action removes **every** detected duplicate of the current document.

### 4.6 Consequence for automatic posting

A bill that would otherwise be posted automatically is **not** posted when duplicates were detected; instead the chatter receives

> Auto-post was disabled on this invoice because a potential duplicate was detected.

### 4.7 Worked example — a duplicate case with its warning

*Setting.* Company "Northwind", currency Euro. A purchase journal. Vendor "Papeterie Lambert" (commercial partner, with one child contact "Papeterie Lambert — Accounts").

**Bill A**, captured on 3 March 2026 by electronic mail:

| Attribute | Value |
|---|---|
| type | `in_invoice` |
| partner | Papeterie Lambert |
| vendor reference | `FA-2026-0412` |
| bill date | 2 March 2026 |
| currency | Euro |
| total | 1 452.00 |
| status | posted |

**Bill B**, captured on 17 March 2026 by hand, same reference typed again:

| Attribute | Value |
|---|---|
| type | `in_invoice` |
| partner | Papeterie Lambert |
| vendor reference | `FA-2026-0412` |
| bill date | 2 March 2026 |
| currency | Euro |
| total | 1 452.00 |
| status | draft |

Evaluate the predicate for B against A: same company ✓; different identifiers ✓; A is posted, which is in the matching states ✓; both types are `in_invoice` ✓; same currency ✓; same commercial partner ✓; purchase case 1 — same reference and both bill dates in 2026 ✓. So **A is a duplicate candidate of B**.

Evaluate the exact-duplicate predicate: reference non-empty and equal ✓; compatible types ✓; same partner ✓; same bill date ✓; same total ✓; B is a purchase document ✓. So B is an **exact** duplicate.

B therefore shows the **red** banner *This document might be a duplicate of* with a button naming `A` and its total, and — because B itself is draft but A is posted, the draft-duplicate flag looks at the **detected duplicates**, and A is posted — the delete action is **hidden**. Had A still been draft, B would offer *Delete duplicate*.

Automatic posting of B is suppressed, and the chatter of B receives *Auto-post was disabled on this invoice because a potential duplicate was detected.*

**Variant.** Suppose B had been typed with reference `FA 2026 0412` (spaces instead of hyphens) but the same partner, the same date 2 March 2026 and the same total 1 452.00. Case 1 fails (references differ) but **case 2 succeeds** (same commercial partner, same non-zero total, same bill date). A is still detected. The exact-duplicate predicate fails because the references differ, so B shows the **amber** banner instead of the red one, and automatic posting is still suppressed.

**Second variant.** Suppose B had the reference `FA-2026-0412` but a bill date of 2 March **2025**. Case 1 fails (different calendar years) and case 2 fails (different bill dates). **No duplicate is detected**, and B posts normally. This is deliberate: suppliers restart their own numbering every year.

---

## 5. Payment terms

### 5.1 The due date of one term line

Let *r* be the **reference date**, which for a document is `invoice_date` when set, otherwise `date`, otherwise today. Let *n* be the line's day count and *d* the line's "days on the next month" value.

| Delay type | Due date |
|---|---|
| `days_after` | `r + n days` |
| `days_after_end_of_month` | `last_day_of_month(r) + n days` |
| `days_after_end_of_next_month` | `last_day_of_month(r + 1 month) + n days` |
| `days_end_of_month_on_the`, with *d* ≤ 0 | `last_day_of_month(r + n days)` |
| `days_end_of_month_on_the`, with *d* > 0 | `(r + n days)` advanced by one month, then the day of month forced to *d* |

Adding a month to a date clamps the day to the length of the target month (31 January plus one month is 28 or 29 February).

### 5.2 Distributing the total over the term lines

Inputs: the reference date; the document currency; the company; the **signed** tax amount and untaxed amount in both currencies; the direction sign; and, optionally, the cash rounding rule.

Let

```formula
total_amount          = tax_amount + untaxed_amount                    (company currency)
total_amount_currency = tax_amount_currency + untaxed_amount_currency  (document currency)
rate                  = | total_amount_currency ÷ total_amount |       (0 when total_amount is 0)
```

Note that `rate` is derived from the two totals rather than read from the document, so that the instalments always add back up to exactly the two totals.

Initialise `residual_amount = total_amount` and `residual_amount_currency = total_amount_currency`.

Then for each term line *i*, **in the term's own line order** (by identifier, that is by creation order):

1. The due date is computed as in §5.1.
2. Let `on_balance_line` be true when *i* is the **last** line.
3. If `on_balance_line`:
   - `company_amount = residual_amount`
   - `foreign_amount = residual_amount_currency`
   
   **The last line is always the balance, whatever its declared type.** This is what guarantees that the instalments sum exactly to the total, with no rounding drift.
4. Else if the line is a **fixed** line:
   - `company_amount = direction_sign × round_to(company_currency, line_value ÷ rate)` — and 0 when the rate is 0
   - `foreign_amount = direction_sign × round_to(document_currency, line_value)`
5. Else (a **percent** line):
   - `company_amount = round_to(company_currency, total_amount × line_value ÷ 100)`
   - `foreign_amount = round_to(document_currency, total_amount_currency × line_value ÷ 100)`
6. If cash rounding applies **and** this is not the balance line: compute the cash rounding difference of `foreign_amount` (see §8); if it is not zero, add it to `foreign_amount` and recompute `company_amount = round_to(company_currency, foreign_amount ÷ rate)` (0 when the rate is 0). Because only cash-rounded amounts are ever subtracted from a cash-rounded total, the balance line is cash-rounded as well.
7. Subtract both amounts from the two residuals.

### 5.3 The early payment discount attached to the distribution

When the term enables the early discount (which requires exactly one line), the distribution additionally answers:

```formula
discount_percentage = the term's percentage
discount_date       = reference_date + discount_days
```

and a discounted balance computed according to the tax-reduction mode:

| Mode | Discounted balance (company currency) | Discounted amount (document currency) |
|---|---|---|
| `excluded` (*Never*) or `mixed` (*Always (upon invoice)*) | `round_to(company, total_amount − untaxed_amount × p)` | `round_to(document, total_amount_currency − untaxed_amount_currency × p)` |
| `included` (*On early payment*) | `round_to(company, total_amount × (1 − p))` | `round_to(document, total_amount_currency × (1 − p))` |

where `p = discount_percentage ÷ 100`.

In the first two modes the discount is applied to the **untaxed** amount only, so the tax stays whole; in the third it is applied to the whole amount including tax.

When cash rounding applies, the discounted document-currency amount is cash-rounded and the discounted balance is recomputed as `round_to(company, discounted_amount_currency ÷ rate)`.

There is a separate, simpler routine used for display and for the payment register, *amount due after discount*:

```formula
discount_amount = (total − untaxed) × p     in modes excluded and mixed
discount_amount = total × p                 in mode included
amount_due      = round_to(currency, total − discount_amount)
```

followed, when the calling context names a document that carries a cash rounding rule, by adding that rule's difference and rounding again.

### 5.4 From the distribution to payable term lines

The distribution is turned into a map — the *needed terms* — keyed by *(document, maturity date, discount date)* and valued by *(company amount, document amount, discount date, discounted balance, discounted amount)*. Two lines of the same term falling on the same date are **merged**: their amounts are summed.

When the document has **no** payment term at all, a single entry is produced, keyed on the document's own due date, valued at the signed totals.

While the document is an unsaved draft, the tax and untaxed amounts fed into the distribution are recomputed from the base lines rather than read from the stored totals.

The synchronizer then creates, updates or deletes payable term lines so that they match the map exactly. Each such line gets:

- **account**: the payable account chosen by §6.1;
- **display type**: `payment_term`;
- **sequence**: 12000, so term lines always sort last;
- **maturity date**: the key's date;
- **label**: computed as follows —
  - if the document has both a payment reference and a vendor reference and they differ → `«vendor reference» - «payment reference»`;
  - else if it has a payment reference → the payment reference;
  - else if it is a bill or a vendor credit note and has a vendor reference → **the vendor reference**;
  - else → empty;
  - and then, when the term has **more than one line**, the label becomes `«label» installment #«index»`, the index being the one-based position of this line among the document's term lines sorted by maturity date (lines with no date sort last), with a leading space stripped when the label was empty.

### 5.5 Worked example — sixty days from end of month

*Setting.* Company "Northwind", company currency Euro. Vendor payment term **"60 days end of month"**: a single line, type *percent*, amount 100, delay type `days_after_end_of_month`, day count 60.

A vendor bill is captured with bill date **14 April 2026** and a total of 3 600.00 Euro (untaxed 3 000.00, tax 600.00).

Step 1 — the due date.

```formula
last_day_of_month(14 April 2026) = 30 April 2026
due_date = 30 April 2026 + 60 days = 29 June 2026
```

Counting: April has 30 days, so 30 April + 30 days = 30 May; + 30 days = 29 June. The due date is **29 June 2026**.

Step 2 — the amounts. The direction sign of a bill is +1, so the signed amounts fed into the distribution are `untaxed_amount_currency = 3 000.00`, `tax_amount_currency = 600.00`, and the same values in company currency because the document is in the company currency. Therefore `total_amount = total_amount_currency = 3 600.00` and `rate = 1`.

There is exactly one line, so it **is** the balance line: it takes the whole residual.

```formula
company_amount = 3 600.00
foreign_amount = 3 600.00
```

Step 3 — the payable term line. One line is produced:

| Attribute | Value |
|---|---|
| account | the vendor's payable account |
| display type | `payment_term` |
| maturity date | 29 June 2026 |
| balance | −3 600.00 (a credit; the sign comes from the counterpart rule of §9) |
| amount in currency | −3 600.00 |
| label | the vendor reference, because there is no payment reference and the document is a bill |

Step 4 — the document due date. It is the **maximum** maturity date among the term lines, that is **29 June 2026**.

**Variant with two instalments.** Suppose the term were: line 1, *percent* 30, delay `days_after_end_of_month`, 0 days; line 2, *percent* 70, delay `days_after_end_of_month`, 60 days.

- line 1 due date: `last_day_of_month(14 April 2026) + 0 = 30 April 2026`; amount `round_to(Euro, 3 600.00 × 30 ÷ 100) = 1 080.00`; residual becomes 2 520.00.
- line 2 is the last line, so it is the balance line: due date 29 June 2026, amount 2 520.00 — which happens to equal 70 % exactly here, but would absorb any rounding drift if it did not.
- The labels become `«vendor reference» installment #1` and `«vendor reference» installment #2`, ordered by maturity date.
- The document due date is 29 June 2026.

**Variant showing the balance rule.** Term: line 1 *percent* 33.33, line 2 *percent* 33.33, line 3 *percent* 33.34, all `days_after` with 0, 30 and 60 days; total 100.00 Euro.

- line 1: `round_to(Euro, 100.00 × 33.33 ÷ 100) = 33.33`; residual 66.67
- line 2: `round_to(Euro, 100.00 × 33.33 ÷ 100) = 33.33`; residual 33.34
- line 3 is last → takes 33.34 exactly.

Sum 100.00. Had the percentages been 33.33 / 33.33 / 33.33 the constraint of §7.3 of `entities.md` would have rejected the term, because the percent lines must sum to exactly 100.

**Variant with the fourth delay type.** Term: one line, delay `days_end_of_month_on_the`, day count 10, days-on-next-month 15. Bill date 25 April 2026.

```formula
25 April 2026 + 10 days = 5 May 2026
5 May 2026 + 1 month = 5 June 2026, then day forced to 15 → 15 June 2026
```

### 5.6 Worked example — early payment discount on a bill

Term: one line, *percent* 100, `days_after` 30; early discount enabled, 2 %, 10 days; mode `included`.

Bill dated 1 June 2026, untaxed 1 000.00, tax 21 % = 210.00, total 1 210.00 Euro.

```formula
discount_date  = 1 June 2026 + 10 days = 11 June 2026
p              = 0.02
discount_amount_currency = round_to(Euro, 1 210.00 × (1 − 0.02)) = round_to(Euro, 1 185.80) = 1 185.80
```

The payable term line is 1 210.00 due 1 July 2026, carrying a discount date of 11 June 2026 and a discounted amount of 1 185.80. Paying on or before 11 June settles the bill for 1 185.80.

In mode `excluded` or `mixed` the discounted amount would instead be `round_to(Euro, 1 210.00 − 1 000.00 × 0.02) = 1 190.00`: the 20.00 reduction applies to the net only, the 210.00 tax being untouched.

---

## 6. Account selection

### 6.1 The payable account of a term line

For each payable term line, in this order — the first non-empty wins:

1. **The account already used by another payable term line of the same document.** (Formally: the account of a payment-term line of the same document other than the ones currently being computed; the first such line found.)
2. The **vendor's payable account property** — `property_account_payable_id` of the document's commercial partner, read in the document's company.
3. The **company partner's payable account property**, read in the same company.
4. The **first active payable account of the company**, taken as an arbitrary but deterministic pick: for each *(company, account type)* pair one account is chosen among the active accounts of that type belonging to that company.

Then, if the document has a fiscal position, the chosen account is passed through the position's **account mapping**, and the mapped account replaces it.

For a **sale** document the same algorithm runs with the receivable property and the receivable type.

### 6.2 The expense account of a product line

For each line whose display type is `product` on an invoice-like document:

1. If the line has a **product**: read the product's accounts in the line's company, resolved through the document's fiscal position; take the **expense** account for a purchase document (the **income** account for a sale document); if it is empty, keep the account already on the line.
2. Else, if the line has a **partner**: take the **most frequently used account for that partner** (see §6.3). If one is found, use it.

Then, for **any** line still without an account and not a presentation line:

3. Look at the accounts of the **last two lines of the same display type** on the document. If those two lines resolve to exactly **one** distinct account **and** the document has more than two lines in total, use that account — the "continue with what the neighbours use" rule.
4. Otherwise use the **journal's default account**.

### 6.3 The most frequently used account for a partner

Given a company, a partner and a document type:

1. Consider the journal items of that company whose partner is the given partner, whose account is active, and whose date is on or after **two years (730 days) before today**.
2. Restrict by document type: for a type beginning with `out_` only accounts of type *income*; for a type beginning with `in_` only accounts of type *expense*, *fixed asset* or *direct cost of revenue*.
3. Group by account, order by the **number of matching journal items descending**, then by the account **code ascending** as a tie-break, and take the first.
4. The answer is cached per *(company, partner, document type)* for the duration of the request.

If no journal item matches, there is no answer and the algorithm falls through to §6.2 step 3.

### 6.4 The most frequent account **and taxes** (quick encoding only)

A second, related query answers both an account and a tax set. Given a company, a partner and a document type:

1. Consider the journal items of that company, for that partner, dated within the last two years.
2. Restrict by direction: for an **inbound** type, accounts whose internal group is *income*; for an **outbound** type — which covers every bill and purchase receipt — accounts whose internal group is **expense**.
3. For each journal item collect its account, its account code and the array of taxes attached to it.
4. Group by *(account, tax array)*, order by the count of items descending and then by the tax array ascending with empty arrays last, and take the first.
5. The answer is the triple *(count, account, tax identifiers)*; when the partner is empty the answer is *(0, none, none)*.

---

## 7. The amount in words

### 7.1 The algorithm

Given a currency and an amount:

1. If no textual-number library is available, answer the empty string and log a warning.
2. Format the amount with exactly `decimal_places` digits after the point, where `decimal_places` is the currency's number of decimal places (2 for a currency whose rounding factor is 0.01). Split the result at the point into the **integral** part and the **fractional** part.
3. Let `integer_value` be the integral part read as a whole number.
4. Resolve the user's language and take its two-letter code.
5. Define `spell(n)`: render *n* in words in that language and then apply **title casing** — that is, upper-case the first letter of every maximal run of letters, lower-casing the rest. If the language is unsupported, render in English instead and title-case that.
6. If `is_zero(currency, amount − integer_value)` — the amount has no fractional part — answer

```formula
words = spell(integer_value) + " " + currency_unit_label
```

7. Otherwise answer

```formula
words = spell(integer_value) + " " + currency_unit_label + " and " + spell(fractional_part_as_integer) + " " + currency_subunit_label
```

   where `fractional_part_as_integer` is the fractional digit string read as a whole number (empty reads as 0).

The two labels are per-currency translatable texts. For the United States dollar they are *Dollars* and *Cents*; for the euro, *Euros* and *Cents*.

### 7.2 Where the result is used

| Use | Difference |
|---|---|
| the cheque's *Amount in Words* | the text exactly as produced |
| the document's *Amount total in words* | the same text with **every comma removed** |

### 7.3 The fill rule on a printed cheque

The words are padded so that no one can add text after them:

```formula
filled = ( words + " " ) left-justified to 200 characters, padding with the asterisk character
```

An empty `words` yields an empty `filled` rather than 200 asterisks.

### 7.4 Worked example — a cheque for one thousand two hundred thirty-four point five six

*Setting.* An outgoing payment of **1 234.56** United States dollars, payment method *Checks*, English language, currency decimal places 2, unit label *Dollars*, subunit label *Cents*.

Step 1 — split.

```formula
formatted = "1234.56"
integral   = "1234"      → integer_value = 1234
fractional = "56"        → 56
```

Step 2 — is there a fractional part? `1234.56 − 1234 = 0.56`, and `round_to(dollar, 0.56) = 0.56 ≠ 0`, so the two-part form applies.

Step 3 — spell each part in English and title-case.

```formula
spell(1234) = title_case("one thousand, two hundred and thirty-four") = "One Thousand, Two Hundred And Thirty-Four"
spell(56)   = title_case("fifty-six")                                  = "Fifty-Six"
```

Title casing capitalises after the hyphen as well, because a hyphen is not a letter.

Step 4 — assemble.

```formula
check_amount_in_words = "One Thousand, Two Hundred And Thirty-Four Dollars and Fifty-Six Cents"
```

That string is 69 characters long.

Step 5 — the fill rule for the printed line.

```formula
filled = "One Thousand, Two Hundred And Thirty-Four Dollars and Fifty-Six Cents " + 130 asterisks
```

— 69 characters, then the separating space (70), then 130 asterisks to reach exactly 200.

Step 6 — the numeric amount printed beside it is the amount formatted according to the currency's own rules, that is `$ 1,234.56` for the United States dollar with the symbol before the amount.

Step 7 — the same payment shown as a **document** total in words (a different field, used on invoices rather than cheques) would read *One Thousand Two Hundred And Thirty-Four Dollars and Fifty-Six Cents* — identical except that the comma after *Thousand* is stripped.

**Variant with no cents.** For 1 234.00 dollars: `formatted = "1234.00"`, `fractional = "00"` reads as 0, and `1234.00 − 1234 = 0` is zero for the dollar, so the single-part form applies and the result is *One Thousand, Two Hundred And Thirty-Four Dollars*.

**Variant in another currency.** For 1 234.56 euros the result is *One Thousand, Two Hundred And Thirty-Four Euros and Fifty-Six Cents*.

---

## 8. Cash rounding

A cash rounding rule holds a **rounding precision** (the smallest coin, strictly positive — otherwise *Please set a strictly positive rounding value.*), a **strategy** (`biggest_tax` — *Modify tax amount*; `add_invoice_line` — *Add a rounding line*, the default), a **rounding method** (`UP`, `DOWN`, `HALF-UP` — *Nearest*, the default) and a profit and a loss account.

```formula
rounded(amount)   = round(amount, to a multiple of rounding_precision, using rounding_method)
difference(currency, amount) = round_to(currency, rounded( round_to(currency, amount) ) − round_to(currency, amount))
```

**Worked example.** Rounding precision 0.05, method *Nearest*, currency with two decimals, amount 23.91. `round_to(currency, 23.91) = 23.91`; `rounded(23.91) = 23.90`; the difference is −0.01. With the *Add a rounding line* strategy a rounding line of −0.01 is added to the document, booked to the **loss** account when it reduces what is owed and to the **profit** account otherwise.

On a purchase document the rounding line participates in the untaxed total (or in the tax total when it carries a tax distribution line), and the payable term line is computed from the cash-rounded total — see §5.2 step 6.

---

## 9. Line defaulting on a vendor bill

When a product is chosen on a bill line the following independent computations run.

### 9.1 The unit

```formula
unit = first matching supplier record's purchase unit, if any
unit = product's reference unit, otherwise
```

where "matching supplier record" means the product's supplier records filtered for the line's company and product. Only lines whose document is still a draft are recomputed.

For a sale document the unit is always the product's reference unit; the supplier-unit rule is payable-specific.

### 9.2 The label

The label is the product's **display name**; then, for a **purchase** journal, the product's **purchase description** is appended on a new line when it is set. (For a sale journal the *sale* description is used instead.) The product is read in the language of the document's partner when it has one, otherwise in the language of the line's partner.

The label is only overwritten when the line has no label yet, or when the existing label is exactly the label the previous product would have produced, or when the product actually changed. Lines of a hashed entry are never relabelled.

### 9.3 The unit price

For a purchase document:

```formula
base_price     = the product's standard price read in the document's company
base_currency  = the company currency
base_taxes     = the product's supplier taxes, filtered to the company
```

1. If the line's unit differs from the product's reference unit, convert the price between the two units.
2. If there are taxes and the document has a fiscal position, adapt the price so that the amount payable is preserved when the original taxes are replaced by the mapped ones.
3. If the document currency differs from the base currency, convert at the document's date, **without rounding**.

The result is written to the unit price. **A line marked as imported is skipped entirely**, so a decoder's price is never overwritten.

### 9.4 The taxes

For a purchase document:

```formula
candidate_taxes = the product's supplier taxes, filtered to the document's company
candidate_taxes = the purchase-typed taxes attached to the line's account, when the product has none
```

Then the candidates are filtered to the line's company and, when the document has a fiscal position, mapped through it.

The recomputation is skipped on presentation lines, on payable term lines, on cost-of-goods lines, and on **imported** lines. It is also skipped when the line has no product, is not a discount line, and there are neither account taxes nor any existing taxes — the guard exists so that clearing the account never silently clears manually chosen taxes.

### 9.5 The quantity and the account

Quantity defaults to **1** on a product line and is forced empty on every other kind. The account follows §6.2.

---

## 10. Partial deductibility

A product line of a **purchase** document carries a **deductibility** percentage, default 100. Values below 100 mean that part of the expense is private and its input tax may not be reclaimed.

The private-share lines are regenerated whenever the multiset of *(label, subtotal, taxes, deductibility, account)* over the document's product lines changes, and only while the document is a **draft purchase document** with at least one product line below 100 % deductibility.

For each such product line, with `sign` the direction sign and `rate` the document rate:

```formula
percentage                     = 1 − deductibility ÷ 100
non_deductible_subtotal        = round_to(document_currency, line_subtotal × percentage)
non_deductible_base            = round_to(document_currency, sign × non_deductible_subtotal)
non_deductible_base_converted  = round_to(company_currency, sign × non_deductible_subtotal ÷ rate)   (0 when rate is 0)
```

`rate` is the document rate, the number of units of the document currency per unit of company currency, so dividing by it converts a document-currency amount into company currency. Both quantities are computed for every partly deductible product line.

and a line is created with:

| Attribute | Value |
|---|---|
| account | **the same account as the product line** |
| display type | `non_deductible_product` |
| label | the product line's label |
| balance | `− non_deductible_base` — the amount **rounded in the document currency and not converted** |
| amount in currency | `− non_deductible_base_converted` — the amount **converted to company currency and rounded there** |
| taxes | the product line's taxes **excluding fixed-amount taxes** |
| sequence | the product line's sequence plus one |

Then one aggregate line is created:

| Attribute | Value |
|---|---|
| account | the journal's **private share account**, falling back to the journal's default account |
| display type | `non_deductible_product_total` |
| label | *private part* — replaced at posting by *«document number» - private part* |
| balance | the sum of the `non_deductible_base` values |
| amount in currency | the sum of the `non_deductible_base_converted` values |
| taxes | none |
| sequence | one more than the highest sequence on the document |

**Compatibility finding — the two currencies are exchanged on these two line kinds.** Everywhere else in this domain a journal item's *balance* holds the amount in the **company** currency and its *amount in currency* holds the amount in the **document** currency (`accounting-effects.md` §1). On the `non_deductible_product` and `non_deductible_product_total` lines the two assignments are the other way round, exactly as written in the tables above: the un-converted document-currency amount is written to the balance, and the converted company-currency amount is written to the amount in currency. Recorded as observed.

The two coincide whenever the document rate is 1 — that is, on every document written in the company currency, which is the ordinary case and the case of the worked example below — so the exchange is invisible there. On a foreign-currency bill the two figures are transposed with respect to every other line of the same entry, and the entry no longer balances in either column by itself.

A corrected behaviour would write `− non_deductible_base_converted` to the balance and `− non_deductible_base` to the amount in currency, and correspondingly for the aggregate line, so that the private-share lines follow the same convention as the product, tax, rounding and payable term lines. A rebuild that implements the correction produces identical entries for every document in the company currency, and balanced entries — rather than transposed ones — for foreign-currency documents.

Finally the tax engine produces, from the negative base lines above, a **non-deductible tax line**:

```formula
non_deductible_tax_amount = Σ over the non-deductible base lines of  − sign × ( total_included − total_excluded )
```

evaluated in both currencies, and booked to the existing non-deductible tax line's account when one exists, otherwise to the journal's private share account, otherwise to the journal's default account, with display type `non_deductible_tax` and label *private part (taxes)* — replaced at posting by *«document number» - private part (taxes)*.

The net effect: the private fraction of the expense is moved out of the expense account into the private share account, and the input tax on that fraction is moved out of the deductible tax account into the same private share account.

**Worked example.** A bill line: 1 unit of "Mobile subscription", 100.00 Euro, tax 20 % purchase, deductibility 75 %.

```formula
percentage                    = 1 − 0.75 = 0.25
non_deductible_subtotal       = round_to(Euro, 100.00 × 0.25) = 25.00
non_deductible_base           = round_to(Euro, +1 × 25.00) = 25.00
non_deductible_base_converted = round_to(Euro, +1 × 25.00 ÷ 1.00) = 25.00
```

The document is in the company currency, so the rate is 1.00 and the two quantities are equal: the exchange described in the compatibility finding above has no visible effect here.

Lines produced, beyond the ordinary expense line of 100.00 debit and the ordinary tax line of 20.00 debit:

| Display type | Account | Amount in currency | Balance |
|---|---|---|---|
| `non_deductible_product` | the expense account | −25.00 | −25.00 (credit) |
| `non_deductible_product_total` | the private share account | +25.00 | +25.00 (debit) |
| `non_deductible_tax` | the private share account | +5.00 | +5.00 (debit) |

and the deductible tax line falls from 20.00 to 15.00, because the tax engine now sees a negative base of 25.00 carrying the same 20 % tax. The payable term line is unchanged at 120.00 credit: the supplier is still owed the full amount.

**Worked example in a foreign currency, showing the observed transposition.** The same line, but the bill is written in United States dollars with a document rate of **1.25** dollars per euro; the company currency is still the euro, both currencies round to 0.01.

```formula
non_deductible_subtotal       = round_to(dollar, 100.00 × 0.25) = 25.00 dollars
non_deductible_base           = round_to(dollar, +1 × 25.00)    = 25.00 dollars
non_deductible_base_converted = round_to(euro,  +1 × 25.00 ÷ 1.25) = 20.00 euros
```

Observed assignment on the `non_deductible_product` line: **balance −25.00** and **amount in currency −20.00**. Observed assignment on the `non_deductible_product_total` line: **balance +25.00** and **amount in currency +20.00**. Both are the transpose of the convention every other line of the same entry follows, where the balance would be −20.00 euros and the amount in currency −25.00 dollars.

Under the corrected behaviour the two lines read: `non_deductible_product` balance −20.00 euros, amount in currency −25.00 dollars; `non_deductible_product_total` balance +20.00 euros, amount in currency +25.00 dollars.

---

## 11. Quick encoding of a bill

When the company's quick encoding setting covers purchase journals, a bill shows a *Total (Tax inc.)* field. Typing a total generates one line that matches it.

1. Ask §6.4 for the most frequent account and taxes of this vendor for this document type.
2. If it answered a non-zero count, use that account and those taxes.
3. Otherwise use the **journal's default account**, and as taxes: the purchase-typed taxes attached to that account; if there are none, the **company's default purchase tax**; then map the result through the fiscal position.
4. Let `remaining = quick_edit_total_amount − current tax-inclusive total of the document`.
5. If the payment term grants an early discount **in mode `mixed`** and there is exactly one tax and it is a percentage tax, the net is

```formula
price_untaxed = round_to(document_currency, remaining ÷ ( (1 − discount_percentage ÷ 100) × (tax_rate ÷ 100) + 1 ) )
```

   — because in that mode the tax is computed on the already-discounted base.
6. Otherwise the net is obtained by treating `remaining` as a tax-included amount under the chosen taxes and taking the tax-excluded total.
7. A single line is proposed with that account, those taxes and that unit price.

Afterwards a reconciliation of rounding is applied: if the resulting document total differs from the typed total, the difference is added to the **first tax group of the untaxed subtotal** so that the printed total matches exactly what was typed. Example: 100.00 typed with a 21 % included tax gives base 82.64 and tax 17.35 for a total of 99.99; the tax is nudged to 17.36.

Posting refuses a mismatch that survives this: *The current total is «current» but the expected total is «expected». In order to post the invoice/bill, you can adjust its lines or the expected Total (tax inc.).*

A separate convenience fills the bill date when quick encoding is on and no date has been typed: it takes today, unless a previous posted document of the same journal and company with a bill date exists, in which case it takes **that document's bill date passed through the accounting-date algorithm** of §2.3.

---

## 12. Abnormal amount and date detection

Both warnings are computed only for **purchase documents** that are stored, in `draft` state, with a non-zero total, and whose vendor has not silenced both warnings. They can be switched off entirely through a context flag.

### 12.1 The statistical base

For each examined draft document, look at the other **posted** documents that share its partner, its type, its company **and its currency**, whose bill date is on or before the examined document's bill date (falling back to its accounting date, then today). Order them by bill date descending, and compute, over a window of those rows:

```formula
date_diff(row_i) = bill_date(row_{i−1}) − bill_date(row_i)     (the gap to the next more recent document)
last_invoice_date   = the most recent bill date
date_diff_mean      = the mean of the gaps
date_diff_deviation = the sample standard deviation of the gaps
amount_mean         = the mean of the totals
amount_deviation    = the sample standard deviation of the totals
```

Only the window rows numbered **between 10 and 30** are kept — that is, the statistics are only used when the vendor has at least ten and at most thirty prior documents in the series. When no row qualifies, the fallbacks are: `last_invoice_date` = the examined bill date, both means 0, and both deviations 10 000 000 000 — deliberately enormous, so that no warning can fire.

### 12.2 The date warning

```formula
if date_diff_mean > 25 then date_diff_deviation ← date_diff_deviation + 1
wiggle_room_date = 2 × date_diff_deviation
```

The extra day exists because months have different lengths: a strictly monthly supplier produces a mean around 30.5 days with a deviation around 1 day, and February would otherwise trip the alarm.

The warning fires when the vendor has not silenced date warnings **and**

```formula
( examined_bill_date − last_invoice_date ) in days  <  integer_part( date_diff_mean − wiggle_room_date )
```

Its text is:

> The billing frequency for «vendor display name» appears unusual. Based on your historical data, the expected next invoice date is not before «last invoice date + (date_diff_mean − wiggle_room_date) days» (every «date_diff_mean» (± «wiggle_room_date») days).
> Please verify if this date is accurate.

It is a single sentence: there is **no** full stop between the date and the opening parenthesis, and the only line break is the one before *Please verify*. The date is rendered in the reader's date format; the two numbers are rendered as integers (the fractional parts of the mean and of the allowance are dropped, not rounded).

### 12.3 The amount warning

```formula
wiggle_room_amount = 2 × amount_deviation
```

The warning fires when the vendor has not silenced amount warnings **and** the total is **outside** the closed interval

```formula
[ amount_mean − wiggle_room_amount , amount_mean + wiggle_room_amount ]
```

Its text is:

> The amount for «vendor display name» appears unusual. Based on your historical data, the expected amount is «amount_mean» (± «wiggle_room_amount»).
> Please verify if this amount is accurate.

with both amounts formatted in the document currency.

### 12.4 Effects

Both texts appear as **warning**-level banners. An abnormal-amount warning also **prevents automatic posting** of the bill. The mass-posting wizard lists the affected vendors and offers to silence the warning permanently for each of them.

---

## 13. Cheque numbering

### 13.1 The journal's cheque sequence

Each journal owns a cheque sequence with no-gap implementation, padding 5 and increment 1. The journal exposes the **next number**:

```formula
check_next_number = the sequence's next value, rendered with the sequence's padding
check_next_number = 1, when the journal has no cheque sequence
```

The value is **not stored**, and its computation depends on the journal's *Manual Numbering* flag and on nothing else. So the displayed preview is recomputed when that flag is written, and **not** when a cheque consumes the sequence: a value read from a record that has been in memory since before a cheque was numbered is stale until the record is read again. Writing the field is the inverse described in §13.3.

Writing that field:

1. Refuse a value containing anything but digits: *Next Check Number should only contains numbers.*
2. Refuse a value lower than the sequence's current next value: *The last check number was «current next value». In order to avoid a check being rejected by the bank, you can only use a greater number.*
3. Refuse a value above **2 147 483 647**: *The check number you entered («value») exceeds the maximum allowed value of 2147483647. Please enter a smaller number.*
4. Otherwise set the sequence's next value to the number and its padding to the **length of the text that was written** — so writing `000100` sets a padding of 6.

### 13.2 The number on a payment

Displayed value while drafting:

```formula
check_number = the journal's cheque sequence next value, rendered with its padding,
               when the journal uses manual numbering and the payment method code is check_printing
check_number = empty, otherwise
```

At **posting**, for every payment whose method is the cheque method **and** whose journal uses manual numbering, the sequence is actually **consumed** and the drawn value is written to the payment.

Writing a number directly sets the journal cheque sequence's padding to the length of the written text.

### 13.3 The pre-printed path

For a journal that does **not** use manual numbering, the numbers come from the stationery. When printing is requested:

```formula
last = the greatest existing check_number of the journal, compared as an integer
width = the number of characters of that value (0 when there is none)
next  = ( last + 1 ) rendered with leading zeros to exactly `width` characters
```

That value pre-fills the wizard. On confirmation, with *k* the entered value read as an integer and *w* the number of characters entered:

1. Every selected draft payment is posted.
2. Every selected payment that is `in_process` and not yet sent is marked as sent.
3. The payments are numbered in recordset order: the first gets *k* rendered with leading zeros to *w* characters, the second *k+1*, and so on.
4. The printable document is produced, and the dialogue closes when the download completes.

### 13.4 Uniqueness

Two constraints protect the numbers.

- **Digits only**: a number containing anything but decimal digits is refused with *Check numbers can only consist of digits*.
- **Unique per journal among posted payments**: two payments in the same journal whose numbers are equal **when compared as integers** — so `0042` clashes with `42` — and whose journal entries are both `posted`, are refused with

> The following numbers are already used:
> «number» in journal «journal display name»

one line per clashing pair. Because the comparison is restricted to posted entries, a cancelled (voided) cheque does not block its own number from a database point of view; the audit trail keeps it visible.

---

## 14. The cheque stub

The stub is the detachable summary of what the cheque pays. Exactly **nine** stub lines fit on a page.

### 14.1 Gathering the paid documents

If the payment has a journal entry (the normal case, after posting):

1. Take the payment's own receivable and payable lines (the *counterpart* lines).
2. Follow their reconciliations in both directions to reach the counterpart entries.
3. Keep only the entries that are **outbound documents, receipts included** — that is, vendor bills, purchase receipts and customer credit notes.
4. Group the partial reconciliations by the entry they touch.

If the payment has no journal entry yet, take the documents explicitly linked to the payment, keep the outbound ones, and walk them in order consuming the payment amount (see §14.3).

### 14.2 One stub line

For a document *I*, with `current_amount` the amount of this payment allocated to it:

```formula
invoice_sign = +1  when I is outbound or a purchase receipt
invoice_sign = −1  otherwise
```

| Column | Value |
|---|---|
| due date | *I*'s due date, formatted |
| number | *I*'s number, or `/`; when *I* has a vendor reference, the number and the reference joined by ` - ` |
| total | `invoice_sign × I.amount_total`, formatted in *I*'s currency |
| residual | `invoice_sign × ( I.amount_residual − current_amount )` formatted in *I*'s currency; the single character `-` when that value is zero in *I*'s currency |
| paid | `invoice_sign × amount_paid` formatted in **the payment's** currency, where `amount_paid` is `current_amount` when one was given, and otherwise the sum over the reconciliations of the debit amount in currency (for an outbound document) or the credit amount in currency (otherwise) |

### 14.3 Allocation when the payment has no entry

```formula
remaining ← payment amount
for each document I, ordered by due date and then by accounting date:
    current_amount = min( remaining , convert(I.amount_residual, from I.currency to payment currency) )
    emit a stub line for I with that current_amount
    remaining ← remaining − current_amount
stop when remaining reaches zero or documents run out
```

### 14.4 Grouping and headers

Documents are grouped into two buckets: **Bills** (types `in_invoice` and `in_receipt`) and **Refunds** (type `out_refund`). Within each bucket they are ordered by due date, falling back to the accounting date. When **both** buckets are non-empty, a header line carrying the bucket name is emitted before each bucket.

### 14.5 Paging

Let `INV_LINES_PER_STUB` be **9**.

**Single-page mode** (the company's multi-page stub option is off): if there are more than 9 stub lines, keep only the first **8** — leaving room for an ellipsis line — otherwise keep up to 9. Exactly one page is produced, and the page is flagged as **cropped** when the payment's entry reconciles more than 9 documents.

**Multi-page mode**: walk the stub lines from the start; at each step take 9 lines, **unless** the ninth line of the prospective page would be a bucket **header**, in which case take only 8 so that a bucket never starts at the very bottom of a page. Repeat until every line is placed.

### 14.6 One printed page

For page *i* (zero-based) carrying stub lines *p*:

| Field | Value |
|---|---|
| sequence number | the cheque number |
| manual sequencing | whether the journal uses manual numbering |
| date | the payment date, formatted |
| partner and partner name | the payee |
| company | the company name |
| currency | the payment currency |
| state | the payment status |
| amount | on page 0, the payment amount formatted in the payment currency; **on every later page, the literal text `VOID`** |
| amount in words | on page 0, the filled amount-in-words line of §7.3; **on every later page, the literal text `VOID`** |
| memo | the payment memo |
| stub cropped | the flag of §14.5: true when the multi-page stub option is **off** and the payment's entry reconciles **more than nine documents**. It is not a statement that lines were dropped — see the note below |
| stub lines | *p* |

So when a stub spills over several pages, **only the first page carries a negotiable cheque**; the others are explicitly voided.

**What the cropped flag counts, and what it does not.** The flag counts **documents reconciled by the payment's entry**. The stub, however, is a list of **lines**, and it carries a bucket header line before each of the *Bills* and *Refunds* buckets whenever both buckets are non-empty (§14.4). The two counts therefore differ by the number of header lines emitted. A payment that settles exactly nine documents split over both buckets produces eleven stub lines, of which single-page mode keeps only eight — three lines are dropped — yet the flag stays false, because nine is not more than nine. Conversely a payment settling ten documents in one bucket produces ten lines, drops two and sets the flag. A layout must therefore treat the flag as "the payment settled more documents than a single page is meant to show", not as "lines were dropped", and a rebuild that wants an ellipsis on every cropped page has to compare the number of stub lines it kept with the number it had.

---

## 15. The invoice analysis report

Every column of the analysis report, together with its sign conventions, its unit conversion and its currency conversion, is specified in `entities.md` §3. Two derivations deserve to be repeated here as formulas.

### 15.1 The weighted average price

The per-row column is

```formula
price_average = − ( ( balance ÷ quantity ) × purchase_sign ÷ factor(line_unit) × factor(template_unit) ) × conversion_rate
```

and the **aggregated** value over any group is deliberately not the mean of that column but

```formula
aggregated_average = Σ price_subtotal ÷ Σ quantity        (0 when Σ quantity is 0)
```

**Worked example.** Two bill lines for the same product, whose reference unit is *Units* and whose purchase unit is *Dozens* (factor such that one dozen contains twelve units).

| Line | Quantity | Unit | Balance (company currency) |
|---|---|---|---|
| 1 | 3 | Dozens | 360.00 debit |
| 2 | 1 | Dozens | 130.00 debit |

Assume `factor(Dozens) ÷ factor(Units)` restates one dozen as 12 units, the conversion rate is 1 and the purchase sign for a bill is −1.

```formula
quantity(line 1) = 3 × 12 × (−1) = −36
quantity(line 2) = 1 × 12 × (−1) = −12
price_subtotal(line 1) = −360.00 × 1 = −360.00
price_subtotal(line 2) = −130.00 × 1 = −130.00
aggregated_average = (−360.00 + −130.00) ÷ (−36 + −12) = −490.00 ÷ −48 = 10.2083…
```

that is, 10.21 currency units per **unit** bought — not per dozen.

### 15.2 The margin

```formula
price_margin = 0                                                                  for any purchase type
price_margin = conversion_rate × ( − balance + converted_quantity × cost )        for out_refund
price_margin = conversion_rate × ( − balance − converted_quantity × cost )        for out_invoice and out_receipt
```

with `converted_quantity = quantity × factor(line_unit) ÷ factor(template_unit)` and `cost` the product's standard price stored for the line's company (absent reads as 0). Bills therefore never contribute a margin; they contribute only to the untaxed amount, the total and the inventory value.

---

## 16. Intercompany clearing amounts

When a bill of company *A* is paid through a payment transaction belonging to company *B* of the same group, two clearing entries are built (see `accounting-effects.md` §8). Their amounts are:

For the **payer's** clearing entry, per settling payment:

```formula
clearing_balance          = − Σ balance over the payment's counterpart lines
clearing_amount_currency  = − Σ amount_in_currency over the payment's counterpart lines
```

For the **biller's** settlement entry:

```formula
total_invoice_balance          = Σ balance over the document's receivable and payable lines
clearing_counterpart_balance   = − total_invoice_balance
```

Both entries are strictly balanced by construction: each has exactly two lines carrying opposite amounts.

---

## 17. Precision reference

| Quantity | Precision used |
|---|---|
| line unit price | the *Product Price* decimal precision, with a minimum display precision |
| line quantity | the *Product Unit* decimal precision |
| line discount | the *Discount* decimal precision |
| payment term percentages | the *Payment Terms* decimal precision (used by the "must sum to 100" constraint) |
| every monetary amount in document currency | the document currency's rounding factor |
| every monetary amount in company currency | the company currency's rounding factor |
| deductibility comparisons | two decimal digits |
| cash rounding | the rule's own rounding precision, then the currency's rounding factor |
| currency conversion of a product price | performed **without** rounding, then rounded only when stored |

---

## 18. The purchase journal dashboard numbers

Each purchase journal card shows four counted-and-summed groups plus three indicators. All of them are computed in one pass over four queries.

### 18.1 The four groups

| Group | Query |
|---|---|
| **Drafts** | documents of the journal, in the user's companies, whose status is `draft` and whose type is any invoice-like type, receipts included |
| **Waiting** (to pay) | documents of the journal, posted, whose payment status is `not_paid` or `partial`, whose type is one of the three purchase types |
| **Late** | the same query as *Waiting*, restricted to the rows whose due date is strictly before today |
| **To review** | documents of the journal, posted, whose reviewed flag is false — **no type restriction**, so a miscellaneous entry of a purchase journal also counts |

Both the *Waiting* and the *Late* numbers come from a **single** query whose result rows carry a late indicator; the two groups are then the rows where that indicator is true and the rows where the "to pay" indicator is true.

### 18.2 What each row carries

For the drafts query the selected columns are: the journal; a document-currency total signed as

```formula
row_amount = amount_total × ( −1 when the type is out_refund or in_refund, otherwise +1 )
```

and a company-currency total signed as

```formula
row_amount_company = amount_total_signed × ( −1 when the type is in_invoice, in_refund or in_receipt, otherwise +1 )
```

plus the currency, the type, the bill date and the company.

For the *Waiting and Late* query the selected columns are: the journal, the company, the currency, the late indicator (*due date is before today*), and two grouped sums:

```formula
group_amount_company = Σ amount_residual_signed
group_amount         = Σ ( amount_residual × ( −1 when the type is in_invoice, otherwise +1 ) )
```

together with a row count. The grouping is by company, journal, currency, late indicator and the "to pay" indicator.

For the *To review* query the same shape is used but over the **totals** rather than the residuals:

```formula
group_amount_company = Σ amount_total_signed
group_amount         = Σ ( amount_total × ( −1 when the type is in_invoice, otherwise +1 ) )
```

### 18.3 Converting the rows into one number

Given a target currency — the journal's currency when it has one, otherwise the journal's company's currency — the counter and the sum are built as:

```formula
count = Σ over rows of the row's count (1 when absent)
```

```formula
sum = Σ over rows of:
        row_amount_company                                      when the row's company currency equals the target currency
        convert( row_amount, row_currency → target, at the row's bill date or today )   otherwise
```

and the result is rounded to the target currency.

### 18.4 Display

The purchase card then shows the title *Bills to pay*, the four counts, and the formatted sums. **The waiting and late sums are negated** for a purchase journal (they are shown as written for a sale journal), so that an amount owed to suppliers reads as a positive figure. The draft sum is not negated, because the query already signed it.

Three further indicators:

| Indicator | Meaning |
|---|---|
| irregular sequences | the journal has at least one numbering hole; the hint reads *Irregularities due to draft, cancelled or deleted bills with a sequence number since last lock date.* |
| unhashed entries | the journal secures entries with a hash and at least one posted entry is not yet hashed |
| sample data | the journal has **no** document at all; the card then offers the sample bill |

---

## 19. The numbering starting pattern

The pattern from which a journal's first number is derived is assembled arithmetically. Let *d* be the accounting date, falling back to the bill date, falling back to today; let *L* be the company's fiscal year end day and *M* its fiscal year end month.

```formula
staggered = ( M ≠ 12 ) or ( L ≠ 31 )
```

If the year is **not** staggered:

```formula
year_part = the four digits of the year of d
```

If it **is** staggered, first clamp the end day to the length of that month:

```formula
L' = min( L , number_of_days( year of d , M ) )
```

then

```formula
year_part = "«last two digits of the year of d»-«last two digits of the year of d plus one»"   when d > date( year of d , M , L' )
year_part = "«last two digits of the year of d minus one»-«last two digits of the year of d»"  otherwise
```

The pattern is then:

| Journal type | Pattern |
|---|---|
| sale, bank, cash, credit card | `«code»/«year part»/00000`, or `«code»/«year part»/0000` when the year is staggered |
| any other, **including purchase** | `«code»/«year part»/«two-digit month of d»/0000` |
| any other, self-billing | `«code»«partner identifier padded to five with zeros»/«year part»/«two-digit month of d»/0000` |

then prefixed with `R` for a credit note in a journal with a separate refund sequence, with `P` for a payment's entry in a journal with a separate payment sequence, and with `D` for a debit note in a journal with a dedicated debit note sequence.

**Worked example.** Company with a fiscal year ending 30 June; purchase journal coded `BILL`; a bill whose accounting date is 12 September 2026.

```formula
staggered = ( 6 ≠ 12 ) → true
L' = min( 30 , 30 ) = 30
date( 2026 , 6 , 30 ) = 30 June 2026;  12 September 2026 > that
year_part = "26-27"
pattern   = "BILL/26-27/09/0000"
```

so the first bill of that month is numbered `BILL/26-27/09/0001`.

**Second worked example.** Same company, a bill dated 3 March 2026.

```formula
3 March 2026 ≤ 30 June 2026
year_part = "25-26"
pattern   = "BILL/25-26/03/0000"
```

**Third worked example.** A self-billing purchase journal coded `SB`, commercial partner identifier 412, calendar fiscal year, bill dated 5 May 2026:

```formula
year_part          = "2026"
partner_identifier = "00412"
pattern            = "SB00412/2026/05/0000"
```

---

## 20. File grouping arithmetic

### 20.1 The similarity score

```formula
similarity( name_1 , name_2 ) = the length of the longest common contiguous substring of the two names
```

computed without any "junk character" heuristic, so that every character counts.

**Worked example.** Three files arrive in one message: `INV-2026-0044.pdf`, `INV-2026-0044.xml` and `scan_44.jpg`.

- The first two have different format labels (Portable Document Format versus none, since only the Portable Document Format label is recognised by the base implementation), so they do not clash.
- Their similarity is the length of `INV-2026-0044.` = **14**.
- `scan_44.jpg` shares at most `4` characters with either (`_44.` versus `44.`, giving 3, or `.` giving 1); the longest common substring is `44.` = 3.

So the grouping places the first two together and, because the image also does not clash by format label with that group, the image joins the group with the highest similarity — which is still that same group. One bill is created carrying all three files. This is the intended outcome: one supplier document delivered in three representations.

### 20.2 When files do clash

**Worked example.** Five files `a.pdf`, `b.pdf`, `c.pdf`, `d.pdf`, `e.pdf` arrive in one message. Each carries the Portable Document Format label. The first file starts a group. The second finds that the only existing group already holds a file of that label, so it starts its own. And so on: five groups, five bills.

### 20.3 Ordering

Files are placed in **decreasing decoder priority**, so that the file most likely to be decodable anchors each group; files with no decoder are placed last and therefore attach themselves to an existing group rather than creating one, whenever their format label allows.
