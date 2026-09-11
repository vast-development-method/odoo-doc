# Calculations

Every formula, every algorithm and every worked example of the payments and bank reconciliation domain.

---

## 0. Notation and conventions

### 0.1 Rounding

Every monetary value is rounded to the currency it belongs to. Writing

```formula
rounded_value = round_to( currency, raw_value )
```

means: round `raw_value` to the nearest multiple of that currency's rounding step, with halves going away from zero. The rounding step of a currency with `d` decimal places is `10^(−d)`. The complete definition, including the sub-step correction used to avoid binary floating-point artefacts, is in `../multi-currency/calculations.md`. Two derived predicates are used constantly:

```formula
is_zero( currency, value )          =  round_to( currency, value ) = 0

compare( currency, a, b )           =  −1  when round_to( currency, a − b ) < 0
                                        0  when round_to( currency, a − b ) = 0
                                       +1  when round_to( currency, a − b ) > 0
```

### 0.2 Signs

A journal item carries two signed quantities:

```formula
balance          = debit − credit                (in the company currency, positive on the debit side)
amount_currency  = the same movement expressed in the item's own currency, with the same sign
```

A residual keeps the sign of the amount it comes from. A receivable line of an unpaid customer invoice has a positive balance and a positive residual; a payable line of an unpaid vendor bill has a negative balance and a negative residual.

A Payment's `amount` is always positive; its direction carries the sign. A Bank Transaction's `amount` is signed as the bank reports it: positive for money in.

A Partial Reconciliation stores three amounts that are **always positive**.

### 0.3 Currency roles

Four currencies appear in the formulas and must never be confused.

| Role | Meaning |
|---|---|
| company currency | The currency the books are kept in. Every balance is expressed in it. |
| journal currency | The currency a liquidity journal is held in. When the journal names no currency, it is the company currency. |
| transaction currency | The currency of a Bank Transaction as the bank reported it: its foreign currency when it has one, otherwise the journal currency. |
| item currency | The currency of one journal item. It equals the company currency when the item carries no foreign amount. |

---

## 1. Residual amounts of a journal item

### 1.1 Which items have a residual

An item has residual amounts when its account is reconcilable, or when its account type is `asset_cash` or `liability_credit_card`. Every other item has residuals of zero and is never reconciled.

### 1.2 The formula

For each item, gather the matchings it takes part in:

```formula
matched_as_debit          = Σ  amount                   over the partial reconciliations whose debit item is this item
matched_as_debit_currency = round_to( item_currency, Σ debit_amount_currency over those same partial reconciliations )

matched_as_credit          = Σ  amount                   over the partial reconciliations whose credit item is this item
matched_as_credit_currency = round_to( item_currency, Σ credit_amount_currency over those same partial reconciliations )
```

The rounding is applied to the *sum*, not to each term, and it uses the decimal places of the currency stored on the matching for that side.

Then:

```formula
amount_residual          = round_to( company_currency, balance − matched_as_debit + matched_as_credit )

amount_residual_currency = round_to( item_currency, amount_currency − matched_as_debit_currency + matched_as_credit_currency )

reconciled = is_zero( company_currency, amount_residual )
             AND is_zero( item_currency, amount_residual_currency )
```

When the item has no company currency set (a record still being edited), the active company's currency is used; when it has no item currency, the company currency is used.

### 1.3 Worked example

A customer invoice of 1 000, in the company currency, partially settled twice: once by 400, once by 550.

```formula
balance                  = +1000.00
matched_as_credit        = 400.00 + 550.00 = 950.00     (the invoice line is the debit side, the payments the credit side,
                                                          so from the invoice line's point of view the matchings are
                                                          "matched as debit": the invoice item IS the debit item)
amount_residual          = round_to( company, 1000.00 − 950.00 + 0.00 ) = 50.00
amount_residual_currency = 50.00
reconciled               = false
```

---

## 2. The reconciliation operation

Reconciling is always performed on a **plan**: a list whose members are either a set of journal items or another plan. The plan expresses both *what* to match and *in which order*.

```
[ A, B ]      means: match the items of A first, then match the items of B.
[ [ A, B ] ]  means: match A, then match B, then match every item of A and B that is still unmatched, all together.
```

The operation runs in four phases: optimise the plan, prepare the matchings virtually, create them, then create the derived records.

### 2.1 Phase one — optimise the plan

Input: the plan as written by the caller. Output: a list of nodes, each node having a set of items, the identifiers of those items, and possibly a list of sub-nodes.

1. Walk the plan. A member that is a set of journal items becomes a **leaf**; a member that is itself a list becomes a **branch** whose sub-nodes are the optimised forms of its members and whose item set is the union of theirs.
2. For a leaf, sort its items. The sort key is, in order: the item's due date, or its date when it has no due date; then the item's currency; then the item's signed foreign amount; then the item's balance. Under the *reduced line sorting* flag the key is shortened to only the first two components.
3. For a leaf, look at the set of currencies present. If more than one currency is present, split the leaf: it keeps its own sorted item set, and it gains one sub-node per currency containing the items of that currency, in the sorted order. This guarantees that items of the same currency are matched with each other first, before any cross-currency matching is attempted.
4. For every top-level node, check that its items may legally be reconciled together (section 2.2). Drop empty nodes.
5. Return the list of nodes and the union of all their items.

### 2.2 Phase one — the eligibility check

Applied to the item set of every top-level node.

1. Collect the partial matching numbers (those starting with the letter `P`) carried by items of the set that are **not** fully reconciled.
2. Narrow the set to the items that are not reconciled, plus the items that are reconciled but whose matching number is not one of those collected numbers. This lets a caller pass a whole matching group, of which some items are already settled, without being refused.
3. If nothing is left, the check passes.
4. If any remaining item is reconciled: "You are trying to reconcile some entries that are already reconciled."
5. If any remaining item belongs to a cancelled entry: "You can not reconcile cancelled entries."
6. Collect the accounts of the remaining items. If there is more than one: "Entries are not from the same account: <the account display names, comma-separated>".
7. If the root companies of the remaining items are more than one: "Entries don't belong to the same company: <the company display names, comma-separated>".
8. If the single account is not reconcilable **and** its type is neither `asset_cash` nor `liability_credit_card`: "Account <the account display name> does not allow reconciliation. First change the configuration of this account to allow it."

### 2.3 Phase two — walking a node

Each node is processed depth-first: its sub-nodes first, in order, then its own item set.

For an item set:

1. Remove from the set every item already recorded as fully reconciled by an earlier step of this same operation.
2. If the remaining items belong to more than one counterparty, re-sort them by counterparty identifier (items without a counterparty first). This groups a mixed selection so that a counterparty's debits meet that counterparty's credits.
3. Run the pairing loop (section 2.4) on the remaining items in that order.

### 2.4 Phase two — the pairing loop

Given an ordered list of item states (each holding the item, its running residual in the company currency and its running residual in its own currency):

1. Build the **debit queue**: the states whose item has a balance greater than zero **or** a signed foreign amount greater than zero, in the given order.
2. Build the **credit queue**: the states whose item has a balance less than zero **or** a signed foreign amount less than zero, in the given order.
3. Set the current debit state and the current credit state to empty.
4. Repeat:
   1. If there is no current debit state, take the next one from the debit queue; if the queue is exhausted, stop.
   2. If there is no current credit state, take the next one from the credit queue; if the queue is exhausted, stop.
   3. Compute one matching between the current debit state and the current credit state (section 3). The computation updates both running residuals in place.
   4. If the computation produced matching values, record them.
   5. If the computation reports that the debit state has nothing left, mark its item fully reconciled for this operation and clear the current debit state.
   6. If it reports the same for the credit state, do likewise.
5. Return the recorded matchings and the set of items marked fully reconciled.

Note that an item can appear in **both** queues: an item whose balance is positive but whose foreign amount is negative, or the reverse, which happens on exchange-difference lines. The loop is nevertheless finite because every iteration either produces a matching that strictly reduces a residual, or removes a state from a queue.

### 2.5 Phases three and four

1. Create every Partial Reconciliation in one batch, in the order they were computed. Attach the created records back to the node that produced them.
2. When the *add cash-basis values* flag is set, store on each new matching the snapshot of the cash-basis values that produced it.
3. Create the exchange-difference entries (section 4) in one batch and post those whose two matched items both belong to posted entries.
4. Link each exchange entry to the matching that produced it: for each matching, in order, find the first unused exchange entry that references either of its two items and is not yet linked, and store it on the matching.
5. If neither the *reverse-cancel* flag nor the *no cash basis* flag is set, then for every node whose items sit on a receivable or payable account and whose company uses cash-basis taxes, create the cash-basis tax entries from that node's matchings, and store the cash-basis snapshot on them. The content of those entries belongs to `../taxes/`.
6. Detect full reconciliations (section 5).
7. Call the post-matching hook: every invoice that was not paid and is now `paid` or `in_payment`, and every invoice that was `in_payment` and is now `paid`, runs its paid hook.

---

## 3. Matching one debit item with one credit item

This is the heart of the domain. It is specified as numbered steps with their exact ordering because the result — how much is matched, in which currency, and whether an exchange difference appears — depends on that ordering.

Inputs: the debit state (item, running company-currency residual, running item-currency residual) and the credit state. Outputs: possibly a matching, possibly exchange-difference values, and two flags saying whether each state is exhausted.

### 3.1 Step 1 — the available residual per currency, for one item

For one item, given the currency of the item it is about to be matched with (the *counterpart currency*), build a map from currency to a pair (residual, rate).

Two rate helpers are used.

```formula
accounting_rate( item, currency ) = | amount_currency ÷ balance |        when balance ≠ 0 in the company currency
                                                                          and amount_currency ≠ 0 in that currency
                                  = undefined                            otherwise
```

```formula
platform_rate( item, other_item, currency ):
    1. if a forced rate was supplied by the register-payment flow, return it
    2. if there is an other item, the item is not a payment-like item and the other item is,
       return accounting_rate( other_item, currency )
    3. let rate_date = the item's entry invoice date when that entry is an invoice-like document,
                       otherwise the item's date
    4. return the conversion rate from the company currency to `currency`
       for the item's company at rate_date
```

An item is *payment-like* when its entry has an originating Payment or an originating Bank Transaction.

Now the map:

1. Let `remaining` be the running company-currency residual, `remaining_curr` the running item-currency residual, `item_currency` the item's currency, `account` the item's account.
2. If `remaining` is not zero in the company currency, add the entry `company currency → (remaining, rate 1)`.
3. If `item_currency` differs from the company currency and `remaining_curr` is not zero in `item_currency`, add the entry `item_currency → (remaining_curr, accounting_rate(item, item_currency))`.
4. If `item_currency` **equals** the company currency, **and** the account type is `asset_receivable` or `liability_payable`, **and** `remaining` is not zero, **and** the counterpart currency differs from the company currency: let `rate = platform_rate(item, other_item, counterpart_currency)` and `residual_in_foreign = round_to(counterpart_currency, remaining × rate)`; if that is not zero, add the entry `counterpart_currency → (residual_in_foreign, rate)`. This is the *mimicking* rule: an item booked in the company currency on a trade account can be matched against a foreign-currency item by pretending it also carries that foreign currency, converted at the rate of the day.
5. Otherwise, if `item_currency` equals the counterpart currency and differs from the company currency and `remaining_curr` is not zero, add the entry `counterpart_currency → (remaining_curr, accounting_rate(item, item_currency))`. (This duplicates the entry added in step 3 under the counterpart key; both keys are the same currency, so the effect is idempotent.)

### 3.2 Step 2 — choose the reconciliation currency

Build the map for the debit item with the credit item's currency as counterpart, and the map for the credit item with the debit item's currency as counterpart. Then:

1. If the debit currency differs from the company currency **and** it is present in both maps, the reconciliation currency is the debit currency.
2. Otherwise, if the credit currency differs from the company currency **and** it is present in both maps, the reconciliation currency is the credit currency.
3. Otherwise, the reconciliation currency is the company currency.

### 3.3 Step 3 — stop when there is nothing to match

Take the entry of the reconciliation currency from each map.

1. If the debit map has no such entry, the debit state is exhausted.
2. If the credit map has no such entry, the credit state is exhausted.
3. If either was exhausted, produce no matching and return.

Otherwise let:

```formula
recon_debit_amount  =   the debit entry's residual
recon_credit_amount = − the credit entry's residual
```

Both are positive for a normal debit-against-credit pair.

### 3.4 Step 4 — detect the exchange-line mode

```formula
exchange_line_mode = ( reconciliation currency = company currency )
                 AND ( debit currency = credit currency )
                 AND ( the debit map has no entry for the debit currency
                       OR the credit map has no entry for the credit currency )
```

This is true when one of the two items is an exchange-difference line: both items share a foreign currency, but one of them has no foreign amount left. In that mode no rate is applied, because an exchange difference must change only the company-currency amount and leave the foreign amount alone.

### 3.5 Step 5 — decide which side is fully matched

```formula
comparison            = compare( reconciliation currency, recon_debit_amount, recon_credit_amount )
min_recon_amount      = min( recon_debit_amount, recon_credit_amount )
debit_fully_matched   = ( comparison ≤ 0 )
credit_fully_matched  = ( comparison ≥ 0 )
```

When the two are equal, both flags are true.

### 3.6 Step 6 — the amounts, when the reconciliation currency is the company currency

1. If in exchange-line mode, both rates are undefined; otherwise the debit rate is the rate of the debit currency entry in the debit map (undefined when absent) and likewise for the credit.
2. `partial_amount = min_recon_amount`.
3. If the debit rate is defined:
   ```formula
   partial_debit_amount_currency = min( round_to( debit_currency, debit_rate × min_recon_amount ),
                                        remaining_debit_amount_currency )
   ```
   otherwise it is zero.
4. If the credit rate is defined:
   ```formula
   partial_credit_amount_currency = min( round_to( credit_currency, credit_rate × min_recon_amount ),
                                         − remaining_credit_amount_currency )
   ```
   otherwise it is zero.

### 3.7 Step 7 — the amounts, when the reconciliation currency is a foreign currency

A helper produces three values, the low, the middle and the high image of an amount after applying a rate. It exists because the amount being converted is itself a rounded number, so its true value lies in an interval.

```formula
range_after_rate( currency_from, currency_to, amount, rate ):
    if rate is undefined or zero:  return ( 0 , 0 , 0 )
    half_step = rounding step of currency_from ÷ 2
    low    = round_to( currency_to, ( amount − half_step ) × rate )
    middle = round_to( currency_to,   amount               × rate )
    high   = round_to( currency_to, ( amount + half_step ) × rate )
    return ( low , middle , high )
```

1. If in exchange-line mode, both rates are undefined; otherwise the debit rate is the rate stored in the debit map for the reconciliation currency, and likewise for the credit.
2. Convert the matched foreign amount back to the company currency on each side, using the **inverse** of the rate:
   ```formula
   debit_range  = range_after_rate( debit_currency , company_currency , min_recon_amount , 1 ÷ debit_rate  )
   credit_range = range_after_rate( credit_currency, company_currency , min_recon_amount , 1 ÷ credit_rate )

   partial_debit_amount  = min( debit_range.middle ,   remaining_debit_amount  )
   partial_credit_amount = min( credit_range.middle , − remaining_credit_amount )
   partial_amount        = min( partial_debit_amount , partial_credit_amount )
   ```
3. **The rounding-tolerance collapse.** If all four of the following hold, the two sides are indistinguishable once rounding is taken into account, and no exchange difference should be created:
   ```formula
   compare( company, partial_debit_amount , credit_range.high ) ≤ 0
   compare( company, partial_debit_amount , credit_range.low  ) ≥ 0
   compare( company, partial_credit_amount, debit_range.high  ) ≤ 0
   compare( company, partial_credit_amount, debit_range.low   ) ≥ 0
   ```
   Then set
   ```formula
   partial_amount        = min( remaining_debit_amount , − remaining_credit_amount )
   partial_debit_amount  = partial_amount
   partial_credit_amount = partial_amount
   ```
4. The foreign-currency amounts of the matching:
   ```formula
   partial_debit_amount_currency  = partial_amount    when debit_currency  = company_currency
                                  = min_recon_amount  otherwise
   partial_credit_amount_currency = partial_amount    when credit_currency = company_currency
                                  = min_recon_amount  otherwise
   ```

### 3.8 Step 8 — the exchange difference

Skipped entirely when either the *no exchange difference* flag or the *no recursive exchange difference* flag is set on the operation.

**When the reconciliation currency is the company currency:**

```formula
if debit_fully_matched:
    debit_exchange = remaining_debit_amount_currency − partial_debit_amount_currency
    if not is_zero( debit_currency, debit_exchange ):
        record an exchange line for the debit item, fixing its FOREIGN residual by debit_exchange
        remaining_debit_amount_currency ← remaining_debit_amount_currency − debit_exchange

if credit_fully_matched:
    credit_exchange = remaining_credit_amount_currency + partial_credit_amount_currency
    if not is_zero( credit_currency, credit_exchange ):
        record an exchange line for the credit item, fixing its FOREIGN residual by credit_exchange
        remaining_credit_amount_currency ← remaining_credit_amount_currency + credit_exchange
```

**When the reconciliation currency is a foreign currency**, four branches, evaluated in this order:

```formula
if debit_fully_matched:
    debit_exchange = remaining_debit_amount − partial_amount
    if not is_zero( company, debit_exchange ):
        record an exchange line for the debit item, fixing its COMPANY residual by debit_exchange
        remaining_debit_amount ← remaining_debit_amount − debit_exchange
        if debit_currency = company_currency:
            remaining_debit_amount_currency ← remaining_debit_amount_currency − debit_exchange
else:
    debit_exchange = partial_debit_amount − partial_amount
    if compare( company, debit_exchange, 0 ) > 0:
        record an exchange line for the debit item, fixing its COMPANY residual by debit_exchange
        remaining_debit_amount ← remaining_debit_amount − debit_exchange
        if debit_currency = company_currency:
            remaining_debit_amount_currency ← remaining_debit_amount_currency − debit_exchange

if credit_fully_matched:
    credit_exchange = remaining_credit_amount + partial_amount
    if not is_zero( company, credit_exchange ):
        record an exchange line for the credit item, fixing its COMPANY residual by credit_exchange
        remaining_credit_amount ← remaining_credit_amount − credit_exchange
        if credit_currency = company_currency:
            remaining_credit_amount_currency ← remaining_credit_amount_currency − credit_exchange
else:
    credit_exchange = partial_amount − partial_credit_amount
    if compare( company, credit_exchange, 0 ) < 0:
        record an exchange line for the credit item, fixing its COMPANY residual by credit_exchange
        remaining_credit_amount ← remaining_credit_amount − credit_exchange
        if credit_currency = company_currency:
            remaining_credit_amount_currency ← remaining_credit_amount_currency − credit_exchange
```

The two "else" branches exist so that, after a *partial* matching, the ratio between the item's company-currency residual and its foreign-currency residual still equals the ratio between its balance and its foreign amount. Without them a partially paid foreign invoice would drift.

If at least one exchange line was recorded, prepare an exchange-difference entry (section 4) dated on the later of the two items' dates, and mark it to be posted when both items belong to posted entries.

### 3.9 Step 9 — produce the matching and update the residuals

```formula
remaining_debit_amount           ← remaining_debit_amount           − partial_amount
remaining_credit_amount          ← remaining_credit_amount          + partial_amount
remaining_debit_amount_currency  ← remaining_debit_amount_currency  − partial_debit_amount_currency
remaining_credit_amount_currency ← remaining_credit_amount_currency + partial_credit_amount_currency
```

The matching values are:

| Stored field | Value |
|---|---|
| `amount` | `partial_amount` |
| `debit_amount_currency` | `partial_debit_amount_currency` |
| `credit_amount_currency` | `partial_credit_amount_currency` |
| `debit_move_id` | the debit item |
| `credit_move_id` | the credit item |

### 3.10 Step 10 — exhaustion

```formula
debit state is exhausted  when is_zero( debit_currency , remaining_debit_amount_currency  )
                           AND is_zero( company        , remaining_debit_amount           )

credit state is exhausted when is_zero( credit_currency, remaining_credit_amount_currency )
                           AND is_zero( company        , remaining_credit_amount          )
```

### 3.11 Worked example — a payment of 1 000 against invoices of 600 and 500

Single currency (the company currency). Customer *Northwind*. Two open invoices and one payment.

| Item | Entry | Account | Due date | Balance | Residual before |
|---|---|---|---|---|---|
| I1 | invoice A | receivable | 10 March | +600.00 | +600.00 |
| I2 | invoice B | receivable | 20 March | +500.00 | +500.00 |
| P | payment | receivable (destination) | 25 March | −1 000.00 | −1 000.00 |

The register-payment flow reconciles the three items together, so the plan is a single leaf containing I1, I2 and P.

**Sort.** The key is (due date, currency, foreign amount, balance). All three are in the company currency, so the foreign amount equals the balance. Order: I1 (10 March), I2 (20 March), P (25 March).

**Split by currency.** One currency only, so no sub-nodes.

**Queues.** Debit queue: I1, I2 (positive balances). Credit queue: P.

**Iteration 1 — I1 against P.**
- Debit map for I1 with counterpart currency = company currency: `{ company → (600.00, 1) }`. Step 4 of section 3.1 does not apply because the counterpart currency is the company currency.
- Credit map for P: `{ company → (−1000.00, 1) }`.
- Reconciliation currency: company currency (neither currency differs from it).
- `recon_debit_amount = 600.00`, `recon_credit_amount = 1000.00`.
- `comparison = compare(company, 600, 1000) = −1` → `debit_fully_matched = true`, `credit_fully_matched = false`.
- `min_recon_amount = 600.00`, `partial_amount = 600.00`.
- Step 6 looks up the rate under the key *debit currency*. Here the debit currency **is** the company currency, so the lookup finds the entry `company → (600.00, 1)` and `debit_rate = 1`. Likewise `credit_rate = 1`. Therefore
  ```formula
  partial_debit_amount_currency  = min( round_to( company, 1 × 600.00 ) , 600.00  ) = 600.00
  partial_credit_amount_currency = min( round_to( company, 1 × 600.00 ) , 1000.00 ) = 600.00
  ```
- Exchange difference: the reconciliation currency is the company currency and `debit_fully_matched` is true, so `debit_exchange = remaining_debit_amount_currency − partial_debit_amount_currency = 600.00 − 600.00 = 0.00`, which is zero: nothing is recorded. `credit_fully_matched` is false, so the credit branch is not evaluated at all. No exchange entry is produced — as expected in a single-currency reconciliation.
- Matching M1: amount 600.00, debit foreign 600.00, credit foreign 600.00, debit item I1, credit item P.
- Residuals: I1 → 0.00 ; P → −1000.00 + 600.00 = −400.00.
- I1 is exhausted; P is not.

**Iteration 2 — I2 against P.**
- Same shape. `recon_debit_amount = 500.00`, `recon_credit_amount = 400.00`, `comparison = +1` → `credit_fully_matched = true`, `debit_fully_matched = false`.
- `min_recon_amount = 400.00`, `partial_amount = 400.00`.
- Matching M2: amount 400.00, debit foreign 400.00, credit foreign 400.00, debit item I2, credit item P.
- Residuals: I2 → 500.00 − 400.00 = 100.00 ; P → −400.00 + 400.00 = 0.00.
- P is exhausted; the credit queue is empty; the loop stops.

**Result.**

| Item | Residual after | Matching number |
|---|---|---|
| I1 | 0.00 | `P<n>` then, if the whole group settles, the full number |
| I2 | 100.00 | `P<n>` |
| P | 0.00 | `P<n>` |

All three items belong to one connected matching group (I1–P, I2–P), so they share one partial matching number. No Full Reconciliation is created, because I2 still has 100.00 left. Invoice A is `paid`; invoice B is `partial`; the Payment is `in_process` until its liquidity side is matched with a bank transaction.

This is exactly why the ordering matters: had the items been sorted by balance instead of due date, the payment would have settled invoice B first and left 100.00 on invoice A.

---

## 4. Exchange-difference entries

### 4.1 When one is prepared

Step 8 of section 3 collects, for one matching, a list of (item, fix) pairs. A *fix* is either a company-currency amount to remove from the item's company residual, or a foreign-currency amount to remove from its foreign residual. The entry is prepared from that list.

### 4.2 The entry header

1. The company is the company of the invoice-like entries among the items, if any, otherwise the company of the items, otherwise the company passed explicitly. If none resolves, nothing is prepared.
2. The journal is the company's exchange-difference journal. If it is empty the preparation still proceeds but the creation later fails (section 4.5).
3. The date starts at the journal's accounting date computed for the supplied exchange date — that is, the supplied date pushed forward past any lock date, as specified in `../general-ledger/`. It is then raised to the maximum of itself and each processed item's date. When there is no journal at all, it starts at the lowest representable date.
4. The entry type is a plain entry, its number is forced to `/` so that numbering happens at posting time, and it is flagged as always tax-exigible.

### 4.3 The two lines per fix

For each (item, fix) pair, in order, with `sequence` starting at zero and increasing by two:

```formula
if the fix is a company-currency amount R:
    amount_residual          = R
    amount_residual_currency = R    when the item's currency is the company currency
                             = 0    otherwise
    if is_zero( company, R ): skip this pair

if the fix is a foreign-currency amount R:
    amount_residual          = 0
    amount_residual_currency = R
    if is_zero( item currency, R ): skip this pair

exchange_account = the company's EXPENSE exchange-difference account   when R > 0
                 = the company's INCOME  exchange-difference account   when R ≤ 0
```

| Line | Account | Debit | Credit | Foreign amount | Other values |
|---|---|---|---|---|---|
| first (sequence `s`) | the item's own account | `−amount_residual` when it is negative, else 0 | `amount_residual` when it is positive, else 0 | `− amount_residual_currency` | the item's counterparty, the item's currency, the item's full reconciliation, and the item itself recorded in the *reconciled lines* relation |
| second (sequence `s+1`) | the exchange account chosen above | `amount_residual` when it is positive, else 0 | `−amount_residual` when it is negative, else 0 | `amount_residual_currency` | the item's counterparty, the item's currency, and, when an analytic distribution was supplied by the caller, that distribution |

Both lines carry the label "Currency exchange rate difference".

The first line cancels the item's drift on the item's own account; the second books the gain or the loss. Because the first line records the original item in the *reconciled lines* relation, writing that relation reconciles the two automatically: the platform matches the item with the line that was just created for it.

### 4.4 Creating the entries

1. If the list of prepared entries is empty, stop (this guard also prevents the matching of an exchange entry from recursively producing another one).
2. For each prepared entry, if it has no journal: "You have to configure the 'Exchange Gain or Loss Journal' in your company settings, to manage automatically the booking of accounting entries related to differences between exchange rates."
3. For each journal used, if its company has no expense exchange account: "You should configure the 'Loss Exchange Rate Account' in your company settings, to manage automatically the booking of accounting entries related to differences between exchange rates."
4. For each journal used, if its company has no income exchange account: "You should configure the 'Gain Exchange Rate Account' in your company settings, to manage automatically the booking of accounting entries related to differences between exchange rates."
5. Create all the entries at once, with the *no exchange difference* flag set so that the reconciliation they trigger does not cascade.
6. Post the entries whose two matched items both belonged to posted entries, with analytic validation switched off.

### 4.5 Reversal on unreconciliation

Deleting the matchings deletes or reverses the exchange entries, as specified in `entities.md`, section 8.5: draft entries are deleted, posted entries are reversed with the cancel flag, dated on the original date or the day after the latest violated lock date, with the reference "Reversal of: <the entry number>".

### 4.6 Worked example — a bank transaction of 1 500 matched to a foreign-currency invoice at a different rate

The company keeps its books in currency **C**. The bank journal is held in currency **F**. The rate moves between the invoice and the receipt.

| Fact | Value |
|---|---|
| Invoice date rate | 1 C buys 0.75 F, so 1 500 F = 2 000 C |
| Transaction date rate | 1 C buys 0.80 F, so 1 500 F = 1 875 C |
| Invoice | one receivable item, balance +2 000.00 C, foreign amount +1 500.00 F, currency F |
| Bank transaction | amount +1 500.00 F in the journal currency, no separate foreign currency |

The transaction's own entry (section 9 below) is:

| Item | Account | Debit (C) | Credit (C) | Foreign amount (F) |
|---|---|---|---|---|
| liquidity | bank account | 1 875.00 | | +1 500.00 |
| suspense | bank suspense | | 1 875.00 | −1 500.00 |

The user matches the suspense item against the invoice's receivable item. Because both must sit on the same account, the reconciliation is performed after the suspense item's account has been switched to the receivable account of the counterparty (this is what the bank-reconciliation step does; see section 10). The pair is then:

- debit item: the invoice receivable item — balance +2 000.00 C, foreign +1 500.00 F, currency F;
- credit item: the transaction's counterpart item — balance −1 875.00 C, foreign −1 500.00 F, currency F.

**Step 1.** Debit map: `{ C → (2000.00, 1) , F → (1500.00, accounting_rate = |1500 ÷ 2000| = 0.75) }`. Credit map: `{ C → (−1875.00, 1) , F → (−1500.00, accounting_rate = |−1500 ÷ −1875| = 0.80) }`.

**Step 2.** The debit currency F differs from C and is present in both maps → the reconciliation currency is **F**.

**Step 3.** `recon_debit_amount = 1500.00`, `recon_credit_amount = 1500.00`.

**Step 4.** Not exchange-line mode: both maps have an entry for their own currency.

**Step 5.** `comparison = 0` → both sides fully matched. `min_recon_amount = 1500.00`.

**Step 7.** Foreign reconciliation currency.

```formula
debit_range  = range_after_rate( F , C , 1500.00 , 1 ÷ 0.75 = 1.333333… )
             half_step = 0.01 ÷ 2 = 0.005
             low    = round_to( C , 1499.995 × 1.333333… ) = 1999.99
             middle = round_to( C , 1500.000 × 1.333333… ) = 2000.00
             high   = round_to( C , 1500.005 × 1.333333… ) = 2000.01

credit_range = range_after_rate( F , C , 1500.00 , 1 ÷ 0.80 = 1.25 )
             low    = round_to( C , 1499.995 × 1.25 ) = 1874.99
             middle = round_to( C , 1500.000 × 1.25 ) = 1875.00
             high   = round_to( C , 1500.005 × 1.25 ) = 1875.01

partial_debit_amount  = min( 2000.00 , 2000.00 ) = 2000.00
partial_credit_amount = min( 1875.00 , 1875.00 ) = 1875.00
partial_amount        = min( 2000.00 , 1875.00 ) = 1875.00
```

The collapse test fails at its first clause: `compare(C, 2000.00, credit_range.high = 1875.01) = +1`, which is not `≤ 0`. The two sides are genuinely different, so an exchange difference is required.

```formula
partial_debit_amount_currency  = 1500.00      (the debit currency is F, not C)
partial_credit_amount_currency = 1500.00
```

**Step 8.** Reconciliation currency is foreign.

```formula
debit_fully_matched  = true
debit_exchange       = remaining_debit_amount − partial_amount = 2000.00 − 1875.00 = 125.00
                       not zero → an exchange line is recorded for the invoice item,
                       fixing its COMPANY residual by +125.00
remaining_debit_amount ← 2000.00 − 125.00 = 1875.00

credit_fully_matched = true
credit_exchange      = remaining_credit_amount + partial_amount = −1875.00 + 1875.00 = 0.00
                       zero → nothing recorded
```

**Step 9.**

```formula
remaining_debit_amount            = 1875.00 − 1875.00 = 0.00
remaining_credit_amount           = −1875.00 + 1875.00 = 0.00
remaining_debit_amount_currency   = 1500.00 − 1500.00 = 0.00
remaining_credit_amount_currency  = −1500.00 + 1500.00 = 0.00
```

Matching: amount 1 875.00 C, debit foreign 1 500.00 F, credit foreign 1 500.00 F.

**The exchange-difference entry**, in the company's exchange journal, dated on the later of the two item dates:

| Line | Account | Debit (C) | Credit (C) | Foreign amount (F) |
|---|---|---|---|---|
| 1 | the customer receivable account (the invoice item's own account) | | 125.00 | 0.00 |
| 2 | expense exchange-difference account | 125.00 | | 0.00 |

Line 1 is reconciled with the invoice receivable item, which therefore ends with a company residual of `2 000.00 − 1 875.00 − 125.00 = 0.00` and a foreign residual of `1 500.00 − 1 500.00 − 0.00 = 0.00`. The invoice is fully paid, and the 125.00 C the company lost to the rate movement sits in the exchange loss account.

**The collapse rule in action.** Change one number: suppose the transaction-date rate had been 0.7500025 instead of 0.80, giving a transaction of 1 500.00 F worth 1 999.99 C. Then `credit_range = (1999.98, 1999.99, 2000.00)` and `partial_debit_amount = 2000.00`. The four tests read: `compare(C, 2000.00, 2000.00) = 0 ≤ 0` ✓; `compare(C, 2000.00, 1999.98) = +1 ≥ 0` ✓; `compare(C, 1999.99, 2000.01) = −1 ≤ 0` ✓; `compare(C, 1999.99, 1999.99) = 0 ≥ 0` ✓. The collapse applies: `partial_amount = min(2000.00, 1999.99) = 1999.99` on both sides, and — because `debit_exchange = 2000.00 − 1999.99 = 0.01` is not zero in C — a one-cent exchange line is still produced for the debit side. The collapse prevents a *second*, spurious difference on the credit side.

---

## 5. Detecting a full reconciliation

Run after the matchings are created.

### 5.1 The connected group

For a set of items, the *connected group* is found through the matching number, not by walking the graph:

1. Collect the distinct non-empty matching numbers of the items.
2. Read every journal item in the database that carries one of those numbers, grouped by number, with elevated privileges.
3. The group is the original set plus every item read in step 2, ignoring numbers that start with the letter `I` (import placeholders are not real matchings yet).

### 5.2 Is the group complete

For each node of the plan, for each of its items not already assigned to a group:

1. Compute the connected group of the node's items.
2. Let `has_multiple_currencies` be true when the group's items carry more than one distinct currency.
3. An item of the group counts as *settled* when:
   - it is flagged reconciled; or
   - it has at least one matching **and** (when `has_multiple_currencies` is true) its company-currency residual is zero, or (otherwise) its item-currency residual is zero.
   An item with **no** matching at all never counts as settled, even when both its residuals are zero: this is the case of an exchange-difference line that has a foreign amount but no balance.
4. The group is fully reconciled when every one of its items counts as settled.

### 5.3 Creating the marker

For every group that is fully reconciled, create one Full Reconciliation linking every matching of the group's items and every item of the group. Creation writes the marker's identifier onto all of them and recomputes the matching numbers.

### 5.4 Worked example — a chain of partial reconciliations reaching full reconciliation

A customer invoice of 1 000.00 in the company currency is settled by three payments of 300.00, 300.00 and 400.00, registered on three different days.

**Day 1 — first payment of 300.**

Plan: `[ invoice receivable item + payment counterpart item ]`.

- Debit I (invoice): residual +1 000.00. Credit P1: residual −300.00.
- `comparison = compare(company, 1000, 300) = +1` → credit fully matched, debit not.
- `partial_amount = 300.00`. Matching M1 created: amount 300.00.
- Residuals: I → 700.00 ; P1 → 0.00.
- Connected group: {I, P1}. I is not settled (700.00 left) → **no** Full Reconciliation.
- Matching numbers: both items get the partial number `P` followed by the identifier of M1. Write it `P17` for this example.
- Invoice payment state: `partial`.

**Day 2 — second payment of 300.**

Plan: `[ invoice receivable item + payment counterpart item ]` again.

- Eligibility: I is not reconciled, so the check passes. I carries the partial number `P17` but it is not reconciled, so the narrowing rule of section 2.2 keeps it.
- `partial_amount = 300.00`. Matching M2 created.
- Residuals: I → 400.00 ; P2 → 0.00.
- Connected group of {I, P2}: the matching number of I is `P17`, so the group is read back as {I, P1, P2}. I still has 400.00 → **no** Full Reconciliation.
- Matching numbers: the graph now has edges M1 (I–P1) and M2 (I–P2). Both edges land in the same connected component, whose number is the smallest matching identifier, that is M1's. All three items keep `P17`.
- Invoice payment state: still `partial`.

**Day 3 — third payment of 400.**

- `comparison = compare(company, 400, 400) = 0` → both sides fully matched.
- `partial_amount = 400.00`. Matching M3 created.
- Residuals: I → 0.00 ; P3 → 0.00.
- Connected group of {I, P3}: I's number `P17` pulls in P1 and P2 as well, so the group is {I, P1, P2, P3}. Every one of them has at least one matching and a zero residual → the group **is** fully reconciled.
- One Full Reconciliation is created, linking matchings M1, M2, M3 and items I, P1, P2, P3. Suppose its identifier is 42.
- Matching numbers: every item of the group now has a full reconciliation, so each one's number becomes the text `42`. The partial number `P17` disappears.
- Invoice payment state: `paid` (or `in_payment` while the payments' own liquidity sides are still outstanding).

**Undoing day 3.** Deleting M3 deletes the Full Reconciliation first, then recomputes the numbers: the surviving edges M1 and M2 still form one component whose smallest identifier is M1's, so I, P1 and P2 return to `P17`, while P3, which now has no matching at all, ends with no matching number. The invoice returns to `partial` with a residual of 400.00.

---

## 6. The matching number

Every journal item that takes part in at least one matching carries a `matching_number`. It is recomputed after every creation and every deletion of matchings.

### 6.1 The algorithm

Input: a set of journal items. Output: a matching number written on each item of the connected closure of that set.

1. Extend the set to its connected closure through the matching numbers (section 5.1).
2. Collect every matching that touches an item of the closure, on either side.
3. Sort the matchings by ascending identifier.
4. Maintain two indexes: `number → list of item identifiers`, and `item identifier → number`.
5. For each matching in order, look up the numbers currently assigned to its debit item and to its credit item:
   1. **Both known and different** — merge the two components. The surviving number is the smaller of the two. Every item of the larger-numbered component is re-pointed at the smaller number, and its item list is appended to the smaller one's.
   2. **Both known and equal** — nothing to do.
   3. **Only the debit known** — add the credit item to that component.
   4. **Only the credit known** — add the debit item to that component.
   5. **Neither known** — start a new component whose number is this matching's own identifier, containing both items.
6. Write the result: for each item, if it has a Full Reconciliation, the matching number is the decimal text of the Full Reconciliation's identifier; otherwise it is the letter `P` followed by the component number.
7. Any item of the closure that received no component at all (it takes part in no matching) has its matching number cleared.

The component number is therefore the **smallest matching identifier** of the component, which makes the number stable as long as the oldest matching of the group survives.

### 6.2 The permitted shapes

A constraint enforces the following, raising an internal error when violated:

| Shape | Rule |
|---|---|
| empty | The item must have no matching. Otherwise: "Should have number". |
| matches `P` followed by digits | The item must have at least one matching, and must have **no** Full Reconciliation. Otherwise: "Should have partials" or "Should not be partial number". |
| all digits | The item must have a Full Reconciliation whose identifier equals that number. Otherwise: "Should not be full number" or "Matching number should be the full reconcile". |
| starts with `I` followed by at least one character | An import placeholder. The item must have no matching at all: "A temporary number can not be used in a real matching". |
| anything else | "Invalid matching number format". |

### 6.3 Deferred matching on import

Items imported with a matching number starting with the letter `I` are not matched yet. The deferred-matching operation:

1. Collects the distinct placeholder numbers among the items considered.
2. Groups every item in the database carrying one of those numbers, by (number, account).
3. For each group, if **every** entry of the group is posted:
   - if the account is not reconcilable, make it reconcilable and record that in the log;
   - reconcile the group's items with the *no exchange difference* and *no cash basis* flags set.

---
