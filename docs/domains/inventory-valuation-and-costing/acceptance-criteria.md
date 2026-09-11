# Acceptance criteria for the Inventory Valuation and Costing domain

Numbered Given / When / Then scenarios with concrete numbers. These are the conformance
suite: an implementation that satisfies all of them behaves equivalently to the system
described in this folder.

> **Reproduced text.** Quoted, bolded strings in this file are user-facing text the
> system emits character for character — error messages, selection labels, button
> labels, action titles. Where such a string contains an abbreviation it belongs to
> the emitted string, not to this specification's prose: "FIFO" stands for *first in
> first out*, "AVCO" for *average cost*, "WIP" for *work in progress*, "MOs" for
> *manufacturing orders*, "BoM" for *bill of materials*, `STJ` for the inventory
> valuation journal code and `LC/` for the landed cost sequence prefix.

Unless a scenario says otherwise, the following baseline applies:

- One company, currency with two decimal places, no foreign currency.
- Product Price decimal precision 2; Product Unit decimal precision 2.
- A warehouse whose stock location has usage `internal` and belongs to the company.
- A vendor location and a customer location, both outside the valued perimeter.
- An inventory-loss location of usage `inventory` belonging to the company.
- Products are storable, their reference unit of measure is the unit, and their movement
  unit of measure is the unit.
- No consignment owner on any movement line.
- Every transfer is validated with every line picked.

Each scenario names its costing method and its valuation mode in the **Given**.

---

## Group A — Average cost

### A1. Average cost after receipts of ten at ten and ten at twelve, then a delivery of five

**Given** a product using the **average cost** costing method and the **periodic**
valuation mode, with nothing on hand and a unit cost of 10.00,

**When** the following happen in order:

1. a receipt of 10 units valued at 10.00 each is validated;
2. a receipt of 10 units valued at 12.00 each is validated;
3. a delivery of 5 units is validated;

**Then**:

| Step | Movement value | Quantity on hand | Product unit cost | Product total value |
|---|---|---|---|---|
| after 1 | 100.00 (incoming) | 10 | 10.00 | 100.00 |
| after 2 | 120.00 (incoming) | 20 | **11.00** | 220.00 |
| after 3 | **55.00** (outgoing) | 15 | **11.00** | **165.00** |

**And** the unit cost after step 2 is computed by the incremental fast path:

```formula
previous_quantity = 20 − 10 = 10
new_unit_cost     = ( 10 × 10.00 + 120.00 ) ÷ 20 = 11.00
```

**And** the delivery is valued at 5 × 11.00 = 55.00, using the unit cost **as it stood
before** the delivery.

**And** the delivery does **not** change the unit cost: it is still 11.00 afterwards.

**And** no journal entry has been produced by any of the three steps, because the
valuation mode is periodic.

### A2. The same sequence under perpetual valuation with no location valuation account

**Given** the same product but with the **perpetual** valuation mode, and neither the
warehouse location nor the vendor location nor the customer location carrying a valuation
account,

**When** the three steps of scenario A1 are performed,

**Then** the values, quantities and unit costs are identical to A1,

**And** still **no** journal entry is produced by the goods movements, because the
valuation-entry test requires at least one of the two locations to carry a valuation
account.

### A3. The closing entry after scenario A1

**Given** the state at the end of scenario A1 (total value 165.00), the periodic
valuation mode, the inventory valuation account with a variation account attached, no
previous closing, and a posted ledger balance of 0.00 on the inventory valuation account,

**When** the closing is run with automatic posting,

**Then** a journal entry is created in the company's inventory journal, dated today,
referenced **"Stock Closing"**, containing exactly:

| Account | Debit | Credit |
|---|---|---|
| Inventory valuation | 165.00 | |
| Inventory variation | | 165.00 |

**And** the entry's identifier is appended to the company's closing register.

### A4. A second closing after a further delivery

**Given** the state at the end of scenario A3, with the closing entry posted,

**When** a delivery of 5 more units is validated (valued at 5 × 11.00 = 55.00, leaving 10
units worth 110.00) and the closing is run again,

**Then** the new entry contains exactly:

| Account | Debit | Credit |
|---|---|---|
| Inventory variation | 55.00 | |
| Inventory valuation | | 55.00 |

because

```formula
balance = 110.00 − 165.00 − 0.00 = −55.00   → the sides swap
```

### A5. Average cost is not disturbed by an outgoing movement

**Given** a product using average cost with 20 units on hand at a unit cost of 11.00,

**When** any quantity is delivered,

**Then** the unit cost is still 11.00 afterwards, and the total value has fallen by
*delivered quantity × 11.00*.

### A6. Average cost with a receipt while nothing is on hand

**Given** a product using average cost with 0 units on hand and a unit cost of 10.00,

**When** a receipt of 10 units valued at 120.00 is validated,

**Then** the fast path's first case does not apply (the previous quantity is 0), the
second does, and the unit cost becomes 120.00 ÷ 10 = **12.00**, with a total value of
120.00.

### A7. Two receipts validated in one batch

**Given** a product using average cost with nothing on hand,

**When** two receipts — one of 1 unit valued at 10.00 and one of 1 unit valued at 5.00 —
are validated **together in one operation**,

**Then** both movements are valued, the quantity on hand is 2, the unit cost is
**7.50** and the total value is 15.00.

### A8. A receipt whose quantity is later reduced

**Given** a product using average cost with nothing on hand,

**When** a receipt of 1 unit at 10.00 is validated, then a second receipt is created for
3 units at 5.00 but only 1 unit is actually picked and its value set to 5.00,

**Then** the quantity on hand is 2.00 and the unit cost is **7.50**.

### A9. Average cost with a consigned receipt

**Given** a product using average cost with 80 units on hand worth 1 273.50 (a unit cost
of 15.91875),

**When** a receipt of 10 units is validated with an **owner** set on its lines, and the
movement's value is then set manually to 990.00,

**Then** the movement's value is **0.00**, because every line is excluded for valuation
and the movement is therefore neither incoming nor outgoing,

**And** the quantity on hand rises by 10 but the total value is unchanged.

### A10. The full average worked sequence

**Given** a product using average cost,

**When** the following happen in order, each receipt having its value set explicitly:

| Step | Event | Value set |
|---|---|---|
| 1 | receive 60 | 900.00 |
| 2 | receive 140 | 2 170.00 |
| 3 | deliver 190 | — |
| 4 | receive 70 | 1 120.00 |
| 5 | deliver 30 | — |
| 6 | receive 10 with an owner set | 990.00 |
| 7 | deliver 50 | — |

**Then**:

| Step | Movement value | Quantity | Unit cost | Total value |
|---|---|---|---|---|
| 1 | 900.00 | 60 | 15.00 | 900.00 |
| 2 | 2 170.00 | 200 | 15.35 | 3 070.00 |
| 3 | **2 916.50** | 10 | 15.35 | 153.50 |
| 4 | 1 120.00 | 80 | 15.91875 | 1 273.50 |
| 5 | **477.56** | 50 | 15.91875 | 795.9375 |
| 6 | **0.00** | 60 (10 consigned) | 15.91875 | 795.9375 |
| 7 | **795.94** | 10 (all consigned) | 15.91875 | **0.00** |

**And** the movement values at steps 5 and 7 are rounded to the currency: 30 × 15.91875 =
477.5625 → 477.56, and 50 × 15.91875 = 795.9375 → 795.94.

**And** at the end the quantity on hand is 10 (the consigned units) and the total value is
0.00.

---

## Group B — First in first out

### B1. First in first out with receipts of ten at ten and ten at twelve, then a delivery of fifteen

**Given** a product using the **first in first out** costing method and the **periodic**
valuation mode, with nothing on hand and a unit cost of 10.00,

**When** the following happen in order:

1. a receipt of 10 units valued at 10.00 each is validated;
2. a receipt of 10 units valued at 12.00 each is validated;
3. a delivery of 15 units is validated;

**Then**:

| Step | Movement value | Quantity on hand | Product unit cost | Product total value |
|---|---|---|---|---|
| after 1 | 100.00 | 10 | 10.00 | 100.00 |
| after 2 | 120.00 | 20 | **11.00** | 220.00 |
| after 3 | **160.00** | 5 | **12.00** | **60.00** |

**And** the delivery's value is built by consuming the stack oldest first:

```formula
from the first receipt:  10 units × 10.00 = 100.00
from the second receipt:  5 units × 12.00 =  60.00
                                     total = 160.00
```

**And** the remaining quantities afterwards are: first receipt **0**, second receipt
**5**, delivery **0** (an outgoing movement never carries a remaining quantity).

**And** the remaining values afterwards are: first receipt 0.00, second receipt
120.00 × 5 ÷ 10 = **60.00**.

**And** the unit cost after step 2 is *total value ÷ quantity on hand* = 220.00 ÷ 20 =
11.00 — note that under first in first out the product's unit cost is the **average** of
what is on hand, not the cost of the oldest layer.

**And** the unit cost after step 3 is 60.00 ÷ 5 = 12.00 — an outgoing movement **does**
move the reported unit cost of a first in first out product.

### B2. The closing entry after scenario B1

**Given** the state at the end of B1, periodic valuation, no previous closing, ledger
balance 0.00,

**When** the closing is run,

**Then**:

| Account | Debit | Credit |
|---|---|---|
| Inventory valuation | 60.00 | |
| Inventory variation | | 60.00 |

### B3. The textbook first in first out sequence

**Given** a product using first in first out,

**When** the following happen in order:

| Step | Event |
|---|---|
| 1 | receive 68 valued at 15.00 each |
| 2 | receive 140 valued at 15.50 each |
| 3 | deliver 94 |
| 4 | receive 40 valued at 16.00 each |
| 5 | receive 78 valued at 16.50 each |
| 6 | deliver 116 |
| 7 | deliver 62 |
| 8 | transfer 10 to a transit location **of the company** |
| 9 | deliver 10 |

**Then**:

| Step | Movement value | Remaining quantities afterwards |
|---|---|---|
| 1 | 1 020.00 | receipt 1: 68 |
| 2 | 2 170.00 | receipt 1: 68, receipt 2: 140 |
| 3 | **1 423.00** | receipt 1: 0, receipt 2: 114 |
| 4 | 640.00 | receipt 2: 114, receipt 3: 40 |
| 5 | 1 287.00 | receipt 2: 114, receipt 3: 40, receipt 4: 78 |
| 6 | **1 799.00** | receipt 2: 0, receipt 3: 38, receipt 4: 78 |
| 7 | **1 004.00** | receipt 3: 0, receipt 4: 54 |
| 8 | **0.00** | unchanged — both ends are inside the valued perimeter |
| 9 | **165.00** | receipt 4: 44 |

**And** step 3's value is 68 × 15.00 + 26 × 15.50 = 1 020.00 + 403.00 = 1 423.00.

**And** step 6's value is 114 × 15.50 + 2 × 16.00 = 1 767.00 + 32.00 = 1 799.00.

**And** step 7's value is 38 × 16.00 + 24 × 16.50 = 608.00 + 396.00 = 1 004.00.

**And** the transfer at step 8 is neither incoming nor outgoing, and its remaining
quantity is 0.

### B4. Non-integer quantities under first in first out

**Given** a product using first in first out with nothing on hand,

**When** a receipt of 1.9 units valued at 10.00 each is validated,

**Then** the movement's remaining quantity is 1.9 and its remaining value is **19.00**.

### B5. Increasing the quantity of an already-completed receipt

**Given** a product using first in first out on which the following happened: a receipt A
of 10 units at 10.00 (value 100.00), a receipt B of 10 units at 8.00 (value 80.00), and a
delivery of 3 (value 30.00),

**When** receipt A's quantity is corrected from 10 to 12 and its value is set to 120.00,

**Then** receipt A's remaining quantity becomes **9**, not 7,

**Because** the stack is rebuilt from the current quantity on hand (19) newest-first:
receipt B offers 10, leaving 9 to cover, and receipt A supplies those 9. The two added
units therefore enter at the **top** of the queue.

**And** a subsequent delivery of 9 units is valued at 9 × (120.00 ÷ 12) = **90.00**.

**And** a subsequent delivery of 20 units, with only 10 units on hand, is valued at
80.00 (the whole of receipt B) plus 10 × 8.00 extrapolated from receipt B = **160.00**.

### B6. First in first out remaining value with a corrected movement value

**Given** a product using first in first out with a receipt A of 10 units at 2.00 (value
20.00) and a receipt B of 10 units at 4.00 (value 40.00), so that the unit cost is 3.00
and the total value is 60.00,

**When** receipt B's value is manually corrected to 60.00,

**Then** the unit cost becomes (20.00 + 60.00) ÷ 20 = **4.00**, receipt A's remaining
value is 20.00 and receipt B's remaining value is 60.00,

**And** after a delivery of 10 units, receipt A's remaining value is 0.00, receipt B's is
60.00, and the unit cost is 60.00 ÷ 10 = **6.00**,

**And** after a further delivery of 10 units, both remaining values are 0.00 and the unit
cost stays at **6.00** (the quantity on hand is zero, so the fallback takes the unit price
of the last incoming movement, which is 60.00 ÷ 10 = 6.00).

---

## Group C — Standard price

### C1. A standard price moved from ten to twelve with thirty on hand

**Given** a product using the **standard price** costing method and the **periodic**
valuation mode, with a unit cost of 10.00 and 30 units on hand, so that its total value is
300.00, and a previous closing that brought the inventory valuation account to a posted
balance of 300.00,

**When** a user writes a unit cost of 12.00 on the product,

**Then**:

1. a **valuation history record** is created carrying the product, the value **12.00**,
   the current instant, the acting user and the description **"Price update from 10.0 to
   12.0 by _the user name_"**;
2. the product's unit cost is 12.00;
3. the product's total value becomes 30 × 12.00 = **360.00**;
4. the quantity on hand is unchanged at 30;
5. **no journal entry is produced**;
6. the value of every completed movement of the product is **unchanged**;
7. the remaining value of every incoming movement of the product becomes *its remaining
   quantity × 12.00*.

**And when** the closing is then run,

**Then** the entry contains exactly:

| Account | Debit | Credit |
|---|---|---|
| Inventory valuation | 60.00 | |
| Inventory variation | | 60.00 |

because

```formula
balance = 360.00 − 300.00 − 0.00 = 60.00
```

### C2. The same change viewed as of a past date

**Given** the state at the end of C1, and that the cost change was recorded at instant T,

**When** the product's total value is asked for as of an instant strictly before T,

**Then** the unit cost used is the one the valuation history held before T — 10.00 — and
the total value is *the quantity on hand at that instant* × 10.00.

**And when** it is asked for as of a plain **date** whose day contains T,

**Then** the date is widened to the last instant of that day, so the change **is** seen
and the unit cost used is 12.00.

### C3. A standard price change on a product with nothing on hand

**Given** a product using standard price with 0 units on hand and a unit cost of 10.00,

**When** the unit cost is written as 12.00,

**Then** a valuation history record is created, the total value stays 0.00, and no
closing entry results from the change alone.

### C4. Writing the same unit cost again

**Given** a product using standard price with a unit cost of 12.00,

**When** 12.00 is written again,

**Then** **no** valuation history record is created, because the handler skips products
whose new cost equals the old one.

### C5. Writing a unit cost on a first in first out product

**Given** a product using first in first out,

**When** a user writes a unit cost on it,

**Then** the write succeeds and the field holds the written value, but **no valuation
history record is created**, because the handler skips products whose costing method is
`fifo`.

### C6. A product created with a unit cost

**Given** nothing,

**When** a product is created with a unit cost of 10.00,

**Then** a valuation history record is created for it carrying the value 10.00 and dated
at the **earliest representable instant**, recording a change from 0 to 10.00.

### C7. Standard price at a date for a non-standard product

**Given** a product using average cost,

**When** its dated unit cost is asked for with a date other than today,

**Then** the request fails with **"You can only get the standard price at a given date for
products with 'Standard Price' as cost method."**

### C8. A standard-price product valued at a date

**Given** a product using standard price whose valuation history holds 10.00 from day one
and 20.00 from day four, and which received 10 units on day one, 10 more on day three and
10 more on day six,

**When** its total value is asked for,

**Then**:

| As of | Unit cost used | Quantity on hand | Total value |
|---|---|---|---|
| day two | 10.00 | 10 | 100.00 |
| day five | 20.00 | 20 | 400.00 |
| today | 20.00 | 30 | 600.00 |

---

## Group D — Negative stock and its correction

### D1. A delivery of five before any receipt, then a receipt at eleven, under average cost

**Given** a product using **average cost** and the **periodic** valuation mode, with
nothing on hand, a unit cost of 10.00, and no previous closing,

**When** the following happen in order:

1. a delivery of 5 units is validated;
2. the closing is run and posted;
3. a receipt of 10 units valued at 11.00 each is validated;
4. the closing is run again;

**Then**:

| Step | Movement value | Quantity on hand | Unit cost | Total value | Ledger balance after |
|---|---|---|---|---|---|
| 1 | **50.00** (outgoing) | −5 | 10.00 | −50.00 | 0.00 |
| 2 | — | −5 | 10.00 | −50.00 | **−50.00** |
| 3 | **110.00** (incoming) | 5 | **11.00** | **55.00** | −50.00 |
| 4 | — | 5 | 11.00 | 55.00 | **55.00** |

**And** the closing entry of step 2 is:

| Account | Debit | Credit |
|---|---|---|
| Inventory variation | 50.00 | |
| Inventory valuation | | 50.00 |

**And** the closing entry of step 4 is:

| Account | Debit | Credit |
|---|---|---|
| Inventory valuation | 105.00 | |
| Inventory variation | | 105.00 |

because

```formula
balance = 55.00 − ( −50.00 ) − 0.00 = 105.00
```

**And the later correction is this**: the total value after step 3 is **55.00**, not
60.00. The replay's negative-recovery branch fired, because the quantity before the
receipt was −5, which is not greater than zero. It set the unit cost from the arriving
goods alone (110.00 ÷ 10 = 11.00) and rebuilt the value as 11.00 × 5 = 55.00 instead of
accumulating −50.00 + 110.00. The 5.00 difference is exactly the re-pricing of the 5 units
that were delivered at 10.00 but actually cost 11.00.

### D2. The same sequence under first in first out

**Given** the same starting point but with the **first in first out** costing method,

**When** the same four steps are performed,

**Then**:

| Step | Movement value | Quantity on hand | Unit cost | Total value |
|---|---|---|---|---|
| 1 | **50.00** | −5 | 10.00 (unchanged — the quantity on hand is not positive and there is no incoming movement to take a price from) | −50.00 |
| 3 | **110.00** | 5 | **11.00** | **55.00** |

**And** the delivery at step 1 is valued by extrapolation: the stack is empty, no movement
was used, so the product's unit cost 10.00 is applied to all 5 units.

**And** after step 3 the total value is the first in first out value of the 5 units on
hand: the stack holds only the receipt, whose bottom quantity is 5, so
110.00 × 5 ÷ 10 = 55.00.

**And** the closing entries are identical to D1.

### D3. Negative stock resolved in two receipts under first in first out

**Given** a product using first in first out and periodic valuation, with a unit cost set
to 8.00 before anything moves,

**When**:

1. 50 units are delivered;
2. the closing is run and posted;
3. 40 units are received valued at 15.00 each;
4. the closing is run and posted;
5. 20 units are received valued at 25.00 each;
6. the closing is run and posted;

**Then**:

| Step | Movement value | Quantity | Total value | Closing entry |
|---|---|---|---|---|
| 1 | **400.00** | −50 | −400.00 | — |
| 2 | — | −50 | −400.00 | valuation credited 400.00, variation debited 400.00 |
| 3 | **600.00** | −10 | **−150.00** | — |
| 4 | — | −10 | −150.00 | valuation debited **250.00**, variation credited 250.00 |
| 5 | **500.00** | 10 | **250.00** | — |
| 6 | — | 10 | 250.00 | valuation debited **400.00**, variation credited 400.00 |

**And** at step 3 the product's unit cost becomes 15.00, because the quantity on hand is
not positive and the fallback takes the unit price of the last incoming movement
(600.00 ÷ 40).

**And** at the end the posted balance of the inventory valuation account is **250.00**,
the posted balance of the variation account is **−250.00**, and the expense account has
never been touched.

### D4. Delivering more than is on hand under first in first out

**Given** a product using first in first out that received 10 units at 10.00 and then
delivered all 10,

**When** a further 21 units are delivered,

**Then** the movement is valued at 21 × 10.00 = **210.00** by extrapolation from the last
movement used — and, because the stack is empty and no movement is used, from the
product's unit cost, which the earlier update left at 10.00.

**And** the quantity on hand is −21 and the total value is **−210.00**.

**And** running the closing before this last delivery, when the total value and the
ledger balance are both 0.00, fails with **"Everything is correctly closed"**.

### D5. Negative stock under average cost recovered by editing an earlier receipt

**Given** a product using average cost on which the following happened: receive 10 at
10.00, receive 10 at 15.00 (unit cost 12.50, total value 250.00), deliver 15 (unit cost
12.50, total value 62.50), deliver 10 (quantity −5, total value −62.50),

**When** the second receipt's quantity is corrected from 10 to 20 and its value to 300.00,

**Then** the quantity on hand becomes **5**, the total value **66.67** and the unit cost
**13.3333333…**

**Because** the replay now reads: 10 in at 100.00 → cost 10.00, quantity 10, value
100.00; 20 in at 300.00 with a previous quantity of 10 > 0 → value 400.00, quantity 30,
cost 13.333333…; 15 out → value 200.00, quantity 15; 10 out → value 66.666666…, quantity
5. The reported total value is 66.67 after currency rounding.

### D6. Setting the quantity of a receipt to zero while stock is negative

**Given** the state before the correction in D5 (quantity −5),

**When** the second receipt's line quantity is set to 0,

**Then** the quantity on hand becomes **−15**.

---

## Group E — Inventory adjustments

### E1. An adjustment of minus three under average cost

**Given** a product using **average cost** and the **perpetual** valuation mode, with 10
units on hand at a unit cost of 12.00 (total value 120.00), the inventory-loss location
carrying a loss account, and the inventory valuation account of the product's category
set,

**When** the counted quantity is entered as 7 and the adjustment is applied,

**Then**:

1. an outgoing goods movement of 3 units is produced from the warehouse to the
   inventory-loss location;
2. its value is 3 × 12.00 = **36.00**;
3. the quantity on hand becomes 7 and the total value **84.00**;
4. the product's unit cost stays **12.00**;
5. a journal entry is produced in the company's inventory journal, dated today,
   containing exactly:

| Account | Debit | Credit |
|---|---|---|
| Inventory loss | 36.00 | |
| Inventory valuation | | 36.00 |

**And** the two journal items each carry the product and the label **"_the movement
reference_ - _the product name_"**.

### E2. The same adjustment under periodic valuation

**Given** the same starting point but with the **periodic** valuation mode, and a previous
closing that left the inventory valuation account at 120.00,

**When** the adjustment of minus 3 is applied and the closing is then run,

**Then** no entry is produced by the adjustment itself,

**And** the closing entry contains exactly:

| Account | Debit | Credit |
|---|---|---|
| Inventory loss | 36.00 | |
| Inventory valuation | | 36.00 |

**Because** part one produces the reclassification of 36.00, and part two then computes

```formula
extra_balance( inventory valuation ) = 0 − 36.00 = −36.00
balance = 84.00 − 120.00 − ( −36.00 ) = 0.00   → nothing further
```

### E3. A positive adjustment

**Given** a product using average cost and perpetual valuation with 5 units on hand at a
unit cost of 10.00, and the inventory-loss location carrying a loss account,

**When** the counted quantity is entered as 10 and the adjustment is applied,

**Then** an incoming goods movement of 5 units is produced **from** the inventory-loss
location, valued at 5 × 10.00 = 50.00 through the product-cost source of the priority
chain,

**And** the journal entry is:

| Account | Debit | Credit |
|---|---|---|
| Inventory valuation | 50.00 | |
| Inventory loss | | 50.00 |

**And** the quantity on hand is 10 and the total value 100.00.

### E4. Two products adjusted at once

**Given** two products using standard price and perpetual valuation, each with its own
inventory valuation account, the first with 5 on hand and the second with 15, and the
inventory-loss location carrying a loss account,

**When** both are counted at 10 and applied together,

**Then** four journal items are produced, in this order: the loss account for the first
product, the first product's inventory valuation account, the second product's inventory
valuation account, and the loss account for the second product — each item carrying its
own product.

### E5. An adjustment with an accounting date

**Given** the situation of E1, and an accounting date of the last day of the previous
month entered on the naming wizard,

**When** the adjustment is applied,

**Then** the journal entry produced is dated on that accounting date, not today,

**And** the goods movement's inventory name reads **"Product Quantity Updated (_the user
display name_) [Accounted on _that date_]"**,

**And** the accounting date field on the stock quantity record is cleared afterwards.

**And when** the counted quantity equals the recorded quantity (a zero adjustment), the
name reads **"Product Quantity Confirmed"** instead of "Product Quantity Updated".

### E6. The accounting date field is hidden for periodic products

**Given** a naming wizard covering only products whose valuation mode is `periodic`,

**When** the wizard form is rendered,

**Then** the accounting date field is hidden, because the "should show accounting date"
computation returns false.

---

## Group F — Returns

### F1. A customer return

**Given** a product using **average cost** and the **periodic** valuation mode, with 20
units on hand at a unit cost of 11.00, and a delivery of 10 units that was validated and
valued at **110.00**,

**When** a return of 4 units is created **from that delivery** and validated,

**Then**:

1. the return movement is classified **incoming** (it comes from the customer location
   into the warehouse);
2. the value priority chain reaches the **return source**, because the return's
   originating movement is outgoing;
3. the value is

   ```formula
   110.00 × 4 ÷ 10 = 44.00
   ```

4. the justification reads **"Value based on original move _the delivery reference_"**;
5. the quantity on hand becomes 14 and the total value rises by 44.00;
6. the product's unit cost is **unchanged at 11.00**, because the goods re-enter at
   exactly the cost at which they left.

### F2. A customer return after the cost has moved

**Given** the same delivery of 10 units valued at 110.00, but after it a receipt that
raised the unit cost to 13.00,

**When** 4 units are returned from that delivery,

**Then** the return is still valued at **44.00** — the return source uses the originating
movement's value, not today's cost.

### F3. A return of everything

**Given** a delivery of 10 units valued at 110.00,

**When** all 10 units are returned from it,

**Then** the return is valued at 110.00 × 10 ÷ 10 = **110.00**, exactly reversing the
delivery.

### F4. A return with no originating movement

**Given** a product using average cost with a unit cost of 13.00,

**When** an incoming movement from the customer location is created and validated with no
originating returned movement,

**Then** the return source does not apply; the chain falls through to the product cost
source and the movement is valued at *quantity × 13.00*.

### F5. A vendor return

**Given** a product using average cost with 20 units on hand at a unit cost of 11.00, of
which 10 were received at 12.00,

**When** 4 units are returned to the vendor from that receipt,

**Then** the return movement is classified **outgoing** (it goes from the warehouse to the
vendor location),

**And** the return source does **not** apply, because the originating movement (the
receipt) is incoming, not outgoing,

**And** the movement is valued at 4 × 11.00 = **44.00** — at today's average, not at the
12.00 the goods were received at.

### F6. The "update quantities on the order" switch

**Given** a return wizard line with the switch left at its default,

**When** the return is created,

**Then** the created movement's corresponding field is **true**,

**And when** the switch is cleared on the wizard line, the created movement's field is
false.

---

## Group G — Landed costs

### G1. A landed cost of one hundred split by quantity across two products

**Given**:

- product A using **average cost** with **perpetual** valuation, weight 10, volume 1;
- product B using **average cost** with **perpetual** valuation, weight 20, volume 1.5;
- a receipt containing a movement of 30 units of A valued at 300.00 and a movement of 20
  units of B valued at 400.00, both validated and entirely still on hand;
- a service product "Freight" flagged as a landed cost, whose expense account is the
  freight expense account;

**When** a landed cost document is created with one cost line for Freight of **100.00**
split **by quantity**, the receipt is selected, "Compute" is pressed and the document is
validated,

**Then** the valuation adjustment lines are:

| Product | Quantity | Weight | Volume | Original value | Allocated | New value |
|---|---|---|---|---|---|---|
| A | 30 | 300 | 30 | 300.00 | **60.00** | 360.00 |
| B | 20 | 400 | 30 | 400.00 | **40.00** | 440.00 |

**Because**

```formula
total_quantity = 30 + 20 = 50
per_unit       = 100.00 ÷ 50 = 2.00
share( A )     = round_to_currency( 30 × 2.00 ) = 60.00
share( B )     = round_to_currency( 20 × 2.00 ) = 40.00
split_total    = 100.00
residue        = round_to_currency( 100.00 − 100.00 ) = 0.00
```

**And** the sum check passes: 60.00 + 40.00 = 100.00 = the document total, and the same
sum equals the single cost line's amount.

**And** the journal entry produced in the document's journal, dated on the document date
and referenced with the document's sequence number, contains exactly:

| Account | Debit | Credit |
|---|---|---|
| Inventory valuation of A | 60.00 | |
| Freight expense | | 60.00 |
| Inventory valuation of B | 40.00 | |
| Freight expense | | 40.00 |

**And** afterwards:

| | Product A | Product B |
|---|---|---|
| Receipt movement value | **360.00** | **440.00** |
| Quantity on hand | 30 | 20 |
| Unit cost | **12.00** | **22.00** |
| Total value | 360.00 | 440.00 |

**And** the receipt movements' value justification gains the block:

```
Additional landed costs:
+ 60.00 (Landed Cost: LC/2026/0001)
```

(with the vendor bill's display name inserted when the document was created from a bill).

**And** the document's state is `done`, its journal entry reference is set, and a message
was posted under the "Landed cost validated" subtype.

### G2. The same landed cost split equally

**Given** the same starting point,

**When** the split method is **equal** instead of by quantity,

**Then** each of the two valuation lines receives 100.00 ÷ 2 = **50.00**, product A's
movement becomes worth 350.00 and product B's 450.00.

### G3. The same landed cost split by weight

**Given** the same starting point,

**When** the split method is **by weight**,

```formula
total_weight = 10 × 30 + 20 × 20 = 300 + 400 = 700
per_unit     = 100.00 ÷ 700 = 0.142857…
share( A )   = round_to_currency( 300 × 0.142857… ) = 42.86
share( B )   = round_to_currency( 400 × 0.142857… ) = 57.14
split_total  = 100.00
residue      = 0.00
```

**Then** product A receives 42.86 and product B receives 57.14.

### G4. The same landed cost split by volume

**Given** the same starting point,

```formula
total_volume = 1 × 30 + 1.5 × 20 = 30 + 30 = 60
per_unit     = 100.00 ÷ 60 = 1.666666…
share( A )   = round_to_currency( 30 × 1.666666… ) = 50.00
share( B )   = round_to_currency( 30 × 1.666666… ) = 50.00
```

**Then** each product receives 50.00.

### G5. The same landed cost split by current cost

**Given** the same starting point,

```formula
total_cost = round_to_currency( 300.00 ) + round_to_currency( 400.00 ) = 700.00
per_unit   = 100.00 ÷ 700.00 = 0.142857…
share( A ) = round_to_currency( 300.00 × 0.142857… ) = 42.86
share( B ) = round_to_currency( 400.00 × 0.142857… ) = 57.14
```

**Then** product A receives 42.86 and product B receives 57.14.

### G6. A landed cost with a rounding residue

**Given** a receipt containing three movements of three different products, each using
average cost with perpetual valuation, each entirely still on hand,

**When** a landed cost of 100.00 is split **equally**,

```formula
raw share   = 100.00 ÷ 3 = 33.333333…
rounded     = 33.33 each
split_total = 99.99
residue     = round_to_currency( 100.00 − 99.99 ) = 0.01
```

**Then** the adjustment line with the **highest identifier** receives 33.33 + 0.01 =
**33.34**, and the three shares sum to exactly 100.00,

**And** the sum check passes.

### G7. A landed cost whose goods are partly consumed

**Given** the situation of G1 but where, by validation time, only 18 of product A's 30
units and none of product B's 20 units are still on hand,

**When** the document is validated,

**Then**:

```formula
posted_amount( A ) = 60.00 × ( 18 ÷ 30 ) = 36.00
posted_amount( B ) = 40.00 × (  0 ÷ 20 ) =  0.00   → no items produced for B
```

**And** the journal entry contains exactly:

| Account | Debit | Credit |
|---|---|---|
| Inventory valuation of A | 36.00 | |
| Freight expense | | 36.00 |

**And** the **full** allocations (60.00 and 40.00) still enter the two receipt movements'
values through the extra source, so product A's receipt is worth 360.00 and product B's
440.00.

### G8. A landed cost on a product using standard price

**Given** a receipt containing a movement of a product using **standard price**,

**When** the landed cost's split is computed,

**Then** that movement is skipped entirely, because the collection of valuation lines
only accepts products whose costing method is `fifo` or `average`,

**And when** the receipt contains **only** such products, the computation fails with
**"You cannot apply landed costs on the chosen Transfers(s). Landed costs can only be
applied for products with FIFO or average costing method."**

### G9. A landed cost on a product using periodic valuation

**Given** a receipt containing a movement of a product using first in first out with the
**periodic** valuation mode, entirely still on hand,

**When** a landed cost of 50.00 is allocated entirely to it and the document is
validated,

**Then** **no journal item is produced for that line**, so, if it is the only line, the
document moves to `done` with **no journal entry at all**,

**And** the movement's value nevertheless rises by 50.00, and the difference is reported
at the next closing.

### G10. A negative landed cost reversing an earlier one

**Given** the state at the end of G1,

**When** a second landed cost document is created targeting the same receipt, with a
Freight cost line of **−100.00** split by quantity, computed and validated,

**Then** the allocations are −60.00 for A and −40.00 for B,

**And** the journal entry is:

| Account | Debit | Credit |
|---|---|---|
| Freight expense | 60.00 | |
| Inventory valuation of A | | 60.00 |
| Freight expense | 40.00 | |
| Inventory valuation of B | | 40.00 |

**And** the two receipt movements return to 300.00 and 400.00, and the unit costs to
10.00 and 20.00.

### G11. A posted landed cost cannot be cancelled

**Given** the document of G1 in the `done` state,

**When** "Cancel" is pressed, or the document is deleted,

**Then** the operation fails with **"Validated landed costs cannot be cancelled, but you
could create negative landed costs to reverse them"**.

### G12. A landed cost with no target

**Given** a draft landed cost document with cost lines but no transfer and no
manufacturing order selected,

**When** "Validate" is pressed,

**Then** the operation fails with **"Please define Transfers on which those additional
costs should apply."**

### G13. A landed cost whose split was edited so that it no longer sums

**Given** a draft landed cost document with one cost line of 100.00 and two adjustment
lines of 60.00 and 40.00,

**When** a user edits one allocation to 50.00 and presses "Validate",

**Then** the document already has adjustment lines, so no recomputation happens, the sum
check fails (110.00 ≠ 100.00) and the operation fails with **"Cost and adjustments lines
do not match. You should maybe recompute the landed costs."**

### G14. A landed cost whose cost line has no account and whose product has no expense account

**Given** a draft landed cost document whose cost line has an empty account and whose cost
product resolves to no expense account,

**When** "Validate" is pressed and the journal entry is being built,

**Then** the operation fails with **"Please configure Stock Expense Account for product:
_the cost product name_."**

### G15. A landed cost created from a vendor bill

**Given** a posted vendor bill in the company currency containing a line for the service
product "Freight" of 100.00 flagged as a landed cost line,

**When** "Create Landed Costs" is pressed,

**Then** a landed cost document is created with the bill as its vendor bill and one cost
line carrying the Freight product, the product's name as the description, the product's
expense account, the amount

```formula
+1 × round_to_currency( 100.00 ÷ 1 ) = 100.00
```

and the product's default split method, falling back to `equal`,

**And** the form of that document is opened,

**And** the "Create Landed Costs" button disappears from the bill, because the bill now
has a landed cost document.

### G16. A landed cost created from a vendor credit note

**Given** the same but on a vendor **credit note** for 100.00,

**When** "Create Landed Costs" is pressed,

**Then** the created cost line's amount is **−100.00**.

### G17. A landed cost created from a foreign-currency bill

**Given** a vendor bill in a currency whose rate on the bill is 2 (two units of the
document currency per unit of company currency), containing a Freight line whose subtotal
is 100.00 in the document currency,

**When** "Create Landed Costs" is pressed,

**Then** the cost line's amount is

```formula
round_to_currency( 100.00 ÷ 2 ) = 50.00
```

in the company currency.

### G18. A landed cost on a manufacturing order

**Given** manufacturing landed costs installed, and a completed manufacturing order whose
finished-goods movement is incoming and whose by-product movement carries a cost share of
zero,

**When** a landed cost document targets that order,

**Then** the targeted movements are the finished-goods movements **minus** the by-product
movement whose cost share is zero.

### G19. A landed cost on a subcontracted receipt

**Given** subcontracting landed costs installed, and a transfer containing a
subcontracting movement,

**When** a landed cost document targets that transfer,

**Then** the subcontracting movement is replaced, in the targeted set, by its
**originating** movements.

### G20. A landed-cost flag on a non-service line

**Given** a vendor bill line whose product is a storable product,

**When** the landed-cost flag is set on it,

**Then** the onchange forces it back to **false**.

### G21. Turning a used landed-cost service product into a storable product

**Given** a service product flagged as a landed cost that is referenced by at least one
journal item carrying the landed-cost flag,

**When** its type is changed to storable, or its landed-cost flag is cleared,

**Then** the write fails with **"You cannot change the product type or disable landed cost
option because the product is used in an account move line."**

---

## Group H — Cost of goods sold and deferred recognition

### H1. An invoice for a partly delivered order under deferred cost recognition

**Given**:

- a product using **first in first out** with the **perpetual** valuation mode;
- the company uses **anglo-saxon accounting**;
- the product's inventory valuation account and expense account are both set;
- a sales order for 10 units;
- 10 units on hand at 10.00 each and, behind them, more units at 12.00 each;

**When**:

1. 6 units are delivered — the delivery is valued at 6 × 10.00 = **60.00**;
2. an invoice is issued and posted for those 6 units;
3. the remaining 4 units are delivered — the delivery is valued at 4 units, of which the
   last 4 at 10.00 are exhausted after the first 6, so these come from the 12.00 layer:
   4 × 12.00 = **48.00**;
4. a second invoice is issued and posted for the remaining 4 units;

**Then** the first invoice's injected items are:

```formula
cogs_quantity   = 0 + 6 = 6
unit_price      = 60.00 ÷ 6 = 10.00
already_posted  = 0.00
cogs_unit_price = | 10.00 × 6 − 0.00 | ÷ 6 = 10.00
amount          = 6 × 10.00 = 60.00
```

| Account | Debit | Credit |
|---|---|---|
| Cost of goods sold (the product's expense account) | 60.00 | |
| Inventory valuation | | 60.00 |

**And** the second invoice's injected items are:

```formula
posted_quantity = 6
cogs_quantity   = 6 + 4 = 10
unit_price      = ( 60.00 + 48.00 ) ÷ ( 6 + 4 ) = 10.80
already_posted  = 60.00
cogs_unit_price = | 10.80 × 10 − 60.00 | ÷ 4 = 48.00 ÷ 4 = 12.00
amount          = 4 × 12.00 = 48.00
```

| Account | Debit | Credit |
|---|---|---|
| Cost of goods sold | 48.00 | |
| Inventory valuation | | 48.00 |

**And** the total cost recognised across the two invoices is 108.00, exactly the value
that left stock, even though the average unit price across both deliveries was 10.80 and
neither invoice used it as its own unit price.

**And** each invoice carries four journal items: receivable, revenue, and the two
injected items, all four with the same commercial partner, the injected two carrying the
display type `cogs`, the product, the line's unit of measure, the line's quantity, the
line's analytic distribution and no taxes.

### H2. Invoicing in full before delivering

**Given** the same configuration, 10 units on hand at 10.00 each, and a sales order for
10 units,

**When** the invoice for all 10 units is posted **before** anything is delivered,

**Then** there are no completed movements, so the unit price is taken from the first in
first out valuation of 10 units:

```formula
unit_price      = fifo_value( 10 ) ÷ 10 = 100.00 ÷ 10 = 10.00
cogs_quantity   = 10
already_posted  = 0.00
cogs_unit_price = | 10.00 × 10 − 0.00 | ÷ 10 = 10.00
amount          = 100.00
```

| Account | Debit | Credit |
|---|---|---|
| Cost of goods sold | 100.00 | |
| Inventory valuation | | 100.00 |

### H3. A credit note reversing an invoice, first in first out

**Given** a posted customer invoice for 1 unit of a product using first in first out
whose cost at the time was 9.00, so the invoice carries a debit of 9.00 to the expense
account and a credit of 9.00 to the inventory valuation account,

**When** the invoice is reversed and the credit note is posted,

**Then** the shortcut that reuses the original unit price does **not** apply (it only
applies to standard price and average cost), the cost is recomputed, the sign is −1, and
the injected items are:

| Account | Debit | Credit |
|---|---|---|
| Inventory valuation | 9.00 | |
| Cost of goods sold | | 9.00 |

### H4. A credit note reversing an invoice, standard price

**Given** the same but with a product using **standard price** whose cost was 9.00 at the
time of the invoice and has since been changed to 11.00,

**When** the invoice is reversed and the credit note is posted,

**Then** the shortcut applies: the injected items use the unit price recorded on the
original invoice's inventory-side item, namely 9.00, so the reversal is exact:

| Account | Debit | Credit |
|---|---|---|
| Inventory valuation | 9.00 | |
| Cost of goods sold | | 9.00 |

### H5. A credit note that is not a reversal

**Given** a product using average cost, a sales order, a posted invoice carrying injected
items, and a credit note created directly (not as a reversal of that invoice) for the
same sales order lines,

**When** the credit note is posted,

**Then** the shortcut still finds the original items — through the sales order lines
rather than through the reversal reference — and reuses their unit price.

### H6. A credit note under negative-number bookkeeping

**Given** a company that keeps its books with negative amounts rather than side
reversals, a product using standard price with perpetual valuation and a cost of 10.00,
and a customer credit note for 1 unit,

**When** the credit note is posted,

**Then** the inventory-side item shows a debit of 0.00 and a credit of **−10.00**, and the
expense-side item shows a debit of **−10.00** and a credit of 0.00.

### H7. A resettable invoice

**Given** a posted invoice carrying two injected items,

**When** it is reset to draft,

**Then** both injected items are deleted and the invoice is left with its two original
items,

**And when** it is posted again, the items are rebuilt — possibly with different amounts
if goods moved in the meantime.

### H8. Copying an invoice

**Given** a posted invoice carrying two injected items,

**When** it is duplicated,

**Then** the copy contains **only** the original items; the injected ones are dropped.

### H9. An invoice line with no label

**Given** a customer invoice line whose label is empty,

**When** the invoice is posted,

**Then** the injected items carry an empty label, the line keeps its empty label, and the
invoice reaches the posted state.

### H10. A product with no expense account

**Given** a product using perpetual valuation whose expense account resolves to nothing
and whose journal has no default account,

**When** an invoice for it is posted,

**Then** the line is **skipped**: no injected items are produced and the invoice posts
with its two original items.

### H11. A drop-shipped line

**Given** a sales order line fulfilled by a drop shipment,

**When** the invoice is posted,

**Then** the line is **not eligible for stock accounting** — because at least one of its
reachable movements is a drop shipment — so no injected items are produced.

### H12. A cost-of-goods-sold unit price with consignment

**Given** a set of movements for one product containing a delivery of 10 units valued at
110.00 and one consigned valued line of 2 units leaving the perimeter,

**Then**:

```formula
signed_quantity       = 10
consigned_quantity    = +2
total_valued_quantity = 12
signed_value          = 110.00
unit_price            = 110.00 ÷ 12 = 9.166666…
```

and the consigned branch of the condition applies even under standard price, because the
consigned quantity is non-zero.

### H13. A cost-of-goods-sold unit price over a delivery and a return

**Given** a delivery of 10 units valued at 110.00 and a customer return of 3 units valued
at 33.00 for the same product,

**Then**:

```formula
signed_quantity       = 10 (out) − 3 (in) = 7
signed_value          = 110.00 − 33.00 = 77.00
unit_price            = 77.00 ÷ 7 = 11.00
```

### H14. A cost-of-goods-sold unit price with several products

**Given** a set of movements covering two different products,

**Then** the unit price is **0** by definition.

---

## Group I — Vendor bills

### I1. A bill priced differently from its receipt, first in first out

**Given**:

- a product using **first in first out** with the **perpetual** valuation mode;
- a purchase order line for 10 units at **10.00** each;
- the receipt of all 10 units validated **before** the bill is posted;

**When** the receipt is validated and then a vendor bill for 10 units at **12.00** each is
posted,

**Then**:

| Moment | Movement value | Quantity | Unit cost | Total value |
|---|---|---|---|---|
| after the receipt | **100.00** (from the purchase order line) | 10 | 10.00 | 100.00 |
| after the bill | **120.00** (from the bill) | 10 | **12.00** | 120.00 |

**And** the movement's justification changes from

```
100.00 for 10.0 Units from P00007 (not billed)
```

to

```
120.00 for 10.0 Units from BILL/2026/03/0007
```

**And** the bill's own items are:

| Account | Debit | Credit |
|---|---|---|
| Inventory valuation | 120.00 | |
| Accounts payable | | 120.00 |

**And** **no** price difference is posted, because the costing method is not `standard`;
the difference is absorbed into the value of the goods.

### I2. The same under standard price with anglo-saxon accounting

**Given** the same purchase order and receipt, but with a product using **standard price**
whose unit cost is **9.00**, perpetual valuation, anglo-saxon accounting on, and the
category's price difference account set,

**When** a vendor bill for 10 units at **10.00** each is posted,

**Then** the bill's own items debit the inventory valuation account 100.00 and credit
accounts payable 100.00,

**And** the price difference mechanism computes

```formula
valuation_unit_price      = 9.00
gross_unit_price          = 10.00
price_unit_difference     = 1.00
relevant_quantity         = 10
price_subtotal_difference = 10.00       → not zero, and the unit prices agree at 2 decimals
```

**And** injects:

| Account | Debit | Credit |
|---|---|---|
| Price difference | 10.00 | |
| Inventory valuation | | 10.00 |

**And** the net debit to the inventory valuation account is 90.00 — ten units at the
standard cost of 9.00 — which matches the product's total value.

### I3. The same under standard price without anglo-saxon accounting

**Given** the same but with anglo-saxon accounting **off**,

**When** the bill is posted,

**Then** **no** price difference items are injected, because the mechanism requires the
anglo-saxon flag.

### I4. A bill under periodic valuation

**Given** a product using **periodic** valuation,

**When** a vendor bill for it is posted,

**Then** the bill line's account is **not** redirected: it stays the product's expense
account, and the bill's items are a debit to expenses and a credit to accounts payable.

### I5. A bill with a discount

**Given** a product using standard price with a cost of 9.00, perpetual valuation and
anglo-saxon accounting,

**When** a vendor bill line for 10 units at 12.00 with a discount that makes the stored
unit price and the computed unit price disagree at the Product Price precision is posted,

**Then** the second condition of the price-difference test fails and **no** price
difference items are injected.

### I6. A bill posted before the receipt

**Given** a purchase order line for 10 units at 10.00 and a product using first in first
out with perpetual valuation,

**When** the bill for 10 units at 12.00 is posted **before** any goods arrive, and the
receipt of 10 units is then validated,

**Then** the receipt's value comes from the bill source, not the quotation source: the
billed quantity is 10, the absorbed quantity is 0, and the movement is valued at
**120.00**.

### I7. A bill covering more than one receipt

**Given** a purchase order line for 10 units at 10.00 fulfilled by two receipts of 6 and
4 units, both validated before the bill,

**When** a bill for 10 units at 12.00 is posted,

**Then** the **first** receipt (the earlier one by date and identifier) sees an absorbed
quantity of 0 and claims min(6, 10) = 6 units, valued at

```formula
120.00 × 6 ÷ 10 = 72.00
```

**And** the **second** receipt sees an absorbed quantity of 6, so its available quantity
is 4 and its available value is

```formula
120.00 × ( 10 − 6 ) ÷ 10 = 48.00
```

and it claims min(4, 4) = 4 units, valued at **48.00**.

**And** the two receipts together are worth 120.00.

### I8. A partial bill

**Given** a purchase order line for 10 units at 10.00 and a receipt of all 10 units
validated,

**When** a bill for only 6 units at 12.00 is posted,

**Then** the bill source claims 6 units and contributes 72.00; the remaining 4 units fall
through to the quotation source and contribute 4 × 10.00 = 40.00,

**And** the movement's total value is **112.00**, with a justification containing both
lines.

### I9. A vendor credit note

**Given** the situation of I1 after the bill,

**When** a vendor credit note for 4 units at 12.00 is posted against the same purchase
order line,

**Then** the bill source accumulates 10 − 4 = 6 units and 120.00 − 48.00 = 72.00, and the
receipt is re-valued at 72.00 for the 6 claimed units plus 4 × 10.00 from the quotation
source = **112.00**.

### I10. A bill in a foreign currency

**Given** a purchase order and a vendor bill in a currency whose rate on the bill lines is
2 (two units of the document currency per unit of company currency), with a line whose
subtotal is 240.00 in the document currency for 10 units,

**When** the bill is posted,

**Then** the bill source contributes

```formula
round_to_currency( 240.00 ÷ 2 ) = 120.00
```

in the company currency, and the receipt is worth 120.00.

**And** the rate used is the **bill's own** rate, not today's rate.

### I11. A purchase order line whose price changes after the receipt

**Given** a validated receipt against a purchase order line at 10.00, not yet billed,

**When** the purchase order line's unit price is changed to 12.00,

**Then** every valued movement of that line is re-valued and the receipt becomes worth
120.00.

### I12. A purchase order line in a different unit of measure

**Given** a purchase order line for 5 packs of 6 units at 60.00 per pack, and a product
whose reference unit is the unit,

**When** a receipt of 2 packs is validated,

**Then**:

```formula
step 3 of the unit-price computation: 60.00 ÷ 6 × 1 = 10.00 per unit
reference quantity of the receipt:    2 × 6 = 12
value:                                10.00 × 12 = 120.00
```

### I13. A receipt in a different unit of measure with no purchase order

**Given** a product whose reference unit is the unit and whose unit cost is 10.00,

**When** a receipt of 1 **dozen** is validated with no purchase order,

**Then** the reference quantity is 12 and the movement's value is **120.00**.

### I14. A receipt whose reference unit is coarser than the movement unit

**Given** a product whose reference unit is the kilogram, using standard price,

**When** a receipt of 11 **grams** is validated,

**Then** the movement's own quantity is 11, its reference quantity is **0.01** kilogram
(11 × 0.001, rounded at the product-unit precision), and the quantity on hand becomes
0.01.

**And when** 11 grams are then delivered, the quantity on hand becomes 0.00.

---

## Group J — Manual value adjustment

### J1. Adjusting the value of one movement

**Given** a product using average cost with three receipts of 1 unit each, all valued at
1.00, so that the quantity on hand is 3, the unit cost is 1.00 and the total value is
3.00,

**When** the first receipt's value is adjusted to **2.00**,

**Then**:

1. a valuation history record is created carrying the movement and the value 2.00;
2. the movement's value becomes 2.00;
3. the product's unit cost becomes **1.3333333…**;
4. the product's total value becomes **4.00**;
5. the sum of the three movements' **remaining values** is **3.99**, because each is
   computed as *remaining quantity × unit cost* and rounded to the currency separately:
   1.33 + 1.33 + 1.33;
6. the first movement's remaining value is **1.33**.

**And when** the closing is run with no previous closing and a ledger balance of 0.00,

| Account | Debit | Credit |
|---|---|---|
| Inventory valuation | 4.00 | |
| Inventory variation | | 4.00 |

### J2. The adjustment suppresses landed costs

**Given** a receipt whose value is 300.00 and to which a landed cost of 60.00 has been
allocated, so that the value priority chain reports 360.00,

**When** the movement's value is adjusted to **350.00**,

**Then** the value is exactly **350.00**, **not** 410.00 — the manual correction claims
the whole quantity and suppresses the extra source.

**And** the movement's value justification reads **"Adjusted on _the date_ by _the user
name_"** followed by the description,

**And** the **computed value description** reads **"Computed value: 360.00"** followed by
the justification the movement would have had, so that the reader can see what was
overridden.

### J3. Several adjustments

**Given** a movement that has been adjusted to 350.00 and then to 330.00,

**Then** the value is 330.00 — the latest record by date, ties broken by the highest
identifier, wins; earlier records remain as history and are not summed.

### J4. Adjusting more than one movement at once

**Given** two movements selected,

**When** the "Adjust Valuation" action is run,

**Then** the operation fails with **"You can only adjust valuation for one move at a
time."**

### J5. The adjustment dialog's details line

**Given** a movement of 10 units worth 120.00,

**When** the adjustment dialog is opened,

**Then** the details line reads **"For 10.0 Units (12.0 per Units)"**.

**And when** the movement's quantity is zero, the details line is empty.

### J6. Adjusting a movement of a lot-valuated product

**Given** a lot-valuated product using average cost,

**When** a movement's value is adjusted,

**Then** the movement is re-valued, the product's unit cost is recomputed and every lot of
that product has its cost recomputed.

### J7. An adjustment as of a past date

**Given** a movement adjusted at instant T,

**When** the movement's value is asked for as of an instant strictly before T,

**Then** the correction is not seen and the chain falls through to the next source.

**And when** it is asked for as of exactly T, the correction **is** seen, because the
lookup uses "date not after the as-of instant".

---

## Group K — Valuation by lot or serial number

### K1. Enabling lot valuation while untracked stock exists

**Given** a tracked product with 5 units on hand in the warehouse carrying no lot,

**When** valuation by lot is switched on,

**Then** the write fails with **"You cannot enable lot valuation because the following
products have on-hand quantities without a lot/serial number:"** followed by the product's
display name.

### K2. Enabling lot valuation cleanly

**Given** a tracked product with nothing on hand,

**When** valuation by lot is switched on,

**Then** the write succeeds, every lot of the product is recomputed, and lots with no cost
receive the product's unit cost.

### K3. A receipt without a lot on a lot-valuated product

**Given** a lot-valuated product,

**When** an incoming movement is validated with a line carrying no lot or serial number,

**Then** the validation fails with **"A lot/serial number is required for product '_the
product display name_' as it has lot valuation enabled."**

### K4. An outgoing movement of a lot-valuated product

**Given** a lot-valuated product with lot X at a cost of 10.00 and lot Y at a cost of
14.00,

**When** a delivery is validated with 3 units of lot X and 2 units of lot Y,

**Then** the movement's value is

```formula
3 × 10.00 + 2 × 14.00 = 30.00 + 28.00 = 58.00
```

— the costing method is not consulted; each line is valued at its own lot's cost.

### K5. A line with no lot on an outgoing movement of a lot-valuated product

**Given** the same product,

**When** a delivery is validated with 3 units of lot X and 2 units carrying no lot,

**Then** the 2 unlotted units are valued at the **product's** unit cost. (The missing-lot
refusal applies only to incoming movements.)

### K6. The total value of a lot-valuated product

**Given** a lot-valuated product with lot X holding 3 units worth 30.00 and lot Y holding
2 units worth 28.00,

**Then** the product's total value is **58.00** and its unit-cost figure is
58.00 ÷ 5 = 11.60.

### K7. Writing a unit cost on a lot-valuated product

**Given** a lot-valuated product using average cost with lots X and Y,

**When** a user writes a unit cost of 12.00 on the product,

**Then** a valuation history record is created for the product, **and** 12.00 is written
onto both lots with the automatic-revaluation flag disabled, so the lots do **not** each
create their own record.

### K8. Writing a cost on one lot

**Given** a lot-valuated product using **average cost** and a lot X at 10.00,

**When** a user writes 12.00 on lot X,

**Then** a valuation history record is created carrying the product, the lot, the value
12.00 and the description **"_the lot name_ price update from 10.0 to 12.0 by _the user
name_"**.

**And when** the product uses standard price or first in first out instead, **no** record
is created.

### K9. The unit cost of a lot-valuated product is an aggregate

**Given** a lot-valuated product whose costing method is not `standard`,

**When** the unit-cost maintenance routine runs,

**Then** the product's unit cost is set to its computed **average cost**, and the product
is excluded from the per-costing-method groups.

### K10. A lot created on a lot-valuated product

**Given** a lot-valuated product with a unit cost of 12.00,

**When** a new lot of that product is created with no cost,

**Then** the lot's cost is set to 12.00 and no valuation history record is produced.

---

## Group L — Perimeter and classification

### L1. An internal transfer

**Given** a transfer from one internal location of the company to another,

**When** it is validated,

**Then** the movement is neither incoming nor outgoing, its value is 0.00, its remaining
quantity is 0 and no journal entry is produced.

### L2. A transfer to a company transit location

**Given** a transfer from the warehouse to a transit location **belonging to the
company**,

**When** it is validated,

**Then** the movement is neither incoming nor outgoing and its value is 0.00, because a
transit location with a company is **inside** the valued perimeter.

### L3. A transfer to a company-less transit location

**Given** a transfer from the warehouse to a transit location belonging to **no** company,

**When** it is validated,

**Then** the movement **is** outgoing, because such a location is outside the valued
perimeter.

### L4. A transfer through an archived internal location

**Given** a transfer whose destination is an **archived** internal location of the
company,

**When** it is validated,

**Then** the movement is neither incoming nor outgoing, because the perimeter test does
not consider the archived flag and the perimeter search includes archived locations.

### L5. A movement with an unpicked line

**Given** a movement with two lines, one picked and one not,

**When** it is validated,

**Then** only the picked line contributes to the valued quantity.

### L6. A movement whose lines are all consigned

**Given** a movement all of whose lines carry an owner other than the company's partner,

**When** it is validated,

**Then** the movement is neither incoming nor outgoing and its value is 0.00, even if a
value was written on it beforehand.

### L7. A movement with a mix of owned and consigned lines

**Given** a receipt of 10 units of which 4 carry a third-party owner,

**When** it is validated,

**Then** the valued quantity is **6** and the movement's value covers only those 6.

### L8. A drop shipment

**Given** a movement from the vendor location directly to the customer location,

**When** it is validated,

**Then** it is flagged as a **drop shipment**, it is valued by the priority chain, and it
produces **no** journal entry.

### L9. A stock quantity record's value

**Given** a product with 100 units on hand across two locations worth 1 250.00 in total,
and a quantity record at one of those locations holding 40 units,

**Then** that record's value is

```formula
40 × 1 250.00 ÷ 100 = 500.00
```

### L10. A consigned stock quantity record's value

**Given** a quantity record whose owner is a third party,

**Then** its value is **0.00**.

### L11. A stock quantity record outside the valued perimeter

**Given** a quantity record at a customer location,

**Then** its value is **0.00**.

### L12. Summing values in a grouped read

**Given** several quantity records grouped by product,

**When** the sum of their values is asked for,

**Then** the aggregation is performed **in memory** over the computed values, not by the
database, and produces the correct total.

---

## Group M — Multi-company, multi-currency and branches

### M1. Two companies with different costing methods

**Given** one product, and a category whose costing method is `average` for company one
and `fifo` for company two,

**When** the product's costing method is read in each company's scope,

**Then** it is `average` in company one and `fifo` in company two.

### M2. The total value across companies

**Given** a product held by two companies with different currencies, worth 100.00 in
company one's currency and 200.00 in company two's currency,

**When** the total value is read with both companies in scope and company one in scope as
the current company,

**Then** the result is

```formula
100.00 + convert( 200.00, company two's currency → company one's currency )
```

### M3. A closing in one company does not touch the other

**Given** two companies each with goods on hand,

**When** the closing is run for company one,

**Then** only company one's accounts and movements are considered, and only one entry is
created, in company one.

### M4. The scheduled job across companies

**Given** company one with a `daily` inventory period, company two with `monthly`, and
company three with `manual`,

**When** the scheduled job runs on a day that is **not** the last of the month,

**Then** only company one is processed.

**And when** it runs on the last day of the month, companies one and two are processed and
company three is not.

### M5. A company whose closing fails during the job

**Given** company one with no inventory journal set and company two correctly configured,
both with a `daily` period,

**When** the scheduled job runs,

**Then** company one's closing raises and is **skipped silently**, and company two's
closing is created and posted.

### M6. A branch company overriding the valuation account

**Given** a parent company and a branch company, and a category whose valuation account
is set to a branch-specific account for the branch and whose valuation mode is
`real_time` for the branch,

**When** a vendor bill is created in the branch for a product of that category,

**Then** the bill line's account is the **branch-specific** account.

### M7. The scope-company rule

**Given** a product belonging to company one,

**When** its costing method is read with company two in scope, where company two is
neither company one nor one of its descendants,

**Then** the costing method is read in **company one**, not company two.

**And when** it is read with a descendant of company one in scope, it is read in that
descendant.

### M8. A bill in a foreign currency and the price difference

**Given** a product using standard price with a cost of 9.00 in the company currency,
perpetual valuation, anglo-saxon accounting, and a vendor bill in a foreign currency whose
rate on the line's date is 2,

**Then** the comparison price is

```formula
convert( 9.00, company currency → document currency, at the line date, without rounding ) = 18.00
```

and the difference is computed in the document currency against the bill's own unit
price, while the resulting journal item balances are converted back into the company
currency **at today's rate**.

---

## Group N — Manufacturing

### N1. The cost of a finished good

**Given** a manufacturing order producing 10 units of a finished good using average cost,
consuming three component movements valued at 78.00, 180.00 and 20.00, with work orders
costing 200.00 in total, no extra unit cost and no by-products,

**Then**:

```formula
total_cost          = 78.00 + 180.00 + 20.00 + 200.00 = 478.00
finished_unit_price = 478.00 × 1.0000 ÷ 10 = 47.80
```

**And** the finished-goods movement is valued, through the production source, at
10 × 47.80 = **478.00**.

### N2. The same with a by-product

**Given** the same order also producing 5 units of a by-product carrying a cost share of
25 percent, using average cost,

**Then**:

```formula
by_product_unit_price = 478.00 × 25 ÷ 100 ÷ 5 = 23.90
finished_unit_price   = 478.00 × round_half_up( 0.75, 0.0001 ) ÷ 10 = 35.85
```

**And** the finished goods take 358.50 and the by-product 119.50, together 478.00.

### N3. A by-product using standard price

**Given** the same order with a by-product using **standard price** whose unit cost is
20.00,

**Then** the by-product's unit price is set to **20.00**, not to a share of the total
cost, but its cost share is still subtracted from the finished goods' share.

### N4. A by-product whose cost share rounds to zero

**Given** a by-product using average cost with a cost share that rounds to zero at two
decimals,

**Then** its unit price is **left unchanged** and it is skipped, while its (zero) cost
share is still accumulated.

### N5. The extra unit cost

**Given** the order of N1 with an extra unit cost of 2.00,

**Then**:

```formula
extra_cost          = 2.00 × 10 = 20.00
total_cost          = 478.00 + 20.00 = 498.00
finished_unit_price = 49.80
```

**And** a back order created from the order inherits the extra unit cost of 2.00.

### N6. Component consumption and finished goods under perpetual valuation

**Given** perpetual valuation, the production location carrying a cost-of-production
account, the order of N1,

**Then** each component movement produces an entry debiting the cost-of-production
account and crediting the component's inventory valuation account by the component's
value,

**And** the finished-goods movement produces an entry debiting the finished good's
inventory valuation account by 478.00 and crediting the cost-of-production account by
478.00,

**And** the labour entry (see N7) credits the work centre and product expense accounts by
200.00 in total and debits the cost-of-production account by 200.00,

**And** the cost-of-production account nets to

```formula
78.00 + 180.00 + 20.00 + 200.00 − 478.00 = 0.00
```

### N7. The labour entry

**Given** a completed manufacturing order with two work orders: one on a work centre
carrying its own expense account, costing 120.00; one on a work centre with no expense
account, costing 80.00, whose finished product's expense account is used instead; the
production location carrying a cost-of-production account; perpetual valuation,

**When** the order completes,

**Then** one journal entry is produced in the product's inventory journal, dated today,
referenced **"_the order name_ - Labour"**, containing:

| Account | Debit | Credit |
|---|---|---|
| Cost of production | 200.00 | |
| Work centre expense | | 120.00 |
| Product expense | | 80.00 |

**And** each item except the last is written onto the time records of the work orders
that used that account, so that a second completion does not post the labour again.

### N8. The labour entry is skipped

**Given** a manufacturing order whose production location carries **no** valuation
account, or whose finished product uses periodic valuation, or whose total work-centre
cost is zero, or whose time records already carry a journal item,

**When** it completes,

**Then** **no** labour entry is produced.

### N9. The work-in-progress entry

**Given** two manufacturing orders in progress, whose consumed component lines are worth
1 200.00 at their unit costs and whose work orders have recorded 300.00 of labour,

**When** the work-in-progress wizard is confirmed with today's posting date and
tomorrow's reversal date,

**Then** one entry is posted:

| Account | Debit | Credit |
|---|---|---|
| Production work in progress | 1 500.00 | |
| Inventory valuation (company fallback) | | 1 200.00 |
| Production work-in-progress overhead | | 300.00 |

**And** one reversal is posted, dated tomorrow, referenced **"Reversal of: _the original
reference_"**, with the sides swapped,

**And** both entries carry the two manufacturing orders.

### N10. Work-in-progress guards

**Given** a work-in-progress wizard whose lines do not balance,

**When** it is confirmed,

**Then** the operation fails with **"Please make sure the total credit amount equals the
total debit amount."**

**And given** a wizard whose reversal date is not after the posting date,

**When** it is confirmed,

**Then** the operation fails with **"Reversal date must be after the posting date."**

**And given** a wizard line carrying both a debit and a credit,

**Then** the database constraint fails with **"A single line cannot be both credit and
debit."**

### N11. Work-in-progress order selection

**Given** five manufacturing orders selected, of which two are `draft` and three are
`confirmed`, `progress` and `to_close`,

**When** the wizard is opened,

**Then** only the three qualifying orders are carried, and the reference reads
**"Manufacturing WIP - _their three names_"**.

**And when** none qualifies, the reference reads **"Manufacturing WIP - Manual Entry"**.

### N12. A kit product is not valued

**Given** manufacturing accounting installed and a product that is a kit,

**Then** its total value and average cost are **0.00**, and it is excluded from the
valuation product domain of the closing.

### N13. Cost of production on the inventory valuation report

**Given** manufacturing accounting installed and at least one location of usage
`production` carrying a valuation account,

**When** the inventory valuation report is loaded,

**Then** it contains a **Cost of Production** section built from the location
reclassification computation restricted to production locations, whose total is the
negation of the total debit.

---

## Group O — Reporting

### O1. The unit cost history of an average-cost product

**Given** a product using average cost on which the following happened in order: receive
10 at 10.00; receive 10 at 15.00; deliver 15; a manual cost change to 14.00,

**Then** the report shows four rows, oldest first for the running replay:

| Row | Resource | Quantity | Value | Added value | Total quantity | Total value | Running unit cost |
|---|---|---|---|---|---|---|---|
| receipt 1 | movement | +10 | 100.00 | 100.00 | 10 | 100.00 | 10.00 |
| receipt 2 | movement | +10 | 150.00 | 150.00 | 20 | 250.00 | 12.50 |
| delivery | movement | −15 | −187.50 | −187.50 | 5 | 62.50 | 12.50 |
| cost change | valuation history | 0 | 14.00 | 14.00 × 5 − 62.50 = 7.50 | 5 | 70.00 | 14.00 |

**And** the rows are displayed newest first, but the running figures are computed oldest
first over **all** rows of the product and company, not only the rows on the page.

### O2. The unit cost history excludes standard-price products' movements

**Given** a product using standard price,

**Then** the report shows **only** the "Adjustment" rows coming from the valuation
history; no movement rows appear.

### O3. The unit cost history uses the reference unit

**Given** a receipt of 1 dozen of a product whose reference unit is the unit,

**Then** the quantity shown on that row is **12**, not 1.

### O4. The unit cost history after a costing method change

**Given** a product whose category's costing method changes from `standard` to `average`,

**Then** movement rows begin to appear from that moment on, because the view's condition
reads the category's current costing method for the company.

### O5. The report row identifiers do not collide

**Given** a movement with identifier 42 and a valuation history record with identifier 42,

**Then** the movement row's identifier is **42** and the valuation history row's is
**−42**.

### O6. The inventory valuation report with no inventory-loss account

**Given** no location of usage `inventory` carrying a valuation account,

**When** the report is loaded,

**Then** the **Inventory Loss** section is **absent** from the report data.

### O7. The inventory valuation report at a past date

**Given** the report loaded with a date equal to today,

**Then** the date is treated as "no date" and everything is computed as of now.

**And given** a date in the past, every figure — physical value, ledger balance,
reclassification and variation — is computed as of that date.

### O8. Generate Entry passes the date only when it differs

**Given** the report loaded with today's date,

**When** "Generate Entry" is pressed,

**Then** the closing operation is called **without** a date.

**And given** a past date, the operation is called **with** it.

### O9. The forecast report's value

**Given** a product with quantity records in a warehouse worth 500.00 in total, read by an
inventory manager,

**Then** the forecast report header shows the value formatted with the company currency's
decimal places and its symbol, placed before or after the number according to the
currency's position setting.

**And given** a reader who is not an inventory manager, the value is absent from the
header.

### O10. The valuation list hides the right columns

**Given** the valuation list opened from a product using **first in first out**,

**Then** the Value and Value Description columns are shown and the Unit Cost column is
hidden.

**And given** a product using average cost or standard price, the Value and Value
Description columns are hidden and the Unit Cost column is shown.

### O11. The lot column of the valuation list

**Given** the valuation list opened from an untracked product,

**Then** the Lots column is hidden.

### O12. The remaining-quantity filter

**Given** the "Remaining" filter applied on the goods movement list with a default product
in the context,

**Then** the list shows exactly the movements appearing in that product's remaining-movement
map, across every company in scope.

**And when** the filter is applied with an unsupported operator, the operation fails with
**"Only is set (= True) is supported in search for remaining_qty."**

---

## Group P — Closing behaviour

### P1. Nothing to close

**Given** a company whose physical value and ledger balance agree and which has no
location reclassification to make,

**When** the closing is run by a human,

**Then** it fails with **"Everything is correctly closed"**.

**And when** it is run by the scheduled job, it returns silently.

### P2. A missing journal

**Given** a company with work to close and no inventory journal,

**When** the closing is run,

**Then** it fails with **"Please set the Journal for Inventory Valuation in the
settings."**

### P3. A missing valuation account

**Given** a company with work to close, an inventory journal, and no inventory valuation
account,

**When** the closing is run,

**Then** it fails with **"Please set the Valuation Account for Inventory Valuation in the
settings."**

### P4. Closing before the last closing

**Given** a company whose last posted closing is dated the thirty-first of March,

**When** a closing is requested for the fifteenth of March,

**Then** it fails with **"It exists closing entries after the selected date. Cancel them
before generate an entry prior to them"**.

### P5. An account with no variation account and no company expense account

**Given** an inventory valuation account carrying no variation account, and a company with
no default expense account,

**When** the closing is run,

**Then** that account is **skipped entirely**: no pair is produced for it, and its ledger
balance is left as it is.

### P6. An account with no variation account but a company expense account

**Given** the same account but a company that does have a default expense account,

**Then** the pair is produced with the company's default expense account as the
counterpart.

### P7. Two closings on the same day

**Given** a closing posted at 09:00 today,

**When** movements happen and a second closing is run at 17:00 the same day,

**Then** the anchor is **09:00 today**, not midnight, so only the movements made since
09:00 are considered by part one.

### P8. A draft closing is not an anchor

**Given** a closing entry created but left in draft,

**When** the next closing is run,

**Then** the anchor walk skips it and lands on the previous posted closing (or on no
anchor at all), so the same period is recomputed.

### P9. The closing register is capped

**Given** a company with ten closing entries already registered,

**When** an eleventh closing is created,

**Then** the register holds ten identifiers and the **oldest** has been dropped.

### P10. A location reclassification with movements in both directions

**Given** an inventory-loss location carrying a loss account, through which goods worth
80.00 left and goods worth 30.00 came back since the last closing, for products under
periodic valuation,

**Then** part one produces one pair for

```formula
balance = 80.00 − 30.00 = 50.00
```

| Account | Debit | Credit |
|---|---|---|
| Inventory loss | 50.00 | |
| Inventory valuation | | 50.00 |

### P11. A location reclassification whose balance nets to zero

**Given** the same amounts in both directions,

**Then** the balance is exactly 0.00 and **no** pair is produced — the test is an exact
comparison to zero, not a currency-rounded one.

### P12. The continental perpetual period variation

**Given** a company using **perpetual** valuation **without** anglo-saxon accounting, an
inventory valuation account carrying both a variation account and a closing expense
account, whose posted ledger balance today is 900.00 and at the fiscal year start was
700.00, with no existing balance on the variation account and no extra balance from parts
one and two,

**Then** part three produces

```formula
balance_over_period = ( 900.00 − 0.00 ) − 700.00 + 0.00 = 200.00
```

| Account | Debit | Credit |
|---|---|---|
| Closing expense account | 200.00 | |
| Variation account | | 200.00 |

### P13. Part three is skipped

**Given** an inventory valuation account carrying a variation account but **no** closing
expense account,

**Then** part three skips that account entirely.

---

## Group Q — Locking and dates

### Q1. Back-dating a completed transfer into a locked fiscal year

**Given** a company whose fiscal-year lock date is the first of January of a past year,
and a completed transfer,

**When** its completion date is written to a date **before** that lock,

**Then** the write fails with **"You cannot modify the scheduled date of operation _the
transfer display name_ because it falls within a locked fiscal period."**

**And when** it is written to a date after the lock, the write succeeds.

### Q2. The sale, purchase and tax locks do not apply

**Given** a company with the sale, purchase and tax lock dates set to the first of January
of a past year, and no fiscal-year lock and no hard lock,

**When** a completed transfer's completion date is written to a date before those locks,

**Then** the write **succeeds** — those three locks are explicitly excluded.

### Q3. The hard lock applies

**Given** a company whose hard lock date is set,

**Then** the behaviour is the same as Q1.

### Q4. A non-completed transfer's planned date

**Given** any lock configuration,

**When** the **planned** date of a transfer that is not yet completed is written to a date
inside the locked period,

**Then** the write succeeds — only the completion date is constrained.

### Q5. The bypass parameter

**Given** the parameter that suppresses the lock check set to a truthy value,

**When** a completed transfer's completion date is written into a locked period,

**Then** the write succeeds and no check is made.

### Q6. Editability of a completed transfer's date

**Given** a completed transfer whose completion date falls inside a locked period,

**Then** the "is the date editable" computation returns false and the interface prevents
the edit before the constraint fires.

---

## Group R — Costing method and configuration changes

### R1. Changing a category's costing method to average

**Given** a category using standard price with products holding stock, changed to
`average`,

**When** the change is saved,

**Then** every product of the category has its unit cost recomputed by a forced average
replay, every lot of every lot-valuated product is recomputed, **no** valuation history
record is created and **no** journal entry is produced.

### R2. Changing a category's costing method to first in first out

**Given** a category changed to `fifo`, and a product of that category with 20 units on
hand worth 220.00,

**Then** the product's unit cost becomes 220.00 ÷ 20 = **11.00**.

**And given** a product of that category with nothing on hand but a last incoming movement
worth 120.00 for 10 units, its unit cost becomes **12.00**.

**And given** a product with nothing on hand and no incoming movement, its unit cost is
left unchanged.

### R3. Changing a category's costing method to standard price

**Given** a category changed to `standard`,

**Then** every product's unit cost is left exactly as it was and stops being recomputed.

### R4. Moving a product to a category with a different costing method

**Given** a product in a category using `standard` moved to a category using `average`,

**When** the write is saved,

**Then** the product is detected before the write (its current method differs from the new
category's) and its unit cost is recomputed after the write.

### R5. Moving a product to no category

**Given** a product moved to no category at all,

**Then** the comparison uses the **company's** fallback costing method instead of a
category's.

### R6. Switching a category from periodic to perpetual

**Given** a category using periodic valuation with goods on hand and a ledger balance of
0.00 on its inventory valuation account,

**When** the valuation mode is changed to `real_time`,

**Then** **no** retroactive entry is produced; movements completed from now on post their
entries; and the accumulated difference is picked up by the next closing.

### R7. Filtering products by valuation mode

**Given** a mixture of products,

**When** a filter "valuation mode equals `real_time`" is applied,

**Then** it matches: products whose category carries `real_time` for the current company;
plus products whose category carries nothing (or that have no category) and whose owning
company carries `real_time`; plus, when the current company itself carries `real_time`,
products with no owning company.

**And when** the filter uses any other operator, it fails with **"You can only use the '='
operator to search on valuation field."**

**And when** it uses any other value, it fails with **"Only the value 'periodic' and
'real_time' are accepted to search on valuation field."**

---

## Group S — Edge cases and robustness

### S1. Deleting the valuation history of a product

**Given** two products using average cost, each with receipts, and the valuation history
of both deleted,

**When** their total value is asked for as of a past date,

**Then** the average replay finds no anchor and replays from the beginning of time,
producing the correct figures rather than wrong ones.

**Worked example.** Product one receives 10 at 10.00 five days ago, has its cost written
to 20.00 four days ago, and receives 10 at 20.00 three days ago. Product two receives 10
at 10.00 five days ago and 10 at 20.00 three days ago. With the history of both deleted
and the valuation asked for two days ago, the results are **400.00** for product one and
**300.00** for product two.

### S2. A batch where only some products have an anchor

**Given** three products valued together, of which two have a valuation history anchor and
one does not,

**Then** the movement filter is **not** narrowed to "not before the oldest anchor",
because the narrowing requires every product of the batch to have one.

### S3. Adding a line to an already-completed incoming movement

**Given** a completed receipt of 10 units at 10.00,

**When** a further line of 1 unit is created on it,

**Then** the movement is **fully re-valued** from the priority chain with the new valued
quantity of 11.

### S4. Editing a line quantity on an already-completed outgoing movement

**Given** a completed delivery of 10 units valued at 100.00,

**When** a line's quantity is edited so that the total becomes 12,

**Then** the value is **scaled**, not recomputed:

```formula
correction = +2, previous quantity = 10, ratio = 0.2
new value  = 100.00 + 0.2 × 100.00 = 120.00
```

**And when** it is edited down to 8:

```formula
correction = −2, previous quantity = 10, ratio = −0.2
new value  = 100.00 − 20.00 = 80.00
```

### S5. A movement created directly in the completed state

**Given** an already-completed transfer,

**When** a movement is created on it in the completed state,

**Then** the creation hook values the outgoing ones and builds the valuation entries for
all of them; the incoming ones are valued when their lines are created.

### S6. A movement whose product has no cost and no source of value

**Given** a product with a unit cost of 0.00, no purchase order, no bill and no
originating movement,

**When** a receipt of 10 units is validated,

**Then** the movement is valued at **0.00** and the goods enter stock at no value.

### S7. Reverting a movement out of the completed state

**Given** a completed movement with a value of 100.00,

**When** its state is reverted,

**Then** the three classification flags become false, but the stored value remains
100.00; nothing reads it while the flags are false.

### S8. A movement with lines crossing in both directions

**Given** a movement with one line bringing goods in from the vendor location and one line
taking goods out to the customer location, both picked,

**Then** both the incoming and the outgoing flags are true; the valued quantity is the
**incoming** quantity; and the movement's value is set by the incoming branch.

### S9. A zero-quantity movement

**Given** a movement whose quantity rounds to zero at the precision of its unit of
measure,

**Then** no journal entry is produced for it, whatever its flags and its value.

### S10. The landed-cost residue with several cost lines

**Given** a landed cost document with two cost lines, each split across the same three
movements with a rounding residue,

**Then** both residues are attached to the adjustment line with the **highest identifier
accumulated so far in the whole computation**, which may belong to the first cost line,

**And** the overall sum check still passes, while the per-cost-line check may not — which
is why recomputing is offered explicitly.

### S11. A landed cost whose movement went negative

**Given** an adjustment line of 60.00 on a movement whose remaining quantity is −10 out of
a line quantity of 30,

**Then**:

```formula
posted_amount = 60.00 × ( −10 ÷ 30 ) = −20.00   → negative, so the sides swap
```

| Account | Debit | Credit |
|---|---|---|
| Cost line counterpart | 20.00 | |
| Inventory valuation | | 20.00 |

### S12. Adding a product to an invoice line whose display type is the injected one

**Given** an injected item,

**When** the generic product onchange would run,

**Then** it is **not** applied to items whose display type is `cogs`.

### S13. Changing the currency of an invoice carrying injected items

**Given** an invoice carrying injected items,

**When** its currency changes,

**Then** the injected items are **excluded** from the recomputation.

### S14. A reversal posted to cancel another entry

**Given** a reversal created explicitly to cancel another entry,

**When** it is posted,

**Then** **no** cost-of-goods-sold items and **no** price-difference items are injected,
and copying preserves the existing injected items rather than dropping them.

### S15. An inventory user validating a delivery of an average-cost product

**Given** a user in the inventory user group only, and a product using average cost with
perpetual valuation,

**When** the user validates a delivery,

**Then** the validation succeeds: the valuation, the unit-cost write and the journal entry
all run with elevated privileges.

### S16. A user with limited access writing a unit cost

**Given** a user without accounting access,

**When** a unit-cost write happens through a flow that user can reach,

**Then** the valuation history record is created with elevated privileges and the write
succeeds.

### S17. The value of a stock quantity record when the reference quantity is zero

**Given** a quantity record holding 5 units of a product whose scoped quantity on hand
computes to zero,

**Then** the record's value is **0.00** — the division is guarded.

### S18. Two receipts at the same instant under first in first out

**Given** two receipts of the same product with identical dates,

**Then** the stack orders them by **identifier**, and the one with the lower identifier is
consumed first.

### S19. Two valuation history records at the same instant

**Given** two records for the same product at the same instant,

**Then** the one with the **higher identifier** wins.

### S20. A movement of the same purchase order line dated the same instant

**Given** two movements of one purchase order line with identical dates, one being valued,

**Then** the other counts as "already absorbing" the bill only when its identifier is
**lower** than the one being valued.

---

## Group T — End-to-end conformance runs

### T1. Perpetual anglo-saxon, first in first out, purchase to sale

**Given** a company using perpetual valuation and anglo-saxon accounting, a product using
first in first out with its inventory valuation account and expense account set, the
inventory-loss location carrying a loss account,

**When** the following happen in order:

| Step | Event |
|---|---|
| 1 | a purchase order for 10 units at 10.00 is confirmed |
| 2 | the receipt of 10 units is validated |
| 3 | the vendor bill for 10 units at 10.00 is posted |
| 4 | 6 units are delivered |
| 5 | the customer invoice for 6 units at 20.00 is posted |

**Then** the ledger holds:

| Step | Account | Debit | Credit |
|---|---|---|---|
| 2 | — | — | — (neither location carries a valuation account) |
| 3 | Inventory valuation | 100.00 | |
| 3 | Accounts payable | | 100.00 |
| 4 | — | — | — |
| 5 | Accounts receivable | 120.00 | |
| 5 | Product sales | | 120.00 |
| 5 | Cost of goods sold | 60.00 | |
| 5 | Inventory valuation | | 60.00 |

**And** the inventory valuation account's balance is 40.00, which equals the product's
total value: 4 units still on hand at 10.00.

### T2. Periodic, average cost, purchase to sale with a closing

**Given** a company using periodic valuation, a product using average cost,

**When**:

| Step | Event |
|---|---|
| 1 | a receipt of 10 units at 10.00 is validated |
| 2 | a receipt of 10 units at 12.00 is validated |
| 3 | 5 units are delivered |
| 4 | the closing is run and posted |

**Then** nothing is posted at steps 1 to 3,

**And** the closing entry is:

| Account | Debit | Credit |
|---|---|---|
| Inventory valuation | 165.00 | |
| Inventory variation | | 165.00 |

**And** the bills, if any, debited the expense account, so the period's cost of goods is
the purchases less the inventory variation, as the continental convention requires.

### T3. Perpetual, standard price, with a price difference and a landed cost

**Given** a company using perpetual valuation and anglo-saxon accounting, a product using
standard price with a unit cost of 9.00 and a price difference account set on its
category, a service product "Freight" flagged as a landed cost,

**When**:

| Step | Event |
|---|---|
| 1 | a purchase order for 10 units at 10.00 is confirmed |
| 2 | the receipt is validated |
| 3 | the vendor bill for 10 units at 10.00 plus a Freight line of 20.00 is posted |
| 4 | a landed cost is created from the bill, targeting the receipt, split equally |

**Then** at step 3 the ledger holds:

| Account | Debit | Credit |
|---|---|---|
| Inventory valuation | 100.00 | |
| Freight expense | 20.00 | |
| Accounts payable | | 120.00 |
| Price difference | 10.00 | |
| Inventory valuation | | 10.00 |

**And** at step 4 the landed cost document is created with one cost line of 20.00; but
the split computation finds **no eligible movement**, because the product uses standard
price, and fails with **"You cannot apply landed costs on the chosen Transfers(s). Landed
costs can only be applied for products with FIFO or average costing method."**

**And** the inventory valuation account's net balance is 90.00, matching the product's
total value of 10 × 9.00.

### T4. The complete negative-stock round trip under periodic valuation

**Given** a company using periodic valuation and a product using first in first out with a
unit cost of 8.00,

**When** the sequence of scenario D3 is performed,

**Then** at every step the posted balance of the inventory valuation account equals the
product's total value, and the expense account is never touched, and the variation account
holds the negation of the inventory valuation balance.

### T5. A full landed cost round trip

**Given** the state at the end of G1,

**When** the reversal of G10 is validated and the closing is then run,

**Then** the inventory valuation account's balance and the products' total values are back
to where they were before G1, and the freight expense account holds 0.00.
