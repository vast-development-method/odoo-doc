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

## 7. Bank statement arithmetic

### 7.1 The ordering index of a transaction

Every Bank Transaction carries a text sort key so that "the transactions before this one" is a single string comparison rather than a compound condition on date, sequence and identifier.

```formula
internal_index = date_as_eight_digits( date )
               ‖ pad_left_with_zeros_to_ten( 2147483647 − sequence )
               ‖ pad_left_with_zeros_to_ten( identifier )
```

where `‖` is text concatenation, `date_as_eight_digits` writes the year on four digits, the month on two and the day on two with no separator, and 2 147 483 647 is the largest value the sequence column can hold.

Three consequences:

1. Transactions sort primarily by date, ascending.
2. Within one date, the **complement** of the sequence is used, so a **higher** sequence sorts **earlier**. This is deliberate: the default record ordering is by index descending, and a user who wants a transaction at the top of the day gives it a higher sequence.
3. Within one date and one sequence, the identifier breaks the tie ascending, so the oldest record comes first.

The index is only computed for a transaction that already exists in the database, because it embeds the identifier.

**Worked example.** A transaction dated 15 March 2026, with sequence 1 and identifier 42:

```formula
date part     = 20260315
sequence part = 2147483647 − 1 = 2147483646 → "2147483646"
identifier    = 42                          → "0000000042"
internal_index = "2026031521474836460000000042"
```

A second transaction the same day with sequence 5 and identifier 43 gives `"2026031521474836420000000043"`, which is **smaller**, so it sorts earlier.

### 7.2 The running balance of a transaction

The running balance is the balance of the journal after the transaction, in occurrence order, anchored on statements so that the whole history does not have to be replayed.

For each journal present in the set of transactions being displayed:

1. Take the smallest and the largest ordering index among those transactions.
2. Find the most recent statement of that journal whose first-line index is **strictly smaller** than the smallest index, ordered by first-line index descending. If one exists, the running balance starts at that statement's starting balance (zero when it has none) and the scan is limited to transactions whose index is greater than or equal to that statement's first-line index. If none exists, the running balance starts at zero and the scan covers everything up to the largest index.
3. Read, in ascending index order, every transaction of that journal whose index is less than or equal to the largest index (plus the lower bound from step 2 when there is one), whose company is the journal's company or one of its descendants, together with: its amount, whether it is the first transaction of its statement, its statement's starting balance, and its entry's state.
4. Walk that list in order, keeping a running total:
   ```formula
   for each transaction in ascending index order:
       if the transaction is the first one of a statement:
           running_total ← that statement's starting balance     (this re-anchors the chain)
       if the transaction's entry state is 'posted':
           running_total ← running_total + the transaction's amount
       if the transaction is one of those being displayed:
           its running balance ← running_total
   ```
5. A transaction that was deleted from the form and therefore does not appear in the scan keeps whatever value it had.

Draft and cancelled transactions do **not** add their amount, but a draft or cancelled transaction that is the first of a statement still re-anchors the total.

### 7.3 The starting balance of a statement

Computed per statement, processing the statements in ascending order of first-line index.

1. Let `first` be the statement's first-line index and `journal` be its journal (or the journal of its transactions when the statement's own field is not yet set).
2. Find the most recent **posted** transaction of that journal whose index is smaller than `first` and that belongs to some statement. Call it the *anchor*.
3. Start from the anchor's statement's **ending balance**, or zero when there is no anchor.
4. Build the set of *in-between* transactions: posted transactions of that journal whose index is smaller than `first` and, when an anchor exists, greater than the anchor's index.
5. When an anchor exists, subtract from the starting balance the amounts of the transactions that belong to the anchor's statement **and** are also part of the statement being computed. This handles the case of a user moving transactions from one statement to another.
6. Add the amounts of the in-between transactions.
7. The result is the statement's starting balance.

```formula
balance_start = ending_balance_of_anchor_statement
              − Σ amount over transactions of the anchor statement that also belong to this statement
              + Σ amount over posted transactions strictly between the anchor and the first transaction
```

### 7.4 Computed and reported ending balances

```formula
balance_end = balance_start + Σ amount over the statement's transactions whose entry state is 'posted'
```

`balance_end_real`, the *reported* ending balance, is a stored editable field whose computation simply copies `balance_end`. A freshly created statement is therefore complete by construction; the user only overrides the reported balance when the bank says otherwise.

### 7.5 Completeness

```formula
is_complete = ( the statement has at least one posted transaction )
          AND compare( statement currency , balance_end , balance_end_real ) = 0
```

### 7.6 Validity

For a single statement:

```formula
previous = the statement of the same journal with the greatest first_line_index
           strictly smaller than this statement's first_line_index, among those that have one

is_valid = ( previous does not exist )
        OR compare( statement currency , balance_start , previous.balance_end_real ) = 0
```

For a set of statements the same test is expressed as a single pass: for every statement that has a first-line index, take the previous statement of the same journal ordered by first-line index, round both the previous reported ending balance and this starting balance to the decimal places of the journal's currency (or the company's when the journal has none), and mark the statement invalid when the two rounded values differ. Statements with no previous statement, and statements with no transactions, are valid.

A journal is flagged as *having invalid statements* when at least one of its statements is invalid.

### 7.7 Worked example — a chain of three statements

Journal held in the company currency. Transactions, in index order:

| Transaction | Amount | Statement |
|---|---|---|
| T1 | +1 000.00 | S1 |
| T2 | −250.00 | S1 |
| T3 | +400.00 | S2 |
| T4 | −100.00 | (none) |
| T5 | +50.00 | S3 |

S1 is the first statement of the journal: its starting balance is 0.00 (no anchor, no in-between transactions), its computed ending balance is `0 + 1000 − 250 = 750.00`. The bank reports 750.00, so S1 is complete and valid.

S2 starts at T3. The anchor is T2 (the last posted transaction before T3 that belongs to a statement); its statement S1 has a reported ending balance of 750.00. No transaction of S1 belongs to S2, and there is nothing between T2 and T3. So S2's starting balance is 750.00, its computed ending balance is `750 + 400 = 1 150.00`.

S3 starts at T5. The anchor is T3, whose statement S2 reports 1 150.00. In between sits T4, which belongs to no statement, so its −100.00 is added: S3's starting balance is `1150 − 100 = 1 050.00`, and its computed ending balance is `1050 + 50 = 1 100.00`.

Now suppose the bank reports 1 145.00 as the ending balance of S2 instead of 1 150.00. Then S2 is **not complete** (the computed 1 150.00 differs from the reported 1 145.00) and S3 is **not valid** (its starting balance of 1 050.00 differs from S2's reported 1 145.00 minus the 100.00 of T4, that is 1 045.00). The journal is flagged as having invalid statements and the dashboard shows the warning.

---

## 8. The journal items of a Bank Transaction

### 8.1 The three currencies

```formula
company_currency     = the journal's company currency
journal_currency     = the journal's own currency, or the company currency when it has none
foreign_currency     = the transaction's foreign currency,
                       or the journal currency, or the company currency, whichever is first non-empty
```

### 8.2 The three amounts

```formula
journal_amount     = the transaction's amount                        (always in the journal currency)

transaction_amount = journal_amount                                   when foreign_currency = journal_currency
                   = the transaction's foreign amount                 otherwise

company_amount     = journal_amount                                   when journal_currency = company_currency
                   = transaction_amount                               when foreign_currency = company_currency
                   = convert( journal_amount ,
                              from journal_currency to company_currency ,
                              for the journal's company , at the transaction's date )   otherwise
```

### 8.3 The two items

| Item | Account | Currency | Foreign amount | Debit | Credit | Label | Counterparty |
|---|---|---|---|---|---|---|---|
| liquidity | the journal's default account | `journal_currency` | `journal_amount` | `company_amount` when positive, else 0 | `−company_amount` when negative, else 0 | the transaction's label | the transaction's counterparty |
| counterpart | the explicit counterpart account when one was supplied, otherwise the journal's suspense account | `foreign_currency` | `− transaction_amount` | `−company_amount` when negative, else 0 | `company_amount` when positive, else 0 | the transaction's label | the transaction's counterparty |

If no counterpart account resolves: "You can't create a new statement line without a suspense account set on the <journal display name> journal."

### 8.4 The rates read back from the entry

Given a Bank Transaction and its entry, the *accounting amounts and currencies* are read back as follows:

1. Split the entry's items into liquidity, suspense and other.
2. If there is a suspense item and no other item, the transaction amount is minus the suspense item's foreign amount and the transaction currency is the suspense item's currency.
3. Otherwise — that is, when the transaction is partially reconciled or is flagged to be checked, so the suspense item can no longer be trusted — the transaction amount is the transaction's own foreign amount when it has a foreign currency, otherwise its plain amount; and the transaction currency is its foreign currency when it has one, otherwise the liquidity item's currency.
4. The journal amount is the sum of the liquidity items' foreign amounts, in the liquidity items' currency.
5. The company amount is the sum of the liquidity items' balances, in the company currency.

### 8.5 Converting a counterpart amount at the bank's own rate

When a journal item is to be settled by this transaction, the amounts that must be written on the counterpart line are computed with the **rates implied by the transaction itself**, not with the rate table. This is what makes a bank's own conversion rate prevail over the company's.

```formula
rate_journal_to_transaction = | transaction_amount | ÷ | journal_amount |     when journal_amount ≠ 0, else 0
rate_company_to_journal     = | journal_amount     | ÷ | company_amount |     when company_amount ≠ 0, else 0
```

Given a currency, a company-currency balance and an amount expressed in that currency:

1. **The given currency is the transaction currency.**
   ```formula
   new_transaction_amount = the given amount
   new_journal_amount     = round_to( journal_currency , new_transaction_amount ÷ rate_journal_to_transaction )
                            or 0 when that rate is 0
   new_balance            = round_to( company_currency , new_journal_amount ÷ rate_company_to_journal )
                            or 0 when that rate is 0
   ```
2. **The given currency is the journal currency.**
   ```formula
   new_transaction_amount = round_to( transaction_currency , the given amount × rate_journal_to_transaction )
   new_balance            = round_to( company_currency , the given amount ÷ rate_company_to_journal )
                            or 0 when that rate is 0
   ```
3. **Any other currency.**
   ```formula
   new_journal_amount     = round_to( journal_currency , the given balance × rate_company_to_journal )
   new_transaction_amount = round_to( transaction_currency , new_journal_amount × rate_journal_to_transaction )
   new_balance            = the given balance
   ```

The result is the pair (`amount_currency` = `new_transaction_amount`, `balance` = `new_balance`) to set on the counterpart journal item.

---

## 9. Reconciling a bank transaction

This is the operation a user performs on the reconciliation screen: taking a transaction whose amount currently sits on the suspense account and explaining it, either by matching it with open journal items, by allocating it to accounts through a reconciliation model, or by creating a Payment for it.

### 9.1 The candidate journal items

The set of journal items a transaction may be matched with is:

1. Entry state is `posted`. When draft documents are allowed (a setting of the reconciliation screen; never allowed for automatic matching or for reconciliation-model suggestions), the set is widened to also include items of draft entries that have a counterparty.
2. The item's display type is not a section, a subsection or a note.
3. The item's company is the transaction's company or one of its descendants — so documents of child companies can be matched, consistently with what the screen shows.
4. The item is not reconciled.
5. The item's account is reconcilable. (Expressed as: the account is one of the reconcilable accounts of the transaction's root company tree — this phrasing lets the database use the index on unreconciled items.)
6. Either the item's account type is neither `asset_receivable` nor `liability_payable`, or the item does not belong to a Payment. A Payment's own receivable or payable counterpart line is therefore never offered: the money side of a Payment is its outstanding line, not its trade line.
7. The item does not belong to this same transaction.

### 9.2 The operation, step by step

The reconciliation screen is part of the extended accounting capability; the stored model constrains it completely, and the following ordering is the one the stored model requires. Steps marked *industry-standard default* are the conventional behavior that the stored contract implies but does not itself encode.

1. **Load the transaction.** Read its journal, its amount, its currency triple, its counterparty and its label, and split its entry into liquidity, suspense and other items.
2. **Determine the open amount.** The open amount is the transaction's residual (section 5 of `state-machines.md`): minus the foreign amount when the transaction has a foreign currency and is not checked, otherwise the sum of the suspense items' residuals or foreign amounts.
3. **Detect the counterparty**, when the transaction has none:
   1. If the bank reported an account number, look for a Bank Account whose account number matches it; if found, its holder is the counterparty.
   2. Otherwise, evaluate the reconciliation models that are partner mappings (section 10) in their ordering; the first one whose label condition matches supplies the counterparty.
   3. *Industry-standard default:* otherwise, match the reported third-party name against known counterparties.
4. **Apply the reconciliation models** in their ordering (section 10). The first model whose every condition holds wins. If it is a partner mapping it only sets the counterparty and evaluation continues with the next model; otherwise its counterpart lines are computed (section 11) and proposed.
5. **Propose matching journal items.** Among the candidates of section 9.1, restricted to the detected counterparty when there is one, propose those whose residual matches the open amount, preferring: an exact amount match on the same counterparty; then a reference match between the transaction label and the document's payment reference or name; then the oldest due date. *Industry-standard default.*
6. **Let the user adjust.** The user may add or remove matched items, add manual counterpart lines with an account and an amount, change the counterparty, or choose a different model.
7. **Compute the counterpart amounts.** For each matched journal item, the amounts to write are produced by the bank's own rates (section 8.5) from the item's residual. For each manual or model line, the amount is the line's own amount converted the same way.
8. **Decide partial or full.** For each matched item, if the open amount is smaller than the item's residual, the match is partial: only the open amount is taken and the item keeps a residual. A partial match is **forbidden** when the item belongs to a Payment whose payment method code matches a transfer-file method (a code equal to `sepa_ct` or starting with `iso20022`) or whose Payment has an online transaction: such a payment is all-or-nothing.
9. **Rewrite the entry.** Replace the suspense item by the computed counterpart items, keeping exactly one liquidity item untouched. Every counterpart item carries the transaction's counterparty, the label of its source (the model line's label, the document's label, or the transaction's label), and, when it comes from a model line, that model line's account, taxes, analytic distribution and the reference to the model itself.
10. **Reconcile.** For each account present among the new counterpart items, reconcile those items with the matched journal items of the same account, through the plan of section 2.
11. **Mark the transaction.** The residual recomputes; when it reaches zero the transaction becomes reconciled. When the model that was applied is automated, the entry is also marked as checked. When the model names a next activity type, an activity of that type is scheduled on the transaction.
12. **Create a Payment when asked.** When the user chooses to record the transaction as a Payment rather than as a direct allocation, a Payment is created with the transaction's counterparty, amount, direction and date, its outstanding account replaced by the transaction's counterpart account, and it is added to the transaction's auto-generated payments so that undoing the reconciliation deletes it.

### 9.3 Undoing

Specified in `entities.md`, section 5.7: remove every matching on the entry's items, delete the auto-generated Payments, clear the entry's items and recreate the default liquidity and suspense pair, and set the checked flag to whether the current user may review entries.

---

## 10. The reconciliation model matching algorithm

Models are evaluated **in their stored order**: by `sequence` ascending, then by identifier ascending. The first model whose every condition holds is the one that applies. This section states the conditions and the exact ordering.

### 10.1 The evaluation, step by step

Input: one Bank Transaction. Output: at most one applicable model, plus possibly a counterparty detected by a partner-mapping model.

1. **Select the candidate models.** Take every Reconciliation Model that is active and whose company is an ancestor of the transaction's company. Order them by `sequence` ascending, then by identifier ascending.
2. **Filter on the journal.** A model applies only when its journal list is empty, or when it contains the transaction's journal. An empty list means every liquidity journal.
3. **Filter on the counterparty.** A model applies only when its counterparty list is empty, or when it contains the transaction's counterparty. An empty list means every counterparty. A transaction with no counterparty passes only when the list is empty.
4. **Filter on the amount.** When the model names no amount condition, it passes. Otherwise let `a` be the absolute value of the transaction's amount, in the transaction currency:
   | Condition | Test |
   |---|---|
   | `lower` | `a ≤ match_amount_max` |
   | `greater` | `a ≥ match_amount_min` |
   | `between` | `match_amount_min ≤ a ≤ match_amount_max` |
   For `lower` the form only asks for the maximum; for `greater` only for the minimum; for `between` both are required.
5. **Filter on the label.** When the model names no label condition, it passes. Otherwise the test is applied against a *text pool* made of the transaction's label, the textual content of its raw transaction details, and its note:
   | Condition | Test |
   |---|---|
   | `contains` | The parameter appears in the pool, ignoring letter case. |
   | `not_contains` | The parameter does not appear in the pool, ignoring letter case. |
   | `match_regex` | The parameter, read as a regular expression, matches somewhere in the pool. |
6. **Classify the model.** A model whose `mapped_partner_id` is set is a **partner mapping**; every other model is a **counterpart model**.
7. **Run the mappings first.** Walk the ordered candidate list. For every partner mapping that passes steps 2 to 5, set the transaction's counterparty to that mapping's counterparty and stop looking for a counterparty. Partner mappings never produce counterpart lines and never stop the search for a counterpart model.
8. **Run the counterpart models.** Walk the ordered candidate list again, now with the counterparty possibly filled in by step 7 so that step 3 can use it. The first counterpart model that passes steps 2 to 5 is the applicable model.
9. **Restrict the proposals.** A counterpart model whose `can_be_proposed` flag is false is never offered spontaneously: it can only be chosen explicitly by the user. The flag is true when the model is not a partner mapping **and** at least one of the following holds: it has a label condition, it has an amount condition, it has a counterparty list, or its trigger is `auto_reconcile`. A model with no condition at all and no automation would otherwise match every transaction.
10. **Apply.** Compute the counterpart lines (section 11). When the model's trigger is `auto_reconcile`, write them, reconcile, and mark the transaction's entry as checked without asking the user. When the trigger is `manual`, present them for confirmation.
11. **Schedule the follow-up.** When the model names a next activity type, schedule an activity of that type on the transaction.
12. **Record the origin.** Every journal item created by a model line stores a reference to the model, so the model can later count and list what it produced.

### 10.2 The ordering rules, restated

- Models: `sequence` ascending, then identifier ascending.
- Within one model, lines: `sequence` ascending, then identifier ascending. The order matters because a `percentage` line consumes the balance left by the lines before it.
- Partner mappings are evaluated before counterpart models, so that a counterpart model that filters on the counterparty can see the counterparty a mapping has just discovered.

### 10.3 Partner mapping

A model is a partner mapping exactly when all four hold:

1. it has a label condition (`match_label` is set);
2. it has exactly one line;
3. that line names a counterparty;
4. that line names **no** account.

Such a model exists purely to say "a transaction whose label looks like this belongs to this counterparty". Because it is a mapping, its `can_be_proposed` flag is false and it never writes anything.

---

## 11. Reconciliation model counterpart amounts

### 11.1 Definitions

```formula
transaction_amount = the transaction's amount in the transaction currency, signed as the bank reports it
open_balance       = what is still unexplained, at the moment the line is evaluated,
                     signed opposite to the transaction amount
                     (it starts at − transaction_amount and is reduced by each line already written)
```

### 11.2 The four modes

Lines are evaluated in order. For each line:

| Mode | Amount of the counterpart line |
|---|---|
| `fixed` | ```formula line_amount = the numeric value of amount_string ``` A negative value produces a debit; a positive value produces a credit. |
| `percentage` | ```formula line_amount = round_to( transaction currency , open_balance × amount_string_value ÷ 100 ) ``` |
| `percentage_st_line` | ```formula line_amount = round_to( transaction currency , transaction_amount × amount_string_value ÷ 100 ) ``` Note that this reads the **transaction** amount, not the open balance, so two such lines of 50 % each always split the original amount, whatever the other lines did. |
| `regex` | Apply the regular expression to the transaction label. If it does not match, the line produces nothing. Otherwise concatenate the capture groups in order, replace a comma by a decimal point, and read the result as a number. That number is the line amount. |

After each line, `open_balance` is reduced by the line amount. When every line has been evaluated and `open_balance` is still not zero, the remainder stays on the suspense account and the transaction is only partially reconciled.

A line with a counterparty writes that counterparty on its journal item; otherwise the transaction's counterparty is used. A line with taxes computes them on its amount through the tax engine of `../taxes/`, treating the line amount as tax-included or tax-excluded according to each tax's own setting, and produces the tax journal items alongside the base one. A line with an analytic distribution copies it onto its journal item.

### 11.3 Worked example — a reconciliation model with a percentage counterpart

Model **"Line with Bank Fees"**, as shipped in the demonstration data:

| Property | Value |
|---|---|
| Label condition | `contains`, parameter `BRT` |
| Line 1 | label "Due amount", account: an income account, amount mode `regex`, amount text `BRT: ([\d,.]+)` |
| Line 2 | label "Bank Fees", account: a finance-expense account, amount mode `percentage`, amount text `100` |

A bank transaction arrives:

```
label  : R:9672938 10/07 AX 9415126318 T:5L:NA BRT: 3358,07 C:
amount : +3 350.00   (in the company currency, which is also the journal currency)
```

**Matching.** The label contains `BRT`, so the model applies. The transaction's entry currently has:

| Item | Account | Debit | Credit | Foreign amount |
|---|---|---|---|---|
| liquidity | bank | 3 350.00 | | +3 350.00 |
| suspense | bank suspense | | 3 350.00 | −3 350.00 |

**Initial open balance.** `open_balance = −transaction_amount = −3 350.00`. A negative open balance means "3 350.00 still has to be credited somewhere".

**Line 1 — the regular-expression line.** Apply `BRT: ([\d,.]+)` to the label. It matches and captures `3358,07`. Replacing the comma by a decimal point gives 3 358.07. The line amount is therefore **−3 358.07** in the credit direction: the gross income is 3 358.07.

```formula
line_1_amount = 3358.07  (credit)
open_balance ← −3350.00 + 3358.07 = +8.07
```

**Line 2 — the percentage line.**

```formula
line_2_amount = round_to( company currency , open_balance × 100 ÷ 100 )
              = round_to( company currency , 8.07 × 1 )
              = 8.07   (debit)
open_balance ← 8.07 − 8.07 = 0.00
```

**The rewritten entry:**

| Item | Account | Debit | Credit | Label |
|---|---|---|---|---|
| liquidity | bank | 3 350.00 | | the transaction label |
| counterpart 1 | income | | 3 358.07 | Due amount |
| counterpart 2 | finance expense | 8.07 | | Bank Fees |

The suspense item is gone, the open balance is zero, and the transaction is reconciled. The 8.07 difference between what was invoiced and what the bank credited is booked as a bank fee.

**The same model with the lines in the other order** would behave completely differently: a `percentage` line of 100 evaluated first would consume the whole 3 350.00, and the regular-expression line would then add 3 358.07 on top, leaving an open balance of −3 358.07. The line ordering is part of the model's definition.

**A second reading, with `percentage_st_line`.** If line 2 had used `percentage_st_line` with the value 100 instead, its amount would be `round_to(company, 3350.00 × 100 ÷ 100) = 3350.00` regardless of what line 1 consumed, and the open balance would end at `8.07 − 3350.00 = −3341.93`, leaving the transaction unreconciled. The two percentage modes are not interchangeable.

---

## 12. Register-payment arithmetic

### 12.1 The grouping key of a journal item

```formula
batch_key( item ) = (
    counterparty          = item.partner_id ,
    account               = item.account_id ,
    currency              = item.currency_id ,
    recipient bank account= item.move_id.partner_bank_id when the entry is an invoice-like document, else empty ,
    counterparty kind     = 'customer' when item.account_type = 'asset_receivable' , else 'supplier'
)
```

Two items belong to the same batch exactly when their five-part keys are equal.

### 12.2 Building the batches

1. Take the selected journal items. If their root companies are more than one: "You can't create payments for entries belonging to different companies." If there are none: "You can't open the register payment wizard without at least one receivable/payable line."
2. Group the items by their batch key, preserving the order in which keys first appear.
3. While grouping, also record, for each counterparty, the set of recipient bank accounts seen on the **inbound** side (items whose balance is positive) and the set seen on the **outbound** side (items whose balance is negative), each as an ordered set.
4. Let `unique_inbound` be the counterparties with exactly one distinct inbound recipient account, and `unique_outbound` the counterparties with exactly one distinct outbound recipient account.
5. Walk the batch keys in their order of first appearance. Skip a key already absorbed by an earlier merge. For the current batch:
   1. `merge = counterparty is in unique_inbound AND counterparty is in unique_outbound`.
   2. If `merge`, look at every later, not-yet-absorbed batch. Absorb it when its key agrees with the current key on **every part except the recipient bank account** (the direction is not part of the key at this stage). Absorbing means appending its items to the current batch and marking its key as seen.
   3. Compute `balance = Σ balance over the batch's items`, and set the batch's direction to `inbound` when it is greater than zero, `outbound` otherwise.
   4. If `merge`, set the batch's recipient bank account to the single account recorded for that counterparty in the resulting direction, and replace the batch's item list with the merged one.
   5. Append the batch to the result.

Merging exists so that a counterparty who has one open invoice and one open credit note — which carry different recipient accounts on the documents but resolve to the same single account per direction — yields **one** net payment rather than two.

### 12.3 Values read from a batch

```formula
company = the batch item company with the fewest ancestors
          (or the root company when the items come from sibling companies)

source_amount          = | Σ amount_residual over the batch's items |          in the company currency
source_amount_currency = source_amount                                          when the batch currency = the company currency
                       = | Σ amount_residual_currency over the batch's items |  otherwise
```

The batch also yields the counterparty, the counterparty kind, the direction and the source currency.

Items come from *sibling companies* when they span more than one company and none of those companies is the ancestor of the others.

### 12.4 The installment reading of a document

For one document, with a payment currency, a payment date and an optional *next payment date*:

1. Take the document's payment-term journal items, sorted by due date (items with no due date last), then by date.
2. Let `sign` be the document's direction sign: `+1` for a plain entry or an outbound document, `−1` otherwise.
3. Walk the items, numbering them from 1. For each, build an installment record holding: the number, the item, the due date (or the item's date when it has none), the residual in the company currency, the residual in the item currency, the two unsigned variants obtained by multiplying each residual by `−sign`, the type (initially `other`) and whether the item is reconciled.
4. If the item is reconciled, leave the type at `other` and move on.
5. **Early payment discount.** If the document is eligible for an early payment discount for the payment currency at the payment date, overwrite the installment's four amounts with the discounted ones — the item's discount foreign amount and discount balance, and their unsigned variants — record the discount itself as `amount_currency − discount_amount_currency` and `balance − discount_balance`, set the type to `early_payment_discount`, and move on.
   Eligibility holds when: the document's currency equals the payment currency; the document type is an invoice or a receipt, of either direction; the document's payment term declares an early discount; either there is no reference date, or the document has no invoice date, or the first payment-term item has a discount date that is not before the reference date; and no payment-term item of the document is already matched with anything.
6. **Installments.** If the item is a payment-term item, classify it:
   - `before_date` when a next payment date was supplied and the item's due date is not after it;
   - otherwise `overdue` when the due date is strictly before the payment date; this also latches the document's first mode to `overdue`;
   - otherwise `next` when no first mode has been latched yet; this latches the first mode to `next`;
   - otherwise `next` when the current mode is `overdue` — after a run of overdue installments, the following one is offered as "the next";
   - the installment's type is the mode so determined.
7. Return the list of installments.

### 12.5 Converting a list of installments into the wizard currency

```formula
group the installments by their item's currency
for each currency:
    residual          = Σ amount_residual          over the installments of that currency
    residual_currency = Σ amount_residual_currency over the installments of that currency

    if currency = wizard currency:
        contribution = residual_currency
    else if currency ≠ company currency AND wizard currency = company currency:
        contribution = convert( residual_currency , from currency to company currency ,
                                for the company , at the payment date )
    else:
        contribution = convert( residual , from company currency to wizard currency ,
                                for the company , at the payment date )

total = Σ contribution
```

The third branch covers both the case where the item is in the company currency and the payment is not, and the case where the item and the payment are in two different foreign currencies: in both, the company-currency residual is the pivot.

### 12.6 The four totals

Given the batches:

1. Read the *next payment date* from the context: when the active domain of the calling screen contains a condition on the field `next_payment_date` whose value is a text, that value read as a date; otherwise none.
2. Build the union of every batch's items, sorted by document, then by due date (items with no due date last).
3. Group by document. For each document, read its installments (section 12.4) and dispatch each installment into one of four buckets:
   - type `early_payment_discount` → the **by-default** bucket; and a copy of it, with the residuals restored to the item's *undiscounted* residuals, into the **for-difference** bucket; latch the discount-applied flag;
   - a payment-term item of type `overdue` → the **common** bucket;
   - a payment-term item of type `before_date` → the **common** bucket, and latch the wizard's first mode to `before_date`;
   - a payment-term item of type `next` → the **full-amount-only** bucket when the previous installment of the same document was of type `next`, `overdue` or `before_date`; the **common** bucket when no previous installment was classified, latching the wizard's first mode to `next`;
   - anything else → the **common** bucket.
   The document's first classified mode also becomes the wizard's first mode when none has been latched yet. The rule that a `next` installment latches `next` even after an `overdue` on another document is deliberate: when several documents are paid together and any one of them offers a next installment, the whole wizard is in `next` mode.
4. Convert each bucket into the wizard currency (section 12.5), giving `common`, `by_default`, `for_difference` and `full_only`.
5. Produce:
   ```formula
   amount_by_default         = | common + by_default |
   full_amount               = | common + by_default + full_only |
   amount_for_difference     = | common + for_difference |
   full_amount_for_difference= | common + for_difference + full_only |
   ```
6. Also produce the set of journal items behind the common and by-default buckets: those are the items that will actually be paid.

### 12.7 The proposed amount

```formula
amount = the previous amount            when there is no journal, no currency, no payment date,
                                          or the user has typed a custom amount
       = amount_by_default              otherwise
```

### 12.8 The installment mode

```formula
installments_mode = 'full'                       when compare( currency , amount , full_amount ) = 0
                  = the wizard's first mode      when compare( currency , amount , amount_by_default ) = 0
                  = 'full'                       otherwise
```

### 12.9 The switch offered under the amount

| Mode | Switch amount | Text |
|---|---|---|
| `full` and the full amount already equals the default amount and the typed amount | 0.00 | nothing is shown |
| `full` otherwise, with a non-zero amount | `amount_by_default` | "This is the full amount." then, when a discount applies, "Consider paying the amount with **early payment discount** instead." and otherwise "Consider paying in **installments** instead." |
| `overdue` | `full_amount` | "This is the overdue amount." then "Consider paying the **full amount**." |
| `before_date` | `full_amount` | "Total for the installments before <the next payment date, or today>." then "Consider paying the **full amount**." |
| `next` | `full_amount` | "This is the next unreconciled installment." then "Consider paying the **full amount**." |

The bold fragments are the clickable part. When the user has typed a custom amount, no switch text is shown at all.

### 12.10 Discount mode

```formula
early_payment_discount_mode = discount_applied
                          AND ( compare( currency , amount , amount_by_default ) = 0
                                OR compare( currency , amount , full_amount ) = 0 )
```

When the mode is on, the difference handling is forced to `reconcile` and the write-off section is hidden: the difference is not a write-off, it is the discount itself, and it is booked by the discount rule of section 12.12.

### 12.11 The payment difference

```formula
payment_difference = amount_for_difference       − amount     when the mode is 'overdue', 'next' or 'before_date'
                   = full_amount_for_difference  − amount     when the mode is 'full'
                   = amount_for_difference       − amount     otherwise
                   = 0.00                                     when there is no payment date
```

The *for-difference* totals are used rather than the default ones precisely so that, in discount mode, the difference equals the discount granted: the default total is discounted, the for-difference total is not.

The difference section is shown when the difference is non-zero, the wizard is not in discount mode, the wizard is editable, grouping is either unavailable or switched on, and the chosen payment method line has an outstanding account.

### 12.12 What the difference becomes

Three cases, decided in this order.

**Case A — the difference is kept open** (`payment_difference_handling` is `open`). No write-off line is produced. The reconciliation matches as much as it can and the document keeps a residual.

**Case B — discount mode.** For every journal item of the batch whose document is eligible for the discount at the payment date, record the pair (item, matched foreign amount = minus the item's foreign residual, matched balance = that amount converted to the company currency at the payment date). Then:

```formula
open_amount_currency = payment_difference × ( −1 when the direction is outbound , +1 when inbound )
open_balance         = convert( open_amount_currency , from the payment currency to the company currency ,
                                for the company , at the payment date )
```

and produce the discount counterpart lines (section 12.13) from those pairs and that open balance. All of them become write-off lines of the Payment.

**Case C — the difference is written off** (`payment_difference_handling` is `reconcile`, not in discount mode, difference not zero).

- **C1 — the chosen difference account is one of the company's two exchange-difference accounts** (`writeoff_is_exchange_account` is true). No write-off line is produced. Instead:
  - when the payment currency is **not** the company currency, the Payment is created with a **forced balance** equal to `Σ amount_residual over the batch's items`, so that the liquidity side is booked at exactly the company-currency amount the documents carry and the whole difference lands on the exchange side;
  - when the payment currency **is** the company currency, a **forced rate** is passed to the reconciliation instead:
    ```formula
    forced_rate = | ( Σ amount_residual_currency over the batch's items ) ÷ amount |   when amount ≠ 0
                = 0                                                                     otherwise
    ```
    The reconciliation then uses that rate in place of the rate table (step 1 of section 3.1), which pushes the whole difference into an exchange-difference entry.
- **C2 — any other account.**
  ```formula
  write_off_amount_currency =   payment_difference     when the direction is inbound
                            = − payment_difference     when the direction is outbound

  write_off_balance = convert( write_off_amount_currency ,
                               from the payment currency to the company currency ,
                               for the company , at the payment date )
  ```
  One write-off line is produced: label = the wizard's write-off label (default `Write-Off`), account = the chosen difference account, counterparty = the wizard's counterparty, currency = the payment currency.

### 12.13 The discount counterpart lines

Produced per document, then summed across documents by identical grouping key.

For one document:

1. Let `discount_percentage` be the payment term's discount percentage. If it is zero, produce nothing.
2. Choose the discount account:
   ```formula
   discount_account = the company's cash-discount LOSS account   when the document is an INBOUND document
                    = the company's cash-discount GAIN account   otherwise
   ```
   An **inbound** document is one the company expects money for: a customer invoice, a customer receipt or a
   vendor credit note. An **outbound** document is one the company expects to pay: a vendor bill, a vendor
   receipt or a customer credit note. Granting a discount on money owed to the company is a loss; obtaining a
   discount on money the company owes is a gain.
3. Determine an analytic distribution for the discount from the analytic distribution models, keyed on the discount account's code, the company, the commercial counterparty and the counterparty's tags.
4. Rebuild the document's product lines with their unit price multiplied by `(100 − discount_percentage) ÷ 100`, treating them as refund lines, and recompute their taxes (fixed-amount taxes are excluded from the recomputation).
5. For each rebuilt line, compute the **base delta**:
   ```formula
   base_delta_amount_currency = round_to( document currency ,
                                          direction_sign × discounted_total_excluded_in_currency
                                          − the original line's foreign amount )
   base_delta_balance         = round_to( company currency ,
                                          direction_sign × discounted_total_excluded
                                          − the original line's balance )
   ```
   and accumulate it under a grouping key made of the counterparty, the currency, the discount account and the analytic distribution — plus, when the payment term's discount computation mode is `included` **and** the document's product lines carry taxes, the line's taxes and its tax grids.
6. Compute the fraction of the document actually being settled:
   ```formula
   percentage_paid = | the payment-term item's foreign residual ÷ the document total |
   ```
7. When tax lines are needed (mode `included` with taxes), compute the tax lines of the rebuilt base lines, subtract from each the tax amount the document already carries for the mirrored repartition line, and keep the delta. Each such delta becomes a counterpart line labelled "Early Payment Discount (<the tax name>)" with
   ```formula
   amount_currency = round_to( document currency , delta_amount_currency × percentage_paid )
   balance         = round_to( company currency  , delta_balance         × percentage_paid )
   ```
8. Each accumulated base delta becomes a counterpart line labelled "Early Payment Discount" with the same two roundings applied to the base delta.
9. **Fix the rounding.** Let
   ```formula
   term_amount_currency = payment_term_item.amount_currency − payment_term_item.discount_amount_currency
   term_balance         = payment_term_item.balance         − payment_term_item.discount_balance

   delta_amount_currency = term_amount_currency − Σ base line amounts − Σ tax line amounts
   delta_balance         = term_balance         − Σ base line balances − Σ tax line balances
   ```
   and add both deltas to the base line with the largest foreign amount. This guarantees that the counterpart lines sum exactly to the discount granted.

Across documents, lines sharing a grouping key are summed. The running open balance starts at the `open_balance` of case B and is reduced by every line's balance. If anything is left:

```formula
exchange_sign = compare( company currency , remaining open balance , 0 )

if exchange_sign > 0 : account = the company's EXPENSE exchange-difference account
if exchange_sign < 0 : account = the company's INCOME  exchange-difference account
```

and one further counterpart line is produced on that account, labelled "Early Payment Discount (Exchange Difference)", with a foreign amount of zero and a balance equal to the remaining open balance.

### 12.14 Worked example — a payment taking a two percent early discount

Customer invoice, no taxes, everything in the company currency.

| Fact | Value |
|---|---|
| Invoice total | 1 000.00 |
| Payment term | 2 % if paid within 7 days, otherwise 30 days |
| Payment-term item | balance +1 000.00, foreign amount +1 000.00, discount balance 980.00, discount foreign amount 980.00, discount date = invoice date + 7 days |
| Payment date | 3 days after the invoice date, so within the discount window |

**Installments.** The document is eligible, so the single installment is of type `early_payment_discount` with amounts 980.00 and a recorded discount of `1000.00 − 980.00 = 20.00`.

**Totals.**

```formula
common  = 0.00
by_default          = 980.00
for_difference      = 1000.00     (the same installment with the undiscounted residual restored)
full_only           = 0.00

amount_by_default           = | 0.00 +  980.00 |            =  980.00
full_amount                 = | 0.00 +  980.00 + 0.00 |     =  980.00
amount_for_difference       = | 0.00 + 1000.00 |            = 1000.00
full_amount_for_difference  = | 0.00 + 1000.00 + 0.00 |     = 1000.00
discount_applied            = true
```

**The proposed amount** is `amount_by_default = 980.00`.

**The mode.** `compare(currency, 980.00, full_amount = 980.00) = 0` → the mode is `full`.

**Discount mode.** `discount_applied` is true and `compare(currency, 980.00, amount_by_default = 980.00) = 0`, so the wizard is in discount mode. The difference handling is forced to `reconcile` and the write-off section is hidden.

**The difference.** The mode is `full`, so

```formula
payment_difference = full_amount_for_difference − amount = 1000.00 − 980.00 = 20.00
```

**The discount counterpart lines.** One eligible item, the payment-term item:

```formula
matched foreign amount = − ( +1000.00 ) = −1000.00
matched balance        = −1000.00

open_amount_currency = 20.00 × ( +1 , inbound ) = 20.00
open_balance         = 20.00
```

Per document: `discount_percentage = 2`. The discount account is the **cash-discount loss** account, because a customer invoice is an inbound document. The single product line of 1 000.00 is rebuilt at `1000.00 × (100 − 2) ÷ 100 = 980.00`.

```formula
direction_sign of a customer invoice = −1
the original product line's balance  = −1000.00

base_delta_balance = round_to( company , (−1) × 980.00 − (−1000.00) ) = round_to( company , 20.00 ) = 20.00
percentage_paid    = | 1000.00 ÷ 1000.00 | = 1.00
base line amount   = round_to( company , 20.00 × 1.00 ) = 20.00
```

No taxes, so no tax lines. The rounding fix: `term_balance = 1000.00 − 980.00 = 20.00`, `delta_balance = 20.00 − 20.00 = 0.00`, so nothing is added.

The remaining open balance is `20.00 − 20.00 = 0.00`, so no exchange line is produced.

**The resulting Payment.** Amount 980.00, one write-off line of +20.00 on the cash-discount loss account, labelled "Early Payment Discount". Its journal entry (section 14) is:

| Item | Account | Debit | Credit |
|---|---|---|---|
| liquidity | outstanding receipts | 980.00 | |
| write-off | cash discount loss | 20.00 | |
| counterpart | customer receivable | | 1 000.00 |

The counterpart of 1 000.00 exactly matches the invoice's receivable item, so the invoice becomes fully paid while only 980.00 of money moved.

**The taxed variant.** Had the invoice been 1 000.00 net plus 21 % tax, total 1 210.00, with a discount computation mode of `included`, the 2 % would apply to both the base and the tax: the discount is `1000.00 × 2 % = 20.00` on the base and `210.00 × 2 % = 4.20` on the tax, that is 24.20 in total; the amount proposed would be `1210.00 − 24.20 = 1185.80`, and the counterpart lines would be one base line of 20.00 on the cash-discount loss account carrying the tax grids of the original line, plus one tax line of 4.20 labelled "Early Payment Discount (<the tax name>)" on the tax account. With a computation mode of `excluded` no tax line is produced and the whole 24.20 sits on the discount account without affecting the tax report.

### 12.15 Worked example — a write-off of three hundredths

Customer invoice of 100.00 in the company currency. The customer transfers 99.97 and the company decides to close the invoice.

| Step | Value |
|---|---|
| `amount_by_default` and `full_amount` | 100.00 |
| The user overwrites the amount with | 99.97 |
| The custom-amount detector | 99.97 differs from all four totals, so the typed value is remembered as a custom amount and the automatic computation stops overwriting it |
| `installments_mode` | 99.97 equals neither the full amount nor the default amount, so the mode is `full` |
| `payment_difference` | `full_amount_for_difference − amount = 100.00 − 99.97 = 0.03` |
| The user sets | difference handling = `reconcile`, difference account = a rounding-difference expense account, label = `Write-Off` |
| `writeoff_is_exchange_account` | false (the account is not an exchange account) |

Case C2 applies:

```formula
write_off_amount_currency = + 0.03        (inbound)
write_off_balance         = + 0.03
```

The Payment is created with amount 99.97 and one write-off line. Its journal entry:

| Item | Account | Debit | Credit |
|---|---|---|---|
| liquidity | outstanding receipts | 99.97 | |
| write-off | rounding difference (expense) | 0.03 | |
| counterpart | customer receivable | | 100.00 |

The counterpart of 100.00 settles the invoice exactly. Had the difference handling been `open`, the entry would have been two lines of 99.97 and the invoice would have kept a residual of 0.03 with payment state `partial`.

For an **outbound** payment the sign flips: paying 99.97 against a bill of 100.00 gives `write_off_amount_currency = −0.03`, that is a credit of 0.03 on the difference account, and the counterpart is a debit of 100.00 on the payable account.

### 12.16 Fixing the balance when the payment currency differs

After the Payments are created but before they are posted, and only when the wizard is editable, each Payment whose currency differs from the currency of the items it will settle is checked:

1. Split the Payment's entry into liquidity, counterpart and write-off items.
2. ```formula
   source_balance  = | Σ amount_residual over the items to reconcile |
   payment_rate    = liquidity item's foreign amount ÷ liquidity item's balance   when that balance ≠ 0, else 0
   source_converted= | source_balance | × payment_rate
   payment_amount_currency = | Σ amount_currency over the counterpart items |
   ```
3. If `source_converted` and `payment_amount_currency` differ by more than zero in the payment currency, the user is not trying to fully settle the items: leave the Payment alone.
4. Otherwise compute `delta = source_balance − | Σ balance over the counterpart items |`. If it is zero in the company currency, leave the Payment alone.
5. Otherwise add `delta` to the debit of the first debit item among the liquidity and counterpart items, and the same `delta` to the credit of the first credit item. This makes the Payment's company-currency amount match the documents exactly.

**Why.** Suppose a currency **B** whose rate against the company currency **A** is 100 to 1. Paying 12.15 A with 0.12 B gives a computed balance of 12.00 A, not 12.15 A, and the 0.15 A difference would never be matched. The correction adds 0.15 A to both sides so the settlement is exact.

---

## 13. The amounts of a Payment's journal entry

### 13.1 Splitting an existing payment entry

For a Payment with an entry, its items are split into three groups:

1. The **valid liquidity accounts** are: the journal's default account, the payment account of the Payment's own method line, the payment accounts of every inbound method line of the journal, the payment accounts of every outbound method line of the journal, and the Payment's outstanding account.
2. An item whose account is one of those is a **liquidity** item.
3. Otherwise, an item whose account type is `asset_receivable` or `liability_payable`, or whose account is the company's inter-bank transfer account, is a **counterpart** item.
4. Every other item is a **write-off** item.
5. **Repair rule.** If exactly one write-off item was found, and either group is empty, that single item is moved into the empty group — first into liquidity if liquidity is empty, then into counterpart if counterpart is empty. This keeps a Payment usable after its journal's outstanding account has been changed.

### 13.2 Preparing the items of a new payment entry

Preconditions: an outstanding account must resolve, otherwise:

> You can't create a new payment without an outstanding payments/receipts account set either on the company or the <the payment method name> payment method in the <the journal display name> journal.

1. Build the default label by concatenating the parts of the label list: the payment method line's name, or the text "No Payment Method" when there is none; then, when the Payment has a memo, the separator ": " and the memo.
2. Collect the supplied write-off lines, if any, and total them:
   ```formula
   write_off_amount_currency = Σ amount_currency over the supplied write-off lines
   write_off_balance         = Σ balance         over the supplied write-off lines
   ```
3. Collect the withholding lines from the withholding hook (empty in the core capability) and total them the same way.
4. If both withholding lines and write-off lines are present, the write-off lines are dropped and their totals reset to zero: the two mechanisms are not combined, because the withholding capability already passes its lines as write-off lines through the synchronisation path.
5. The liquidity amount in the payment currency:
   ```formula
   liquidity_amount_currency =   amount     when the direction is inbound
                             = − amount     when the direction is outbound
                             =   0          otherwise
   ```
6. The liquidity balance:
   ```formula
   if no write-off lines were supplied AND a forced balance was supplied:
       sign             = +1 when liquidity_amount_currency > 0 , else −1
       liquidity_balance = sign × | forced balance |
   else:
       liquidity_balance = convert( liquidity_amount_currency ,
                                    from the payment currency to the company currency ,
                                    for the company , at the payment date )
   ```
7. Subtract the withholding totals from both liquidity amounts:
   ```formula
   liquidity_amount_currency ← liquidity_amount_currency − withholding_amount_currency
   liquidity_balance         ← liquidity_balance         − withholding_balance
   ```
8. The counterpart amounts:
   ```formula
   counterpart_amount_currency = − liquidity_amount_currency − write_off_amount_currency − withholding_amount_currency
   counterpart_balance         = − liquidity_balance         − write_off_balance         − withholding_balance
   ```
9. Emit the items in this order: liquidity, counterpart, write-off, withholding.

Note that step 8 subtracts the withholding total **twice** in effect — once in step 7 from the liquidity side and once here — which is what makes the counterpart carry the gross amount while the liquidity carries the net.

### 13.3 The company-currency signed amount

```formula
amount_company_currency_signed = Σ balance over the liquidity items            when an entry exists
                               = convert( amount_signed ,
                                          from the payment currency to the company currency ,
                                          for the company , at the payment date )   otherwise
```

---

## 14. International bank account number validation

### 14.1 Normalisation

```formula
normalised = the account number with every character that is neither a letter nor a digit removed
sanitised  = normalised, upper-cased
```

The stored sanitized number uses the same rule: every non-alphanumeric character removed, the rest upper-cased.

### 14.2 The country template

A table maps each supported two-letter country code to a template. A template is a text of the same length as a valid number for that country, written in groups of four separated by spaces for readability, in which each position is one of:

| Symbol | Meaning |
|---|---|
| the two literal country letters | the first two positions |
| `k` | a check digit of the international number itself |
| `B` | a position of the national bank code |
| `S` | a position of the branch code |
| `C` | a position of the account number |
| `K` | a position of a national check digit |
| `T` | a position of an account-type code (used by two countries) |
| `A` | a position of a balance-account number (used by one country) |
| `F` | a position of a fiscal identification number (used by one country) |
| `R` | a reserved zero (used by one country) |
| `M`, `N` | further country-specific positions |

The complete shipped table, with the template's length once the spaces are removed:

| Country code | Template | Length |
|---|---|---|
| `ad` | `ADkk BBBB SSSS CCCC CCCC CCCC` | 24 |
| `ae` | `AEkk BBBC CCCC CCCC CCCC CCC` | 23 |
| `al` | `ALkk BBBS SSSK CCCC CCCC CCCC CCCC` | 28 |
| `at` | `ATkk BBBB BCCC CCCC CCCC` | 20 |
| `az` | `AZkk BBBB CCCC CCCC CCCC CCCC CCCC` | 28 |
| `ba` | `BAkk BBBS SSCC CCCC CCKK` | 20 |
| `be` | `BEkk BBBC CCCC CCKK` | 16 |
| `bg` | `BGkk BBBB SSSS TTCC CCCC CC` | 22 |
| `bh` | `BHkk BBBB CCCC CCCC CCCC CC` | 22 |
| `br` | `BRkk BBBB BBBB SSSS SCCC CCCC CCCT N` | 29 |
| `by` | `BYkk BBBB AAAA CCCC CCCC CCCC CCCC` | 28 |
| `ch` | `CHkk BBBB BCCC CCCC CCCC C` | 21 |
| `cr` | `CRkk BBBC CCCC CCCC CCCC CC` | 22 |
| `cy` | `CYkk BBBS SSSS CCCC CCCC CCCC CCCC` | 28 |
| `cz` | `CZkk BBBB SSSS SSCC CCCC CCCC` | 24 |
| `de` | `DEkk BBBB BBBB CCCC CCCC CC` | 22 |
| `dk` | `DKkk BBBB CCCC CCCC CC` | 18 |
| `do` | `DOkk BBBB CCCC CCCC CCCC CCCC CCCC` | 28 |
| `ee` | `EEkk BBSS CCCC CCCC CCCK` | 20 |
| `es` | `ESkk BBBB SSSS KKCC CCCC CCCC` | 24 |
| `fi` | `FIkk BBBB BBCC CCCC CK` | 18 |
| `fo` | `FOkk CCCC CCCC CCCC CC` | 18 |
| `fr` | `FRkk BBBB BSSS SSCC CCCC CCCC CKK` | 27 |
| `gb` | `GBkk BBBB SSSS SSCC CCCC CC` | 22 |
| `ge` | `GEkk BBCC CCCC CCCC CCCC CC` | 22 |
| `gi` | `GIkk BBBB CCCC CCCC CCCC CCC` | 23 |
| `gl` | `GLkk BBBB CCCC CCCC CC` | 18 |
| `gr` | `GRkk BBBS SSSC CCCC CCCC CCCC CCC` | 27 |
| `gt` | `GTkk BBBB MMTT CCCC CCCC CCCC CCCC` | 28 |
| `hr` | `HRkk BBBB BBBC CCCC CCCC C` | 21 |
| `hu` | `HUkk BBBS SSSC CCCC CCCC CCCC CCCC` | 28 |
| `ie` | `IEkk BBBB SSSS SSCC CCCC CC` | 22 |
| `il` | `ILkk BBBS SSCC CCCC CCCC CCC` | 23 |
| `is` | `FSkk BBBB SSCC CCCC FFFF FFFF FF` | 26 |
| `it` | `ITkk KBBB BBSS SSSC CCCC CCCC CCC` | 27 |
| `jo` | `JOkk BBBB SSSS CCCC CCCC CCCC CCCC CC` | 30 |
| `kw` | `KWkk BBBB CCCC CCCC CCCC CCCC CCCC CC` | 30 |
| `kz` | `KZkk BBBC CCCC CCCC CCCC` | 20 |
| `lb` | `LBkk BBBB CCCC CCCC CCCC CCCC CCCC` | 28 |
| `li` | `LIkk BBBB BCCC CCCC CCCC C` | 21 |
| `lt` | `LTkk BBBB BCCC CCCC CCCC` | 20 |
| `lu` | `LUkk BBBC CCCC CCCC CCCC` | 20 |
| `lv` | `LVkk BBBB CCCC CCCC CCCC C` | 21 |
| `mc` | `MCkk BBBB BSSS SSCC CCCC CCCC CKK` | 27 |
| `md` | `MDkk BBCC CCCC CCCC CCCC CCCC` | 24 |
| `me` | `MEkk BBBC CCCC CCCC CCCC KK` | 22 |
| `mk` | `MKkk BBBC CCCC CCCC CKK` | 19 |
| `mr` | `MRkk BBBB BSSS SSCC CCCC CCCC CKK` | 27 |
| `mt` | `MTkk BBBB SSSS SCCC CCCC CCCC CCCC CCC` | 31 |
| `mu` | `MUkk BBBB BBSS CCCC CCCC CCCC CCCC CC` | 30 |
| `nl` | `NLkk BBBB CCCC CCCC CC` | 18 |
| `no` | `NOkk BBBB CCCC CCK` | 15 |
| `om` | `OMkk BBBC CCCC CCCC CCCC CCC` | 23 |
| `pk` | `PKkk BBBB CCCC CCCC CCCC CCCC` | 24 |
| `pl` | `PLkk BBBS SSSK CCCC CCCC CCCC CCCC` | 28 |
| `ps` | `PSkk BBBB CCCC CCCC CCCC CCCC CCCC C` | 29 |
| `pt` | `PTkk BBBB SSSS CCCC CCCC CCCK K` | 25 |
| `qa` | `QAkk BBBB CCCC CCCC CCCC CCCC CCCC C` | 29 |
| `ro` | `ROkk BBBB CCCC CCCC CCCC CCCC` | 24 |
| `rs` | `RSkk BBBC CCCC CCCC CCCC KK` | 22 |
| `sa` | `SAkk BBCC CCCC CCCC CCCC CCCC` | 24 |
| `se` | `SEkk BBBB CCCC CCCC CCCC CCCC` | 24 |
| `si` | `SIkk BBSS SCCC CCCC CKK` | 19 |
| `sk` | `SKkk BBBB SSSS SSCC CCCC CCCC` | 24 |
| `sm` | `SMkk KBBB BBSS SSSC CCCC CCCC CCC` | 27 |
| `tn` | `TNkk BBSS SCCC CCCC CCCC CCCC` | 24 |
| `tr` | `TRkk BBBB BRCC CCCC CCCC CCCC CC` | 26 |
| `ua` | `UAkk BBBB BBCC CCCC CCCC CCCC CCCC C` | 29 |
| `vg` | `VGkk BBBB CCCC CCCC CCCC CCCC` | 24 |
| `xk` | `XKkk BBBB CCCC CCCC CCCC` | 20 |

### 14.3 The validation, written as arithmetic

Let `N` be the sanitized number.

**Step 1 — not empty.** If `N` is empty: "There is no IBAN code."

**Step 2 — known country.** Let `cc` be the first two characters of `N`, lower-cased. If `cc` is not a key of the template table: "The IBAN is invalid, it should begin with the country code"

**Step 3 — length and alphabet.** Let `T` be the template for `cc` with its spaces removed. If the length of `N` differs from the length of `T`, or `N` contains any character that is not a letter or a digit:

> The international bank account number does not seem to be correct. You should have entered something like this <the template with its spaces>
> Where B = National bank code, S = Branch code, C = Account No, k = Check digit

**Step 4 — rotate.** Move the first four characters of `N` to the end:

```formula
R = N[5..len(N)] ‖ N[1..4]
```

(positions are one-based and inclusive).

**Step 5 — expand to digits.** Replace each character of `R` by its base-thirty-six value written in decimal, and concatenate:

```formula
value( '0' ) = 0 , value( '1' ) = 1 , … , value( '9' ) = 9
value( 'A' ) = 10 , value( 'B' ) = 11 , … , value( 'Z' ) = 35

D = decimal_text( value( R[1] ) ) ‖ decimal_text( value( R[2] ) ) ‖ … ‖ decimal_text( value( R[last] ) )
```

A digit contributes one character, a letter contributes two.

**Step 6 — the modulo test.** Read `D` as an integer and test:

```formula
D mod 97 = 1
```

If it is not 1: "This IBAN does not pass the validation check, please verify it."

Because `D` is far too large for machine integers, the remainder is computed by a left-to-right fold:

```formula
r ← 0
for each character c of D, from left to right:
    r ← ( r × 10 + digit_value( c ) ) mod 97
the result is r
```

### 14.4 Worked example — checking `BE62 5100 0754 7061`

**Step 1.** Sanitized: `BE62510007547061`. Not empty.

**Step 2.** Country code `be` is in the table.

**Step 3.** The template is `BEkk BBBC CCCC CCKK`; with the spaces removed it is `BEkkBBBCCCCCCCKK`, of length 16. The sanitized number also has 16 characters, and all of them are letters or digits. The check passes.

**Step 4 — rotate.**

```formula
N = B E 6 2 5 1 0 0 0 7 5 4 7 0 6 1
R = 5 1 0 0 0 7 5 4 7 0 6 1 B E 6 2
```

**Step 5 — expand.** Every character of `R` except `B` and `E` is a digit and contributes itself. `B` has base-thirty-six value 11 and `E` has value 14.

```formula
D = 5 1 0 0 0 7 5 4 7 0 6 1 | 11 | 14 | 6 2
  = 510007547061111462
```

**Step 6 — the fold.**

| character | running remainder |
|---|---|
| 5 | (0 × 10 + 5) mod 97 = 5 |
| 1 | (5 × 10 + 1) mod 97 = 51 |
| 0 | (51 × 10 + 0) mod 97 = 510 mod 97 = 25 |
| 0 | (25 × 10 + 0) mod 97 = 250 mod 97 = 56 |
| 0 | (56 × 10 + 0) mod 97 = 560 mod 97 = 75 |
| 7 | (75 × 10 + 7) mod 97 = 757 mod 97 = 78 |
| 5 | (78 × 10 + 5) mod 97 = 785 mod 97 = 9 |
| 4 | (9 × 10 + 4) mod 97 = 94 |
| 7 | (94 × 10 + 7) mod 97 = 947 mod 97 = 74 |
| 0 | (74 × 10 + 0) mod 97 = 740 mod 97 = 61 |
| 6 | (61 × 10 + 6) mod 97 = 616 mod 97 = 34 |
| 1 | (34 × 10 + 1) mod 97 = 341 mod 97 = 50 |
| 1 | (50 × 10 + 1) mod 97 = 501 mod 97 = 16 |
| 1 | (16 × 10 + 1) mod 97 = 161 mod 97 = 64 |
| 1 | (64 × 10 + 1) mod 97 = 641 mod 97 = 59 |
| 4 | (59 × 10 + 4) mod 97 = 594 mod 97 = 12 |
| 6 | (12 × 10 + 6) mod 97 = 126 mod 97 = 29 |
| 2 | (29 × 10 + 2) mod 97 = 292 mod 97 = 1 |

The remainder is **1**, so the number is valid.

**A one-digit change.** Replace the last digit by 2, giving `BE62 5100 0754 7062`. The rotated text becomes `510007547062BE62`, the expanded text `510007547062111462`, and the fold gives, from the 61 obtained after the tenth character: 6 → 616 mod 97 = 34; 2 → 342 mod 97 = 51; 1 → 511 mod 97 = 26; 1 → 261 mod 97 = 67; 1 → 671 mod 97 = 89; 4 → 894 mod 97 = 21; 6 → 216 mod 97 = 22; 2 → 222 mod 97 = 28. The remainder is 28, not 1, so the number is rejected with "This IBAN does not pass the validation check, please verify it."

### 14.5 Formatting and the derived parts

**Pretty form.** A valid number is rewritten as groups of four characters separated by single spaces, the last group being shorter when the length is not a multiple of four. An invalid number is left exactly as typed. Both creating and modifying a Bank Account apply this rewriting: when the value validates, it is stored normalised then prettified; when it does not, it is stored as typed.

**Basic bank account number.** The sanitized number with its first four characters removed. Asking for it on an account whose type is not the international form raises: "Cannot compute the BBAN because the account number is not an IBAN."

**Any named part.** Given a part name — bank, branch, account, check, national check, account type, balance account, fiscal code or reserved — the corresponding template symbol is `B`, `S`, `C`, `k`, `K`, `T`, `A`, `F` or `R`. Then:

```formula
strip the two country characters from both the number and the template
walk the two texts position by position
concatenate the number's characters whose template character equals the symbol
```

An unknown part name, or a country with no template, yields nothing.

**Worked example.** For `IT60X0542811101000000123456` with the template `ITkk KBBB BBSS SSSC CCCC CCCC CCC`: the part `bank` gives `05428`, and the part `account` gives `000000123456`.

**Type inference.** The account type is `iban` when the validation of section 14.3 succeeds, and `bank` otherwise. The type is recomputed whenever the account number changes, and a constraint re-runs the validation on every account whose inferred type is the international one — so an account that once validated cannot be silently corrupted.

---

## 15. Structured payment references

A *structured payment reference* is a payment communication that carries its own check digits, so the receiving bank can verify that the payer typed it correctly. Seven schemes are recognised.

### 15.1 Sanitisation

Applied before every check.

1. Remove every whitespace character.
2. If what remains matches the pattern `three plus signs or three asterisks or nothing`, then three digits, a slash, four digits, a slash, five digits, then the same three plus signs, three asterisks or nothing as at the start — remove every plus sign, asterisk and slash.
3. Otherwise leave the text as it is.

Examples: ` RF18 1234 5678 9  ` becomes `RF18123456789`; `+++020/3430/57642+++` becomes `020343057642`; `***020/3430/57642***` becomes `020343057642`.

### 15.2 Two shared algorithms

**The trailing check-digit algorithm** (used by two schemes):

```formula
luhn_valid( digits ):
    total ← 0
    walk the digits from right to left, numbering them 1, 2, 3, …
    for each digit d at position p:
        if p is even:
            x ← d × 2
            if x > 9 : x ← x − 9
        else:
            x ← d
        total ← total + x
    return ( total mod 10 = 0 )
```

**The modulo ninety-seven check-digit algorithm** (used by the international scheme):

```formula
expand( text ):
    replace each character by its base-thirty-six value written in decimal,
    concatenating the results ( '0'..'9' → 0..9 , 'A'..'Z' → 10..35 )

remainder97( text ) = expand( text ) read as an integer, modulo 97
                      ( computed by the left-to-right fold of section 14.3 )

check_digits_for( text ) = 98 − remainder97( text ‖ "00" ) , written on two digits
```

### 15.3 The seven checks

Every check takes the sanitized reference.

**1. Belgian.** The reference must be exactly twelve digits: ten digits `a` followed by two digits `b`.

```formula
valid ⟺ ( a mod 97 ) = ( b mod 97 )
```

Note that `b` is compared *after* its own reduction modulo 97, which makes the check digits `97` and `00` interchangeable, as the national scheme requires.

**Worked example.** `020343057642`: `a = 0203430576`, `b = 42`. `0203430576 mod 97`: the fold gives 42. `42 mod 97 = 42`. Equal, so the reference is valid.

**2. Danish.** The reference must match: an optional leading plus sign; then either `71<` followed by exactly fifteen digits, or `75<` followed by exactly sixteen digits; then a plus sign; then exactly eight digits; then `<`. The captured digit block (fifteen or sixteen digits) must satisfy the trailing check-digit algorithm.

**Worked example.** `+71<022646321691221+88655702<` — the block is `022646321691221`, fifteen digits. Doubling every second digit from the right: 1, 2×2=4, 2, 1×2=2, 9, 6×2=12→3, 1, 2×2=4, 3, 6×2=12→3, 4, 6×2=12→3, 2, 2×2=4, 0. Sum = 1+4+2+2+9+3+1+4+3+3+4+3+2+4+0 = 45. Hmm — 45 is not a multiple of ten, so this particular illustration would be rejected; the scheme requires the issuer to have appended a check digit that makes the total a multiple of ten.

**3. Finnish.** The reference must be all digits, at least two of them: a body `a` of one to nineteen digits followed by one check digit `c`.

```formula
total ← 0
walk the digits of a from RIGHT to LEFT, numbering them 0, 1, 2, …
for each digit d at index i:
    weight ← 7 when i mod 3 = 0
            ← 3 when i mod 3 = 1
            ← 1 when i mod 3 = 2
    total ← total + weight × d

expected ← ( 10 − ( total mod 10 ) ) mod 10
valid ⟺ expected = c
```

**Worked example.** Reference `1232`. Body `123`, check `2`. From the right: 3 with weight 7 → 21; 2 with weight 3 → 6; 1 with weight 1 → 1. Total 28. `expected = (10 − 8) mod 10 = 2`. Equal, valid.

**4. Norwegian and Swedish.** The reference must be all digits and must satisfy the trailing check-digit algorithm.

**5. Slovenian.** The reference must start with `SI01`. Strip those four characters. What is left must contain at most two hyphens and must match exactly three groups of digits separated by two hyphens. Let `core` be the three groups concatenated, which must be digits and at least two characters long. Split `core` into a body and a last digit `c`.

```formula
weights ← 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13   truncated to the length of the body
total   ← Σ over i of ( body read from RIGHT to LEFT )[i] × weights[i]
expected ← 11 − ( total mod 11 )
if expected is 10 or 11 : expected ← 0
valid ⟺ expected = c
```

**Worked example.** `SI0112-34567-8`. Stripping the prefix gives `12-34567-8`, three groups. `core = 12345678`, body `1234567`, check `8`. Reading the body from the right with the weights: 7×2=14, 6×3=18, 5×4=20, 4×5=20, 3×6=18, 2×7=14, 1×8=8. Total 112. `112 mod 11 = 2`, `expected = 11 − 2 = 9`, which is neither 10 nor 11. `9 ≠ 8`, so this reference is rejected; the correct check digit for that body is 9.

**6. Dutch.** Three permitted shapes.

```formula
if the reference is exactly 7 digits : valid
if the reference is not 9 to 16 digits : invalid
if the reference is exactly 15 digits : invalid

c    ← the first digit
body ← the remaining digits, left-padded with zeros to 16 characters, then reversed
weights ← 2, 4, 8, 5, 10, 9, 7, 3, 6, 1        (cycling)

total ← Σ over i of body[i] × weights[ i mod 10 ]

expected ← 11 − ( total mod 11 )
if expected is 11 : expected ← 0
if expected is 10 : expected ← 1
valid ⟺ expected = c
```

**7. International creditor reference.** The reference must begin with `RF`, be between five and twenty-five characters long, and contain only digits and capital letters after the prefix. Move the first four characters to the end and apply the modulo ninety-seven test:

```formula
valid ⟺ remainder97( reference[5..end] ‖ reference[1..4] ) = 1
```

**Generating one.** Given a base number `n`:

```formula
check   = check_digits_for( n ‖ "RF" )
result  = "RF" ‖ check ‖ " " ‖ ( the digits of n in groups of four, separated by single spaces )
```

**Worked example.** `n = 123456789`. `n ‖ "RF" = "123456789RF"`. Expanding, `R` is 27 and `F` is 15, giving `1234567892715`; appending `"00"` gives `123456789271500`. The fold produces the remainder 80, so `check = 98 − 80 = 18`. The result is `RF18 1234 5678 9`.

### 15.4 The combined test

```formula
is_structured( reference ):
    r ← sanitize( reference )
    if r is empty : return false
    return Belgian(r) OR Danish(r) OR Finnish(r) OR NorwegianSwedish(r)
           OR Slovenian(r) OR Dutch(r) OR International(r)
```

The order of the disjuncts is the order written above; because it is a disjunction, the first scheme that accepts the reference settles the matter.

### 15.5 The per-country test

```formula
is_structured_for_country( reference , country_code ):
    r ← sanitize( reference )
    if country_code is 'BE' : return Belgian(r)
    if country_code is 'FI' : return Finnish(r)
    if country_code is 'NO' : return NorwegianSwedish(r)
    if country_code is 'SE' : return NorwegianSwedish(r)
    if country_code is 'NL' : return Dutch(r)
    if country_code is 'SI' : return Slovenian(r)
    otherwise                : return International(r)
```

The country code is compared upper-cased. Note that Denmark has no per-country entry: a Danish reference is only recognised by the combined test.

---

## 16. Quick response code payment data

A payable quick response code encodes, in a single scannable image, everything the payer's banking application needs: the beneficiary, the account, the amount, the currency and the communication.

### 16.1 Choosing a generator

Generators register themselves with a code, a display name and a sequence. Two are shipped:

| Code | Name | Sequence |
|---|---|---|
| `sct_qr` | Single Euro Payments Area Credit Transfer quick response code | 20 |
| `emv_qr` | Merchant-presented quick response code | 30 |

The selection, given an amount, a free communication, a structured communication, a currency, the payer and optionally a forced generator:

1. If no bank account is given at all, produce nothing.
2. If no currency is given: "Currency must always be provided in order to generate a QR-code"
3. Build the candidate list: the single forced generator when one was named, otherwise every registered generator sorted by sequence ascending.
4. For each candidate in order:
   1. Ask for its **eligibility error** for this account, this payer and this currency. If there is one, this candidate cannot be used.
   2. Otherwise ask for its **data error** for this amount, currency, payer and communications. If there is one, this candidate cannot be used.
   3. If there is no error at all, this candidate wins; return the values (generator code, amount, currency, payer, free communication, structured communication).
   4. When the caller asked for errors not to be silenced and the candidate failed, raise: "The following error prevented '<the generator name>' QR-code to be generated though it was detected as eligible: " followed by the error text.
5. If no candidate won, produce nothing.

The result is then turned either into a rendering address — the barcode rendering route followed by the generation parameters as query arguments — or into an embedded image, by rendering the barcode and encoding it as an inline image address.

### 16.2 The Single Euro Payments Area credit transfer code

**Eligibility.** Every failing test contributes a line; the lines are joined by a carriage return and a line feed.

| Test | Message |
|---|---|
| The currency is not the euro | "Can't generate a SEPA QR Code with the <the currency name> currency." |
| The account type is not the international form | "Can't generate a SEPA QR code if the account type isn't IBAN." |
| The sanitized account number is empty, or its first two characters are not the code of a country of the Single Euro Payments Area that actually uses the international account form | "Can't generate a SEPA QR code with a non SEPA iban." |

The set of accepted country codes is: the codes of the countries of the Single Euro Payments Area zone, minus the codes `AX`, `NC`, `YT`, `TF`, `BL`, `RE`, `MF`, `GP`, `PM`, `PF`, `GF`, `MQ`, `JE`, `GG` and `IM`. Those are territories that belong to the zone but whose accounts carry another country's prefix.

**Data check.** When neither an account holder name nor a partner name is set: "The account receiving the payment must have an account holder name or partner name set."

**The payload.** Twelve fields joined by newline characters, in this exact order:

| # | Field | Value |
|---|---|---|
| 1 | Service tag | the literal `BCD` |
| 2 | Version | the literal `002` |
| 3 | Character set | the literal `1` |
| 4 | Identification code | the literal `SCT` |
| 5 | Beneficiary bank identifier code | the bank's code, or empty |
| 6 | Beneficiary name | the account holder name, or the partner's name, truncated to 71 characters |
| 7 | Beneficiary account number | the sanitized account number |
| 8 | Currency and amount | the currency's name immediately followed by the amount rounded to the currency and written with exactly the currency's number of decimal places |
| 9 | Purpose | empty |
| 10 | Structured remittance information | the sanitized structured communication when it passes the combined structured test of section 15.4, otherwise empty |
| 11 | Unstructured remittance information | empty when field 10 is filled; otherwise the free communication, truncated to 141 characters |
| 12 | Beneficiary-to-originator information | empty |

**Rendering parameters.** Barcode type: quick response; quiet zone 0; width 128; height 128; human-readable on; value: the twelve fields joined by newlines.

**Worked example.** Beneficiary *Deco Addict*, account `BE62 5100 0754 7061`, bank identifier code `GEBABEBB`, amount 1 234.50 euro, structured communication `+++020/3430/57642+++`.

The structured communication sanitizes to `020343057642`, which passes the Belgian check, so field 10 carries it and field 11 is empty.

```
BCD
002
1
SCT
GEBABEBB
Deco Addict
BE62510007547061
EUR1234.50

020343057642


```

(The last three lines are the empty purpose is on line 9, the empty unstructured information on line 11 and the empty beneficiary-to-originator information on line 12.)

### 16.3 The merchant-presented code

**Eligibility.** In the base capability the generator is never eligible: when there is no account, "A bank account is required for EMV QR Code generation."; otherwise "No EMV QR Code is available for the country of the account <the account number>." Country-specific capability packages override this and supply the merchant account information for their scheme.

**Data check**, in this order:

| Test | Message |
|---|---|
| No merchant account information | "Missing Merchant Account Information." |
| The partner has no city | "Missing Merchant City." |
| No proxy type | "Missing Proxy Type." |
| No proxy value | "Missing Proxy Value." |

**The field encoding.** Every field is written as a tag of two digits, then a length of two digits, then the value. A field whose value is empty or absent contributes nothing at all.

```formula
serialise( tag , value ) = ""                                            when value is empty or absent
                         = two_digits( tag ) ‖ two_digits( length( value ) ) ‖ value   otherwise
```

**The fields, in this exact order:**

| Tag | Field | Value |
|---|---|---|
| 00 | Payload format indicator | the literal `01` |
| 01 | Point of initiation | the literal `12`, meaning a dynamic code |
| (scheme) | Merchant account information | supplied by the country-specific scheme, together with its own tag |
| 52 | Merchant category code | the scheme's category code, `0000` by default |
| 53 | Transaction currency | the three-digit numeric code of the currency, taken from the table below |
| 54 | Transaction amount | absent when the amount is zero in its currency; written without a decimal part when the amount is a whole number; otherwise written as it is |
| 58 | Country code | the account's country code |
| 59 | Merchant name | the partner's name with accents removed, truncated to 25 characters, or the literal `NA` |
| 60 | Merchant city | the partner's city with accents removed, truncated to 15 characters, or empty |
| 62 | Additional data field | present only when the account is set to include the reference; supplied by the scheme from the cleaned communication |

Accent removal also maps the two characters `đ` and `Đ` to `d` and `D`. The communication used for the additional data field is the structured communication when there is one, otherwise the free communication; accents are removed and then every character outside the set {space, the letters A to Z in both cases, the digits, underscore, at sign, full stop, backslash, solidus, number sign, ampersand, plus sign, hyphen} is deleted.

**The checksum.** After all the fields, the literal `6304` is appended (tag 63, length 04), then the sixteen-bit cyclic redundancy check of everything written so far, as four upper-case hexadecimal digits.

```formula
crc16( bytes ):
    crc ← 0xFFFF
    for each byte b of the text encoded as eight-bit characters:
        crc ← crc XOR ( b shifted left by 8 )
        repeat 8 times:
            if crc AND 0x8000 is non-zero:
                crc ← ( crc shifted left by 1 ) XOR 0x1021
            else:
                crc ← crc shifted left by 1
    return crc AND 0xFFFF
```

The polynomial is `0x1021`, the initial value is `0xFFFF`, the input is not reflected and the output is neither reflected nor complemented.

**The currency table.** The numeric code of a currency, by its name:

| Name | Code | Name | Code | Name | Code | Name | Code |
|---|---|---|---|---|---|---|---|
| ARS | 032 | HUF | 348 | MXN | 484 | THB | 764 |
| AUD | 036 | ISK | 352 | NPR | 524 | AED | 784 |
| BHD | 048 | INR | 356 | NZD | 554 | TND | 788 |
| KHR | 116 | ILS | 376 | NOK | 578 | GBP | 826 |
| CAD | 124 | JPY | 392 | QAR | 634 | USD | 840 |
| LKR | 144 | KRW | 410 | RUB | 643 | TWD | 901 |
| CNY | 156 | KWD | 414 | SAR | 682 | RSD | 941 |
| HRK | 191 | MYR | 458 | SGD | 702 | RON | 946 |
| CZK | 203 | MUR | 480 | VND | 704 | TRY | 949 |
| DKK | 208 | | | ZAR | 710 | XOF | 952 |
| HKD | 344 | | | SEK | 752 | XPF | 953 |
| | | | | CHF | 756 | BGN | 975 |
| | | | | | | EUR | 978 |
| | | | | | | UAH | 980 |
| | | | | | | PLN | 985 |
| | | | | | | BRL | 986 |

A currency absent from this table cannot be encoded; the generator fails for it.

**Rendering parameters.** Barcode type: quick response; quiet zone 0; width 128; height 128; human-readable on; value: the assembled text.

**Worked example of the encoding.** Suppose a scheme supplies the merchant account information under tag 26 with the value `AAAABBBB`, the merchant is *Cafe Lumiere* in *Hanoi*, the country code is `VN`, the currency is the Vietnamese currency (numeric code 704) and the amount is 50 000.

```formula
serialise(  0 , "01"           ) = "000201"
serialise(  1 , "12"           ) = "010212"
serialise( 26 , "AAAABBBB"     ) = "26" ‖ "08" ‖ "AAAABBBB"  = "2608AAAABBBB"
serialise( 52 , "0000"         ) = "52040000"
serialise( 53 , "704"          ) = "5303704"
serialise( 54 , "50000"        ) = "540550000"
serialise( 58 , "VN"           ) = "5802VN"
serialise( 59 , "Cafe Lumiere" ) = "5912Cafe Lumiere"
serialise( 60 , "Hanoi"        ) = "6005Hanoi"
```

concatenated, then `6304`, then the four hexadecimal digits of the checksum of that whole text.

---

## 17. Liquidity journal dashboard figures

Each figure of a liquidity journal's dashboard card is defined here. All of them are restricted to the user's active companies.

### 17.1 Number to reconcile

Count the Bank Transactions of the journal whose company is active, whose reconciled flag is **not** true, whose entry is **checked**, and whose entry state is `posted`.

### 17.2 Number to check and amount to check

Group the Bank Transactions of the journal whose company is active, whose entry is **not** checked and whose entry state is `posted`; the figure is the count and the sum of the amounts, formatted in the journal's currency (or the company's currency when the journal has none).

### 17.3 Running balance of the journal

```formula
anchor_statement = the statement of the journal, among the active companies, that has a first-line index,
                   ordered by date descending then identifier descending, first one

unlinked = the Bank Transactions of the journal, among the active companies, that belong to NO statement,
           whose entry is not cancelled, and whose ordering index is greater than or equal to
           the anchor statement's first-line index (or to the empty string when there is no anchor)

current_statement_balance = ( the anchor statement's reported ending balance , or 0 )
                          + Σ amount over unlinked
```

The card also records whether the journal *has statement lines at all*: true when an anchor statement exists or at least one unlinked transaction was found.

### 17.4 Direct bank payments

Payments booked straight onto the journal's bank account rather than onto an outstanding account:

```formula
group the Payments whose matched flag is TRUE, whose entry state is 'posted',
whose journal is this journal, whose company is active, and whose outstanding account
IS the journal's default account,
by company, journal and entry currency

signed_amount = − amount   for an outbound payment
              = + amount   for an inbound payment

direct_payments_balance = Σ signed_amount, each group converted into the journal's currency
                          ( through the group's company-currency total when the currencies differ )
nb_direct_payments      = the number of groups contributing
```

### 17.5 Account balance shown on the card

```formula
account_balance = current_statement_balance + direct_payments_balance
```

### 17.6 Outstanding payments balance

```formula
group the Payments whose matched flag is NOT TRUE, whose entry state is 'posted',
whose journal is this journal and whose company is active,
by company, journal and payment currency

outstanding_pay_account_balance = Σ signed_amount, converted into the journal's currency
has_outstanding                 = whether any group contributed
```

### 17.7 Converting a group into the journal's currency

```formula
for each group:
    if the group's currency = the target currency:
        add the group's signed amount
    else:
        add convert( the group's company-currency total ,
                     from the group's company currency to the target currency ,
                     for the group's company , at the current date )
round the total to the target currency
```

### 17.8 Miscellaneous operations

Journal items posted on the journal's default account that belong neither to a Bank Transaction nor to a Payment, dated after the journal's last statement date (or after the company's fiscal-year lock date when there is no statement, or unrestricted when there is neither):

```formula
domain = company is active
     AND the item belongs to no Bank Transaction
     AND the item's entry state is 'posted'
     AND the item belongs to no Payment
     AND ( account = the journal's default account AND date > date_limit )      for each journal, combined with OR

nb_misc_operations       = the number of such items, grouped by account
misc_operations_balance  = the sum of their foreign amounts, grouped by account
```

The balance is only shown when every one of those items is in the journal's own currency; otherwise the count is shown in a warning colour and no balance is given.

### 17.9 Last statement and validity

The card also carries: the last statement of the journal (the one with the greatest date, then the greatest identifier), its reported ending balance, whether the journal has at least one statement, whether the last statement is visible (true when the company has no fiscal-year lock date, or when the last statement's date is after that lock date), whether the journal has invalid statements, and the journal's bank feed source.

### 17.10 The graph

For a liquidity journal the card shows a filled line graph of the balance over the last thirty days, keyed "Cash: Balance", "Bank: Balance" or "Credit Card: Balance" according to the journal type, and built **backwards** from today.

1. Read the daily totals: group the Bank Transactions of the journal whose entry date is later than thirty days ago and whose company is active, by entry date and journal, summing the amounts, ordered by date descending.
2. If there is no daily total **and** the journal has no statement lines at all, the graph is filled with sample data instead: six points, one every five days over the last thirty days, each with a random value between −5 and 15, and the series is keyed "Sample data" and flagged as sample.
3. Otherwise:
   1. Start the running amount at the journal's current running balance (section 17.3).
   2. If there is no daily total, or the most recent daily total is earlier than today, add a point for today at that running amount, so the curve always reaches at least today.
   3. Walk the daily totals from the most recent to the oldest. For each, prepend a point dated on that day at the current running amount, then subtract that day's total from the running amount. Prepending, rather than appending, is what makes the curve read forwards while being computed backwards.
   4. If the oldest date processed is not exactly thirty days ago, prepend one last point dated thirty days ago at the remaining running amount, so the curve always starts a month back.
4. Each point carries the value rounded to the journal's currency, a short date label and a long date label, both formatted in the user's language.

---

## 18. Selecting the journal, the methods and the recipient account

The register-payment screen and the Payment record share the same selection logic, expressed here once.

### 18.1 The journals available for a batch

```formula
journals = every Journal whose company is visible from the batch's company
           AND whose type is one of 'bank', 'cash', 'credit'

available = journals having at least one INBOUND  payment method line   when the batch's direction is inbound
          = journals having at least one OUTBOUND payment method line   otherwise
```

On the register-payment screen the available set is the **union** over every batch. On a Payment the set is built slightly differently: the journals considered are those whose company is an ancestor **or** a descendant of the active company, and the filter is the same.

### 18.2 The journal chosen for a batch

Let `company` be the batch's company with the fewest ancestors, `wanted_currency` the batch's currency and `wanted_account` the batch's recipient bank account.

```formula
base = Journals of `company`
       whose type is one of 'bank', 'cash', 'credit'
       AND that are in the screen's available set

if wanted_account is set:
    try, in this order:
        1. base AND currency = wanted_currency AND bank account = wanted_account
        2. base AND                                bank account = wanted_account
        3. base AND currency = wanted_currency
        4. base
else:
    try, in this order:
        1. base AND currency = wanted_currency
        2. base

return the first journal found, in the journal ordering; nothing when none matches
```

The order encodes the preference: a journal that matches both the currency and the account is best; matching the account alone beats matching the currency alone; and any liquidity journal is better than none.

### 18.3 When the chosen journal is overridden

On the register-payment screen the journal is recomputed whenever the available set changes, and only when the current journal is no longer in it. The recomputation, in order:

1. If exactly one preferred payment method line is named on the documents being paid, use that line's journal.
2. Otherwise, when the screen is editable, use the journal chosen for the single batch (section 18.2).
3. Otherwise, use the first liquidity journal of the screen's company that is in the available set.

### 18.4 The recipient bank accounts available for a batch

```formula
if the batch's direction is inbound:
    available = the journal's own bank account          (money is received on the company's account)
else:
    company   = the batch's company with the fewest ancestors
    available = the bank accounts of the batch's counterparties
                whose company is empty or equal to `company`
```

The account finally used is the batch's own recipient account when it is among the available ones, otherwise the first available one.

### 18.5 The payment method line chosen

On a Payment, in order:

1. the counterparty's default inbound line when the direction is inbound and that line is available;
2. the counterparty's default outbound line when the direction is outbound and that line is available;
3. the current line when it is still available;
4. the first available line;
5. none.

On the register-payment screen, in order:

1. keep the current line when it is still available;
2. otherwise, when exactly one preferred payment method line is named on the documents and it is available, use it;
3. otherwise the first available line;
4. otherwise none.

When a batch's direction differs from the direction of the line chosen on the screen, that batch uses the first available line of the journal for **its own** direction instead.

### 18.6 Whether grouping is offered

```formula
if there is exactly one batch:
    can_group = ( the batch has more than one journal item )
            AND NOT ( the batch's items all belong to ONE document AND that document is invoice-like )
else:
    lines_to_pay = the journal items behind the common and by-default buckets of section 12.6
    can_group    = any batch has more than one of its items inside lines_to_pay
```

And the switch's own default:

```formula
group_payment = ( every item of the single batch belongs to one document )   when the screen is editable
              = false                                                        otherwise
```

### 18.7 How many payments will be created, and how many are blocked

```formula
total = the number of batches                                   when grouping is on
      = the number of journal items to be paid                  when grouping is off

for each batch:
    account = the account chosen on the screen, when there is one,
              otherwise the batch's own chosen account (section 18.4)
    if a recipient account is required:
        if there is no account          : add the batch's counterparties to the missing list
        else if the account is untrusted:
            blocked ← blocked + 1                               when grouping is on
                    + the number of the batch's items to be paid when grouping is off
            add the account to the untrusted list
```

### 18.8 Edit mode

```formula
edit_mode = the screen is editable
        AND ( the first batch has exactly one journal item OR grouping is on )
```

In edit mode exactly one Payment is created, from the screen's own values. Outside it, Payments are created from the batch values, one per batch when grouping is on and one per document when it is off.

---

## 19. The memo of a payment

### 19.1 The rule

```formula
if the journal items belong to exactly ONE document:
    memo = that document's payment reference,
           or its own reference,
           or its number
else if ANY of the documents is an OUTBOUND document (a vendor bill, an outgoing credit note, an incoming receipt):
    memo = the distinct values of ( payment reference, or reference, or number )
           over every document, with the empty ones dropped, sorted, joined by ", "
else:
    memo = the next value of the company's group-payment sequence
```

### 19.2 Which items the rule reads

```formula
if the screen is editable AND the installment mode is 'full', OR the user typed a custom amount:
    read every selected journal item
else:
    read only the journal items behind the common and by-default buckets of section 12.6
```

So, when a user pays only the next installment of several documents, the memo names only the documents actually being paid.

### 19.3 Worked examples

| Selection | Memo |
|---|---|
| one customer invoice whose payment reference is `INV/2026/0007` | `INV/2026/0007` |
| one customer invoice with no payment reference and no reference, numbered `INV/2026/0007` | `INV/2026/0007` |
| two vendor bills referenced `BILL-2` and `BILL-1` | `BILL-1, BILL-2` |
| two customer invoices | the next group-payment number, for example `GROUP/2026/00001` |
| three customer invoices of which only the next installment of two is being paid | still the next group-payment number, because more than one document is being paid |
