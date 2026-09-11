# Units of Measure and Packaging — Glossary

Every term used in this folder, defined in full. Terms are listed alphabetically. Where a term
names a stored field, the storage name is given in code font.

---

**Absolute quantity** (`factor`) — How many of the **root unit** of its tree one of a unit
contains. Derived, stored, and recomputed recursively: a unit's absolute quantity is its
**contained quantity** multiplied by its **reference unit**'s stored absolute quantity, or its
contained quantity alone when it has no reference unit. This is the number the conversion
arithmetic uses. Declared with unlimited decimal precision.

**Additional trading units** (`uom_ids`) — The list of units, beyond a product's **own unit**, in
which that product may be ordered, sold, purchased, moved and invoiced. A unit becomes a
**packaging** of a product by being on this list. The product's own unit may not appear on it.

**Allowed units** (`allowed_uom_ids`) — The derived list of units a particular document line may
use. For a sales line, an invoice line and a bill-of-materials line: the product's own unit plus
its additional trading units. For a purchase line, a stock move and a stock move line: the same,
plus the units of the product's **vendor price list lines**. A selection restriction, not a
stored constraint.

**Ancestor** — A unit reachable by following **reference unit** links upwards. A unit is
considered to share an ancestor with itself.

**Archival flag** (`active`) — Whether a unit is offered for selection. Clearing it hides the
unit from default listings and selectors and changes nothing else: existing references remain
valid and conversions through the unit keep working. The supported alternative to deletion.

**Away from zero** (`UP`) — The rounding method that pushes any non-zero fractional part to the
next step further from zero. **The default rounding method of the quantity conversion
operation.** Never produces zero from a non-zero input.

**Barcode** — In this domain, a scannable code bound to a (product variant, unit) pair by a
**Product Unit Barcode**, or a code identifying a **package type**. Product barcodes and
packaging barcodes share one namespace; package type barcodes are in a separate one.

**Bill of materials** — A production recipe holding a produced quantity in its own unit and
component lines each with their own unit. Owned by the manufacturing domain; listed here for the
conversions it performs.

**Bulk weight** — The weight of the goods in a transfer that are not inside any package, computed
by converting each move line's quantity into the product's own unit and multiplying by the
product's dimensionless weight.

**Company scoping** — Filtering a record by the reader's active companies. Units of measure are
**not** company scoped; package types and product unit barcodes are.

**Comparison** — The three-way test between two quantities at the `Product Unit` precision,
returning minus one, zero or plus one. Both values are rounded **before** the difference is
taken, which is why it is not interchangeable with the **zero test**.

**Compensation term** — The small quantity added to or subtracted from a normalised value inside
the rounding function to absorb binary representation error. Equal to two raised to the power of
the base-two logarithm of the absolute normalised value, minus fifty.

**Contained quantity** (`relative_factor`) — How many of its **reference unit** one of a unit
contains. The only quantity a user types. Required, defaults to one, may not be zero, and must be
exactly one on a **root unit**. Stored with unlimited decimal precision.

**Conversion** — See **quantity conversion** and **price conversion**.

**Counting tree** — The **tree** whose root is the counting unit, holding the pack of six, the
dozen, and any packaging a business creates on top of them. The only tree in which "one label per
item" is meaningful.

**Decimal precision** (`decimal.precision`) — A named number of decimal digits. The record named
`Product Unit` supplies the rounding precision of every unit of measure.

**Demand** (`product_uom_qty`) — The planned quantity of a stock move, expressed in the move's
unit.

**Destination unit** — The unit a quantity or a price is being converted to.

**Display name** — The label shown for a record. For a unit, either the plain name or the
**formatted display name**.

**Euclidean division at a precision** — A division yielding a whole quotient and a remainder,
computed by snapping both operands onto the precision grid and scaling them to whole numbers
before dividing, so that neither the quotient nor the remainder carries representation error.

**Feature group** — A security group that grants no access rights and exists only to show or hide
controls. The multiple-units group is one.

**Forest** — The set of all **trees** of units. Seven trees exist as delivered.

**Formatted display name** — The richer label of a unit, requested by a caller for selection
lists: the unit's name, a tab character, then the contained quantity and the reference unit's
name enclosed in double hyphen-minus characters. Falls back to the plain name for a root unit.

**Full-packaging reservation** — The policy under which a partial packaging is never reserved;
see **whole-packaging rounding** and **reservation policy**.

**Half away from zero** (`HALF-UP`) — The rounding method that goes to the nearest step, with a
value exactly halfway going away from zero. **The default rounding method of the rounding
function itself**, and the method used at every crossing that produces a physical quantity.

**Half to even** (`HALF-EVEN`) — The rounding method that goes to the nearest step, with a value
exactly halfway going to the nearer even multiple of the step. Available but not used by this
domain.

**Half towards zero** (`HALF-DOWN`) — The rounding method that goes to the nearest step, with a
value exactly halfway going towards zero. Available but not used by this domain.

**Hierarchy path** (`parent_path`) — The materialised path of ancestor identifiers of a unit,
from the root down to and including the unit itself, separated by solidus characters and ending
with one. Indexed. Used by the **shared-ancestor test**.

**Identity short-circuit** — Any of the several places where an operation returns its input
unchanged because the source and destination units are the same record: in quantity conversion
(the factor arithmetic is skipped but the rounding still runs), in price conversion (the price is
returned untouched), and in whole-packaging rounding (the quantity is returned untouched).

**Inflation** — The systematic over-statement produced by away-from-zero rounding. Worst for
small quantities of large units: one item is one hundredth of a pallet of five thousand seven
hundred sixty, an over-statement of more than fifty-six fold.

**Length unit label** (`length_uom_name`) — The display name of the unit in which the
dimensionless length, width and height numbers of a package type are to be read, resolved from
the volume system parameter.

**Materialised path** — See **hierarchy path**.

**Move** — See **stock move**.

**Move line** — See **stock move line**.

**Multiple-units feature group** — The security group named `Manage Multiple Units of Measure`.
A pure **feature group**: it reveals unit selectors and packaging controls throughout the
application and grants no access right.

**Own unit** (`uom_id`) — A product's primary unit: the unit in which its stock is kept, its cost
is expressed, and to which every other quantity for it is ultimately converted. Required.
Defaults to the counting unit. Changing it **relabels** existing records rather than converting
them.

**Package** — A physical container holding goods. Owned by the warehouse domain; relevant here
because it takes its type, its numbering and its weight arithmetic from this domain.

**Package type** (`stock.package.type`) — The definition of a physical container: outer
dimensions, tare weight, maximum weight, barcode, numbering sequence, reuse policy, storage
capacities and routes. Distinct from a unit of measure, which defines a quantity rather than a
container.

**Packaging** — A unit of measure used to describe how many items travel together. Not a separate
entity: a packaging is a unit of measure that appears in some product's **additional trading
units**. A packaging is "of a product" only through that list.

**Packaging quantity** (`packaging_uom_qty`) — The demand of a stock move expressed in the move's
**packaging unit**, converted away from zero. Stored.

**Packaging unit** (`packaging_uom_id`) — The unit a stock move inherits from the sales or
purchase document line it came from, used for printing and for the full-packaging reservation
policy. Defaults to the move's own unit where no document line supplied one.

**Physical quantity** — A quantity expressed in the root unit of its tree: the stored number
multiplied by its unit's absolute quantity. The invariant that a rebuild can test at every stage
of a quantity's life.

**Precision digits** — The number of decimal digits of a **decimal precision** record. Two for
`Product Unit` as delivered.

**Price conversion** — Turning a price per one source unit into a price per one destination unit.
Multiplies by the destination unit's absolute quantity and divides by the source unit's. **Never
rounded.** The inverse of quantity conversion.

**Product Unit** — The name of the **decimal precision** record that supplies the rounding
precision of every unit of measure. Two digits as delivered.

**Product Unit Barcode** (`product.uom`) — The entity binding one barcode to one (product
variant, unit of measure) pair.

**Propagation parameter** (`stock.propagate_uom`) — The system parameter deciding whether a
document line's unit is carried onto the stock move generated from it, or whether the move is
created in the product's own unit. Unset by default, meaning the move takes the product's own
unit.

**Quantity conversion** — Turning a quantity expressed in a source unit into a quantity expressed
in a destination unit. Multiplies by the source unit's absolute quantity, divides by the
destination unit's, and rounds onto the destination unit's rounding precision with the requested
method, defaulting to **away from zero**.

**Quantity in the product's unit** (`quantity_product_uom`) — The stored copy of a stock move
line's picked quantity, converted into the product's own unit half away from zero. The number
that changes quantities on hand.

**Quantity on hand** — A stored stock figure. Always expressed in the product's **own unit**;
carries no unit reference of its own.

**Real quantity** (`product_qty`) — The stored copy of a stock move's demand, converted into the
product's own unit half away from zero. May not be written directly.

**Reference data** — Records delivered with a capability. The thirty units, the `Product Unit`
precision and the feature group are reference data. Delivered as non-updatable, so an upgrade
does not overwrite an administrator's changes.

**Reference unit** (`relative_uom_id`) — The unit that a unit's **contained quantity** is
expressed in. Empty on a **root unit**. Deleting a unit cascades to every unit that names it
here.

**Reservation policy** (`packaging_reserve_method`) — The product-category setting deciding
whether a partial packaging may be reserved. Values: reserve only full packagings, or reserve
partial packagings. Defaults to partial.

**Root unit** — A unit with no reference unit. Defines a **tree**. Its contained quantity must be
exactly one, and therefore so is its absolute quantity.

**Rounding function** — The single platform function that rounds a value onto a precision grid
under one of five methods, with normalisation, accurate step inversion and a **compensation
term**.

**Rounding precision** (`rounding`) — The smallest representable quantity in a unit: ten raised
to minus the number of digits of `Product Unit`. Identical on every unit; exposed per unit only
for convenience.

**Sequence** (`sequence`) — The ordering weight of a unit, derived on creation as the smaller of
one thousand and the whole-number part of one hundred times the contained quantity, and editable
thereafter.

**Shared-ancestor test** — The predicate deciding whether two units may meaningfully be
converted: true when their **hierarchy paths** agree on at least their first element. Costs no
database access beyond the two records.

**Shipping weight** — A transfer's total weight: the **bulk weight** plus, for each outermost
destination package, either the declared shipping weight of that package or its computed weight
(the package type's tare plus the converted weight of its contents).

**Snap** — See **whole-packaging rounding**.

**Source unit** — The unit a quantity or a price is currently expressed in.

**Step** — The distance between two representable values on a precision grid. Ten raised to minus
the number of digits, or a directly supplied positive number. The rounding function supports
arbitrary steps; this domain uses one hundredth for quantities and one for whole-packaging
rounding.

**Stock move** (`stock.move`) — A planned or completed movement of goods, holding a demand in its
own unit, a real quantity in the product's own unit, and a packaging unit and packaging quantity.

**Stock move line** (`stock.move.line`) — A recorded pick, holding a quantity in its own unit and
a quantity in the product's own unit.

**Tare** (`base_weight`) — The weight of an empty container, held on a **package type** and added
to the content weight when a package's weight is computed.

**Timesheet input control** (`timesheet_widget`) — An annotation on a unit naming the input
control the desktop client should use when a duration is entered in that unit.

**Towards zero** (`DOWN`) — The rounding method that discards the fractional part. Used by the
first leg of the reservation double conversion and by the full-packaging snap.

**Trade code** — The standard international code for a unit, carried on exchanged invoices,
resolved from the unit's external identifier through a fixed mapping, with a fallback to the code
for a plain piece.

**Tree** — A connected set of units linked by **reference unit** links, headed by a **root
unit**. Conversion is possible within a tree and impossible between trees. Replaces the older
notion of a unit category; no category entity exists.

**Unit of measure** (`uom.uom`) — A named quantity scale, defined by how much of its reference
unit one of it contains. Serves simultaneously as a measurement scale, a counting scale and a
packaging definition.

**Unit propagation** — See **propagation parameter**.

**Unlimited decimal precision** — The storage characteristic of the contained quantity and the
absolute quantity: an exact decimal with no fixed scale, so that values such as twenty-eight and
three thousand four hundred ninety-five ten-thousandths survive storage exactly. Not to be
confused with the **rounding precision**, which governs conversion results.

**Unprotected list** — The set of short external identifiers whose units may be deleted even
though they were delivered as reference data. By default the working hour, the dozen and the pack
of six; the working hour is removed from the list when the time-recording capability is
installed.

**Vendor price list line** (`product.supplierinfo`) — A vendor's quotation for a product, holding
a unit, a minimum quantity in that unit, a price per that unit and a discount.

**Volume unit label** (`volume_uom_name`) — The display name of the unit in which a product's
dimensionless volume number is to be read, resolved from the volume system parameter.

**Weight unit label** (`weight_uom_name`) — The display name of the unit in which a product's or
a package type's dimensionless weight numbers are to be read, resolved from the weight system
parameter.

**Whole-packaging rounding** — Snapping a quantity onto a whole multiple of a packaging quantity:
divide by the packaging quantity, round to a whole number with the requested method, multiply
back. Never implemented with a remainder operation. Returns the quantity untouched when the
packaging unit and the destination unit are the same record.

**Zero test** — The test deciding whether a quantity counts as zero at the `Product Unit`
precision: true when the value is exactly zero, or when its rounded absolute value is strictly
smaller than the step. Rounds **after** any subtraction, which is why it is not interchangeable
with the **comparison**.
