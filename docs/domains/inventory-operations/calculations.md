# Inventory Operations — Calculations and Algorithms

Every formula and every algorithm of the domain is specified here, as numbered steps, with preconditions, postconditions, failure conditions, rounding rules and worked numeric examples. Nothing in this file is optional: an implementation that deviates from a step order produces different records.

## Conventions used throughout this file

- **Product unit** — the unit of measure that belongs to the product. Every Stock Quantity record is expressed in it. Reserved counters are expressed in it.
- **Line unit** — the unit chosen on the Stock Move (`product_uom`) or on the Stock Move Line (`product_uom_id`). The demand and the processed quantity of a move are expressed in the line unit.
- **Packaging unit** — an optional third unit carried from a sales or purchase document.
- `convert(quantity, from unit, to unit, method)` — the unit conversion owned by `../units-of-measure-and-packaging/`. The `method` is one of `half-up` (round half away from zero at the target unit's precision), `down` (truncate towards zero at the target unit's precision), `up`, or `none` (no rounding at all).
- `compare(a, b, unit)` — returns −1, 0 or +1 after rounding both operands at the unit's rounding step. `is_zero(a, unit)` is `compare(a, 0, unit) = 0`.
- `round_digits(x, n)` — round half away from zero to *n* decimal digits.
- **Product Unit precision** — the number of decimal digits of the decimal-precision setting named `Product Unit`. It is a *digit count*, independent of any unit's rounding step. Several algorithms deliberately use this instead of a unit's rounding step; each such place is called out.

---

# 1. Quantity arithmetic on a Stock Quantity record

## 1.1 Available quantity of one record

```formula
record_available_quantity = record_on_hand_quantity − record_reserved_quantity
```

Both operands are in the product unit. No rounding is applied; the subtraction is exact.

## 1.2 Available quantity of a set of records

Given a product, a Location and optional lot, container and owner, and a matching mode (strict or loose):

1. Gather the candidate records with the gathering algorithm of section 3.
2. If the product is not tracked (tracking mode `none`):
   ```formula
   available = sum(on_hand over gathered records) − sum(reserved over gathered records)
   ```
   If negative results are not allowed and `compare(available, 0, product unit) < 0`, return 0. Otherwise return `available`.
3. If the product is tracked (tracking mode `lot` or `serial`), build one bucket per distinct lot found, plus one bucket named *untracked*:
   1. For each gathered record: when the record has no lot **and** the matching mode is strict **and** a lot was requested, skip the record entirely. When the record has no lot, add its available quantity to the *untracked* bucket. Otherwise add it to that lot's bucket.
   2. If negative results are allowed, return the sum of all buckets, negative ones included.
   3. Otherwise return the sum of only those buckets whose value is strictly greater than zero at the product unit's precision.

The asymmetry in step 3.3 is deliberate: a negative bucket for one lot must not reduce the availability offered by another lot.

**Worked example.** Product tracked by lot, in one Location. Records: lot A on hand 10 reserved 4; lot B on hand 3 reserved 0; no lot, on hand −2 reserved 0.
- Buckets: A = 6, B = 3, untracked = −2.
- Negative results allowed: 6 + 3 − 2 = **7**.
- Negative results not allowed: 6 + 3 = **9**.

---

# 2. Removal strategies

## 2.1 Selecting the strategy

Preconditions: a product and a Location.

1. If the product's category names a removal strategy, return that strategy's method key. Stop.
2. Otherwise walk the Location tree upwards starting at the Location itself: at each step, if that Location names a removal strategy, return its method key and stop; then move to its parent.
3. If no Location in the chain names one, return `fifo`.

The lookup is performed with elevated rights so that a user who cannot read the category or the parent Locations still gets the correct ordering.

## 2.2 Ordering per strategy

| Method key | Full name | Database ordering | Post-ordering in memory |
|---|---|---|---|
| `fifo` | first in first out | incoming date ascending, then identifier ascending | none |
| `least_packages` | least packages | incoming date ascending, then identifier ascending | none (the container pre-selection of section 2.3 has already narrowed the set) |
| `lifo` | last in first out | incoming date descending, then identifier descending | none |
| `closest` | closest location | no database ordering | sort by the full location name ascending, then by identifier descending |
| any other | — | fails with "Removal strategy *the method key* not implemented." | — |

After the strategy ordering, **one final sort is always applied**: records that have a lot come before records that have none. This sort is stable, so it preserves the strategy ordering inside each of the two groups.

**Worked example of first in first out across two arrival batches.** One product, one Location, no lots. Records:

| Record | On hand | Reserved | Incoming date |
|---|---|---|---|
| Q1 | 40 | 0 | 3 March, 08:00 |
| Q2 | 25 | 0 | 10 March, 08:00 |

A demand of 50 gathers Q1 then Q2 and takes 40 from Q1 and 10 from Q2. With last in first out the ordering is Q2 then Q1, and the same demand takes 25 from Q2 and 25 from Q1.

**Worked example of closest location.** Two internal Locations under the same stock Location: `WH/Stock/Shelf 1` (record identifier 91) and `WH/Stock/Shelf 10` (record identifier 42). Ordering by full location name ascending gives `WH/Stock/Shelf 1` before `WH/Stock/Shelf 10` because the comparison is textual, not numeric. Within one Location, the higher identifier comes first.

## 2.3 Least packages: the container pre-selection

Precondition: the strategy is `least_packages` **and** a requested quantity greater than zero was supplied.

The goal is to choose the smallest number of containers whose combined free quantity covers the requested quantity, and to restrict the gathering to those containers. The search is a best-first (A-star) search.

1. Read, for the candidate set defined by the gathering filter, the free quantity per container:
   ```formula
   container_free_quantity = sum over the container's records of ( on_hand − reserved )
   ```
   Keep only containers whose free quantity is strictly positive. Order the result by free quantity descending.
2. Split the result into two parts:
   - Entries that belong to no container are expanded into as many single-unit entries as the integer part of their free quantity. Each such entry has a free quantity of exactly 1 and no container.
   - Entries that belong to a container and whose free quantity is non-zero are kept as they are.
3. Append the single-unit entries after the container entries. Call the resulting ordered list *entries* and let *size* be its length.
4. If no real container was found in step 2, abandon the pre-selection and return the unmodified gathering filter.
5. Define a search node as a triple (remaining quantity, tuple of taken entries, next index). Define the estimate of a node as:
   ```formula
   estimate(node) = number_of_taken_entries + remaining_quantity ÷ free_quantity_of_entry_at_next_index
   ```
   when the next index is inside the list, and simply the number of taken entries when it is not.
6. Put the initial node (requested quantity, empty tuple, index 0) in a priority queue with priority 0. Keep a *best leaf* initialised to the same node.
7. While the queue is not empty:
   1. Pop the node with the lowest estimate. Call it *current*.
   2. If the remaining quantity of *current* is less than or equal to zero, build the filter from *current* (step 8) and stop.
   3. Walk the list from *current*'s next index to the end. Track the free quantity of the previously examined entry and skip any entry whose free quantity equals it, so that only one branch is opened per distinct free quantity. For each examined entry at position *i* (using the position **after** the entry, that is *i* + 1, as the child's next index):
      - Compute the child's remaining quantity as *current*'s remaining quantity minus the entry's free quantity.
      - If that is negative, the child over-selects. Replace the best leaf by the child when the best leaf still has a positive remaining quantity, or when the child takes strictly fewer entries, or when it takes the same number of entries and leaves a larger (that is, less negative) remaining quantity. Do not enqueue the child.
      - Otherwise, if *i* + 1 has reached the end of the list and the remaining quantity is still non-zero, the child cannot be completed. Replace the best leaf by the child when the child's remaining quantity is smaller than the best leaf's. Do not enqueue the child.
      - Otherwise enqueue the child with its estimate as priority.
8. Build the filter from a node: the records whose container is one of the node's container entries, **or** whose identifier is one of the identifiers drawn for the node's single-unit entries (drawn by listing the container-less records matching the gathering filter and popping one identifier per single-unit entry), intersected with the original gathering filter.
9. If the queue empties without an exact cover, build the filter from the best leaf.
10. If the search runs out of memory, log the event and return the unmodified gathering filter.

Postcondition: the returned filter is always at least as restrictive as the original, never broader.

**Worked example.** Containers and free quantities: P1 = 12, P2 = 8, P3 = 5, and three loose units. Requested quantity 13.
- Ordered entries: P1 (12), P2 (8), P3 (5), loose (1), loose (1), loose (1).
- Taking P1 alone leaves 1; taking P1 then P2 over-selects by 7 with two entries; taking P1 then P3 over-selects by 4 with two entries; taking P1 then a loose unit reaches exactly 0 with two entries. The exact cover {P1, one loose unit} is returned.

---

# 3. Gathering

Gathering turns "I need product X at Location L" into an ordered list of Stock Quantity records.

## 3.1 The gathering filter

Inputs: a product, a Location, optionally a lot, a container and an owner, and a matching mode.

**Loose matching** (the mode used when a move looks for stock it has not yet chosen):

1. The record's product equals the requested product.
2. If a lot was requested: the record's lot is either that lot or empty.
3. If a container was requested: the record's container equals it.
4. If an owner was requested: the record's owner equals it.
5. The record's Location is the requested Location or any of its descendants.

Characteristics that were **not** requested impose no condition at all: a loose gathering without a lot will happily return records that carry lots.

**Strict matching** (the mode used when the exact characteristics are already known):

1. The record's product equals the requested product.
2. The record's lot is either empty or exactly the requested lot (and only empty when no lot was requested).
3. The record's container equals the requested container exactly, empty included.
4. The record's owner equals the requested owner exactly, empty included.
5. The record's Location equals the requested Location exactly — descendants are **not** included.

In both modes, when the expiration context is active the filter additionally requires the record's removal date to be on or after the supplied cut-off date, or to be empty.

## 3.2 The gathering algorithm

1. Determine the removal strategy for the product and the Location (section 2.1).
2. Build the filter for the requested matching mode (section 3.1).
3. If the strategy is `least_packages` **and** a requested quantity was supplied, narrow the filter with the container pre-selection (section 2.3).
4. Determine the database ordering for the strategy (section 2.2).
5. If a pre-loaded record cache is available in the calling context **and** the matching mode is strict **and** the strategy is not `least_packages`, take the records from the cache instead of querying: first the cache entry for (product, Location, lot, container, owner) when a lot was requested, then always the cache entry for (product, Location, no lot, container, owner). Otherwise query the database with the filter and the ordering.
6. If the strategy is `closest`, re-sort the result in memory by full location name ascending, then identifier descending.
7. Sort the result so that records with a lot come before records without one, preserving the previous order inside each group.
8. Return the ordered list.

Postcondition: the caller may consume the list from the front; the first record is the one the strategy says should leave first.

## 3.3 The reservation-quantity computation

This is the routine that decides *how much* may be taken and *from which records*. It is used by the reservation algorithm and by the "set a processed quantity" routine.

Inputs: a product, a Location, a wanted quantity in the product unit (which may be negative, meaning "give back"), optionally the line unit, a lot, a container, an owner, and a matching mode.

1. Gather the candidate records (section 3.2), passing the wanted quantity so that the least-packages pre-selection can run.
2. Compute the available quantity over those gathered records (section 1.2) with negative results **not** allowed. Call it *available*.
3. If the calling context carries a packaging unit **and** the product's category asks for full-packaging reservation, reduce the figure:
   ```formula
   available = convert( convert( min(wanted, available), product unit, packaging unit, down ), packaging unit, product unit, half-up )
   ```
   In words: express the smaller of what is wanted and what is available in whole packaging units, rounding down, then convert back. This guarantees that only whole packagings are reserved.
4. Set `wanted = min(wanted, available)`.
5. If the matching mode is loose, a line unit was supplied and it differs from the product unit, re-express the wanted quantity so that it is representable in the line unit:
   ```formula
   wanted = convert( convert( wanted, product unit, line unit, down ), line unit, product unit, half-up )
   ```
   This step is skipped in strict mode, because in strict mode the caller already fixed the unit.
6. If the product is tracked by serial number and `compare(wanted, integer_part(wanted), product unit) ≠ 0`, set `wanted` to 0. A fractional serial-tracked quantity can never be reserved.
7. Decide the direction:
   - If `compare(wanted, 0, product unit) > 0` (taking): set
     ```formula
     available = sum( on_hand over gathered records whose on_hand > 0 ) − sum( reserved over all gathered records )
     ```
     Note that this second figure deliberately ignores the on-hand quantity of *negative* records while still subtracting their reservations.
   - If `compare(wanted, 0, product unit) < 0` (giving back): set `available = sum(reserved over gathered records)`. If `compare(|wanted|, available, product unit) > 0`, fail with "It is not possible to unreserve more products of *the product display name* than you have in stock."
   - If the wanted quantity is exactly zero, return an empty result.
8. Build a map of *negative pockets*: for every gathered record whose available quantity is strictly negative, accumulate that negative amount under the key (Location, lot, container, owner).
9. Walk the gathered records in order. For each record:
   - **Taking.** Let *takeable* be the record's available quantity. If it is less than or equal to zero, skip the record. Look up the negative pocket for the record's key: if it is non-zero, let *offset* be the smaller of the pocket's absolute value and *takeable*; add *offset* back to the pocket (bringing it towards zero) and subtract *offset* from *takeable*. If *takeable* is now less than or equal to zero, skip the record. Otherwise set *takeable* to the smaller of *takeable* and the remaining wanted quantity, append the pair (record, *takeable*) to the result, and reduce both the remaining wanted quantity and *available* by *takeable*.
   - **Giving back.** Let *givable* be the smaller of the record's reserved quantity and the absolute remaining wanted quantity. Append the pair (record, −*givable*). Increase both the remaining wanted quantity and *available* by *givable*.
   - After either branch, stop the walk as soon as the remaining wanted quantity is zero at the product unit's precision, or *available* is zero at the product unit's precision.
10. Return the list of pairs.

The negative-pocket mechanism in steps 8 and 9 prevents a negative record (created by delivering goods that were not yet received) from being ignored: the positive records that share its exact characteristics absorb it before they can be reserved by anyone else.

**Worked example of the negative pocket.** In `WH/Stock`, product P: record A has on hand −5 reserved 0; record B has on hand 12 reserved 0. Both have the same lot, container and owner (all empty). A demand of 10 is reserved:
- Available quantity (step 2, product untracked) = (−5 + 12) − 0 = 7, so the wanted quantity becomes 7.
- Step 7 recomputes *available* as 12 − 0 = 12 (only positive on-hand counted).
- Negative pocket for the shared key = −5.
- Record A is skipped (its available quantity is −5, not positive).
- Record B: takeable = 12; the pocket offsets 5, leaving 7; the smaller of 7 and the remaining wanted 7 is 7. The result is a single pair (B, 7).

---

# 4. Writing a quantity onto the records

## 4.1 Increasing or decreasing an on-hand or a reserved figure

Inputs: a product, a Location, a signed on-hand delta or a signed reserved delta (at least one must be given, else "Quantity or Reserved Quantity should be set."), optionally a lot, container, owner and an incoming date.

1. Gather with **strict** matching.
2. When a lot was requested, narrow the gathered set:
   - if the on-hand delta is strictly positive, keep only the records that carry a lot;
   - otherwise keep the records that carry a lot **or** whose on-hand quantity is strictly positive. In words: never take quantity away from a lot-less record that is already negative.
3. Compute the incoming date to stamp:
   - If the Location bypasses reservation, start from an empty list of candidate dates.
   - Otherwise start from the incoming dates of the gathered records whose on-hand quantity is strictly positive.
   - Append the supplied incoming date when one was given.
   - If the list is non-empty, take its minimum; otherwise take the current instant.
   In words: **when several arrival dates exist for the same characteristics, the oldest one wins**, which is what makes first in first out stable when quantities merge.
4. If the gathered set is non-empty, take a write lock on its first record (the gathering order already put the right one first) and use that record. Otherwise create a new record with the product, Location, lot, container, owner and the computed incoming date.
5. Write on that record: the incoming date always; the new on-hand quantity as the old one plus the delta when an on-hand delta was given; the new reserved quantity as the greater of zero and the old one plus the delta when a reserved delta was given. The reserved counter can therefore never be driven below zero by this routine.
6. Return the available quantity recomputed with strict matching and negative results allowed, together with the incoming date that was stamped.

## 4.2 Changing only a reservation

Calling the routine of 4.1 with only a reserved delta. Nothing else differs.

---

# 5. Reservation algorithm

Reserving a Stock Move means creating Stock Move Lines and raising the reserved counters so that the goods are promised to this move.

## 5.1 Preconditions and the selected set

Input: a set of Stock Moves and an optional forced quantity.

1. Record, before anything is changed, the current processed quantity of every move in the set. Call it the move's *already reserved* figure. This snapshot is taken once, up front, because the reservation itself invalidates the field.
2. If no forced quantity was supplied, restrict the set to the moves that are not picked and whose status is `confirmed`, `waiting` or `partially_available`. With a forced quantity, every move of the input set is processed.
3. Pre-load the quantity records for the moves that have originating moves and do not bypass reservation, keyed by (product, Location, lot, container, owner); this cache is what the gathering algorithm consults in strict mode.

## 5.2 Per-move processing

For each move of the selected set, in the order given:

1. Work in the move's own company.
2. Compute the missing quantity in the line unit:
   ```formula
   missing_line_units = move_demand − already_reserved
   ```
   or, when a forced quantity was supplied, `missing_line_units = forced quantity`.
3. If `compare(missing_line_units, 0, product unit rounding) ≤ 0`, mark the move as fully assigned and continue with the next move.
4. Convert to the product unit:
   ```formula
   missing = convert(missing_line_units, line unit, product unit, half-up)
   ```
5. **Branch A — the move bypasses reservation** (source Location usage is vendor, customer, inventory loss or production, or the product is not storable):
   1. If the move has originating moves, compute what they brought and what the siblings already took (section 5.3) and walk the resulting map. For each key (Location, lot, container, owner) with an available quantity, create one line for the smaller of the remaining missing quantity and that available quantity, forcing that key's Location, lot, lot name, owner and container onto the line. Reduce the missing quantity. Stop when it reaches zero.
   2. If a missing quantity remains and the product is tracked by serial number and the Operation Type allows creating or using lots, create one line of exactly one unit for each whole unit of the missing quantity.
   3. Otherwise, if a missing quantity remains, look for a line of this move that can absorb it: same unit as the move, same source Location, same destination Location, same Transfer, not picked, no lot, no destination container, no source container and no owner. If one exists, raise its quantity by the missing quantity converted into the line unit rounding half away from zero. If none exists, create a new line for the missing quantity.
   4. Mark the move as fully assigned and as needing put-away.
6. **Branch B — the move does not bypass reservation and has no originating move:**
   1. If the demand is zero and no quantity was forced, mark the move as fully assigned and continue.
   2. If the move's supply method is advanced (make to order), skip the move entirely: it must wait for its supplying move.
   3. Reserve: run the reservation-quantity computation (section 3.3) for the move's product at the move's source Location with the missing quantity, loose matching, and the move's line unit; create or extend the lines as described in section 5.4; let *taken* be the total taken.
   4. If *taken* is zero at the product unit's precision, continue with the next move — the move keeps its current status.
   5. Mark the move as needing put-away. If `compare(missing, taken, product unit rounding) = 0`, mark the move fully assigned; otherwise mark it partially available.
7. **Branch C — the move does not bypass reservation and has originating moves:**
   1. Compute the distribution map (section 5.3). If it is empty, continue with the next move.
   2. Subtract from the map what this move has already reserved: for each existing line of the move with a non-zero quantity in the product unit, reduce the map entry for its (source Location, lot, source container, owner) key by that quantity, when such an entry exists.
   3. For each remaining map entry, in map order, compute
      ```formula
      need = move_real_quantity − sum(existing line quantities in product unit) − sum(quantities already taken in this loop)
      ```
      and run the reservation-quantity computation with **strict** matching for the smaller of the entry's quantity and *need*, at the entry's Location, lot, container and owner. Collect the line values; remember the taken quantity under the key (need, Location, lot, container, owner) only when new line values were produced (an update of an existing line is already reflected in the sum of the existing lines).
   4. Create all the collected lines in one operation.
   5. Walk the remembered taken quantities in order. Skip the zero ones. For each non-zero one, mark the move as needing put-away; if `need − taken` is zero at the product unit's precision, mark the move fully assigned and stop the walk; otherwise mark it partially available.
8. If the product is tracked by serial number, set the move's serial-number count to its demand.

## 5.3 What the parents brought and the siblings took

This subroutine is what makes a chained move reserve exactly the goods its predecessor delivered, in the very Location and container it delivered them to.

**Incoming side.**

1. Collect the completed moves reachable as: the move's originating moves, then *their* destination moves, then *their* originating moves. (The double hop deliberately includes sibling chains that feed the same intermediate step.)
2. Take all their detail lines and group them by the key (destination Location, lot, destination container, owner).
3. For each group sum the line quantities converted into the product unit.

**Outgoing side.**

1. Take the destination moves of this move's originating moves, minus this move itself. Call them the *siblings*.
2. Among the siblings, take those whose status is done and collect their detail lines, grouped by the key (source Location, lot, source container, owner), summing quantities converted into the product unit.
3. Among the siblings, take those whose status is partially available or assigned, **plus** those that this very reservation pass has already marked assigned or partially available, and collect their detail lines grouped by the same key; for those groups the sum is the sum of the line quantities already expressed in the product unit. These groups **replace** (not add to) any group produced by step 2 with the same key.

**Difference.** For each key present on the incoming side:
```formula
distributable(key) = incoming(key) − outgoing(key)
```
Drop the keys whose distributable quantity is not strictly greater than zero at the product unit's rounding. The result is the distribution map.

## 5.4 Creating or extending the lines for a reservation

Given the list of (record, quantity) pairs produced by section 3.3 and the move being reserved:

1. Build the *candidate* map: for each existing detail line of the move that has **no** destination container and whose product is not serial-tracked, index it by (source Location, lot, source container, owner). At most one candidate per key.
2. Merge the pairs that share the same (Location, lot, container, owner) so that one key produces one entry; sum their quantities.
3. For each merged entry:
   1. Add its quantity to the running total taken.
   2. Look up a candidate for the entry's key. If one exists, compute
      ```formula
      line_units = round_digits( convert(quantity, product unit, candidate line unit, half-up), Product Unit precision )
      back = convert(line_units, candidate line unit, product unit, half-up)
      ```
      If `round_digits(quantity, Product Unit precision) = round_digits(back, Product Unit precision)`, increase the candidate's quantity by `line_units` and go to the next entry.
   3. Otherwise, if the product is tracked by serial number **and** the Operation Type allows creating or using lots, produce one new line of exactly one unit per whole unit of the entry's quantity, each carrying the record's Location, lot, container and owner.
   4. Otherwise produce one new line for the entry's quantity, carrying the record's Location, lot, container and owner.
4. Create all the produced lines in one operation.

Creating a detail line is itself what raises the reserved counter: see section 5.5.

## 5.5 What creating a detail line does

1. Stamp the company from the move, else from the Transfer. Inherit the move's picked flag when none was given. When a quantity record was picked in the screen, copy its product, lot, container, Location and owner onto the line.
2. Insert the rows.
3. For each new line that has no move: if its Transfer is not done, link it to an existing move of the same Transfer and product (preferring one whose processed quantity is still below its real quantity); if none exists, or if the Transfer is already done, create a move for it with a demand of zero (or, for a done Transfer, with the line's quantity as demand) already in the Transfer's status.
4. For each new line whose status is not done: decide whether it must reserve. When the line has a move, it must reserve exactly when that move does not bypass reservation; when it has no move, it must reserve exactly when the product is storable and the Location does not bypass reservation. If it must reserve and its quantity in the product unit is non-zero, raise the reserved counter for (product, source Location, lot, source container, owner) by that quantity, and mark the move for status recomputation.
5. Recompute the statuses of the marked moves.
6. For each new line whose status **is** done (a line added onto an already completed Transfer), move the quantity immediately: decrease the source, compensate lot-less negatives if needed, increase the destination, then unreserve and re-reserve the downstream moves. See section 7.3.

## 5.6 Put-away after reservation

Every move marked as needing put-away has its detail lines pushed through the put-away selection of section 9.

## 5.7 Whole-container detection after reservation

Unless explicitly suppressed, the containing Transfer re-runs the whole-container detection of section 10.3 after every reservation pass.

---

# 6. Unreserving

1. Build the set of moves to unreserve: skip a move that is cancelled; skip a move that is done **and** whose destination Location usage is inventory loss; skip a move that is picked. Fail with "You cannot unreserve a stock move that has been set to 'Done'." for any other done move.
2. Walk the detail lines of those moves. A line that is picked is kept, and its move is remembered as *not to recompute*. Every other line is scheduled for deletion.
3. Delete the scheduled lines. Deleting a line lowers the reserved counter for its (product, source Location, lot, source container, owner) by its quantity in the product unit, using strict matching — but only when the line has a move and that move does not bypass reservation for the line's source Location.
4. Recompute the status of every unreserved move except those remembered in step 2 (deleting a line already triggers the recomputation for the others).

---

# 7. Changing a detail line

## 7.1 Changing an open line

When any of the source Location, destination Location, lot, source container, destination container, owner or unit changes — the destination container alone excepted — or when the quantity changes, and the line is not done and the product is storable:

1. Compute the new quantity in the product unit: when the quantity or the unit is being written, convert the new quantity from the new unit rounding half away from zero; otherwise keep the line's current figure. A negative result fails with "Reserving a negative quantity is not allowed."
2. Unreserve the **old** characteristics in full: lower the reserved counter for the old (Location, lot, container, owner) by the line's current quantity in the product unit.
3. Reserve the **new** characteristics up to what is available: raise the reserved counter for the new key by the new quantity, unless the move bypasses reservation for the new source Location. The routine of section 4.1 clamps the counter at zero, so an over-reservation simply reserves what exists.
4. Mark the move for status recomputation when the quantity or the unit actually changed.

## 7.2 Changing a done line

When the line's move is done and the product is storable:

1. Undo the original movement: decrease the destination (using the destination container) by the line's current quantity in the product unit, remembering the incoming date that came back; then increase the source by the same quantity, stamping that remembered date.
2. Remember the downstream moves that are neither done nor cancelled.
3. Post a note on the Transfer describing the change (see `interfaces.md`, section "Change notes").
4. Apply the write.
5. Redo the movement with the new values: decrease the source by the new quantity in the product unit; increase the destination (with the destination container) by the same quantity. If the available quantity at the source came back negative, free other reservations for the absolute value of that negative (section 7.4).
6. Re-check the serial-number uniqueness constraint.
7. Unreserve and re-reserve the remembered downstream moves.

## 7.3 The line date

When the date is not being written explicitly and the unit, the quantity or the picked flag is, the line's date is re-stamped with the current instant in exactly two situations: the line is becoming picked while it was not, or the line is already picked and the new quantity in the product unit is strictly greater than the old one. Lines that are draft, cancelled or done are never re-stamped this way.

## 7.4 Freeing other reservations

When a movement drives the available quantity at a source key negative, some other open document had reserved goods that no longer exist. The algorithm takes them back.

Inputs: the product, the Location, the quantity to free (a positive number in the product unit), the lot, the container, the owner, and a set of line identifiers that must never be touched.

1. Add the current line's identifier to the untouchable set.
2. If the current move bypasses reservation for that Location, do nothing.
3. Search for candidate lines: status neither done nor cancelled; exactly this product, lot, Location, owner and source container; quantity in the product unit strictly greater than zero; not picked; identifier not in the untouchable set.
4. Sort the candidates: first those that belong to the **current** Transfer, then by the Transfer's scheduled date descending (falling back to the move's date, and to zero when there is neither), then by identifier descending. In words: rob the current document first, then the latest-scheduled documents.
5. Walk the sorted candidates. For each one, remember its move as needing re-reservation. If the candidate's quantity in the product unit is less than or equal to the quantity still to free, subtract it and schedule the whole candidate for deletion; stop as soon as nothing is left to free. Otherwise reduce the candidate's quantity by the quantity still to free converted into the candidate's own unit rounding half away from zero, and stop.
6. Every move that owned a deleted candidate, and every move remembered in step 5, has its supply method reset to take-from-stock and its originating links cleared.
7. Delete the scheduled candidates.
8. Re-run the reservation on the remembered moves, walking them in reverse order.

---

# 8. Setting a processed quantity directly

## 8.1 Writing the quantity field of a move

1. For each move, check that the value respects the general decimal precision:
   ```formula
   rounded = round_digits(new_quantity, Product Unit precision)
   ```
   If `rounded ≠ new_quantity` at that precision, collect the error "The quantity done for the product *the product display name* doesn't respect the rounding precision defined on the system. Please change the quantity done or the rounding precision in your settings." and skip the move.
2. Compute the delta against what the detail lines actually add up to:
   ```formula
   delta = new_quantity − sum over lines of convert(line_quantity, line unit, move line unit, none)
   ```
3. If the delta is strictly positive, run the distribution of section 8.2 with the move's whole new quantity.
4. If the delta is strictly negative, run the reduction of section 8.3 with the absolute delta.
5. If any errors were collected, fail with all of them joined by line breaks.

## 8.2 Distributing a processed quantity over the lines

Input: a target quantity expressed in the move's line unit. Convert it once into the product unit without rounding; call it *remaining*. Keep the original figure as *total*.

1. Walk the existing detail lines in order. For each line:
   1. Let *line quantity* be its quantity; skip the line when that is negative.
   2. Convert *line quantity* into the product unit without rounding when the line's unit differs from the product unit.
   3. If *remaining*, expressed back in the move's line unit, is zero, schedule the line for deletion and continue.
   4. If *line quantity* is strictly greater than *remaining*, set the line's quantity to *remaining* (converted into the line's own unit without rounding when the units differ), set *remaining* to zero and continue.
   5. If the line already has a destination container, subtract *line quantity* from *remaining* and continue without touching the line.
   6. Otherwise subtract the smaller of *remaining* and *line quantity* from *remaining*. If *remaining*, expressed in the move's line unit, is now zero or less, continue.
   7. Try to enlarge this line: run the reservation-quantity computation with strict matching at the line's Location, lot, container and owner for *remaining*. Let *extra* be the total it offers. Remember the records it touched so that they are not used twice. If *extra* is less than or equal to *remaining*, reduce *remaining* by *extra*, then set the line's quantity to *extra* plus its own original quantity (converted into the line's unit when needed).
2. If *remaining*, expressed in the move's line unit, is still strictly positive and the move does not bypass reservation, run the reservation-quantity computation with loose matching at the move's source Location for *total*. Walk the resulting pairs, skipping the records already consumed in step 1.7; for each one create a new line for the smaller of *remaining* and the pair's quantity and reduce *remaining*; stop as soon as *remaining* reaches zero.
3. If *remaining*, expressed in the move's line unit, is still strictly positive:
   - when the product is not serial-tracked, create a single line for the whole remaining quantity expressed in the move's line unit;
   - when it is serial-tracked, create one line of exactly one unit, in the product unit, for each whole unit of the remaining quantity.
4. Apply the collected create, update and delete operations to the move's line list.
5. Push the newly created lines through the put-away selection of section 9.

## 8.3 Reducing a processed quantity

Input: a positive quantity to remove, expressed in the move's line unit.

1. Walk the detail lines **in reverse order of identifier** — the reverse of the order in which the removal strategy created them, so that the goods reserved last are the ones given back first.
2. Skip a picked line when the calling context asks to reduce only unpicked lines.
3. Stop as soon as the quantity to remove is zero at the move's line unit.
4. For the current line, compute the amount to remove from it as the smaller of its own quantity and the quantity to remove converted into the line's unit without rounding. Skip the line when that amount is zero at the line's unit.
5. If the amount equals the line's whole quantity and the line is neither done nor cancelled, schedule the line for deletion; otherwise reduce the line's quantity by that amount.
6. Reduce the quantity still to remove by the amount converted back into the move's line unit without rounding.
7. Delete the scheduled lines. Each deletion lowers the reserved counters.

---

# 9. Put-away selection

Put-away answers: "goods of product P (possibly inside container C) are arriving in Location L — where exactly should they go?"

## 9.1 Entry point

Inputs: the arrival Location, the product, a quantity in the product unit, optionally a container, optionally a packaging unit, and optionally a map of additional quantities per Location (used to account for goods that the current operation is already sending to those Locations).

1. Determine the container type: the container's type when a container was given, else the packaging unit's container type when a packaging unit was given, else none.
2. Build the product set: the products named in the calling context, plus the product given as input.
3. Build the category chain: when all the products in the set share exactly one category, that category and all of its ancestors; otherwise the empty set.
4. Select the arrival Location's own put-away rules that pass all three tests:
   - the rule names no product, or names a product in the product set;
   - the rule names no category, or names a category in the chain;
   - the rule names no container type, or the determined container type is among them.
5. Sort the selected rules by specificity, most specific first. The sort key is the four-tuple, compared as booleans in descending order:
   1. the rule names at least one container type;
   2. the rule names a product;
   3. the rule's category is the product's *own* category (not merely an ancestor);
   4. the rule names a category at all.
   Ties keep the rules' own order, which is priority ascending then product.
6. Determine the candidate Locations: the Locations named in the calling context when there are any, else the arrival Location's internal descendants (itself included when it is internal).
7. If at least one rule was selected, build the occupancy map (section 9.2) and run the rule walk (section 9.3).
8. If the rule walk returned a Location, that is the answer.
9. Otherwise, when the arrival Location's usage is virtual and there is at least one candidate Location, the answer is the first candidate Location. In every other case the answer is the arrival Location itself.

## 9.2 The occupancy map

The occupancy map gives, per candidate Location, the figure that the capacity checks compare against. It is only built when at least one candidate Location carries a storage category.

**When a container with a container type is being put away**, the figure is a *count of containers*:

```formula
occupancy(location) = number of distinct destination containers of that type on open move lines targeting the location
                    + number of distinct containers of that type already stored in the location
```

Open means the line's status is not draft, done or cancelled. Lines listed as excluded in the calling context are skipped.

**Otherwise** the figure is a *quantity of the product in the product unit*:

```formula
occupancy(location) = sum of on-hand quantities of the product's records in the location
                    + sum over open move lines targeting the location of convert(line quantity, line unit, product unit, half-up)
```

Finally, every entry of the supplied additional-quantity map is added to the corresponding Location's figure. Those entries may be negative: a screen recomputing put-away for lines it is about to rewrite passes the negation of those lines' own quantities so that they are not counted twice.

## 9.3 The rule walk

Keep a set of *already rejected* Locations, initially empty. For each selected rule in order:

1. Let *target* be the rule's destination sublocation.
2. If the rule's sublocation mode is **last used**, look for the most recent completed detail line whose destination Location is inside *target* and whose product is this product — and, when the rule names container types, whose destination container is of one of those types — ordered by date descending, limit one. When such a line exists, replace *target* by its destination Location.
3. Let *children* be the internal descendants of *target*.
4. **If the rule names no storage category:**
   - if *target* is already in the rejected set, go to the next rule;
   - if *target* passes the capacity check of section 9.4 with the occupancy figure of *target*, return *target*;
   - otherwise go to the next rule.
5. **If the rule names a storage category**, narrow *children* to the descendants whose storage category is exactly that category, then:
   1. **First pass — prefer a Location that already holds the same thing.** For each Location of *children* not already rejected:
      - when a container type was determined: if the Location holds at least one quantity record whose container is of that type, run the capacity check; return the Location when it passes, otherwise add it to the rejected set;
      - otherwise: if the Location's occupancy figure is strictly greater than zero, run the capacity check; return the Location when it passes, otherwise add it to the rejected set.
   2. **Second pass — any Location of the right category.** For each Location of *children* whose storage category is that category and which is not already rejected, run the capacity check; return the Location when it passes, otherwise add it to the rejected set.
6. When every rule has been walked without returning, return nothing.

## 9.4 The capacity check

Input: a Location, the product, a quantity in the product unit, optionally a container, and the Location's occupancy figure.

1. If the Location has no storage category, the check **passes**.
2. Let *positive records* be the Location's quantity records whose on-hand quantity is strictly greater than zero at the product unit's precision.
3. **Mixing policy.**
   - `empty` — if there is at least one positive record, the check **fails**.
   - `same` — take the product as input, or, when no product was given (a container is being put away), the products named in the calling context. The check **fails** when there is at least one positive record whose product is not that product, or when more than one product is involved. The check also **fails** when at least one open detail line (status neither done nor cancelled) targets this Location with a different product.
   - `mixed` — no test.
4. Compute the Location's forecasted weight (section 11.1), excluding the detail lines listed as excluded in the calling context.
5. **Weight and count, container branch** (a container with a container type was given):
   - Let *incoming package weight* be the sum over the open detail lines whose destination container is that container of `line quantity in the product unit × product weight`.
   - If `storage category maximum weight < forecasted weight + incoming package weight`, the check **fails**.
   - Look for a container capacity of the storage category for that container type. If one exists and `occupancy ≥ its quantity`, the check **fails**.
6. **Weight and quantity, product branch** (no container, or a container with no type):
   - If `storage category maximum weight < forecasted weight + product weight × quantity`, the check **fails**.
   - Look for a product capacity of the storage category for that product. If one exists and `occupancy ≥ its quantity`, the check **fails**.
   - If one exists and `quantity + occupancy > its quantity`, the check **fails**.
7. Otherwise the check **passes**.

Note that step 6 contains two distinct tests: the first rejects a Location that is already at or over capacity even for a zero quantity (so that a line with no quantity yet is not directed at a full shelf), and the second rejects a Location that would be pushed over capacity by this quantity.

**Worked example.** Storage category "Small shelf" with maximum weight 50 and a product capacity of 30 units of product P (weight 1 per unit). Location `WH/Stock/Shelf A` carries that category and already holds 22 units of P. A put-away of 5 units is attempted with no container.
- Forecasted weight = 22.
- Weight test: 50 < 22 + 1 × 5 = 27? No — passes.
- First quantity test: occupancy 22 ≥ 30? No — passes.
- Second quantity test: 5 + 22 = 27 > 30? No — passes.
- The Location is accepted.
A put-away of 12 units instead: 12 + 22 = 34 > 30, so the Location is rejected and the walk continues to the next candidate.

## 9.5 Applying put-away to a set of detail lines

Put-away is applied to lines, not to moves, because different lines of one move may be directed to different shelves.

1. If the calling context forbids put-away, do nothing.
2. Group the lines by their destination container's **outermost** container.
3. For each group:
   - Let *candidate Locations* be the internal descendants of the moves' destination Location.
   - Let *excluded lines* be the identifiers of every line of this group (so that a line never counts its own quantity as occupancy).
   - **If the group's outermost container has a container type**, resolve one Location for the whole group, passing the group's products in the calling context and the candidate Locations, and write that Location on every line of the group.
   - **Else if the group has an outermost container but no type**, resolve a Location line by line, removing each line from the excluded set once it has been resolved, and stop early once more than one distinct Location has been used. If, at the end, more than one Location was used, the grouping is abandoned: every line of the group is reset to its move's destination Location. In words: a container may not be split across shelves.
   - **Else (no container at all)**, resolve a Location line by line, passing the move's packaging unit so that container-type rules can apply, and removing each line from the excluded set once resolved.

---

# 10. Containers

## 10.1 Whole-container test

A container is fully represented by a set of detail lines when, grouping both sides by (product, lot):

```formula
for every key: sum of container record quantities (key) = sum of line quantities in the product unit (key)
```

and no key exists on one side only. The comparison uses the decimal-precision setting named `Product Unit of Measure`. An empty set of lines is treated as a match.

## 10.2 The container weight

```formula
container_weight = container type base weight
                 + sum over all descendant containers of ( their container type base weight )
                 + sum over all contained quantity records of ( record on-hand quantity × product weight )
```

When the weight is asked for in the context of one Transfer, a different figure is produced, because the container's contents are still being assembled:

```formula
container_weight_for_transfer = container type base weight
      + sum over the transfer's detail lines whose destination container is this container of
            ( convert(line quantity, line unit, product unit, none) × product weight )
      + sum over every container that has this one as destination container, directly or transitively, of
            ( that container's type base weight + the same line sum for that container )
```

## 10.3 Whole-container detection on a Transfer

1. Group the Transfer's detail lines by source container. Skip the group with no container.
2. For each group: when the Transfers owning those lines form a single document (or a single batch) **and** the container is fully represented by the lines of that document filtered to storable products and to the container or any of its descendants (section 10.1), and the container's type is not reusable, then write onto every line of the group that has no destination container yet and whose status is neither done nor cancelled: the destination container is the container itself, and the entire-package flag is true.
3. Finally, run the container promotion of section 10.4 on the destination containers of all the Transfer's lines.

## 10.4 Container promotion for whole containers

When every child of a container has been added, the container itself is considered added.

1. Group the containers by their parent container.
2. For each parent that is not empty: when the parent's children are exactly the group and the parent's type is not reusable — and, when a list of allowed containers was supplied, the parent is in that list — set the destination container of every container of the group to the parent.
3. If any container of the set now has a destination container, recurse on those destination containers so that the next level up is considered too.

## 10.5 Applying the destination containers at completion

1. Restrict to the containers not yet processed.
2. Group them by their destination container.
3. For the group whose destination container is empty, clear the parent container of every container of the group and mark them processed.
4. For every other group:
   1. Read the Locations of the group's containers after the movement. If there is more than one, fail with "Packages *the list of container names* are moved to different locations while being in the same container *the container name*."
   2. Read the destination container's own contents (records with a non-zero quantity). If there are any and their Location is not the group's new Location, fail with "Can't move a container having packages in another location (*the other location display name*) to a different location (*the new location display name*)."
   3. Write on every container of the group: the parent container becomes the destination container, and the destination container is cleared. Mark them processed.
5. If any processed container's parent itself has a destination container or a parent, recurse on those parents.

## 10.6 Put in pack

1. Split the selection: *lines to pack* are the lines that are neither done nor cancelled, have a strictly positive quantity and have **no** destination container; *containers to pack* are the outermost destination containers of the remaining such lines. When the selection is not forced and at least one line among those with a positive quantity is picked, the unpicked ones are ignored entirely.
2. Fail when the selection spans more than one Operation Type: "You cannot pack products into the same package when they are from different transfers with different operation types".
3. For the lines to pack:
   1. If they point at more than one destination Location, open the destination chooser first; picking a Location rewrites all of them.
   2. If the Operation Type asks to set a container type and none of a container, a container type or a name was supplied, open the put-in-pack wizard first.
   3. Otherwise create or take the container: the given container when one was supplied; else a new container with the given name and the given type; else a new container with the given name and, when the move's packaging unit resolves to exactly one container type, that type.
   4. When exactly one line is being packed, re-run put-away for that line with the new container so that the container is directed to a Location that can accept it.
   5. Write the container as the destination container of every line being packed.
   6. When the Operation Type asks to print the container label automatically, return the label document action instead of the container.
4. For the containers to pack (nesting an existing container into a new one):
   1. Remember the whole current destination chain of those containers.
   2. Create or take the new container as in step 3.3.
   3. Set it as the destination container of each of them. Then clear the destination container of every remembered container that no longer has any open detail line, so that a broken chain does not leave stale links.
   4. Re-run put-away on the new container's detail lines, because the outermost container changed.

## 10.7 Unpacking

1. Clear the parent container of every child container.
2. If the container directly holds quantity records, relocate them to the same Location with no container and the reference "Quantities unpacked", then run the record housekeeping of section 12.

## 10.8 Removing containers from a Transfer

1. Collect the whole destination chain of the containers being removed and all of their open detail lines.
2. For each of those lines (restricted to the Transfers named in the context when the context names any): when the line's destination container is one of the removed containers, delete the line if it was added as an entire package, otherwise merely clear its destination container.
3. Delete the collected lines; clear the destination container of the collected updates.
4. Delete the moves that had a zero demand and no detail lines left.
5. Clear the destination container of every container that had one of the removed containers as destination, and of the removed containers themselves.
6. Clear the destination container of every container of the remembered chain that no longer has any open detail line.
7. Re-run put-away on the surviving lines, because the outermost container changed.

---

# 11. Weights and volumes

## 11.1 Location weight

Two figures are produced per Location.

```formula
net_weight(location)      = Σ over the location's quantity records ( on_hand × product weight )

forecast_weight(location) = net_weight(location)
                          − Σ over open detail lines leaving the location ( quantity in product unit × product weight )
                          + Σ over open detail lines arriving in the location ( quantity in product unit × product weight )
```

Open means the line's status is not draft, done or cancelled. When a set of excluded line identifiers is supplied, those lines are omitted from both sums.

## 11.2 Transfer weights

```formula
weight_bulk(transfer) = Σ over the transfer's detail lines that have no destination container
                          ( convert(line quantity, line unit, product unit, half-up) × product weight )
```

```formula
shipping_weight(transfer) = weight_bulk(transfer)
                          + Σ over the outermost destination containers of the transfer's lines
                              ( the container's manually entered shipping weight, when set,
                                otherwise the container weight for this transfer of section 10.2 )
```

The shipping weight is stored and may be overwritten by hand.

```formula
shipping_volume(transfer) = Σ over the transfer's moves
                              ( convert(processed quantity, line unit, product unit, half-up) × product volume )
```

## 11.3 Batch load

```formula
estimated_shipping_weight(batch) =
      Σ over the destination containers of the batch's lines that have a manual shipping weight ( that weight )
    + Σ over the destination containers of the batch's lines that have no manual weight but have a type ( the type's base weight )
    + Σ over the batch's detail lines whose destination container was not already counted with a manual weight
          ( product weight × line quantity in the product unit )
```

```formula
estimated_shipping_volume(batch) =
      Σ over the destination containers with a type but no manual weight
          ( length × width × height ÷ 1 000 000 000 )
    + Σ over the batch's detail lines whose destination container was not already counted with a manual weight
          ( product volume × line quantity in the product unit )
```

```formula
used_weight_percentage = 100 × estimated_shipping_weight ÷ vehicle_category_weight_capacity
used_volume_percentage = 100 × estimated_shipping_volume ÷ vehicle_category_volume_capacity
```

Both percentages are left empty when the corresponding capacity is zero or absent.

**Worked example.** A batch carries one container of type "Pallet" (base weight 20, dimensions 1200 × 800 × 1500 in millimetres) holding 40 units of a product weighing 2 and of volume 0.01, plus 6 loose units of the same product. The container has no manual shipping weight.
- Weight = 20 (base) + (40 + 6) × 2 = 20 + 92 = **112**.
- Volume = 1200 × 800 × 1500 ÷ 1 000 000 000 = 1.44, plus 46 × 0.01 = 0.46, total **1.9**.
- With a vehicle category whose maximum weight is 800: percentage = 100 × 112 ÷ 800 = **14 percent**.

---

# 12. Record housekeeping

Three maintenance passes run together as one housekeeping task, in this order: merge, clean reservations, delete empties. The task runs when the quantity screens are opened, unless the system parameter that skips it is set.

## 12.1 Merging duplicate records

Concurrent transactions can create two records with the same characteristics. The merge pass collapses them.

1. Group the records by (product, company, Location, lot, container, owner) and keep only the groups with more than one row. When the pass is run on a restricted set, the grouping is limited to the Locations and products of that set.
2. For each group, keep the row with the lowest identifier and write onto it:
   ```formula
   kept.on_hand   = Σ on_hand over the group
   kept.reserved  = max( 0, Σ reserved over the group )
   kept.counted   = Σ counted over the group
   kept.in_date   = min( in_date over the group )
   ```
3. Delete the other rows of the group.
4. The whole pass runs inside a save point; if it fails, the failure is logged and nothing is changed.

## 12.2 Cleaning reservations

This pass makes the reserved counters agree with the detail lines that justify them.

1. Group the records with a non-zero reserved counter by (product, Location, lot, container, owner); for each group note the total reserved and the records themselves.
2. Group the open detail lines — status waiting-another-move, waiting, partially available or assigned, non-zero quantity in the product unit, storable product — by the same key, summing quantities in the product unit. Call this the *justified* map.
3. For each reserved group:
   - if the Location bypasses reservation, lower the reserved counter by the whole reserved total (bringing it to zero);
   - otherwise, when the reserved total differs from the justified figure at the product unit's precision, adjust the counter by `justified − reserved`.
   - Remove the key from the justified map once handled.
4. For each key left in the justified map (lines that reserve against no counter at all): skip it when the Location bypasses reservation or when the product-level bypass hook says so; otherwise raise the counter by the justified quantity.

## 12.3 Deleting empty records

Delete every record for which all four of the following hold, where *n* is the greater of 6 and twice the number of decimal digits of the shipped product-unit precision record:

- the on-hand quantity rounded to *n* digits is zero, or is absent;
- the reserved quantity rounded to *n* digits is zero;
- the counted quantity rounded to *n* digits is zero, or is absent;
- no user is assigned for counting.

---

# 13. Move merging

Two Stock Moves of the same Transfer that describe the same thing are collapsed into one so that the document stays readable and reservation stays efficient.

## 13.1 The merge key

Two moves may merge only when **every** field of the key is equal:

1. product
2. unit price
3. supply method
4. source Location
5. intermediate destination Location
6. final Location
7. line unit
8. owner restriction
9. original return move
10. propagate-cancel flag
11. transfer description
12. never-attribute values

To that base key:

- the scheduled date is **added** when the system parameter that restricts merging to the same date is set;
- the supply method is **removed** when the merge is an "extra move" merge;
- the deadline is **added** unless the system parameter that ignores the deadline when merging is set.

Floating-point members of the key are compared as strings formatted at their own precision, so that two values that differ only by representation error still merge. The unit price uses the smaller of the currency's decimal places and the price precision setting.

## 13.2 The merge algorithm

1. Build the candidate sets. Without an explicit target, every Transfer of the moves being merged contributes its whole move list as one candidate set. With an explicit target, one candidate set is formed from the target plus the moves being merged.
2. Compute the list of key fields (section 13.1).
3. Separate the **negative** moves — those whose real quantity is strictly less than zero — from the rest. Detach them from their Transfer immediately, so that the notes written on the Transfer are not polluted. Their key is the same key minus the transfer description.
4. For each candidate set:
   1. Drop the moves that are done, cancelled or draft, and the negative moves.
   2. Group the remainder by the full key. For each group of more than one move:
      - re-link every detail line of the group to the group's first move;
      - write the merged values on the first move (section 13.3);
      - schedule the other moves for deletion and remember the first move as merged.
   3. Index the group's surviving move under the negative key.
5. For each negative move, walk the positive moves indexed under the same negative key:
   - Compute the combined value:
     ```formula
     new_total_value = positive_real_quantity × positive_unit_price + negative_real_quantity × negative_unit_price
     ```
   - If the positive move's demand is greater than or equal to the absolute demand of the negative move, the negative move is fully absorbed: add the negative demand (a negative number) to the positive demand; set the positive unit price to
     ```formula
     new_unit_price = round_digits( new_total_value ÷ positive_real_quantity , Product Price precision )
     ```
     or zero when the real quantity is zero; link to the positive move those destination moves of the negative move whose source Location equals the positive move's destination Location, and those originating moves of the negative move whose destination Location equals the positive move's source Location. Schedule the negative move for deletion. When the positive demand has become zero, schedule the positive move for cancellation. Stop walking.
   - Otherwise the positive move is fully consumed: add the positive demand to the negative demand, recompute the negative move's unit price with the same formula on its own real quantity, set the positive demand to zero and schedule the positive move for cancellation. Continue with the next positive move.
6. Every move scheduled for deletion or cancellation has its propagate-cancel flag cleared first.
7. Cancel and then delete the moves scheduled for deletion.
8. Cancel the moves scheduled for cancellation that are not picked.
9. Return the input moves plus the merged survivors, minus the deleted ones.

## 13.3 The merged values

```formula
merged_demand   = Σ demands over the group            (or the first move's demand, for an extra-move merge)
merged_date     = min(dates)  when every transfer of the group ships as soon as possible
                = max(dates)  otherwise
merged_dest     = union of all destination moves
merged_orig     = union of all originating moves
merged_state    = relevant status among the moves     (see state-machines.md, section 2.3)
merged_origin   = the distinct origins joined by "/"
```

---

# 14. Splitting a move

Input: a quantity to split off, expressed in the **product** unit.

1. Fail when the move is done or cancelled: "You cannot split a stock move that has been set to 'Done' or 'Cancel'." Fail when it is draft: "You cannot split a draft move. It needs to be confirmed first."
2. If the quantity is zero at the product unit's precision, produce nothing.
3. Decide the unit of the new move:
   ```formula
   line_units = convert(split_quantity, product unit, line unit, half-up)
   back       = convert(line_units, line unit, product unit, half-up)
   ```
   When `split_quantity` and `back` are equal at the Product Unit precision, the new move keeps the original line unit and its demand is `line_units`. Otherwise the new move is created in the product unit and its demand is `split_quantity`.
4. Build the new move's values by copying the original and overriding: the demand as decided; the supply method; the destination moves that are neither done nor cancelled; all the originating moves; the original return move; the unit price; the deadline. When an owner restriction is forced, it is set. When the calling context forces a source Location, it is set.
5. Reduce the original move's demand:
   ```formula
   new_demand = round_digits( convert( max(0, original_real_quantity − split_quantity), product unit, line unit, none ), Product Unit precision )
   ```
   and write it with unreservation suppressed, so that the existing reservation is preserved on the original move.
6. Recompute the original move's status.
7. Return the values for the new move; the caller creates it.

---

# 15. Backorder creation on the moves

1. For each move being completed, compare the processed quantity with the demand at the **Product Unit precision** (a digit count), not at the line unit's rounding step. Only when the processed quantity is strictly smaller does a backorder move arise.
2. Compute the quantity to carry over:
   ```formula
   carry_over = convert( demand − processed , line unit, product unit, half-up )
   ```
3. Split the move by that quantity (section 14) and collect the resulting values.
4. Create all the backorder moves in one operation.
5. Confirm them with merging disabled and with supply-request creation disabled, and with whole-container detection suppressed (their destination containers must not be disturbed while the parent Transfer is being completed).

---

# 16. Completion algorithm

This is the algorithm that actually moves the goods. Input: a set of Stock Moves and a flag saying whether backorders are forbidden.

1. Confirm the moves of the set that are still in draft, with merging disabled. Add the results to the set.
2. Restrict the set to the moves that exist and whose status is neither done nor cancelled.
3. **Prune.** For each move of the set:
   - when the move is picked, schedule for deletion every one of its detail lines that is not picked;
   - when the move is not an adjustment move and (its processed quantity is not strictly positive **or** it is not picked): cancel it if its demand is zero at the line unit, or if backorders are forbidden.
   Delete the scheduled lines.
4. **Select.** Keep the moves that are not cancelled, that either have a strictly positive processed quantity or are adjustment moves, and that are picked. Call this set *to complete*.
5. Run the company consistency check on *to complete*.
6. **Backorder moves.** Unless backorders are forbidden, run section 15 on *to complete*.
7. **Move the goods.** Sort the detail lines of *to complete* in their natural order (destination container descending, then identifier) and run the line completion of section 17 on them.
8. **Container consistency.** For every destination container of the picked lines of *to complete* that holds more than one quantity record: if those records with a strictly positive quantity sit in more than one Location, fail with "You cannot move the same package content more than once in the same transfer or split the same package into two location." followed by a line break, "Package: " and the container name.
9. If any line of *to complete* has the same source container and destination container, run the empty-record deletion of section 12.3.
10. Write onto *to complete*: status done, and the date set to the current instant.
11. **Push.** Take the moves of *to complete* that must not skip the push step — a move skips it when it is an adjustment move, or when one of its destination moves has a source Location that is an ancestor or a descendant of this move's destination Location — and apply the push rules (section 18).
12. **Propagate.** Group the destination moves of *to complete* by company and re-run the reservation on each group, with elevated rights, in that company.
13. If the calling context marks the operation as a scrap, stop here and return the moves.
14. **Transfer backorder.** When the moves belong to a Transfer and backorders are not forbidden, create the Transfer backorder (section 19). When any move of that backorder is assigned, run whole-container detection on it.
15. Re-check the serial-number uniqueness constraint over the destination Locations and lots of *to complete*.
16. Run the order-synchronisation hook (a no-operation in this domain; other domains use it to write back delivered quantities).
17. Return *to complete*.

---

# 17. Completing the detail lines

## 17.1 Checks

For every line, in order:

1. Check the rounding: let
   ```formula
   unit_rounded = round(line quantity at the line unit's rounding step, half away from zero)
   digit_rounded = round_digits(line quantity, Product Unit precision)
   ```
   When those two differ at the Product Unit precision, fail with "The quantity done for the product \"*the product display name*\" doesn't respect the rounding precision defined on the unit of measure \"*the line unit name*\". Please change the quantity done or the rounding precision of your unit of measure."
2. Classify by quantity:
   - **Strictly positive.** When the product is not tracked, the line is fine. Otherwise: when the line is not excluded from the lot requirement, record the line as *tracked without lot*. A line is excluded when its move has an Operation Type, or the move is an adjustment move, or the line already has a lot, or the move belongs to a scrap. Then: when there is no Operation Type, or the line already has a lot, or the Operation Type allows neither creating nor using existing lots, the line is fine. Otherwise, when the Operation Type allows creating lots, record the line for lot resolution under the key (product, company); when it does not, record it as *tracked without lot*.
   - **Strictly negative.** Fail with "No negative quantities allowed".
   - **Zero.** When the line is not an adjustment line, schedule it for deletion.
3. **Lot resolution.** For each (product, company) group recorded above, search the existing Lots of that product whose company is that company or empty and whose name is one of the typed names. For each line: when a Lot with that name was found, link it; when the line has a typed name but no Lot was found, record it for creation; when the line has no typed name at all, record it as *tracked without lot*.
4. If any line was recorded as *tracked without lot*, fail with "You need to supply a Lot/Serial Number for product:\n" followed by one line per distinct product display name, each prefixed by "- ".
5. Create the missing Lots (section 17.2) and link them.
6. Delete the lines scheduled for deletion. Run the company consistency check on the survivors.

## 17.2 Creating the missing lots

1. Build one creation entry per distinct (product, typed name) pair, except that for a product tracked by **lot** only one Lot is created per pair while for a product tracked by **serial number** one Lot is created per line even when the names repeat.
2. Each entry carries the typed name and the product; it also carries the line's company when the product itself belongs to a company and the line's company is that company or one of its descendants.
3. Create the Lots and write each one back onto all the lines that share its key.

## 17.3 Moving the quantities

1. Pre-load the quantity records for the surviving lines' products across their source and destination Locations, restricted to the lines' lots or to no lot.
2. Unless the calling context says to ignore destination containers, build the Package History snapshots (section 17.4) **before** anything moves, and create them.
3. For each surviving line, in order:
   1. Release the reservation: lower the reserved counter at the source by the line's quantity in the product unit. (Skipped when the move bypasses reservation for the source Location.)
   2. Take the goods out: decrease the on-hand quantity at the source by the line's quantity in the product unit, using the line's lot, source container and owner. Remember the returned available quantity and the returned incoming date.
   3. If the returned available quantity is negative and the line has a lot, try to compensate with untracked stock: read the strictly-matched available quantity at the source with no lot; when it is non-zero, take the smaller of it and the absolute moved quantity away from the lot-less records and add the same amount to the lot's records. This keeps the lot-level figure consistent instead of leaving a negative lot beside a positive untracked pocket.
   4. Put the goods in: increase the on-hand quantity at the destination Location by the same quantity, with the line's lot, the line's **destination** container and the line's owner, stamping the remembered incoming date.
   5. If the available quantity returned in step 3.2 was negative, free other reservations (section 7.4) for its absolute value, excluding every line already processed in this loop.
   6. Add the line to the excluded set.
4. Unless destination containers are being ignored, apply the destination containers to the containers themselves (section 10.5).
5. Stamp the current instant as the date of every surviving line.

## 17.4 Package History snapshots

For every container in the full destination chain of the lines' destination containers, one snapshot is created carrying: the container, its full name, its current Location, its destination Location, the lines whose destination container is that container, the Transfers of those lines, the parent container before the move and that parent's full name, the destination container and its destination-path name, and the outermost destination container.

---

# 18. Push rules

When goods arrive somewhere and a rule says they must continue, the push step creates the continuation.

## 18.1 Selecting the rule

For each completed move:

1. Determine the Warehouse to consider: the move's own Warehouse, else the Warehouse of the Transfer's Operation Type. When the move's destination Location belongs to a company that is not among the active ones, the rule search is performed with elevated rights, the move is read with all the user's companies allowed, and no Warehouse is considered.
2. Collect the container types of the containers in the destination chain of the move's detail lines and add their Routes to the move's own preferred Routes.
3. Ask the rule engine of `../replenishment-and-procurement/` for a push rule for the move's product at the move's destination Location, passing those Routes, the Warehouse and the packaging unit.
4. While the returned rule carries an applicability filter and the move does not match it, add the rule to an exclusion list and ask again excluding those rules.
5. Accept the rule only when the move is not a return, or when it is a return whose original move's destination Location differs from the rule's destination Location. (This is what stops the return of a return from re-pushing.)

## 18.2 Running the rule

**Automatic, no step added.**

1. Remember the move's current destination Location.
2. Write the new date — the move's date plus the rule's lead time in days — and the rule's destination Location onto the move.
3. Re-point the detail lines: their destination Location becomes the put-away result for the new destination Location, or that Location itself.
4. If the destination Location actually changed, run the push step again on the same move and return the first new move it produces (this walks a chain of transparent rules).

**Manual operation.**

1. Compute the new date as above.
2. Build the copy values:
   - demand: the move's processed quantity, or its demand when that demand is negative;
   - origin: the move's origin, else the Transfer reference, else the single character `/`;
   - source Location: the move's destination Location;
   - destination Location: the rule's destination Location, overridden by the move's final Location when that final Location is a descendant of the rule's destination Location, and overridden by the contact's own customer Location when the move has a contact and the rule's destination Location has customer usage;
   - final Location: the move's final Location when the move's destination Location is not already inside it, otherwise empty;
   - the rule, the new date, the move's deadline, the company (the rule's, else the rule's Warehouse's, else the Operation Type's Warehouse's), no Transfer, the rule's Operation Type, the rule's propagate-cancel flag, the Warehouse (the rule's, else the move's destination Location's), and the advanced supply method.
3. Copy the move with those values.
4. When the new move must skip the push step, force its destination Location to its final Location.
5. When the new move bypasses reservation, force its supply method to take-from-stock.
6. When the new move's source Location does **not** bypass reservation, link the new move as a destination move of the original.
7. Return the new move.

## 18.3 Re-wiring the chain

After the rule has run, for each destination move of the original that is not the newly created one:

- when a new move was created, the original has a final Location, and the destination move's source Location equals that final Location, the destination move is *propagated*: it is unlinked from the original and linked to the new move instead;
- otherwise, when the destination move's source Location is not inside the original's destination Location, the link is broken: the destination move loses the originating link, its supply method is reset to take-from-stock and its status is recomputed.

All the new moves are then confirmed with elevated rights.

---

# 19. Transfer backorder

1. Determine the moves to carry over: those of the Transfer whose status is neither done nor cancelled. (When an explicit list is supplied — the split action — that list is used instead, restricted to the Transfer.)
2. Recompute the statuses of those moves.
3. If there are none, no backorder is created.
4. Copy the Transfer with an empty reference, no moves, no detail lines, the original as back-order link, and the same return link.
5. Move the carried-over moves and their detail lines into the backorder and clear their picked flag.
6. Clear the backorder's responsible.
7. Post on the original Transfer: "The backorder *link to the backorder* has been created."
8. When the backorder's Operation Type reserves at confirmation, run the availability check on it.

---

# 20. Validation algorithm

This is the complete sequence performed when a person presses Validate on one or more Transfers.

1. Drop the Transfers that are already done from the working set.
2. **Immediate transfer.** Confirm the Transfers of the working set that are in draft. Then, for every move of those draft Transfers whose processed quantity is zero while its demand is not, copy the demand into the processed quantity.
3. **Sanity check** (unless suppressed by the caller, which is what a batch validation does):
   1. Compute, per Transfer, whether it has *no quantity at all*: let *has pick* be true when at least one move that is neither done nor cancelled is picked; the Transfer has no quantity when every move that is neither done nor cancelled — restricted to the picked ones when *has pick* is true — has a zero processed quantity at the Product Unit precision.
   2. Collect the Transfers with no moves and no detail lines at all.
   3. For the Transfers whose Operation Type allows creating or using lots, collect the lines that need checking: for a Transfer that has no quantity at all, every line whose product is tracked; for the others, only the lines whose product is tracked, which are picked and whose quantity is non-zero. (When the check is run for a batch, the two cases are decided once for the whole batch instead of per Transfer.) Any such line with neither a Lot nor a typed name marks its Transfer and its product.
   4. Report. When a single Transfer is being validated: fail with "You can’t validate an empty transfer. Please add some products to move before proceeding." if it has no moves; else fail with "Transfer trouble alert! Validating a zero quantity transfer? You're not moving invisible goods around are you?\nSet some quantities and let's get moving!" if it has no quantity; else fail with "You need to supply a Lot/Serial number for products *the comma-separated product display names*." if lots are missing. When several Transfers are being validated, build one message: "Transfers *the comma-separated references*: Please add some items to move." for the empty ones, then "\n\nTransfers *the references*: You need to supply a Lot/Serial number for products *the product display names*." for the ones missing lots; fail with that message when it is not empty.
4. **Pre-completion hooks.**
   1. Remember the identifiers of the Transfers being validated in the calling context, so that a wizard can resume the validation.
   2. For each Transfer: when at least one move has a processed quantity and no move other than the scrap moves is picked, mark every move of the Transfer as picked.
   3. Unless the backorder step is suppressed, run the backorder decision (section 20.1). When it returns a set of Transfers, open the backorder wizard on them and stop; the validation resumes when the person answers.
   4. Run the text-message warning hook: when the company has text-message validation on, the Operation Type is a delivery, the contact has a telephone number, and the company has not yet been warned, open the one-time warning wizard and stop.
5. **Completion.** Split the working set:
   - *not to backorder* = the Transfers whose Operation Type never creates backorders, plus — when the calling context names Transfers that must not be backordered — those of them whose Operation Type does not always create backorders;
   - *to backorder* = the rest.
   Complete the first group with backorders forbidden, then the second group with backorders allowed. Completing a Transfer means: run the company consistency check; write the Transfer's owner onto every move as owner restriction and onto every detail line as owner; run the move completion of section 16 on the moves whose status is draft, waiting-another-move, partially available, assigned or waiting; stamp the completion instant and reset the priority to normal; re-reserve downstream (section 20.2); unpack inter-company containers (section 20.3); send the confirmation message and the confirmation text message.
6. **Automatic printing.** Collect the print actions the Operation Types ask for (see `interfaces.md`, section "Automatic printing").
7. **Reception report.** When the user is in the reception-report group and at least one validated Transfer's Operation Type asks to show the report: collect the moves of those Transfers that are storable, not cancelled, have a processed quantity and have no destination move. When there are any, and at least one open move in the same Warehouse (excluding vendor Locations, excluding the validated Transfers themselves) demands one of those products, open the report. If there are no print actions, return the report action directly; otherwise return it as the follow-up action of the print action.
8. Return the print action when there is one, otherwise true.

## 20.1 The backorder decision

1. Skip every Transfer whose Operation Type does not ask (that is, whose backorder policy is always or never).
2. A Transfer needs a decision when at least one of its moves that is not cancelled satisfies either:
   - the move has a non-zero demand and is not picked; or
   - the move's *picked quantity* is strictly less than its demand at the Product Unit precision.
3. The picked quantity of a move is: when the move is picked but at least one of its lines is not, the sum over the picked lines only of their quantities converted into the move's line unit without rounding; otherwise the move's whole processed quantity.
4. Return the Transfers that need a decision.

## 20.2 Re-reservation after completion

The completed moves of the Transfers whose Operation Type is a receipt or an internal transfer trigger a search for open moves they could now satisfy:

1. Skip entirely when the system parameter that disables automatic reservation is set.
2. Build the product-and-location filter: for each distinct destination Location among the completed moves, the moves of those products whose source Location is that Location or one of its ancestors.
3. Add the static conditions: status confirmed or partially available; supply method take-from-stock; and either a reservation date on or before today or an Operation Type that reserves at confirmation.
4. Order the result by priority descending, then date ascending, then identifier ascending.
5. Re-sort it so that the moves sharing a document reference with the completing moves come first.
6. Run the reservation on the result.

## 20.3 Inter-company unpacking

When the container group is active and the system parameter that unpacks inter-company containers is set, for each validated Transfer that has a contact: find the company whose contact is that contact or one of its ancestors; when that company differs from the Transfer's company, unpack the destination containers of every move whose destination Location has transit usage.

---

# 21. Over-processing

Recording more than the demand is allowed and produces no extra record.

1. The move's processed quantity simply exceeds its demand.
2. The backorder test of section 15 compares processed against demand and finds processed greater, so **no backorder move** is created.
3. The backorder decision of section 20.1 finds the picked quantity greater than the demand, so **no backorder question** is asked.
4. The status recomputation of `state-machines.md`, section 1.3, finds processed greater than or equal to demand, so the move is `assigned` before completion.
5. The extra quantity reaches the destination through the detail lines exactly like the rest.
6. Where the extra quantity comes from depends on the source: for a Location that bypasses reservation the goods are simply created; for an internal Location the extra quantity drives the available quantity negative, which triggers the reservation-freeing of section 7.4 and may leave a negative quantity record behind.

**Worked example.** A receipt demands 10 units and 12 are received. The move's demand stays 10, its processed quantity is 12, it completes, no backorder is created, and 12 units arrive in stock.

---

# 22. Dates

## 22.1 Deadline propagation

When a deadline is written on a set of moves:

1. Remember the set in the calling context so that the propagation cannot loop.
2. For each move, let `delta = old deadline − new deadline`, or zero when the move had no deadline.
3. For every move linked to it upstream or downstream that is neither done nor cancelled and that has not already been visited:
   - when it has a deadline and the delta is non-zero, subtract the delta from its deadline (shifting the whole chain by the same amount);
   - otherwise, when it has no deadline or a different one, set its deadline to the new deadline.

## 22.2 Delay alert

```formula
delay_alert_date(move) = max( dates of the originating moves that are neither done nor cancelled )
                         when that maximum is strictly later than the move's own date
                       = empty otherwise
```

A Transfer's delay alert date is the maximum over its moves.

When the deadline of a move changes because of an upstream delay, a note is posted on the affected documents: subject "Deadline updated due to delay on *the upstream document name*", body "The deadline has been automatically updated due to a delay on *a link to the upstream document*." The note is skipped when the document's most recent message already carries that subject.

## 22.3 Scheduled date of a Transfer

```formula
scheduled_date = min( dates of the moves that are neither done nor cancelled )   when the shipping policy is as soon as possible
               = max( dates of the moves that are neither done nor cancelled )   when the shipping policy is all at once
```

with the previous value, or the current instant, as the fallback when there are no such moves.

## 22.4 Deadline of a Transfer

```formula
date_deadline = min( deadlines of the non-cancelled moves that have one )   when the shipping policy is as soon as possible
              = max( deadlines of the non-cancelled moves that have one )   when the shipping policy is all at once
```

```formula
has_deadline_issue = ( date_deadline exists ) and ( date_deadline < scheduled_date )
```

## 22.5 Reservation date

```formula
reservation_date = date_part(move date) − reservation_days_before                 for a normal move
                 = date_part(move date) − reservation_days_before_priority        for an urgent move
```

computed only when the Operation Type reserves before the scheduled date and the move's status is draft, waiting, waiting-another-move or partially available. When the Operation Type reserves manually, the reservation date is empty. When it reserves at confirmation, the date is stamped as today at confirmation time.

A move is eligible for automatic reservation when it bypasses reservation, or its Operation Type reserves at confirmation, or it has a reservation date that is on or before today.

## 22.6 Date categories

Let *start of today* be midnight of the current day in the reading user's time zone, expressed in coordinated universal time. Define the boundaries:

```formula
start_yesterday = start_today − 1 day
start_day_1     = start_today + 1 day
start_day_2     = start_today + 2 days
start_day_3     = start_today + 3 days
```

| Category | Condition on the date |
|---|---|
| `before` | earlier than *start yesterday* |
| `yesterday` | on or after *start yesterday* and earlier than *start today* |
| `today` | on or after *start today* and earlier than *start day 1* |
| `day_1` | on or after *start day 1* and earlier than *start day 2* |
| `day_2` | on or after *start day 2* and earlier than *start day 3* |
| `after` | on or after *start day 3* |

An empty date belongs to no category.

## 22.7 Next count date

**On a Location** (the stored next-count field):

1. When the Location has no company, or its usage is neither internal nor transit, or its counting frequency is zero, the date is empty.
2. Otherwise, when the Location has a last-count date:
   ```formula
   days_until_next = counting_frequency − ( today − last_count_date )   (in whole days)
   next_count_date = today + 1 day                       when days_until_next ≤ 0
                   = last_count_date + counting_frequency days   otherwise
   ```
3. Otherwise `next_count_date = today + counting_frequency days`.
4. When the arithmetic overflows the calendar, fail with "The selected Inventory Frequency (Days) creates a date too far into the future."

**For a quantity record in a Location** (the date written onto new records and after each application):

1. When the Location's usage is neither internal nor transit, the date is empty.
2. Compute the company's yearly date when the company names an annual inventory month:
   ```formula
   day  = min( max(annual_inventory_day, 1) , last day of the chosen month in the current year )
   candidate = current year, chosen month, day
   if candidate ≤ today:
       day  = min( max(annual_inventory_day, 1) , last day of the chosen month in the next year )
       candidate = next year, chosen month, day
   ```
   The clamping in the second branch is what handles the twenty-ninth of February in a non-leap year.
3. When the Location has its own next-count date, the answer is the smaller of that date and the company's yearly date (or that date alone when the company has no yearly date).
4. Otherwise the answer is the company's yearly date, or empty when there is none.

**Worked example.** Today is 11 September 2026. A Location has a counting frequency of 30 and was last counted on 1 September 2026; the company's annual inventory is set to 31 December.
- Location next-count date: days until next = 30 − 10 = 20, so 1 September + 30 days = **1 October 2026**.
- Company yearly date: 31 December 2026, which is after today, so it stands.
- Record next-count date = min(1 October 2026, 31 December 2026) = **1 October 2026**.

---

# 23. Inventory counting

## 23.1 Difference and outdated flag

```formula
inventory_difference = counted_quantity − on_hand_quantity     when the counted flag is set
                     = 0                                        otherwise
```

```formula
is_outdated = counted flag is set
              and compare( counted_quantity − inventory_difference , on_hand_quantity , product unit ) ≠ 0
```

Because the difference is stored, `counted − difference` is the on-hand quantity as it was when the count was entered. The record is outdated exactly when the on-hand quantity has moved since.

## 23.2 Applying the counts

1. Mark every record in the set as counted.
2. Resolve the inventory-loss Location per record: the product's own inventory-loss Location in the record's company; when the product names none, the company-level default for that field.
3. For each record:
   - skip it when the application comes from the product's own quantity field and the difference is zero;
   - when the difference is strictly positive, build a move of that difference **from** the inventory-loss Location **to** the record's Location, with the record's container as the destination container;
   - when it is not, build a move of the absolute difference **from** the record's Location **to** the inventory-loss Location, with the record's container as the source container.
   Each move is built with: the product, the product unit, the difference as demand, the company, status confirmed, the owner as owner restriction, the adjustment flag, the picked flag, and one detail line carrying the same quantity, Locations, lot, containers and owner. When the calling context supplies an adjustment reference, it is stamped on the move.
4. Create all the moves and complete them with destination containers ignored.
5. When an explicit date was supplied, write it as the date of the created moves.
6. Trigger the re-reservation search (section 20.2) from those moves.
7. Stamp today's date as the last-count date of every Location involved.
8. Recompute each record's next-count date (section 22.7).
9. Clear the counted quantity, the difference, the counted flag and the assignee on every record.

## 23.3 Conflict handling

Before applying, the records that are outdated are collected. When there is at least one, the conflict screen is opened instead, carrying the whole set and the outdated subset. Choosing to keep the counted quantities applies them unchanged; choosing to discard clears the counted quantities of the outdated records.

## 23.4 Reverting an adjustment

For each completed adjustment detail line with a non-zero quantity, build the mirror move: same product and unit, the line's quantity as demand, status confirmed, source and destination Locations swapped, the adjustment flag, the picked flag, and one detail line with the quantities and characteristics swapped in the same way (the destination container becomes the source container and vice versa). The reference of each mirror move is "*the original reference* [reverted]". Create the moves and complete them. When no line qualifies, show the notification "There are no inventory adjustments to revert."

## 23.5 Last count date

For a quantity record, the last count date is the latest date among the completed adjustment detail lines that match the record on product, on lot (or have no lot), on owner (or have none), on container (either as source or as destination container, or have none) and on Location (either as source or as destination Location). Formally, the search groups completed adjustment lines by (product, lot, source container, owner, destination container, source Location, destination Location) taking the maximum date, then indexes every combination of (Location, container, product, lot, owner) reachable from that group — the two Locations crossed with the two containers — keeping the maximum date per combination. The record reads its own combination.

---

# 24. Derived indicators

## 24.1 Transfer availability text

Only Transfers whose status is waiting-another-operation, waiting or ready and whose Operation Type is a delivery or an internal transfer are evaluated; all the others have no text and no state.

1. Start with the text "Available" and the state `available`.
2. If any move has a product and its forecast availability is strictly less than its real quantity — or, for a draft move, strictly less than zero — the text becomes "Not Available" and the state becomes `late`.
3. Otherwise take the greatest forecast expected date among the moves that have one. When there is one, the text becomes "Exp " followed by that date formatted for the reader, and the state becomes `late` when the Transfer's scheduled date is earlier than that date, `expected` otherwise.

## 24.2 Move forecast information

1. Every move of a non-storable product gets its real quantity as forecast availability.
2. For the storable ones, pre-read the forecast figures per (Warehouse, date) pair, where the date is the later of the move's date and the current instant and the Warehouse is the source Location's for a consuming or internal move and the destination Location's for an incoming move.
3. Per move:
   - a move whose status is assigned gets its processed quantity converted into the product unit, rounding half away from zero;
   - a draft move whose free quantity at that key is greater than or equal to its real quantity gets that free quantity;
   - a consuming move in draft gets the forecast quantity when that covers its real quantity, and otherwise `forecast quantity − real quantity`;
   - a consuming move in status waiting-another-move, waiting or partially available is deferred to the outgoing computation of step 4;
   - an internal move gets its free quantity when that covers its real quantity;
   - an incoming move gets the forecast quantity at the destination Warehouse, plus its own real quantity when it is still draft.
4. The deferred consuming moves are grouped by Warehouse and then by source Location and handed to the forecast report of `../replenishment-and-procurement/`, which returns per move a pair (expected quantity, expected date). For each report line that names an outgoing move: when the line is marked as filled, the expected quantity accumulates the line's quantity; otherwise it is the negated line quantity. When the line also names an incoming move, the expected date is the greater of that move's date and the date accumulated so far.

## 24.3 Detail button visibility

The button that opens the detail screen of a move is shown when:

1. the move has a product, and
2. the move is not in draft, and
3. it is not the case that the Operation Type allows neither creating nor using lots while the user is in neither the container group nor the multi-location group,

and then: always when the move has more than one detail line; otherwise when the user is in the multi-location group, the container group or the consignment group, or when the product is tracked.

## 24.4 Reception report visibility

For a Transfer (or a batch), the allocation button is shown when:

1. the user is in the reception-report group, and
2. the Operation Type exists and is not a delivery, and
3. the Transfer has at least one move with a storable product that is not cancelled, and
4. at least one other move exists with: status confirmed, partially available or waiting-another-move — plus assigned when the Transfer is already done — a strictly positive real quantity, a source Location inside the Operation Type's Warehouse view Location but not of vendor usage, a Transfer outside the set being evaluated, a product among the Transfer's products, and either no originating move at all or an originating move among the Transfer's own moves.

## 24.5 Operation overview graph

1. Read the scheduled dates of the Operation Type's Transfers whose status is ready, waiting-another-operation or waiting.
2. Bucket each date by the categories of section 22.6 and count.
3. Emit six buckets in this order with these labels and kinds: "Before" (past), "Yesterday" (past), "Today" (present), "Tomorrow" (future), "The day after tomorrow" (future), "After" (future). The series is named "Transfers".
4. When all six counts are zero, the series is renamed "Sample data", every bucket's kind becomes "sample" and the series carries no Operation Type identifier, so that clicking it does nothing.

## 24.6 Lot on-hand quantity

1. Take the Location filter from the product-quantity computation of `../replenishment-and-procurement/` (it honours the Location, Warehouse, company and date context).
2. Add the lot condition, and the owner and container conditions when those are in the context.
3. Sum the on-hand quantities of the matching records per lot.
4. When the context asks for a date in the past, correct the figure with the movements that happened after that date:
   ```formula
   quantity_at_date(lot) = current_quantity(lot)
                         − Σ over completed incoming detail lines of that lot dated after the cut-off ( quantity in product unit )
                         + Σ over completed outgoing detail lines of that lot dated after the cut-off ( quantity in product unit )
   ```

## 24.7 Quantity from a picked quantity record

When a person picks a quantity record in the detail screen and the line has no quantity yet:

```formula
move_demand_in_line_unit   = convert(move demand, move line unit, line unit, half-up)
move_done_in_line_unit     = convert(move processed quantity, move line unit, line unit, half-up)
record_available_in_line_unit = convert(record available quantity, product unit, line unit, half-up)

line_quantity = max( 0 , min( record_available_in_line_unit , move_demand_in_line_unit − move_done_in_line_unit ) )
                when move_demand_in_line_unit > move_done_in_line_unit
              = max( 0 , record_available_in_line_unit )
                otherwise
```

## 24.8 Assigning lots on a move

Writing the lot list of a move rebuilds its detail lines.

1. Skip untracked products. Skip a move that is already assigned, all of whose lines carry a lot from the list, and whose set of line lots is exactly the list.
2. The working unit is the product unit for a serial-tracked product and the move's line unit otherwise.
3. Compute the free quantity:
   ```formula
   free = convert( max(processed quantity, demand) , line unit, product unit, half-up )
   ```
4. Walk the existing lines: skip zero-quantity lines; a line with neither a Lot nor a typed name is remembered as *available*; a line whose lot name is in the list keeps that lot, is remembered as assigned, and its quantity in the product unit is subtracted from *free*; any other line is deleted.
5. Compute the surplus:
   ```formula
   extra = free − ( number of requested lots not yet assigned )
   ```
   This reserves at least one product unit per lot, because every lot needs its own line.
6. When the move bypasses reservation, for each unassigned lot: reuse an available line when there is one, writing the lot, its name, the working unit and a quantity of one for a serial-tracked product or the line's own quantity otherwise, and reduce *extra* by that quantity in the product unit minus one; otherwise create a line of one unit, enlarged by the whole surplus when the product is lot-tracked and the surplus is positive (the surplus is then zeroed).
7. When the move does not bypass reservation, gather the quantity records at the source Location grouped by lot. For each unassigned lot walk its records: skip records with no lot or no available quantity; take
   ```formula
   take = min( record available quantity , max( extra + (0 if something was already reserved for this lot else 1) , 1 ) )
   ```
   and create a line for it (forced to exactly one unit in the product unit for a serial-tracked product); reduce *extra* by the taken quantity, or by the taken quantity minus one when this was the first line for the lot. Stop for that lot once something was reserved and the surplus is exhausted. When nothing at all could be reserved for a lot, create a line of one unit anyway.
8. When the move does not bypass reservation and available lines remain, delete them all and re-create them in order while the surplus lasts, each for the smaller of its own quantity and the remaining surplus. Re-creating them moves them to the end of the line list, which makes the reduction algorithm of section 8.3 give them back before the lot-bearing lines.
9. Force a recomputation of the move's processed quantity.

## 24.9 Printed-document aggregation

Detail lines are grouped for printing by the key

```formula
line_key = product identifier ‖ "_" ‖ product display name ‖ "_" ‖ description ‖ "_" ‖ line unit identifier ‖ "_" ‖ packaging unit identifier
```

with the container identifier appended when the line has a destination container. The description is the move's transfer description with the product display name, or failing that the product name, stripped from its front.

For each group:

```formula
group_quantity            = Σ convert(line quantity, line unit, move line unit)
group_packaging_quantity  = Σ convert(group_quantity, move line unit, packaging unit)
```

and, unless the strict mode is requested, an ordered quantity:

1. Start from the move's demand.
2. Walk the chain of backorders of the Transfer (the backorders, their backorders, and so on) and add the demands of the moves whose own key is a prefix of this key.
3. Subtract the quantities of the other detail lines of the same move whose key is a prefix of this key.
4. Round at the move's line unit.

Finally, for every move of the Transfer and of its backorders that has a demand but a zero processed quantity: when the move is cancelled, or when it is confirmed with no detail lines at all, a line with no processed quantity is added for it (or its demand is added to an existing group whose key it prefixes).

---

# 25. Reception report

## 25.1 Building the report

Preconditions: the documents are Transfers that are not deliveries and not cancelled. When the set mixes done and not-done documents, the report refuses: "This report cannot be used for done and not done transfers at the same time". When the set is empty: "No transfers selected or a delivery order selected".

1. Take the moves of the documents whose product is storable and which are not cancelled. Call them the *incoming* moves.
2. Compute, per product, the total quantity already claimed by the destination moves of those incoming moves.
3. Walk the incoming moves. For each one:
   ```formula
   move_quantity = real quantity, or, when that is zero, convert(processed quantity, line unit, product unit, half-up)
   ```
   When the move has destination moves, the part already assigned is the smaller of the product's remaining claimed total and the move quantity; that amount is deducted from the claimed total and recorded as *already assigned* for the product, together with the move. The rest of the move quantity is recorded as *draft expected* for the product when the move is still draft, and as *assignable* (a list of (quantity, move) pairs) otherwise.
4. Find the demands to offer. Search the moves with: status confirmed, partially available or waiting-another-move — plus assigned when the documents are already done — a strictly positive real quantity, a source Location inside the Warehouse view Location but not of vendor usage, no originating move, a product among those collected in step 3, and a Transfer outside the document set. Order them by reservation date, then priority descending, then date, then identifier.
5. Group those demands by product and walk them. For each demand:
   1. Determine its source document: the originating document of the move; when the move has a Transfer and that Transfer is not the source document, the pair (Transfer, source document); otherwise the source document alone. Skip the demand when it has none.
   2. Compute the quantity still to cover: the demand's real quantity, reduced by its already-processed quantity (converted into the product unit) when the documents are not done and the demand is partially available.
   3. Consume the product's assignable list from the front, accumulating until the quantity to cover is reached; partially consumed entries are written back with their remainder.
   4. When a non-zero quantity was accumulated, emit a report line for it under the demand's source, naming the incoming moves consumed.
   5. When the demand is still not covered and the product has a draft expected quantity, emit a second, non-assignable line for the smaller of that expected quantity and the shortfall, and reduce the expected quantity.
6. For each product with an already-assigned total, walk the destination moves of the recorded incoming moves, skipping once the total is exhausted, and emit for each one a line marked as assigned for the smaller of the remaining total and that move's real quantity.
7. Group the emitted lines by source document for printing, and format each source's scheduled date.

## 25.2 Assigning

Inputs: the demands to link, the quantity to link to each, and the incoming moves offered for each.

1. **Split first.** For every demand whose real quantity is strictly greater than the quantity being linked, split off the excess (section 14) and remember the pairing. Create all the split moves at once, write them directly to status confirmed (not confirmed through the normal path, so that no unintended reservation is created) and copy each parent's reservation date onto them.
2. For each demand:
   1. When the demand was split:
      - if the first offered incoming move is not done and the demand already had a processed quantity, move **all** of the demand's detail lines onto the split move, so that only the unreserved part is being assigned;
      - if the first offered incoming move is done and the demand's processed quantity exceeds the quantity being linked, move all the detail lines onto the split move, then pull lines back onto the original demand, preferring the lines whose source Location is one of the incoming moves' destination Locations: accumulate their quantities until the linked quantity is reached, splitting the last line in two when it would overshoot.
   2. Walk the offered incoming moves from the last to the first. Skip a move whose product differs, or whose remaining quantity — its move quantity minus the sum of the real quantities of its destination moves — is not strictly positive. Otherwise link it as an originating move of the demand, share the document references both ways, switch the demand's supply method to advanced, and reduce the quantity still to link by the smaller of the move quantity and that quantity. Stop when nothing is left to link.
3. Recompute the statuses of the demands and of the split moves.
4. Run the reservation on the demands, so that a done incoming move immediately reserves its goods instead of letting another document take them.

## 25.3 Unassigning

1. For each offered incoming move that really is an originating move of the demand: unlink it, un-share the references, and accumulate the smaller of the requested quantity and that move's move quantity. Stop once the requested quantity is reached.
2. When the demand still has originating moves and is not done, split off the quantity that is still linked (the sum of the real quantities of the remaining originating moves): create the split move with the advanced supply method, the parent's reservation date and status confirmed; move all the demand's detail lines onto it; recompute both processed quantities; when the split move is now over-reserved, hand the excess back to the original demand line by line, splitting the last line when needed; clear the original demand's originating links and recompute the split move's status.
3. Set the demand's supply method back to take-from-stock and unreserve it.

---

# 26. Traceability tree

The traceability report is a tree of completed Stock Move Lines, unfolded one level at a time.

## 26.1 The root set

The root set depends on what the report was opened from:

| Opened from | Root set |
|---|---|
| A Lot | every completed detail line of that lot |
| One detail line, with a lot name in the calling context | the lines recorded as *produced from* that line by the production genealogy, when there are any |
| A Transfer | the completed detail lines of its moves that carry a lot |
| A manufacturing order | the completed detail lines of its finished moves |

## 26.2 Walking upstream from a line

Given a starting line, the walk collects the lines that brought the same lot into the Location the starting line took it from. It is a breadth-first walk with a "seen" set that prevents cycles.

1. Seed the "seen" set and the queue with the starting lines.
2. Pop a line from the queue.
3. **Chained case.** When the line's move has originating moves, the candidates are the completed detail lines of those originating moves whose lot equals this line's lot, minus the already-seen lines.
4. **Unchained case.** Otherwise, when the line's source Location's usage is internal or transit, the candidates are the completed detail lines with the same product, the same lot, a destination Location exactly equal to this line's source Location, a date on or before this line's date, and an identifier not already seen.
5. **Otherwise** (the goods came from outside), there are no candidates: stop for this line.
6. When the walk is unfolding one specific line, only descend into the candidates when that line is among them.
7. Add the candidates to the "seen" set and continue.
8. The result is the "seen" set minus the starting lines.

Note the asymmetry between steps 3 and 4: a chained move is followed by its explicit links, while an unchained one is matched heuristically by product, lot, Location and date. Both are restricted to completed lines.

## 26.3 What each node shows

| Column | Value |
|---|---|
| Reference | The Transfer's reference when the line has one; otherwise the literal text "Inventory Adjustment" when the move is an adjustment move; otherwise the Scrap's reference when the move's destination usage is inventory loss and it belongs to a Scrap; otherwise empty. The reference also carries the model and identifier it links to, so the node can be opened. |
| Product | The product's display name. |
| Date | The **move's** date, formatted for the reader without a time zone shift. |
| Lot/Serial Number | The lot's name. |
| From | For a receipt, the Transfer's contact name; otherwise the source Location's display name. |
| To | For a delivery, the Transfer's contact name; otherwise the destination Location's display name. |
| Quantity | The line's quantity converted into the **product** unit, rounding half away from zero, rendered at the `Product Unit` precision, followed by the product unit's name. |

Each node also carries a *usage* marker used for colouring: `internal` when both Locations have internal usage, `in` when only the destination has, and `out` in every other case.

## 26.4 Unfoldability

A node can be unfolded when it has consumed lines recorded by the production genealogy, **or** when the report was not opened from a Lot, the line carries a lot, and the upstream walk of section 26.2 finds at least one line.

## 26.5 Ordering and printing

The nodes of one level are sorted by date **descending**. Printing renders exactly the nodes the person has currently unfolded, in landscape, with the reference of the record the report was opened from as the document title and that record's company when it has one.

## 26.6 Delivery discovery for a lot


Which outgoing Transfers finally carried a lot — possibly after the lot was consumed to produce another lot.

1. Seed a queue with the lots being asked about.
2. While the queue is not empty, search the completed detail lines of those lots that are outgoing — meaning the line's Operation Type kind is delivery, or its move's Operation Type kind is delivery, or the line has produced lines. For each such line:
   - when the line has produced lines, record each produced lot as a child of this lot and enqueue the children not yet seen;
   - otherwise record the line as a *leaf* line of this lot.
3. Initialise each lot's Transfer set with the Transfers of its leaf lines; mark those lots for propagation.
4. Repeatedly take a marked lot and push its Transfer set up to each of its parents; whenever a parent gains a Transfer it did not have, mark the parent for propagation too.
5. The answer for a lot is its accumulated Transfer set.

The contacts of a lot are the contacts of those Transfers, ordered by completion date descending.

---

# 27. Serial-number generation

## 27.1 Generating a series from a first name

1. Find every run of digits in the first name. When there is none, append the digit `0` to the name and start again.
2. Take the **last** run of digits. Its length is the padding; its value is the starting number.
3. Split the name on that exact digit run. The prefix is every part except the last, re-joined with the digit run itself (so that a name such as `BAV023B00001S00001` keeps `BAV023B00001S` as prefix); the suffix is the last part.
4. Produce the requested count of names:
   ```formula
   name(i) = prefix ‖ zero_pad( starting_number + i , padding ) ‖ suffix       for i = 0 … count − 1
   ```

**Worked example.** First name `SN00007`, count 3 → `SN00007`, `SN00008`, `SN00009`. First name `LOT-A`, count 2 → the name becomes `LOT-A0`, giving `LOT-A0`, `LOT-A1`.

## 27.2 The next serial number for a product

Take the most recently created Lot of that product belonging to the company or to no company; generate two names from its name and return the second. When there is none, return nothing.

## 27.3 Generating serial numbers on a move

1. The count is the one supplied, else the move's own serial-number count. A count of zero fails with "The number of Serial Numbers to generate must be greater than zero."
2. Generate the names (section 27.1) and build one entry per name with a quantity of one.
3. When the Operation Type allows using existing lots, resolve or create the Lot records for those names now and replace the typed names by the Lot links.
4. Turn the entries into detail-line operations (section 27.5) and write them onto the move.

## 27.4 Quantity per lot when generating lots

When generating **lots** (not serial numbers) with a requested number of lots *count* and a total quantity *quantity*:

```formula
whole_lines = integer_part( quantity ÷ count )
leftover    = quantity − whole_lines × count
```

produces `whole_lines` entries of `count` units each, plus one entry of `leftover` units when the leftover is non-zero. Note that the requested number is used as the *quantity per lot*, not as the number of lots.

## 27.5 Turning entries into detail-line operations

1. Take the move's detail lines that have neither a Lot nor a typed name; they will be reused in order.
2. For each entry:
   - when a reusable line is left, update it with the entry's values and add the entry's quantity to the running per-Location tally of the line's current destination Location;
   - otherwise create a new line with the move's Transfer, source Location, product and product unit, the entry's values, and a destination Location resolved by put-away for the entry's quantity with the running tally passed as the additional-quantity map; then add the quantity to that Location's tally.
3. When the caller supplied an origin line, the new lines also copy its owner and source container, and its destination Location is used instead of running put-away.

## 27.6 Parsing a pasted list of lot names

1. Split the pasted text on line breaks and drop the empty lines.
2. For each line, start an entry with the whole line as the typed name and a quantity of one.
3. Replace semicolons by tabulation characters and split the line on tabulation characters.
4. For each part after the first, try to convert it into field data: a part that is a plain number (with either a dot or a comma as the decimal separator) becomes the entry's quantity. When a part converts, the typed name is reduced to the first part. When a part does not convert, the whole original line is kept as the typed name and the remaining parts are ignored.

## 27.7 Advancing the product's lot sequence

When names were generated from a first name and the product has a lot numbering sequence:

```formula
first_number = current sequence next number − increment
final_number = first_number + count                 when the first name equals the sequence rendering of first_number
             = first_number + increment + count     when it equals the sequence rendering of first_number + increment
             = first_number                         otherwise
```

When the final number differs from the first number, the sequence's next number is set to the final number. The two cases exist because the "new" button of the generation screen may already have consumed one number.

---

# 28. Company consistency

Every entity of this domain that names both a company and links to company-scoped records is checked: a link to a record whose company is set and differs from the record's own company is refused. The Locations, Operation Types, Lots, containers and contacts referenced by a move, a detail line or a Transfer must therefore all belong to the same company or to no company.

Additional explicit checks:

- A Stock Rule's company must equal its Route's company when the Route has one: "Rule *the rule name* belongs to *the rule's company name* while the route belongs to *the route's company name*."
- A Route's every rule must satisfy the same equality, checked from the Route side with the same message.
- A Lot may not change company while it sits in a Location of another company.
- A Location, an Operation Type, a Warehouse and a Put-away Rule may never change company at all.

---

# 29. The daily quantity series

The read-only daily series (`report.stock.quantity`) answers "how much of this product will this warehouse hold on each day of a window around today?" without storing anything. It is regenerated from the Stock Moves and the Stock Quantity records on every read.

Let *P* be the horizon in months, taken from the system parameter `stock.report_stock_quantity_period` with a default of **3**. The window runs from *today − P months* to *today + P months*, one row per calendar day.

## 29.1 Resolving a warehouse from a location

Every Location is matched to a Warehouse by testing its materialised path against each Warehouse's view Location: the Location belongs to that Warehouse when its path contains the view Location's identifier as a complete segment, or starts with it. A Location that matches no Warehouse resolves to *no warehouse*.

## 29.2 Selecting the relevant moves

Take the Stock Moves that satisfy all of:

1. the product is storable;
2. the source Warehouse and the destination Warehouse **differ** (either may be absent);
3. the demand in the product unit is non-zero, or the processed quantity is non-zero;
4. the status is neither draft nor cancelled;
5. either the status is not done, or the date is on or after *today − P months*.

The destination Warehouse is resolved from the **final** Location when the move is not done and one is set, and from the intermediate destination Location otherwise; once the move is done it is always resolved from the intermediate destination Location. This is what makes a chain that has not run yet already point at the warehouse the goods are really going to.

For each selected move, also compute the processed quantity expressed in the product unit:

```formula
processed_in_product_unit = processed_quantity × line_unit_factor ÷ product_unit_factor
```

## 29.3 Duplicating for inter-warehouse moves

A move between two Warehouses must appear twice: as an outgoing movement of the source Warehouse and as an incoming movement of the destination Warehouse. Each selected move is therefore expanded into two rows, an original and a duplicate:

| Value | Original row | Duplicate row |
|---|---|---|
| demand in product unit | the move's | the move's when the two Warehouses differ, else 0 |
| processed quantity | the move's | the move's when both Warehouses exist and differ, else 0 |
| processed quantity in the product unit | the move's | the move's when both Warehouses exist and differ, else 0 |
| source Warehouse | the move's | none |
| destination Warehouse | the move's, but only when there is no source Warehouse, or no destination Warehouse, or the two are equal | the move's, but only when both exist and they differ |

The effect is that a purely incoming move keeps one meaningful row, a purely outgoing move keeps one, and an inter-warehouse move yields one row with only a source Warehouse and one row with only a destination Warehouse.

## 29.4 The three kinds of row

**Forecasted receipts and deliveries.** From the expanded rows whose demand is non-zero and whose status is not done, emit one row per move:

```formula
state    = "out"  when a source Warehouse is set and no destination Warehouse is
         = "in"   when a destination Warehouse is set and no source Warehouse is
date     = the move's date, truncated to the day
quantity = − demand_in_product_unit   for "out"
         = + demand_in_product_unit   for "in"
warehouse= the source Warehouse for "out", the destination Warehouse for "in"
```

**Forecasted stock, from the quantity records.** For every day of the whole window and every Stock Quantity record whose Location is internal and belongs to a Warehouse, or is transit, emit a row with state `forecast`, that day, the record's on-hand quantity, its company and its Warehouse. In other words the current stock is repeated on every day of the window and then corrected by the movement rows below.

**Forecasted stock, from the movements.** For every expanded row whose demand is non-zero, or which is done with a non-zero processed quantity, emit one `forecast` row per day of a range and with a signed quantity:

| Move status | Day range | Signed quantity |
|---|---|---|
| done | from *today − P months* to *the move's day − 1 day* | `+ processed_in_product_unit` for an outgoing row, `− processed_in_product_unit` for an incoming row |
| not done | from the later of the move's day and *today − P months*, to *today + P months* | `− demand_in_product_unit` for an outgoing row, `+ demand_in_product_unit` for an incoming row |

The sign is deliberately inverted for done moves: the current stock already contains their effect, so the days **before** the move must be corrected backwards.

## 29.5 Aggregation

Group everything by (product, product template, state, date, company, warehouse) and sum the quantities. The row identifier is the smallest underlying identifier of the group; rows built from quantity records use the negated record identifier so that they cannot collide with rows built from moves.

**Worked example.** Today is 11 September 2026, the horizon is 1 month, and warehouse `WH` holds 30 units of Bolt. One delivery move of 8 is planned for 20 September and one done receipt of 5 was completed on 5 September.

- From the quantity record: every day from 11 August to 11 October carries `forecast` +30.
- From the planned delivery: `out` −8 on 20 September; and `forecast` −8 on every day from 20 September to 11 October.
- From the done receipt: `forecast` −5 on every day from 11 August to 4 September (the day before the move), because the +5 is already inside the 30.
- The resulting forecast reads 25 up to 4 September, 30 from 5 to 19 September, and 22 from 20 September onwards.

---

# 30. Rules diagram

The routes report draws, for one product and a set of Warehouses, the rules that can move it.

1. For each chosen Warehouse, collect the Routes that apply: the Routes selected on the product, the Routes selected on the product's category chain, the Routes selected on that Warehouse, and the Warehouse's own generated Routes.
2. From those Routes, collect the rules whose Warehouse is the chosen one or none.
3. Lay them out as a graph whose nodes are Locations and whose edges are rules, each edge labelled with the rule's Operation Type, its action, its supply method and its lead time.
4. Mark the rules the product would actually take, by running the rule-matching selection for that product at each destination Location in turn.

---

# 31. Detail-line date stamping

The date carried by a detail line is not the move's date; it is the instant the line last *became more real*.

| Event | Effect on the line's date |
|---|---|
| The line is created | The current instant (the field's default). |
| The line becomes picked while it was not | The current instant. |
| The line is already picked and its quantity in the product unit increases | The current instant. |
| The line is already picked and its quantity decreases | Unchanged. |
| The line is completed | The current instant, written for every surviving line at the end of the completion. |
| The move's date is written while the move is done | The move's new date is copied onto the lines. |
| The line's status is draft, cancelled or done | No automatic re-stamping. |
| A date is written explicitly in the same operation | The explicit value wins; no automatic re-stamping. |

---

# 32. Ordering rules that matter

Several algorithms depend on an ordering. They are collected here because getting one of them wrong produces different records, not merely a different display.

| Where | Ordering | Why it matters |
|---|---|---|
| Gathering, first in first out | incoming date ascending, then identifier ascending | Decides which goods leave. |
| Gathering, last in first out | incoming date descending, then identifier descending | Same. |
| Gathering, closest location | full location name ascending, then identifier descending | Same. The name comparison is textual, so `Shelf 10` sorts before `Shelf 2`. |
| Gathering, final pass | records with a lot before records without | Guarantees that tracked stock is consumed before untracked pockets. |
| Reducing a processed quantity | detail lines in **reverse** identifier order | Gives back the goods that were reserved last, preserving the strategy's choice for the rest. |
| Freeing other reservations | the current Transfer first, then scheduled date descending, then identifier descending | Robs the document at hand first, then the latest-promised ones. |
| Availability check on a Transfer | priority descending, having a deadline before not having one, deadline ascending, date ascending, identifier ascending | Decides who gets scarce goods. |
| The daily reservation job | reservation date, priority descending, date ascending, identifier ascending | Same, across documents. |
| Re-reservation after a completion | priority descending, date ascending, identifier ascending, then the moves sharing a document reference with the completing moves first | Lets the document that caused the arrival claim it. |
| Reception report demands | reservation date, priority descending, date, identifier | Decides which demand is offered the arrival first. |
| Reception report, linking incoming moves | the offered moves walked from **last** to first | Consumes the most recently offered arrivals first. |
| Put-away rule walk | container type, then product, then own category, then any category — all descending as booleans; ties keep priority ascending then product | Decides which shelf. |
| Move merging | the candidate sets are the whole move list of each Transfer; inside a set, groups keep their natural sequence-then-identifier order and the **first** move of a group survives | Decides which record keeps the identifier. |
| Detail lines at completion | destination container descending, then identifier ascending | Ensures that packed lines are completed before loose ones, so that the container snapshots are consistent. |
| Transfers | priority descending, scheduled date ascending, identifier descending | Display only. |
| Operation Types | favourite first, then sequence, then identifier | Display only. |

---

# 33. The location scope of the product quantity figures

The five quantity figures shown on a product — on hand, free, incoming, outgoing and forecasted — are owned by `../replenishment-and-procurement/`, but the way the **set of Locations** they look at is resolved belongs here, because it is the same resolution that the "on hand" filter on Stock Quantity records uses and the same that the lot on-hand quantity uses.

## 33.1 Resolving the location set from the reading context

1. Read a Location value from the calling context under either of two names; when it is not a list, wrap it in one.
2. Read a Warehouse value the same way.
3. Each value may be an identifier or a piece of text. Text is resolved by searching the corresponding entity on each of its searchable name fields with a case-insensitive containment test, and taking the union of the matches.
4. Then:
   - **Warehouses given, Locations also given** — take the view Locations of the Warehouses; keep only the given Locations whose materialised path starts with one of those view Locations' paths.
   - **Warehouses given, no Locations** — take the view Locations of the Warehouses.
   - **No Warehouses, Locations given** — take the given Locations.
   - **Neither given** — take the view Locations of every Warehouse of the reader's enabled companies.
5. When the resulting set is empty, all three filters below are the always-false filter, so every figure is zero.

## 33.2 Building the three filters

Let *D* be the set of the chosen Locations **and all of their descendants**, computed recursively.

**Strict mode** (asked for by the calling context): *D* is just the chosen Locations, with no descendants.

Three filters are produced:

| Filter | Applies to | Definition |
|---|---|---|
| Quantity filter | Stock Quantity records | the record's Location is in *D* |
| Incoming filter | Stock Moves | the move *arrives* in *D* **and** does not *leave* from *D* |
| Outgoing filter | Stock Moves | the move *leaves* from *D* **and** does not *arrive* in *D* |

"Leaves from *D*" means the move's source Location is in *D*. "Arrives in *D*" is deliberately different for done and open moves:

- a **done** move arrives in *D* when its intermediate destination Location is in *D*;
- an **open** move arrives in *D* when its final Location is set and is in *D*, or its final Location is empty and its intermediate destination Location is in *D*.

This is what makes a two-step receipt count as *incoming* for the warehouse as a whole from the moment the first step is planned, instead of only when the last step is created.

A third mode, asked for by the calling context when the caller only wants what already physically happened, drops the open-move treatment entirely: the incoming filter becomes "the done destination is in *D* and the source is not", and the outgoing filter becomes "the source is in *D* and the done destination is not".

## 33.3 The figures

With the three filters, and with optional restrictions on lot, owner and container taken from the calling context:

```formula
qty_available     = Σ on_hand over the matching quantity records
reserved          = Σ reserved over the matching quantity records
incoming_qty      = Σ real_quantity over the open moves matching the incoming filter
outgoing_qty      = Σ real_quantity over the open moves matching the outgoing filter
free_qty          = round( qty_available − reserved − expired_unreserved )
virtual_available = round( qty_available + incoming_qty − outgoing_qty − expired_unreserved )
```

where *open* means the status is waiting-another-move, waiting, assigned or partially available, every rounding is at the **product** unit's rounding step, and *expired unreserved* is zero unless the expiration context is active, in which case it is the available quantity of the records whose removal date is on or before the cut-off.

## 33.4 Evaluating at a past instant

When the calling context carries an as-of instant that is earlier than now:

1. A date value with no time part is pushed to the very end of that day.
2. The on-hand figure is corrected by replaying the completed moves that happened **after** the instant:
   ```formula
   qty_available_at(instant) = current_on_hand
                             − Σ over completed moves matching the incoming filter, dated after the instant ( processed quantity in the product unit )
                             + Σ over completed moves matching the outgoing filter, dated after the instant ( processed quantity in the product unit )
   ```
3. The incoming and outgoing figures keep their own date restrictions (from and to) applied to the move date.

## 33.5 Setting the on-hand quantity directly on a product

Writing the on-hand figure of a storable product creates, in counting mode, one Stock Quantity record for that product in the stock Location of the company's first Warehouse with the written value as its counted quantity, and applies it at once. The write is ignored when the recomputation itself triggered it, and when the value is negative. Writing it before the product is saved fails with "Save the product form before updating the Quantity On Hand."

## 33.6 Movement counters on a product

```formula
nbr_moves_in  = number of completed detail lines of the product whose Operation Type kind is receipt and whose date is within the last year
nbr_moves_out = number of completed detail lines of the product whose Operation Type kind is delivery and whose date is within the last year
```

## 33.7 Transfer descriptions

A move's printed description is resolved in this order:

1. the manual override written on the move, when there is one;
2. otherwise the product's own per-kind transfer description — the receipt description for a receipt, the delivery description for a delivery, the internal description for an internal transfer, and nothing for any other kind — when it is not empty;
3. otherwise the product's generic description: for a **delivery** always the product's display name; for any other kind the product's description rendered as plain text when it is not empty, and the display name otherwise.

The reading is done in the language of the Transfer's contact, else the move's contact, else the reader.

---

# 34. Three fully worked traces

These traces follow one operation from the first read to the last write, naming every intermediate figure. They exist so that an implementation can be checked step by step rather than only by its end state.

## 34.1 A reservation that partially succeeds

**Starting state.** Product Bolt, product unit "Units" with a rounding step of 0.01. `WH/Stock` has two internal children. The `Product Unit` precision is 2. No removal strategy is set anywhere, so first in first out applies.

| Quantity record | Location | Lot | Container | Owner | On hand | Reserved | Incoming date |
|---|---|---|---|---|---|---|---|
| Q1 | `WH/Stock/Shelf A` | — | — | — | 6.00 | 2.00 | 2 March |
| Q2 | `WH/Stock/Shelf B` | — | `PACK0001` | — | 9.00 | 0.00 | 5 March |
| Q3 | `WH/Stock/Shelf B` | — | — | — | −1.00 | 0.00 | 5 March |

**The move.** Demand 12 Units, source `WH/Stock`, destination `Customers`, status `confirmed`, no detail line, not picked, supply method take-from-stock, no originating move.

**Step 1 — the snapshot.** The move's processed quantity is 0, so the *already reserved* figure is 0.

**Step 2 — the selected set.** The move is not picked and its status is `confirmed`, so it is selected.

**Step 3 — the missing quantity.** `missing_line_units = 12 − 0 = 12`. Compared against zero at the product unit's rounding it is positive, so the move is processed. Converted to the product unit: `missing = 12`.

**Step 4 — the branch.** The source Location's usage is internal, so reservation is not bypassed. The move has no originating move. Branch B applies.

**Step 5 — the reservation-quantity computation.**

- *Gathering.* Strategy `fifo`; loose matching at `WH/Stock` with no lot, container or owner requested; the filter is therefore "product is BOLT and Location is a descendant of `WH/Stock`". Ordering: incoming date ascending, then identifier ascending. Result: Q1 (2 March), then Q2 and Q3 (both 5 March, ordered by identifier: Q2 then Q3). The final "lots first" sort changes nothing, no record having a lot.
- *Available quantity.* The product is untracked, so `available = (6 + 9 − 1) − (2 + 0 + 0) = 12.00`. Negative results are not allowed but the figure is positive.
- *Full packaging.* Not requested.
- *Clamp.* `wanted = min(12, 12) = 12`.
- *Unit re-expression.* The line unit equals the product unit, so nothing changes.
- *Serial check.* The product is untracked.
- *Direction.* `wanted` is positive. Recompute `available` as the sum of **positive** on-hand quantities minus **all** reservations: `(6 + 9) − (2 + 0 + 0) = 13.00`.
- *Negative pockets.* Q3's available quantity is −1.00, strictly negative, so the pocket for the key (`WH/Stock/Shelf B`, no lot, no container, no owner) is −1.00. Q1's and Q2's keys have no pocket, Q2 carrying a container and therefore a different key from Q3.
- *Walk.*
  - Q1: takeable = 6 − 2 = 4.00. No pocket for its key. `min(4, 12) = 4`. Append (Q1, 4.00). Remaining wanted 8.00, available 9.00.
  - Q2: takeable = 9 − 0 = 9.00. Its key carries **no** pocket, because the pocket belongs to the container-less key. `min(9, 8) = 8`. Append (Q2, 8.00). Remaining wanted 0.00 → stop.
- *Result.* [(Q1, 4.00), (Q2, 8.00)], total taken 12.00.

**Step 6 — creating the lines.** No existing line, so no candidate. The two pairs have different keys, so no merging. Two lines are created:

| Line | Source Location | Lot | Source container | Owner | Quantity |
|---|---|---|---|---|---|
| L1 | `WH/Stock/Shelf A` | — | — | — | 4.00 |
| L2 | `WH/Stock/Shelf B` | — | `PACK0001` | — | 8.00 |

Creating them raises the reserved counters: Q1 goes to 6.00 on hand / 6.00 reserved; Q2 goes to 9.00 on hand / 8.00 reserved.

**Step 7 — the status.** `missing` was 12 and `taken` is 12, equal at the product unit's rounding, so the move is marked fully assigned.

**Step 8 — put-away.** The lines are grouped by outermost destination container. L1 has none; L2 has none either, its `PACK0001` being a **source** container. Both are resolved line by line against `Customers`, which has no put-away rule, so both keep `Customers`.

**Step 9 — whole containers.** The Transfer re-runs the whole-container detection. Grouping by source container gives one group for `PACK0001` holding L2 alone. The container's contents are 9.00 and the line claims 8.00, so the whole-container test fails and nothing is flagged.

**End state.** The move is `assigned` with a processed quantity of 12.00; Q3 is untouched at −1.00; the available quantity of Bolt at `WH/Stock` is now `(6 + 9 − 1) − (6 + 8) = 0.00`.

## 34.2 A validation that creates a backorder

**Starting state.** The Transfer of the previous trace, plus a second move of 5 Paint that could not be reserved at all. The Operation Type's backorder policy is "ask" and it reserves at confirmation. The Transfer's shipping policy is as soon as possible, so its status is `assigned`.

**The person's input.** They set the Bolt move's processed quantity to 12 (it already is) and tick it as picked; they leave the Paint move untouched.

**Step 1 — immediate transfer.** The Transfer is not in draft, so nothing is copied.

**Step 2 — sanity check.**
- *No quantity at all?* At least one move that is neither done nor cancelled is picked, so *has pick* is true; the test therefore looks only at the picked moves; the Bolt move has 12.00, which is not zero; the Transfer passes.
- *No moves?* It has two.
- *Missing lots?* Paint is lot-tracked and the Operation Type uses existing lots, but the Paint line — there is none — contributes nothing, and the Bolt move is untracked. The Transfer passes.

**Step 3 — picked marking.** At least one move is picked, so the marking does **not** run; the Paint move stays unpicked.

**Step 4 — the backorder decision.** The policy is "ask". For the Bolt move: it is picked and its picked quantity is 12, which is not below its demand of 12 — no trigger. For the Paint move: its demand is 5 and it is **not** picked — trigger. The Transfer needs a decision, so the backorder screen opens.

**Step 5 — the person chooses "Create backorder".** The validation resumes with the backorder step suppressed and nothing declared as not-to-be-backordered.

**Step 6 — completion.**
- *Prune.* The Bolt move is picked and both its lines are picked, so nothing is deleted. The Paint move has a processed quantity of 0 and is not picked, so the first half of the condition is true; but backorders are allowed and its demand is 5, not zero, so it is **not** cancelled.
- *Select.* The set to complete is the Bolt move alone: the Paint move is excluded because it is not picked.
- *Backorder moves.* The Bolt move's processed quantity (12.00) is not below its demand (12) at the `Product Unit` precision, so **no** backorder move is split off it.
- *Move the goods.* L1 and L2 are completed in order (destination container descending, then identifier: neither has a destination container, so by identifier). For L1: release 4.00 of reservation on Q1; decrease Q1 by 4.00, leaving 2.00 on hand and 2.00 reserved, returning an available quantity of 0.00 and an incoming date of 2 March; increase `Customers` by 4.00 stamping 2 March. For L2: release 8.00 on Q2; decrease Q2 by 8.00, leaving 1.00 on hand and 0.00 reserved; increase `Customers` by 8.00 stamping 5 March. Neither returned a negative available quantity, so no reservation is freed.
- *Status.* The Bolt move becomes `done` with the current instant as its date.
- *Push.* `Customers` has no push rule.
- *Propagate.* The Bolt move has no destination move.
- *Transfer backorder.* The moves that are neither done nor cancelled are the Paint move alone. A backorder Transfer is created by copying the original with an empty reference, no moves, no lines, the original as back-order link. The Paint move is moved into it with its picked flag cleared. The backorder's responsible is cleared. A note is posted on the original.
- *Reserve the backorder.* The Operation Type reserves at confirmation, so the backorder's availability is checked; Paint is still unavailable, so its move stays `confirmed` and the backorder's status is `confirmed`.

**Step 7 — after completion.** The completion instant is stamped on the original Transfer and its priority is reset. The Operation Type kind is delivery, so the re-reservation search does **not** run. The delivery confirmation message and text message are sent if the company asks for them.

**End state.** The original Transfer is `done` with one move of 12; the backorder is `confirmed` with one move of 5; `WH/Stock` holds 3.00 Bolt (2.00 on Q1 plus 1.00 on Q2) minus the −1.00 on Q3, that is a net 2.00.

## 34.3 A put-away that walks two rules

**Starting state.** Arrival Location `WH/Stock`, whose internal descendants are `Bin 1`, `Bin 2` and `Bin 3`, all three carrying the Storage Category "Shelf" (maximum weight 40, product capacity 20 Bolt, mixing policy "If all products are same"). Bolt weighs 0.5.

| Location | On hand | Open lines arriving |
|---|---|---|
| `Bin 1` | 18.00 Bolt | — |
| `Bin 2` | 4.00 Paint | — |
| `Bin 3` | 0 | 5.00 Bolt |

**The rules on `WH/Stock`:**

| Rule | Product | Category | Container type | Target | Mode | Storage category | Priority |
|---|---|---|---|---|---|---|---|
| R1 | — | All | — | `WH/Stock` | No | — | 10 |
| R2 | Bolt | — | — | `WH/Stock` | Closest Location | Shelf | 10 |

**The request.** Put away 6.00 Bolt, no container, no packaging unit, no additional-quantity map.

**Step 1 — container type.** None.

**Step 2 — product set.** {Bolt}.

**Step 3 — category chain.** Bolt's own category and its ancestors, say {Hardware, All}.

**Step 4 — rule selection.** R1 passes (no product; its category "All" is in the chain; no container type). R2 passes (its product is in the set). Both selected.

**Step 5 — specificity sort.** R2's key is (false, **true**, false, false); R1's key is (false, false, false, **true**). Comparing the four booleans in order, descending: R2 wins on the second element. Order: R2, then R1.

**Step 6 — candidate Locations.** No Locations in the calling context, so the internal descendants of `WH/Stock`: `Bin 1`, `Bin 2`, `Bin 3` (and `WH/Stock` itself when it is internal, which it is).

**Step 7 — the occupancy map.** At least one candidate carries a storage category, and no typed container is involved, so the map is a quantity map for Bolt:

```
Bin 1 : 18.00 (records) + 0 (lines) = 18.00
Bin 2 : 0 + 0 = 0
Bin 3 : 0 + 5.00 = 5.00
WH/Stock : 0
```

**Step 8 — the walk, rule R2.** The mode is closest location, so the target `WH/Stock` is kept and its children are narrowed to those carrying the category "Shelf": `Bin 1`, `Bin 2`, `Bin 3`.

*First pass — prefer a Location that already holds BOLT.* Walk in order:
- `Bin 1`: occupancy 18.00 > 0, so it qualifies. Capacity check: forecasted weight = 18 × 0.5 = 9; mixing policy "same" — the only positive record is Bolt, and no open line targets `Bin 1` with another product, so it passes; weight test `40 < 9 + 0.5 × 6 = 12`? No; first quantity test `18 ≥ 20`? No; second `6 + 18 = 24 > 20`? **Yes** → the check fails. `Bin 1` is added to the rejected set.
- `Bin 2`: occupancy 0, not strictly positive, skipped by this pass.
- `Bin 3`: occupancy 5.00 > 0, so it qualifies. Capacity check: forecasted weight = 0 (records) − 0 (leaving) + 5 × 0.5 (arriving) = 2.5; mixing policy "same" — there is no positive record, but an open line targets `Bin 3` with Bolt, which is the same product, so it passes; weight test `40 < 2.5 + 3 = 5.5`? No; `5 ≥ 20`? No; `6 + 5 = 11 > 20`? No → the check **passes**. Return `Bin 3`.

**Step 9 — the answer.** `Bin 3`. Rule R1 is never reached.

**Variation.** Had `Bin 3` failed too, the first pass would have rejected it, the second pass would have walked `Bin 1` (already rejected, skipped), `Bin 2` (capacity check: the mixing policy "same" finds a positive Paint record, so it **fails**) and `Bin 3` (already rejected). R2 would have returned nothing, the walk would have moved to R1, whose target `WH/Stock` carries no storage category — and `WH/Stock` itself has no storage category either, so its capacity check passes trivially and `WH/Stock` is returned.

---

# 35. Reference: every rounding decision in the domain

| Place | Rounded at | Method |
|---|---|---|
| Converting the demand into the real quantity | the product unit's rounding step | half away from zero |
| Converting a detail line's quantity into the product unit | the product unit's rounding step | half away from zero |
| Summing the detail lines into the move's processed quantity | **not rounded** | — |
| Comparing the processed quantity against the demand for the status | the line unit's rounding step | — |
| Comparing the processed quantity against the demand for the backorder test | the `Product Unit` precision (a digit count) | — |
| Comparing the processed quantity against the demand for the backorder **question** | the `Product Unit` precision | — |
| Checking that a written processed quantity is representable | the `Product Unit` precision | half away from zero |
| Checking a detail line at completion | both the line unit's rounding step and the `Product Unit` precision, compared against each other | half away from zero |
| Re-expressing a reservation in the line unit | the line unit's rounding step, then back at the product unit's | down, then half away from zero |
| Deciding whether a reserved quantity fits an existing line | the `Product Unit` precision | half away from zero |
| Full-packaging reservation | the packaging unit's rounding step, then back | down, then half away from zero |
| Splitting a move: choosing the new move's unit | the `Product Unit` precision | half away from zero |
| Splitting a move: the surviving demand | the `Product Unit` precision | half away from zero after an unrounded conversion |
| The unit price after absorbing a negative move | the `Product Price` precision | half away from zero |
| The merge key's floating members | each field's own precision, formatted as text | half away from zero |
| The whole-container test | the `Product Unit of Measure` precision | zero test |
| Deleting empty quantity records | the greater of 6 digits and twice the shipped product-unit precision | — |
| Distributing a processed quantity over the lines | **not rounded** during the walk | — |
| Reducing a processed quantity | **not rounded** during the walk | — |
| Aggregating for a printed document | the move's line unit | — |
| The traceability quantity | the `Product Unit` precision | half away from zero |
| The label count per unit | whole part of the line quantity | truncated |
| Reception report labels | whole number at or above the demand | up |
| The product quantity figures | the product unit's rounding step | — |

Everything not in this table is exact arithmetic.
