# Units of Measure and Packaging — Configuration

Everything an administrator can set, everything the system ships with, and everything a rebuild
must create at installation time.

---

## 1. Delivered reference data: the decimal precision

One decimal precision record is delivered by the units capability, and it is delivered with the
flag that forces its creation even where a record of the same name would otherwise be skipped.

| External identifier | Usage (`name`) | Digits (`digits`) |
|---|---|---|
| `uom.decimal_product_uom` | `Product Unit` | 2 |

This single record is the rounding precision of every unit of measure in the system. Changing it
changes every conversion at once; see [`calculations.md`](calculations.md) section 15.

Other decimal precisions referenced by fields in this domain but owned elsewhere:

| Usage | Used by | Owning domain |
|---|---|---|
| `Product Price` | The minimum display precision of unit prices on order and bill lines | [`../pricing-and-pricelists/`](../pricing-and-pricelists/) |
| `Stock Weight` | The stored precision of a product's weight and of a transfer's shipping weight | [`../inventory-operations/`](../inventory-operations/) |
| `Volume` | The stored precision of a product's volume | [`../products-and-catalog/`](../products-and-catalog/) |
| `Discount` | The stored precision of a discount percentage | [`../pricing-and-pricelists/`](../pricing-and-pricelists/) |

---

## 2. Delivered reference data: the units of measure

All thirty records are delivered as **non-updatable** reference data: a later upgrade of the
capability does not overwrite a value an administrator has changed.

### 2.1 The full table

| External identifier | Name (`name`) | Tree | Reference unit (`relative_uom_id`) | Contains (`relative_factor`) | Absolute quantity (`factor`) | Sequence (`sequence`) | Active on delivery | Rounding precision (`rounding`) |
|---|---|---|---|---|---|---|---|---|
| `uom.product_uom_unit` | `Units` | Counting | — (root) | 1 | 1 | 100 | yes | 0.01 |
| `uom.product_uom_pack_6` | `Pack of 6` | Counting | `Units` | 6 | 6 | 600 | yes | 0.01 |
| `uom.product_uom_dozen` | `Dozens` | Counting | `Units` | 12 | 12 | 1000 | no (archived) | 0.01 |
| `uom.product_uom_hour` | `Hours` | Working time | — (root) | 1 | 1 | 100 | yes | 0.01 |
| `uom.product_uom_day` | `Days` | Working time | `Hours` | 8 | 8 | 800 | yes | 0.01 |
| `uom.product_uom_minute` | `Minutes` | Working time | `Hours` | 0.0166667 | 0.0166667 | 1 | yes | 0.01 |
| `uom.product_uom_millimeter` | `mm` | Length | — (root) | 1 | 1 | 100 | yes | 0.01 |
| `uom.product_uom_cm` | `cm` | Length | `mm` | 10 | 10 | 1000 | no (archived) | 0.01 |
| `uom.product_uom_meter` | `m` | Length | `cm` | 100 | 1000 | 1000 | yes | 0.01 |
| `uom.product_uom_km` | `km` | Length | `m` | 1000 | 1000000 | 1000 | no (archived) | 0.01 |
| `uom.product_uom_inch` | `in` | Length | `cm` | 2.54 | 25.4 | 254 | no (archived) | 0.01 |
| `uom.product_uom_foot` | `ft` | Length | `in` | 12 | 304.79999999999995 | 1000 | no (archived) | 0.01 |
| `uom.product_uom_yard` | `yd` | Length | `ft` | 3 | 914.3999999999999 | 300 | no (archived) | 0.01 |
| `uom.product_uom_mile` | `mi` | Length | `yd` | 1760 | 1609343.9999999998 | 1000 | no (archived) | 0.01 |
| `uom.product_uom_square_meter` | `m²` | Surface | — (root) | 1 | 1 | 100 | yes | 0.01 |
| `uom.product_uom_square_foot` | `ft²` | Surface | `m²` | 0.092903 | 0.092903 | 9 | no (archived) | 0.01 |
| `uom.product_uom_milliliter` | `ml` | Volume | — (root) | 1 | 1 | 100 | yes | 0.01 |
| `uom.product_uom_litre` | `L` | Volume | `ml` | 1000 | 1000 | 1000 | yes | 0.01 |
| `uom.product_uom_cubic_meter` | `m³` | Volume | `L` | 1000 | 1000000 | 1000 | no (archived) | 0.01 |
| `uom.product_uom_floz` | `fl oz (US)` | Volume | `L` | 0.0295735 | 29.5735 | 2 | no (archived) | 0.01 |
| `uom.product_uom_qt` | `qt (US)` | Volume | `fl oz (US)` | 32 | 946.352 | 1000 | no (archived) | 0.01 |
| `uom.product_uom_gal` | `gal (US)` | Volume | `qt (US)` | 4 | 3785.408 | 400 | no (archived) | 0.01 |
| `uom.product_uom_cubic_inch` | `in³` | Volume | `L` | 0.0163871 | 16.3871 | 1 | no (archived) | 0.01 |
| `uom.product_uom_cubic_foot` | `ft³` | Volume | `in³` | 1728 | 28316.9088 | 1000 | no (archived) | 0.01 |
| `uom.product_uom_gram` | `g` | Mass | — (root) | 1 | 1 | 100 | yes | 0.01 |
| `uom.product_uom_kgm` | `kg` | Mass | `g` | 1000 | 1000 | 1000 | yes | 0.01 |
| `uom.product_uom_ton` | `Ton` | Mass | `kg` | 1000 | 1000000 | 1000 | yes | 0.01 |
| `uom.product_uom_oz` | `oz` | Mass | `g` | 28.3495 | 28.3495 | 1000 | no (archived) | 0.01 |
| `uom.product_uom_lb` | `lb` | Mass | `oz` | 16 | 453.592 | 1000 | no (archived) | 0.01 |
| `uom.product_uom_kwh` | `KWH` | Energy | — (root) | 1 | 1 | 100 | yes | 0.01 |

The names in the table are the delivered, untranslated names. `Ton` is the metric tonne of one
thousand kilograms. The volume names carry a parenthesised country marker, the two letters `US`,
which abbreviate United States; the marker is part of the delivered name and is reproduced
exactly, because the fluid ounce, the quart and the gallon differ between customary systems and
the delivered values are the United States ones. The energy unit's delivered name, `KWH`,
abbreviates kilowatt hour and is likewise reproduced exactly.

### 2.2 Which units are active on delivery

Sixteen of the thirty are archived when the units capability is installed alone. Two further
capabilities reactivate some of them.

| Capability installed | Units it reactivates |
|---|---|
| Expense recording | `km` |
| United States accounting localisation | `in`, `ft`, `ft²`, `oz`, `lb`, `gal (US)` |

Nothing reactivates the dozen, the centimetre, the kilometre (except expenses), the yard, the
mile, the cubic metre, the fluid ounce, the quart or the cubic inch and cubic foot. A business
that needs them sets the archival flag by hand.

**The dozen is archived on delivery.** Any scenario in this specification that uses dozens
presumes the administrator has reactivated it. The test fixtures of the system do exactly that.

### 2.3 The tree structure

| Tree | Root | Members |
|---|---|---|
| Counting | `Units` | `Units`, `Pack of 6`, `Dozens` |
| Working time | `Hours` | `Hours`, `Days`, `Minutes` |
| Length | `mm` | `mm`, `cm`, `m`, `km`, `in`, `ft`, `yd`, `mi` |
| Surface | `m²` | `m²`, `ft²` |
| Volume | `ml` | `ml`, `L`, `m³`, `fl oz (US)`, `qt (US)`, `gal (US)`, `in³`, `ft³` |
| Mass | `g` | `g`, `kg`, `Ton`, `oz`, `lb` |
| Energy | `KWH` | `KWH` |

### 2.4 Units added by country localisations

Several country capabilities add further units. They are listed because a rebuild that installs
the same capability must create the same rows, and because two of them sit oddly in their tree.

| Capability | Name | Reference unit | Contains | Active |
|---|---|---|---|---|
| One North American country's localisation | `Activity` | — (root) | 1 | yes |
| The same | `Job` | — (root) | 1 | yes |
| The same | `Service Unit` | — (root) | 1 | yes |
| One Eurasian country's electronic invoicing | `Year` | `Days` | 365 | no |
| The same | `Month` | `Days` | 30 | no |
| The same | `Minute` | `Hours` | 0.0166667 | no |
| The same | `Second` | its own `Minute` | 0.0166667 | no |
| The same | `Pack` | `Units` | 1 | no |
| The same | `Box` | `Units` | 1 | no |
| The same | `Crate` | `Units` | 1 | no |
| The same | `Parcel` | `Units` | 1 | no |
| The same | `Package` | `Units` | 1 | no |
| The same | `Pallet` | `Units` | 1 | no |
| The same | `Bags` | `Units` | 1 | no |
| The same | `Set` | `Units` | 1 | no |
| The same | `Pair` | `Units` | 2 | no |
| The same | `Gross Ton` | `kg` | 1000 | no |
| The same | `Milligram - mg` | `g` | 0.001 | no |
| The same | `Milliliter - ml` | `ml` | 1 | yes |
| The same | `Cubic Centimeter - cm³` | `L` | 0.001 | no |
| The same | `Cubic Millimeter - mm³` | `ml` | 0.000001 | no |
| The same | `Square Centimeter - cm²` | `cm` | 1000 | no |
| The same | `Standard Cubic Meter` | `L` | 1 | no |
| The same | `Kilowatt` | — (root) | 1 | no |
| The same | `Kilowatt Hour` | `Units` | 1 | no |
| The same | `Megawatt Hour` | `Units` | 1 | no |

Two anomalies a rebuild should reproduce rather than correct, because external documents depend
on the identifiers:

- the localisation's `Square Centimeter - cm²` is placed in the **length** tree with the
  centimetre as its reference unit, so it converts against lengths, not against surfaces;
- the localisation's `Kilowatt Hour` and `Megawatt Hour` are placed in the **counting** tree with
  the counting unit as reference and a contained quantity of one, so both are numerically
  identical to a plain unit and neither relates to the delivered energy root.

The packaging-shaped units of that localisation all contain exactly one, so they are labels
rather than quantities: choosing `Pallet` there does not multiply anything.

### 2.5 Standard trade codes on the delivered units

Where the accounting capability is installed, each unit maps to a standard international trade
code used on exchanged documents. The mapping is by external identifier, and any unit not in the
mapping falls back to the code for a plain piece.

| Unit | Trade code |
|---|---|
| `Units` | `C62` |
| `Dozens` | `DZN` |
| `kg` | `KGM` |
| `g` | `GRM` |
| `Ton` | `TNE` |
| `Days` | `DAY` |
| `Hours` | `HUR` |
| `Minutes` | `MIN` |
| `m` | `MTR` |
| `km` | `KMT` |
| `cm` | `CMT` |
| `mm` | `MMT` |
| `L` | `LTR` |
| `lb` | `LBR` |
| `oz` | `ONZ` |
| `in` | `INH` |
| `ft` | `FOT` |
| `yd` | `YRD` |
| `mi` | `SMI` |
| `fl oz (US)` | `OZA` |
| `qt (US)` | `QTL` |
| `gal (US)` | `GLL` |
| `m³` | `MTQ` |
| `in³` | `INQ` |
| `ft³` | `FTQ` |
| `KWH` | `KWH` |
| anything else | `C62` |

**A defect worth reproducing verbatim.** The mapping also contains entries for a square metre and
a square foot, but under external identifiers that do not match the delivered ones. The delivered
square metre and square foot therefore fall through to the plain-piece code. A rebuild that
"fixes" the identifiers will emit different codes on exchanged documents from the system it
replaces. Reproduce the mapping as given, and treat the correction as a deliberate, documented
deviation if it is made.

The reverse mapping, from a trade code to a unit, is the inverse of the same table, with an
unknown code resolving to the counting unit.

### 2.6 National unit codes on the delivered units

Each country capability adds one annotation field and fills it for the units it recognises.

| Country capability | Field (storage name) | Type | Examples |
|---|---|---|---|
| One South American country | `l10n_ar_afip_code` | Text | Kilogram `01`, metre `02`, litre `05`, counting unit `07`, dozen `09`, gram `14`, kilometre `17`, centimetre `20`, tonne `29`, everything else recognised `98` |
| Another South American country | `l10n_cl_sii_code` | Text | Counting unit `10`, dozen `11`, foot `13`, metre `14`, kilogram `6`, litre `9` |
| One South Asian country | `l10n_in_code` | Text | Counting unit `UNT-UNITS`, dozen `DOZ-DOZENS`, kilogram `KGS-KILOGRAMS`, gram `GMS-GRAMMES`, tonne `TON-TONNES`, litre `LTR-LITRES`, millilitre `MLT-MILILITRE`, metre `MTR-METERS`, centimetre `CMS-CENTIMETERS`, kilometre `KME-KILOMETRE`, yard `YDS-YARDS`, square metre `SQM-SQUARE METERS`, square foot `SQF-SQUARE FEET`, cubic metre `CBM-CUBIC METERS`, United States gallon `UGS-US GALLONS`, everything else `OTH-OTHERS` |
| One Central European country | `l10n_hu_edi_code` | Selection | A closed list of national codes |
| One South European country | `l10n_es_edi_facturae_uom_code` | Selection | A closed list of national codes |
| One North African country | `l10n_eg_unit_code_id` | Link to a code record | A separate entity holding a name and a code |
| One South East Asian country | `l10n_id_uom_code` | Link to a code record | A separate entity holding the national codes |

A field of this kind is revealed on the unit form only when one of the reader's active companies
has the matching fiscal country, which is what the computed fiscal-country-codes field on the
unit is for.

### 2.7 The timesheet input control annotation

Where the time-recording capability is installed, one further annotation exists on the unit.

| Unit | `timesheet_widget` | Meaning |
|---|---|---|
| `Hours` | `float_time` | Time is entered as hours and minutes on a clock-style control. |
| `Days` | `float_toggle` | Time is entered by stepping through a small set of day fractions. |

---

## 3. Security groups

### 3.1 The multiple-units feature group

| Property | Value |
|---|---|
| External identifier | `uom.group_uom` |
| Name | `Manage Multiple Units of Measure` |
| Kind | Feature flag |
| Access rights carried | none |
| Implied by | The settings toggle described in section 5 |

Everything the group reveals is listed in [`business-rules.md`](business-rules.md) under the
security rules. In short: unit selectors, the additional-units list on a product, the packaging
reservation policy on a category, and the unit name inside a package's content description.

### 3.2 Access rights on the unit entity

| Rule identifier | Entity | Group | Create | Read | Update | Delete |
|---|---|---|---|---|---|---|
| `uom.access_uom_uom_manager` | `uom.uom` | Settings administrators | yes | yes | yes | yes |
| `uom.access_uom_uom_user` | `uom.uom` | Internal users | no | yes | no | no |

There are **no record rules** on the unit entity: every reader who may read one may read all of
them, in every company.

### 3.3 Access rights on the other entities of the domain

| Entity | Read | Create / Update / Delete |
|---|---|---|
| `product.uom` (Product Unit Barcode) | Catalogue readers | Catalogue managers |
| `stock.package.type` | Warehouse users | Warehouse managers |
| `decimal.precision` | Internal users | Settings administrators |

Package types carry a company field and therefore participate in the standard multi-company
record rule: a reader sees types whose company is empty or is one of the reader's active
companies.

---

## 4. System parameters

| Key | Values | Default when absent | Effect |
|---|---|---|---|
| `product.weight_in_lbs` | `1`, or anything else | absent | `1` makes the dimensionless weight numbers on products and package types read as pounds; anything else makes them read as kilograms. |
| `product.volume_in_cubic_feet` | `1`, or anything else | absent | `1` makes volumes read as cubic feet **and** lengths read as feet; anything else makes volumes read as cubic metres and lengths as millimetres. |
| `stock.propagate_uom` | `1`, or anything else | absent | `1` carries the document line's unit onto the generated stock move; anything else creates the move in the product's own unit and converts the quantity half away from zero. |

These three parameters are global to the installation, not per company. A rebuild must read them
with elevated rights, because ordinary users may not read system parameters.

### 4.1 Interaction of the weight and volume parameters

The two parameters are independent, which permits an inconsistent configuration: weights in
kilograms with lengths in feet, or weights in pounds with volumes in cubic metres. The system
does not object. A rebuild should not add a cross-check, because existing installations rely on
setting only one of the two.

### 4.2 Interaction of the propagation parameter with packaging

When propagation is off (the default), a move created from a line in a packaging unit is created
in the product's own unit. Its **packaging unit** is nonetheless set from the originating line, so
the packaging quantity is still available for printing and for the full-packaging reservation
policy. Turning propagation on makes the move's unit and its packaging unit coincide, which makes
the packaging quantity equal to the demand.

---

## 5. Settings exposed in the user interface

One setting governs the whole domain. It appears in three application settings pages, all writing
the same underlying flag.

| Setting storage name | Label | Help text | Implies |
|---|---|---|---|
| `group_uom` | `Units of Measure & Packagings` | `Sell and purchase products in different units of measure or packagings` | `uom.group_uom` |

| Settings page | Block title | Extra control shown when the setting is on |
|---|---|---|
| Accounting settings | `Units & Packagings` | A link button labelled `Units & Packagings` that opens the unit list. |
| Warehouse settings | The product block | The same link button. |
| Purchasing settings | The product block | The same link button. |

Turning the setting **off** does not delete anything and does not convert anything. Products keep
their additional units; document lines keep their units; conversions keep happening. The user
simply stops seeing unit selectors and therefore stops creating new lines in anything but the
product's own unit.

---

## 6. Numbering sequences

The domain defines no sequence of its own. It **creates** sequences on behalf of package types.

| Sequence | When created | Code | Prefix | Padding | Company |
|---|---|---|---|---|---|
| Package type sequence | When a package type is saved with a sequence prefix and no sequence yet | The prefix | The prefix | 7 | The package type's company |

Its generated name is the words `Package Type Sequence`, a space, and the prefix.

**Fallback.** When a package must be named and either more than one package type is in hand or
the single package type owns no sequence, the installation's generic package sequence is used
instead. That sequence is owned by [`../inventory-operations/`](../inventory-operations/).

**Worked example.** A package type named `Euro pallet` is saved with the sequence prefix `PAL/`.
A sequence is created with the name `Package Type Sequence PAL/`, the code `PAL/`, the prefix
`PAL/` and a padding of seven. The first package made from it is named `PAL/0000001`.

---

## 7. Scheduled jobs

**This domain defines no scheduled job.** Nothing recomputes absolute quantities on a timer;
nothing re-rounds stored quantities after a precision change; nothing prunes archived units.

Jobs owned by other domains that depend on this one's arithmetic — the replenishment scheduler,
the procurement runner, the forecast recomputation — are specified in those domains.

---

## 8. Installation order and prerequisites

A rebuild must create the domain's data in this order.

1. Create the `Product Unit` decimal precision with two digits. Every later step needs it,
   because every unit derives its rounding precision from it.
2. Create the root units in the order of the table in section 2.1, each with no reference unit
   and a contained quantity of exactly one: the counting unit, the working hour, the millimetre,
   the square metre, the millilitre, the gram, the energy unit.
3. Create the derived units in the order of the table, each after its reference unit, so that the
   recursive absolute quantity can be computed from a stored value.
4. Apply the archival flags of the table.
5. Create the feature group and the two access rules.
6. Where later capabilities are installed, apply their reactivations, their added units, their
   trade codes and their national codes.

**Why the order of step 3 matters.** The absolute quantity is computed from the reference unit's
*stored* absolute quantity. Creating a unit before its reference unit would either fail or store
a zero, and a later recomputation would produce a different last digit for the foot, the yard and
the mile than the one tabulated in [`calculations.md`](calculations.md).

---

## 9. Configuration a business must perform

The delivered data is a starting point, not a working configuration. The steps below are the
minimum for the packaging scenario this specification serves.

1. **Enable the feature.** Turn the `Units of Measure & Packagings` setting on. Without it, unit
   selectors never appear.
2. **Reactivate the dozen** if dozens are traded. It is archived on delivery.
3. **Create the packaging units.** For a business trading in boxes of twelve dozens on pallets of
   forty boxes:

   | Name | Reference unit | Contains |
   |---|---|---|
   | `Box of 12 Dozens` | `Dozens` | 12 |
   | `Pallet of 40 Boxes` | `Box of 12 Dozens` | 40 |

   Creating them in this order is required, for the reason given in section 8.
4. **Attach the packagings to each product.** Add `Box of 12 Dozens` and `Pallet of 40 Boxes` to
   the additional-units list of every product that ships that way. A packaging is only selectable
   on a document line for a product that lists it.
5. **Bind barcodes** if cartons are scanned: one Product Unit Barcode row per (product, unit)
   pair, with a barcode that collides with no product barcode and no other packaging barcode.
6. **Create the package types** that describe the physical cartons and pallets, with their
   dimensions, tare weights and maximum weights, and a sequence prefix if packages must be
   numbered per type.
7. **Choose the reservation policy** on each product category: full packagings or partial.
8. **Decide the weight and volume interpretation** by setting or leaving the two system
   parameters.
9. **Decide whether units propagate to stock moves** by setting or leaving the propagation
   parameter.
10. **Review the precision.** Two digits is right for counting and for kilograms. It is wrong for
    a business keeping stock in grams and selling in kilograms, or keeping stock in units and
    selling on pallets of thousands; raise it to three or four digits before any document exists.

---

## 10. Configuration checklist for a rebuild

| Item | Must exist | Verified by |
|---|---|---|
| The `Product Unit` precision with two digits | yes | Every unit reports a rounding precision of one hundredth. |
| Thirty delivered units with the tabulated contained quantities | yes | The absolute quantity column of section 2.1 matches. |
| Sixteen of them archived | yes | The active column of section 2.1 matches. |
| The seven trees | yes | The shared-ancestor test agrees with the table in [`calculations.md`](calculations.md). |
| The feature group with no access rights | yes | Adding a user to it grants no new read or write permission. |
| The two access rules on the unit entity | yes | An internal user can read units and cannot create one. |
| No record rule on the unit entity | yes | A user of company A sees a unit created by company B. |
| The three system parameters honoured | yes | Setting each changes the derived labels and the move creation as tabulated. |
| The package type sequence creation | yes | Saving a package type with a prefix creates a sequence with padding seven. |
| No scheduled job | yes | The job list contains nothing from this domain. |
