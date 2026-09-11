# Inventory Operations — Accounting Effects

**This domain produces no journal entries.**

Not one operation specified in this folder — confirming a Transfer, reserving goods, validating a Transfer, creating a backorder, returning goods, scrapping, adjusting a count, relocating quantities, packing, unpacking, batching — writes a Journal Entry (`account.move`) or a Journal Item (`account.move.line`). The inventory operations domain moves physical quantities and records where they are; it never touches the ledger.

What it does instead is produce, at a precisely defined instant and with a precisely defined content, the records that the valuation domain turns into journal entries. This file specifies that hand-over exactly, so that an implementation can build the two domains independently and still obtain the same financial result.

---

# 1. What is handed over, and when

## 1.1 The trigger

The single hand-over point is the moment a Stock Move reaches the status `done`. That happens inside step 10 of the completion algorithm (`calculations.md`, section 16), after the quantities have physically moved on the Stock Quantity records and before the push rules run.

At that instant the following is guaranteed and must remain true for any consumer:

| Guarantee | Where it is established |
|---|---|
| The move's processed quantity equals the sum of its detail lines' quantities, converted into the move's line unit. | `calculations.md`, section 16, step 7 and the recomputation of the quantity field. |
| Every detail line of the move has moved its quantity: the source Location's on-hand figure has decreased and the destination Location's has increased by exactly the line's quantity converted into the product unit. | `calculations.md`, section 17.3. |
| The move's date is the instant of completion. | `calculations.md`, section 16, step 10. |
| The move's demand has already been reduced by whatever went into a backorder move, so demand and processed quantity are consistent with what actually moved (except in the over-processing case, where the processed quantity legitimately exceeds the demand). | `calculations.md`, section 15. |
| The move's source Location, destination Location, product, line unit, lot references, containers and owner will not change afterwards, unless a person explicitly unlocks the Transfer and edits a completed line, which replays the movement. | `business-rules.md`, sections 5 and 6, and `calculations.md`, section 7.2. |

## 1.2 The fields the valuation domain reads

| Field on the Stock Move | Purpose for valuation |
|---|---|
| `product_id` (Product) | Which product is being valued. |
| `product_qty` (Real Quantity) | The quantity in the product unit; the basis of every valuation formula. |
| `quantity` (processed quantity) and `product_uom` (line unit) | The quantity as it was recorded, when the valuation needs the document's own unit. |
| `price_unit` (Unit Price) | The unit cost, in the company currency and in the product unit, that the creating domain wrote onto the move. This domain never computes it; it only carries it, and it preserves it through splits, backorders and merges. |
| `location_id` (Source Location) and its usage | Whether the goods are leaving a valued place. |
| `location_dest_id` (Intermediate Location) and its usage | Whether the goods are entering a valued place. |
| `date` (Date Scheduled, meaning the completion instant once done) | The accounting date of the resulting entry. |
| `company_id` (Company) | Which ledger is concerned. |
| `is_inventory` (Inventory) | Whether the movement is a count correction rather than a business movement. |
| `scrap_id` (Scrap operation) | Whether the movement is a scrap. |
| `origin_returned_move_id` (Original return move) | Whether the movement reverses an earlier one. |
| `picking_id` and its Operation Type kind | Which business document caused it. |
| `move_line_ids` with their lots, containers, owners and Locations | The granularity a lot-level or owner-level valuation needs. |

## 1.3 The classification this domain guarantees

The valuation domain decides *whether* a movement is valued from the usages of the two Locations. This domain guarantees the following classification, which is stable and computable from the move alone:

| Source usage | Destination usage | Nature of the movement |
|---|---|---|
| vendor | internal or transit | Goods entering the company from outside. |
| internal or transit | customer | Goods leaving the company to outside. |
| internal or transit | internal or transit | An internal displacement; no change of ownership. |
| internal | inventory loss | A count correction downwards, or a scrap. |
| inventory loss | internal | A count correction upwards. |
| internal | production | Consumption by manufacturing. |
| production | internal | Output of manufacturing. |
| transit | internal, other company | The receiving half of an inter-company or inter-warehouse resupply. |
| internal | transit | The sending half of the same. |

Two predicates are computed by this domain and exposed for that purpose: a move is **incoming** when its source Location's usage is customer or vendor, or is transit with no company; a move is **outgoing** when its destination Location's usage is customer or vendor, or is transit with no company. A third predicate, **counts for received quantity**, is true when the source usage is vendor or transit.

## 1.4 What the valuation domain does with it

See `../inventory-valuation-and-costing/`. In outline: it creates one valuation layer per valued completed move, applies the costing method of the product's category, and posts the journal entries — the stock input and output accounts, the stock interim accounts, the inventory-loss account, the scrap account and the cost-of-goods-sold account. None of those accounts, and none of those entries, is named, chosen or written here.

---

# 2. Movements that are deliberately *not* handed over

| Movement | Why it produces nothing |
|---|---|
| A move that is cancelled | It never reaches the completed status. |
| A move whose processed quantity is zero and which is not an adjustment move | The pruning step of the completion algorithm cancels or skips it. |
| A reservation | Nothing physical has happened; only the reserved counters changed. |
| A put-away redirection | Only a destination Location was rewritten before the goods moved. |
| Creating, nesting, unpacking or renaming a container, as long as the goods stay where they are | No quantity changed Location. The relocation mechanism that *does* move goods creates adjustment-style moves, which are handed over normally. |
| Changing the counted quantity of a Stock Quantity record without applying it | No move exists yet. |
| Requesting a count, snoozing it, or reassigning it | Administrative only. |
| Creating, merging, cancelling or validating a Batch Transfer | The batch is a grouping document; every financial consequence comes from the Transfers it contains. |
| Splitting a Transfer without validating it | Nothing is completed. |
| Assigning or unassigning in the Reception Report | Only chain links and supply methods change. |

---

# 3. The one place where a money figure is touched

The move merging algorithm (`calculations.md`, section 13) recomputes a move's unit price when it absorbs a negative move:

```formula
new_total_value = positive_real_quantity × positive_unit_price + negative_real_quantity × negative_unit_price
new_unit_price  = round_to_price_precision( new_total_value ÷ surviving_real_quantity )
```

where the rounding precision is the decimal-precision setting named `Product Price`, and the result is zero when the surviving real quantity is zero.

This is a weighted average of the two unit prices by their signed quantities. It is the only arithmetic on a money amount in the whole domain, and it happens **before** anything is completed, so the valuation domain only ever sees the already-merged figure.

The merge key itself compares the unit price as a string formatted at a precision equal to the smaller of the company currency's decimal places and the `Product Price` precision, so that two moves whose prices differ only by representation error still merge.

---

# 4. Effects on other domains that are not accounting

For completeness, these are the non-financial consequences a completed move has outside this domain:

| Domain | Consequence |
|---|---|
| `../purchasing/` | The received quantity of a purchase order line is recomputed from the completed moves whose source usage is vendor or transit. |
| `../sales/` | The delivered quantity of a sales order line is recomputed from the completed moves; the delivery status of the order follows. |
| `../manufacturing/` | Component consumption and finished-goods production are completed moves of this domain. |
| `../replenishment-and-procurement/` | The forecast, the free quantity and the quantity to order of every reordering rule touching the products and Warehouses concerned are marked for recomputation whenever a move is created, written or deleted. |
| `../delivery-and-shipping/` | The shipping weight and volume of a Transfer are read at validation to price the shipment. |
| `../messaging-and-activities/` | The confirmation message, the confirmation text message, the change notes, the backorder note and the shortage activity are posted. |

---

# 5. The hand-over in numbers

Three worked examples of exactly what the valuation domain receives.

## 5.1 A receipt

**Given** a receipt of 10 units of BOLT at a unit price of 3.00 in the company currency, validated on 11 September 2026 at 14:05.

**The record handed over:** one Stock Move with

| Field | Value |
|---|---|
| product | BOLT |
| real quantity | 10 |
| processed quantity and line unit | 10, Units |
| unit price | 3.00 |
| source Location and usage | `Vendors`, vendor |
| destination Location and usage | `WH/Stock`, internal |
| date | 11 September 2026 14:05 |
| company | Acme |
| adjustment flag, scrap link, original return move | all empty |
| Operation Type kind | receipt |

**Classification:** goods entering the company from outside. **Incoming** is true; **outgoing** is false; **counts for received quantity** is true.

## 5.2 A delivery of the same goods

**Given** 4 of those units are delivered on 12 September 2026 at 09:30. The delivery move carries no unit price, because nobody wrote one: the valuation domain computes the outgoing cost itself.

**The record handed over:** one Stock Move with real quantity 4, source `WH/Stock` (internal), destination `Customers` (customer), date 12 September 2026 09:30.

**Classification:** goods leaving the company. **Outgoing** is true.

## 5.3 A count that removes one unit

**Given** a count of `WH/Stock` finds 5 instead of 6 on 13 September 2026.

**The record handed over:** one Stock Move with real quantity 1, source `WH/Stock` (internal), destination `Inventory adjustment` (inventory loss), the adjustment flag **set**, the picked flag set, the reference "Product Quantity Updated (*the counting user's display name*)", date 13 September 2026.

**Classification:** a count correction downwards. The adjustment flag is what distinguishes it from a scrap, which has the same Location usages but carries a Scrap link instead.

---

# 6. Ordering guarantees the valuation domain can rely on

| Guarantee | Why it holds |
|---|---|
| A move never reaches `done` before its quantities have moved on the Stock Quantity records. | The completion algorithm moves the quantities in step 7 and writes the status in step 10. |
| The moves of one Transfer reach `done` together, in one operation. | They are written in one bulk write. |
| A backorder move is created **before** the parent move is completed. | The backorder-move step is step 6, the status write is step 10. |
| A push-created move is created **after** the pushing move is done. | The push step is step 11. |
| A destination move is re-reserved **after** the originating move is done. | The propagation step is step 12. |
| The container history exists **before** the containers move. | The snapshots are written in step 2 of the line completion, the quantities move in step 3. |
| An edit of a completed line undoes the whole original movement before applying the new one. | The replay of section 7.2 of `calculations.md` is strictly undo-then-redo, never a delta. |

---

# 7. The one cross-domain field this domain owns

The unit price on a Stock Move (`price_unit`) is written by whichever domain created the move — purchasing writes the purchase price, manufacturing writes the component cost, an inter-company resupply writes the sending company's cost. This domain **never computes it** but does guarantee three things about it:

1. A split preserves it: the backorder move carries the same unit price as its parent.
2. A merge that absorbs a negative move recomputes it as the weighted average of section 3.
3. Two moves whose unit prices differ never merge, so a merge can never silently average two different costs except through the negative-absorption path.
