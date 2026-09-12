# General Ledger — Calculations and Algorithms

Every formula and every algorithm of the domain, with its rounding rule, its precision, its currency, its evaluation order and at least one worked numeric example.

Contents:

1. [Rounding conventions](#1-rounding-conventions)
2. [Balance, debit and credit](#2-balance-debit-and-credit)
3. [The foreign-currency amount of a journal item](#3-the-foreign-currency-amount-of-a-journal-item)
4. [The balance invariant](#4-the-balance-invariant)
5. [The direction sign and the amount totals of an entry](#5-the-direction-sign-and-the-amount-totals-of-an-entry)
6. [The cumulated balance of a list of items](#6-the-cumulated-balance-of-a-list-of-items)
7. [Lock dates: the effective value and the violation test](#7-lock-dates-the-effective-value-and-the-violation-test)
8. [The accounting date rule](#8-the-accounting-date-rule)
9. [The numbering grammar](#9-the-numbering-grammar)
10. [Deducing the reset periodicity](#10-deducing-the-reset-periodicity)
11. [Extracting the format from a previous number](#11-extracting-the-format-from-a-previous-number)
12. [Finding the previous number of a chain](#12-finding-the-previous-number-of-a-chain)
13. [The starting number of a fresh chain](#13-the-starting-number-of-a-fresh-chain)
14. [Assigning the next number under concurrency](#14-assigning-the-next-number-under-concurrency)
15. [Chain-end tests and gap detection](#15-chain-end-tests-and-gap-detection)
16. [The resequencing algorithm](#16-the-resequencing-algorithm)
17. [The inalterability hash chain](#17-the-inalterability-hash-chain)
18. [The reconciliation algorithm](#18-the-reconciliation-algorithm)
19. [Exchange differences at reconciliation](#19-exchange-differences-at-reconciliation)
20. [Full-reconcile detection and matching numbers](#20-full-reconcile-detection-and-matching-numbers)
21. [The fiscal year](#21-the-fiscal-year)
22. [The opening entry](#22-the-opening-entry)
23. [Partner ledger figures](#23-partner-ledger-figures)
24. [Account code generation](#24-account-code-generation)
25. [Journal dashboard figures](#25-journal-dashboard-figures)
26. [The automatic transfer amounts](#26-the-automatic-transfer-amounts)
27. [The order in which numbers are assigned](#27-the-order-in-which-numbers-are-assigned)
28. [The proposed number shown before posting](#28-the-proposed-number-shown-before-posting)
29. [Numbering holes per journal](#29-numbering-holes-per-journal)
30. [The display name of an entry](#30-the-display-name-of-an-entry)
31. [Which items are reportable for tax](#31-which-items-are-reportable-for-tax)
32. [Whether a journal item affects the tax report](#32-whether-a-journal-item-affects-the-tax-report)
33. [The dashboard figures of a liquidity journal](#33-the-dashboard-figures-of-a-liquidity-journal)
34. [Worked end-to-end example](#34-worked-end-to-end-example)

---

## 1. Rounding conventions

Every monetary quantity is rounded to the **rounding step** of its currency. A currency carries a rounding step (for example 0.01, or 0.05 for a currency whose smallest coin is five cents, or 1 for a currency with no subdivision) and a derived number of decimal places.

The rounding rule is *round half away from zero on the rounding step*:

```formula
rounded_value = sign( value ) × floor( | value | ÷ rounding_step + 0.5 ) × rounding_step
```

with the refinement that the intermediate division is performed in a way that avoids binary representation error: the value is first normalised by the inverse of the rounding step when that step is smaller than one, rounded as an integer, and then denormalised. The complete arithmetic, the alternative methods (always away from zero, always towards zero, half towards zero, half to even) and the comparison and zero tests are specified in `../multi-currency/`.

Three derived operations are used constantly in this domain:

| Operation | Definition |
|---|---|
| *is zero* for a currency | the absolute value, rounded to the rounding step of that currency, is zero |
| *compare* two amounts for a currency | round the difference to the rounding step; return −1, 0 or +1 according to its sign |
| *round* for a currency | as above |

Unless a formula says otherwise:

- amounts in the **company currency** (the balance, the debit, the credit, every "signed" total, the matched amount of a partial reconciliation) are rounded to the company currency;
- amounts in the **document currency** or the **item currency** are rounded to that currency;
- the **percentage** of an analytic distribution is handled with two decimals;
- **quantities** use the product-unit precision, which belongs to `../units-of-measure-and-packaging/`.

---

## 2. Balance, debit and credit

The stored triple of a Journal Item is (`balance`, `debit`, `credit`). The **balance** is the authoritative value: it is the field written, and the two others are derived from it.

Normal accounting (the company does **not** use storno):

```formula
debit  = balance          when balance > 0, otherwise 0
credit = − balance        when balance < 0, otherwise 0
```

Storno accounting (the item is flagged as storno):

```formula
debit  = balance          when balance < 0, otherwise 0
credit = − balance        when balance > 0, otherwise 0
```

so that a reversal appears as a negative debit on the original debit side rather than as a credit.

The inverse direction also exists, because a user may type into the debit or the credit box:

```formula
balance = debit − credit
```

and typing a negative number into either box sets the storno flag. When a creation supplies a debit or a credit and no balance, the balance written is the difference of the two and the two supplied values are discarded.

**Worked example.** A miscellaneous entry books 1 250.00 in the company currency to an expense account and the same to a payable account.

| Item | balance | debit | credit |
|---|---|---|---|
| Expense | +1 250.00 | 1 250.00 | 0.00 |
| Payable | −1 250.00 | 0.00 | 1 250.00 |

Reversing it normally gives balances −1 250.00 and +1 250.00, that is a credit of 1 250.00 on the expense account. Under storno accounting the reversal instead gives a **debit of −1 250.00** on the expense account, so that the gross turnover of the expense account stays correct.

### Default balance of a new item of a miscellaneous entry

When a new item is added to a non-invoice entry and no amount is supplied, the proposed balance is the amount that would make the entry balanced:

```formula
proposed_balance = − ( sum of the balances of every other item of the entry )
```

For an invoice-like document the proposed balance is zero, because the amount comes from the price and the quantity.

---

## 3. The foreign-currency amount of a journal item

Every item carries a currency and an amount in that currency. When the item currency is the company currency the two amounts are equal; otherwise they are linked by a rate.

### The rate of an item

```formula
currency_rate = invoice_currency_rate                      when the entry is an invoice-like document
currency_rate = conversion_rate( company_currency → item_currency, at the rate date )   otherwise
currency_rate = 1                                          when the item has no currency
```

The rate date is the document date when the entry has one, otherwise the accounting date, otherwise today. An invoice-like document with no stored rate uses 1.

### Deriving one amount from the other

```formula
amount_currency = round_to_item_currency( balance × currency_rate )
```

and, in the inverse direction (the user typed the foreign amount):

```formula
balance = round_to_company_currency( amount_currency ÷ currency_rate )
```

When the item currency equals the company currency and the entry is not an invoice-like document, the foreign amount is forced to be exactly equal to the balance.

**Worked example.** Company currency is the euro (rounding 0.01). An item in United States dollars (rounding 0.01) is entered for 1 000.00 dollars at a rate of 1.10 dollars per euro.

```formula
balance = round_to_euro( 1 000.00 ÷ 1.10 ) = round_to_euro( 909.0909… ) = 909.09
```

and the stored triple is balance 909.09, debit 909.09, credit 0.00, foreign amount 1 000.00, currency United States dollar.

### Sign coherence

A database check refuses any accountable item whose balance and foreign amount have opposite signs. Both may be zero, and one may be zero while the other is not (this is exactly what an exchange-difference item looks like).

---

## 4. The balance invariant

Every Journal Entry must balance in the **company currency**, at the precision of the company currency. The check runs around every creation, every write and every deletion of an entry or of its items, and around every reconciliation, and it can be suspended only by an explicit internal flag.

```formula
for each entry:  round_to_company_currency( sum over its items of balance ) = 0
```

The check is deliberately expressed on the stored balances and not on any computed field, so that it is valid even in the middle of a recomputation. Equivalently, and using the two derived columns:

```formula
round_to_company_currency( sum of debit ) = round_to_company_currency( sum of credit )
```

Failure messages:

- one entry: "The entry is not balanced."
- several entries: "The following entries are unbalanced:" followed by one line per entry, indented by two spaces and a hyphen, carrying the entry number.

There is **no** invariant in the document currency: an entry may perfectly well be unbalanced in a foreign currency, and that imbalance is exactly what an exchange-difference entry later corrects.

**Worked example of a tolerated foreign imbalance.** A payment of 1 000.00 dollars at 1.10 is matched against an invoice of 1 000.00 dollars booked at 1.05. In euros the two items are 909.09 and 952.38; the difference of 43.29 euros is booked by an exchange-difference entry, and each entry on its own balances in euros.

---

## 5. The direction sign and the amount totals of an entry

### The direction sign

```formula
direction_sign = +1    when the document type is a plain entry, or the document is outbound
direction_sign = −1    otherwise (the document is inbound)
```

Outbound documents are vendor bills, purchase receipts and customer credit notes. Inbound documents are customer invoices, sales receipts and vendor credit notes.

### The totals

The computation walks the items once and accumulates four pairs of running totals — untaxed, tax, grand total and residual — each in both the company currency (from the balance) and the document currency (from the foreign amount).

For an **invoice-like** document:

| Item display type | Contributes to |
|---|---|
| tax, non-deductible tax, or rounding produced by a tax distribution line | tax **and** grand total |
| product, rounding without a tax distribution line, non-deductible product, non-deductible product total | untaxed **and** grand total |
| payment term | residual only, and using the **residual amounts** of the item, not its balance |
| section, subsection, note, and every other type | nothing |

For a **plain entry**, only items with a non-zero debit contribute, and they contribute to the grand total only:

```formula
total_company  = sum of balance over items whose debit is non-zero
total_document = sum of amount_currency over items whose debit is non-zero
```

The published fields are then:

```formula
amount_untaxed                    = direction_sign × total_untaxed_document
amount_tax                        = direction_sign × total_tax_document
amount_total                      = direction_sign × total_document
amount_residual                   = − direction_sign × total_residual_document
amount_untaxed_signed             = − total_untaxed_company
amount_untaxed_in_currency_signed = − total_untaxed_document
amount_tax_signed                 = − total_tax_company
amount_total_signed               = | total_company |            for a plain entry
amount_total_signed               = − total_company               otherwise
amount_residual_signed            = total_residual_company
amount_total_in_currency_signed   = | amount_total |              for a plain entry
amount_total_in_currency_signed   = − ( direction_sign × amount_total )   otherwise
```

**Worked example — customer invoice.** One product line of 100.00 with a 21 % tax, in the company currency.

| Item | display type | balance | foreign amount |
|---|---|---|---|
| Product | product | −100.00 | −100.00 |
| Tax | tax | −21.00 | −21.00 |
| Receivable | payment term | +121.00 | +121.00 |

Running totals: untaxed −100.00, tax −21.00, grand total −121.00, residual +121.00 (nothing matched yet). The direction sign of a customer invoice is −1, so:

- `amount_untaxed` = −1 × (−100.00) = 100.00
- `amount_tax` = 21.00
- `amount_total` = 121.00
- `amount_residual` = −(−1) × 121.00 = 121.00
- `amount_total_signed` = −(−121.00) = 121.00
- `amount_untaxed_signed` = −(−100.00) = 100.00

**Worked example — vendor bill.** The same figures with the opposite item signs. The direction sign is +1, the items are product +100.00, tax +21.00 and payable −121.00. Then `amount_untaxed` = 100.00, `amount_total` = 121.00 and `amount_total_signed` = −121.00: the signed company total of a purchase document is negative, which is what makes a purchase reduce the signed turnover in the reports.

**Worked example — plain entry.** Two items: +1 250.00 on an expense account and −1 250.00 on a payable account. Only the debit item counts, so the grand total is 1 250.00, the direction sign is +1, `amount_total` is 1 250.00 and `amount_total_signed` is the absolute value 1 250.00.

---

## 6. The cumulated balance of a list of items

The running total shown in a list of journal items is computed by the database over the **current ordering and the current filter**, not over the whole ledger:

```formula
cumulated_balance( item at position k ) = sum of balance over the items at positions 1 … k
```

where the positions follow the **reverse** of the displayed ordering, so that the running total reads correctly from the bottom of the list upwards. The value is computed only when the list explicitly requests it; otherwise it is zero.

**Worked example.** Items displayed in descending date order with balances +100, −40, +25 (newest first). Reversing the order gives +25, −40, +100, whose running totals are 25, −15, 85. Displayed newest first the column reads 85, −15, 25: the newest row shows the closing balance.

---

## 7. Lock dates: the effective value and the violation test

### The five lock dates

| Name | Applies to |
|---|---|
| Global Lock Date | every entry |
| Sales Lock Date | entries in a sale journal |
| Purchase Lock Date | entries in a purchase journal |
| Tax Return Lock Date | entries that affect the tax report |
| Hard Lock Date | every entry; no exception possible |

An entry *affects the tax report* when at least one of its items carries a tax, is a tax line, or carries at least one tax grid.

### The effective value of a soft lock date for a user

```
1. Start with the minimal representable date.
2. For every company in the chain from the company of the entry up to the root company:
   a. If that company has no value for the lock date, skip it.
   b. Otherwise look for an active exception of that company, for that lock date field,
      whose beneficiary is the acting user or nobody, and whose relaxed value is strictly
      smaller than the company value. Among the candidates take the one with the smallest
      relaxed value, treating "no value at all" as the smallest.
   c. If such an exception exists, take its relaxed value (or the minimal date when the
      exception removes the lock date entirely); otherwise take the company value.
   d. Keep the maximum of the running value and the value just obtained.
3. The result is the effective lock date.
```

The same computation performed with exceptions ignored gives the *regular* lock date.

### The effective hard lock date

```formula
effective_hard_lock_date = maximum over the company and all its ancestors of their hard lock date
```

No exception is consulted.

### The violation test for one soft lock date

```
1. Compute the regular lock date (exceptions ignored).
2. If the tested date is strictly after it, there is no violation.
3. Otherwise compute the effective lock date (exceptions applied).
4. If the tested date is strictly after the effective lock date, there is no violation:
   the exception covers the user.
5. Otherwise the violation is the effective lock date.
```

The comparison is "on or before": a date exactly equal to the lock date **is** locked.

### The violation list of an entry

```
1. Collect the violations of the Global Lock Date, of the Sales Lock Date when the journal
   is a sale journal, of the Purchase Lock Date when the journal is a purchase journal, and
   of the Tax Return Lock Date when the entry affects the tax report.
2. Add the Hard Lock Date when the tested date is on or before the effective hard lock date.
3. Sort the pairs (date, lock name) chronologically.
```

The list is rendered for the user as a comma-and-"and" list of "*label* (*date*)" items, sorted chronologically.

### The fiscal lock date used for deletions and copies

```formula
fiscal_lock_date( journal ) = max( effective Global Lock Date , effective Hard Lock Date )
fiscal_lock_date( sale journal )     = max( previous , effective Sales Lock Date )
fiscal_lock_date( purchase journal ) = max( previous , effective Purchase Lock Date )
```

**Worked example.** The company sets the Global Lock Date to 31 December 2025 and the Hard Lock Date to 30 June 2025. A bookkeeper has an active exception relaxing the Global Lock Date to 30 November 2025 until the fifth of February.

- For that bookkeeper on the third of February, an entry dated 15 December 2025 is refused only by the hard lock test? No: the effective Global Lock Date is 30 November 2025 and 15 December is after it, so no Global violation; the effective Hard Lock Date is 30 June 2025 and 15 December is after it, so no hard violation. The entry is accepted.
- The same entry recorded by a colleague without an exception: the effective Global Lock Date is 31 December 2025, the date 15 December is on or before it, and the entry is refused.
- An entry dated 20 June 2025 is refused for everybody, because 20 June is on or before the Hard Lock Date and no exception can relax it.

---

## 8. The accounting date rule

When a document is recorded in the past — either because the user typed a past document date or because the period is locked — the accounting date is pushed forward so that the numbering stays increasing and no closed period is touched.

```
Input: a candidate date, whether the entry affects the tax report, and optionally
       the already computed list of violated lock dates.

 1. Compute the violated lock dates for the candidate date, unless they were given.
 2. Let TODAY be the current date in the user time zone.
 3. Let LAST be the highest number already used in the numbering chain of this entry,
    falling back to the relaxed search (a number of a previous period) when the current
    period has none.
 4. Deduce the reset periodicity from LAST.
 5. If at least one lock date is violated, replace the candidate date with the day after
    the latest violated lock date.
 6. If the document is a sale document:
      a. If no lock date was violated, return the candidate date unchanged.
      b. If there is no LAST at all, or the periodicity is monthly:
         the result is the earlier of TODAY and the last day of the month of the candidate date.
      c. If the periodicity is yearly:
         the result is the earlier of TODAY and the last day of the year of the candidate date.
      d. Otherwise return the candidate date.
 7. Otherwise (purchase document, plain entry, anything else):
      a. If there is no LAST at all, or the periodicity is monthly or year-range-monthly:
         - if (year, month) of TODAY is later than (year, month) of the candidate date,
           the result is the last day of the month of the candidate date;
         - otherwise return the later of the candidate date and TODAY.
      b. If the periodicity is yearly:
         - if the year of TODAY is later than the year of the candidate date,
           the result is the thirty-first of December of the candidate year;
         - otherwise return the later of the candidate date and TODAY.
      c. Otherwise return the candidate date.
```

**Worked example 1 — a late vendor bill.** Today is 10 March. A vendor bill is entered with the document date 20 January. The purchase journal numbers monthly and already has January numbers. No lock date is violated. Step 7a applies: March is later than January, so the accounting date becomes 31 January. The bill is numbered in the January chain and lands in the January period.

**Worked example 2 — a vendor bill of the current month.** Today is 10 March, the document date is 3 March. Step 7a applies, (March, 2026) is not later than (March, 2026), so the accounting date is the later of 3 March and 10 March, that is **10 March**. The bill is dated today, not at its document date, because the March numbering is already in use.

**Worked example 3 — a locked period.** Today is 10 March. The Global Lock Date is 31 January. A vendor bill with document date 20 January is entered. Step 5 moves the candidate to 1 February; step 7a then sees that March is later than February and returns 28 February (or 29 in a leap year).

**Worked example 4 — a customer invoice in the past, nothing locked.** Today is 10 March, the invoice date is 20 January, the sale journal numbers yearly. No lock date is violated, so step 6a returns 20 January unchanged: sale documents keep their own date unless a lock forces a move.

**Worked example 5 — a customer invoice in a locked period.** Same as above but the Global Lock Date is 31 January. Step 5 moves the candidate to 1 February; step 6c (yearly periodicity) returns the earlier of 10 March and 31 December, that is **10 March**.

### The warning message shown before posting

When a violation exists, the form shows:

> The date is being set prior to: *the list of lock dates*. The Journal Entry will be accounted on *the resulting accounting date* upon posting.

---

## 9. The numbering grammar

A document number is decomposed into named parts. Written as plain patterns over characters, with "digit" meaning a character from zero to nine and "non-digit" meaning any character that is not a digit:

| Part | Content |
|---|---|
| first prefix | any characters, as few as possible |
| year | either four digits beginning with 19, 20 or 21, or exactly two digits followed by a non-digit; and preceded either by a non-digit or by the start of the number |
| end year | the same shape, used for a year range |
| second prefix | one non-digit (or, in some shapes, zero or more non-digits, as few as possible) |
| third prefix | one or more non-digits, as few as possible |
| month | two digits from 01 to 12 |
| counter | a run of digits |
| suffix | zero or more non-digits, as few as possible |

Five shapes are recognised, tried in this order:

| Shape | Reading | Required parts |
|---|---|---|
| Year range, monthly | first prefix, year, one non-digit, end year, one non-digit, month, one or more non-digits, counter, suffix | counter, year, end year, month |
| Monthly | first prefix, year, zero or more non-digits, month, one or more non-digits, counter, suffix | counter, month, year |
| Year range | optionally (first prefix, year, one non-digit, end year, one or more non-digits), then counter, suffix | counter, year, end year |
| Yearly | first prefix, a two- or four-digit year, one or more non-digits, counter, suffix | counter, year |
| Fixed | first prefix, up to nine digits as the counter, suffix | counter |

The counter of the "fixed" shape is limited to nine digits, which is what caps the counter width.

A journal may override the whole grammar with its own pattern. The override, when present, replaces all five shapes and may define the parts named first prefix, year, second prefix, month, third prefix, counter and suffix.

### The prefix and the counter stored on the entry

Independently of the shape, two stored values are derived from the number with the **fixed** reading only:

```formula
sequence_prefix = the part of the number before the counter
sequence_number = the counter read as an integer, or 0 when there is none
```

**Worked examples of the decomposition.**

| Number | Prefix | Counter |
|---|---|---|
| `INV/2026/00042` | `INV/2026/` | 42 |
| `BILL/2026/03/0007` | `BILL/2026/03/` | 7 |
| `MISC/25-26/11/0001` | `MISC/25-26/11/` | 1 |
| `RINV/2026/00003` | `RINV/2026/` | 3 |
| `00000001` | *(empty)* | 1 |
| `/` | `/` | 0 |

---

## 10. Deducing the reset periodicity

Given a reference number, the periodicity at which the counter restarts is deduced by trying the five shapes in the order listed above and returning the first one whose required parts are all present, with one extra consistency rule for year ranges.

```
For each shape, in order (year range monthly, monthly, year range, yearly, fixed):
  1. Try to read the reference number with that shape. If it does not fit, continue.
  2. If the shape produced both a year and an end year:
       a. If the end year is written with more characters than the year, the shape is
          rejected; continue with the next shape.
       b. If the year plus one, truncated to the number of characters of the end year,
          is not equal to the end year, the shape is rejected; continue.
  3. If every required part of the shape is present, return the periodicity of that shape.
If no shape fits, raise the message:
  "The sequence regex should at least contain the seq grouping keys. For instance:" followed
  by the example pattern.
```

"Truncated to a number of characters" means taking the remainder of the division by ten raised to that number:

```formula
truncate( year , length ) = year modulo 10^length
```

**Worked examples.**

| Reference number | Periodicity | Why |
|---|---|---|
| `INV/2026/00042` | yearly | The monthly shape would need a month of two digits after the year; `00042` read as `00` plus `042` does not satisfy the month range and the required parts, so the yearly shape wins |
| `BILL/2026/03/0007` | monthly | Year 2026, month 03, counter 7 |
| `MISC/25-26/11/0001` | year range monthly | Year 25, end year 26, and 25+1 truncated to two characters equals 26 |
| `MISC/25-27/0001` | yearly | The year-range shape is rejected because 25+1 = 26 ≠ 27; the yearly shape then reads year 25 |
| `00000001` | never | Only the fixed shape fits |
| `ABC` | never | Fixed shape with an empty counter |

---

## 11. Extracting the format from a previous number

Once the periodicity is known, the previous number is read again with the corresponding shape and turned into a *format* (a template with holes) and a set of *values*.

```
 1. Read the previous number with the shape of the deduced periodicity.
 2. Record the length of the counter, the length of the year and the length of the end year
    as they appear in the previous number.
 3. If the shape produced no counter but did produce a first prefix and a suffix, move the
    suffix into the first prefix and empty the suffix. (A number that is only text is a
    prefix, not a suffix.)
 4. Convert the counter, the year, the month and the end year to integers, using zero when
    the part is absent.
 5. Build the template by walking the parts of the shape in order and emitting:
      - for the counter: a field padded with zeros to the recorded counter length
      - for the month: a field padded with zeros to two characters
      - for the year: a field padded with zeros to the recorded year length
      - for the end year: a field padded with zeros to the recorded end-year length
      - for every prefix or suffix part: the part itself
```

Filling the template with the values reproduces the previous number exactly. This is the invariant the whole numbering relies on.

**Worked example.** Previous number `BILL/2026/03/0007`, periodicity monthly. The parts are: first prefix `BILL/`, year `2026`, second prefix `/`, month `03`, third prefix `/`, counter `0007`, suffix empty. Recorded lengths: counter 4, year 4. The template is

`BILL/` + (year on 4 digits) + `/` + (month on 2 digits) + `/` + (counter on 4 digits)

and the values are year 2026, month 3, counter 7. Filling them back gives `BILL/2026/03/0007`.

---

## 12. Finding the previous number of a chain

The next number is always derived from the **previous number of the same chain**, found by an ordered database lookup. The ordering is what makes the prefix important: the search takes the greatest counter among the entries that share the prefix of the most recently created entry of the chain.

```
Input: the entry being numbered, and a "relaxed" flag.

 1. Build the chain restriction:
      - same journal, and the number is not the placeholder and not empty;
      - exclude the entry itself;
      - when the journal has a dedicated credit-note numbering: the document type is a
        credit note if the entry is one, and is not a credit note otherwise;
      - when the journal has a dedicated payment numbering: the entry is the entry of a
        payment if the entry being numbered is, and is not otherwise;
      - when the journal is a self-billing journal: the same commercial counterpart; and
        when the entry has no counterpart yet, the restriction is made impossible so that
        a fresh chain is started.
 2. When the flag is not relaxed, additionally restrict to the period:
      a. Take the number of the latest entry of the chain dated on or before the entry
         date; if there is none, take the number of the earliest entry of the chain.
      b. Deduce the periodicity from that number.
      c. Compute the period boundaries for that periodicity (see below) and restrict the
         search to entries dated within them.
      d. Exclude prefixes belonging to a finer periodicity than the one deduced, using the
         exclusion table below, unless the journal overrides the grammar.
 3. Among the remaining entries, take the prefix of the one with the highest identifier.
 4. Among the entries with that prefix, return the number with the highest counter.
```

### Period boundaries

| Periodicity | Boundaries |
|---|---|
| never | the first representable day to the last representable day |
| month | the first and the last day of the month of the entry date |
| year | the first of January and the thirty-first of December of the year of the entry date |
| year range | the first and the last day of the **fiscal** year of the company containing the entry date |
| year range monthly | the month of the entry date, **truncated** at the fiscal year boundary (see below) |

For the year-range-monthly periodicity the month is cut when the fiscal year ends inside it. Let the fiscal year end on day *D* of month *M*, and let the entry fall in month *M*:

- when the entry day is at most *D*, the range is the first of the month to day *D*;
- when the entry day is after *D*, the range is day *D*+1 to the last day of the month.

In both cases the year and end year forced into the number are the years of the fiscal year that contains the entry date.

### Prefix exclusion

The five shapes overlap: a monthly number can also be read by the fixed and the yearly shapes. To stop a yearly chain from picking up a monthly number, prefixes are excluded:

| Deduced periodicity | Excluded prefixes |
|---|---|
| year, or year range | every prefix that the **monthly** shape can read up to the counter |
| never | every prefix that the **yearly** shape can read up to the counter (which also excludes monthly, year-range and year-range-monthly prefixes) |
| month, year range monthly | nothing excluded |

The exclusion is switched off when the journal overrides the grammar.

**Worked example.** A sale journal numbers `INV/2026/00001`, `INV/2026/00002`. A user manually creates `FACT/2026/00001`. The next automatically numbered invoice searches for the highest counter among the entries sharing the prefix of the most recently created entry. The most recent entry is `FACT/2026/00001`, whose prefix is `FACT/2026/`, so the next number is `FACT/2026/00002`. Had the search instead compared numbers alphabetically it would have produced `INV/2026/00003` because `INV` sorts after `FACT`; the rule is deliberately "the prefix of the newest entry", not "the alphabetically greatest".

---

## 13. The starting number of a fresh chain

When no previous number exists at all, the chain starts from a synthetic number whose counter is zero, so that the first assignment yields one.

```
 1. Let DATE be the accounting date, falling back to the document date, falling back to today.
 2. Compute the year part:
      a. If the fiscal year ends on the thirty-first of December, the year part is the four
         digits of the year of DATE.
      b. Otherwise (a staggered fiscal year):
         - let LAST DAY be the fiscal-year end day, capped at the number of days of the
           fiscal-year end month in the year of DATE;
         - if DATE is after (year of DATE, fiscal end month, LAST DAY), the year part is the
           two digits of the year of DATE, a hyphen, and the two digits of the following year;
         - otherwise it is the two digits of the previous year, a hyphen, and the two digits
           of the year of DATE.
 3. Compose:
      a. For a sale, bank, cash or credit-card journal:
           journal code + "/" + year part + "/" + ("0000" for a staggered fiscal year,
           otherwise "00000")
      b. Otherwise, for a self-billing journal:
           journal code + the counterpart identifier padded with zeros to five characters
           + "/" + year part + "/" + the month on two digits + "/0000"
         and when there is no counterpart yet, the literal text "[Partner id]" is used in
         place of the identifier.
      c. Otherwise:
           journal code + "/" + year part + "/" + the month on two digits + "/0000"
 4. Prefix the result with the letter R when the journal has a dedicated credit-note
    numbering and the document is a credit note.
 5. Prefix the result with the letter P when the journal has a dedicated payment numbering
    and the document is the entry of a payment.
```

So sale and liquidity journals number **yearly** by default and every other journal numbers **monthly** by default. The counter is shortened to four digits for a staggered fiscal year because the year part is then six characters instead of four.

**Worked examples.**

| Journal | Fiscal year end | Date | Document | Starting number | First number assigned |
|---|---|---|---|---|---|
| Sale, code `INV` | 31 December | 3 March 2026 | invoice | `INV/2026/00000` | `INV/2026/00001` |
| Sale, code `INV` | 31 December | 3 March 2026 | credit note, dedicated numbering | `RINV/2026/00000` | `RINV/2026/00001` |
| Miscellaneous, code `MISC` | 31 December | 3 March 2026 | entry | `MISC/2026/03/0000` | `MISC/2026/03/0001` |
| Miscellaneous, code `MISC` | 31 March | 3 March 2026 | entry | `MISC/25-26/03/0000` | `MISC/25-26/03/0001` |
| Miscellaneous, code `MISC` | 31 March | 3 April 2026 | entry | `MISC/26-27/04/0000` | `MISC/26-27/04/0001` |
| Bank, code `BNK1` | 31 March | 3 March 2026 | payment, dedicated numbering | `PBNK1/25-26/0000` | `PBNK1/25-26/0001` |

---

## 14. Assigning the next number under concurrency

```
 1. Find the previous number of the chain (section 12) with the relaxed flag off.
 2. If none was found, mark the chain as new and find the previous number again with the
    relaxed flag on; if that also fails, use the starting number of section 13.
 3. Extract the format and the values from that number (section 11).
 4. If the chain is new:
      a. Deduce the periodicity from the number used.
      b. Compute the period boundaries and the forced year range for that periodicity.
      c. Set the counter value to zero.
      d. Set the year value to the start year of the period, truncated to the recorded year
         length; set the end year value to the end year of the period, truncated likewise;
         set the month value to the month of the entry date.
 5. Increment under a lock:
      a. Build a cache key from the format filled with counter zero and from the grouping
         value of the entry (its journal).
      b. If the key is already in the transaction cache, increment the cached counter and
         the result is the number built from it, without touching the stored data.
      c. Otherwise open a savepoint and loop:
           - increase the counter by one;
           - build the candidate number;
           - write it directly into the number column of this entry;
           - if the write succeeds, remember the counter in the transaction cache and
             the result is the candidate;
           - if the write violates the uniqueness constraint, roll back to the savepoint
             and loop again.
 6. Store the number on the record, schedule the recomputation of every stored computed
    field that depends on it, and recompute the prefix and the counter.
```

Two properties matter for a reimplementation:

- **Correctness under concurrency comes from the unique index, not from a counter table.** The direct write takes an exclusive lock on the index entry of the candidate number. A competing transaction that wants the same number blocks until this one commits or rolls back, and then discovers the conflict and retries with the next value. The entry must already satisfy the partial condition of that index (it must be posted with a number different from the placeholder) for the lock to be taken, which is why numbering happens as part of the write that sets the state to posted.
- **After the first lock, further numbers of the same chain in the same transaction come from an in-memory counter**, because opening one savepoint per number would be prohibitively slow. The cache is cleared whenever the number field is written by an ordinary write.

**Worked example.** The chain `INV/2026/` has reached `INV/2026/00007`. Two users post an invoice at the same instant.

1. Both read the previous number `INV/2026/00007`, both build the format `INV/` + year on four digits + `/` + counter on five digits with values year 2026 and counter 7.
2. The first transaction writes `INV/2026/00008` and holds the index lock.
3. The second transaction tries `INV/2026/00008`, blocks, and when the first commits it receives a uniqueness violation, rolls back to its savepoint, tries `INV/2026/00009` and succeeds.
4. No number is skipped and no number is duplicated.

---

## 15. Chain-end tests and gap detection

### Is this entry the last of its chain?

```
 1. Find the previous number of the chain restricted to the prefix of this entry.
 2. If there is none, the entry is the last (it is the only one).
 3. Otherwise extract the format and the values from that previous number, increase the
    counter by one, rebuild the number and compare it with the number of this entry.
    They are equal exactly when the entry is the last.
```

### Are these entries the last ones of their chain?

```
 1. Group the entries by (format, all values except the counter).
 2. For each group:
      a. Let the counters be the set of counter values in the group. If the largest minus
         the smallest is not equal to the size of the group minus one, the group is not
         contiguous: answer no.
      b. Take the entry with the largest counter and apply the single-entry test above.
         If it is not the last of its chain, answer no.
 3. If every group passes, answer yes.
```

This is the test that decides whether a set of entries may be deleted without leaving a hole.

### Gap detection

A stored boolean marks the entry that *opens* a gap. It is refreshed for the entry itself and for its immediate neighbours whenever the prefix, the counter, the journal or the number changes, and whenever entries are deleted.

```
For each entry being updated:
 1. Find, in the same journal and the same prefix and with the same suffix, the two entries
    with the highest counters below it, and the two entries with the lowest counters above
    it. "Same suffix" is tested by requiring the number to end with the counter followed by
    the suffix of the entry.
 2. Define the predicate "X made a gap, given its previous neighbour P and its next
    neighbour N":
      X has a real number, and either
        (a) P exists, has a real number, and the counter of X is not the counter of P plus
            one; or
        (b) N exists, X is not posted, and P is posted.
 3. Set the flag of the entry itself to the predicate, unless the entry was numbered through
    the locked increment in this transaction while being posted (that path cannot produce a
    gap).
 4. Set the flag of the next neighbour: when the update invalidates the current entry, the
    next neighbour is flagged as soon as a previous neighbour exists; otherwise apply the
    predicate to the next neighbour with the current entry as its previous neighbour and the
    second next entry as its next neighbour.
 5. Set the flag of the previous neighbour by applying the predicate to it, with the second
    previous entry as its previous neighbour and either nothing (when the update invalidates
    the current entry) or the current entry as its next neighbour.
 6. Invalidate the "has sequence holes" indicator of the journal.
```

**Worked example.** A journal holds `MISC/2026/03/0001`, `MISC/2026/03/0002`, `MISC/2026/03/0003`. The user deletes the second one. The refresh then examines the third entry: its previous neighbour is now the first entry, whose counter is 1, and 3 is not 1 + 1, so the third entry is flagged as having made a gap. A dashboard indicator and a dedicated filter use this flag to show the journals with holes.

---

## 16. The resequencing algorithm

```
Input: a set of entries of one journal, a first number, and an ordering choice.

 1. Deduce the periodicity from the first number, and extract its format and values.
 2. Partition the entries into periods. The key of an entry is:
      - the year of its accounting date, for the yearly periodicity;
      - the start year and the end year of its fiscal year joined by a hyphen, for the
        year-range periodicity;
      - the same, followed by a slash and the month, for the year-range-monthly periodicity;
      - the pair (year, month), for the monthly periodicity;
      - a single constant, for the fixed periodicity.
 3. For every period, in the order in which the periods were first met:
      a. Compute the period boundaries and the forced year range.
      b. Build the list of new numbers: for the position i (counting from zero) within the
         period, fill the format with
            month  = the month of the period start,
            year   = the forced start year, or the year of the period start, truncated to
                     the recorded year length,
            end year = the forced end year, or the year of the period end, truncated to the
                     recorded end-year length,
            counter = i + ( the counter of the first number, if this is the LAST period of
                            the partition; otherwise 1 ).
      c. Assign those numbers to the entries of the period, in the order chosen:
         - "keep current order": sorted by (prefix, counter);
         - "reorder by accounting date": sorted by (date, current number, identifier).
 4. Clear the number of every selected entry and flush that to the database, so that the
    uniqueness constraint does not fire while the numbers are being swapped.
 5. Write the new numbers one entry at a time.
```

Refusal: renumbering by date in a journal that secures its entries with a hash is refused with "You can not reorder sequence by date when the journal is locked with a hash."

**Worked example.** A miscellaneous journal numbering monthly holds, in February, `MISC/2026/02/0003` dated 5 February and `MISC/2026/02/0007` dated 2 February; and in March `MISC/2026/03/0002` dated 4 March. The user selects the three, sets the first number to `MISC/2026/02/0001` and chooses "reorder by accounting date".

- The periodicity is monthly, the format is `MISC/` + year on 4 + `/` + month on 2 + `/` + counter on 4, and the counter of the first number is 1.
- The partition has two periods: February (two entries) and March (one entry). March is the last period met.
- February is not the last period, so its counters start at 1: the numbers are `MISC/2026/02/0001` and `MISC/2026/02/0002`, assigned by date, so 2 February gets `0001` and 5 February gets `0002`.
- March is the last period, so its counter starts at the counter of the first number, that is 1: the single March entry becomes `MISC/2026/03/0001`.

---

## 17. The inalterability hash chain

Each secured entry carries a hash that binds the hash of the previous secured entry of the same chain to the exact content of this entry. Any later modification of a hashed field breaks the verification.

### The four hash versions

A hash carries a **version**. Four exist, numbered one to four; four is the current one and the one every new hash is computed with. A verification must be able to reproduce all four, because an entry hashed under an older version keeps its old hash for ever and is re-verified under that version. Nothing else in the domain depends on the version.

The three things that vary from one version to the next are the set of fields, the way a monetary amount is turned into text, and the way the result is stored. Everything else — the key naming, the sorting, the serialisation, the chaining — is identical in all four.

| | Version 1 | Version 2 | Version 3 | Version 4 (current) |
|---|---|---|---|---|
| Entry fields | `date`, `journal_id`, `company_id` | `name`, `date`, `journal_id`, `company_id` | same as 2 | same as 2 |
| Item fields | `debit`, `credit`, `account_id`, `partner_id` | `name`, `debit`, `credit`, `account_id`, `partner_id` | same as 2 | same as 2 |
| A monetary amount is written | in the plain numeric form | in the plain numeric form | with exactly the decimals of the currency | with exactly the decimals of the currency |
| The result is stored | as the bare digest | as the bare digest | as the bare digest | as a dollar sign, the digit 4, a dollar sign, then the digest |

A version number outside one to four is not a version: asking for one is an error, not a fallback.

### The fields that enter the hash

At the current version the fields are:

| Level | Fields, in this exact order of naming |
|---|---|
| Entry | `name` (the number), `date`, `journal_id`, `company_id` |
| Each item | `name` (the label), `debit`, `credit`, `account_id`, `partner_id` |

Version one omits the two `name` fields at both levels; versions two, three and four all use the list above. The order in the table is the order in which the values are collected, but it does not survive into the input string, because the keys are sorted before serialisation.

### Rendering each value as text

```
 - a link field is rendered as the decimal identifier of the target record, or as the text
   "False" when empty (the identifier of an empty link is the boolean false);
 - a monetary field is rendered in one of two ways, according to the version:
     * versions 3 and 4 — with exactly the number of decimals of the currency of the record,
       without a thousands separator and with a dot as the decimal separator, so an amount of
       one thousand in a two-decimal currency is written "1000.00" and a zero amount "0.00";
     * versions 1 and 2 — in the plain numeric form of the stored number, that is the
       shortest decimal text that reproduces the stored double-precision value exactly, always
       carrying a decimal point and at least one digit after it, with a dot as the decimal
       separator and no thousands separator; so the same amount of one thousand is written
       "1000.0" and a zero amount "0.0". A magnitude below one ten-thousandth but not zero,
       or of ten to the sixteenth or above, is written in exponent notation in this form;
 - every other field is rendered with the ordinary textual representation of its value:
   a date as "YYYY-MM-DD", a text as itself, an empty value as "False".
```

The monetary rule applies to the two amount fields of an item, `debit` and `credit`, and to nothing else; the currency whose decimals are used is the currency of the **item**, that is the company currency, because the debit and the credit are always expressed in it.

### Building the input string

```
 1. Create an empty mapping from key to text.
 2. For each entry field, in the order listed above, add the entry:
      key   = the storage name of the field
      value = the rendered text
 3. For each item of the entry, in the natural order of the items, and for each item field
    in the order listed above, add the entry:
      key   = the word "line", an underscore, the decimal identifier of the item,
              an underscore, the storage name of the field
      value = the rendered text
 4. Serialise the mapping as a textual object notation document with:
      - the keys sorted in ascending lexical order of their characters,
      - only the characters of the basic Latin alphabet escaped as needed (any character
        outside that range is escaped as an escape sequence),
      - no indentation,
      - a comma and no space between entries,
      - a colon and no space between a key and its value,
      - every value written as a text string.
 5. Take the hash of the previous entry of the chain. When that previous hash begins with a
    dollar sign, strip the leading "dollar sign, version, dollar sign" marker and keep only
    the digest: the version marker never enters the computation.
 6. Concatenate the previous digest (the empty string when the chain has no previous hash)
    and the serialised document, encode the result in the eight-bit Unicode transformation
    format, and compute the two-hundred-and-fifty-six-bit secure hash of that byte string,
    rendered as lowercase hexadecimal.
 7. Store the result. At version 4 the stored value is: a dollar sign, the version number,
    a dollar sign, then the hexadecimal digest. At versions 1, 2 and 3 the stored value is
    the bare hexadecimal digest with no marker at all. Step 5 is what makes the two forms
    interchangeable when they meet in one chain.
 8. The digest just computed (without the marker, when there is one) becomes the previous
    digest for the next entry of the chain.
```

Step 5 has no version of its own: it strips a marker whenever the previous stored hash carries one, whatever version is being computed. So a chain whose older part was hashed at version 3 (bare digests) and whose newer part is hashed at version 4 (marked digests) chains correctly across the boundary, and re-verifying the newer part still feeds it the bare digest of the older one.

### Worked example

Chain with two entries in journal number 3 of company number 1.

Entry A: number `MISC/2026/03/0001`, date 2026-03-05, two items with identifiers 51 and 52. Item 51: label `Rent`, debit 1000.00, credit 0.00, account 12, no partner. Item 52: label `Rent`, debit 0.00, credit 1000.00, account 34, no partner. The company currency has two decimals.

The mapping, once sorted by key, is:

| Key | Value |
|---|---|
| `company_id` | `1` |
| `date` | `2026-03-05` |
| `journal_id` | `3` |
| `line_51_account_id` | `12` |
| `line_51_credit` | `0.00` |
| `line_51_debit` | `1000.00` |
| `line_51_name` | `Rent` |
| `line_51_partner_id` | `False` |
| `line_52_account_id` | `34` |
| `line_52_credit` | `1000.00` |
| `line_52_debit` | `0.00` |
| `line_52_name` | `Rent` |
| `line_52_partner_id` | `False` |
| `name` | `MISC/2026/03/0001` |

serialised as one line: an opening brace, then each key in double quotes, a colon, the value in double quotes, separated by commas, then a closing brace. The chain has no previous hash, so the previous digest is the empty string and the input to the hash function is exactly that serialised line. The stored value is a dollar sign, the digit 4, a dollar sign and the sixty-four hexadecimal characters of the digest.

Entry B, numbered `MISC/2026/03/0002`, is serialised the same way, and its input is the digest of entry A (without the marker) followed by its own serialised line.

### Verification

```
For each journal of the company:
 1. Select every entry of the journal that carries a hash, ordered by the legacy securing
    number ascending with empty values last, then by prefix, then by counter ascending.
 2. Walk the entries in batches, keeping per prefix: the first verified entry, the last
    verified entry and the first corrupted entry.
 3. For each entry, the previous entry is the last verified entry of the same prefix, or —
    when the entry carries a legacy securing number — the last verified entry overall.
 4. Recompute the hash starting at hash version one and increasing the version until it
    matches or the maximum version is reached.
 5. The first entry whose hash cannot be reproduced marks the prefix as corrupted; the
    remaining entries of that prefix are not checked.
```

Step 4 is why the version table above must be reproduced in full: a verification that only knew the current version would report every entry hashed before it as corrupted. The retry starts at version one and stops at the first version whose recomputation matches the stored value, or at version four without a match, in which case the entry is corrupted. The retry is per entry and the version reached is carried forward as the starting version of the next entry of the same walk, so a chain hashed entirely at one version costs one attempt per entry after the first.

The report shows, per journal and prefix, either "Entries are correctly hashed" with the first and last verified entry, their hashes and their dates, or "Corrupted data on journal entry with id *the identifier* (*the number*)." A journal with no hashed entry reports "There is no journal entry flagged for accounting data inalterability yet."

Running the report without the accounting user group is refused: "Please contact your accountant to print the Hash integrity result."

---

## 18. The reconciliation algorithm

Reconciliation matches debit items against credit items on the same account, creating one Partial Reconciliation per matched pair and, when the whole matched group nets to zero, one Full Reconciliation.

### 18.1 Residual amounts

The residual of an item is its amount minus everything already matched:

```formula
amount_residual          = round_to_company_currency(
                              balance
                              − sum of the matched amounts where the item is the debit side
                              + sum of the matched amounts where the item is the credit side )

amount_residual_currency = round_to_item_currency(
                              amount_currency
                              − sum of the matched debit-side amounts in the item currency
                              + sum of the matched credit-side amounts in the item currency )

reconciled = ( amount_residual is zero for the company currency )
             and ( amount_residual_currency is zero for the item currency )
```

The sums in the item currency are themselves rounded to the item currency before being subtracted.

An item whose account neither allows matching nor is a bank-and-cash or credit-card account has both residuals forced to zero and is never considered reconciled.

**Worked example.** A receivable item of 1 210.00 (balance +1 210.00, foreign amount +1 210.00, company currency) is matched twice: 500.00 and 300.00. Its residual is 1 210.00 − 800.00 = 410.00 in both currencies, and it is not reconciled.

### 18.2 Preconditions

Before anything is matched, the set of items is checked. Items that are already reconciled are dropped **unless** they carry a partial matching number that is shared with an unreconciled item of the set (which means the group is being extended). Then:

| Condition | Message |
|---|---|
| Any remaining item is already reconciled | "You are trying to reconcile some entries that are already reconciled." |
| Any item belongs to a cancelled entry | "You can not reconcile cancelled entries." |
| The items use more than one account | "Entries are not from the same account: *the account display names*" |
| The items belong to more than one company hierarchy | "Entries don't belong to the same company: *the company display names*" |
| The account neither allows matching nor is a bank-and-cash or credit-card account | "Account *the account display name* does not allow reconciliation. First change the configuration of this account to allow it." |

### 18.3 Building the plan

A reconciliation request is a *plan*: a list whose members are either a set of items or another plan. Sets are processed in order; a nested plan is processed before the set that contains it, and then the union of everything is processed once more so that leftovers match each other.

```
 1. For each member of the plan:
      a. If it is a set of items, sort it and split it by currency:
           - sort by (due date, falling back to the accounting date), then currency, then
             foreign amount, then balance;
             in the reduced sorting mode only the first two keys are used;
           - when the set uses more than one currency, create one child node per currency
             containing the items of that currency, in the sorted order.
      b. If it is a nested plan, process its members recursively and take the union of their
         items as the node.
 2. Check the preconditions on every node.
 3. Discard empty nodes.
```

Processing a node means: process its children first, then process the items of the node itself that are not yet fully matched. Within a node, when more than one counterpart is present the items are additionally sorted by counterpart identifier so that items of the same counterpart meet first.

### 18.4 The available residuals of one item, seen from a counterpart currency

Before two items can be matched, the currency in which the match will be expressed must be chosen. For that, each item publishes the residuals it can offer, one per currency, each with the rate that converts the company currency into it.

```
Input: an item with its residual in the company currency and its residual in its own
       currency, and the currency of the counterpart item.

 1. If the residual in the company currency is not zero, publish the company currency with
    that residual and the rate 1.
 2. If the item currency differs from the company currency and the residual in the item
    currency is not zero, publish the item currency with that residual and the accounting
    rate of the item (see below).
 3. If the item is expressed in the company currency, sits on a receivable or payable
    account, has a non-zero company residual, and the counterpart is in another currency:
      a. Determine the rate to the counterpart currency (see below).
      b. Convert the company residual at that rate and round to the counterpart currency.
      c. If the result is not zero, publish the counterpart currency with that residual and
         that rate. (This is what lets an invoice kept in the company currency be matched in
         the currency of the payment.)
 4. Otherwise, if the item currency **is** the counterpart currency and differs from the
    company currency and the residual in the item currency is not zero, publish the
    counterpart currency with that residual and the accounting rate.
```

The **accounting rate** of an item is the rate implied by the item itself:

```formula
accounting_rate = | amount_currency ÷ balance |
```

computed only when neither the balance nor the foreign amount is zero.

The **rate to the counterpart currency** is chosen in this order:

1. a rate forced by the caller (the payment register wizard forces the rate it displayed);
2. when the item is not a payment or bank transaction and the counterpart **is** one, the accounting rate of the counterpart, so that the invoice adopts the rate of the payment;
3. otherwise the rate of the currency table on the relevant date, which is the document date for an invoice-like document and the accounting date otherwise.

### 18.5 Choosing the currency of the match

```
 1. If the debit currency differs from the company currency and both items published a
    residual in it, the match is made in the debit currency.
 2. Otherwise, if the credit currency differs from the company currency and both items
    published a residual in it, the match is made in the credit currency.
 3. Otherwise the match is made in the company currency.
```

If either item published nothing in the chosen currency, that item is considered exhausted and is dropped from the iteration.

### 18.6 Computing one partial match

Let the debit residual in the reconciliation currency be *D* and the credit residual, sign-flipped to be positive, be *C*.

```formula
matched_amount_in_reconciliation_currency = min( D , C )
debit_fully_matched  = ( D ≤ C )
credit_fully_matched = ( D ≥ C )
```

*Exchange-line mode* is detected first: the reconciliation currency is the company currency, both items share the same currency, and at least one of them published nothing in that shared currency. This is what an exchange-difference item looks like, and in that mode no rate is applied, because the exchange item is meant to change only the company-currency amount.

#### Case A — the match is made in the company currency

```formula
matched_amount = min( D , C )

matched_debit_amount_in_debit_currency  = min( round_to_debit_currency( debit_rate × matched_amount ) ,
                                               debit residual in the debit currency )
matched_credit_amount_in_credit_currency = min( round_to_credit_currency( credit_rate × matched_amount ) ,
                                                − credit residual in the credit currency )
```

with both currency amounts set to zero when the corresponding rate is absent (which is the case in exchange-line mode, and whenever the item has no residual left in its own currency).

#### Case B — the match is made in a foreign currency

First each side is converted back into the company currency, using the **inverse** of its rate, and a tolerance range is computed around the result to absorb the rounding of the original amounts:

```formula
half_step_of_source_currency = rounding_step_of_source_currency ÷ 2

low    = round_to_company_currency( ( matched_amount_in_reconciliation_currency − half_step ) ÷ rate )
middle = round_to_company_currency(   matched_amount_in_reconciliation_currency             ÷ rate )
high   = round_to_company_currency( ( matched_amount_in_reconciliation_currency + half_step ) ÷ rate )
```

Then:

```formula
debit_side_company_amount  = min( middle_of_the_debit_side  , debit residual in company currency )
credit_side_company_amount = min( middle_of_the_credit_side , − credit residual in company currency )
matched_amount             = min( debit_side_company_amount , credit_side_company_amount )
```

**Rounding-noise suppression.** If the debit-side amount falls inside the tolerance range of the credit side **and** the credit-side amount falls inside the tolerance range of the debit side, the difference is pure rounding noise, not a genuine exchange difference. In that case all three values are replaced by

```formula
matched_amount = min( debit residual in company currency , − credit residual in company currency )
```

so that no exchange-difference entry is produced and no residual is left open.

The amounts in each item currency are then:

```formula
matched_debit_amount_in_debit_currency   = matched_amount                                  when the debit item is in the company currency
matched_debit_amount_in_debit_currency   = matched_amount_in_reconciliation_currency       otherwise
matched_credit_amount_in_credit_currency = matched_amount                                  when the credit item is in the company currency
matched_credit_amount_in_credit_currency = matched_amount_in_reconciliation_currency       otherwise
```

#### Updating the residuals

```formula
new_debit_residual_company   = debit_residual_company   − matched_amount
new_credit_residual_company  = credit_residual_company  + matched_amount
new_debit_residual_currency  = debit_residual_currency  − matched_debit_amount_in_debit_currency
new_credit_residual_currency = credit_residual_currency + matched_credit_amount_in_credit_currency
```

An item is dropped from the iteration when **both** of its residuals are zero at their respective precisions.

#### The stored match

| Stored field | Value |
|---|---|
| debit item | the debit item |
| credit item | the credit item |
| amount | the matched amount in the company currency (always positive) |
| amount in the debit currency | the matched debit amount (always positive) |
| amount in the credit currency | the matched credit amount (always positive) |

### 18.7 The pairing loop

```
 1. Split the items of the node into a debit list — items whose balance is positive, or
    whose foreign amount is positive — and a credit list — items whose balance is negative,
    or whose foreign amount is negative — each keeping the node order.
 2. Take the first debit item and the first credit item.
 3. Compute one partial match (18.6).
 4. Record the match when it produced an amount.
 5. Advance the debit pointer when the debit item is exhausted, and the credit pointer when
    the credit item is exhausted.
 6. Repeat until either list is exhausted.
```

Note that an item can appear in **both** lists in a degenerate case (a zero balance with a positive foreign amount, and the converse); the filters are deliberately "or" filters so that an item with a zero balance but a non-zero foreign amount, such as an exchange-difference item, still participates.

### 18.8 Worked examples

**Example 1 — plain partial match in the company currency.** Invoice receivable +1 210.00, payment credit −500.00, both in the company currency.

- Both publish the company currency: debit residual 1 210.00 rate 1; credit residual −500.00 rate 1.
- Reconciliation currency: company currency. *D* = 1 210.00, *C* = 500.00.
- Matched amount 500.00. The credit is fully matched, the debit is not.
- Stored match: amount 500.00, debit amount 500.00, credit amount 500.00.
- New residuals: 710.00 and 0.00. The credit item is dropped.

**Example 2 — two currencies, exchange difference.** Company currency euro. Invoice in dollars: balance +952.38, foreign amount +1 000.00 (rate 1.05 dollars per euro). Payment in dollars: balance −909.09, foreign amount −1 000.00 (rate 1.10).

- The debit item publishes the euro (952.38, rate 1) and the dollar (1 000.00, accounting rate 1 000.00 ÷ 952.38 = 1.05).
- The credit item publishes the euro (−909.09, rate 1) and the dollar (−1 000.00, accounting rate 1.10).
- The debit currency is the dollar and both published it, so the match is in dollars. *D* = 1 000.00, *C* = 1 000.00, matched amount in dollars 1 000.00, both sides fully matched.
- Case B: the debit side converts back at 1 ÷ 1.05 giving 952.38 (tolerance range computed from half a cent of a dollar); the credit side converts back at 1 ÷ 1.10 giving 909.09.
- 952.38 does not lie inside the tolerance range of 909.09 (the two differ by more than a rounding step), so the noise suppression does not apply.
- Matched amount in euros = min(952.38, 909.09) = 909.09.
- Stored match: amount 909.09, debit amount in dollars 1 000.00, credit amount in dollars 1 000.00.
- New residuals: the debit item keeps 952.38 − 909.09 = 43.29 euros with 0.00 dollars; the credit item keeps 0.00 and 0.00.
- Because the debit side is fully matched in the reconciliation currency but keeps a company-currency residual, an exchange difference of 43.29 euros is prepared (section 19).

**Example 3 — rounding-noise suppression.** Company currency euro. Debit item: balance 377 554.00, foreign amount 20 000.00. Credit item: balance −5 314.62, foreign amount −281.53. The match is made in the foreign currency for 281.53.

- Debit side: rate 20 000.00 ÷ 377 554.00; converting 281.53 back gives 5 314.64 with a tolerance range of [5 314.54, 5 314.73].
- Credit side: rate 281.53 ÷ 5 314.62; converting 281.53 back gives 5 314.62 with a tolerance range of [5 314.53, 5 314.71].
- 5 314.64 lies inside [5 314.53, 5 314.71] and 5 314.62 lies inside [5 314.54, 5 314.73], so the two are indistinguishable at the rounding precision.
- The matched amount is therefore forced to min(377 554.00, 5 314.62) = 5 314.62, no exchange difference is created and the credit item is left with nothing open.

---

## 19. Exchange differences at reconciliation

### When a difference is produced

Let the *reconciliation currency* be the currency in which the match was made.

**If the match was made in the company currency:**

```formula
debit_exchange_amount  = debit_residual_currency  − matched_debit_amount_in_debit_currency     when the debit side is fully matched
credit_exchange_amount = credit_residual_currency + matched_credit_amount_in_credit_currency   when the credit side is fully matched
```

Each is produced only when it is not zero at the precision of the corresponding item currency, and it corrects the **foreign** residual only.

**If the match was made in a foreign currency:**

```formula
debit_exchange_amount  = debit_residual_company − matched_amount                when the debit side is fully matched
debit_exchange_amount  = debit_side_company_amount − matched_amount             otherwise, and only when it is strictly positive
credit_exchange_amount = credit_residual_company + matched_amount               when the credit side is fully matched
credit_exchange_amount = matched_amount − credit_side_company_amount            otherwise, and only when it is strictly negative
```

Each is produced only when it is not zero at the precision of the company currency, and it corrects the **company-currency** residual only. The two "otherwise" branches exist so that a partially matched item keeps a residual whose ratio between the two currencies stays equal to the ratio between its own balance and foreign amount.

After a difference is produced, the corresponding residual is reduced by it before the match itself is subtracted.

### The exchange-difference entry

| Property | Value |
|---|---|
| Document type | plain entry |
| Number | the placeholder, so that the number is taken at posting |
| Journal | the exchange journal of the company |
| Date | the greater of the accounting dates of the two matched items, then pushed through the accounting-date rule of the exchange journal; the running value is further raised to the date of each corrected item |
| Always tax exigible | true |
| Company | the company of the invoice-like entry among the two, otherwise the company of the items |

For each corrected item, two items are created, in this order:

| # | Account | Debit | Credit | Foreign amount | Currency | Partner | Other |
|---|---|---|---|---|---|---|---|
| 1 | the account of the corrected item | the company amount when it is negative, else 0 | the company amount when it is positive, else 0 | minus the foreign amount | the currency of the corrected item | the partner of the corrected item | carries the full-reconcile link of the corrected item and is immediately matched with it |
| 2 | the exchange **expense** account when the amount to fix is positive, the exchange **income** account otherwise | the company amount when it is positive, else 0 | minus the company amount when it is negative, else 0 | the foreign amount | the same currency | the same partner | carries the analytic distribution when one was supplied |

Both items are labelled "Currency exchange rate difference".

When the correction is expressed in the company currency, the foreign amount of the pair is the company amount when the item currency **is** the company currency and zero otherwise. When the correction is expressed in the foreign currency, the company amount of the pair is zero and only the foreign amount is non-zero.

The entry is posted immediately when **both** matched items belong to posted entries; otherwise it stays draft and is posted together with them.

### Configuration failures

| Condition | Message |
|---|---|
| No exchange journal on the company | "You have to configure the 'Exchange Gain or Loss Journal' in your company settings, to manage automatically the booking of accounting entries related to differences between exchange rates." |
| No loss account | "You should configure the 'Loss Exchange Rate Account' in your company settings, to manage automatically the booking of accounting entries related to differences between exchange rates." |
| No gain account | "You should configure the 'Gain Exchange Rate Account' in your company settings, to manage automatically the booking of accounting entries related to differences between exchange rates." |

**Worked example (continuing example 2 of section 18).** The debit item keeps 43.29 euros open with nothing open in dollars. The correction amount is +43.29, so the second item uses the **expense** exchange account. The entry contains:

| Account | Debit | Credit | Foreign amount |
|---|---|---|---|
| the receivable account of the invoice | 0.00 | 43.29 | 0.00 dollars |
| the exchange loss account | 43.29 | 0.00 | 0.00 dollars |

and the first item is matched with the invoice item, which then reaches a zero residual in both currencies.

---

## 20. Full-reconcile detection and matching numbers

### Detection

```
 1. Take every item touched by the plan, and extend the set with every item transitively
    matched with them through their matching numbers (ignoring imported numbers).
 2. Group the extended set into connected batches: two items are in the same batch when they
    share a matching number.
 3. For each batch, decide whether it is fully reconciled. An item counts as reconciled when:
      - it is already flagged reconciled; or
      - it participates in at least one match and:
          * the batch uses more than one currency: its residual in the company currency is
            zero at the company precision;
          * the batch uses exactly one currency: its residual in that currency is zero at
            that precision.
    An item that participates in no match at all does **not** count as reconciled, even when
    both its residuals are zero — this covers an item with a zero balance but a non-zero
    foreign amount.
 4. A batch in which every item counts as reconciled produces one Full Reconciliation
    carrying every match and every item of the batch.
```

### Matching numbers

The matches of a set of items form an undirected graph whose vertices are items and whose edges are matches. Each connected component receives a number:

```
 1. Take every match of the extended set, ordered by ascending identifier.
 2. Maintain a mapping from item to component number and a mapping from component number to
    the list of its items.
 3. For each match, look up the component of its debit item and of its credit item:
      - both known and different: merge the two components into the one with the smaller
        number; every item of the larger component is renumbered;
      - only the debit known: add the credit item to that component;
      - only the credit known: add the debit item to that component;
      - neither known: create a new component whose number is the identifier of this match,
        containing both items.
 4. Write the matching number of every item:
      - the decimal identifier of its Full Reconciliation when it has one;
      - otherwise the letter P followed by the component number.
 5. Items of the extended set that ended up in no component have their matching number
    cleared.
```

The component number is therefore the smallest match identifier of the component, which makes the label stable as long as the earliest match survives.

### Imported matching numbers

A number written from outside the system is prefixed with the letter `I` so that it cannot be confused with a real one. Any write of a matching number that does not already start with `I` is rewritten as `I` followed by the supplied text, unless the caller explicitly bypasses the rule.

At posting, imported numbers are resolved:

```
 1. Collect the distinct imported numbers among the posted items.
 2. Group every item carrying one of them by (number, account).
 3. For each group, when every entry of the group is posted:
      a. If the account does not allow matching, switch the flag on (and log that the
         configuration was changed).
      b. Reconcile the group, with exchange differences and cash-basis entries suppressed.
```

### Integrity of the matching number

A validation refuses any inconsistent combination:

| Situation | Refused |
|---|---|
| A number that is neither a decimal number, nor `P` followed by digits, nor `I` followed by anything | yes |
| An imported number on an item that already participates in real matches | "A temporary number can not be used in a real matching" |
| A partial number on an item with no match | yes |
| A partial number on an item that has a Full Reconciliation | yes |
| A decimal number on an item with no Full Reconciliation | yes |
| A decimal number different from the identifier of the Full Reconciliation | yes |
| No number at all on an item that participates in matches | yes |

---

## 21. The fiscal year

The fiscal year of a company is defined by the day and the month on which it ends.

```
Input: a date, the fiscal-year end day D and the fiscal-year end month M.

 1. If M is December and D is 31, the fiscal year is the calendar year of the date.
 2. Otherwise, let END be the date (year of the input, M, D), with D reduced to the number
    of days of that month when it overflows.
    a. If the input date is after END, the fiscal year runs from the day after END to the
       same day one year later.
    b. Otherwise it runs from the day after the equivalent END of the previous year to END.
```

**Worked examples.** Fiscal year ending 31 March.

| Date | Fiscal year |
|---|---|
| 3 March 2026 | 1 April 2025 to 31 March 2026 |
| 31 March 2026 | 1 April 2025 to 31 March 2026 |
| 1 April 2026 | 1 April 2026 to 31 March 2027 |

Fiscal year ending 29 February: a request in 2027 (not a leap year) caps the day at 28.

---

## 22. The opening entry

The opening entry carries the initial balance of every account. It is a single plain entry of the company, and the tool that maintains it always keeps it balanced by adjusting one line on the current-year-earnings account.

### Default values of a new opening entry

| Property | Value |
|---|---|
| Reference | "Opening Journal Entry" |
| Company | the company |
| Journal | the first miscellaneous journal of the company; when there is none the operation fails with "Please install a chart of accounts or create a miscellaneous journal before proceeding." |
| Date | the opening date of the company minus one day; when no opening date is set, the first of January of the current year minus one day, that is the thirty-first of December of the previous year |

### The balancing account

The company must have an account of type "Current Year Earnings". When none exists, one is created:

```
 1. Start from the code 999999.
 2. While an account with that code exists in the company (including archived ones),
    decrease the code by one.
 3. Create the account with that code, the name "Profit or Loss Appropriation" and the type
    "Current Year Earnings".
```

### Updating the entry

```
Input: for each account, a pair (debit to set, credit to set); either member may be absent,
       meaning "leave that side alone".

 1. Refuse the operation when an opening entry exists and is not draft:
    "You cannot import the "openning_balance" if the opening move (…) is already posted. …"
 2. Index the existing items of the opening entry by (account, side), where the side is
    "debit" when the balance or the foreign amount is positive and "credit" otherwise.
 3. Initialise the running open balance with the credit total minus the debit total of the
    existing items on the balancing account.
 4. For every requested (account, side, amount):
      a. Let the amount be positive for the debit side and negative for the credit side.
      b. Add the amount to the running open balance.
      c. If the amount is zero for the company currency, delete every existing item of that
         (account, side) and subtract each deleted balance from the running open balance.
      d. Otherwise, if such an item exists, update the first one with the new balance and
         the converted foreign amount (subtracting its old balance from the running open
         balance first), and delete the others.
      e. Otherwise create a new item labelled "Opening balance" with that balance, that
         foreign amount, the currency of the account when it forces one and the company
         currency otherwise.
 5. Add the two balancing items on the current-year-earnings account:
      debit side  = max( − running open balance , 0 )
      credit side = − max( running open balance , 0 )
    labelled "Automatic Balancing Line"; their foreign amount is equal to their balance.
 6. Write the collected commands on the existing entry, or create it with the default values
    above when it does not exist.
```

The conversion of a balance into the foreign amount of an item uses the company currency converted to the account currency at the date of the entry.

**Worked example.** A new company sets its opening date to 1 January 2026, so the opening entry is dated 31 December 2025. Two accounts are given opening balances: a bank account with a debit of 10 000.00 and a supplier account with a credit of 4 000.00.

| Account | Debit | Credit |
|---|---|---|
| Bank | 10 000.00 | |
| Suppliers | | 4 000.00 |
| Profit or Loss Appropriation | | 6 000.00 |

The running open balance after the two requested amounts is 10 000.00 − 4 000.00 = 6 000.00, so the balancing item takes a credit of 6 000.00 and the entry balances.

---

## 23. Partner ledger figures

### Amount receivable and amount payable

```formula
partner_credit = sum of amount_residual over the posted, unreconciled items of that partner
                 whose account type is Receivable, in the company hierarchy of the active
                 company
partner_debit  = − ( sum of amount_residual over the posted, unreconciled items of that
                     partner whose account type is Payable, in the same hierarchy )
```

Both are read in the company currency. A partner with no such item reports zero.

### Total invoiced

```formula
total_invoiced = sum of the untaxed subtotals of the customer invoices and credit notes of
                 the partner and of every partner below it in the hierarchy, excluding draft
                 and cancelled documents
```

### Days of sales outstanding

```formula
days_since_oldest_invoice = today − ( the earliest document date among the sale documents of
                                      the commercial entity that are neither draft nor
                                      cancelled, in the active company )

days_sales_outstanding = ( partner_credit ÷ total_invoiced_tax_included ) × days_since_oldest_invoice
```

where the tax-included total is the sum of the signed company-currency totals of those documents. The result is zero when that total is zero. When the partner has no such document, the earliest date used is today, so the numerator of the day count is zero.

**Worked example.** A customer has 12 100.00 still open on receivable accounts. The oldest of its sale documents is dated 90 days ago, and the signed company-currency total of all its sale documents is 60 500.00.

```formula
days_sales_outstanding = ( 12 100.00 ÷ 60 500.00 ) × 90 = 0.2 × 90 = 18
```

---

## 24. Account code generation

Repeated here as a formula because the rule matters for reimplementation. Let the starting code be split by the rule "the longest trailing run of non-digits is the tail, the digit run immediately before it is the number, and everything before is the head":

```formula
candidate( k ) = head + ( number + k , written with leading zeros to the original width ) + tail
```

for k = 1, 2, 3 … while the number plus k is strictly less than ten raised to the original width. When the digit run is empty, or the walk is exhausted:

```formula
fallback( 0 ) = starting_code + ".copy"
fallback( n ) = starting_code + ".copy" + ( n + 1 )      for n = 1 … 98
```

The first candidate that is *available* is used. Availability is defined as: no account with that code belongs to the active company, to any of its ancestors or to any of its descendants, and the code has not already been handed out in this same operation.

---

## 25. Journal dashboard figures

Each journal shows a small set of numbers on the accounting dashboard.

### Balance of a liquidity journal

```formula
journal_balance = sum of amount_currency over the items on the default account of the journal
                  whose entry is not cancelled and whose display type is not a section,
                  subsection or note                            when the journal has a
                                                                foreign currency
journal_balance = sum of balance over the same items            otherwise
```

Note that the aggregation is deliberately **not** restricted to the journal itself: it is driven by the account, so a transfer booked in another journal on the same bank account is included.

### Outstanding payment accounts

The set of accounts used by the money-in methods of the journal, and the set used by the money-out methods; both are read from the payment method lines.

### Entries to hash

The dashboard reports whether a journal still has entries to secure by running the chain selection of section 17 with an early stop as soon as one entry is found.

### Numbering holes

A journal has holes when at least one of its entries carries the gap flag. The indicator is invalidated whenever the gap detection runs.

---

## 26. The automatic transfer amounts

### Share of each item

```formula
total_amount = ( percentage ÷ 100 ) × ( sum of the balances of the selected items )
percentage   = min( ( total_amount ÷ sum of the balances of the selected items ) × 100 , 100 )
```

The two fields are inverse views of each other; the minimum with one hundred exists to absorb a rounding error that would otherwise produce a value slightly above one hundred. When the sum of balances is zero the percentage is one hundred.

The nature of the transfer is deduced from the sign:

```formula
account_type = revenue    when the sum of the selected balances is negative
account_type = expense    otherwise
```

### Change of period — the two mirrored entries

For each selected item, the reported amounts are:

```formula
reported_debit           = round_to_company_currency( ( percentage ÷ 100 ) × debit )
reported_credit          = round_to_company_currency( ( percentage ÷ 100 ) × credit )
reported_amount_currency = round_to_item_currency(    ( percentage ÷ 100 ) × amount_currency )
```

Two entries are produced per item.

**In the destination entry (dated at the chosen date):**

| Account | Debit | Credit | Foreign amount |
|---|---|---|---|
| the account of the source item | reported debit | reported credit | reported foreign amount |
| the accrual account (revenue or expense according to the deduced nature) | reported credit | reported debit | minus the reported foreign amount |

**In the cancelling entry (dated at the lock-safe equivalent of the source date):**

| Account | Debit | Credit | Foreign amount |
|---|---|---|---|
| the account of the source item | reported credit | reported debit | minus the reported foreign amount |
| the accrual account | reported debit | reported credit | reported foreign amount |

Both items keep the partner and the analytic distribution of the source item, and are labelled "Cut-off *the source number*" when the percentage is one hundred, or "Cut-off *the source number* *the percentage*%" otherwise, with the percentage written with two decimals.

The "lock-safe equivalent" of a date is that date pushed through the accounting-date rule of section 8 for the chosen journal, with no tax involvement. Items whose source dates map to the same lock-safe date share one cancelling entry.

After posting, the pairs of mirror items (same label, same reconcilable account, same partner, same currency, same analytic distribution) are reconciled with each other.

**Worked example.** A customer invoice posted on 20 December books 1 200.00 of revenue. On 31 December the accountant defers 100 % of it to January. The chosen date is 1 January, the revenue accrual account is a deferred-revenue liability account, the journal numbers monthly and nothing is locked.

- The lock-safe equivalent of 20 December, computed in January, is 31 December.
- The cancelling entry dated 31 December: debit the revenue account 1 200.00, credit the deferred-revenue account 1 200.00.
- The destination entry dated 1 January: debit the deferred-revenue account 1 200.00, credit the revenue account 1 200.00.
- The two deferred-revenue items are then reconciled with each other.

### Change of account

The selected items are regrouped and mirrored on the destination account.

```
 1. For every selected item whose account is not already the destination account:
      a. The counterpart currency is the item currency, unless the destination account
         forces a currency different from the company currency, in which case it is that
         currency and the counterpart foreign amount is the balance of the item converted
         into it at the item date.
      b. Accumulate, per (partner, counterpart currency): the counterpart foreign amount,
         the balance, and per analytic account the quantity
             balance × percentage_of_that_analytic_account
      c. Accumulate the source items per (partner, item currency, account, analytic
         distribution).
 2. For every (partner, counterpart currency) whose accumulated foreign amount or balance is
    not zero, create the counterpart item:
        account   = the destination account
        debit     = the accumulated balance rounded to the company currency, when positive
        credit    = minus it, when negative
        foreign   = the accumulated foreign amount, made to carry the sign of the balance
        analytic  = for each analytic account, 100 × ( accumulated amount ÷ accumulated
                    balance ), or 100 when the accumulated balance is zero
        label     = "Transfer from *the source account display name*" when exactly one source
                    account exists, otherwise "Transfer counterpart"
 3. For every (partner, currency, account, analytic distribution) group whose accumulated
    balance is not zero, create the mirror item:
        account   = the source account
        debit     = minus the accumulated balance when it is negative
        credit    = the accumulated balance when it is positive
        foreign   = the accumulated foreign amount with the opposite sign of the balance
        label     = "Transfer to *the destination account display name*"
 4. Create one entry with those items, of type plain entry, numbered with the placeholder,
    in the chosen journal, at the chosen date, referenced "Transfer entry to *the destination
    account display name*", in the deepest child company among the companies of the accounts
    used and the active company.
 5. Post it, then reconcile: per (partner, currency, source account) the source items with
    the matching mirror items when the source account allows matching; and the already
    destination-account items with the matching counterpart items when the destination
    account allows matching.
```

**Worked example.** Two receivable items of the same customer, 300.00 and 700.00, are transferred to a doubtful-debt account.

- Accumulated per (customer, company currency): balance 1 000.00.
- Counterpart item: debit 1 000.00 on the doubtful-debt account, labelled "Transfer from *the receivable account*".
- Mirror item: credit 1 000.00 on the receivable account, labelled "Transfer to *the doubtful-debt account*".
- After posting, the two original items and the mirror item are reconciled together on the receivable account.

---

## 27. The order in which numbers are assigned

When several entries are posted in one operation, the order in which they consume numbers is **not** the order in which they were selected or created. The numbering computation first sorts the set:

```formula
sort key = ( accounting date , reference or the empty string , original identifier )
```

ascending on all three components. Then, walking the sorted list:

```
 1. Skip every cancelled entry.
 2. Let HAS_NAME mean that the number exists and is not the placeholder.
 3. If the entry was never posted and its number does not match its accounting date
    (see the date-alignment test below), clear the number and continue with the next entry.
 4. If the entry has an accounting date, has no number, and is not draft, assign the next
    number of its chain.
 5. After the walk, run the inverse of the number field: recompute the payment reference of
    every entry that now has a real number, and refresh the gap flags.
```

The date-alignment test used in step 3 is:

```
 1. If the entry has no number or no accounting date, it matches.
 2. Read the number with the shape of its deduced periodicity to obtain the year, the end
    year and the month values.
 3. Compute the period boundaries for that periodicity from the accounting date.
 4. The year matches when the number carries no year, or when the year value equals the start
    year of the period truncated to the number of characters the year occupies in the number.
 5. The end year matches under the same rule against the end year of the period.
 6. The month matches when the number carries no month, or when the month value equals the
    month of the accounting date.
 7. The number matches the date when the year, the end year and the month all match.
```

**Worked example.** The highest number of the chain is `XMISC/2016/00001`. Six draft entries are created in this order, with these accounting dates:

| Creation order | Accounting date |
|---|---|
| 1 | 5 March 2019 |
| 2 | 6 March 2019 |
| 3 | 7 March 2019 |
| 4 | 4 March 2019 |
| 5 | 5 March 2019 |
| 6 | 5 March 2019 |

The first entry already carries a number (it was the first of the period) and that number is cleared by hand. All six are posted in one operation. The sort key orders them 4 March, then the three 5 March entries in creation order, then 6 March, then 7 March, and the numbers assigned are:

| Creation order | Number |
|---|---|
| 1 | `XMISC/2019/00002` |
| 2 | `XMISC/2019/00005` |
| 3 | `XMISC/2019/00006` |
| 4 | `XMISC/2019/00001` |
| 5 | `XMISC/2019/00003` |
| 6 | `XMISC/2019/00004` |

---

## 28. The proposed number shown before posting

A draft entry shows the number it *would* take, without consuming it.

```
 1. The placeholder is shown only when the number is empty or is the placeholder value, the
    accounting date is known, and the chain has **no** previous number under the strict
    search (that is, the entry would be the first of its period).
 2. The next sequence format and its values are computed as at posting.
 3. The counter value is increased by one and the format is filled.
 4. Otherwise no placeholder is shown.
```

The restriction to the first entry of a period is deliberate: for any later entry the proposed value would be wrong as soon as another user posts first.

---

## 29. Numbering holes per journal

The dashboard indicator is computed per journal and per numbering prefix:

```
 1. Group the journals by the fiscal lock date that applies to the acting user **with the
    exceptions ignored**, for that journal.
 2. For each group, collect the companies that are the journal company or a descendant of it.
 3. Select the distinct pairs (journal, numbering prefix) among the entries of those journals
    and companies that carry the gap flag and whose accounting date is strictly after that
    lock date.
 4. A journal reports a hole when at least one such pair exists.
```

Entries inside a locked period are deliberately excluded: their holes can no longer be corrected, so reporting them would be noise.

The list opened by the indicator shows every entry of the journals and prefixes concerned, so that the user sees the hole in context rather than the single flagged entry.

---

## 30. The display name of an entry

Two forms exist. `entities.md` states them in prose; this section states them as procedures, and the two must agree. The choice between them is made by one switch in the reading context.

### 30.1 The ordinary form

```
 1. Let NAME be the empty text.
 2. When, and only when, the entry is draft, set NAME to the one fixed word group of the
    document type of the entry. There are exactly seven and no other, all reproduced
    verbatim: "Draft Entry" for a plain entry, "Draft Invoice" for a customer invoice,
    "Draft Credit Note" for a customer credit note, "Draft Bill" for a vendor bill,
    "Draft Vendor Credit Note" for a vendor credit note, "Draft Sales Receipt" for a sales
    receipt and "Draft Purchase Receipt" for a purchase receipt.
    A posted entry and a cancelled entry add nothing here: no prefix and no suffix
    distinguishes the cancelled state in the display name.
 3. When the number exists and is not the placeholder "/":
      a. set NAME to NAME, a space and the number;
      b. remove the leading and trailing spaces of NAME, so a posted numbered entry ends up
         as the bare number;
      c. when the full display mode is requested:
           - when the entry has a counterpart, append a comma, a space and the counterpart
             name;
           - when the entry has an accounting date, append a comma, a space and that date
             formatted for the reading language.
 4. When the caller asks for the reference to be shown and the entry has one, append a space,
    an opening parenthesis, the shortened reference and a closing parenthesis. The reference
    is shortened by collapsing every run of whitespace to a single space and then, when the
    result is still longer than fifty characters, cutting it at a word boundary and adding a
    space and three dots between square brackets. The computation of the stored display name
    always asks for the reference.
 5. The result is NAME, which is the empty text when the entry is neither draft nor numbered.
```

**Worked example.** A posted vendor bill numbered `BILL/2026/03/0007` whose reference is "Order 4711" displays as `BILL/2026/03/0007 (Order 4711)`. The same document while draft and unnumbered displays as "Draft Bill". The same document while draft, unnumbered, read in the full display mode, displays as "Draft Bill (Order 4711)" — the counterpart and the date are added only in step 3c, which is reached only when a number exists.

### 30.2 The amount-total form

```
 1. Format the grand total of the entry in the currency of the entry, with its symbol and
    with exactly the decimals of that currency; call it AMOUNT.
 2. When the entry is a sale document (customer invoice, customer credit note or sales
    receipt) and it is posted:
      the result is the number, then — only when the entry has a reference — a space, a
      hyphen, a space and the reference, then the text " at " and AMOUNT. Stop.
 3. Otherwise let LABEL be:
      - the reference when the entry is a purchase document (vendor bill, vendor credit note
        or purchase receipt) and it has one, otherwise the number of such an entry;
      - the number for every other entry.
    An empty value gives an empty LABEL.
 4. When LABEL is not empty and the entry is draft:
      the result is LABEL, " at ", AMOUNT, a space and "(Draft)". Stop.
 5. When LABEL is not empty:
      the result is LABEL, " at " and AMOUNT. Stop.
 6. Otherwise the result is "Draft (", AMOUNT and ")".
```

**Worked examples**, with a company currency of euros and a grand total of 1 150.00:

| Entry | Result |
|---|---|
| posted customer invoice `INV/2026/00021`, no reference | `INV/2026/00021` at €1,150.00 |
| posted customer invoice `INV/2026/00021`, reference "Order 4711" | `INV/2026/00021` - Order 4711 at €1,150.00 |
| draft vendor bill, no number, reference "Order 4711" | Order 4711 at €1,150.00 (Draft) |
| posted plain entry `MISC/2026/03/0004` | `MISC/2026/03/0004` at €1,150.00 |
| draft plain entry with no number at all | Draft (€1,150.00) |

Step 6 is the only place the word "Draft" appears in this form for an entry that is not draft: an entry with no label at all is announced as a draft whatever its state.

### 30.3 The labels of the document types

The labels used by the type-name field of the entry, which is a different thing from the display name, are: Journal Entry, **Invoice** (not "Customer Invoice"), **Credit Note** (not "Customer Credit Note"), Vendor Bill, Vendor Credit Note, Sales Receipt and Purchase Receipt. The two overrides exist so that the name printed to a customer says simply "Invoice". Those labels are **not** the ones used by step 2 of the ordinary form, which uses the seven fixed word groups quoted there.

---

## 31. Which items are reportable for tax

A Journal Item is *tax exigible* — that is, reportable in the tax report now rather than at payment time — when at least one of these holds:

1. the entry it belongs to is marked "always tax exigible" (which is the case for every entry that is not an invoice-like document and that collects no cash-basis values);
2. the item carries neither an originating tax nor any tax (it only has grids);
3. the entry is itself a cash-basis entry;
4. the originating tax of the item is not exigible on payment;
5. at least one of the taxes of the item is not exigible on payment.

The last two are deliberately "at least one": an item carrying a mixture of cash-basis and ordinary taxes is reportable, which is why mixing the two on one item while they share a grid is forbidden.

---

## 32. Whether a journal item affects the tax report

Used by the tax lock check and by the accounting-date rule:

```formula
affects_tax_report( item ) = ( the item carries at least one tax )
                          or ( the item is a tax line )
                          or ( the item carries at least one grid whose applicability is taxes )

affects_tax_report( entry ) = at least one of its items affects the tax report
```

---

## 33. The dashboard figures of a liquidity journal

Only the figures that are aggregations of journal items are specified here; the statement-driven figures belong to `../payments-and-bank-reconciliation/`.

### The account-driven balance

```formula
journal_balance = sum over the items on the default account of the journal, whose entry is
                  not cancelled and whose display type is not a section, a subsection or a
                  note, of:
                      amount_currency   when the journal has a foreign currency different
                                        from the company currency
                      balance           otherwise
```

together with the number of such items. The aggregation deliberately ignores the journal of the item: an amount booked on the bank account from another journal is part of the bank balance.

### The running balance of the last statement

```formula
running_balance = closing balance of the most recent statement of the journal that has a
                  first-line index, taken in descending date then descending identifier order
                + the sum of the amounts of the transactions of the journal that belong to no
                  statement, whose entry is not cancelled, and whose internal index is greater
                  than or equal to the first-line index of that statement
```

with the closing balance read as zero when no such statement exists. The pair (whether anything at all was found, the resulting amount) is what the card shows.

### The outstanding accounts

```formula
inbound_outstanding_accounts  = the set of outstanding accounts of the money-in method lines
outbound_outstanding_accounts = the set of outstanding accounts of the money-out method lines
```

---

## 34. Worked end-to-end example

This example ties the whole domain together. Company currency: euro, rounding 0.01. Fiscal year ends 31 December. Today is 10 March 2026.

### Step 1 — the chart

A generic chart is loaded. Among the accounts created: `400000` Suppliers (Payable, reconcilable), `411000` Customers (Receivable, reconcilable), `550000` Bank (Bank and Cash), `600000` Purchases (Expenses), `700000` Sales (Income), `999999` Profit or Loss Appropriation (Current Year Earnings), `550001` Bank Suspense Account (Current Assets), `550002` Outstanding Receipts (Current Assets, reconcilable), `550003` Outstanding Payments (Current Assets, reconcilable), `999997` Cash Discount Gain, `999998` Cash Discount Loss, `999001` Cash Difference Gain, `999002` Cash Difference Loss, `580000` Liquidity Transfer.

Journals created: `INV` Sales, `BILL` Purchases, `MISC` Miscellaneous Operations, `EXCH` Exchange Difference, `CABA` Cash Basis Taxes, `BNK1` Bank.

### Step 2 — the opening entry

The opening date is 1 January 2026, so the entry is dated 31 December 2025 in the journal `MISC`. Amounts recorded: Bank 12 000.00 debit, Customers 5 000.00 debit, Suppliers 3 000.00 credit.

| Account | Debit | Credit | Label |
|---|---|---|---|
| 550000 Bank | 12 000.00 | | Opening balance |
| 411000 Customers | 5 000.00 | | Opening balance |
| 400000 Suppliers | | 3 000.00 | Opening balance |
| 999999 Profit or Loss Appropriation | | 14 000.00 | Automatic Balancing Line |

Running open balance: 12 000.00 + 5 000.00 − 3 000.00 = 14 000.00, so the balancing item is a credit of 14 000.00. The entry is posted and takes the number `MISC/2025/12/0001`.

### Step 3 — a miscellaneous accrual

On 5 March 2026 the accountant books an accrued expense of 1 200.00 in the journal `MISC`:

| Account | Debit | Credit |
|---|---|---|
| 600000 Purchases | 1 200.00 | |
| 400000 Suppliers | | 1 200.00 |

Posting assigns `MISC/2026/03/0001` (the March chain is new; the chain of December 2025 is a different period). The entry is balanced, no lock date is violated, and no hash is taken because `MISC` does not secure.

### Step 4 — a second entry the same month

On 7 March another entry of 800.00 is posted: it takes `MISC/2026/03/0002`.

### Step 5 — a mistake and its reversal

The 1 200.00 entry was wrong. On 31 March it is reversed with the reference "Reversal of: MISC/2026/03/0001":

| Account | Debit | Credit |
|---|---|---|
| 600000 Purchases | | 1 200.00 |
| 400000 Suppliers | 1 200.00 | |

The reversal takes `MISC/2026/03/0003`. Because the original is a plain entry and the reversal date is not in the future, the reversal is a cancelling one: it is posted immediately and the two items on `400000` are reconciled. The credit of 1 200.00 and the debit of 1 200.00 net to zero, so a Full Reconciliation is created and both items take its identifier as their matching number.

### Step 6 — a foreign-currency receivable

On 1 April 2026 a customer invoice is posted in the journal `INV` for 1 000.00 foreign units at a rate of 1.05 foreign units per euro:

| Account | Debit | Credit | Foreign amount |
|---|---|---|---|
| 411000 Customers | 952.38 | | 1 000.00 |
| 700000 Sales | | 952.38 | −1 000.00 |

The number is `INV/2026/00001` (a sale journal numbers yearly). The total in the document currency is 1 000.00, the signed company total is 952.38.

### Step 7 — the payment and the exchange difference

On 20 May the customer pays 1 000.00 foreign units; the rate that day is 1.10, so the bank receives 909.09 euros. The bank entry in the journal `BNK1`:

| Account | Debit | Credit | Foreign amount |
|---|---|---|---|
| 550000 Bank | 909.09 | | 1 000.00 |
| 411000 Customers | | 909.09 | −1 000.00 |

Matching the two receivable items: both publish the foreign currency, so the match is made in it for 1 000.00 units. Converting back, the invoice side gives 952.38 euros and the payment side 909.09 euros; they lie outside each other's tolerance ranges, so the matched amount in euros is the smaller, 909.09.

| Match | Amount (euros) | Amount on the debit side | Amount on the credit side |
|---|---|---|---|
| invoice ↔ payment | 909.09 | 1 000.00 foreign units | 1 000.00 foreign units |

The invoice item keeps 43.29 euros with nothing left in foreign units, so an exchange difference of +43.29 is produced in the journal `EXCH`, dated 20 May:

| Account | Debit | Credit | Foreign amount |
|---|---|---|---|
| 411000 Customers | | 43.29 | 0.00 |
| Exchange loss | 43.29 | | 0.00 |

The first of the two is matched with the invoice item, which then reaches zero in both currencies. A Full Reconciliation now covers the three receivable items; their matching number is its identifier.

### Step 8 — closing the quarter

On 5 July the accountant sets the Global Lock Date to 30 June 2026. The validation checks that no unreconciled bank transaction exists on or before that date. Afterwards:

- posting a new entry dated 15 June moves its accounting date out of the locked period;
- modifying the posted entry `MISC/2026/03/0001` is refused with "You cannot add/modify entries prior to and inclusive of: Global Lock Date (06/30/2026).";
- a lock exception granted to one accountant until the tenth of July, relaxing the Global Lock Date to 31 May 2026, lets that accountant modify a June entry while nobody else can.

### Step 9 — securing the half-year

On 15 July the administrator switches on the hash on the journal `MISC` and runs the secure-entries wizard up to 30 June 2026. The chains of `MISC` are examined:

- the chain with the prefix `MISC/2025/12/` holds `MISC/2025/12/0001`;
- the chain with the prefix `MISC/2026/03/` holds `MISC/2026/03/0001`, `0002` and `0003`.

Both are contiguous, neither holds an unreconciled bank transaction, so four entries are hashed in ascending counter order per chain. Each receives the message "This journal entry has been secured." From then on none of them can be reset to draft, renumbered, edited in a hashed field, or have an item deleted.

### Step 10 — the year end

On 31 December 2026 the accountant posts the profit-or-loss appropriation entry by hand, moving the balance of the income and expense accounts to the retained-earnings account, then sets the Global Lock Date to 31 December 2026. Once the audit is over, the administrator sets the Hard Lock Date to the same day; from that moment no exception can reopen the year and the Hard Lock Date can never be lowered.
