# Units of Measure and Packaging — Acceptance Criteria

Numbered Given / When / Then scenarios with concrete numbers. A rebuild that satisfies all of
them behaves equivalently to the system being specified, for this domain.

Unless a scenario says otherwise, the following baseline holds:

- the decimal precision named `Product Unit` has **two** digits, so every unit's rounding
  precision is one hundredth;
- the multiple-units feature group is enabled;
- the thirty delivered units exist with the contained quantities of
  [`configuration.md`](configuration.md), the dozen having been reactivated;
- three further units exist, created in this order:

  | Name | Reference unit | Contains | Absolute quantity |
  |---|---|---|---|
  | `Box of 12` | `Units` | 12 | 12 |
  | `Box of 12 Dozens` | `Dozens` | 12 | 144 |
  | `Pallet of 40 Boxes` | `Box of 12 Dozens` | 40 | 5760 |

- a product **P** exists whose own unit is `Units` and whose additional units are `Box of 12`,
  `Box of 12 Dozens` and `Pallet of 40 Boxes`;
- the unit propagation parameter is unset, so stock moves are created in the product's own unit.

---

## A. Reference data and derived values

**SCENARIO-001 — The delivered counting tree exists.**
Given a fresh installation of the units capability,
When the units are listed including archived ones,
Then `Units` exists with no reference unit, a contained quantity of one and an absolute quantity
of one; `Pack of 6` exists with `Units` as reference unit, a contained quantity of six and an
absolute quantity of six; `Dozens` exists with `Units` as reference unit, a contained quantity of
twelve, an absolute quantity of twelve, and an archival flag of no.

**SCENARIO-002 — The delivered mass tree exists.**
Given a fresh installation,
When the units are listed including archived ones,
Then `g` has an absolute quantity of one; `kg` has one thousand; `Ton` has one million; `oz` has
twenty-eight and three thousand four hundred ninety-five ten-thousandths; `lb` has four hundred
fifty-three and five hundred ninety-two thousandths; and `oz` and `lb` are archived.

**SCENARIO-003 — The delivered length tree reproduces its floating-point residue.**
Given a fresh installation,
When the absolute quantities of the length tree are read,
Then `mm` is one, `cm` is ten, `m` is one thousand, `km` is one million, `in` is twenty-five and
four tenths, `ft` is the value produced by multiplying twelve by twenty-five and four tenths in
binary floating point, `yd` is three times that value, and `mi` is one thousand seven hundred
sixty times the yard's value.
And a rebuild that uses exact decimal arithmetic instead documents the deviation for `ft`, `yd`
and `mi`.

**SCENARIO-004 — Every unit reports the same rounding precision.**
Given the baseline,
When the rounding precision of each of the thirty-three units is read,
Then all thirty-three return one hundredth.

**SCENARIO-005 — The rounding precision follows the global setting.**
Given the baseline,
When the `Product Unit` decimal precision is changed to three digits,
Then every unit reports one thousandth.

**SCENARIO-006 — An unknown precision falls back to two digits.**
Given the baseline,
When the `Product Unit` decimal precision record is deleted,
Then every unit reports one hundredth.

**SCENARIO-007 — Derived sequences.**
Given the baseline,
When the sequences of the delivered units are read,
Then `Units` is one hundred, `Pack of 6` is six hundred, `Dozens` is one thousand, `Minutes` is
one, `in` is two hundred fifty-four, `yd` is three hundred, `ft²` is nine, `in³` is one,
`fl oz (US)` is two, `gal (US)` is four hundred, and every unit whose contained quantity is ten
or more is one thousand.

**SCENARIO-008 — Derived absolute quantities of the configured packagings.**
Given the baseline,
When the absolute quantities of the three created units are read,
Then `Box of 12` is twelve, `Box of 12 Dozens` is one hundred forty-four, and
`Pallet of 40 Boxes` is five thousand seven hundred sixty.

**SCENARIO-009 — Changing a contained quantity recomputes descendants.**
Given the baseline,
When the contained quantity of `Box of 12 Dozens` is changed from twelve to twenty-four,
Then its absolute quantity becomes two hundred eighty-eight and the absolute quantity of
`Pallet of 40 Boxes` becomes eleven thousand five hundred twenty,
And no stored quantity on any document changes.

---

## B. The conversion algorithm

**SCENARIO-010 — Two and a half dozens into units.**
Given the baseline,
When two and one half is converted from `Dozens` to `Units`,
Then the result is thirty, under every one of the five rounding methods.

**SCENARIO-011 — Seven units into dozens, default rounding.**
Given the baseline,
When seven is converted from `Units` to `Dozens` with the default rounding method,
Then the result is nought point five nine.

**SCENARIO-012 — Seven units into dozens, half away from zero.**
Given the baseline,
When seven is converted from `Units` to `Dozens` with half-away-from-zero rounding,
Then the result is nought point five eight.

**SCENARIO-013 — Seven units into dozens, towards zero.**
Given the baseline,
When seven is converted from `Units` to `Dozens` with towards-zero rounding,
Then the result is nought point five eight.

**SCENARIO-014 — Seven units into dozens at other precisions.**
Given the baseline,
When the `Product Unit` precision is set in turn to zero, three and four digits and seven is
converted from `Units` to `Dozens` with the default rounding method,
Then the results are one, nought point five eight four, and nought point five eight three four
respectively.

**SCENARIO-015 — One pound into kilograms.**
Given the baseline,
When one is converted from `lb` to `kg` with the default rounding method,
Then the result is nought point four six.
And with half-away-from-zero rounding the result is nought point four five.
And with towards-zero rounding the result is nought point four five.

**SCENARIO-016 — One pound into kilograms at three digits.**
Given the baseline with the `Product Unit` precision set to three digits,
When one is converted from `lb` to `kg`,
Then the result is nought point four five four under both the default and the half-away-from-zero
methods, and nought point four five three under towards-zero rounding.

**SCENARIO-017 — One hundred grams into kilograms at a rounding of one thousandth.**
Given the baseline with the `Product Unit` precision set to three digits,
When one hundred is converted from `g` to `kg`,
Then the result is nought point one, under every rounding method.

**SCENARIO-018 — One gram into kilograms at the shipped precision.**
Given the baseline,
When one is converted from `g` to `kg`,
Then the result is nought point nought one with the default method, and zero with
half-away-from-zero and with towards-zero rounding.

**SCENARIO-019 — One million twenty thousand grams into tonnes.**
Given the baseline,
When one million twenty thousand is converted from `g` to `Ton`,
Then the result is one and two hundredths.

**SCENARIO-020 — One thousand two hundred thirty-four grams into kilograms.**
Given the baseline,
When one thousand two hundred thirty-four is converted from `g` to `kg` with the default method,
Then the result is one and twenty-four hundredths.

**SCENARIO-021 — A dozen into units is exact.**
Given the baseline,
When one is converted from `Dozens` to `Units`,
Then the result is exactly twelve, not twelve and one hundredth and not thirteen.
(This is the regression guard for a naive reciprocal factor: storing one twelfth as the dozen's
inverse and multiplying would yield twelve and a fraction, which the default rounding would lift
to thirteen at a precision of zero digits.)

**SCENARIO-022 — Conversion to a coarser unit at zero digits rounds up.**
Given the baseline with the `Product Unit` precision set to zero digits and a unit `Score` with
`Units` as reference unit and a contained quantity of twenty,
When two is converted from `Units` to `Score` with the default method,
Then the result is one.

**SCENARIO-023 — Zero converts to zero.**
Given the baseline,
When zero is converted from `Pallet of 40 Boxes` to `Units` with any rounding method,
Then the result is exactly zero and no rounding is performed.

**SCENARIO-024 — A conversion with no destination unit is not rounded.**
Given the baseline,
When seven is converted from `Units` with no destination unit and rounding requested,
Then the result is seven, unrounded.
And when one is converted from `Box of 12 Dozens` with no destination unit,
Then the result is one hundred forty-four.

**SCENARIO-025 — A conversion to the same unit skips the factor arithmetic.**
Given the baseline,
When twenty-two and forty-three hundredths is converted from `Units` to `Units`,
Then the result is twenty-two and forty-three hundredths.
And when twenty-two and four hundred thirty-three thousandths is converted from `Units` to
`Units` with the default method,
Then the result is twenty-two and forty-four hundredths.

**SCENARIO-026 — A cross-tree conversion with failure tolerated returns the input.**
Given the baseline,
When five is converted from `kg` to `Units` with failure tolerated,
Then the result is five, unchanged.

**SCENARIO-027 — A cross-tree conversion with failure raised fails.**
Given the baseline,
When five is converted from `kg` to `Units` with failure raised,
Then the operation fails and no quantity is produced.

**SCENARIO-028 — A round trip is not an identity.**
Given the baseline,
When seven is converted from `Units` to `Dozens` with the default method and the result is
converted back to `Units` with the default method,
Then the final result is seven and eight hundredths.

**SCENARIO-029 — Negative quantities convert symmetrically.**
Given the baseline,
When minus seven is converted from `Units` to `Dozens`,
Then the result is minus nought point five nine with the default method and minus nought point
five eight with half-away-from-zero rounding.

**SCENARIO-030 — The working-time tree reproduces its inexact minute.**
Given the baseline,
When one is converted from `Hours` to `Minutes` with the default method,
Then the result is sixty, the unrounded value being fifty-nine and nine hundred ninety-nine
thousandths and change.
And when ninety is converted from `Minutes` to `Hours` with the default method,
Then the result is **one and fifty-one hundredths**, not one and a half, because the unrounded
value is one and five hundred three-millionths and the default method rounds away from zero.
And with half-away-from-zero rounding the result is one and five tenths.

**SCENARIO-031 — The volume tree.**
Given the baseline,
When one is converted from `gal (US)` to `L` with the default method,
Then the result is three and seventy-nine hundredths, the unrounded value being three and seven
hundred eighty-five thousandths four hundred eight millionths.
And one `L` converted to `ml` is one thousand exactly.

**SCENARIO-032 — The counting ladder in both directions.**
Given the baseline,
When each of the following conversions is performed with the default rounding method,
Then the results are as tabulated:

| Quantity | From | To | Result |
|---|---|---|---|
| 1 | `Units` | `Pack of 6` | 0.17 |
| 1 | `Units` | `Dozens` | 0.09 |
| 1 | `Units` | `Box of 12 Dozens` | 0.01 |
| 1 | `Units` | `Pallet of 40 Boxes` | 0.01 |
| 1 | `Pack of 6` | `Units` | 6 |
| 1 | `Pack of 6` | `Dozens` | 0.5 |
| 1 | `Dozens` | `Pack of 6` | 2 |
| 1 | `Dozens` | `Box of 12 Dozens` | 0.09 |
| 1 | `Box of 12 Dozens` | `Units` | 144 |
| 1 | `Box of 12 Dozens` | `Pack of 6` | 24 |
| 1 | `Box of 12 Dozens` | `Dozens` | 12 |
| 1 | `Box of 12 Dozens` | `Pallet of 40 Boxes` | 0.03 |
| 1 | `Pallet of 40 Boxes` | `Units` | 5760 |
| 1 | `Pallet of 40 Boxes` | `Pack of 6` | 960 |
| 1 | `Pallet of 40 Boxes` | `Dozens` | 480 |
| 1 | `Pallet of 40 Boxes` | `Box of 12 Dozens` | 40 |
| 0.5 | `Pallet of 40 Boxes` | `Units` | 2880 |
| 2.5 | `Dozens` | `Pack of 6` | 5 |

**SCENARIO-033 — The complete matrix.**
Given the baseline,
When every ordered pair of distinct units within each tree is converted for a quantity of one
under the away-from-zero, half-away-from-zero and towards-zero methods,
Then every result matches the corresponding row of the conversion matrix in
[`calculations.md`](calculations.md) section 7.

---

## C. Price conversion

**SCENARIO-040 — Twenty-four per dozen becomes two per unit.**
Given the baseline,
When a price of twenty-four expressed per `Dozens` is converted to a price per `Units`,
Then the result is exactly two.

**SCENARIO-041 — Two per unit becomes twenty-four per dozen.**
Given the baseline,
When a price of two expressed per `Units` is converted to a price per `Dozens`,
Then the result is exactly twenty-four.

**SCENARIO-042 — Price conversion is not rounded.**
Given the baseline,
When a price of twenty-four expressed per `Box of 12 Dozens` is converted to a price per `Units`,
Then the result is one sixth at the full precision the representation allows, and is **not**
rounded to sixteen or seventeen hundredths.

**SCENARIO-043 — Price conversion short-circuits.**
Given the baseline,
When a price of zero is converted between any two units, the result is zero;
And when any price is converted from `Units` to `Units`, the result is that price unchanged;
And when any price is converted with no destination unit, the result is that price unchanged.

**SCENARIO-044 — Price and quantity conversion are inverses.**
Given the baseline and a quantity Q in unit A and a price R per unit A,
When Q is converted to unit B and R is converted to a price per unit B,
Then the product of the two converted values equals Q multiplied by R, up to the rounding applied
to the quantity.
Worked instance: one hundred forty-four `Units` at two each is two hundred eighty-eight; one
`Box of 12 Dozens` at two hundred eighty-eight each is two hundred eighty-eight.

**SCENARIO-045 — A retail price per unit converted to a price per dozen.**
Given the baseline,
When nineteen and ninety-nine hundredths per `Units` is converted to a price per `Dozens`,
Then the result is two hundred thirty-nine and eighty-eight hundredths.

---

## D. Whole-packaging rounding

**SCENARIO-050 — Snapping down to whole boxes.**
Given the baseline,
When one thousand six hundred, expressed in `Units`, is snapped to whole `Box of 12 Dozens` with
towards-zero rounding,
Then the result is one thousand five hundred eighty-four.

**SCENARIO-051 — Snapping with half-away-from-zero and away-from-zero.**
Given the same,
When the rounding method is half away from zero, the result is one thousand five hundred
eighty-four;
And when it is away from zero, the result is one thousand seven hundred twenty-eight.

**SCENARIO-052 — Snapping to a packaging larger than the quantity.**
Given the baseline,
When one thousand six hundred `Units` is snapped to whole `Pallet of 40 Boxes` with towards-zero
rounding,
Then the result is zero.

**SCENARIO-053 — Snapping when the units coincide is an identity.**
Given the baseline,
When twenty-two and forty-three hundredths, expressed in `Units`, is snapped to whole `Units`
with towards-zero rounding,
Then the result is twenty-two and forty-three hundredths, **not** twenty-two.

**SCENARIO-054 — Snapping zero.**
Given the baseline,
When zero is snapped to any packaging with any method,
Then the result is zero.

**SCENARIO-055 — Snapping with the default half-away-from-zero method rounds up past the halfway
point.**
Given the baseline,
When twenty `Units` is snapped to whole `Dozens`,
Then the result is twenty-four with half-away-from-zero rounding and twelve with towards-zero
rounding.

---

## E. Reservation

**SCENARIO-060 — Reservation with a matching unit reserves exactly.**
Given product P with ninety-two units on hand and free, and a delivery move for sixty units whose
unit is `Units`,
When reservation runs,
Then sixty units are reserved.

**SCENARIO-061 — The double conversion protects a move in a packaging unit.**
Given product P with fifty-eight units on hand and free, and a delivery move whose unit is
`Box of 12` demanding five boxes,
When reservation runs with the category policy set to partial,
Then the reserved quantity is fifty-seven and ninety-six hundredths units, because fifty-eight
converts towards zero to four and eighty-three hundredths boxes, which converts back half away
from zero to fifty-seven and ninety-six hundredths units.

**SCENARIO-062 — Full-packaging reservation refuses a partial box.**
Given product P whose category requires full packagings, with fifty-eight units on hand and free,
and a delivery move for five `Box of 12` carrying `Box of 12` as its packaging unit,
When reservation runs,
Then forty-eight units are reserved and ten remain free.

**SCENARIO-063 — Full-packaging reservation with the classic pallet example.**
Given a product whose category requires full packagings, a unit `Pallet of 1000` containing one
thousand `Units`, an order for two pallets, and one thousand six hundred units on hand,
When reservation runs,
Then one thousand units are reserved.
And when the category policy is partial instead,
Then one thousand six hundred units are reserved.

**SCENARIO-064 — Full-packaging reservation on a product with no packaging does not lose a fraction.**
Given a product whose category requires full packagings, whose own unit is `Units`, with
twenty-two and forty-three hundredths units on hand, and a move whose packaging unit is the
product's own unit,
When the reservable quantity is computed for a request of twenty-two and forty-three hundredths,
Then it is twenty-two and forty-three hundredths.

**SCENARIO-065 — A serial-tracked product cannot reserve a fraction.**
Given a serial-tracked product and a computed reservable quantity of nought point nine two after
the double conversion,
When the reservation proceeds,
Then the reserved quantity is zero, because the quantity is not a whole number at the
`Product Unit` precision.

---

## F. Selling in a packaging

**SCENARIO-070 — Five boxes of twelve when stock is kept in units.**
Given product P, a customer order line for five `Box of 12`, and ninety-two units on hand,
When the order is confirmed,
Then a stock move exists with the unit `Units`, a demand of sixty, a real quantity of sixty, a
packaging unit of `Box of 12` and a packaging quantity of five.

**SCENARIO-071 — The same with unit propagation enabled.**
Given the same, with the unit propagation parameter set,
When the order is confirmed,
Then the stock move has the unit `Box of 12`, a demand of five, a real quantity of sixty, a
packaging unit of `Box of 12` and a packaging quantity of five.

**SCENARIO-072 — Picking in units.**
Given the move of SCENARIO-070,
When the operator records sixty in a move line whose unit is `Units` and validates,
Then the move line's quantity in the product's unit is sixty, quantities on hand fall by sixty,
the move's picked quantity is sixty, and the order line's delivered quantity is five.

**SCENARIO-073 — Picking in boxes.**
Given the move of SCENARIO-070,
When the operator changes the move line's unit to `Box of 12`, records five and validates,
Then the move line's quantity in the product's unit is sixty, quantities on hand fall by sixty,
the move's picked quantity — expressed in the move's unit of `Units` — is sixty, and the order
line's delivered quantity is five.

**SCENARIO-074 — Invoicing in the order's unit.**
Given the delivered order of SCENARIO-072,
When an invoice is created,
Then the invoice line's unit is `Box of 12` and its quantity is five,
And after posting, the order line's invoiced quantity is five.

**SCENARIO-075 — The price is converted to the line's unit.**
Given product P with a catalogue price of two per `Units`, and an order line in `Box of 12`,
When the line's unit is set,
Then the unit price becomes twenty-four and the line subtotal for a quantity of five is one
hundred twenty.

**SCENARIO-076 — The price list rule is matched on the product's own unit.**
Given product P with a catalogue price of fifty per `Units` and a price list rule granting ten
per cent for a minimum quantity of twenty-four,
When an order line requests three `Dozens`,
Then the rule matches, because three dozens converts to thirty-six units,
And the unit price is five hundred forty per dozen,
And the line total is one thousand six hundred twenty.

**SCENARIO-077 — A unit outside the allowed list is refused on a sales line.**
Given product P whose additional units do not include `kg`,
When a user attempts to set `kg` on a sales order line for P through the user interface,
Then the unit is not offered and the selection is refused.

**SCENARIO-078 — Two cart lines in different units are not merged.**
Given a storefront visitor,
When the visitor adds one `Box of 12` of P and then one `Units` of P to the cart,
Then two separate cart lines exist.

**SCENARIO-079 — An unavailable storefront unit is refused.**
Given product P and a unit `Ton` that is not among P's available units,
When a cart addition names `Ton`,
Then the request fails with the message `This product is not available (anymore) in this unit of measure.`

---

## G. Buying in a vendor's unit

**SCENARIO-080 — Ordering three boxes from a vendor quoting per box.**
Given product P whose own unit is `Units`, and a vendor price list line quoting one hundred
twenty per `Box of 12 Dozens` with a five per cent discount and a minimum quantity of two,
When a purchase line is created for three `Box of 12 Dozens`,
Then the vendor line is selected, the unit price is one hundred twenty, the discount is five per
cent, the total quantity is four hundred thirty-two, and the unit price in the product's unit is
five sixths.

**SCENARIO-081 — Ordering the same physical quantity in units.**
Given the same,
When the purchase line is instead created for four hundred thirty-two `Units`,
Then the vendor line is still selected, because four hundred thirty-two units converts to three
boxes,
And the unit price is five sixths per unit.

**SCENARIO-082 — The receipt move.**
Given the confirmed order of SCENARIO-080 and the propagation parameter unset,
When the receipt is created,
Then the move's unit is `Units`, its demand is four hundred thirty-two, its packaging unit is
`Box of 12 Dozens` and its packaging quantity is three.

**SCENARIO-083 — A partial receipt reports back in the line's unit.**
Given the receipt of SCENARIO-082,
When four hundred units are received and validated,
Then the purchase line's received quantity is two and seventy-eight hundredths boxes.

**SCENARIO-084 — The vendor unit is allowed on a purchase line even when the product does not list it.**
Given a product whose additional units are empty and a vendor quoting in `Box of 12 Dozens`,
When a purchase line for that product is edited,
Then `Box of 12 Dozens` is offered in the unit selector.

**SCENARIO-085 — The discounted vendor price.**
Given the vendor price list line of SCENARIO-080,
When its discounted price is read,
Then it is one hundred twenty divided by one hundred forty-four, multiplied by nought point nine
five, that is nought point seven nine one six six six and so on per unit, unrounded.

---

## H. Manufacturing

**SCENARIO-090 — A bill of materials with mixed units.**
Given a bill yielding one `Box of 12 Dozens` of a finished product from three `kg` of a material
whose own unit is `g`, and one `Units` of a fastener,
When a production order is created for two and one half `Box of 12 Dozens`,
Then the scaling factor is two and one half,
And the material raw move demands seven and one half `kg` with a real quantity of seven thousand
five hundred grams,
And the fastener raw move demands two and one half `Units`.

**SCENARIO-091 — The scaling factor is unrounded.**
Given a bill yielding three `Units` from one `Units` of a component,
When a production order is created for one `Units`,
Then the scaling factor is one third, unrounded,
And the component demand is one third rounded away from zero at two digits, that is nought point
three four.

**SCENARIO-092 — The component demand rounds away from zero.**
Given the same bill with seven `Units` of the component,
When a production order is created for one `Units`,
Then the component demand is seven multiplied by one third, rounded away from zero, that is two
and thirty-four hundredths.

**SCENARIO-093 — The produced quantity converts half away from zero.**
Given a production order in `Box of 12 Dozens` for a product whose own unit is `Units`,
When a producing quantity of two and one half is recorded,
Then the finished move's quantity in the product's unit is three hundred sixty.

---

## I. Validations and error messages

**SCENARIO-100 — A contained quantity of zero is refused.**
Given the baseline,
When a unit is saved with a contained quantity of zero,
Then the save fails with `The conversion ratio for a unit of measure cannot be 0!`

**SCENARIO-101 — A root unit whose contained quantity is not one is refused.**
Given the baseline,
When a unit is saved with no reference unit and a contained quantity of five,
Then the save fails with `Reference unit of measure is missing.`

**SCENARIO-102 — A protected unit cannot be deleted.**
Given the baseline,
When a deletion of `Units` is attempted,
Then the deletion fails with a message beginning `The following units of measure are used by the system and cannot be deleted: ` followed by `Units`, then a newline, then `You can archive them instead.`

**SCENARIO-103 — The dozen and the pack of six can be deleted.**
Given a fresh installation with no document referencing them,
When a deletion of `Dozens` and `Pack of 6` is attempted,
Then the deletion succeeds.

**SCENARIO-104 — The working hour can be deleted unless time recording is installed.**
Given a fresh installation without the time-recording capability,
When a deletion of `Hours` is attempted, the deletion succeeds.
And given the same installation with the time-recording capability,
When the deletion is attempted, it fails with the protection message naming `Hours`.

**SCENARIO-105 — Editing a protected unit's contained quantity warns.**
Given a protected unit created more than one day ago,
When its contained quantity is edited,
Then a non-blocking warning is raised whose title is `Warning for ` followed by the unit's name
and whose body is the four-paragraph text of [`business-rules.md`](business-rules.md).

**SCENARIO-106 — Editing a user-created packaging does not warn.**
Given `Box of 12 Dozens`, created by a user,
When its contained quantity is edited,
Then no warning is raised.

**SCENARIO-107 — A duplicate packaging barcode is refused.**
Given a Product Unit Barcode row with the barcode `5410013101234`,
When a second row is saved with the same barcode,
Then the save fails with `A barcode can only be assigned to one packaging.`

**SCENARIO-108 — A packaging barcode colliding with a product barcode is refused.**
Given a product variant with the barcode `5410013101234`,
When a Product Unit Barcode row is saved with the same barcode,
Then the save fails with `A product already uses the barcode`

**SCENARIO-109 — A product barcode colliding with a packaging barcode is refused.**
Given a Product Unit Barcode row with the barcode `5410013101234` in company A,
When a product variant of company A is saved with the same barcode,
Then the save fails with `A packaging already uses the barcode`

**SCENARIO-110 — A duplicate package type barcode is refused.**
Given a package type with the barcode `PT-001`,
When a second package type is saved with the same barcode,
Then the save fails with `A barcode can only be assigned to one package type!`

**SCENARIO-111 — Negative package dimensions are refused.**
Given the baseline,
When a package type is saved with a height of minus one,
Then the save fails with `Height must be positive`
And the same holds for width with `Width must be positive`, for length with `Length must be positive`, and for the maximum weight with `Max Weight must be positive`.

**SCENARIO-112 — Zero package dimensions are accepted.**
Given the baseline,
When a package type is saved with a height, width, length and maximum weight all zero,
Then the save succeeds.

**SCENARIO-113 — A completed move's unit cannot be changed.**
Given a completed stock move,
When its unit is written,
Then the write fails with `You cannot change the UoM for a stock move that has been set to 'Done'.`
And the abbreviation in that message stands for *unit of measure*; the message is reproduced
exactly as the system produces it.

**SCENARIO-114 — The real quantity of a move cannot be written.**
Given any stock move,
When its real quantity field is written directly,
Then the write fails with the programming-error message naming both storage names.

**SCENARIO-115 — An off-grid picked quantity blocks validation.**
Given a transfer whose move has a picked quantity of one and two hundred thirty-four
thousandths, recorded while the precision was three digits, and the precision then reduced to two
digits,
When the transfer is validated,
Then the validation fails with a paragraph beginning with a blank line, then
`The quantity done for the product ` followed by the product's display name, then ` doesn't respect the rounding precision defined on the system.`, then a newline, then `Please change the quantity done or the rounding precision in your settings.`

**SCENARIO-116 — A serial-tracked move line may not resolve to other than one.**
Given a serial-tracked product whose own unit is `Units`,
When a move line for it is edited to a quantity resolving to two in the product's unit,
Then the edit fails with `You can only process 1.0 ` followed by the product's own unit's name,
then ` of products with unique serial number.`

**SCENARIO-117 — A counter-sale transfer refuses a conversion that rounds to zero.**
Given a counter sale with a move for one `g` of a product whose own unit is `Ton`,
When the transfer is prepared,
Then it fails with the conversion error whose first line is
`Conversion Error: The following unit of measure conversions result in a zero quantity due to rounding:`
followed by one line reading ` - From "g" to "Ton"` and then the explanatory closing paragraph.

**SCENARIO-118 — A duplicate decimal precision usage is refused.**
Given the baseline,
When a second decimal precision named `Product Unit` is saved,
Then the save fails with `Only one value can be defined for each given usage!`

**SCENARIO-119 — Reducing a decimal precision warns.**
Given the `Product Unit` precision at two digits,
When it is edited to one digit,
Then a non-blocking warning is raised with the title `Warning for Product Unit` and the
five-paragraph text of [`business-rules.md`](business-rules.md).

---

## J. Changing a product's own unit

**SCENARIO-130 — Relabelling when every document uses the product's own unit.**
Given product Q whose own unit is `Units`, with one sales order line for ten `Units` and one
stock move for ten `Units`,
When the product's own unit is changed to `kg`,
Then the save succeeds,
And the sales order line reads ten `kg`,
And the stock move reads ten `kg`,
And no quantity anywhere has changed.

**SCENARIO-131 — The change is refused when a document uses another unit.**
Given product P with one sales order line for five `Box of 12`,
When the product's own unit is changed to `kg`,
Then the save fails with a message beginning `As other units of measure (ex : ` followed by the
display name of `Box of 12`, then `) than ` followed by the display name of `Units`, then
` have already been used for this product, the change of unit of measure can not be done.If you want to change it, please archive the product and create a new one.`
And the missing space before `If` is reproduced exactly.

**SCENARIO-132 — The change is refused when a posted document uses another unit.**
Given product P with a posted invoice line in `Box of 12`,
When the product's own unit is changed,
Then the save fails with
`This product is already being used in posted Journal Entries.` followed by a newline and
`If you want to change its Unit of Measure, please archive this product and create a new one.`

**SCENARIO-133 — A warning precedes the change when history exists.**
Given product Q with at least one stock move,
When the own unit is changed in the user interface,
Then a non-blocking warning titled `What to expect ?` is shown, whose body names the old and new
unit display names and states that existing records will be updated by replacing the unit name.

**SCENARIO-134 — No warning when no history exists.**
Given a freshly created product with no document,
When its own unit is changed,
Then no warning is shown.

---

## K. Security and visibility

**SCENARIO-140 — An internal user may read but not write units.**
Given a user holding only the internal-user group,
When the user reads the unit list, the read succeeds;
When the user attempts to create, edit or delete a unit, the attempt is refused.

**SCENARIO-141 — A settings administrator may manage units.**
Given a user holding the settings-administrator group,
When the user creates, edits and deletes an unprotected unit,
Then all three succeed.

**SCENARIO-142 — Units are visible across companies.**
Given two companies and a unit created while company A was active,
When a user whose active company is B lists the units,
Then the unit is visible.

**SCENARIO-143 — The feature group hides controls and changes no data.**
Given the baseline and an order line for five `Box of 12`,
When the multiple-units feature group is removed from the reader,
Then the unit column disappears from the line,
And the line's stored unit and quantity are unchanged,
And confirming the order still produces a move for sixty units.

**SCENARIO-144 — The package content description omits the unit without the group.**
Given a package holding five `Box of 12` of product P,
When the description is read by a user holding the feature group, it reads the quantity, the
unit name and the product name;
When it is read by a user without the group, it reads the quantity and the product name only.

---

## L. Rounding function conformance

**SCENARIO-150 — The compensation term repairs a representation error.**
Given the rounding function,
When two and six hundred seventy-five thousandths is rounded to two digits half away from zero,
Then the result is two and sixty-eight hundredths, not two and sixty-seven hundredths.

**SCENARIO-151 — Half to even.**
Given the rounding function at zero digits,
When nought point five, one and one half, and two and one half are rounded half to even,
Then the results are zero, two and two.

**SCENARIO-152 — Half towards zero.**
Given the rounding function at zero digits,
When minus nought point five is rounded half towards zero, the result is zero;
And when it is rounded half away from zero, the result is minus one.

**SCENARIO-153 — Arbitrary steps.**
Given the rounding function,
When one and three tenths is rounded onto a step of one half half away from zero,
Then the result is one and one half.

**SCENARIO-154 — Noise below the compensation term is absorbed.**
Given the rounding function at two digits with the away-from-zero method,
When one plus one quadrillionth is rounded,
Then the result is one, not one and one hundredth.

**SCENARIO-155 — A genuine excess is not absorbed.**
Given the same,
When one plus one ten-billionth is rounded,
Then the result is one and one hundredth.

**SCENARIO-156 — The comparison rounds before subtracting.**
Given the comparison operation at two digits,
When six thousandths is compared with two thousandths,
Then the result is plus one, because they round to one hundredth and zero.

**SCENARIO-157 — The zero test rounds after subtracting.**
Given the zero test at two digits,
When the difference of six thousandths and two thousandths, that is four thousandths, is tested,
Then the result is true.

**SCENARIO-158 — Euclidean division at a precision.**
Given the euclidean division at two digits,
When eight is divided by one and six tenths,
Then the quotient is five and the remainder is zero — and specifically **not** a remainder of one
and five thousand nine hundred ninety-nine ten-thousandths.

---

## M. Configuration and installation

**SCENARIO-170 — Installation order.**
Given an empty installation,
When the domain's reference data is loaded in the order of
[`configuration.md`](configuration.md) section 8,
Then every unit's absolute quantity matches the table in section 2.1 of that file.

**SCENARIO-171 — Sixteen delivered units are archived.**
Given a fresh installation of the units capability alone,
When the archived units are listed,
Then they are the dozen, the centimetre, the kilometre, the inch, the foot, the yard, the mile,
the square foot, the cubic metre, the fluid ounce, the quart, the gallon, the cubic inch, the
cubic foot, the ounce and the pound.

**SCENARIO-172 — Expense recording reactivates the kilometre.**
Given a fresh installation,
When the expense capability is installed,
Then the kilometre's archival flag is yes.

**SCENARIO-173 — The United States accounting localisation reactivates six units.**
Given a fresh installation,
When that localisation is installed,
Then the inch, the foot, the square foot, the ounce, the pound and the gallon have an archival
flag of yes.

**SCENARIO-174 — The package type sequence is created from a prefix.**
Given the baseline,
When a package type named `Euro pallet` is saved with the sequence prefix `PAL/`,
Then a numbering sequence exists with the name `Package Type Sequence PAL/`, the code `PAL/`, the
prefix `PAL/` and a padding of seven,
And the first package made from that type is named `PAL/0000001`.

**SCENARIO-175 — Renaming the prefix renames the sequence.**
Given the package type of SCENARIO-174,
When its sequence prefix is changed to `EUR/`,
Then the existing sequence's name becomes `Package Type Sequence EUR/` and its prefix becomes
`EUR/`.

**SCENARIO-176 — The weight parameter selects the weight unit.**
Given the baseline,
When the weight system parameter is set to one,
Then the weight unit label on products and package types is the pound's display name;
When it is unset,
Then it is the kilogram's display name.

**SCENARIO-177 — The volume parameter selects both the volume and the length unit.**
Given the baseline,
When the volume system parameter is set to one,
Then the volume unit label is the cubic foot's display name and the length unit label is the
foot's display name;
When it is unset,
Then they are the cubic metre's and the millimetre's display names.

**SCENARIO-178 — The domain has no scheduled job.**
Given a fresh installation,
When the scheduled jobs are listed,
Then none belongs to this domain.

**SCENARIO-179 — The domain posts nothing.**
Given the baseline,
When a unit is created, edited, archived and deleted, a packaging barcode is created and
deleted, a package type is created and deleted, and the `Product Unit` precision is changed,
Then no journal entry and no journal item is created, modified or deleted.

---

## N. Exchange formats

**SCENARIO-190 — The trade code of a delivered unit.**
Given the accounting capability,
When the trade code of each delivered unit is resolved,
Then it matches the table in [`configuration.md`](configuration.md) section 2.5.

**SCENARIO-191 — The square metre falls back to the plain-piece code.**
Given the accounting capability,
When the trade code of the delivered square metre is resolved,
Then it is the plain-piece code, because the mapping's entry uses a different external
identifier,
And the same holds for the delivered square foot.

**SCENARIO-192 — A user-created unit falls back to the plain-piece code.**
Given `Box of 12 Dozens`,
When its trade code is resolved,
Then it is the plain-piece code.

**SCENARIO-193 — An unknown incoming trade code becomes the counting unit.**
Given an incoming exchanged invoice line whose trade code is not in the mapping,
When the unit is resolved,
Then it is the counting unit.

**SCENARIO-194 — An incoming unit from a different tree is discarded.**
Given an incoming line whose trade code resolves to the kilogram, matched to a product whose own
unit is `Units`,
When the line is created,
Then its unit is `Units` and its quantity is the number that was on the incoming document,
unconverted.

---

## O. Multi-company and archival

**SCENARIO-200 — Archiving a unit does not break existing documents.**
Given an open order line in `Box of 12 Dozens`,
When `Box of 12 Dozens` is archived,
Then the line still reads five `Box of 12 Dozens`,
And confirming the order still produces a move for seven hundred twenty units,
And the unit no longer appears in selectors.

**SCENARIO-201 — Reactivating restores selectability.**
Given the archived unit of SCENARIO-200,
When its archival flag is set,
Then it appears again in selectors.

**SCENARIO-202 — A unit used on a document cannot be deleted even if unprotected.**
Given `Box of 12 Dozens` referenced by a sales order line,
When a deletion is attempted,
Then it is refused by the referential restriction on the line's unit.

**SCENARIO-203 — Deleting a unit cascades to descendants.**
Given `Box of 12 Dozens` with the descendant `Pallet of 40 Boxes`, neither referenced by any
document,
When `Box of 12 Dozens` is deleted,
Then `Pallet of 40 Boxes` is deleted too, together with every packaging barcode bound to either.

---

## P. End-to-end equivalence checks

**SCENARIO-210 — The physical-quantity invariant across the sale path.**
Given the five-boxes-of-twelve scenario,
When the physical quantity is computed at each of the eleven states of the quantity lifecycle,
Then every value is sixty root units exactly.

**SCENARIO-211 — The physical-quantity invariant with an awkward quantity.**
Given an order line for seven `Units` on a product P, confirmed and delivered,
When the physical quantity is computed at each state,
Then every value is seven root units, and the only figure that differs is the packaging quantity,
which reads nought point five nine `Dozens` when the packaging unit is the dozen — that is seven
and eight hundredths root units, within the bound of one hundredth of a dozen.

**SCENARIO-212 — Changing the precision mid-flight.**
Given an open transfer whose move line quantity is one and two hundred thirty-four thousandths,
When the `Product Unit` precision is reduced from three digits to two and the transfer is
validated,
Then validation fails with the message of SCENARIO-115,
And when the precision is restored to three digits,
Then validation succeeds.

**SCENARIO-213 — Changing a packaging's contained quantity mid-flight.**
Given an open order line for five `Box of 12 Dozens`, confirmed, with a move demanding seven
hundred twenty units,
When the contained quantity of `Box of 12 Dozens` is changed from twelve to twenty-four,
Then the move's demand is still seven hundred twenty,
And the move's packaging quantity, being derived, becomes two and fifty hundredths,
And the order line still reads five `Box of 12 Dozens`, now meaning one thousand four hundred
forty items.

**SCENARIO-214 — The audit bound.**
Given any two consecutive states of a quantity in the lifecycle of
[`state-machines.md`](state-machines.md) section 3,
When the physical quantities of the two states are compared,
Then they differ by at most one step of the `Product Unit` precision multiplied by the absolute
quantity of the unit that was rounded at that crossing.
