# Accounting effects of the Inventory Valuation and Costing domain

Every journal entry this domain produces, listed line by line with its account selection
rule, its side, its amount formula, its currency handling, its date, its partner, its
analytic distribution, its tax handling and its reconciliation behaviour.

> **Reproduced text.** Quoted, bolded strings in this file are user-facing text the
> system emits character for character — error messages, selection labels, button
> labels, action titles. Where such a string contains an abbreviation it belongs to
> the emitted string, not to this specification's prose: "FIFO" stands for *first in
> first out*, "AVCO" for *average cost*, "WIP" for *work in progress*, "MOs" for
> *manufacturing orders*, "BoM" for *bill of materials*, `STJ` for the inventory
> valuation journal code and `LC/` for the landed cost sequence prefix.

Terms used throughout:

- **Journal Entry** (`account.move`, table `account_move`) — the accounting document.
- **Journal Item** (`account.move.line`, table `account_move_line`) — one line of it.
- **Inventory valuation account** — the account selected by the rule described in
  [section 0.2](#02-the-account-selection-rules), key `stock_valuation`.
- **Variation account** — the account attached to the inventory valuation account, key
  `stock_variation`.
- **Location valuation account** — the account carried by a location
  (`valuation_account_id`).
- **Display type `cogs`** — the marker put on every journal item this domain *injects*
  into an invoice or a bill; those items are dropped when the document is copied, deleted
  when it is reset to draft or cancelled, and never re-derived from the currency or from
  the product.

---

## 0. What decides whether anything is posted at all

### 0.1 The two switches

| Situation | Goods movements post? | Vendor bill debits inventory? | Cost recognised at the customer invoice? | Period variation posted? |
|---|---|---|---|---|
| Periodic valuation | No | No (the bill debits the expense account) | No | Yes, by the closing entry |
| Perpetual valuation, anglo-saxon accounting on | Yes, when a location valuation account is involved | Yes | Yes | Not by the closing (see below) |
| Perpetual valuation, anglo-saxon accounting off (continental) | Yes, when a location valuation account is involved | No (the bill debits the expense account) | No | Yes, by part three of the closing |

Two refinements:

- Under **perpetual** valuation a goods movement only posts when at least one of its two
  locations carries a **location valuation account**. A plain receipt from a vendor
  location into a warehouse, where neither location carries such an account, posts
  nothing by itself; the inventory asset is instead debited by the vendor bill. A
  delivery to a customer location likewise posts nothing by itself; the inventory asset
  is credited by the customer invoice's injected cost line.
- Movements through **inventory-loss**, **scrap** and **production** locations are the
  ones that normally carry a location valuation account, and they are therefore the ones
  that post goods-movement entries.

### 0.2 The account selection rules

Every rule below is evaluated **in the company of the record being posted**, and the
category is read with that company in scope.

| Key | Rule, in order |
|---|---|
| Inventory valuation (`stock_valuation`) | 1. the inventory valuation account of the product's category for that company; 2. the company-level fallback of that company-dependent field; 3. the company's inventory valuation account. The category tree is **not** walked. |
| Variation (`stock_variation`) | The variation account attached to whichever account the previous rule selected. |
| Expense (`expense`) | 1. the product's own expense account; 2. the first non-empty expense account found by walking the category tree upwards from the product's category; 3. the default expense account of the product's owning company, or of the current company when the product has none. |
| Income (`income`) | Same shape as the expense rule, with the income accounts. |
| Inventory journal (`stock_journal`) | 1. the inventory journal of the product's category for that company; 2. the company-level fallback of that company-dependent field; 3. the company's inventory journal. |
| Production (`production`) | When the product has a category: the production account of that category for that company, **even when empty** — no further fallback. When it has none: the company-level fallback, but only under perpetual valuation; otherwise empty. |
| Price difference | The price difference account of the product's **category**, mapped through the document's fiscal position. Only consulted under the standard costing method. |
| Location valuation | The valuation account carried by the location itself. No fallback. |

Whenever the caller asks for the *mapped* accounts, every selected account is passed
through the fiscal position of the document before use.

---

## 1. Goods movement valuation entry

**Produced by.** The completion of one or more goods movements; also the creation of a
movement in an already-completed state; also the application of an inventory adjustment
(which is a goods movement).

**Condition, per movement.** All of the following must hold, or the movement contributes
nothing:

1. The product is storable.
2. The movement is valued — that is, its incoming flag or its outgoing flag is set.
3. At least one of its two locations carries a location valuation account.
4. Its quantity is not zero at the precision of its unit of measure.
5. The product's valuation mode is `real_time`.

**One entry per batch.** All the qualifying movements of one completion are folded into a
**single** journal entry, not one per movement.

**Journal.** The **company's inventory journal**. (Note: the per-category inventory
journal is *not* consulted for this entry; it is consulted by the manufacturing labour
entry and by the default of a landed cost document.)

**Date.** The forced accounting period date from the context when one is in effect
(supplied by an inventory adjustment with an accounting date, or by a stock quantity
record carrying one), otherwise **today in the user's time zone**.

**Reference.** The distinct references of the movements of the batch, joined by a comma
and a space. If the joined text is longer than 43 characters, it is cut to its first 40
characters followed by three dots.

**Partner.** The accounting partner of the partner of the transfer, when the batch has
one; otherwise empty.

**Lines, per qualifying movement.** Two items, each carrying:

- the product of the movement;
- the label **"_the movement reference_ - _the product name_"**;
- the analytic distribution derived by the generic analytic distribution machinery from
  the product, the partner and the company (this domain sets none explicitly on the
  items);
- no taxes.

The account selection and the sides depend on **which end of the movement carries a
location valuation account**:

| Case | Debited account | Credited account |
|---|---|---|
| The **source** location carries a valuation account | the product's inventory valuation account | the **source** location's valuation account |
| Otherwise (the destination carries one) | the **destination** location's valuation account | the product's inventory valuation account |

The amount of both items is:

```formula
amount = move_value
```

expressed in the company currency, with no currency conversion (the movement's value is
already in the company currency). When the subcontracting accounting integration is
installed, the amount is reduced for a component movement feeding a completed
subcontracting movement whose product does not use standard price:

```formula
amount = move_value − production_extra_unit_cost
                     × convert( move_quantity → product_reference_unit )
```

**Posting.** The entry is created with elevated privileges and posted immediately. Each
contributing movement stores the entry's identifier in its journal entry reference.

**Reconciliation.** None. These items are not reconciled.

### 1.1 Worked example: an inventory adjustment that reduces stock

A product uses average cost with a unit cost of 10.00 and perpetual valuation. The
inventory-loss location of the company carries a loss account. A count reduces the
quantity on hand by 3 units, producing an outgoing movement from the warehouse to the
inventory-loss location with a value of 3 × 10.00 = 30.00.

The source location (the warehouse) carries **no** valuation account; the destination
(inventory loss) does. The second case applies:

| Account | Debit | Credit |
|---|---|---|
| Inventory loss (the location's valuation account) | 30.00 | |
| Inventory valuation (the product's account) | | 30.00 |

### 1.2 Worked example: an inventory adjustment that increases stock

The same product, counted 5 units higher. The movement goes **from** the inventory-loss
location **into** the warehouse, and is valued at 5 × 10.00 = 50.00. Now the **source**
location carries a valuation account, so the first case applies:

| Account | Debit | Credit |
|---|---|---|
| Inventory valuation (the product's account) | 50.00 | |
| Inventory loss (the location's valuation account) | | 50.00 |

### 1.3 Worked example: consuming a component into production

A component worth 78.00 leaves the warehouse for the production location, which carries a
cost-of-production account. The destination carries the account:

| Account | Debit | Credit |
|---|---|---|
| Cost of production (the production location's account) | 78.00 | |
| Inventory valuation of the component | | 78.00 |

When the finished good comes back out of the production location into the warehouse with
a value of 478.00, the **source** carries the account:

| Account | Debit | Credit |
|---|---|---|
| Inventory valuation of the finished good | 478.00 | |
| Cost of production | | 478.00 |

The cost-of-production account is left holding 478.00 − 78.00 − (the other components) −
(the labour posted separately) — that is, zero once every component and the labour entry
have gone through it.

---

## 2. Cost of goods sold at the customer invoice

**Produced by.** Posting a customer invoice or a customer credit note, unless the posting
is being made to cancel another entry.

**Condition, per invoice line.** All of the following:

1. The document is a sales document (customer invoice, customer credit note, or customer
   receipt).
2. The line is eligible for stock accounting: its product is storable **and** none of the
   goods movements reachable from the line is a drop shipment.
3. The product's valuation mode is `real_time`.
4. An inventory valuation account can be selected for the product under the document's
   fiscal position, **and** a counterpart can be selected: the product's expense account,
   falling back to the journal's default account. If either is empty, the line is
   skipped.
5. The computed amount in the document currency is not zero, **and** the computed unit
   price is not zero at the Product Price precision.

**Sign.**

```formula
sign = −1   when the document is a customer credit note
sign = +1   otherwise
```

**Amount.**

```formula
unit_price = the cost-of-goods-sold value of the line
             ( see calculations.md, section 9.2 )

amount_currency = sign × convert( line_quantity → product_reference_unit ) × unit_price
```

**The two items.** Both carry: the first 64 characters of the line label (or the empty
string when the line has none); the document; the commercial partner of the document; the
product; the line's unit of measure; the line's quantity; the line's analytic
distribution; no taxes; display type `cogs`; and a reference back to the originating
invoice line.

| Item | Account | Unit price recorded | Amount in currency |
|---|---|---|---|
| Inventory side | the product's **inventory valuation account** (mapped through the fiscal position) | `unit_price` | `− amount_currency` |
| Expense side | the product's **expense account**, else the journal's default account | `− unit_price` | `+ amount_currency` |

On a customer **invoice** the amount in currency is positive, so the inventory side is a
**credit** and the expense side is a **debit** — the goods leave the asset and become an
expense. On a customer **credit note** the sign flips and the two swap.

**Currency.** The amount is expressed in the document currency through the amount-in-
currency field; the company-currency balance is derived by the generic mechanism at the
document's rate. Note that the unit price fed in is a **company-currency** cost: the
conversion is left to the generic amount-in-currency handling.

**Reconciliation.** None.

**Removal.** Resetting the document to draft, or cancelling it, deletes both items.
Copying the document drops them.

### 2.1 Worked example

A product using first in first out cost 9.00 per unit and is sold for 10.00. One unit is
delivered and invoiced. The invoice's own items are:

| Account | Debit | Credit |
|---|---|---|
| Accounts receivable | 10.00 | |
| Product sales (income) | | 10.00 |

Posting injects:

| Account | Debit | Credit |
|---|---|---|
| Cost of goods sold (the product's expense account) | 9.00 | |
| Inventory valuation | | 9.00 |

### 2.2 Worked example: a credit note reversing an invoice

The invoice above is reversed. Because the product uses first in first out, the shortcut
that reuses the original unit price does **not** apply (it applies only to standard price
and average cost), so the cost is recomputed. The sign is −1, so the amount in currency is
−9.00 and the two items swap:

| Account | Debit | Credit |
|---|---|---|
| Inventory valuation | 9.00 | |
| Cost of goods sold | | 9.00 |

For a product using **standard price** or **average cost**, the credit note instead
reuses the unit price recorded on the original document's inventory-side item, so the
reversal is exact even if the cost has moved since.

### 2.3 Negative-number (storno) presentation

When the company keeps its books with negative amounts instead of side reversals, a
customer credit note presents the injected items as negative amounts on the *same* sides
as the invoice would have used: the inventory side shows a credit of −9.00 and the
expense side a debit of −9.00.

---

## 3. Vendor bill line routed to the inventory asset

**Produced by.** The account computation of a purchase-document line — not a separate
entry, but a redirection of the line's own account.

**Condition.** The document is a purchase document; the line is eligible for stock
accounting; the product's valuation mode is `real_time`; and an inventory valuation
account can be selected for the product under the document's fiscal position.

**Effect.** The line's account becomes the **inventory valuation account** instead of the
expense account.

Under periodic valuation, or for a product that is not storable, or for a drop-shipped
line, the line keeps the expense account.

### 3.1 Worked example

A storable product with perpetual valuation is billed for 100.00.

| Account | Debit | Credit |
|---|---|---|
| Inventory valuation | 100.00 | |
| Accounts payable | | 100.00 |

The same product under periodic valuation:

| Account | Debit | Credit |
|---|---|---|
| Expenses | 100.00 | |
| Accounts payable | | 100.00 |

---

## 4. Price difference at the vendor bill

**Produced by.** Posting a vendor bill, vendor credit note or vendor receipt, unless the
posting is being made to cancel another entry.

**Condition, per line.** All of the following:

1. The document type is vendor bill, vendor credit note or vendor receipt.
2. The **company uses anglo-saxon accounting**.
3. The line is eligible for stock accounting.
4. The product's costing method is `standard`. (Products using first in first out or
   average cost absorb the difference into the value of the goods instead.)
5. A price difference account can be selected — the category's price difference account,
   mapped through the document's fiscal position. When it is empty, the line is skipped.
6. The subtotal difference is not zero for the document currency, **and** the line's
   stored unit price and its computed unit price agree at the Product Price precision.

**The two items.** Both carry: the first 64 characters of the line label; the document;
the line's partner or the document's commercial partner; the product; the line's unit of
measure; the relevant quantity (the line quantity); the line's analytic distribution; no
taxes; display type `cogs`.

| Item | Account | Balance in company currency |
|---|---|---|
| Difference | the **price difference account** | `convert( relevant_quantity × price_unit_difference, document_currency → company_currency, at today's date )` |
| Reversal | **the account the bill line itself uses** (the inventory valuation account under perpetual valuation) | `convert( relevant_quantity × ( − price_unit_difference ), document_currency → company_currency, at today's date )` |

The difference formula is in
[calculations.md](calculations.md#10-price-difference-at-the-vendor-bill-under-standard-price).
Note the conversion date: **today**, not the bill date.

**Reconciliation.** None.

**Removal.** Resetting to draft or cancelling deletes both items; copying drops them.

### 4.1 Worked example

Standard cost 9.00, bill price 10.00, quantity 10, no discount, no taxes, company
currency. The bill's own items debit the inventory valuation account 100.00 and credit
accounts payable 100.00. Posting injects:

| Account | Debit | Credit |
|---|---|---|
| Price difference | 10.00 | |
| Inventory valuation | | 10.00 |

Net effect: inventory asset +90.00, price difference +10.00, payable −100.00.

---

## 5. Landed cost entry

**Produced by.** Validating a landed cost document.

**Condition.** At least one adjustment line qualifies. An adjustment line qualifies when
**all** of the following hold:

1. It names a goods movement.
2. The movement's product has the `real_time` valuation mode. (Products under periodic
   valuation are skipped: their landed cost still changes the value of the goods, but no
   entry is produced — the closing picks the change up.)
3. The movement's **remaining quantity** at validation time is not zero.

**Journal.** The **document's** journal (`account_journal_id`), whose default is the
company's landed cost journal, falling back to the company-level fallback of the category
inventory journal.

**Date.** The document's date.

**Reference.** The document's name (its sequence number).

**Type.** A plain journal entry.

**Lines, per qualifying adjustment line.** Two items, each carrying the adjustment line's
description as the label, the product of the adjustment line, and a quantity of **zero**.

| Item | Account selection | Side |
|---|---|---|
| Inventory side | the **inventory valuation account** of the adjustment line's product (unmapped) | debited when the amount is positive, credited when negative |
| Counterpart | the **account of the cost line**, else the **expense account** of the cost line's product | credited when the amount is positive, debited when negative |

If the counterpart is still empty, the validation is refused with **"Please configure
Stock Expense Account for product: _the cost product name_."**

If the cost line has **no product at all**, the adjustment line produces nothing.

**Amount.**

```formula
amount = additional_landed_cost × ( move_remaining_quantity ÷ adjustment_line_quantity )
```

expressed in the company currency.

**Posting.** The entry is created only when at least one item was produced; the document
then stores its identifier and the entry is posted. Afterwards, **every movement named by
the document's adjustment lines is re-valued**, so that the landed cost enters the
movement's value through the extra source.

**Reconciliation.** None.

### 5.1 Worked example: one hundred split by quantity across two products, fully in stock

A landed cost of 100.00 is split by quantity across a receipt of 30 units of product A
and 20 units of product B, both using average cost with perpetual valuation. Both
receipts are entirely still on hand. The cost line's account is the freight expense
account.

Shares: A gets 60.00, B gets 40.00 (see
[calculations.md](calculations.md#73-worked-example-one-hundred-split-by-quantity-across-two-products)).

```formula
posted_amount( A ) = 60.00 × ( 30 ÷ 30 ) = 60.00
posted_amount( B ) = 40.00 × ( 20 ÷ 20 ) = 40.00
```

| Account | Debit | Credit |
|---|---|---|
| Inventory valuation of product A | 60.00 | |
| Freight expense | | 60.00 |
| Inventory valuation of product B | 40.00 | |
| Freight expense | | 40.00 |

(When both products share the same inventory valuation account and the same counterpart,
the four items are still produced as four items; they are not merged.)

### 5.2 Worked example: partly consumed

The same landed cost, but by validation time only 18 of product A's 30 units and none of
product B's 20 units are still on hand.

```formula
posted_amount( A ) = 60.00 × ( 18 ÷ 30 ) = 36.00
posted_amount( B ) = 40.00 × ( 0 ÷ 20 )  = 0.00   → no items produced for B
```

| Account | Debit | Credit |
|---|---|---|
| Inventory valuation of product A | 36.00 | |
| Freight expense | | 36.00 |

The remaining 24.00 of product A's share and the whole 40.00 of product B's share are not
posted at all — the goods they relate to have already left, and their cost was recognised
at the price that was known when they left.

### 5.3 Worked example: a negative landed cost

A landed cost line of −50.00 reverses an earlier freight allocation and is split entirely
onto one movement that is fully in stock.

```formula
posted_amount = −50.00 × ( 20 ÷ 20 ) = −50.00   → negative, so the sides swap
```

| Account | Debit | Credit |
|---|---|---|
| Freight expense | 50.00 | |
| Inventory valuation | | 50.00 |

---

## 6. The inventory valuation closing entry

**Produced by.** The closing operation on a company, invoked manually from the inventory
valuation report or by the daily scheduled job.

**Journal.** The **company's inventory journal**. If it is empty, the operation is refused
with **"Please set the Journal for Inventory Valuation in the settings."**

**Date.** The report date when one was given, otherwise today.

**Reference.** **"Stock Closing"**.

**Company.** The company being closed.

**Posting.** Posted immediately only when the caller asked for automatic posting (the
scheduled job always does; the manual action offers the entry in draft). The entry's
identifier is appended to the company's stored closing list before posting.

**Reconciliation.** None.

The entry is built from three parts, each producing balanced pairs of items through the
same rule: a negative balance swaps the two accounts and uses the absolute value.

### 6.1 Part one — location reclassification

One balanced pair per (location valuation account, inventory valuation account) pair
whose balance is not exactly zero.

| Item | Account | Side |
|---|---|---|
| 1 | the **inventory valuation account** of the category (falling back to the company's) | credited by the balance |
| 2 | the **location valuation account** | debited by the balance |

Label: **"Closing: Location Reclassification - [_the location account display name_]"**.
No product reference. No analytic distribution. No taxes.

```formula
balance = Σ value of the periodic-valuation movements that went out into that location
          − Σ value of the periodic-valuation movements that came back from it
```

restricted to movements dated after the last closing instant and not after the report
date. See
[calculations.md](calculations.md#112-part-one-location-reclassification).

### 6.2 Part two — the global stock variation

One balanced pair per inventory valuation account whose balance is not zero for the
company currency.

| Item | Account | Side |
|---|---|---|
| 1 | the **variation account** of that account, falling back to the company's default expense account | credited by the balance |
| 2 | the **inventory valuation account** | debited by the balance |

If neither a variation account nor a company default expense account can be found, the
account is skipped entirely and no pair is produced for it.

Label: **"Closing: Stock Variation Global for company [_the company display name_]"**.

```formula
balance = physical_value( account ) − posted_ledger_value( account ) − extra_balance( account )
```

where the extra balance is the net debit already proposed by part one on that account.

### 6.3 Part three — the continental perpetual period variation

One balanced pair per inventory valuation account that carries **both** a variation
account and a closing expense account, and whose period balance is not zero for the
company currency.

| Item | Account | Side |
|---|---|---|
| 1 | the **variation account** | credited by the amount |
| 2 | the **closing expense account** | debited by the amount |

Label: **"Closing: Stock Variation Over Period"**.

```formula
amount = ( posted_ledger_value_today( account ) − extra_balance( account ) )
         − posted_ledger_value_at_fiscal_year_start( account )
         + Σ posted balance already on the variation account up to the report date
```

See
[calculations.md](calculations.md#114-part-three-the-continental-perpetual-period-variation).

### 6.4 Worked example of a complete closing

A company using periodic valuation, with one inventory valuation account and its
variation account, and an inventory-loss location carrying a loss account. Since the last
closing: goods worth 30.00 went out through the inventory-loss location, and the physical
value of the goods on hand is 250.00 while the ledger balance of the inventory valuation
account is 0.00.

**Part one** produces:

| Account | Debit | Credit |
|---|---|---|
| Inventory loss | 30.00 | |
| Inventory valuation | | 30.00 |

**Part two** then computes, with an extra balance of −30.00 on the inventory valuation
account (it was credited 30.00 by part one):

```formula
balance = 250.00 − 0.00 − ( −30.00 ) = 280.00
```

| Account | Debit | Credit |
|---|---|---|
| Inventory valuation | 280.00 | |
| Inventory variation | | 280.00 |

**Part three** produces nothing, because the inventory valuation account carries no
closing expense account.

The complete entry:

| Account | Debit | Credit |
|---|---|---|
| Inventory loss | 30.00 | |
| Inventory valuation | | 30.00 |
| Inventory valuation | 280.00 | |
| Inventory variation | | 280.00 |

Net effect on the inventory valuation account: +250.00, exactly the physical value.

---

## 7. Manufacturing labour entry

**Produced by.** The completion of a manufacturing order, after its inventory has been
posted, for each order that reached the completed state.

**Condition.** All of the following:

1. The finished product's valuation mode, read in the order's company, is `real_time`.
2. The **production location** of the product carries a location valuation account.
3. No time record of the order's work orders is already linked to a journal item (that
   is, the labour has not already been posted).
4. The total work-centre cost is not zero for the company currency.

**Journal.** The **product's inventory journal** — the category's journal for the
company, else the company-level fallback, else the company's.

**Date.** Today in the user's time zone.

**Reference and label.** **"_the manufacturing order name_ - Labour"**.

**Type.** A plain journal entry.

**Lines.** One item per distinct expense account, plus one balancing item on the
production location's account.

| Item | Account selection | Amount |
|---|---|---|
| per expense account | the **work centre's expense account**, falling back to the finished product's **expense account** | debited by the sum, over the work orders using that account, of `round_to_currency( work_order_cost )` |
| balancing | the **production location's valuation account** | credited by the total work-centre cost |

Formally, a map from account to amount is built by adding each work order's rounded cost
to its account, and then subtracting the total from the production location's account;
every entry of that map becomes one journal item whose balance is the **negation** of the
mapped amount. Because the mapped amounts are positive for the expense accounts and
negative for the production account, the expense accounts end up **credited** by their
amount and the production account **debited** by the total.

> Read that carefully: the sign convention is the reverse of the intuitive one. The work
> centre expense accounts are **credited** and the production location's account is
> **debited**, because the labour is being capitalised **into** the goods rather than
> expensed.

**Posting.** Posted immediately. Every item except the last is then written back onto the
time records of the work orders that used that account, so that the labour is not posted
twice.

### 7.1 Worked example

A manufacturing order runs two work orders. The first uses a work centre with its own
expense account and costs 120.00; the second uses a work centre with no expense account
and costs 80.00, so it falls back to the finished product's expense account. The
production location carries a cost-of-production account.

| Account | Debit | Credit |
|---|---|---|
| Cost of production (production location) | 200.00 | |
| Work centre expense | | 120.00 |
| Product expense | | 80.00 |

The finished good then leaves the production location carrying that 200.00 inside its
value, which credits the cost-of-production account by the same amount when the
finished-goods movement posts its own entry.

---

## 8. Work in progress entry and its reversal

**Produced by.** Confirming the work-in-progress accounting wizard.

**Preconditions and failures.**

1. The sum of the credits must equal the sum of the debits, compared in the company
   currency; otherwise the operation is refused with **"Please make sure the total credit
   amount equals the total debit amount."**
2. The reversal date must be **after** the posting date; otherwise the operation is
   refused with **"Reversal date must be after the posting date."**

**Journal.** The wizard's journal, whose default is the company-level fallback of the
category inventory journal.

**Date.** The wizard's date.

**Reference.** The wizard's reference, whose default is **"Manufacturing work in progress - _the list
of order names_"**, or **"Manufacturing work in progress - Manual Entry"** when no order qualifies.

**Type.** A plain journal entry, carrying the selected manufacturing orders in its
work-in-progress order collection.

**Lines.** One item per wizard line, with the line's label, account, debit and credit.
The default lines are:

| Item | Account selection | Amount |
|---|---|---|
| **"WIP - Component Value"** | the company-level fallback of the category **inventory valuation account** | credited by the component value |
| **"WIP - Overhead"** | the company's **production work-in-progress overhead account**, falling back to the company-level fallback of the category **production account** | credited by the overhead value |
| **"Manufacturing WIP - _the list of order names_"** | the company's **production work-in-progress account** | debited by the sum of the two |

```formula
component_value = Σ over the picked component movement lines of the selected orders
                  whose quantity is non-zero and whose date is not after the cut-off, of
                  line_quantity_in_reference_unit
                  × ( lot_cost when the product is valuated by lot and the line carries a lot,
                      else product_unit_cost )

overhead_value  = Σ work-order cost of the selected orders up to the cut-off
```

The cut-off is the wizard's date widened to the **last instant of that day**, or, when no
date is set, the current instant with the time set to 23:59:59.

**Posting and reversal.** The entry is created with elevated privileges and posted. A
reversal is then produced and posted, dated at the reversal date, referenced
**"Reversal of: _the original reference_"**, and carrying the same manufacturing orders.

**Reconciliation.** The reversal is created by the generic reversal mechanism, which
reconciles the reversal against the original where the accounts allow it.

### 8.1 Worked example

Two manufacturing orders are in progress. Components already consumed are worth 1 200.00
at their unit costs; work orders have recorded 300.00 of labour.

| Account | Debit | Credit |
|---|---|---|
| Production work in progress | 1 500.00 | |
| Inventory valuation (company fallback) | | 1 200.00 |
| Production work-in-progress overhead | | 300.00 |

and, the following day:

| Account | Debit | Credit |
|---|---|---|
| Inventory valuation (company fallback) | 1 200.00 | |
| Production work-in-progress overhead | 300.00 | |
| Production work in progress | | 1 500.00 |

---

## 9. What this domain deliberately does **not** post

| Event | Why nothing is posted |
|---|---|
| A change of the unit cost of a product | The change creates a valuation history record and moves the value of the goods on hand, but no entry. Under periodic valuation the next closing picks the change up; under perpetual valuation the difference likewise surfaces at the closing, because no movement occurred. |
| A change of the costing method of a category | Same reasoning; the recomputation goes through the "disable automatic revaluation" path and produces neither a history record nor an entry. |
| A manual correction of the value of a movement | The movement's value changes, and with it the product's total value; the entry already posted for that movement, if any, is **not** amended. The difference appears at the next closing. |
| An internal transfer between two locations inside the valued perimeter | The movement is neither incoming nor outgoing, so it has no value and no entry. |
| A movement of goods owned by a third party (consignment) | The lines are excluded for valuation, so the movement is neither incoming nor outgoing. |
| A drop shipment | The movement is flagged as a drop shipment rather than incoming or outgoing, so it produces no valuation entry; the corresponding bill and invoice lines are also excluded from the stock-accounting mechanisms because the line eligibility test rejects any line whose movements include a drop shipment. |
| A landed cost on a product under periodic valuation | The adjustment line is skipped when the entry is built; the value of the goods still changes and the closing reports it. |
| A landed cost whose movement has nothing left in stock | No items are produced for that line. |
| A goods movement under periodic valuation | By definition: periodic valuation posts only at the closing. |
| A goods movement under perpetual valuation where neither location carries a valuation account | The counterpart is supplied instead by the vendor bill (on the way in) or by the injected cost-of-goods-sold line on the customer invoice (on the way out). |

---

## 10. Summary table of every entry

| # | Entry | Trigger | Journal | Debit | Credit | Amount |
|---|---|---|---|---|---|---|
| 1a | Goods movement, source carries the account | movement completes under perpetual valuation | company inventory journal | product inventory valuation | source location valuation | movement value |
| 1b | Goods movement, destination carries the account | movement completes under perpetual valuation | company inventory journal | destination location valuation | product inventory valuation | movement value |
| 2 | Cost of goods sold, customer invoice | invoice posts under perpetual valuation | the invoice's own journal | product expense (or journal default) | product inventory valuation | quantity × cost-of-goods-sold unit price |
| 2r | Cost of goods sold, customer credit note | credit note posts | the credit note's own journal | product inventory valuation | product expense | same, sign reversed |
| 3 | Vendor bill routed to the asset | bill line's account computation under perpetual valuation | the bill's own journal | product inventory valuation | accounts payable | line amount |
| 4 | Price difference | vendor bill posts, anglo-saxon, standard price | the bill's own journal | price difference account | the bill line's account | quantity × (bill unit price − standard cost), converted at today's rate |
| 5 | Landed cost | document validated | document journal | product inventory valuation | cost line account (or cost product expense) | allocated amount × remaining quantity ÷ line quantity |
| 6a | Closing, location reclassification | closing | company inventory journal | location valuation | product inventory valuation | net value that left through that location |
| 6b | Closing, global variation | closing | company inventory journal | product inventory valuation | variation account (or company expense) | physical value − ledger value − already proposed |
| 6c | Closing, period variation (continental perpetual) | closing | company inventory journal | closing expense account | variation account | ledger movement over the fiscal year plus the existing variation balance |
| 7 | Manufacturing labour | manufacturing order completes under perpetual valuation | product inventory journal | production location valuation | work centre expense, else product expense | rounded work-order cost per account |
| 8 | Work in progress | wizard confirmed | wizard journal | production work-in-progress account | inventory valuation fallback and overhead account | component value and overhead value |
| 8r | Work in progress reversal | automatically, with the above | wizard journal | the reverse of entry 8 | | same |

Every amount above is in the **company currency**; only entries 2 and 4 involve a
currency conversion, and their conversion rules are stated in their own sections.
