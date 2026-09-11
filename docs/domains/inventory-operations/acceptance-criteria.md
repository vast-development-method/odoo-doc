# Inventory Operations — Acceptance Criteria

Numbered scenarios in Given / When / Then form, with concrete numbers. An implementation that satisfies every scenario of this file behaves identically to the specification for the cases the scenarios cover.

> **Reproduced literals.** A few strings in this file are reproduced exactly as the system emits them — error messages, selection labels, generated record names — and therefore keep abbreviations that this specification would otherwise spell out. They are: `UoM` for unit of measure, `SN` for serial number, `ZPL` for the Zebra printer command language, `PDF` for Portable Document Format, `GS1` for Global Standards One, and the suffix `(MTO)` for make to order, that is the supply method this specification calls *advanced* or *trigger another rule*. Wherever such a string is quoted, the quotation is verbatim and must be reproduced character for character.

## Shared fixture

Unless a scenario says otherwise, assume:

- One company, **Acme**, whose currency has two decimal places.
- The decimal-precision setting named `Product Unit` has **2** digits.
- One Warehouse, short name `WH`, receiving in one step and delivering in one step. Its Locations are `WH` (virtual), `WH/Stock` (internal), `WH/Input` (internal, inactive), `WH/Quality Control` (internal, inactive), `WH/Output` (internal, inactive), `WH/Packing Zone` (internal, inactive).
- Shared Locations `Vendors` (vendor usage), `Customers` (customer usage), `Inter-company transit` (transit usage, inactive), and the company's `Inventory adjustment` (inventory-loss usage), `Production` (production usage) and `Scrap` (inventory-loss usage).
- Operation Types `WH: Receipts` (receipt, sequence prefix `IN`), `WH: Delivery Orders` (delivery, prefix `OUT`), `WH: Internal Transfers` (internal, prefix `INT`), `WH: Pick` (`PICK`), `WH: Pack` (`PACK`), `WH: Quality Control` (`QC`), `WH: Storage` (`STOR`), `WH: Cross Dock` (`XD`).
- Product **Bolt**: storable, untracked, unit of measure "Units" with a rounding step of 0.01, weight 0.5, volume 0.001.
- Product **Drill**: storable, tracked by serial number, unit "Units".
- Product **Paint**: storable, tracked by lot, unit "Units".
- The multi-location group, the lot group and the container group are all active.
- All the other settings are at their shipped defaults.

Every assertion about a quantity is an assertion about the value **after** rounding at the stated precision.

---

# A. Receipts

## Scenario 1 — One-step receipt, happy path

**Given** the Warehouse receives in one step
**And** there is no stock of Bolt anywhere

**When** an inventory user creates a Transfer with Operation Type `WH: Receipts`, contact "Vendor V", and one move of **10** Bolt
**And** confirms it
**And** sets the processed quantity to 10 and validates it

**Then** the Transfer's reference is `WH/IN/00001`
**And** its source Location is `Vendors` and its destination Location is `WH/Stock`
**And** immediately after confirmation the move's status is `assigned` and the Transfer's status is `assigned`, because the source Location bypasses reservation
**And** after validation the Transfer's status is `done`, its completion date is the instant of validation, and its priority is `0`
**And** exactly one Stock Quantity record exists for (Bolt, `WH/Stock`, no lot, no container, no owner) with an on-hand quantity of **10.00**, a reserved quantity of **0.00** and an incoming date equal to the validation instant
**And** one Stock Quantity record exists for (Bolt, `Vendors`) with an on-hand quantity of **−10.00**
**And** no backorder Transfer exists
**And** no journal entry is produced by this domain.

## Scenario 2 — Two-step receipt

**Given** the Warehouse is reconfigured to receive in **two steps**
**Then** `WH/Input` becomes active, the `WH: Storage` Operation Type becomes active, and the receipt Route is rewritten to contain exactly two rules: a pull rule from `Vendors` to `WH/Stock` through `WH: Receipts` with the take-from-stock supply method and the propagate-cancel flag set, and a push rule from `WH/Input` to `WH/Stock` through `WH: Storage` with the advanced supply method and the propagate-cancel flag **cleared**
**And** the default destination of `WH: Receipts` becomes `WH/Input`
**And** the default source of `WH: Storage` becomes `WH/Input`.

**When** an inventory user receives **10** Bolt through `WH/IN/00002` and validates it

**Then** 10.00 Bolt sit in `WH/Input`
**And** a second Stock Move exists with source `WH/Input`, destination `WH/Stock`, demand 10, Operation Type `WH: Storage`, supply method advanced, and the receipt move recorded as its originating move
**And** that move belongs to a new Transfer numbered `WH/STOR/00001`
**And** that move's status is `assigned`, because the completion of the receipt re-reserved its destination moves against exactly the goods that arrived
**And** the detail line of the storage move takes from `WH/Input` and puts into `WH/Stock`.

**When** the user validates `WH/STOR/00001`

**Then** 10.00 Bolt sit in `WH/Stock`, `WH/Input` holds 0.00, and both Transfers are `done`.

## Scenario 3 — Three-step receipt

**Given** the Warehouse is reconfigured to receive in **three steps**
**Then** `WH/Input` and `WH/Quality Control` are active, `WH: Quality Control` and `WH: Storage` are active, and the receipt Route contains exactly three rules: pull `Vendors` → `WH/Stock` through `WH: Receipts` (take-from-stock, propagate cancel set); push `WH/Input` → `WH/Quality Control` through `WH: Quality Control` (advanced, propagate cancel set); push `WH/Quality Control` → `WH/Stock` through `WH: Storage` (advanced, propagate cancel **cleared**).

**When** 10 Bolt are received and each of the three Transfers is validated in turn

**Then** three Transfers exist, numbered `WH/IN/...`, `WH/QC/...` and `WH/STOR/...`
**And** the three moves form a chain, each the originating move of the next
**And** the goods end in `WH/Stock`.

## Scenario 4 — Cancelling the first step of a three-step receipt

**Given** the three-step receipt of scenario 3 has been validated up to `WH/QC/...`, which is still open

**When** the user cancels the receipt move — which is impossible because it is done

**Then** the operation fails with "You cannot cancel a stock move that has been set to 'Done'. Create a return in order to reverse the moves which took place."

**Given** instead that the receipt is still open

**When** the user cancels the receipt Transfer

**Then** the receipt move is cancelled
**And** because its rule sets the propagate-cancel flag, the quality control move is cancelled too
**And** because the quality control rule has that flag cleared, the storage move is **not** cancelled; instead its supply method is reset to take-from-stock and its link to the quality control move is dropped.

## Scenario 5 — Receipt of three serial-numbered units

**Given** Drill is tracked by serial number
**And** `WH: Receipts` allows creating new lots and does not allow using existing ones

**When** an inventory user creates `WH/IN/00003` with one move of **3** Drill
**And** confirms it
**And** uses the serial-number generation with the first name `SN00007` and a count of 3
**And** validates the Transfer

**Then** the generation produces exactly the names `SN00007`, `SN00008`, `SN00009`
**And** the move has exactly three detail lines, each with a quantity of **1** in the product unit and one of those typed names
**And** at completion three Lot records are created, one per name, each linked to Drill and to the company
**And** three Stock Quantity records exist in `WH/Stock`, one per serial number, each with an on-hand quantity of **1.00**
**And** the serial-number uniqueness constraint holds: no serial number appears twice.

**When** the user instead types a first name of `DRILL-A` with a count of 2

**Then** the names produced are `DRILL-A0` and `DRILL-A1`, because a first name with no digits has a zero appended before the series is built.

## Scenario 6 — Serial number already in stock

**Given** serial number `SN00007` of Drill is on hand in `WH/Stock`

**When** a person types `SN00007` again on a receipt line

**Then** a warning is shown: "The Serial Number (SN00007) is already used in location(s): WH/Stock.\n\nIs this expected? For example, this can occur if a delivery operation is validated before its corresponding receipt operation is validated. In this case the issue will be solved automatically once all steps are completed. Otherwise, the serial number should be corrected to prevent inconsistent data."
**And** the entry is **not** blocked; only validation of a resulting quantity above one in the same Location tree fails, with "The serial number has already been assigned: \n Product: Drill, Serial Number: SN00007".

## Scenario 7 — Missing lot at validation

**Given** `WH/IN/00004` has one move of 5 Paint with a processed quantity of 5 and no lot on its detail line
**And** `WH: Receipts` allows creating lots

**When** the user validates

**Then** the validation fails with "You need to supply a Lot/Serial number for products PAINT."

**Given** instead that both lot switches of the Operation Type are turned **off**

**When** the user validates

**Then** the validation succeeds and 5.00 Paint arrive in `WH/Stock` with **no** lot.

## Scenario 8 — Over-processing a receipt

**Given** `WH/IN/00005` demands **10** Bolt

**When** the user records a processed quantity of **12** and validates

**Then** the move's demand stays **10** and its processed quantity is **12**
**And** no backorder question is asked and no backorder Transfer is created
**And** **12.00** Bolt arrive in `WH/Stock`
**And** the move's status before completion was `assigned`, because the processed quantity was greater than or equal to the demand.

---

# B. Deliveries

## Scenario 9 — One-step delivery, fully available

**Given** `WH/Stock` holds **20.00** Bolt with 0.00 reserved
**And** `WH: Delivery Orders` reserves at confirmation and its backorder policy is "ask"

**When** an inventory user creates `WH/OUT/00001` with one move of **8** Bolt and confirms it

**Then** the move is reserved: one detail line is created for 8 from `WH/Stock` to `Customers`
**And** the Stock Quantity record now shows on hand **20.00** and reserved **8.00**, so its available quantity is **12.00**
**And** the move's status is `assigned` and the Transfer's status is `assigned`.

**When** the user validates with a processed quantity of 8

**Then** the Transfer is `done`
**And** `WH/Stock` holds **12.00** Bolt with **0.00** reserved
**And** `Customers` holds **−8.00** Bolt
**And** no backorder is created.

## Scenario 10 — Three-step delivery

**Given** the Warehouse is reconfigured to deliver in **three steps**
**Then** `WH/Packing Zone` and `WH/Output` become active; `WH: Pick` and `WH: Pack` become active; the default destination of `WH: Pick` becomes `WH/Packing Zone`, the default destination of `WH: Pack` becomes `WH/Output`, and the default source of `WH: Delivery Orders` becomes `WH/Output`
**And** the delivery Route contains exactly three rules: pull `WH/Stock` → `Customers` through `WH: Pick` (take-from-stock, propagate carrier); push `WH/Packing Zone` → `WH/Output` through `WH: Pack` (advanced, propagate carrier); push `WH/Output` → `Customers` through `WH: Delivery Orders` (advanced, propagate carrier).

**Given** `WH/Stock` holds 20.00 Bolt

**When** a need for **8** Bolt at `Customers` reaches the rule engine

**Then** one pick move is created: source `WH/Stock`, intermediate destination `WH/Packing Zone`, final destination `Customers`, Operation Type `WH: Pick`, grouped into `WH/PICK/00001`
**And** it reserves 8.00 from `WH/Stock`.

**When** `WH/PICK/00001` is validated

**Then** 8.00 Bolt sit in `WH/Packing Zone`
**And** a pack move is created: source `WH/Packing Zone`, destination `WH/Output`, Operation Type `WH: Pack`, supply method advanced, the pick move as originating move, grouped into `WH/PACK/00001`, status `assigned`.

**When** `WH/PACK/00001` is validated

**Then** 8.00 Bolt sit in `WH/Output`
**And** a delivery move is created: source `WH/Output`, destination `Customers`, Operation Type `WH: Delivery Orders`, supply method advanced, the pack move as originating move, grouped into `WH/OUT/00002`, status `assigned`.

**When** `WH/OUT/00002` is validated

**Then** `WH/Output` holds 0.00, `Customers` holds −8.00, and the three Transfers are all `done`
**And** the three moves form one chain and share one set of document references.

## Scenario 11 — Shipping policy "when all products are ready"

**Given** `WH/Stock` holds 5.00 Bolt and 0.00 Paint
**And** a delivery Transfer demands 5 Bolt and 3 Paint
**And** its shipping policy is **all at once**

**When** it is confirmed and reserved

**Then** the Bolt move is `assigned` and the Paint move is `confirmed`
**And** the Transfer's status is **`confirmed`** ("Waiting"), because with the all-at-once policy the most important open move being `partially_available` or `confirmed` yields `confirmed`.

**Given** instead the shipping policy is **as soon as possible**

**Then** the Transfer's status is **`assigned`** ("Ready"), because the relevant status among the moves is `partially_available`, which the Transfer computation raises to `assigned`.

## Scenario 12 — Partial delivery **with** a backorder

**Given** `WH/Stock` holds **6.00** Bolt
**And** `WH/OUT/00003` demands **10** Bolt
**And** the Operation Type's backorder policy is "ask"

**When** the Transfer is confirmed and reserved

**Then** one detail line of 6 is created, the move's status is `partially_available` and the Transfer's status is `assigned`.

**When** the user validates

**Then** the backorder screen opens, because the picked quantity (6) is below the demand (10).

**When** the user chooses "Create backorder"

**Then** the original move's demand becomes **6** and its processed quantity is **6**, and it becomes `done`
**And** a new Stock Move of demand **4** is created with the same product, unit, supply method, chain links, unit price and deadline
**And** a new Transfer is created whose back-order link points at `WH/OUT/00003`, whose reference is `WH/OUT/00004`, whose responsible is empty, and which holds the 4-unit move with its picked flag cleared
**And** a note is posted on `WH/OUT/00003`: "The backorder WH/OUT/00004 has been created."
**And** because the Operation Type reserves at confirmation, the backorder's availability is checked immediately; with 0.00 Bolt left in `WH/Stock` the 4-unit move stays `confirmed`
**And** `WH/Stock` holds **0.00** Bolt and `Customers` holds **−6.00**.

## Scenario 13 — Partial delivery **without** a backorder

**Given** the same starting point as scenario 12

**When** the user validates and chooses "No backorder"

**Then** the original move keeps its demand of **10** and its processed quantity of **6**
**And** it becomes `done`
**And** **no** backorder move and **no** backorder Transfer are created
**And** `WH/Stock` holds 0.00 Bolt and `Customers` holds −6.00
**And** `WH/OUT/00003` is `done`.

**Given** instead that the Operation Type's backorder policy is **never**

**When** the user validates

**Then** no question is asked and the same result is obtained.

## Scenario 14 — Partial delivery with one line untouched

**Given** a delivery Transfer demands 10 Bolt and 4 Paint
**And** 10 Bolt are processed and 0 Paint are processed, and no move is explicitly picked

**When** the user validates and chooses "No backorder"

**Then** the pruning step of the completion algorithm cancels the Paint move, because its processed quantity is zero and backorders are forbidden
**And** the Bolt move completes
**And** the Transfer's status is `done`, because not every done move is a scrap.

## Scenario 15 — Splitting a transfer without validating

**Given** a delivery Transfer demands 10 Bolt and 6 Paint
**And** the processed quantities are set to 4 Bolt and 0 Paint

**When** the user presses the split action

**Then** the Bolt move is split: the original keeps a demand of 4 with a processed quantity of 4, and a new move of demand 6 is produced
**And** the new Bolt move **and** the whole Paint move are moved into a new backorder Transfer
**And** neither Transfer is validated; both remain open.

**Given** instead that every processed quantity is zero

**Then** the split fails with "*the transfer display name*: Nothing to split. Fill the quantities you want in a new transfer in the done quantities"

**Given** instead that every move is exactly fully processed

**Then** the split fails with "*the transfer display name*: Nothing to split, all demand is done. For split you need at least one line not fully fulfilled"

**Given** instead that one move is processed above its demand

**Then** the split fails with "*the transfer display name*: Can't split: quantities done can't be above demand"

---

# C. Reservation

## Scenario 16 — First in first out across two arrival batches

**Given** `WH/Stock` uses the first in first out removal strategy (no strategy is set anywhere, so the fallback applies)
**And** two Stock Quantity records exist for Bolt in `WH/Stock`, both with no lot, no container and no owner, created as two separate arrivals:

| Record | On hand | Reserved | Incoming date |
|---|---|---|---|
| Q1 | 40.00 | 0.00 | 3 March 2026, 08:00 |
| Q2 | 25.00 | 0.00 | 10 March 2026, 08:00 |

**When** a delivery move of **50** Bolt is reserved

**Then** the gathering returns Q1 before Q2
**And** the reservation takes **40.00** from Q1 and **10.00** from Q2
**And** Q1 shows on hand 40.00, reserved 40.00, available 0.00
**And** Q2 shows on hand 25.00, reserved 10.00, available 15.00
**And** the move is `assigned` with two detail lines, one per record, in that order.

**Given** instead that `WH/Stock` names the last in first out strategy

**Then** the gathering returns Q2 before Q1
**And** the reservation takes **25.00** from Q2 and **25.00** from Q1.

**Given** instead that the two records carry lots `L1` (3 March) and `L2` (10 March) and the strategy is last in first out

**Then** the first gathered record is the one with the **later** incoming date, that is `L2`.

## Scenario 17 — Merging two arrivals keeps the earliest incoming date

**Given** Paint lot `L1` is received into `WH/Stock` with an incoming date of **11 September 2026**, quantity 1

**When** the same lot is received again into the same Location with an explicit incoming date of **6 September 2026**, quantity 1

**Then** exactly **one** Stock Quantity record exists for (Paint, `WH/Stock`, `L1`)
**And** its on-hand quantity is **2.00**
**And** its incoming date is **6 September 2026** — the oldest of the candidate dates wins.

## Scenario 18 — Available quantity over several records

**Given** Bolt in `WH/Stock` has four Stock Quantity records:

| Record | On hand | Reserved |
|---|---|---|
| Q1 | 5.00 | 2.00 |
| Q2 | 10.00 | 12.00 |
| Q3 | 8.00 | 3.00 |
| Q4 | 35.00 | 12.00 |

**Then** the total on-hand quantity is **58.00**, the total reserved quantity is **29.00**, and the available quantity of the product at that Location is **29.00**
**And** the gathering returns all four records.

**When** a further **10.00** is reserved

**Then** the available quantity becomes **19.00**
**And** the four records still exist.

## Scenario 19 — Reservation blocked, then released

**Given** Bolt in `WH/Stock` has two Stock Quantity records:

| Record | On hand | Reserved |
|---|---|---|
| Q1 | 5.00 | 7.00 |
| Q2 | 12.00 | 10.00 |

**Then** the available quantity is **0.00** (5 − 7 = −2 and 12 − 10 = 2, and the sum is 0)

**When** a move of **10** Bolt asks to be reserved

**Then** **nothing** is reserved: the wanted quantity is clamped to the available quantity, which is 0
**And** the available quantity is still **0.00**
**And** the move's status stays `confirmed`.

**When** the document that held the 7.00 reservation on Q1 is cancelled, releasing it

**Then** Q1 shows on hand 5.00, reserved 0.00; Q2 shows on hand 12.00, reserved 10.00; the available quantity becomes **7.00**

**When** the blocked move of 10 is reserved again

**Then** **7.00** is reserved across the two records in the strategy order
**And** the move becomes `partially_available`
**And** its detail line or lines total 7.00.

**When** the document that held the remaining 10.00 on Q2 is also cancelled and the move is reserved a third time

**Then** the remaining **3.00** is reserved and the move becomes `assigned`.

## Scenario 20 — Unreserving

**Given** a move of 8 Bolt is `assigned` with one detail line of 8, not picked
**And** the Stock Quantity record shows on hand 20.00 and reserved 8.00

**When** the user presses unreserve

**Then** the detail line is deleted
**And** the record shows on hand 20.00 and reserved **0.00**
**And** the move's status returns to `confirmed`.

**Given** instead that the detail line is **picked**

**When** the user presses unreserve

**Then** the line survives, the reserved quantity stays 8.00, and the move's status is not recomputed.

**Given** instead that the move is `done` and its destination usage is not inventory loss

**Then** unreserving fails with "You cannot unreserve a stock move that has been set to 'Done'."

## Scenario 21 — Raising the demand of a reserved move

**Given** a move of 8 Bolt is `assigned` with 8 reserved
**And** the Transfer is unlocked

**When** the user raises the demand to **12**

**Then** the move is **not** unreserved (its processed quantity 8 is not greater than the new demand 12)
**And** its status becomes `partially_available`
**And** a note describing the change is posted on the Transfer.

**When** the user instead lowers the demand to **5**

**Then** the move **is** unreserved entirely, because its processed quantity 8 is greater than 5
**And** the reserved counter returns to 0.00
**And** the move's status is recomputed to `confirmed`.

## Scenario 22 — Reservation of a serial-tracked product refuses fractions

**Given** Drill is tracked by serial number and `WH/Stock` holds 3 units under three serial numbers

**When** a move asks to reserve **2.5** Drill

**Then** the reservation-quantity computation sets the wanted quantity to **0** and nothing is reserved, because the quantity is not a whole number.

## Scenario 23 — Reserve only full packagings

**Given** Bolt's product category asks for full-packaging reservation
**And** a move carries a packaging unit "Box of 6" whose conversion to the product unit is 6
**And** `WH/Stock` holds **20.00** Bolt free

**When** a move of **18** Bolt is reserved

**Then** the availability is first rounded **down** to whole boxes: the smaller of 18 and 20 is 18, which is exactly 3 boxes, so 18.00 is reserved.

**When** instead the move demands **20** Bolt

**Then** the availability is rounded down to 3 boxes, that is **18.00**, and only 18.00 is reserved; the move becomes `partially_available`.

## Scenario 24 — Least packages

**Given** Bolt in `WH/Stock` sits in three containers and loose:

| Container | Free quantity |
|---|---|
| P1 | 12.00 |
| P2 | 8.00 |
| P3 | 5.00 |
| (loose) | 3.00 |

**And** `WH/Stock` names the least-packages removal strategy

**When** a move of **13** Bolt is reserved

**Then** the container pre-selection finds the exact cover {P1, one loose unit} — two entries — rather than {P1, P3} (two entries but over-selecting by 4) or {P2, P3} (two entries, under by 0? no: 13 exactly, but examined later because the search opens the largest-first branch with the better estimate first)
**And** the gathering is restricted to the records of P1 and to one container-less record
**And** the reservation takes 12.00 from P1 and 1.00 loose.

## Scenario 25 — Closest location

**Given** Bolt sits in `WH/Stock/Shelf 1` (record identifier 91, 5.00 free) and `WH/Stock/Shelf 10` (record identifier 42, 5.00 free)
**And** `WH/Stock` names the closest-location strategy

**When** a move of 5 Bolt from `WH/Stock` is reserved

**Then** the records are ordered by full location name ascending — which is textual, so `WH/Stock/Shelf 1` comes before `WH/Stock/Shelf 10` — and the reservation takes from `WH/Stock/Shelf 1`.

**Given** both records sit in the **same** Location

**Then** the tie is broken by identifier **descending**, so record 91 is taken before record 42.

## Scenario 26 — Category strategy beats location strategy

**Given** `WH/Stock` names the last in first out strategy
**And** Bolt's product category names the first in first out strategy

**When** Bolt is reserved from `WH/Stock`

**Then** the **first in first out** ordering is used, because the product category is consulted before any Location.

## Scenario 27 — Strategy inherited from an ancestor location

**Given** `WH/Stock` names the last in first out strategy and `WH/Stock/Shelf 1` names none

**When** goods are reserved from `WH/Stock/Shelf 1`

**Then** the walk up the tree finds the strategy on `WH/Stock` and uses last in first out.

---

# D. Gathering

## Scenario 28 — Strict versus loose matching

**Given** Bolt sits in `WH/Stock/Shelf 1` with 4.00 and in `WH/Stock/Shelf 2` with 6.00, both without lot, container or owner

**When** the gathering is run **loosely** for (Bolt, `WH/Stock`)

**Then** both records are returned, because loose matching accepts descendants of the Location.

**When** the gathering is run **strictly** for (Bolt, `WH/Stock`)

**Then** **no** record is returned, because strict matching requires the Location to be exactly `WH/Stock`.

**When** the gathering is run strictly for (Bolt, `WH/Stock/Shelf 1`, no lot, no container, no owner)

**Then** exactly the 4.00 record is returned.

## Scenario 29 — Loose matching ignores unrequested characteristics

**Given** Paint sits in `WH/Stock` with 5.00 under lot `L1` and 2.00 with no lot

**When** the gathering is run loosely for (Paint, `WH/Stock`) with no lot requested

**Then** both records are returned, and the lot-bearing one comes **first**, because of the final "lots before no lot" sort.

**When** the gathering is run loosely for (Paint, `WH/Stock`, lot `L1`)

**Then** both records are still returned, because loose matching accepts a record with the requested lot **or with no lot**.

**When** the gathering is run strictly for (Paint, `WH/Stock`, lot `L1`)

**Then** both are again returned, because strict matching also accepts an empty lot; but when the available quantity is computed for a tracked product with strict matching and a requested lot, the lot-less record is **skipped**, so the available quantity is 5.00, not 7.00.

## Scenario 30 — Available quantity with a negative bucket

**Given** Paint in `WH/Stock` has: lot `A` on hand 10.00 reserved 4.00; lot `B` on hand 3.00 reserved 0.00; no lot, on hand −2.00 reserved 0.00

**When** the available quantity is asked for with negative results **allowed**

**Then** the answer is **7.00** (6 + 3 − 2).

**When** it is asked for with negative results **not allowed**

**Then** the answer is **9.00** (6 + 3, the negative bucket being dropped).

## Scenario 31 — The negative pocket during reservation

**Given** Bolt in `WH/Stock` has record A with on hand **−5.00** reserved 0.00 and record B with on hand **12.00** reserved 0.00, all other characteristics identical and empty

**When** a move of **10** Bolt is reserved

**Then** the available quantity is 7.00, so the wanted quantity is clamped to **7.00**
**And** record A is skipped, because its available quantity is not positive
**And** record B contributes **7.00** only: its 12.00 is first reduced by the 5.00 negative pocket of the shared key
**And** exactly one detail line of 7.00 is created
**And** the move becomes `partially_available`.

---

# E. Put-away

## Scenario 32 — Put-away to a shelf

**Given** `WH/Stock` has two internal children, `WH/Stock/Shelf A` and `WH/Stock/Shelf B`
**And** a Put-away Rule exists: when a product arrives in `WH/Stock`, store Bolt to `WH/Stock/Shelf A`, priority 10, sublocation mode "No", no storage category, no container type

**When** a receipt of **10** Bolt into `WH/Stock` is confirmed

**Then** the move's destination Location stays `WH/Stock`
**But** its detail line's destination Location is rewritten to **`WH/Stock/Shelf A`**

**When** the receipt is validated

**Then** the 10.00 Bolt are recorded in `WH/Stock/Shelf A`, not in `WH/Stock`.

## Scenario 33 — Put-away rule specificity

**Given** three Put-away Rules on `WH/Stock`, all with priority 10:
 - R1: no product, category "All", store to `WH/Stock/Shelf A`
 - R2: product Bolt, store to `WH/Stock/Shelf B`
 - R3: no product, category "All / Hardware" (Bolt's own category), store to `WH/Stock/Shelf C`
**And** none names a container type

**When** Bolt arrives in `WH/Stock`

**Then** the rules are sorted by the four-part specificity key, most specific first: R2 (names a product) comes first, then R3 (names the product's own category), then R1 (names an ancestor category)
**And** the chosen Location is **`WH/Stock/Shelf B`**.

**Given** instead that a fourth rule R4 names the container type "Pallet" and stores to `WH/Stock/Shelf D`
**And** the goods arrive inside a container of type "Pallet"

**Then** R4 wins over all three others, because naming a container type is the first element of the specificity key
**And** the chosen Location is `WH/Stock/Shelf D`.

## Scenario 34 — Storage capacity blocks a shelf

**Given** the Storage Category "Small shelf" has a maximum weight of **50** and a product capacity of **30** Bolt
**And** `WH/Stock/Shelf A` carries that category and already holds **22.00** Bolt
**And** Bolt weighs 0.5 per unit
**And** a Put-away Rule sends Bolt arriving in `WH/Stock` to `WH/Stock/Shelf A` with the closest-location mode and that storage category
**And** `WH/Stock/Shelf B` also carries the category and is empty

**When** **5** Bolt arrive

**Then** the capacity check for Shelf A runs: the forecasted weight is 22 × 0.5 = 11; the weight test `50 < 11 + 0.5 × 5 = 13.5` fails to trigger; the first quantity test `22 ≥ 30` is false; the second `5 + 22 = 27 > 30` is false; so Shelf A is **accepted**.

**When** instead **12** Bolt arrive

**Then** the second quantity test `12 + 22 = 34 > 30` is true, Shelf A is **rejected** and added to the rejected set, the walk continues, and `WH/Stock/Shelf B` is chosen.

**When** instead Shelf A already holds **30.00** Bolt and 1 unit arrives

**Then** the **first** quantity test `30 ≥ 30` already rejects it, even before the incoming quantity is considered.

## Scenario 35 — Mixing policy

**Given** the Storage Category "Single product" has the mixing policy "If all products are same"
**And** `WH/Stock/Shelf A` carries it and holds 4.00 Paint

**When** Bolt is put away there

**Then** the check fails, because a positive record exists whose product is not Bolt.

**Given** instead the policy is "If the location is empty" and the Shelf holds 4.00 of **any** product

**Then** the check fails.

**Given** instead the policy is "Allow mixed products"

**Then** neither test applies and only the weight and capacity tests decide.

## Scenario 36 — A container may not be split across shelves

**Given** a container `PACK0001` with no container type holds 10 Bolt and 10 Paint
**And** put-away rules would send Bolt to `WH/Stock/Shelf A` and Paint to `WH/Stock/Shelf B`

**When** the container's detail lines are pushed through the put-away application

**Then** the line-by-line resolution produces two different Locations
**And** because the group has an outermost container, the grouping is abandoned: **every** line of the group is reset to the move's own destination Location, `WH/Stock`.

## Scenario 37 — Put-away with a container type resolves one location for the whole group

**Given** the same container but with the container type "Pallet"
**And** a Put-away Rule naming the container type "Pallet" that sends to `WH/Stock/Shelf D`

**When** the lines are pushed through put-away

**Then** one Location is resolved for the whole group and written on **every** line: `WH/Stock/Shelf D`.

---

# F. Containers

## Scenario 38 — A whole package is moved

**Given** container `PACK0001` of type "Box" (not reusable) sits in `WH/Stock` and holds exactly 10.00 Bolt and nothing else
**And** a delivery Transfer demands 10 Bolt

**When** the Transfer is confirmed and reserved

**Then** one detail line of 10 is created with `PACK0001` as source container
**And** the whole-container detection runs: the lines of this single Transfer reproduce exactly the container's contents (grouping by product and lot, 10 equals 10, and no key is present on one side only)
**And** the container's type is not reusable
**And** therefore the line is given `PACK0001` as **destination container** and is flagged as an entire package.

**When** the Transfer is validated

**Then** a Package History snapshot is created first, recording `PACK0001`, its full name, its Location before (`WH/Stock`), its destination Location (`Customers`), its parent before and after, and the lines
**And** the 10.00 Bolt move from `WH/Stock` to `Customers` **with** `PACK0001` as their container in the destination
**And** the container's own Location becomes `Customers`
**And** the Transfer's container counter reads 1 and reads from the history records, because the Transfer is done.

## Scenario 39 — Adding an existing container to a transfer

**Given** container `PACK0002` in `WH/Stock` holds 4.00 Bolt and 2.00 Paint under lot `L1`
**And** an internal Transfer from `WH/Stock` to `WH/Stock/Shelf A` is open

**When** the user adds `PACK0002` through the add-entire-packages action

**Then** two detail lines are created, one per quantity record, each with `PACK0002` as **both** source and destination container, each flagged as an entire package, each with the record's own lot and owner
**And** put-away is applied to those lines
**And** any pre-existing line of the Transfer that already drew from `PACK0002` is deleted first.

## Scenario 40 — Container promotion

**Given** container `PALLET1` holds exactly two child containers, `BOX1` and `BOX2`, and its type is not reusable
**And** a Transfer takes over both `BOX1` and `BOX2` in full

**When** the container promotion runs

**Then** `BOX1` and `BOX2` both get `PALLET1` as their **destination container**
**And** if `PALLET1` itself is the only child of a further container whose type is not reusable, the promotion recurses one level up.

**Given** instead that `PALLET1`'s type is **reusable**

**Then** no promotion happens: the two boxes travel and the pallet stays.

## Scenario 41 — Put in pack

**Given** a delivery Transfer has three detail lines with positive quantities, none of them in a container, all with the same destination Location
**And** the Operation Type does not ask to set a container type

**When** the user presses put in pack

**Then** a new container is created, drawing its name from the container sequence (for example `PACK0000005`)
**And** all three lines receive it as destination container
**And** when exactly one line had been selected, put-away is re-run for that line with the container so that it is directed at a Location that can accept a container of that type.

**Given** instead that the three lines point at **two different** destination Locations

**Then** the destination chooser opens first; the Location chosen there is written onto all three lines before the container is created.

**Given** instead that the Operation Type asks to set a container type and nothing was supplied

**Then** the put-in-pack screen opens first, offering an existing container, a container type or a free name.

## Scenario 42 — Splitting a container's content across two locations fails at validation

**Given** a Transfer's lines put the content of one destination container into two different Locations

**When** the Transfer is validated

**Then** the container-consistency check of the completion algorithm fails with "You cannot move the same package content more than once in the same transfer or split the same package into two location." followed by a line break, "Package: " and the container name.

## Scenario 43 — Unpacking

**Given** `PACK0003` in `WH/Stock` holds 6.00 Bolt and one child container `BOX9`

**When** the user unpacks `PACK0003`

**Then** `BOX9` loses its parent container
**And** an adjustment-style move with the reference "Quantities unpacked" relocates the 6.00 Bolt from (WH/Stock, PACK0003) to (WH/Stock, no container)
**And** the housekeeping pass runs on the affected records, so that the resulting duplicates are merged.

---

# G. Counting

## Scenario 44 — Adjustment of plus five

**Given** `WH/Stock` holds **12.00** Bolt with 0.00 reserved
**And** the record's counted flag is off

**When** an inventory user types a counted quantity of **17** on the record

**Then** the counted flag turns on and the difference field reads **+5.00**
**And** the outdated indicator is false.

**When** the user presses Apply

**Then** one Stock Move is created with: product Bolt, the product unit, a demand of **5**, source `Inventory adjustment`, destination `WH/Stock`, status confirmed, the adjustment flag set, the picked flag set, and one detail line of 5 with the same Locations and the record's container
**And** the move is completed immediately, with destination containers ignored
**And** `WH/Stock` holds **17.00** Bolt
**And** `Inventory adjustment` holds **−5.00** Bolt
**And** the move's reference reads "Product Quantity Updated (*the acting user's display name*)"
**And** `WH/Stock`'s last-count date becomes today
**And** the record's counted quantity, difference, counted flag and assignee are cleared
**And** the record's scheduled count date is recomputed from the Location's frequency and the company's annual date
**And** no journal entry is produced by this domain.

**When** the user instead types a counted quantity of **9**

**Then** the difference reads **−3.00**, the move goes **from** `WH/Stock` **to** `Inventory adjustment` with a demand of 3, and `WH/Stock` ends with 9.00.

## Scenario 45 — Conflict on a count

**Given** the counted quantity 17 was typed while the on-hand quantity was 12
**And** a delivery then removed 2.00 Bolt, so the on-hand quantity is now 10.00 while the recorded difference is still +5.00

**Then** the outdated indicator is true, because 17 − 5 = 12 no longer equals 10.

**When** the user presses Apply

**Then** the conflict screen opens listing the record.

**When** the user chooses "Keep counted quantity"

**Then** the count is applied as typed: the difference is recomputed as 17 − 10 = **+7.00** and the resulting move has a demand of 7, bringing the on-hand quantity to **17.00**.

**When** the user instead chooses "Keep difference"

**Then** the counted quantity is recomputed as 10 + 5 = **15.00**, the difference stays **+5.00**, and the resulting move has a demand of 5, bringing the on-hand quantity to **15.00**.

## Scenario 46 — Requesting a count

**Given** an inventory manager selects three records, one of which carries the lot-tracked product Paint under lot `L1` in `WH/Stock`
**And** `WH/Stock` also holds Paint under lot `L2` and Paint with no lot

**When** they request a count for **1 October 2026** assigned to user "Alice"

**Then** the set is extended with every other record of (Paint, `WH/Stock`), so the two sibling records are included
**And** each record of the extended set has its scheduled date set to 1 October 2026 and its assignee set to Alice
**And** **no** counted quantity is written and no counted flag is turned on.

**Given** instead the lot group is not active, or no selected record carries a tracked product

**Then** the set is not extended and only the three selected records are scheduled.

## Scenario 46 bis — Applying with a reference label and a counting date

**Given** three records carry counted quantities

**When** the manager uses "apply all" with the reference "Yearly count" and the counting date **30 September 2026**

**Then** only the records whose counted flag is set are applied
**And** every adjustment move created carries the reference "Yearly count" and the date 30 September 2026.

## Scenario 47 — Cyclic and annual count dates

**Given** today is **11 September 2026**
**And** `WH/Stock` has a counting frequency of **30** days and a last-count date of **1 September 2026**
**And** the company's annual inventory is set to month 12, day 31

**Then** the Location's next-count date is **1 October 2026** (30 − 10 = 20 days remain, so last count plus 30 days)
**And** the company's yearly date is **31 December 2026** (it is after today, so the current year stands)
**And** a quantity record in that Location gets the smaller of the two, **1 October 2026**.

**Given** instead that the last count was **1 August 2026**

**Then** 30 − 41 = −11, which is not positive, so the Location's next-count date becomes **12 September 2026** (today plus one day).

**Given** instead that the Location has no frequency at all

**Then** the Location's next-count date is empty and the record gets the company's yearly date, **31 December 2026**.

**Given** the company's annual inventory is set to month 2, day 31, and the current year is not a leap year

**Then** the day is clamped to the last day of February, **28**.

## Scenario 48 — Reverting an adjustment

**Given** the adjustment of scenario 44 produced one completed detail line of 5 from `Inventory adjustment` to `WH/Stock`

**When** the user selects it and presses revert

**Then** one mirror move is created and completed: demand 5, source `WH/Stock`, destination `Inventory adjustment`, adjustment flag set, picked flag set, containers swapped
**And** its reference is "Product Quantity Updated (*the user's display name*) [reverted]"
**And** `WH/Stock` returns to 12.00 Bolt.

**When** the user selects a line that is not an adjustment line

**Then** the notification "There are no inventory adjustments to revert." is shown.

## Scenario 49 — A manager deletes a quantity record

**Given** a record holds 3.00 Bolt

**When** an inventory manager deletes it

**Then** the deletion is replaced by an adjustment: the counted quantity is set to 0 and applied, producing a move of 3 to `Inventory adjustment`
**And** the record ends with 0.00 and is removed by the housekeeping pass.

**When** a plain inventory user deletes it

**Then** the operation fails with "Quants are auto-deleted when appropriate. If you must manually delete them, please ask a stock manager to do it."

---

# H. Returns

## Scenario 50 — Returning a delivery

**Given** `WH/OUT/00005` is `done`, having delivered **8** Bolt from `WH/Stock` to `Customers`
**And** the delivery Operation Type's return Operation Type is `WH: Receipts`

**When** the user opens the return screen and asks to return **3**

**Then** a new Transfer is created with: the Operation Type `WH: Receipts`; the source Location `Customers` (the original's destination); the destination Location the return type's default destination, that is `WH/Stock`; the origin "Return of WH/OUT/00005"; the return link pointing at `WH/OUT/00005`; an empty responsible
**And** one move is created with a demand of 3, the original move recorded as the original return move, the take-from-stock supply method, and the original Transfer's document references
**And** a note linking the two is posted
**And** the return Transfer is confirmed and its availability is checked; because the source Location has customer usage it bypasses reservation and the move becomes `assigned` at once.

**When** the return is validated

**Then** `WH/Stock` holds 3.00 Bolt more and `Customers` holds 3.00 less (that is, −5.00)
**And** `WH/OUT/00005`'s return counter reads 1.

**When** the user instead presses "return all"

**Then** each line is pre-filled with the original move's processed quantity minus the quantities of the returns already made against it — here 8 − 3 = **5** if the 3-unit return already exists, and 8 if none does.

## Scenario 51 — Returning an open transfer is refused

**Given** `WH/OUT/00006` is `assigned`

**When** the user opens the return screen

**Then** it fails with "You may only return Done pickings."

**Given** two done Transfers are selected

**Then** it fails with "You may only return one picking at a time."

**Given** a done Transfer all of whose moves went to an inventory-loss Location

**Then** it fails with "No products to return (only lines in Done state and not fully returned yet can be returned)."

**Given** the return screen is confirmed with every quantity at zero

**Then** it fails with "Please specify at least one non-zero quantity."

## Scenario 52 — Exchange on a receipt

**Given** `WH/IN/00006` is done, having received 10 Bolt

**When** the user asks for a return **and exchange** of 4

**Then** a return Transfer is created and confirmed as in scenario 50
**And** a second Transfer is created by copying the return with the same preparation, holding a move of 4
**And** that second Transfer's moves have their original-return link and their originating links cleared, so the exchange is independent
**And** the second Transfer records the return as its own return link
**And** both are confirmed and reserved.

## Scenario 53 — Exchange on a delivery

**Given** `WH/OUT/00007` is done, having delivered 4 Bolt

**When** the user asks for a return and exchange of 4

**Then** the return Transfer is created as usual
**And** instead of copying a second Transfer, one supply request of 4 Bolt is raised at the original move's intermediate destination Location, carrying the Transfer's document references, the move's date, the Warehouse, the contact, the final Location and the company
**And** the rule engine produces the replacement delivery through the normal routes.

## Scenario 54 — A return ignores the backorder policy

**Given** a return Transfer demands 5 and only 2 are processed
**And** the Operation Type's backorder policy is "ask"

**When** the return is validated

**Then** the "should ignore backorders" test is true because the return link is set.

---

# I. Scrap

## Scenario 55 — Scrapping available goods

**Given** `WH/Stock` holds 12.00 Bolt

**When** an inventory user creates a Scrap for **2** Bolt from `WH/Stock` to `Scrap` and validates it

**Then** the Scrap's reference is drawn from the company's scrap sequence, for example `SP/00001`
**And** one Stock Move is created already picked, from `WH/Stock` to `Scrap`, with one detail line of 2 carrying the lot, container and owner chosen on the Scrap
**And** the move is completed with backorder creation suppressed
**And** `WH/Stock` holds **10.00** Bolt and `Scrap` holds **2.00**
**And** the Scrap's status is `done` and its date is the validation instant.

**When** the replenish switch was on

**Then** a supply request for 2 Bolt at `WH/Stock` is additionally run.

## Scenario 56 — Scrapping more than is available

**Given** `WH/Stock` holds 1.00 Bolt

**When** the user validates a Scrap of 2 Bolt

**Then** the shortage screen opens, titled "Bolt: Insufficient Quantity To Scrap"

**When** the user confirms anyway

**Then** the scrap is performed, `WH/Stock` holds **−1.00** Bolt, and the reservation-freeing routine takes back any reservation that the negative figure invalidated.

**When** the user instead validates a Scrap of quantity 0

**Then** it fails with "You can only enter positive quantities."

## Scenario 57 — Scrap inside a transfer

**Given** an open delivery Transfer holds one move of 10 Bolt
**And** the user scraps 1 Bolt from that Transfer

**Then** the scrap move carries the Transfer, so the Transfer's scrap indicator becomes true
**And** the Transfer's status computation excludes that move from the "all done are scrapped" test only when other moves are done normally.

**Given** every other move of the Transfer is cancelled and the only done move is the scrap

**Then** the Transfer's status becomes **`cancel`**, not `done`.

---

# J. State machines

## Scenario 58 — Move status recomputation

**Given** a move with a demand of 10 in a unit whose rounding step is 0.01

| Processed quantity | Originating moves | Supply method | Resulting status |
|---|---|---|---|
| 0 | none | take from stock | `confirmed` |
| 0 | none | advanced | `waiting` |
| 0 | one, still open with a positive demand | take from stock | `waiting` |
| 0 | one, done | take from stock | `confirmed` |
| 4 | any | any | `partially_available` |
| 10 | any | any | `assigned` |
| 12 | any | any | `assigned` |

**And** a move whose status is already `done` or `cancel` never changes
**And** a move in `draft` with a processed quantity of 0 never changes.

## Scenario 59 — Transfer status derivation

**Given** a Transfer with two moves and the as-soon-as-possible shipping policy

| Move statuses | Transfer status |
|---|---|
| one `draft`, one `assigned` | `draft` |
| both `cancel` | `cancel` |
| one `done`, one `cancel` (the cancelled one not a scrap) | `done` |
| one `done` into an inventory-loss Location, one `cancel` not into one | `cancel` |
| one `assigned`, one `confirmed` | `assigned` (the relevant status is `partially_available`, which is raised to `assigned`) |
| both `confirmed` | `confirmed` |
| one `waiting`, one `confirmed` | `confirmed` (the least important open move wins for the as-soon-as-possible policy) |
| both `waiting` | `waiting` |

**Given** the same Transfer with the **all-at-once** shipping policy

| Move statuses | Transfer status |
|---|---|
| one `assigned`, one `confirmed` | `confirmed` |
| one `assigned`, one `partially_available` | `confirmed` |
| both `assigned` | `assigned` |
| all open moves have a demand of zero | `assigned` |

**Given** the Transfer's source Location bypasses reservation and every move uses the take-from-stock supply method

**Then** the Transfer's status is `assigned` regardless of the move statuses.

## Scenario 60 — Batch status

**Given** a batch with three Transfers

| Transfer statuses | Batch status after recomputation |
|---|---|
| all `cancel` | `cancel` |
| two `done`, one `cancel` | `done` |
| one `done`, two `assigned` | unchanged (`draft` or `in_progress`) |
| none (the batch has no Transfer) | unchanged |

**And** a batch already in `done` or `cancel` is never recomputed
**And** confirming an empty batch fails with "You have to set some pickings to batch."

## Scenario 61 — Scrap status

**Given** a Scrap in `draft`

**When** it is validated

**Then** it becomes `done`, and deleting it afterwards fails with "You cannot delete a scrap which is done."

---

# K. Merging and splitting

## Scenario 62 — Two identical moves merge

**Given** a Transfer receives two Stock Moves, both for Bolt, both from `Vendors` to `WH/Stock`, both in the product unit, both with the take-from-stock supply method, the same unit price, the same description, the same deadline and the same final Location
**And** the demands are 4 and 6

**When** the Transfer is confirmed

**Then** the two moves merge into one with a demand of **10**
**And** the surviving move's date is the **minimum** of the two dates, because the Transfer ships as soon as possible
**And** the surviving move's origin is the distinct origins joined by "/"
**And** the detail lines of the second move are re-linked to the first
**And** the second move is cancelled and deleted.

**Given** instead the shipping policy is all at once

**Then** the surviving move's date is the **maximum** of the two.

**Given** instead the system parameter that restricts merging to the same date is set and the dates differ

**Then** the two moves do **not** merge.

## Scenario 63 — A negative move is absorbed

**Given** a positive move of demand **10** and a negative move of demand **−4**, identical on every key field except the description

**When** the merge runs

**Then** the negative move is first detached from its Transfer
**And** the positive move's demand becomes **6**
**And** its unit price becomes `round_to_price_precision( (10 × positive price + (−4) × negative price) ÷ 6 )`
**And** the destination moves of the negative move whose source Location equals the positive move's destination Location are linked to the positive move, and likewise for the originating moves
**And** the negative move is cancelled and deleted.

**Given** instead the positive demand is **3** and the negative demand is **−4**

**Then** the positive move is fully consumed: its demand becomes 0 and it is cancelled (unless it is picked); the negative move's demand becomes **−1** and its unit price is recomputed on its own real quantity; the walk continues to the next positive move.

## Scenario 64 — A remaining negative move becomes a return

**Given** after merging a move of demand **−4** survives, from `WH/Stock` to `Customers`, with a final Location of `Customers`
**And** the delivery Operation Type has `WH: Receipts` as its return Operation Type

**When** the confirmation finishes

**Then** the move's source becomes `Customers`, its destination becomes `WH/Stock`, its final Location becomes `WH/Stock`
**And** its demand becomes **+4**
**And** its Operation Type becomes `WH: Receipts`
**And** its supply method becomes take-from-stock
**And** its chain links are re-wired by matching Locations
**And** it is grouped into its own Transfer.

## Scenario 65 — Splitting a move

**Given** a confirmed move of demand **10** in a unit "Dozen" whose conversion to the product unit is 12, so its real quantity is 120

**When** the move is split by **36** product units

**Then** 36 converts to exactly 3 dozen and back to 36, so the new move keeps the unit "Dozen" with a demand of **3**
**And** the original move's demand becomes `round_digits(convert(max(0, 120 − 36), product unit, Dozen, none), 2)` = **7.00**
**And** the original move's status is recomputed
**And** the original move's reservation is **not** released, because the write suppresses unreservation.

**Given** instead the split quantity is **5** product units

**Then** 5 ÷ 12 is not representable in whole dozens, so the new move is created **in the product unit** with a demand of 5.

---

# L. Rounding and precision

## Scenario 66 — A processed quantity that breaks the general precision

**Given** the `Product Unit` precision is 2 digits

**When** a person writes a processed quantity of **1.005** on a move

**Then** the write fails with "The quantity done for the product Bolt doesn't respect the rounding precision defined on the system. Please change the quantity done or the rounding precision in your settings."

## Scenario 67 — A quantity that breaks the unit's rounding at completion

**Given** the line unit's rounding step is **1** (whole units only)
**And** the `Product Unit` precision is 2 digits
**And** a detail line has a quantity of **2.50**

**When** the Transfer is validated

**Then** the completion fails with "The quantity done for the product \"Bolt\" doesn't respect the rounding precision defined on the unit of measure \"Units\". Please change the quantity done or the rounding precision of your unit of measure."

## Scenario 68 — The backorder test uses digits, not the unit's rounding step

**Given** a move with a demand of **10** and a processed quantity of **9.999** in a unit whose rounding step is **1**

**Then** comparing at the unit's rounding step would say "equal", but the backorder test compares at the `Product Unit` precision of 2 digits, where 10.00 is greater than 10.00? No: 9.999 rounds to 10.00 at two digits, so the comparison says **equal** and **no** backorder move is created.

**Given** instead a processed quantity of **9.99**

**Then** at two digits 9.99 is strictly less than 10.00 and a backorder move of `convert(10 − 9.99, line unit, product unit, half-up)` = **0.01** is created.

## Scenario 69 — Reserving in a different unit

**Given** the move's line unit is "Dozen" (12 product units) and the product unit is "Units"
**And** `WH/Stock` holds **20.00** Units free

**When** a move of **2** Dozen (24 product units) is reserved loosely

**Then** the wanted quantity is clamped to the available 20.00
**And** step 5 of the reservation-quantity computation re-expresses it: 20 ÷ 12 rounded **down** at the Dozen's precision is 1 Dozen; 1 Dozen back to Units, rounding half away from zero, is **12.00**
**And** only **12.00** Units are reserved, so that the reservation is expressible in the move's own unit
**And** the move becomes `partially_available`.

## Scenario 70 — Creating a detail line in the product unit when the line unit cannot express the quantity

**Given** a move whose line unit is "Dozen" reserves **5.00** product units

**Then** 5 ÷ 12 at the Dozen's precision, converted back, does not equal 5
**And** therefore the detail line is created **in the product unit** with a quantity of 5.00, not in Dozens.

---

# M. Chains and the reception report

## Scenario 71 — The reception report offers an arrival to a waiting delivery

**Given** a delivery move of **10** Bolt from `WH/Stock` is `confirmed` with nothing reserved, and it has no originating move
**And** a receipt Transfer of **10** Bolt into `WH/Stock` is open, and its move has no destination move

**When** the user opens the Reception Report from the receipt

**Then** one line appears under the delivery's source document, with a quantity of 10, the product Bolt, the unit, and an Assign button
**And** the line is marked assignable.

**When** the user presses Assign

**Then** the receipt move becomes the originating move of the delivery move
**And** the delivery move's supply method becomes advanced
**And** the document references of the two source documents are shared both ways
**And** the reservation is re-run on the delivery move.

**When** the receipt is then validated

**Then** the delivery move reserves exactly the goods the receipt brought, in the Location, lot and container the receipt put them into.

## Scenario 72 — Assigning only part of a demand splits it

**Given** the delivery move demands **10** and only **4** are being assigned

**When** Assign is pressed

**Then** the delivery move is split: a new move of **6** is created, written directly to `confirmed` and carrying the parent's reservation date
**And** the 4-unit move gets the incoming move as originating move.

## Scenario 73 — Unassigning

**Given** the assignment of scenario 71 exists and neither move is done

**When** the user presses Unassign for the whole quantity

**Then** the link is removed, the shared references are removed, the delivery move's supply method returns to take-from-stock, and the delivery move is unreserved.

## Scenario 74 — Deadline propagation along a chain

**Given** three chained moves A → B → C, with deadlines 10, 11 and 12 September 2026 respectively

**When** B's deadline is written as **9 September 2026**

**Then** the shift is `11 − 9 = 2` days
**And** A's deadline becomes 10 − 2 = **8 September 2026**
**And** C's deadline becomes 12 − 2 = **10 September 2026**
**And** the propagation does not loop, because each visited move is remembered.

**Given** instead that A has no deadline at all

**Then** A's deadline is set to **9 September 2026**, the value written on B.

## Scenario 75 — Delay alert

**Given** move C is scheduled for 12 September and its originating move B, still open, is scheduled for 15 September

**Then** C's delay alert date is **15 September**, because the maximum open originating date is later than C's own date
**And** C's Transfer's delay alert date is the maximum over its moves
**And** a note is posted on the affected documents with the subject "Deadline updated due to delay on *the upstream document name*".

**Given** instead B is scheduled for 10 September

**Then** C's delay alert date is empty.

---

# N. Multi-company

## Scenario 76 — Resupply between two warehouses of the same company

**Given** Acme has two Warehouses, `WH` and `WH2`, both delivering in one step
**And** `WH2` lists `WH` as a supplying Warehouse

**Then** the company's internal transit Location is activated
**And** a Route named "WH2: Supply Product from WH" is created, selectable on the Warehouse, on products and on product categories, with both Warehouse links and Acme as company
**And** three Stock Rules are created: an extra supply-on-order rule from `WH/Stock` to the transit Location through `WH: Delivery Orders` named with the suffix "(MTO)"; a pull rule from `WH/Stock` to the transit Location through `WH: Delivery Orders` with the destination taken from the rule and the take-from-stock supply method; a pull rule from the transit Location to `WH2/Stock` through `WH2: Receipts` with the advanced supply method.

**When** `WH` is reconfigured to deliver in **two steps**

**Then** the rule whose destination is the transit Location is re-pointed at `WH/Output` and given the advanced supply method
**And** a rule from `WH/Stock` to `WH/Output` through `WH: Pick` is created or un-archived
**And** the supply-on-order rules from `WH/Stock` to a transit Location are archived.

## Scenario 77 — Record visibility

**Given** a reader has companies A and B enabled and company C disabled

**Then** they see: Transfers, Operation Types, Warehouses, Stock Moves, Scraps and Reordering Rules of A and B only; Locations, Lots, Stock Move Lines, Stock Quantity records, Stock Rules, Routes, containers and Storage Categories of A, of B **and those with no company**
**And** they never see records of C through any of those entities.

## Scenario 78 — Company immutability

**When** a person writes a different company on a Warehouse, a Location, an Operation Type or a Put-away Rule

**Then** the write fails with "Changing the company of this record is forbidden at this point, you should rather archive it and create a new one."

## Scenario 79 — Inter-company unpacking

**Given** the container group is active and the system parameter that unpacks inter-company containers is set
**And** a Transfer of company A has a contact belonging to company B
**And** one of its moves lands in a transit Location

**When** the Transfer is validated

**Then** the destination containers of that move are unpacked.

---

# O. Housekeeping

## Scenario 80 — Merging duplicate quantity records

**Given** two Stock Quantity records exist for exactly (Bolt, `WH/Stock`, no lot, no container, no owner), created by two concurrent transactions:

| Record | On hand | Reserved | Counted | Incoming date |
|---|---|---|---|---|
| Q1 (lower identifier) | 4.00 | 1.00 | 0 | 5 September 2026 |
| Q2 | 6.00 | 2.00 | 0 | 1 September 2026 |

**When** the merge pass runs

**Then** Q1 survives with on hand **10.00**, reserved **3.00**, counted **0**, incoming date **1 September 2026**
**And** Q2 is deleted.

**Given** instead the reserved figures are 1.00 and −4.00

**Then** the surviving reserved figure is `max(0, −3.00)` = **0.00**.

## Scenario 81 — Cleaning reservations

**Given** a Stock Quantity record for (Bolt, `WH/Stock`) shows reserved **5.00**
**And** the open detail lines that draw from exactly that key total **3.00**

**When** the clean-reservations pass runs

**Then** the counter is adjusted by 3 − 5 = **−2.00**, ending at 3.00.

**Given** instead the record sits in a Location that bypasses reservation

**Then** the counter is lowered by its whole value, ending at 0.00.

**Given** instead open detail lines total 2.00 against a key that has **no** record with a reservation

**Then** a reservation of 2.00 is raised for that key, creating the record if needed — unless the Location bypasses reservation.

## Scenario 82 — Deleting empty records

**Given** the shipped product-unit precision record has 2 digits, so the rounding used is `max(6, 4)` = **6** digits
**And** a record has on hand 0.0000001, reserved 0, counted 0 and no assignee

**Then** rounded to six digits the on-hand quantity is 0, so the record is deleted.

**Given** instead the record has an assignee

**Then** it is **not** deleted.

---

# P. Batches and waves

## Scenario 83 — Validating a batch with an empty transfer

**Given** a batch holds three ready delivery Transfers: T1 with 5 processed, T2 with 3 processed and T3 with nothing processed

**When** the batch is validated

**Then** T3 is **detached** from the batch rather than validated, and a note is posted on the batch: "*a link to T3* was removed from the batch, no quantity processed"
**And** T1 and T2 are validated together with the sanity check run once over both
**And** a note is posted on each of them: "**Transferred by:** Batch Transfer *a link to the batch*"
**And** T3's zero-quantity moves lose their picked flag
**And** the batch's status becomes `done` once T1 and T2 are done.

## Scenario 84 — Lot supplied on one transfer of a batch

**Given** a batch holds two Transfers, each with one move of a lot-tracked product
**And** the lot is typed on the first Transfer's line only, and the second Transfer has nothing processed

**When** the batch is validated

**Then** the sanity check is run once for the whole batch, so the second Transfer's untouched lines are not checked
**And** the validation succeeds.

## Scenario 85 — Automatic batching by contact

**Given** the delivery Operation Type has automatic batches on, grouping by contact, with a maximum of 3 Transfers and no line limit
**And** two ready delivery Transfers already sit in batch `BATCH/OUT/00001` for contact "Customer C"

**When** a third ready delivery Transfer for "Customer C" is confirmed

**Then** it is added to `BATCH/OUT/00001`, because 2 + 1 ≤ 3.

**When** a fourth is confirmed

**Then** the limit blocks it; no other unbatched ready Transfer of that contact exists, so a new batch is created holding it alone, with the confirming Transfer's responsible, and it is confirmed at once because auto-confirm is on
**And** its description is "Customer C".

## Scenario 86 — Automatic waving by product

**Given** the delivery Operation Type has automatic batches on, grouping waves by product
**And** a validation produces two backorders, each with one line of Bolt and one line of Paint

**When** the automatic waving runs

**Then** the Bolt lines of both backorders are collected into one wave and the Paint lines into another
**And** each wave's description is the product's display name
**And** each contributing Transfer is split: a copy carrying only the taken lines is created with the wave as batch, and the moves are split by the taken quantities.

## Scenario 87 — Merging two batches

**Given** two batches of the same Operation Type, both waves, both in `in_progress`, scheduled on 12 and 10 September respectively

**When** they are merged

**Then** the first-selected batch survives and receives the other's Transfers and detail lines
**And** the responsible, description and scheduled date of the **10 September** batch (the earliest) are written onto the survivor
**And** the other is deleted
**And** a notification names the survivor with a link to it.

**Given** instead one is a batch and the other a wave

**Then** the merge fails with "Batch transfers cannot be merged with wave transfers and vice versa."

## Scenario 88 — Dispatch: setting a dock

**Given** the delivery Operation Type uses dispatch management and lists `WH/Output` as a dock
**And** a batch holds two delivery Transfers whose moves source from `WH/Stock`

**When** the dock `WH/Output` is set on the batch

**Then** every move of every Transfer of the batch has its **source** Location rewritten to `WH/Output`, because the Operation Type kind is delivery
**And** the Transfers are re-ordered by the contact's postal code ascending and their batch sequence is stamped 0, 1, …

**When** the dock is cleared

**Then** every move whose source Location is no longer inside its Transfer's own source Location is reset to that Transfer's source Location.

## Scenario 89 — Dispatch: load percentages

**Given** a batch carries one container of type "Pallet" (base weight 20, 1200 by 800 by 1500 millimetres) holding 40 units of a product weighing 2 and of volume 0.01, plus 6 loose units of the same product
**And** the vehicle's category has a maximum weight of 800 and a maximum volume of 12

**Then** the estimated shipping weight is 20 + (40 + 6) × 2 = **112**
**And** the estimated shipping volume is (1200 × 800 × 1500) ÷ 1 000 000 000 + 46 × 0.01 = 1.44 + 0.46 = **1.9**
**And** the weight percentage is 100 × 112 ÷ 800 = **14**
**And** the volume percentage is 100 × 1.9 ÷ 12 ≈ **15.83**.

---

# Q. Editing completed work

## Scenario 90 — Changing a done quantity

**Given** `WH/OUT/00008` is done, having delivered 8.00 Bolt from `WH/Stock`, which now holds 12.00
**And** the Transfer is unlocked

**When** the user changes the detail line's quantity from 8 to **6**

**Then** the original movement is undone: `Customers` is reduced by 8.00 and `WH/Stock` is increased by 8.00, keeping the incoming date that came back
**And** a note describing the change is posted
**And** the new movement is applied: `WH/Stock` is reduced by 6.00 and `Customers` is increased by 6.00
**And** `WH/Stock` therefore ends at 12 + 8 − 6 = **14.00**
**And** the downstream moves, if any, are unreserved and re-reserved.

**When** the user instead tries to **delete** the line

**Then** it fails with "Deleting product moves after the transfer is done?\n\nThat would be like going back in time to revert all operations triggered after this move. Who knows what the end result would be, So let's not do it.\n\nTry changing the “done” quantity to 0 instead."

## Scenario 91 — Editing a done quantity that makes the source negative

**Given** `WH/Stock` holds 2.00 Bolt and another open document has reserved 2.00 of it
**And** a done detail line of 6.00 is raised to **8.00**

**Then** after the replay the source available quantity is negative by 2.00
**And** the reservation-freeing routine runs: the candidates are searched (open, exactly matching characteristics, positive quantity, not picked, not this line) and sorted with the current Transfer first, then by scheduled date descending, then by identifier descending
**And** 2.00 is taken back from the first candidate, deleting it when it held exactly 2.00
**And** the affected moves have their supply method reset to take-from-stock, their originating links cleared, and are re-reserved in reverse order.

---

# R. Locking and permissions

## Scenario 92 — The lock flag

**Given** an open, locked Transfer

**Then** the demand of its non-draft moves is not editable.

**When** the user unlocks it

**Then** the demand becomes editable.

**Given** a done, locked Transfer

**Then** the processed quantities are not editable and the scheduled date is frozen.

**When** the Transfer is cancelled

**Then** the lock flag is forced on.

## Scenario 93 — Counted quantity outside counting mode

**Given** the counting-mode context is **not** active

**When** a counted quantity is written on a Stock Quantity record

**Then** the inverse does nothing at all: no adjustment move is created and no error is raised.

## Scenario 94 — Restricted fields in counting mode

**Given** the counting-mode context is active

**When** a person writes the product, the Location, the lot, the container or the owner on a record

**Then** it fails with "Quant's editing is restricted, you can't do this operation." — unless the record sits in an inventory-loss Location, in which case the write is silently ignored.

## Scenario 95 — Non-manager sees only their own counts

**Given** a user is in the inventory user group but not the manager group

**When** they open the physical inventory screen

**Then** the screen is pre-filtered to the records whose assignee is that user.

---

# S. Configuration side effects

## Scenario 96 — Switching the reservation method to "before scheduled date"

**Given** the delivery Operation Type currently reserves at confirmation, and three open moves exist with scheduled dates 20, 21 and 22 September, the second one urgent
**And** the type is switched to "Before scheduled date" with 3 ordinary days and 1 urgent day

**Then** each open move of that type gets a reservation date: 17 September, **20** September (urgent, one day) and 19 September.

**When** the type is switched **away** to "Manually"

**Then** every move of that type whose status is not assigned, done or cancelled has its reservation date cleared.

## Scenario 97 — Renaming a warehouse

**Given** the Warehouse `WH` is named "Acme Main" and is renamed "Acme Central"

**Then** every Route of the Warehouse has the first occurrence of "Acme Main" in its name replaced by "Acme Central", and so does every rule of those Routes and the supply-on-order rule
**And** the names of the eight numbering sequences are rewritten accordingly
**And** the sequence prefixes, which are built from the **short name**, are unchanged.

**When** the short name is changed from `WH` to `WHC`

**Then** the parent Location of the stock Location is renamed `WHC`
**And** the eight sequence prefixes become `WHC/IN/`, `WHC/OUT/`, and so on
**And** existing Transfer references are **not** renumbered.

## Scenario 98 — Multi-warehouse group

**Given** one company has exactly one active Warehouse and the multi-warehouse group is granted to internal users

**When** a second Warehouse is created

**Then** nothing is withdrawn; the group stays granted and the multi-location group is granted too.

**Given** two Warehouses exist and one is archived, leaving one per company everywhere

**Then** the multi-warehouse group is withdrawn from internal users and from every user who held it.

## Scenario 99 — Withdrawing settings that are in use

**When** an administrator tries to switch off Storage Locations while both it and the multi-warehouse group are implied for internal users

**Then** it fails with "You can't deactivate the multi-location if you have more than once warehouse by company"

**When** an administrator tries to switch off Lots & Serial Numbers while at least one product is tracked

**Then** it fails with "You have product(s) in stock that have lot/serial number tracking enabled. \nSwitch off tracking on all the products before switching off this setting."

## Scenario 100 — Numbering

**Given** the Warehouse short name is `WH`

**Then** the first Transfer of each generated Operation Type is numbered `WH/IN/00001`, `WH/OUT/00001`, `WH/PICK/00001`, `WH/PACK/00001`, `WH/QC/00001`, `WH/STOR/00001`, `WH/INT/00001`, `WH/XD/00001`
**And** the first batch of a delivery Operation Type whose prefix is `OUT` is named `BATCH/OUT/00001`
**And** the first wave is named `WAVE/OUT/00001`
**And** the first Scrap of the company is `SP/00001`
**And** a container created with no name and no container type is `PACK0000001`.

**When** an Operation Type's sequence prefix is changed to `RECV`

**Then** its sequence is renamed and re-prefixed to `WH/RECV/`, its padding stays 5, and the next Transfer is numbered `WH/RECV/00002` if the counter had already reached 1.

**Given** another Operation Type of the same company already uses the prefix `RECV`

**Then** a warning is shown — "This sequence prefix is already being used by another operation type. It is recommended that you select a unique prefix to avoid issues and/or repeated reference values or assign the existing reference sequence to this operation type." — but the change is accepted.

---

# T. Location tree and weights

## Scenario 101 — Full names and paths

**Given** the tree `WH` (virtual) → `WH/Stock` (internal) → `Shelf A` (internal)

**Then** `WH`'s full name is `WH` (it has no parent)
**And** `WH/Stock`'s full name is `WH/Stock`
**And** `Shelf A`'s full name is `WH/Stock/Shelf A`
**And** a virtual Location's own full name is just its name even when it has a parent
**And** `Shelf A`'s materialised path starts with `WH/Stock`'s path, which starts with `WH`'s path.

## Scenario 102 — Archiving a location tree

**Given** `WH/Stock/Shelf A` holds 3.00 Bolt

**When** the user archives `WH/Stock`

**Then** it fails with "You can't disable locations WH/Stock/Shelf A because they still contain products."

**Given** instead every descendant is empty

**Then** `WH/Stock` and every descendant are archived — unless a Warehouse uses one of them as its stock or view Location, in which case it fails with "You cannot archive location WH/Stock because it is used by warehouse WH".

## Scenario 103 — Location weight

**Given** `WH/Stock/Shelf A` holds 10.00 Bolt (weight 0.5 each)
**And** an open detail line will take 4.00 Bolt out of it and another will bring 6.00 Bolt into it

**Then** the net weight is 10 × 0.5 = **5.00**
**And** the forecasted weight is 5.00 − 4 × 0.5 + 6 × 0.5 = **6.00**.

## Scenario 104 — Transfer weights

**Given** a Transfer has one line of 4.00 Bolt with no container and one line of 6.00 Bolt whose destination container is `PACK0004` of type "Box" (base weight 1.2, no manual shipping weight)

**Then** the bulk weight is 4 × 0.5 = **2.00**
**And** the shipping weight is 2.00 + (1.2 + 6 × 0.5) = **6.20**
**And** the shipping volume is (4 + 6) × 0.001 = **0.01**.

**Given** instead `PACK0004` has a manual shipping weight of **5.00**

**Then** the shipping weight is 2.00 + 5.00 = **7.00**.

---

# U. Printed documents and aggregation

## Scenario 105 — Aggregating lines for a delivery document

**Given** a delivery move of 10 Bolt has two detail lines of 6 and 4, both in the same unit, both with the same description and no container

**Then** the two lines fall into one group whose key is built from the product identifier, the product display name, the description, the line unit identifier and the packaging unit identifier
**And** the group's processed quantity is **10**
**And** the group's ordered quantity is the move's demand, **10**.

**Given** the Transfer has a backorder whose move demands 4 more of the same product with the same key

**Then** the ordered quantity of the group on the parent's document becomes 10 + 4 = **14**, then reduced by the quantities of the other lines of the same move — here none — giving **14**.

**Given** one line has the destination container `PACK0005`

**Then** its key carries the container identifier as a suffix, so it forms a group of its own and is printed under that container.

## Scenario 106 — Cancelled and untouched moves on the document

**Given** a Transfer has a move of 3 Paint that was cancelled and a move of 2 Paint that is confirmed with no detail line

**Then** each of them adds a line to the printed document with no processed quantity and an ordered quantity equal to its demand, provided no existing group's key is a prefix of theirs; otherwise their demand is added to that group's ordered quantity.

---

# V. Traceability

## Scenario 107 — Upstream and downstream

**Given** lot `L1` of Paint was received in `WH/IN/00010`, moved internally in `WH/INT/00003` and delivered in `WH/OUT/00011`

**When** the traceability tree is opened for `L1`

**Then** the tree shows the three completed detail lines
**And** walking upstream from the delivery line reaches the internal line and then the receipt line
**And** walking downstream from the receipt line reaches the internal line and then the delivery line
**And** each node shows the reference, the date, the product, the lot, the quantity with its unit, the source and destination Locations and the document
**And** the walk never revisits a line.

## Scenario 108 — Delivery discovery through production

**Given** lot `L1` of a component was consumed to produce lot `L2` of a finished good, recorded as a production genealogy link
**And** `L2` was delivered in `WH/OUT/00012`

**When** the delivery list of `L1` is computed

**Then** the walk finds no leaf line for `L1` itself but records `L2` as its child
**And** `L2`'s leaf line names `WH/OUT/00012`
**And** the propagation pushes that Transfer up to `L1`
**And** `L1`'s delivery list is `[WH/OUT/00012]` and its delivery count is 1
**And** its contact list is the contact of that Transfer.

---

# W. Barcodes

## Scenario 109 — Aggregate barcode of a quantity record

**Given** the separator parameter is `,` and the maximum length is 400
**And** Drill has a valid structured product barcode `12345678901231`
**And** a quantity record holds 1 unit of Drill under serial number `SN00007`

**Then** the record's structured barcode is `01` + `12345678901231` + `21` + `SN00007`, with no quantity part because the product is serial-tracked with a quantity of one
**And** the aggregate barcode is that string followed by a tabulation character.

**Given** instead the product is Paint with the same barcode, a quantity of 5 and lot `L1`, and no unit-specific application identifier applies

**Then** the quantity part is `30` followed by `00000005` (the rounded quantity left-padded to eight digits)
**And** the tracking part is `10` + `L1`.

**Given** the lot name is longer than twenty characters

**Then** the record produces **nothing**.

**Given** the separator parameter is absent

**Then** the whole generation produces an empty list.

---

# X. Error catalogue spot checks

## Scenario 110 — Validating an empty transfer

**Given** a single Transfer with no moves and no detail lines

**When** it is validated

**Then** it fails with "You can’t validate an empty transfer. Please add some products to move before proceeding."

## Scenario 111 — Validating a zero-quantity transfer

**Given** a single Transfer whose every open move has a processed quantity of zero

**When** it is validated

**Then** it fails with "Transfer trouble alert! Validating a zero quantity transfer? You're not moving invisible goods around are you?\nSet some quantities and let's get moving!"

## Scenario 112 — Several transfers validated together

**Given** two Transfers are validated together: T1 has no moves and T2 is missing a lot on a Paint line

**Then** the failure carries one merged message: "Transfers T1: Please add some items to move.\n\nTransfers ['T2']: You need to supply a Lot/Serial number for products ['Paint']."
**And** the zero-quantity case is **not** reported in this form.

## Scenario 113 — Negative quantity on a detail line

**When** a person writes −1 as a detail-line quantity

**Then** it fails with "You can not enter negative quantities."

**When** a detail line reaches completion with a strictly negative quantity

**Then** it fails with "No negative quantities allowed".

## Scenario 114 — Serial-tracked line with a quantity other than one

**When** a person writes 2 on a detail line of Drill

**Then** it fails with "You can only process 1.0 Units of products with unique serial number."

## Scenario 115 — Duplicate lot name

**Given** lot `L1` of Paint already exists for Acme

**When** another lot `L1` of Paint is created for Acme

**Then** it fails with "The combination of lot/serial number and product must be unique within a company including when no company is defined.\nThe following combinations contain duplicates:\n - Product: Paint, Lot/Serial Number: L1"

**Given** instead the existing lot has **no** company

**Then** the creation for Acme still fails with the same message, because the company-less pool is compared against.

**Given** instead the existing lot belongs to another company Beta

**Then** the creation for Acme **succeeds**: lots of different companies are not compared with each other.

## Scenario 116 — Location barcode collision

**Given** `WH/Stock` has the barcode `WHSTOCK` in Acme

**When** another Location of Acme is given the same barcode

**Then** it fails with "The barcode for a location must be unique per company!"

**Given** instead the other Location belongs to Beta

**Then** it succeeds.

## Scenario 117 — Storage capacity constraints

**When** a Storage Category is given a maximum weight of −1

**Then** it fails with "Max weight should be a positive number."

**When** a capacity is given a quantity of 0

**Then** it fails with "Quantity should be a positive number."

**When** a second capacity for the same product is added to the same category

**Then** it fails with "Multiple capacity rules for one product."

## Scenario 118 — Destination container recursion

**Given** container `A` has `B` as destination container and `B` has `C`

**When** `C` is given `A` as destination container

**Then** it fails with "A package can't have one of its contained packages as destination container."

## Scenario 119 — Replenishment location uniqueness

**Given** `WH/Stock` is marked as a replenishment Location

**When** `WH/Stock/Shelf A` is also marked

**Then** it fails with "Another parent/sub replenish location Stock exists, if you wish to change it, uncheck it first"

## Scenario 120 — Rule and route company mismatch

**Given** a Route belongs to Acme

**When** one of its rules is given the company Beta

**Then** it fails with "Rule *the rule display name* belongs to Beta while the route belongs to Acme."
