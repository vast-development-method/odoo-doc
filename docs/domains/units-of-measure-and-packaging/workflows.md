# Units of Measure and Packaging — Workflows

End-to-end operational procedures. Each workflow states who performs it, what must be true
beforehand, the numbered steps, the records created or changed at each step, and what is true
afterwards. Numbers in the examples are taken from [`calculations.md`](calculations.md) so that
the two files can be checked against each other.

Roles used throughout:

| Role | What it means here |
|---|---|
| Settings administrator | May create, edit and delete units of measure and decimal precisions. |
| Catalogue manager | May edit products, their additional units and their packaging barcodes. |
| Warehouse manager | May edit package types, storage capacities and product categories. |
| Salesperson | May create and confirm customer orders. |
| Buyer | May create and confirm purchase orders. |
| Warehouse operator | May pick, pack and validate transfers. |
| Production planner | May create bills of materials and production orders. |
| Accountant | May create and post invoices and bills. |

---

## 1. Enabling the feature

**Performed by:** a settings administrator.

**Precondition:** none.

**Steps.**

1. Open the settings of the accounting, warehouse or purchasing application — all three expose
   the same control.
2. Turn on the setting labelled `Units of Measure & Packagings`.
3. Save.

**Records changed.** The settings record writes the feature group onto the groups that imply it,
which adds the group to every user who holds the implying group.

**Postcondition.** Unit columns and selectors become visible on order lines, bill lines, invoice
lines, moves, move lines, bills of materials and production orders. The additional-units list
appears on the product form. The reservation policy appears on the product category form.

**What does not change.** No stored quantity, no unit, no conversion. Turning the setting off
again hides the controls and changes nothing else.

---

## 2. Defining a packaging

**Performed by:** a settings administrator (to create the unit) and a catalogue manager (to
attach it to products).

**Precondition:** the feature is enabled; the reference unit already exists.

### 2.1 Creating the unit

1. Open the list of units and packagings, reached from the settings page link or from the
   technical menu.
2. Create a record.
3. Type the name. For a carton of twelve dozens, `Box of 12 Dozens`.
4. Type the contained quantity and choose the reference unit. For the carton: twelve, of
   `Dozens`.
5. Save.

**Validation applied.** The contained quantity must not be zero. If no reference unit is chosen,
the contained quantity must be exactly one.

**Records created.** One unit of measure row, with:

- the contained quantity as typed;
- the absolute quantity derived as the contained quantity multiplied by the reference unit's
  stored absolute quantity — one hundred forty-four for the example;
- the sequence derived as the smaller of one thousand and one hundred times the contained
  quantity — one thousand for the example;
- the hierarchy path built from the reference unit's path.

**Postcondition.** The unit converts against every other unit of its tree. It is **not yet**
usable on any document, because no product lists it.

### 2.2 Creating a deeper packaging

1. Repeat 2.1 with the name `Pallet of 40 Boxes`, a contained quantity of forty and the reference
   unit `Box of 12 Dozens`.

**Postcondition.** The absolute quantity is five thousand seven hundred sixty. The pallet
converts against units, packs of six, dozens and boxes.

**Order matters.** Creating the pallet before the box is impossible, because the box must exist
to be chosen as reference unit. This is also why the installation order of
[`configuration.md`](configuration.md) is prescriptive.

### 2.3 Attaching the packaging to products

1. Open a product.
2. In the additional-units list, add `Box of 12 Dozens` and `Pallet of 40 Boxes`.
3. Save.

**Validation applied.** The product's own unit may not be added to the list.

**Postcondition.** Every document line for that product now offers the product's own unit, the
box and the pallet. Moves and move lines for the product offer the same three, plus any unit used
by the product's vendors.

### 2.4 Binding a barcode to a packaging

**Performed by:** a catalogue manager.

1. Open the packaging unit, or open the product's barcode list.
2. Add a row: the unit, the product variant, the barcode, and optionally the company.
3. Save.

**Validation applied.** The barcode must not already be used by another packaging row anywhere in
the installation, and must not already be used by any product variant.

**Postcondition.** Scanning that barcode in the warehouse application identifies the variant and
the packaging quantity at once.

---

## 3. Selling in a packaging

**Performed by:** a salesperson, then a warehouse operator, then an accountant.

**Precondition:** the feature is enabled; the product lists the packaging among its additional
units; stock is kept in the product's own unit.

**Worked scenario.** A product whose own unit is `Units`, with an additional unit `Box of 12`.
The customer orders five boxes. Ninety-two units are on hand and free.

### 3.1 Quoting

1. The salesperson creates a quotation and adds a line for the product.
2. The line's unit defaults to the product's own unit, `Units`, and the quantity defaults to one.
3. The salesperson changes the unit to `Box of 12`. The allowed list permits it.
4. The salesperson types a quantity of five.

**What the system recomputes at step 3 and step 4.**

- The **price list rule** is re-matched. The quantity is converted into the product's own unit for
  matching: five boxes is sixty units.
- The **unit price** is recomputed. The price found for the product is converted from the
  product's own unit into the line's unit: a catalogue price of two per unit becomes twenty-four
  per box.
- The **discount** is recomputed from the ratio between the price before the rule and the price
  after it.

**Records created.** One sales order line with a quantity of five and the unit `Box of 12`.

### 3.2 Confirming

1. The salesperson confirms the order.

**Records created.** A delivery transfer and one stock move per storable line.

**For our line, the move is created with:**

| Field | Value | How |
|---|---|---|
| Unit | `Units` | The propagation parameter is off by default, so the move takes the product's own unit. |
| Demand | 60 | Converted from five boxes into units, half away from zero. |
| Real quantity | 60 | Derived from the demand and the move's unit; an identity here. |
| Packaging unit | `Box of 12` | Copied from the order line's unit by the sales capability. |
| Packaging quantity | 5 | The demand converted from the move's unit into the packaging unit, away from zero. |

**If the propagation parameter were on**, the move's unit would be `Box of 12`, its demand five,
its real quantity sixty, its packaging unit `Box of 12` and its packaging quantity five.

### 3.3 Reserving

1. Reservation runs, automatically or on demand.

**Steps the system performs.**

1. Gather the quantities on hand for the product at the source location. Ninety-two are free.
2. If the product's category requires full packagings **and** the move carries a packaging unit,
   snap the smaller of the wanted quantity and the available quantity down to a whole number of
   packagings. Sixty is already five whole boxes, so nothing changes. Had only fifty-eight been
   free, the snap would give forty-eight.
3. If the move's unit differs from the product's own unit, convert the quantity down into the
   move's unit and back up half away from zero. Here the move is in the product's own unit, so
   this step is skipped.
4. Reserve.

**Records changed.** The reserved quantity on the quantity-on-hand row increases by sixty. One
move line is created with the move's unit and a suggested quantity.

### 3.4 Picking

1. The operator opens the transfer.
2. The operator may pick in units or, if the operator changes the move line's unit, in boxes.

**If the operator picks sixty units:** the move line stores sixty with the unit `Units`, and its
quantity in the product's unit is sixty.

**If the operator changes the move line's unit to `Box of 12` and picks five:** the move line
stores five with the unit `Box of 12`, and its quantity in the product's unit is sixty, converted
half away from zero.

**Either way** the move's picked quantity is the sum of the move lines' quantities converted into
the *move's* unit without rounding: sixty.

### 3.5 Validating

1. The operator validates the transfer.

**Guard applied.** Every move's picked quantity must survive a round trip through the `Product
Unit` precision. If the precision was reduced while the transfer was open, the validation is
refused with the rounding message.

**Records changed.** Quantities on hand fall by sixty. The move becomes complete and its unit is
frozen. The delivered quantity on the order line is recomputed as the move's picked quantity
converted into the line's unit half away from zero: five boxes.

### 3.6 Invoicing

1. The accountant creates the invoice from the order.

**Records created.** An invoice line with the order line's unit, `Box of 12`, and a quantity
taken from the ordered or the delivered figure according to the product's invoicing policy: five.

**Records changed after posting.** The invoiced quantity on the order line is recomputed as the
invoice line's quantity converted into the order line's unit, away from zero: five.

### 3.7 What a rebuild should check on this workflow

| Checkpoint | Expected |
|---|---|
| Order line | 5 `Box of 12` |
| Move demand and unit | 60 `Units` (propagation off) or 5 `Box of 12` (propagation on) |
| Move packaging quantity and unit | 5 `Box of 12` in both cases |
| Reserved | 60 in the product's own unit |
| Move line in the product's unit | 60 |
| Quantity on hand change | −60 |
| Delivered on the line | 5 |
| Invoiced on the line | 5 |

---

## 4. Selling in a packaging when stock is short

**Precondition:** as in workflow 3, but only fifty-eight units are free, and the product's
category requires **full packagings**.

**Steps.**

1. The order is confirmed; the move demands sixty units.
2. Reservation gathers fifty-eight free units.
3. The full-packaging snap applies: the smaller of sixty and fifty-eight is fifty-eight;
   fifty-eight divided by twelve is four and eight hundred thirty-three thousandths; rounded
   towards zero to a whole number, four; multiplied back, forty-eight.
4. Forty-eight units are reserved. Ten stay free.
5. The transfer shows a partial availability.

**With the policy set to partial instead:** fifty-eight units are reserved, and the delivery note
will read four and eighty-four hundredths boxes — the demand of fifty-eight converted into the
packaging unit away from zero.

**Postcondition in the full-packaging case.** The customer receives four whole boxes now and a
backorder for the fifth.

---

## 5. Buying in a vendor's unit

**Performed by:** a buyer, then a warehouse operator, then an accountant.

**Precondition:** the product exists with its own unit; a vendor price list line quotes the
product in a different unit.

**Worked scenario.** A product whose own unit is `Units`. A vendor quotes one hundred twenty per
`Box of 12 Dozens` with a five per cent discount and a minimum quantity of two boxes.

### 5.1 Creating the line

1. The buyer adds a line for the product on a purchase order for that vendor.
2. The unit defaults to the product's own unit. The buyer changes it to `Box of 12 Dozens`; the
   allowed list for a purchase line includes vendor units, so the box is offered even if the
   product does not list it among its additional units.
3. The buyer types a quantity of three.

**What the system recomputes.**

- The **vendor line** is selected by comparing the quantity, converted into the vendor's unit,
  against each candidate's minimum quantity. Three boxes is three boxes; three is not below two,
  so the line applies.
- The **unit price** is the vendor's price converted from the vendor's unit into the line's unit.
  Both are the box here, so the conversion short-circuits and the price is one hundred twenty.
- The **discount** is copied from the vendor line: five per cent.
- The **total quantity** is stored as the ordered quantity converted into the product's own unit
  away from zero: three boxes is four hundred thirty-two units.
- The **unit price in the product's unit** is displayed as the line price converted into the
  product's own unit: one hundred twenty times one divided by one hundred forty-four, which is
  five sixths.

**If the buyer had instead left the line in `Units` and typed four hundred thirty-two:** the
vendor selection would convert four hundred thirty-two units into boxes, giving three, which
still satisfies the minimum; the price would be converted from the box to the unit, giving five
sixths per unit.

### 5.2 Confirming

1. The buyer confirms the order.

**Records created.** A receipt transfer and one move per line.

**For our line,** with the propagation parameter off:

| Field | Value |
|---|---|
| Move unit | `Units` |
| Move demand | 432 (three boxes converted half away from zero) |
| Packaging unit | `Box of 12 Dozens` (copied from the purchase line by the purchasing capability) |
| Packaging quantity | 3 |

**Split across existing moves.** Where part of the quantity can be attached to an existing move
and part cannot, the attachable part and the remainder are each adjusted independently by the
propagation rule, and the remainder becomes an extra move with no chaining.

### 5.3 Receiving

1. The operator records what arrived and validates.
2. The received quantity on the purchase line is recomputed from the completed moves, converted
   into the line's unit away from zero.

**If four hundred thirty-two units arrive:** the received quantity is three boxes exactly.

**If four hundred arrive:** four hundred divided by one hundred forty-four is two and seven
hundred seventy-seven thousandths; away from zero at two digits gives two and seventy-eight
hundredths boxes. The buyer sees a received quantity of two and seventy-eight hundredths against
an ordered three.

### 5.4 Billing

1. The accountant records the vendor bill.
2. The bill line's unit defaults to the unit of the first applicable vendor price list line,
   which is the box.
3. The invoiced quantity reported back onto the purchase line is the bill line's quantity
   converted into the purchase line's unit away from zero.

---

## 6. Manufacturing with a different unit at each level

**Performed by:** a production planner.

**Precondition:** the feature is enabled.

**Worked scenario.** A bill of materials that yields one `Box of 12 Dozens` of a finished product
from three `kg` of a material whose own unit is `g`, and one `Units` of a fastener.

### 6.1 Defining the bill

1. Create the bill. Its unit defaults to the first unit in identifier order and is immediately
   replaced by the finished product's own unit when the product is chosen. The planner changes it
   to `Box of 12 Dozens`.
2. Set the produced quantity to one.
3. Add a component line for the material. Its unit defaults to the component's own unit, `g`; the
   planner changes it to `kg` and types three.
4. Add a component line for the fastener, one `Units`.

### 6.2 Creating a production order

1. Create a production order for the finished product.
2. Set its unit — in this example the order's unit is `Box of 12 Dozens` as well — and a quantity
   of two and one half.

### 6.3 What the system computes

1. **The scaling factor.** The order's quantity is converted into the bill's unit **unrounded**
   and divided by the bill's produced quantity:

   ```formula
   factor = ( 2.5 × 144 ÷ 144 ) ÷ 1 = 2.5
   ```

2. **Each component's demand.** The component line's quantity multiplied by the factor, then
   rounded **away from zero** at the `Product Unit` precision:

   ```formula
   material  = round_away_from_zero( 3 × 2.5 ) = 7.5 kg
   fastener  = round_away_from_zero( 1 × 2.5 ) = 2.5 Units
   ```

3. **Each raw-material move.** Created with the component line's unit and that demand; its real
   quantity is the demand converted into the component's own unit half away from zero:

   ```formula
   material real quantity = round_half_away_from_zero( 7.5 × 1000 ÷ 1 ) = 7500 g
   fastener real quantity = round_half_away_from_zero( 2.5 × 1 ÷ 1 ) = 2.5 Units
   ```

4. **The finished move.** Created for the produced quantity converted into the finished product's
   own unit.

**Note the fractional fastener.** The system does not object to consuming two and a half
fasteners; whole-component enforcement is not part of this domain. A business that needs whole
components must either set the `Product Unit` precision to zero digits — which affects everything
— or plan whole batches.

### 6.4 Recording production

1. The planner records a producing quantity in the order's unit.
2. The producing quantity is converted into the product's own unit half away from zero before it
   becomes a finished move quantity.
3. For a serial-tracked finished product the producing quantity is instead derived from the count
   of serial numbers, converted from the product's own unit into the order's unit half away from
   zero.

---

## 7. Changing a product's own unit

**Performed by:** a catalogue manager.

**Precondition:** the product exists.

**Steps.**

1. Open the product and change the unit.
2. **If any historical record exists** — a stock move, a sales order line, a purchase order line
   or a bill of materials — a warning is shown before saving, stating that the change applies a
   conversion of one old unit equals one new unit and that existing records will be updated by
   replacing the unit name. The user may proceed.
3. On saving, each capability checks its own records. For each kind, the existing records are
   grouped by the unit they hold. If any group's unit differs from the product's *current* own
   unit, the save is **refused** with the message naming the offending unit.
4. If every check passes, the unit reference on all those records is rewritten to the new unit,
   **without touching any quantity**.
5. Where the accounting capability is installed, a further check refuses the save when any posted
   journal item for the product holds a unit different from the template's own unit.

**Postcondition.** Every affected record now names the new unit and holds the number it held
before. A move for sixty is now a move for sixty of the new unit.

**When the change is impossible.** If any document was created in a unit other than the product's
own, the change cannot be made at all. The prescribed remedy, stated in the message, is to
archive the product and create a new one.

**Rebuild note.** This is deliberately *not* a conversion. A rebuild that converts will produce
different quantities on every existing document.

---

## 8. Changing the global rounding precision

**Performed by:** a settings administrator.

**Precondition:** understand that the change is global and immediate.

**Steps.**

1. Open the decimal precisions and find the one named `Product Unit`.
2. Change the number of digits.
3. If the number is being reduced, a warning appears stating that the precision has been reduced,
   that existing data will not be updated, and that changing decimal precisions in a running
   database is not recommended. The user may proceed.
4. Save.

**Records changed.** One decimal precision row. The lookup cache is cleared.

**Immediate effects.**

- Every unit's reported rounding precision changes.
- Every subsequent conversion rounds differently.
- Every subsequent quantity comparison and zero test uses the new grid.

**Effects a rebuild must *not* implement.**

- No stored quantity is rewritten.
- No document is recomputed.
- No warning is issued about existing off-grid data.

**Consequence to plan for.** Any open transfer whose picked quantity is now off the grid fails
validation with the rounding message until the quantity is edited or the precision is restored.

**Recommended sequence when a business must increase precision.** Increase it; nothing breaks,
because the old grid is a subset of the new one. **Decreasing** should be done only when no
transfer is open.

---

## 9. Reserving only full packagings

**Performed by:** a warehouse manager (configuration), then the system (execution).

**Steps to configure.**

1. Open the product category.
2. Set the reservation policy to `Reserve Only Full Packagings`. The control is visible only when
   the feature is enabled.
3. Save.

**Execution, per reservation.**

1. The reservation is invoked with the move's packaging unit in context. This happens only for
   moves that carry one, that is moves originating from a sales or purchase document line.
2. The available quantity is computed in the product's own unit.
3. The smaller of the wanted quantity and the available quantity is snapped down to a whole
   number of packagings, using the packaging quantity derived by converting one packaging unit
   into the product's own unit away from zero.
4. The snapped figure becomes the available quantity for the rest of the reservation.

**Edge case that the identity short-circuit protects.** When the packaging unit *is* the product's
own unit — which happens whenever a line was created in the product's own unit — the snap returns
the quantity untouched. Without that short-circuit, twenty-two and forty-three hundredths would
become twenty-two.

---

## 10. Scanning a packaging barcode

**Performed by:** a warehouse operator.

**Precondition:** a Product Unit Barcode row binds the barcode to the pair (variant, unit).

**Steps.**

1. The operator scans the code in the warehouse application.
2. The system looks the code up among product variants first and among packaging rows second, or
   in whichever order the nomenclature dictates; the two namespaces are disjoint by construction,
   so at most one match exists.
3. A packaging match yields both the variant and the unit.
4. The operator's current operation gains a move line for that variant with that unit and a
   quantity of one, or the quantity of an existing matching line is increased by one.

**Postcondition.** One scan records one packaging, not one item. The move line's quantity in the
product's own unit is the packaging's absolute quantity ratio applied to one, rounded half away
from zero.

---

## 11. Printing labels for picked goods

**Performed by:** a warehouse operator.

**Steps.**

1. Choose the transfer and open the label printing wizard.
2. For each move line, the system decides how many labels to print.
3. The decision uses the shared-ancestor test: only a move line whose unit belongs to the
   **counting** tree yields one label per item; a move line in a mass, volume, length, surface,
   time or energy unit yields a single label, because "one label per kilogram" is meaningless.

**Postcondition.** A move line of five `Box of 12` — a counting-tree unit — yields labels for the
quantity converted into the counting root, that is sixty; a move line of seven and a half `kg`
yields one label.

---

## 12. Receiving an exchanged electronic document with a unit

**Performed by:** the document exchange process.

**Steps.**

1. The incoming document names a quantity, a standard trade code and a product reference.
2. The trade code is translated into a unit through the reverse of the trade-code mapping; an
   unrecognised code becomes the counting unit.
3. The product is matched.
4. The shared-ancestor test is applied between the translated unit and the matched product's own
   unit. **If they do not share an ancestor, the translated unit is discarded** and the product's
   own unit is used on the created line, with the quantity left as it was.
5. The line is created.

**Postcondition.** A document naming kilograms for a product counted in units produces a line in
units with the number of kilograms as its quantity. This is a deliberate fail-soft: the document
is accepted, and the discrepancy is visible to a human rather than blocking the exchange.

---

## 13. Adding a product to a cart in a chosen packaging

**Performed by:** a storefront visitor.

**Steps.**

1. The visitor opens a product page. A unit selector appears only when the product has multiple
   units in the sense of the derived predicate: the feature is enabled, the product is not a
   combination offer, and its own unit together with its additional units number more than one.
2. The visitor chooses a unit and a quantity and adds to the cart.
3. The system validates the unit: if the product does not support multiple units, the chosen unit
   is ignored and the product's own unit is used; otherwise the unit must be among the product's
   available units, and a unit that is not is refused with the message stating that the product is
   not available in that unit.
4. An existing cart line for the same product **and the same unit** is increased; otherwise a new
   line is created.
5. Where stock is checked, the available quantity is converted from the product's own unit into
   the cart line's unit — the availability itself unrounded, the cart quantity rounded — before
   the comparison.

**Postcondition.** Two cart lines may exist for the same product in different units; they are not
merged.

---

## 14. Correcting a packaging definition that is already in use

**Performed by:** a settings administrator.

**The problem.** A unit was created as a box of twelve and should have been a box of twenty-four.
Documents already exist.

**What happens if the contained quantity is simply changed.**

1. A warning appears only if the unit is protected; a user-created packaging is not protected, so
   **no warning appears at all**.
2. The absolute quantity of the unit and of every descendant is recomputed immediately.
3. **Every existing document that names the unit now means something different.** An open order
   line for five boxes now means one hundred twenty items rather than sixty. A completed move is
   unaffected, because it stores its own real quantity in the product's own unit; but its
   packaging quantity, being derived, is recomputed and will disagree with the delivery note that
   was printed.

**The safe procedure instead.**

1. Create a **new** unit with the correct contained quantity and a distinguishable name.
2. Add it to the additional units of the affected products.
3. Remove the old unit from those lists so no new line can use it.
4. Archive the old unit once every document naming it is complete.
5. Never delete the old unit: the referential restriction on document lines will refuse, and
   deleting would cascade to any descendant.

---

## 15. Auditing a quantity that looks wrong

**Performed by:** anyone investigating a discrepancy.

**Procedure.**

1. Identify the document and the field. Establish which unit that field is expressed in, from the
   table in [`entities.md`](entities.md) section 6.
2. Establish the physical quantity: the stored number multiplied by the unit's absolute quantity.
3. Walk back to the previous state in the quantity lifecycle of
   [`state-machines.md`](state-machines.md) section 3, and compute the same product there.
4. The two should agree to within one step of the `Product Unit` precision multiplied by the
   absolute quantity of the unit that was rounded at the crossing.
5. If the discrepancy exceeds that bound, the cause is one of:
   - a unit's contained quantity was changed after the document was created;
   - the `Product Unit` precision was changed after the document was created;
   - the product's own unit was changed, which relabels without converting;
   - a rebuild applied the wrong rounding method at the crossing — consult the master table in
     [`calculations.md`](calculations.md) section 12.

**Worked audit.** An order line reads seven `Units`; the delivery note reads nought point five
nine `Dozens`; the customer complains that they received seven, not seven and eight hundredths.

1. Seven units is seven root units.
2. Nought point five nine dozens is seven and eight hundredths root units.
3. The crossing is "move demand to packaging quantity", which rounds **away from zero**. The
   bound is one hundredth of a dozen, that is twelve hundredths of a unit. The discrepancy of
   eight hundredths of a unit is within the bound.
4. Conclusion: the system is behaving as specified. The delivery note's packaging figure is an
   over-statement inherent in away-from-zero rounding, and the physical quantity shipped —
   recorded in the product's own unit — is exactly seven.
