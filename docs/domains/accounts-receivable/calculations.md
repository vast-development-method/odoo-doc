# Calculations and algorithms of the Accounts Receivable domain

This file is the heart of the domain. It specifies, with exact rounding and evaluation order:

1. The dynamic line synchronisation — how product lines, discount allocation lines, cash rounding
   lines, tax lines, early payment discount lines and payment term lines are kept consistent.
2. The payment term due-date algorithm.
3. The payment term amount distribution algorithm in two currencies.
4. The early payment discount in its three computation modes.
5. Cash rounding under both strategies.
6. The document totals.
7. The numbering of documents.
8. The payment reference generators, including the modulo arithmetic written out.
9. The instalment and portal payment computations.
10. Worked numeric examples.

## Conventions used throughout

- **round(x, C)** means: round *x* to the number of decimal places of currency *C*, half away from
  zero on the currency's rounding step. The precise arithmetic is in
  [`../multi-currency/calculations.md`](../multi-currency/calculations.md). Where a currency is not
  named, the document currency is meant.
- **is_zero(x, C)** means: *x* rounded to currency *C* equals zero.
- **document currency** is the currency of the document; **company currency** is the currency of the
  company that owns it.
- **rate** is the document's currency rate: the number of units of the document currency that one
  unit of the company currency buys. Therefore

```formula
amount_in_company_currency = round( amount_in_document_currency ÷ rate , company_currency )
```

  and, symmetrically, an amount known in company currency is multiplied by the rate to reach the
  document currency. When the two currencies are identical the rate is one and both amounts are
  equal.
- **sign** is the direction sign of the document: `+1` for an outbound document (customer credit
  note, vendor bill, purchase receipt) or a plain entry, `−1` for an inbound document (customer
  invoice, sales receipt, vendor credit note).
- A **balance** is an amount in company currency carrying the accounting sign: positive is a debit,
  negative is a credit.
- An **amount in currency** is the same amount expressed in the line's currency, with the same sign.

---

# 1. The dynamic line synchronisation

## 1.1 What the problem is

A customer document holds two kinds of information that must always agree:

- the *commercial* view — what is billed, how much, with which taxes, with which discount, payable
  in how many instalments;
- the *accounting* view — a balanced set of journal items.

The user only edits the commercial view (the product lines, the payment term, the cash rounding
method, the currency, the partner). Every other journal item is derived. The synchronisation is the
machinery that derives them, and it must run on **every** write to the document or to any of its
lines, not only on creation, because a single change (say, raising a quantity) ripples into the tax
lines, the discount lines, the rounding line, the early payment discount lines and the instalment
amounts.

The design is: each derived line family declares a **key** (what makes two lines of that family the
same line) and a **needed map** (which lines of that family should exist and with which amounts).
The synchroniser compares the state before the write with the state after, and creates, updates,
deletes or *recycles* lines so the two agree — while never destroying a value the user typed by hand.

## 1.2 The line families and their keys

| Family | Display type | Key fields | Needed map produced by |
| --- | --- | --- | --- |
| Payment term | `payment_term` | document, maturity date, early discount deadline | the document's "needed terms" |
| Discount allocation | `discount` | account, document, currency rate | each product line's "discount allocation needed" |
| Early payment discount | `epd` | account, analytic distribution, taxes, tax grids, document | each product line's "early discount needed" |
| Tax | `tax` | the tax repartition grouping key (see step 5) | the tax engine |
| Cash rounding | `rounding` | there is at most one such line per document | the cash rounding routine |
| Non-deductible (purchase only) | `non_deductible_product`, `non_deductible_product_total`, `non_deductible_tax` | rebuilt wholesale | the non-deductible routine |

An early payment discount line whose key would be empty (because the payment term no longer carries a
discount in the "always" mode) is still tracked, under a synthetic key made of its own identifier, so
that it can be cleaned up rather than orphaned.

## 1.3 The order of the synchronisation stack

The synchronisers are *nested context managers*: each one takes a snapshot before the write happens,
the write then happens in the innermost scope, and each one reconciles its family afterwards, in
reverse order of entry. They are entered in ascending order of their sequence number, which means
that the **last** one entered is the **first** one to reconcile after the write.

| Sequence | Synchroniser | Applies to |
| --- | --- | --- |
| 10 | payment term lines | invoices only |
| 20 | automatic balancing line | plain entries only, and only when taxes are involved |
| 30 | cash rounding line | invoices only |
| 40 | discount allocation lines | invoices only |
| 50 | tax lines | invoices, and any entry whose lines carry taxes or tax repartitions |
| 60 | non-deductible base lines | invoices only (effective on purchase documents) |
| 70 | early payment discount lines | invoices only |
| 80 | partner propagation | invoices only |

So the reconciliation order after a write is: 80, 70, 60, 50, 40, 30, 20, 10 — partner first,
payment term lines last. That is the key ordering property: **the payment term lines are recomputed
after everything else**, so they always see the final totals, including the taxes, the cash rounding
and the early payment discount lines.

Three further properties hold:

- The whole stack is wrapped in a recursion guard. If a synchroniser's own writes would re-enter the
  stack, the re-entry is skipped. That is what makes the fixed point reachable in one pass.
- A caller may suppress the stack entirely (for example while reversing a plain journal entry, where
  the lines must be copied verbatim and negated).
- A posted document is never synchronised: every synchroniser skips documents whose status is not
  draft.

## 1.4 The generic reconciliation algorithm for a keyed family

This algorithm is run by the payment term, discount allocation and early payment discount
synchronisers. Let *F* be the family, *K* its key field, *N* its needed-map field and *D* its dirty
flag.

**Before the write:**

1. Build `existing_before`: the mapping from every line of the document whose key *K* is non-empty
   to that key.
2. Build `needed_before`: the aggregation of the needed maps *N* of all relevant records (see
   1.5 for the aggregation rule).
3. Collect the records whose dirty flag *D* is true, and clear the flag on all of them.

**The write happens.**

**After the write:**

4. Recollect the dirty records. If none is dirty, stop: nothing in this family can have changed.
5. Rebuild `existing_after` and `needed_after` the same way.
6. Drop from `needed_before` every key that names a line identifier that no longer exists in the
   database. (This prevents resurrecting a line the user explicitly deleted.)
7. Build the *before-to-after* key translation: for each line present both before and after, map its
   old key to its new key.
8. **If `needed_after` equals `needed_before`, stop.** Nothing in the need changed, therefore any
   difference in the lines is a deliberate user edit and must be preserved.
9. **If `needed_before` was empty and the set of non-trivial existing keys changed, stop.** ("Non
   trivial" means a key that does not merely name a line identifier.) This is the case where the
   caller created the derived lines itself, by hand or by import; the synchroniser must not fight it.
10. Group `existing_after` by key, so that several lines may share a key.
11. Compute the deletions:
    - every line that existed before whose old key is absent from `needed_after`, whose old key is
      still present in the after-grouping, and whose translated new key is also absent from
      `needed_after`;
    - plus every line that exists after whose key is absent from `needed_after` and that is not
      already in the list.
12. Compute the creations: every key of `needed_after` that has no line in the after-grouping.
13. Compute the updates: for every key of `needed_after` that has lines, every such line at least one
    of whose values differs from the needed value.
14. **Recycle**: while there is both something to delete and something to create, pop one of each and
    *rewrite* the line to be deleted with the key, the values and the display type of the line to be
    created. This preserves line identifiers across a change of maturity date or account, which
    matters because other records point at those lines.
15. Delete whatever remains in the deletion list (with the automatic-deletion guard bypassed, since
    this is the platform itself deleting its own lines).
16. Create whatever remains in the creation list, in a cleaned context so that no stray default
    leaks into the new lines.
17. Apply the updates.

## 1.5 Aggregating needed maps

Several records may need the same derived line (for example two product lines that need the same
early payment discount grouping key). The aggregation rule is:

1. Start from an empty result.
2. For each contributing map, for each key and value pair:
   - if the key is new, copy the value;
   - otherwise, add every **monetary** field of the value into the accumulated value, and remember
     whether *any* accumulated monetary field ended up non-zero. If all of them ended at zero, drop
     the key entirely — a derived line whose every amount cancels out must not exist.
3. A contributing map that is the literal "invalidated" marker is skipped; the write that invalidated
   it will have scheduled a recomputation.
4. Finally, convert every floating-point value to the representation the storage layer would give it,
   so that a comparison against the stored line never fails for a representation-only difference.

## 1.6 Step-by-step: what happens on one write

The following is the complete, ordered algorithm. It is what an implementation must reproduce.

**Preconditions.** A document exists (possibly unsaved). The caller is writing one or more fields on
the document or on its lines. The document's status is draft (otherwise steps 2 to 11 are skipped).

1. **Enter the recursion guard.** If the stack is already active for this document, perform the write
   and return; do not re-enter.

2. **Snapshot for the payment term family.** Record the existing payment term keys, the document's
   needed terms, and clear the needed-terms-dirty flag.

3. **Snapshot for the automatic balancing line** (plain entries only). Record whether the entry
   currently carries any tax.

4. **Snapshot for the cash rounding line.** (Nothing is recorded; the routine recomputes from
   scratch afterwards.)

5. **Snapshot for the discount allocation family.** Record the existing discount allocation keys, the
   needed maps of the product lines, and clear their dirty flags.

6. **Snapshot for the tax family.** Record, per document:
   - the document-level values: currency, partner, document type, currency rate, document date;
   - for every *base* line (display type `product`, `epd`, `rounding` or `non_deductible_product`):
     the grouping-key fields plus, on an invoice, the unit price, the quantity, the discount and the
     deductibility; on a non-invoice, the amount in currency instead;
   - for every *tax* line: the amount in currency, the balance and the analytic distribution.

7. **Snapshot for the non-deductible family.** Record a multiset of (label, subtotal, taxes,
   deductibility, account) over the product lines.

8. **Snapshot for the early payment discount family.** Record the existing keys, the needed maps of
   the product lines, and clear their dirty flags.

9. **Snapshot for the partner propagation.** Record the commercial entity of the document.

10. **Snapshot for the line-level invoice synchronisation** (the inner scope that keeps a line's own
    derived values, such as its currency and its label, consistent with the document).

11. **Perform the write.**

12. **Reconcile the partner** (sequence 80). If the commercial entity changed, write it onto every
    line of the document.

13. **Reconcile the early payment discount lines** (sequence 70) using the generic algorithm of 1.4
    with the early-discount key and needed map. The needed map itself is computed as specified in
    section 4.3.

14. **Reconcile the non-deductible lines** (sequence 60). Purchase documents only; specified in
    [`../accounts-payable/calculations.md`](../accounts-payable/calculations.md).

15. **Reconcile the tax lines** (sequence 50). This is specified in full in section 1.7.

16. **Reconcile the discount allocation lines** (sequence 40) using the generic algorithm of 1.4.
    The needed map is computed as specified in section 5.

17. **Recompute the cash rounding line** (sequence 30) as specified in section 6.

18. **Reconcile the automatic balancing line** (sequence 20). Plain entries only: when the entry
    carries taxes (before or after), the platform makes sure the entry balances by creating or
    updating a line labelled "Automatic Balancing Line" on the journal's default account, or, absent
    one, the company's journal suspense account. When taxes were removed entirely, every tax line is
    deleted and every tax grid cleared first.

19. **Reconcile the payment term lines** (sequence 10) using the generic algorithm of 1.4 with the
    term key and the document's needed terms. The needed terms are computed as specified in
    section 3.

**Postconditions.**

- Every derived line family agrees with its need.
- The sum of the balances of all lines is zero (the balance invariant enforced by the ledger).
- No line the user typed has been silently changed.

## 1.7 Reconciling the tax lines in detail

For each document of the batch whose status is draft:

1. Collect the current tax lines (those with a tax repartition) and the current base lines (display
   types `product`, `epd`, `rounding`, `non_deductible_product`).

2. Decide the **rounding source**, a value that answers the question "should the existing tax line
   amounts be taken as authoritative, or recomputed from the bases?". The decision cascade, first
   match wins:

   | Condition | Rounding source |
   | --- | --- |
   | The document is an invoice **and** its currency or its document type changed | recompute from the bases |
   | A base line that carried taxes was removed | keep the tax amounts only if some tax line value also changed |
   | At least one base line changed | keep the tax amounts when **either** none of the changed lines has taxes now or had taxes before, **or** the tax line set itself changed or some tax line field is currently protected from recomputation (which is how a manual tax amount is marked). Additionally: if the amounts were to be kept **and** any changed line carries an explicit amount in currency or balance, stop — the caller supplied everything and nothing must be recomputed. |
   | The currency rate changed (and nothing above matched) | the special source "reapply the currency rate": keep the tax amounts in document currency and recompute only their company-currency counterparts |
   | none of the above | stop; nothing to do |

3. Build the base lines and the tax lines for the tax engine:
   - one base line per product line, carrying the unit price, the quantity, the discount, the
     document's currency rate, the direction sign and the label; when the product line came from a
     global discount or a down payment, it is flagged as such so that the engine treats it specially;
   - when the document is stored: one base line per early payment discount line, one per cash
     rounding line that is not itself a tax line, and one per non-deductible base line. Each of these
     is passed with quantity one, a unit price equal to the signed amount in currency, and the "the
     amount given is already tax-excluded" mode;
   - when the document is **not yet stored**, the early payment discount lines and the non-deductible
     lines do not exist as records yet, so equivalent base lines are synthesised from the needed maps
     of the product lines;
   - one tax line per existing tax line, carrying its amounts.
   Under the "reapply the currency rate" source, each tax line's balance is first replaced by its
   amount in currency divided by the new rate, rounded to the company currency.

4. Let the tax engine compute the rounded tax details for every base line (this is the taxes domain's
   job; see [`../taxes/calculations.md`](../taxes/calculations.md)). The engine rounds the raw
   amounts per tax first, then per base line, then distributes the residual difference over the base
   lines, so that the sum of the rounded per-line amounts equals the rounded total per tax.

5. Ask the engine to prepare the tax lines. The engine groups every tax repartition contribution by
   the **tax repartition grouping key**, which is:

   | Component | Value |
   | --- | --- |
   | partner | the base line's partner |
   | currency | the base line's currency |
   | analytic distribution | the base line's distribution, unless the tax is not analytic **and** the repartition line is used in the tax closing, in which case none |
   | account | the repartition line's account, or, when it has none, the base line's own account |
   | taxes | the taxes the repartition line propagates onto the tax line |
   | tax grids | the tax grids of the repartition line |
   | tax repartition line | the repartition line itself |
   | grouped tax | the tax group record, when the tax is part of a group |

   For each grouping key it accumulates: the tax base amount, the amount in currency and the balance,
   each multiplied by the base line's sign. A grouping key whose accumulated amounts are zero in both
   currencies is dropped, unless the repartition explicitly asks to keep zero lines.

   The engine returns four lists: tax lines to add, tax lines to delete, tax lines to update (each
   with its grouping key and its new amounts), and base lines to update (each with its new tax grids,
   its new amount in currency and its new balance).

6. Handle the non-deductible tax line (purchase documents only): if no base line is non-deductible
   any more, delete it; otherwise compute its amount as the negated signed difference between the
   tax-included and the tax-excluded totals of the non-deductible base lines, and update or create it
   on the journal's non-deductible account, or the journal's default account, labelled
   "private part (taxes)", with a sequence one above the highest line sequence.

7. Apply the results:
   - group the base line updates by (currency, set of values) and write each group in one operation,
     writing only where a value actually differs;
   - delete the tax lines to delete, with the automatic-deletion guard bypassed;
   - create the tax lines to add, each with display type `tax` and the document identifier;
   - apply the tax line updates the same way as the base line updates.

**Why the base lines are updated too.** A product line's balance and amount in currency are *not*
what the user typed: they are the tax-excluded totals the engine computed, which may differ from
quantity × price × (1 − discount) by a rounding cent that the engine deliberately shifted onto one
line so that the per-tax totals add up. That shifted cent is the "delta" in the formula

```formula
product_line_amount_currency = sign × ( total_excluded_currency + delta_total_excluded_currency )
product_line_balance         = sign × ( total_excluded          + delta_total_excluded )
```

## 1.8 What the user sees while editing

Two line values are computed outside the synchronisation, on every keystroke, because the
synchronisation only runs when the document is saved:

```formula
price_subtotal = total_excluded_currency of the line's own tax computation
price_total    = total_included_currency of the line's own tax computation
```

Both are computed by running the tax engine on that single line alone, with the company's rounding
method. They are stored, and they are what the line list shows. They are *display* values: the
authoritative accounting amounts are the balance and the amount in currency produced in step 7.

---

# 2. Due date algorithm

Let *R* be the reference date: the document date if set, otherwise the accounting date, otherwise
today. For a payment term line with day count *n*:

| Delay type | Due date |
| --- | --- |
| Days after invoice date | *R* + *n* days |
| Days after end of month | (last day of the month of *R*) + *n* days |
| Days after end of next month | (last day of the month following the month of *R*) + *n* days |
| Days end of month on the *d* | see below |

For the fourth type, let *d* be the "days on the next month" text read as an integer; when the text
cannot be read as an integer, *d* is taken as one.

- If *d* ≤ 0: the due date is the last day of the month of (*R* + *n* days).
- Otherwise: take *R* + *n* days, move forward exactly one month, then set the day of month to *d*.
  When the target month is shorter than *d*, the last day of the target month is used.

**Worked examples** with *R* = 2026-01-20:

| Delay type | *n* | *d* | Due date | Why |
| --- | --- | --- | --- | --- |
| Days after invoice date | 0 | — | 2026-01-20 | same day |
| Days after invoice date | 45 | — | 2026-03-06 | 20 January + 45 days: 11 days left in January, 28 in February, 6 in March |
| Days after end of month | 0 | — | 2026-01-31 | last day of January |
| Days after end of month | 15 | — | 2026-02-15 | 31 January + 15 days |
| Days after end of next month | 0 | — | 2026-02-28 | last day of February 2026 |
| Days end of month on the | 0 | 10 | 2026-02-10 | 20 January + 0 days → 20 January; +1 month → 20 February; day set to 10 → 10 February |
| Days end of month on the | 15 | 10 | 2026-03-10 | 20 January + 15 days → 4 February; +1 month → 4 March; day set to 10 → 10 March |
| Days end of month on the | 10 | 0 | 2026-01-31 | *d* ≤ 0, so the last day of the month of 30 January |

---

# 3. The payment term amount distribution

## 3.1 Inputs

The document supplies:

| Input | Meaning |
| --- | --- |
| reference date | the document date, else the accounting date, else today |
| document currency | the currency of the document |
| company | the owning company, hence the company currency |
| tax amount in currency | the accounting-signed tax total in document currency |
| tax amount | the accounting-signed tax total in company currency |
| untaxed amount in currency | the accounting-signed net total in document currency |
| untaxed amount | the accounting-signed net total in company currency |
| sign | see below |
| cash rounding | the document's cash rounding method, or none |

All four amounts carry the **accounting** sign, which for a customer invoice makes them positive
(the receivable side is a debit) and for a customer credit note negative.

There are two ways these are obtained:

- **Document already stored.** Let *s* be `+1` when the document is inbound (receipts included) and
  `−1` otherwise. Then the tax amount in currency is the stored customer-facing tax total multiplied
  by *s*; the tax amount is the stored signed tax total; the untaxed amount in currency is the
  stored customer-facing net total multiplied by *s*; the untaxed amount is the stored signed net
  total. The sign parameter handed to the distribution is this same *s*.
- **Document not yet stored** (a new record being edited in the form, whose lines have no database
  identifiers yet): the totals are recomputed from scratch. The base and tax lines are built as in
  step 3 of section 1.7 but without taking existing tax lines as authoritative, the accounting data
  is added, the tax lines are prepared, and then, with *s* now being the **direction sign**:

```formula
untaxed_amount_currency = Σ over base line updates of ( s × amount_currency )
untaxed_amount          = Σ over base line updates of ( s × balance )
tax_amount_currency     = Σ over tax lines to add   of ( s × amount_currency )
tax_amount              = Σ over tax lines to add   of ( s × balance )
```

Because a base line's amount in currency is itself the direction sign times the tax-excluded total,
multiplying by the direction sign again yields a positive figure for a customer invoice, matching
the stored case. The sign parameter handed to the distribution in this branch is the direction sign,
which is the opposite of the stored case; it only affects fixed-amount payment term lines.

## 3.2 The distribution

```formula
total_amount          = tax_amount + untaxed_amount
total_amount_currency = tax_amount_currency + untaxed_amount_currency
rate                  = | total_amount_currency ÷ total_amount |     (zero when total_amount is zero)
```

Note this **effective rate**: it is derived from the two totals rather than read from the document's
currency rate field. Using it guarantees that the instalments, expressed back in company currency,
add up exactly to the document's company-currency total.

Then, walking the payment term lines **in their stored order** (which is creation order), with a
running residual initialised to the two totals:

1. Compute the line's due date by the algorithm of section 2.
2. Decide the line's two amounts:
   - **If this is the last line of the term** — whatever its kind, percent or fixed — the amounts are
     the whole remaining residual:

     ```formula
     company_amount = residual_amount
     foreign_amount = residual_amount_currency
     ```

     This is the *balance rule*. It is what makes the instalments add up exactly to the total, with no
     rounding drift, whatever the percentages.
   - **Else if the line is a fixed-amount line**:

     ```formula
     foreign_amount = sign × round( value_amount , document_currency )
     company_amount = sign × round( value_amount ÷ rate , company_currency )
     ```

     (both zero when the rate is zero). The fixed amount is therefore expressed in the *document*
     currency.
   - **Else (a percent line)**:

     ```formula
     company_amount = round( total_amount          × value_amount ÷ 100 , company_currency )
     foreign_amount = round( total_amount_currency × value_amount ÷ 100 , document_currency )
     ```

3. **Cash rounding correction**, applied only when a cash rounding method is set and this is *not*
   the last line:

   ```formula
   difference = cash_rounding_difference( document_currency , foreign_amount )
   if not is_zero( difference , document_currency ):
       foreign_amount = foreign_amount + difference
       company_amount = round( foreign_amount ÷ rate , company_currency )    (zero when rate is zero)
   ```

   The invariant this preserves: the input total in document currency is assumed to be already cash
   rounded; each non-final instalment is made cash rounded; therefore the residual left for the last
   line — the total minus a sum of cash-rounded amounts — is itself cash rounded.

4. Subtract both amounts from the running residual and record the line.

## 3.3 From the distribution to the needed terms

Each produced instalment becomes one entry of the document's needed terms:

- **Key**: the triple (document, maturity date as a date, early discount deadline).
- **Value**: the balance (the company amount), the amount in currency (the foreign amount), the early
  discount deadline, the early discount balance and the early discount amount in currency.

When two instalments fall on the same date they merge: the key is identical, so the balances and the
amounts in currency are added together. That is why a term with, say, two lines both at zero days
produces a single receivable line.

When the document has **no payment term**, the needed terms hold exactly one entry:

- **Key**: (document, the due date, no discount deadline, zero discount balance, zero discount amount).
- **Value**: the balance is the document's accounting-signed total in company currency; the amount in
  currency is its accounting-signed total in document currency.

When the document is not an invoice, or has no invoice lines, the needed terms are empty and no
payment term line is produced.

## 3.4 The due date of the document

```formula
invoice_date_due = max over the needed term keys of ( maturity date )
```

falling back to the previously stored due date, and then to today. Because the needed terms are
recomputed whenever the payment term, the document date, the currency, the total or the due date
change, the displayed due date always equals the latest instalment.

## 3.5 Worked example: thirty percent immediately, balance in forty-five days

**Setup.** A customer invoice dated 2026-01-20, in the company currency (so the rate is one and both
currencies coincide), with a net total of 1 000.00 and one tax of 21 % giving 210.00, hence a gross
total of 1 210.00. The payment term has two lines, in this order:

| Order | Kind | Amount | Delay type | Days |
| --- | --- | --- | --- | --- |
| 1 | Percent | 30 | Days after invoice date | 0 |
| 2 | Percent | 70 | Days after invoice date | 45 |

**Signs.** For a stored customer invoice the totals handed to the distribution are the
*accounting-signed* totals, which for a customer invoice are positive: `untaxed_amount` = 1 000.00,
`tax_amount` = 210.00, therefore `total_amount` = 1 210.00. Likewise in document currency. The
effective rate is |1 210.00 ÷ 1 210.00| = 1. The sign parameter handed to the distribution is `+1`
because the document is inbound; it is used only by fixed-amount lines, of which there are none
here.

**Line 1 (percent, not the last line).**

```formula
company_amount = round( 1210.00 × 30 ÷ 100 ) = round( 363.00 ) = 363.00
foreign_amount = round( 1210.00 × 30 ÷ 100 ) = 363.00
```

No cash rounding is configured, so no correction. Due date: 20 January 2026 + 0 days = 2026-01-20.
Residual becomes 1 210.00 − 363.00 = 847.00.

**Line 2 (the last line — the balance rule).**

```formula
company_amount = residual = 847.00
foreign_amount = 847.00
```

Due date: 20 January 2026 + 45 days = 2026-03-06. Residual becomes zero.

**Resulting receivable lines.** Two journal items on the customer's receivable account. Their
balances are the instalment amounts as computed, that is **positive**, hence two **debits** — which
is correct: a customer invoice debits the receivable account.

| Instalment | Maturity date | Balance (debit) | Label |
| --- | --- | --- | --- |
| 1 | 2026-01-20 | 363.00 | `installment #1` |
| 2 | 2026-03-06 | 847.00 | `installment #2` |

The document's due date is the latest maturity date, 2026-03-06. The two amounts add to 1 210.00
exactly, which equals the sum of the credits (1 000.00 of revenue plus 210.00 of tax).

**Sign note.** The distribution receives a sign parameter that differs between a document that is
already stored and one that is still being typed in the form: for a stored document it is `+1` when
the document is inbound and `−1` otherwise; for a document not yet stored it is the direction sign,
which is the opposite. The parameter is consumed only by fixed-amount payment term lines, so the
difference is observable only on a term that mixes a fixed line with other lines while the document
is unsaved.

**Why the balance rule matters.** Suppose instead the term were three equal thirds of a total of
100.00. Percent lines would give 33.33 and 33.33, and the last line, under the balance rule, would
give 100.00 − 33.33 − 33.33 = 33.34. Without the balance rule the three rounded thirds would be
33.33 each and the instalments would fall one cent short of the total, leaving the document
permanently unbalanced.

## 3.6 Worked example with cash rounding interaction

Same invoice as 3.5 but with a cash rounding method of 0.05, nearest, and a gross total of 1 210.00
(already a multiple of 0.05).

- Line 1: 30 % of 1 210.00 is 363.00. The cash rounding difference of 363.00 at a step of 0.05 is
  round-to-nearest-0.05(363.00) − 363.00 = 363.00 − 363.00 = 0.00. No correction.
- Line 2 (last): 1 210.00 − 363.00 = 847.00, which is a multiple of 0.05. Consistent.

Now change the percentage to 33 %:

- Line 1: 33 % of 1 210.00 is 399.30. The difference at a step of 0.05 is 399.30 − 399.30 = 0.00
  (399.30 is already a multiple of 0.05). No correction.
- Change it to 37 %: 37 % of 1 210.00 is 447.70, already a multiple of 0.05; no correction.
- Change it to 31 %: 31 % of 1 210.00 is 375.10. The nearest multiple of 0.05 is 375.10 itself;
  no correction.
- Change it to 17 %: 17 % of 1 210.00 is 205.70 — a multiple of 0.05. To see a correction, take a
  total of 1 207.35 with a step of 0.05 and a first line of 30 %: 30 % of 1 207.35 is 362.205, which
  rounds to the currency as 362.21 (wait — the currency has two decimals, so 362.205 rounds to
  362.21 half away from zero). The nearest multiple of 0.05 to 362.21 is 362.20, so the difference
  is −0.01 and the first instalment becomes 362.20. The last instalment then takes
  1 207.35 − 362.20 = 845.15, which is itself a multiple of 0.05. Both instalments are cash-rounded
  and their sum is the total.

---

# 4. Early payment discount

## 4.1 The three computation modes

A payment term may carry a single early payment discount: a percentage granted if the customer pays
within a number of days of the document date. Because the discount reduces the amount actually
received, it also — in most jurisdictions — reduces the tax base. Jurisdictions disagree on *when*
that reduction is recognised, which is what the three modes encode.

| Value | Label | Meaning |
| --- | --- | --- |
| `included` | On early payment | The discount reduces the taxable base **only if and when** the customer actually pays early. The invoice is issued at full value; the reduction of base and tax is booked at payment time. |
| `excluded` | Never | The discount never reduces the taxable base. Only the net part is discounted; the tax is always owed in full. The invoice is issued at full value; at payment time only a net write-off is booked. |
| `mixed` | Always (upon invoice) | The tax is computed on the already-discounted base **at invoicing time**, whether or not the customer pays early. The invoice therefore carries extra journal items (the early payment discount lines) that shift base from the income accounts. |

Defaults by company country: Belgium → `mixed`; the Netherlands → `excluded`; every other country →
`included`.

## 4.2 The discounted amount to pay

Two formulas, both used and both rounded to the document currency.

**Formula A — the amount due after discount.** It takes two inputs: the gross total *G* and the tax
total *X* of the document. With *p* = `discount_percentage ÷ 100`:

```formula
if mode is 'excluded' or 'mixed':
    discount_amount = ( G − X ) × p
else:                                                              (mode 'included')
    discount_amount = G × p
amount_due = round( G − discount_amount , currency )
```

Because *G* − *X* is the **net** total, the first branch reads: in the `excluded` and `mixed` modes
the discount is a percentage of the net amount only, so the tax stays fully payable. In the
`included` mode the discount is a percentage of the whole gross amount, because the tax will be
reduced too.

This is the formula the printed document uses to state "*amount* due if paid before *date*", and the
one the payment-term preview uses (the preview passes a tax total of zero, so both branches reduce
to `round( G × (1 − p) )`).

When a cash rounding method is configured on the document being previewed, the amount due is then
cash-rounded:

```formula
difference = cash_rounding_difference( currency , amount_due )
if not is_zero( difference , currency ):
    amount_due = round( amount_due + difference , currency )
```

**Formula B — the discounted amounts stored on the receivable lines.** This is the authoritative one
for the document's own data. It is applied by the payment term distribution and works on the
*signed* totals of section 3.1:

```formula
if mode is 'excluded' or 'mixed':
    discount_balance         = round( total_amount          − untaxed_amount          × p , company_currency )
    discount_amount_currency = round( total_amount_currency − untaxed_amount_currency × p , document_currency )
else:                                                              (mode 'included')
    discount_balance         = round( total_amount          × (1 − p) , company_currency )
    discount_amount_currency = round( total_amount_currency × (1 − p) , document_currency )
```

where *p* is `discount_percentage ÷ 100`. Here the totals are the *signed* totals of 3.1, so
`untaxed_amount` is the net part and `total_amount` is the gross part, and the first branch reads
correctly as "the gross total less a percentage of the net part".

When cash rounding applies, the discounted amount is corrected the same way as an instalment:

```formula
difference = cash_rounding_difference( document_currency , discount_amount_currency )
if not is_zero( difference , document_currency ):
    discount_amount_currency = discount_amount_currency + difference
    discount_balance         = round( discount_amount_currency ÷ rate , company_currency )   (zero when rate is zero)
```

The discount **deadline** is:

```formula
discount_date = reference_date + discount_days
```

where the reference date is the document date (else the accounting date, else today).

**Cross-check on the worked example of 4.4.** In the `excluded` mode, formula A with *G* = 1 210.00
and *X* = 210.00 gives a discount of (1 210.00 − 210.00) × 0.02 = 20.00 and an amount due of
1 190.00; formula B with `total_amount` = −1 210.00 and `untaxed_amount` = −1 000.00 gives
−1 210.00 − (−1 000.00 × 0.02) = −1 190.00, the same figure in accounting sign. In the `included`
mode, formula A gives 1 210.00 × 0.98 = 1 185.80 and formula B gives −1 210.00 × 0.98 = −1 185.80.

## 4.3 The `mixed` mode: the early payment discount lines on the invoice

Only in this mode does the invoice itself carry extra journal items. The algorithm, run per document
that has at least one product line with taxes and a payment term with an early discount in the
`mixed` mode:

1. Let *p* be `discount_percentage ÷ 100` and let the label suffix be the percentage formatted with
   a per-cent sign.
2. Build one base line per product line of the document (unit price, quantity, discount, rate, sign),
   add the tax details, and round them.
3. **Split out the taxes that cannot be discounted.** A tax can be discounted unless its amount kind
   is a fixed amount or a code-driven amount. Every non-discountable tax is dispatched into a
   separate base line, so that the discount never touches it.
4. Aggregate the base lines by the grouping key (account, analytic distribution, the set of taxes
   actually applied).
5. For each non-empty grouping key, compute the discount:

```formula
epd_amount_currency = round( sign × total_excluded_currency of the group × p , document_currency )
epd_balance         = round( sign × total_excluded          of the group × p , company_currency )
```

6. Produce **two** needed entries per grouping key:
   - a *base-shift* entry, keyed by (document, the grouping key, display type `epd`), labelled
     "Early Payment Discount (*the percentage*)", which will carry the **negative** of the discount:
     it removes the discounted part from the taxed base, so the tax engine computes the tax on the
     reduced base;
   - a *counterpart* entry, keyed by (document, the account, the analytic distribution, display type
     `epd`) with **no taxes**, labelled identically, which will carry the **positive** discount: it
     puts the removed amount back, untaxed, so that the net total of the invoice is unchanged.
7. **Distribute** the two totals across the individual product lines of the group so that each line
   contributes proportionally and the rounded parts still add up to the group total. The distribution
   uses the raw (unrounded) tax-excluded amount of each base line as the weight, and the smooth
   delta-distribution rule of the tax engine, which rounds each share to the currency's precision and
   allocates the remaining units one by one to the largest weights. For each line:

```formula
base_shift_entry.amount_currency   −= share of epd_amount_currency
counterpart_entry.amount_currency  += share of epd_amount_currency
base_shift_entry.balance           −= share of epd_balance
counterpart_entry.balance          += share of epd_balance
```

8. The resulting per-product-line maps become the "early discount needed" values, which the generic
   reconciliation of 1.4 turns into actual `epd` lines.

Because the `epd` lines are themselves base lines for the tax engine (step 3 of 1.7 includes them),
the tax lines that the engine then produces are computed on the *reduced* base. The net total of the
invoice is unchanged, because the base-shift and the counterpart cancel.

## 4.4 Worked example: two percent within ten days, all three modes

**Setup.** A customer invoice dated 2026-01-20 for a single product line: quantity 1, unit price
1 000.00, no line discount, one tax of 21 % (not price-included, not fixed). Company currency equals
document currency. The payment term is a single line of 100 % at 30 days, with an early payment
discount of 2 % within 10 days. The document's income account is "Product Sales"; the company's cash
discount write-off loss account is "Cash Discount Granted".

Common quantities: *p* = 0.02; discount deadline = 2026-01-20 + 10 days = 2026-01-30.

### Mode `included` — "On early payment"

**On the invoice.** Nothing changes. The invoice carries:

| Line | Account | Debit | Credit |
| --- | --- | --- | --- |
| product | Product Sales | | 1 000.00 |
| tax 21 % | Tax Received | | 210.00 |
| instalment | Customers | 1 210.00 | |

The receivable line additionally stores: discount deadline 2026-01-30, discount amount in currency
`round(1 210.00 × (1 − 0.02)) = 1 185.80`, discount balance 1 185.80.

**If the customer pays 1 185.80 on 2026-01-25.** The difference of 24.20 is written off against both
the base and the tax, because in this mode the discount reduces the taxable base. The write-off, as
computed by the counterpart algorithm of section 4.5, is:

| Line | Account | Debit | Credit |
| --- | --- | --- | --- |
| write-off base | Cash Discount Granted | 20.00 | |
| write-off tax | Tax Received | 4.20 | |

and the 1 210.00 receivable is fully reconciled against the 1 185.80 payment plus the 24.20
write-off. The tax actually declared becomes 210.00 − 4.20 = 205.80, on a base of 980.00.

**If the customer pays 1 210.00 on 2026-02-19** (after the deadline), nothing special happens: the
discount is simply not granted.

### Mode `excluded` — "Never"

**On the invoice.** Identical to the `included` mode; the invoice is issued at full value. The
receivable line stores: discount deadline 2026-01-30, discount amount in currency
`round(1 210.00 − 1 000.00 × 0.02) = round(1 210.00 − 20.00) = 1 190.00`, discount balance 1 190.00.

**If the customer pays 1 190.00 on 2026-01-25.** The difference of 20.00 is written off entirely
against the net account; the tax is untouched:

| Line | Account | Debit | Credit |
| --- | --- | --- | --- |
| write-off | Cash Discount Granted | 20.00 | |

The tax declared stays 210.00 on a base of 1 000.00.

### Mode `mixed` — "Always (upon invoice)"

**On the invoice.** The discount is anticipated. Applying 4.3 with a single product line and a
single grouping key:

```formula
epd_amount_currency = round( (−1) × 1000.00 × 0.02 ) = −20.00
```

(the sign is the direction sign `−1` of a customer invoice; the stored line amounts carry the
accounting sign). Two `epd` lines are produced:

- the base-shift line, on the income account, carrying the taxes of the product line, with an
  accounting amount of +20.00 (a debit of 20.00, reducing the credited revenue that is taxed);
- the counterpart line, on the same account, with no taxes, with an accounting amount of −20.00
  (a credit of 20.00).

The tax engine then computes the 21 % tax on a base of 1 000.00 − 20.00 = 980.00, giving 205.80.
The invoice becomes:

| Line | Display type | Account | Debit | Credit | Taxed base |
| --- | --- | --- | --- | --- | --- |
| product | product | Product Sales | | 1 000.00 | 1 000.00 at 21 % |
| early payment discount (2.0%) | epd | Product Sales | 20.00 | | −20.00 at 21 % |
| early payment discount (2.0%) | epd | Product Sales | | 20.00 | not taxed |
| tax 21 % | tax | Tax Received | | 205.80 | |
| instalment | payment_term | Customers | 1 205.80 | | |

Net total: 1 000.00 − 20.00 + 20.00 = 1 000.00. Tax total: 205.80. Gross total: 1 205.80.

The receivable line stores: discount deadline 2026-01-30, discount amount in currency
`round(1 205.80 − 1 000.00 × 0.02) = round(1 205.80 − 20.00) = 1 185.80`, discount balance 1 185.80.

**If the customer pays 1 185.80 on 2026-01-25.** The remaining 20.00 is written off against the net
account only (the tax was already reduced on the invoice):

| Line | Account | Debit | Credit |
| --- | --- | --- | --- |
| write-off | Cash Discount Granted | 20.00 | |

**If the customer pays the full 1 205.80 late**, the invoice is simply settled; the anticipated
reduction of tax stands, which is exactly the intent of this mode.

### Side-by-side summary

| | `included` | `excluded` | `mixed` |
| --- | --- | --- | --- |
| Invoice net total | 1 000.00 | 1 000.00 | 1 000.00 |
| Invoice tax total | 210.00 | 210.00 | 205.80 |
| Invoice gross total | 1 210.00 | 1 210.00 | 1 205.80 |
| Extra lines on the invoice | none | none | two `epd` lines of 20.00 |
| Stored discounted amount | 1 185.80 | 1 190.00 | 1 185.80 |
| Discount actually granted if paid early | 24.20 | 20.00 | 20.00 |
| Write-off split at payment | 20.00 net + 4.20 tax | 20.00 net | 20.00 net |

## 4.5 The write-off computed at payment time

When a payment settles a receivable line that is still eligible for the discount, the platform
prepares the counterpart journal items so that the full receivable is cleared. Eligibility requires
all of:

- the payment currency equals the document currency;
- the document type is one of customer invoice, sales receipt, vendor bill, purchase receipt;
- the payment term carries an early discount;
- either there is no reference date, or the document has no document date, or the reference date is
  on or before the deadline stored on the first payment term line;
- none of the payment term lines is already reconciled.

The counterpart preparation, per payment term line:

1. If the discount percentage is zero, produce nothing.
2. Collect the current tax amounts of the document, indexed by the **inverse** repartition line —
   that is, for each tax line, the repartition line at the same position in the *opposite* document
   kind's repartition list (invoice ↔ refund). This is what makes the write-off book on the refund
   side of each tax.
3. Build one base line per product line, marked as a refund, with every fixed-amount tax removed and
   the unit price multiplied by (100 − discount_percentage) ÷ 100. Add and round the tax details, and
   add the accounting data.
4. Choose the write-off account: the company's *cash discount write-off loss* account for an inbound
   document, the *cash discount write-off gain* account for an outbound document. Resolve an analytic
   distribution for it from the distribution models, keyed by the account prefix, the company, the
   commercial entity and the partner's tags.
5. Let

```formula
term_amount_currency = payment_term_line.amount_currency − payment_term_line.discount_amount_currency
term_balance         = payment_term_line.balance         − payment_term_line.discount_balance
```

6. For each base line, compute the **base delta** — the difference between the discounted
   tax-excluded amount and what the invoice actually booked:

```formula
amount_currency = round( direction_sign × discounted_total_excluded_currency − invoice_line.amount_currency , document_currency )
balance         = round( direction_sign × discounted_total_excluded          − invoice_line.balance          , company_currency )
```

   and accumulate it per grouping key. The grouping key is (partner, currency, the write-off account,
   the analytic distribution); in the `included` mode it additionally carries the taxes and the tax
   grids, so that the write-off itself is taxed and reported.

7. Let

```formula
percentage_paid = | payment_term_line.amount_residual_currency ÷ document.amount_total |
```

8. **Only in the `included` mode and only when the product lines carry taxes**, compute the tax
   deltas: prepare the tax lines from the discounted base lines and, for each, subtract the tax
   amount the invoice actually booked for the inverse repartition line. Each resulting delta becomes
   a write-off tax line labelled "Early Payment Discount (*the tax name*)", with

```formula
amount_currency = round( tax_delta_amount_currency × percentage_paid , document_currency )
balance         = round( tax_delta_balance         × percentage_paid , company_currency )
```

9. Each accumulated base delta becomes a write-off base line labelled "Early Payment Discount", with

```formula
amount_currency = round( base_delta_amount_currency × percentage_paid , document_currency )
balance         = round( base_delta_balance         × percentage_paid , company_currency )
```

10. **Fix the rounding.** Compute

```formula
delta_amount_currency = term_amount_currency − Σ base line amounts − Σ tax line amounts
delta_balance         = term_balance         − Σ base line balances − Σ tax line balances
```

    and add both deltas to the base line with the largest amount in currency. This guarantees that
    the write-off exactly clears the receivable line, to the cent, whatever rounding happened above.

---

# 5. Discount allocation lines

When the company configures a **separate account for the discount**, the discounted part of every
product line is moved out of the revenue account into that account, so that gross revenue and
discounts granted can be read separately.

The account is chosen as: for a sale document, the company's *separate account for income discount*…
— precisely, for a sale document the company's "separate account for expense discount"
(`account_discount_expense_allocation_id`), and for a purchase document the company's "separate
account for income discount" (`account_discount_income_allocation_id`). When the relevant one is not
set, no discount allocation lines are produced at all.

For each product line whose account differs from the discount allocation account, let

```formula
discounted_amount_currency = round( direction_sign × quantity × price_unit × discount ÷ 100 , line_currency )
discounted_balance         = round( discounted_amount_currency ÷ currency_rate , company_currency )
```

If the discounted amount rounds to zero, the line contributes nothing. Otherwise it contributes two
needed entries:

| Entry | Key (account, document, rate) | Amount in currency | Balance |
| --- | --- | --- | --- |
| reversal on the revenue account | the product line's own account | + discounted amount | + discounted balance |
| allocation | the discount allocation account | − discounted amount | − discounted balance |

Both are labelled "Discount" and have display type `discount`.

**Analytic distribution.** The analytic distribution of the produced lines is a weighted merge. For
every product line and every one of its two entries, and for every analytic account in that product
line's distribution with percentage *q*:

```formula
weighted_amount( key , analytic_account ) += entry_balance × q ÷ 100
```

accumulated over all product lines sharing the same key. Then, per key,

```formula
distribution( analytic_account ) = 100 × weighted_amount( key , analytic_account ) ÷ Σ weighted_amount( key , · )
```

with the denominator replaced by one when it would be zero. The result is a percentage map that sums
to one hundred (up to the storage precision) and reflects how much of the discount belongs to each
analytic account.

**Worked example.** Two product lines on a customer invoice, both with an income account "Product
Sales", the discount allocation account being "Discounts Granted":

| Line | Quantity | Unit price | Discount | Analytic distribution |
| --- | --- | --- | --- | --- |
| A | 10 | 100.00 | 10 % | Project Alpha 100 % |
| B | 5 | 200.00 | 20 % | Project Alpha 50 %, Project Beta 50 % |

Direction sign is `−1`; rate is one.

```formula
A: discounted = round( −1 × 10 × 100.00 × 10 ÷ 100 ) = −100.00
B: discounted = round( −1 ×  5 × 200.00 × 20 ÷ 100 ) = −200.00
```

Both lines share the key (Product Sales, this document, rate 1) for their reversal entries and the
key (Discounts Granted, this document, rate 1) for their allocation entries, so the aggregation of
1.5 adds them:

| Produced line | Account | Amount in currency | Balance | Analytic distribution |
| --- | --- | --- | --- | --- |
| Discount | Product Sales | −300.00 | −300.00 | Alpha 66.67 %, Beta 33.33 % |
| Discount | Discounts Granted | +300.00 | +300.00 | Alpha 66.67 %, Beta 33.33 % |

The analytic percentages come from weighted amounts of −100.00 (A, Alpha) + −100.00 (B, Alpha) =
−200.00 for Alpha and −100.00 for Beta, a total of −300.00, hence 200 ÷ 300 = 66.666…% and
100 ÷ 300 = 33.333…%.

A debit of 300.00 on "Product Sales" and a credit of 300.00 on "Discounts Granted": revenue is
reduced by the discount and the discount appears in its own account.

---

# 6. Cash rounding

## 6.1 The two primitives

**Round to the coin multiple.** Given an amount *a*, a rounding precision *r* (the smallest coin) and
a rounding method:

| Method | Result |
| --- | --- |
| `UP` (Up) | the smallest multiple of *r* that is greater than or equal to *a* in absolute value — that is, the magnitude is rounded away from zero to a multiple of *r*, keeping the sign |
| `DOWN` (Down) | the multiple of *r* nearest zero whose absolute value does not exceed \|*a*\| , keeping the sign |
| `HALF-UP` (Nearest) | the nearest multiple of *r*; a tie is resolved away from zero |

Formally, with *q* = *a* ÷ *r*:

```formula
UP       : result = r × ( sign(a) × ceiling( | q | ) )
DOWN     : result = r × ( sign(a) × floor(   | q | ) )
HALF-UP  : result = r × ( sign(a) × floor(   | q | + 0.5 ) )
```

**The cash rounding difference.** Given a currency *C* and an amount *a*:

```formula
a'         = round( a , C )
difference = round( round_to_coin( a' ) − a' , C )
```

Rounding *a* to the currency first matters: it removes sub-cent noise before the coin rounding.

**Worked values** at *r* = 0.05 on a two-decimal currency:

| Amount | Nearest | Up | Down |
| --- | --- | --- | --- |
| 23.91 | 23.90, difference −0.01 | 23.95, difference +0.04 | 23.90, difference −0.01 |
| 23.93 | 23.95, difference +0.02 | 23.95, difference +0.02 | 23.90, difference −0.03 |
| 23.925 → rounded to currency 23.93 | 23.95, difference +0.02 | 23.95, difference +0.02 | 23.90, difference −0.03 |
| 1 207.37 | 1 207.35, difference −0.02 | 1 207.40, difference +0.03 | 1 207.35, difference −0.02 |
| −23.91 | −23.90, difference +0.01 | −23.95, difference −0.04 | −23.90, difference +0.01 |

## 6.2 The recomputation algorithm

Run on every draft invoice at stack sequence 30.

1. Find the existing cash rounding line (the line whose display type is `rounding`).
2. **If the document has no cash rounding method**: delete the existing line, if any, and stop.
3. **If the strategy changed** since the line was produced — the old strategy is deduced as
   "modify the biggest tax amount" when the line carries an originator tax, and "add a rounding
   line" otherwise — delete the existing line and treat the family as empty.
4. Compute the amount to round: take every line whose account is neither receivable nor payable,
   excluding the rounding line itself, and sum their amounts in currency. Call it *T*. (Note that *T*
   is the accounting-signed total, that is, minus the customer-facing gross total on a customer
   invoice.)
5. Compute the difference:

```formula
diff_amount_currency = cash_rounding_difference( document_currency , T )
if document_currency = company_currency:
    diff_balance = diff_amount_currency
else:
    diff_balance = convert( diff_amount_currency , document_currency → company_currency ,
                            company , document_date or accounting_date )
```

6. **If the difference is zero** in the document currency (both the balance and the amount in
   currency are tested against the document currency's precision), delete the existing line and stop:
   the document is already rounded.
7. **If the existing line already carries exactly this difference** in both currencies (compared at
   the document currency's precision), stop: nothing to do.
8. Otherwise build the rounding line's values: the balance, the amount in currency, the commercial
   entity as partner, the document, the document currency, the company, the company currency, and
   display type `rounding`. Then, per strategy:

   - **Add a rounding line** (`add_invoice_line`):
     - label: the name of the cash rounding method;
     - account: when the balance is strictly positive and the method's *loss* account is set, the
       loss account; otherwise the *profit* account. Both are read in the document's company because
       they are company-dependent;
     - taxes: cleared.
   - **Modify the biggest tax amount** (`biggest_tax`):
     - find the tax line with the largest absolute balance; if there is none, stop (no tax, nothing
       to modify);
     - label: the biggest tax line's label followed by " (rounding)";
     - account: the biggest tax line's account;
     - tax repartition line: the biggest tax line's repartition line;
     - tax grids: the biggest tax line's grids;
     - taxes: the biggest tax line's taxes.

9. Write the values onto the existing line, or create the line if there is none.

## 6.3 Interaction with the tax engine

A rounding line produced by the *add a rounding line* strategy is itself a base line for the tax
engine (step 3 of 1.7), passed with quantity one, an already-tax-excluded amount and the special kind
"cash rounding". A rounding line produced by the *biggest tax* strategy is a tax line (it has a tax
repartition), so it is *not* a base line and it is counted as tax in the document totals.

This is what makes the two strategies differ in the totals block:

- *Add a rounding line*: the net total changes by the difference, the tax total does not.
- *Modify the biggest tax amount*: the tax total changes by the difference, the net total does not.

## 6.4 Worked example: cash rounding to five hundredths, both strategies

**Setup.** A customer invoice with three product lines:

| Line | Quantity | Unit price | Tax | Net |
| --- | --- | --- | --- | --- |
| A | 3 | 12.34 | 21 % | 37.02 |
| B | 1 | 45.67 | 21 % | 45.67 |
| C | 2 | 9.99 | 6 % | 19.98 |

Net total: 37.02 + 45.67 + 19.98 = 102.67.
Tax at 21 % on a base of 37.02 + 45.67 = 82.69: round(82.69 × 0.21) = round(17.3649) = 17.36.
Tax at 6 % on a base of 19.98: round(19.98 × 0.06) = round(1.1988) = 1.20.
Tax total: 18.56. Gross total: 102.67 + 18.56 = 121.23.

The cash rounding method rounds to 0.05 with the nearest method.

```formula
nearest multiple of 0.05 to 121.23 = 121.25
difference (customer facing)       = 121.25 − 121.23 = +0.02
```

In accounting signs, the sum of the non-receivable lines is *T* = −121.23, and

```formula
round_to_coin( −121.23 ) = −121.25
diff_amount_currency     = −121.25 − (−121.23) = −0.02
```

so the rounding line carries a balance of −0.02, that is, a **credit of 0.02**.

### Strategy "Add a rounding line"

The difference is negative, so the *profit* account is used (the loss account is used only when the
balance is strictly positive). Say the profit account is "Cash Rounding Gain".

| Line | Display type | Account | Debit | Credit |
| --- | --- | --- | --- | --- |
| A | product | Product Sales | | 37.02 |
| B | product | Product Sales | | 45.67 |
| C | product | Product Sales | | 19.98 |
| Rounding (the method's name) | rounding | Cash Rounding Gain | | 0.02 |
| Tax 21 % | tax | Tax Received | | 17.36 |
| Tax 6 % | tax | Tax Received | | 1.20 |
| Instalment | payment_term | Customers | 121.25 | |

Totals shown to the customer: net 102.69, tax 18.56, gross 121.25. The net moved by the two cents
because the rounding line is a base line.

### Strategy "Modify the biggest tax amount"

The biggest tax line by absolute balance is the 21 % line at 17.36. The rounding line therefore
copies that line's account, repartition line, grids and taxes, and is labelled "Tax 21% (rounding)".

| Line | Display type | Account | Debit | Credit |
| --- | --- | --- | --- | --- |
| A | product | Product Sales | | 37.02 |
| B | product | Product Sales | | 45.67 |
| C | product | Product Sales | | 19.98 |
| Tax 21 % | tax | Tax Received | | 17.36 |
| Tax 21% (rounding) | rounding, with a tax repartition | Tax Received | | 0.02 |
| Tax 6 % | tax | Tax Received | | 1.20 |
| Instalment | payment_term | Customers | 121.25 | |

Totals shown to the customer: net 102.67, tax 18.58, gross 121.25. The tax moved by the two cents.

### The same example rounded down

With the rounding method "Down", the nearest multiple of 0.05 not exceeding 121.23 in magnitude is
121.20, so the difference is −0.03 customer-facing, that is, an accounting balance of +0.03: a
**debit** of 0.03. Under the "add a rounding line" strategy the balance is now strictly positive, so
the **loss** account is used (say "Cash Rounding Loss"), and the totals become net 102.64, tax 18.56,
gross 121.20.

With the rounding method "Up", the multiple is 121.25 — the same as nearest in this case — and the
result is identical to the first table.

---

# 7. Document totals

One routine computes every amount field of a document. It walks the lines once.

## 7.1 The walk

Initialise six accumulators to zero: untaxed and untaxed-in-currency, tax and tax-in-currency,
residual and residual-in-currency, plus a running total and total-in-currency.

For each line of the document:

**If the document is an invoice (receipts included):**

| Line's display type | Contribution |
| --- | --- |
| `tax`, `non_deductible_tax`, or `rounding` **with** a tax repartition | add the balance to the tax accumulator and to the total; add the amount in currency to the tax-in-currency accumulator and to the total-in-currency |
| `product`, `rounding` (without a tax repartition), `non_deductible_product`, `non_deductible_product_total` | add the balance to the untaxed accumulator and to the total; add the amount in currency to the untaxed-in-currency accumulator and to the total-in-currency |
| `payment_term` | add the **residual** to the residual accumulator and the **residual in currency** to the residual-in-currency accumulator |
| anything else (`discount`, `epd`, `cogs`, sections, subsections, notes) | ignored |

Note that the `discount` and `epd` lines are deliberately ignored: they always come in pairs that
cancel, so including them would be a no-op, and excluding them keeps the accumulators cheap.

**If the document is a plain journal entry:** only lines with a non-zero debit contribute, adding
their balance and amount in currency to the running total.

## 7.2 The assignments

With *s* the direction sign:

```formula
amount_untaxed                     = s × untaxed_in_currency
amount_tax                         = s × tax_in_currency
amount_total                       = s × total_in_currency
amount_residual                    = −s × residual_in_currency
amount_untaxed_signed              = −untaxed
amount_untaxed_in_currency_signed  = −untaxed_in_currency
amount_tax_signed                  = −tax
amount_total_signed                = | total |            if the document type is a plain entry
                                   = −total               otherwise
amount_residual_signed             = residual
amount_total_in_currency_signed    = | amount_total |     if the document type is a plain entry
                                   = −( s × amount_total ) otherwise
```

**Reading the signs.** On a customer invoice *s* = −1. A product line is a credit, so its balance is
negative; the untaxed accumulator is negative; `amount_untaxed = −1 × negative = positive`, which is
the customer-facing net total. The signed variant `amount_untaxed_signed = −untaxed` is also
positive — for a customer invoice the signed and unsigned net totals coincide. On a customer credit
note *s* = +1, the product lines are debits, the accumulators are positive, `amount_untaxed` is
positive (the credit note shows a positive amount to the customer) but `amount_untaxed_signed` is
negative, which is what a revenue report must add.

**Worked check on the three-line example of 6.4** (before cash rounding):

- untaxed_in_currency = −37.02 − 45.67 − 19.98 = −102.67; `amount_untaxed` = −1 × −102.67 = 102.67 ✓
- tax_in_currency = −17.36 − 1.20 = −18.56; `amount_tax` = 18.56 ✓
- total_in_currency = −121.23; `amount_total` = 121.23 ✓
- residual_in_currency of the single receivable line, unpaid, = +121.23; `amount_residual` =
  −(−1) × 121.23 = 121.23 ✓
- `amount_total_signed` = −(−121.23) = 121.23; a customer invoice contributes a positive figure to
  turnover ✓

## 7.3 Worked example: a three-line invoice with two taxes, end to end

This is the full trace an implementation must be able to reproduce.

**Setup.** Customer invoice, document date 2026-03-15, currency equal to the company currency,
payment term "Immediate", no cash rounding, no early discount, no discount allocation account.

| Line | Product | Quantity | Unit price | Line discount | Taxes |
| --- | --- | --- | --- | --- | --- |
| 1 | Consulting hour | 12 | 85.00 | 0 % | Sales 21 % |
| 2 | Printed manual | 40 | 14.50 | 10 % | Sales 21 % |
| 3 | Digital subscription | 1 | 299.00 | 0 % | Sales 6 % |

**Step 1 — per-line net amounts.**

```formula
line_net = round( quantity × unit_price × ( 1 − discount ÷ 100 ) , currency )
```

| Line | Computation | Net |
| --- | --- | --- |
| 1 | 12 × 85.00 × 1 = 1 020.00 | 1 020.00 |
| 2 | 40 × 14.50 × 0.90 = 522.00 | 522.00 |
| 3 | 1 × 299.00 × 1 = 299.00 | 299.00 |

Net total: 1 841.00.

**Step 2 — per-tax bases.** Both lines 1 and 2 carry the same 21 % tax, so they share a tax grouping
key (same partner, same currency, same analytic distribution — none —, same account "Product Sales",
same tax set). Line 3 carries the 6 % tax and therefore a different key.

| Tax | Base | Raw tax | Rounded tax |
| --- | --- | --- | --- |
| Sales 21 % | 1 020.00 + 522.00 = 1 542.00 | 1 542.00 × 0.21 = 323.82 | 323.82 |
| Sales 6 % | 299.00 | 299.00 × 0.06 = 17.94 | 17.94 |

Tax total: 341.76. Gross total: 1 841.00 + 341.76 = 2 182.76.

**Step 3 — the rounding delta.** The engine rounds per tax first: round(323.82) = 323.82. It then
rounds each line's share: line 1 contributes 1 020.00 × 0.21 = 214.20 and line 2 contributes
522.00 × 0.21 = 109.62; 214.20 + 109.62 = 323.82 exactly, so the delta is zero and no cent is shifted.
The same holds for the 6 % tax. (Section 7.4 gives an example where the delta is not zero.)

**Step 4 — the payment term.** "Immediate" is a single percent line of 100 % at zero days, so it is
also the last line, and the balance rule gives it the whole total: 2 182.76, due 2026-03-15.

**Step 5 — the journal items.**

| # | Display type | Account | Label | Debit | Credit | Tax grids |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | product | Product Sales | Consulting hour | | 1 020.00 | base 21 % |
| 2 | product | Product Sales | Printed manual | | 522.00 | base 21 % |
| 3 | product | Product Sales | Digital subscription | | 299.00 | base 6 % |
| 4 | tax | Tax Received | Sales 21 % | | 323.82 | tax 21 % |
| 5 | tax | Tax Received | Sales 6 % | | 17.94 | tax 6 % |
| 6 | payment_term | Customers | (the payment reference) | 2 182.76 | | |

Debits 2 182.76, credits 1 020.00 + 522.00 + 299.00 + 323.82 + 17.94 = 2 182.76. Balanced.

**Step 6 — the document totals.**

| Field | Value |
| --- | --- |
| Untaxed Amount | 1 841.00 |
| Tax | 341.76 |
| Total | 2 182.76 |
| Amount Due (before any payment) | 2 182.76 |
| Untaxed Amount Signed | 1 841.00 |
| Tax Signed | 341.76 |
| Total Signed | 2 182.76 |
| Amount Due Signed | 2 182.76 |

(The residual accumulator holds the receivable line's residual, +2 182.76 as a debit; the signed
residual is that accumulator directly, and the customer-facing residual is its negation times the
direction sign, which for a customer invoice gives the same positive figure.)

**Step 7 — the totals block shown on the document.**

| Row | Amount |
| --- | --- |
| Untaxed Amount | 1 841.00 |
| Sales 21 % on 1 542.00 | 323.82 |
| Sales 6 % on 299.00 | 17.94 |
| **Total** | **2 182.76** |

## 7.4 A rounding-delta example

Two lines, each with a unit price of 21.53 and a price-included 21 % tax, currency with two decimals,
global rounding:

```formula
raw net per line  = 21.53 ÷ 1.21 = 17.79338843…
rounded per line  = 17.79
raw net total     = 35.58677686…
rounded net total = 35.59
delta             = 35.59 − 17.79 − 17.79 = +0.01
```

The delta is added to the **first** line, so the stored balances are 17.80 and 17.79 while both
displayed subtotals remain 17.79.

```formula
raw tax per line  = 21.53 ÷ 1.21 × 0.21 = 3.73661157…
rounded per line  = 3.74
raw tax total     = 7.473223141…
rounded tax total = 7.47
delta             = 7.47 − 3.74 − 3.74 = −0.01
```

so one line's tax detail becomes 3.73 and the other stays 3.74, and the single tax line carries 7.47.

Final figures: untaxed 17.80 + 17.79 = 35.59, tax 7.47, total 21.53 + 21.53 = 43.06. The check
35.59 × 0.21 = 7.4739 ≈ 7.47 holds.

---

# 8. Numbering

## 8.1 When a number is assigned

The number is computed whenever the "posted before" flag, the status, the journal, the accounting
date, the document type or the originating payment changes. The records are processed sorted by
accounting date, then reference, then internal identifier — so that a batch of documents created in
one operation receives consecutive numbers in date order.

For each document:

1. If the status is cancelled, leave the number alone.
2. If the document has never been posted **and** its current number does not match its accounting
   date's period, clear the number and stop. (This is what makes a draft's number follow its date.)
3. If the document has an accounting date, has no number (or has the placeholder `/`), and its status
   is **not** draft, take the next number of the sequence.

A draft therefore normally has no number; it shows the *placeholder* instead, which is the number it
would take.

## 8.2 The sequence scope

The sequence is not a database counter: it is derived from the highest existing number that matches a
scope. The scope for a customer document is:

- the same journal;
- a number other than `/`;
- when the journal has a **dedicated credit note sequence**: the same side — credit notes are
  numbered among credit notes, everything else among everything else;
- when the journal has a **dedicated payment sequence**: payments among payments, non-payments among
  non-payments;
- when the journal has a **dedicated debit note sequence**: debit notes among debit notes,
  non-debit-notes among non-debit-notes;
- when the journal is a self-billing journal: the same commercial entity (and when there is no
  partner the scope is empty, forcing a fresh start);
- an accounting date inside the period implied by the detected reset rule.

The **reset rule** is deduced from a reference number: the last number, at or before this document's
date, inside the scope; failing that, the first number in the scope. The grammar of a number and the
five reset rules (never, yearly, monthly, by fiscal year range, by fiscal year range and month) are
specified in [`../general-ledger/calculations.md`](../general-ledger/calculations.md). When the
journal defines its own number pattern, that pattern overrides the built-in grammars.

To stop a looser grammar from matching a stricter number, an **anti-pattern** is applied: when the
reset rule is yearly or by fiscal year range, numbers whose prefix matches the monthly grammar are
excluded; when the reset rule is "never", numbers whose prefix matches the yearly grammar are
excluded (which also excludes monthly and the two range grammars).

## 8.3 The starting number

When no number exists in the scope, a starting number is built:

1. Let *D* be the accounting date, else the document date, else today.
2. The year part is the four-digit year of *D*, unless the company's fiscal year does not end on
   31 December, in which case it is a two-digit range: when *D* is after the fiscal year end of its
   own calendar year, the range is (two-digit year of *D*)-(two-digit year of *D* + 1); otherwise it
   is (two-digit year of *D* − 1)-(two-digit year of *D*). The fiscal year end day is clamped to the
   number of days in the fiscal year end month.
3. For a sale, bank, cash or credit journal the starting number is

   `journal code` `/` `year part` `/` `00000`

   with the counter shortened to `0000` when the fiscal year is staggered (to keep the number short).
4. For any other journal it is

   `journal code` `/` `year part` `/` `two-digit month` `/` `0000`

   and, on a self-billing journal, the commercial entity's identifier padded to five digits is
   appended to the journal code.
5. If the journal has a dedicated credit note sequence and the document is a credit note, the letter
   `R` is prefixed.
6. If the journal has a dedicated payment sequence and the document is a payment, the letter `P` is
   prefixed.
7. If the journal has a dedicated debit note sequence and the document is a debit note of a customer
   invoice or a vendor bill, the letter `D` is prefixed.

**Examples.** A sale journal with code `INV`, a calendar fiscal year, a document dated 2026-03-15:
the starting number is `INV/2026/00000`, so the first document takes `INV/2026/00001`. Its credit
notes, when the journal has a dedicated credit note sequence, start at `RINV/2026/00001`. Its debit
notes, when the journal has a dedicated debit note sequence, start at `DINV/2026/00001`. With a
fiscal year ending on 30 June and a document dated 2026-03-15 (before the end), the year part is
`25-26` and the starting number is `INV/25-26/0000`.

## 8.4 The accounting date correction

When a document is dated in a closed period, the accounting date is pushed forward so the number
stays increasing. Let *L* be the last violated lock date, *today* be today, *H* be the highest number
in the scope (or, absent that, the last number found with the relaxed scope), and let *reset* be the
reset rule deduced from *H*.

1. If any lock date is violated, the candidate date becomes *L* + 1 day.
2. For a **sale document**, and only when a lock date was violated:
   - if there is no highest number, or the reset rule is monthly: the result is the earlier of *today*
     and the last day of the candidate date's month;
   - if the reset rule is yearly: the earlier of *today* and the last day of the candidate date's
     year.
3. For a **non-sale document** (always, not only on a lock violation):
   - if there is no highest number, or the reset rule is monthly or fiscal-year-range monthly: when
     today's (year, month) is after the candidate's (year, month), the result is the last day of the
     candidate's month; otherwise the later of the candidate and today;
   - if the reset rule is yearly: when today's year is after the candidate's year, the result is
     31 December of the candidate's year; otherwise the later of the candidate and today.
4. Otherwise the candidate date stands.

## 8.5 Gap detection and the number warning

Two mechanisms guard the sequence:

- A **gap flag** is stored on the document that is the first one breaking the natural order. The
  journal dashboard and the numbering report read it.
- A **warning** is shown in the form while the user types a number by hand: when the typed number
  does not immediately follow the highest number in the scope, or repeats it, a non-blocking message
  appears (the exact texts are listed in [`business-rules.md`](business-rules.md)).

A unique index over (number, journal) restricted to posted documents with a number other than `/`
makes a duplicate impossible; the violation reads *"Another entry with the same name already
exists."*

---

# 9. The payment reference

A customer invoice may carry a *payment reference*: the string the customer is asked to quote when
paying, so that the incoming bank line can be matched automatically. It is computed, on posting, for
a customer invoice that has none. Which generator is used is decided by two fields on the journal:
the reference **model** (field `invoice_reference_model`, labelled Communication Standard) and the
reference **type** (field `invoice_reference_type`, labelled Communication Type). Their stored
selection values are reproduced below because the pairing decides the generator; the labels are the
ones shown in the interface.

| Stored model value | Label | Stored type value | Label |
| --- | --- | --- | --- |
| `odoo` | Full Reference | `invoice` | Based on Invoice |
| `euro` | European | `partner` | Based on Customer |
| `number` | Numbers only | | |

The six combinations:

## 9.1 Full Reference, Based on Invoice

The document number itself.

Example: `INV/2026/00042`.

## 9.2 Full Reference, Based on Customer

```formula
reference = "CUST" + "/" + ( partner reference , or the partner identifier when there is none )
```

Example: a customer whose internal reference is `dumb customer 97` gives `CUST/dumb customer 97`.

## 9.3 Numbers only, Based on Invoice

Take the result of 9.1 and keep only its digits.

Example: `INV/2026/00042` gives `202600042`.

## 9.4 Numbers only, Based on Customer

Take the result of 9.2 and keep only its digits.

Example: `CUST/customer 97` gives `97`.

## 9.5 European, Based on Invoice — the structured creditor reference

The reference follows the international creditor reference standard.

1. Let *J* be the journal code when that code is pure ASCII and alphanumeric; otherwise the journal's
   internal identifier.
2. Let *N* be the document's internal identifier, left-padded with zeros to six digits.
3. Let the **data** be the concatenation *J* followed by *N*.
4. Compute the two check digits as follows:
   1. Append the literal `RF` to the data.
   2. Replace every letter by two digits: `A` → 10, `B` → 11, …, `Z` → 35. Digits stay as they are.
   3. Read the whole thing as one (very large) decimal integer *V*.
   4. The check number is 98 − ( *V* × 100 mod 97 ). Equivalently, with the standard
      international-bank-account-number check arithmetic: the check digits are the two-digit value
      *c* such that ( *V* × 100 + *c* ) mod 97 = 1.
   5. Format *c* with two digits, padding with a leading zero when needed.
5. The reference is `RF`, then the two check digits, then a space, then the data split into groups of
   four characters separated by spaces.

**Worked example.** Journal code `INV`, invoice identifier 37.

- data = `INV` + `000037` = `INV000037`
- append `RF`: `INV000037RF`
- letters to digits: `I`→18, `N`→23, `V`→31, `R`→27, `F`→15, giving
  `18` `23` `31` `000037` `27` `15` = 182331000037 2715, that is
  *V* = 1823310000372715
- *V* × 100 = 182331000037271500; 182331000037271500 mod 97 = 31; 98 − 31 = 67
- check digits = `67`
- groups of four over `INV000037`: `INV0`, `0003`, `7`
- reference = `RF67 INV0 0003 7`

## 9.6 European, Based on Customer

1. *J* as in 9.5.
2. Take the partner's internal reference, keep only its digits, and take the last 21 characters; when
   the partner has no reference, take the last 21 characters of the partner's identifier instead.
3. Prepend *J* and take the last 21 characters of the result.
4. Apply the same check-digit computation and formatting as 9.5.

**Worked example.** A partner whose internal reference is `food buyer 654`, journal code `INV`.
Digits only: `654`. Prepend the journal code: `INV654`. Check digits over `INV654RF`: letters
`I`→18, `N`→23, `V`→31, then `654`, then `R`→27, `F`→15, giving

*V* = 1 823 316 542 715

*V* × 100 = 182 331 654 271 500, whose remainder modulo 97 is 82, so the check number is
98 − 82 = 16. Split into groups of four: `INV6`, `54`. The reference is `RF16 INV6 54`.

## 9.7 Unknown combination

When the journal's reference model and type do not name a known generator, the operation fails with:

> The combination of reference model and reference type on the journal is not implemented

## 9.8 Where the reference is used

- It becomes the label of every payment term line (with the instalment marker appended when there is
  more than one instalment).
- It is printed on the document as the communication to quote.
- Its sanitised form — every character that is not a Latin letter or a digit removed — is indexed, so
  that a bank statement line's free text can be matched against it.
- It is embedded in the quick response code payload as the structured communication, when the code
  generator supports one and the reference is a valid structured reference.

---

# 10. Instalments and the portal payment amounts

When the customer opens the document in the portal, or when a payment is registered, the platform
computes which instalment is next and how much to collect.

## 10.1 Building the instalment list

Given the payment term lines of a document, a payment currency, a payment date (default: today) and
an optional "next payment date" boundary:

1. Sort the lines by maturity date (a missing maturity date sorts last), then by accounting date.
2. Walk them, numbering from one. For each line record: the number, the line, the maturity date (or
   the accounting date when there is none), the residual and the residual in currency, their
   *unsigned* counterparts (the residual multiplied by minus the direction sign), a kind, and whether
   the line is reconciled.
3. A reconciled line keeps the kind `other` and is skipped for the rest of the walk.
4. If the document is still eligible for the early payment discount at the given payment currency and
   payment date (the eligibility test of 4.5), the line's kind becomes `early_payment_discount` and
   its amounts are replaced by the stored discounted amounts; the difference between the full and the
   discounted amount is recorded as the discount. The walk then moves on.
5. Otherwise the line's kind is decided:

   | Condition | Kind |
   | --- | --- |
   | a "next payment date" boundary was given and the maturity date is on or before it | `before_date` |
   | the maturity date is strictly before the payment date | `overdue`, and this also becomes the first mode |
   | no first mode has been set yet | `next`, and this becomes the first mode |
   | the current mode is `overdue` | `next` (the first instalment after the overdue block) |

## 10.2 Deriving the next payment values

Let *show instalments* mean that there is more than one instalment.

| Situation | Instalment state | Amount due | Next amount to pay | Next reference | Next due date |
| --- | --- | --- | --- | --- | --- |
| instalments shown and some are overdue | `overdue` | the document residual | the sum of the unsigned residuals of all overdue instalments | the document number, a hyphen, the first overdue instalment's number | the first overdue instalment's maturity date |
| instalments shown and some remain unreconciled | `next` | the document residual | the first unreconciled instalment's unsigned residual | the document number, a hyphen, that instalment's number | that instalment's maturity date |
| an early payment discount instalment exists | `epd` | that instalment's unsigned residual (the discounted amount) | the document residual (the full amount) | the document number | that instalment's maturity date |
| otherwise | none | the document residual | the document residual | the document number | the document due date |

In the early-discount case three extra values are produced:

```formula
days_left = max( 0 , discount_date − today )
```

and the message

> Discount of *the discount amount formatted in the document currency* if paid within *days_left* days

or, when no day remains,

> Discount of *the discount amount formatted in the document currency* if paid today

When the caller supplies a **custom amount**, and that amount is neither the next amount to pay nor
(in the early-discount case) the discounted amount due, the state is forced to `next`, the next
amount becomes the custom amount, the reference becomes the plain document number and the next due
date becomes the first instalment's maturity date.

The complete result also carries: the payment status, the amount already paid (the total minus the
residual), the document due date, the list of unreconciled instalments, and whether only one
instalment remains.

## 10.3 Worked example: a partial payment

**Setup.** The three-line invoice of 7.3, gross total 2 182.76, single instalment due 2026-03-15,
payment reference `INV/2026/00042`. On 2026-03-20 the customer pays 1 000.00 by bank transfer; the
payment is registered and reconciled against the invoice's receivable line.

**Before the payment.**

| Quantity | Value |
| --- | --- |
| Receivable line balance | +2 182.76 |
| Receivable line residual | +2 182.76 |
| `amount_residual` | 2 182.76 |
| `payment_state` | `not_paid` |

**The reconciliation.** A partial reconciliation of 1 000.00 is created between the payment's
receivable line (a credit of 1 000.00) and the invoice's receivable line (a debit of 2 182.76).

**After the payment.**

```formula
receivable_residual          = round( 2182.76 − 1000.00 , company_currency ) = 1182.76
receivable_residual_currency = 1182.76
amount_residual              = −(−1) × 1182.76 = 1182.76
```

| Quantity | Value |
| --- | --- |
| Receivable line residual | +1 182.76 |
| Line reconciled flag | false (residual not zero) |
| `amount_residual` | 1 182.76 |
| `payment_state` | `partial` — the residual is not zero and a reconciliation row exists |
| Amount paid (portal) | 2 182.76 − 1 182.76 = 1 000.00 |
| Next amount to pay | 1 182.76 |
| Next reference | `INV/2026/00042` (one instalment only, so no hyphenated number) |

**Journal items.** The payment produces its own entry (specified in
[`../payments-and-bank-reconciliation/accounting-effects.md`](../payments-and-bank-reconciliation/accounting-effects.md)):

| Line | Account | Debit | Credit |
| --- | --- | --- | --- |
| liquidity | Outstanding Receipts | 1 000.00 | |
| counterpart | Customers | | 1 000.00 |

No journal item is added to the invoice. The invoice's payment status changes only because its
receivable line's residual changed.

**If the remaining 1 182.76 is then paid**, the receivable line's residual reaches zero, its
reconciled flag becomes true, a full reconciliation record is created, and the payment status becomes
`paid` — or `in_payment` when the company distinguishes the two and the payments are not yet matched
with a bank statement.

## 10.4 Worked example: a credit note against a paid invoice

**Setup.** The invoice of 7.3 (gross 2 182.76) has been fully paid on 2026-03-20; its payment status
is `paid` and its receivable line is fully reconciled against the payment. On 2026-04-02 the customer
returns the 40 printed manuals and a full credit note is issued for line 2 only.

**Step 1 — produce the credit note.** The reversal wizard is opened on the invoice with reversal date
2026-04-02 and the reason "Manuals returned". Because the source is an invoice and the plain
**Reverse** button is used, the credit note is created as a *draft* that is **not** automatically
reconciled (automatic cancellation applies only to plain journal entries and to the
"reverse and create invoice" mode). The credit note copies every line; the user then deletes lines 1
and 3, leaving line 2.

The credit note's header:

| Field | Value |
| --- | --- |
| Type | Customer Credit Note |
| Reversal of | the invoice |
| Reference | `Reversal of: INV/2026/00042, Manuals returned` |
| Accounting date, document date, due date | 2026-04-02 |
| Journal | the invoice's journal |
| Payment term | cleared, unless the invoice's term used the `mixed` early-discount mode, in which case it is kept |
| Salesperson, origin | copied from the invoice |
| Auto-post | `at_date` when the reversal date is in the future, otherwise `no` |
| Recipient bank account | cleared and recomputed |

**Step 2 — the amounts.** One product line of 40 × 14.50 with a 10 % discount and the 21 % tax:

```formula
net = round( 40 × 14.50 × 0.90 ) = 522.00
tax = round( 522.00 × 0.21 )     = 109.62
gross = 631.62
```

**Step 3 — the journal items.** The direction sign of a credit note is `+1`, so the product line is a
**debit** and the receivable line a **credit**:

| # | Display type | Account | Debit | Credit |
| --- | --- | --- | --- | --- |
| 1 | product | Product Sales | 522.00 | |
| 2 | tax | Tax Received | 109.62 | |
| 3 | payment_term | Customers | | 631.62 |

**Step 4 — post the credit note.** It receives a number from the credit-note sequence when the
journal has a dedicated one (for example `RINV/2026/00007`), otherwise from the ordinary sequence.

**Step 5 — settle it.** The invoice is already paid, so the credit note's 631.62 credit on the
receivable account has nothing to offset on that invoice: its residual stays 631.62 and its payment
status is `not_paid`. Two courses of action exist:

- **Refund the customer**: register an outbound payment of 631.62 against the credit note. The
  payment produces a debit of 631.62 on the customer account and a credit of 631.62 on the
  outstanding payments account; reconciling it with the credit note's receivable line brings the
  credit note's residual to zero, and its payment status to `paid`.
- **Keep the credit as an open balance**: leave the credit note unreconciled. It then appears in the
  "outstanding credits" block of every future invoice of that customer, ready to be attached in one
  click, and it lowers the customer's receivable aggregate by 631.62.

**Step 6 — the effect on the invoice.** None. The invoice remains `paid`. If, instead, the invoice
had been *unpaid* and the credit note had been reconciled against it, the invoice's residual would
have dropped by 631.62 and its payment status would have become `partial`; had the credit note
covered the whole invoice, the status would have become `reversed` (residual zero, no payment
counterpart, and the only counterpart type being a customer credit note) rather than `paid`.

---

# 11. Foreign-currency documents

## 11.1 The rate

```formula
expected_rate = conversion_rate( company_currency → document_currency , company , rate_date )
rate_date     = invoice_date , or today when there is none
```

The rate is stored on the document (it is a field, not a lookup) so that a later change of the rate
table does not silently restate a posted document. A user may override it; the override survives
until the currency, the company, the document date or the taxable supply date changes.

Constraint: the rate must be strictly positive. A non-positive rate is refused; see
[`business-rules.md`](business-rules.md).

## 11.2 Where the rate is applied

| Quantity | Rule |
| --- | --- |
| Every line's rate | equals the document's rate on an invoice; on a non-invoice it is looked up per line at the document date |
| A line's balance from its amount in currency | `round( amount_currency ÷ rate , company_currency )` |
| A line's amount in currency from its balance | `round( balance × rate , line_currency )` |
| The payment term distribution | uses the *effective* rate of 3.2, `|total_in_currency ÷ total_company|`, not the document rate, so that the instalments add up in both currencies |
| The cash rounding line's balance | converted from the document currency at the document date (or the accounting date) using the currency rate table, **not** the document's stored rate |
| A tax line under "reapply the currency rate" | `balance = round( amount_currency ÷ new_rate , company_currency )`, the amount in currency being preserved |

## 11.3 Worked example: a foreign-currency invoice

**Setup.** A company whose currency is the euro issues a customer invoice in United States dollars on
2026-05-04. The rate table gives 1 euro = 1.0850 dollars on that date, so the document's rate is
1.0850. The invoice has one product line: quantity 4, unit price 250.00 dollars, no discount, one tax
of 21 %.

**Step 1 — amounts in the document currency.**

```formula
net_currency   = round( 4 × 250.00 , USD ) = 1000.00
tax_currency   = round( 1000.00 × 0.21 , USD ) = 210.00
gross_currency = 1210.00
```

**Step 2 — amounts in the company currency.**

```formula
net_balance   = round( 1000.00 ÷ 1.0850 , EUR ) = round( 921.658986… ) = 921.66
tax_balance   = round(  210.00 ÷ 1.0850 , EUR ) = round( 193.548387… ) = 193.55
gross_balance = 921.66 + 193.55 = 1115.21
```

**Step 3 — the payment term.** A single 100 % line at 30 days. It is the last line, so the balance
rule gives it the whole residual in both currencies: 1 210.00 dollars and 1 115.21 euros. Note that
1 210.00 ÷ 1.0850 = 1 115.2074… would round to 1 115.21 as well, but the balance rule guarantees the
match even where the two would differ.

**Step 4 — the journal items.** All in accounting signs; the direction sign is `−1`.

| # | Display type | Account | Debit (EUR) | Credit (EUR) | Amount in currency (USD) |
| --- | --- | --- | --- | --- | --- |
| 1 | product | Product Sales | | 921.66 | −1 000.00 |
| 2 | tax | Tax Received | | 193.55 | −210.00 |
| 3 | payment_term | Customers | 1 115.21 | | +1 210.00 |

Both the euro column and the dollar column balance.

**Step 5 — the totals.**

| Field | Currency | Value |
| --- | --- | --- |
| Untaxed Amount | dollars | 1 000.00 |
| Tax | dollars | 210.00 |
| Total | dollars | 1 210.00 |
| Untaxed Amount Signed | euros | 921.66 |
| Tax Signed | euros | 193.55 |
| Total Signed | euros | 1 115.21 |
| Total in Currency Signed | dollars | 1 210.00 |

**Step 6 — a payment at a different rate.** On 2026-06-10 the customer pays 1 210.00 dollars into a
dollar bank account; the rate that day is 1 euro = 1.1000 dollars, so the payment's euro value is
round(1 210.00 ÷ 1.1000) = 1 100.00.

Reconciling the payment with the invoice clears the dollar residual exactly (1 210.00 against
1 210.00) but leaves a euro difference:

```formula
exchange_difference = 1115.21 − 1100.00 = 15.21
```

The platform creates an **exchange difference entry** in the exchange journal, dated at the
reconciliation date, carrying:

| Line | Account | Debit | Credit |
| --- | --- | --- | --- |
| exchange loss | Foreign Exchange Loss | 15.21 | |
| counterpart | Customers | | 15.21 |

and reconciles the counterpart line with the invoice's receivable line so that both residuals reach
zero. The invoice's payment status becomes `paid`. The mechanics, the account selection and the
reversal on unreconciliation belong to
[`../multi-currency/accounting-effects.md`](../multi-currency/accounting-effects.md).

**Step 7 — what happens if the rate is edited after the lines exist.** Changing the document's rate
sets the tax synchronisation's rounding source to "reapply the currency rate": the dollar amounts of
the tax lines are preserved and only their euro balances are recomputed as
`round( amount_currency ÷ new_rate , EUR )`. The product line amounts are recomputed from the unit
price as usual, and the payment term lines are then rebuilt from the new totals.

---

# 12. Miscellaneous computed quantities

## 12.1 The total in words

```formula
amount_total_words = the gross total spelled out in the document currency, with every comma removed
```

Example: a total of 2 182.76 in euros gives `Two Thousand One Hundred Eighty-Two Euros and
Seventy-Six Cents` (the exact wording is the currency's own spelling rule).

## 12.2 The payment term preview

Given the example amount and the example date, the preview runs the distribution of section 3 with
zero tax, the example amount as the untaxed amount in both currencies and a sign of `+1`, then groups
the produced instalments by date and renders one row per date:

> **N#** Installment of *amount* due on *date*

When an early discount exists, an extra row is rendered above:

> Early Payment Discount: *amount* if paid before *date*

where the amount is computed by formula A of 4.2 with a tax total of zero, so both branches reduce
to `round( example_amount × (1 − p) )` whatever the computation mode. The date is the example date
plus the discount days, formatted in the reader's language.

## 12.3 The abnormal-amount and abnormal-date warnings

For a customer with enough history, the platform compares the document's total and date with the
customer's usual pattern and, when the deviation is large, shows a warning banner. Both warnings can
be silenced per customer with the two "ignore abnormal" flags. The exact texts are listed in
[`business-rules.md`](business-rules.md).

## 12.4 The days-sales-outstanding of a customer

Specified with its worked example in [`entities.md`](entities.md), section 6.2.
