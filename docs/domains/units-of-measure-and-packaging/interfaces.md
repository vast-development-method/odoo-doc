# Units of Measure and Packaging — Interfaces

Navigation, screens, controls, named operations, routes, printable documents, notifications and
exchange formats through which this domain is reached.

---

## 1. Navigation

The domain has **no application of its own**. Its single window action is mounted as a
configuration entry in three applications, always behind the multiple-units feature group.

| Menu path | Mounted by | Sequence within its parent | Visible to |
|---|---|---|---|
| Sales → Configuration → `Units & Packagings` | The sales application | 35 | Holders of the multiple-units feature group |
| Inventory → Configuration → `Units & Packagings` | The warehouse application | 5 | Holders of the multiple-units feature group |
| Purchase → Configuration → `Units & Packagings` | The purchasing application | 10 | Holders of the multiple-units feature group |

In addition, every settings page that exposes the feature toggle shows, when the toggle is on, a
link button labelled `Units & Packagings` that opens the same action. Three settings pages do so:
accounting, warehouse and purchasing.

Package types are reached from the warehouse application's configuration menu through their own
window action named `Package Types`; that entry belongs to
[`../inventory-operations/`](../inventory-operations/) and is listed here only because the
records it manages are specified in this folder.

---

## 2. Window actions

### 2.1 Units and packagings

| Property | Value |
|---|---|
| External identifier | `uom.product_uom_form_action` |
| Title | `Units & Packagings` |
| Entity | `uom.uom` |
| Opening view | The list view |
| Search view | The unit search view |
| Empty-state message | A friendly-face placeholder whose text is `Add a new unit of measure` |

### 2.2 Packaging barcodes of one unit

Returned dynamically by a button on the unit form rather than declared as a stored action.

| Property | Value |
|---|---|
| Kind | Window action returned by an operation |
| Title | `Packaging Barcodes` |
| Entity | `product.uom` |
| View mode | List only, using the packaging barcode list view |
| Filter | Rows whose unit is the unit the button was pressed on |
| Context passed | A default unit equal to the unit the button was pressed on |

### 2.3 Package types

| Property | Value |
|---|---|
| Title | `Package Types` |
| Entity | `stock.package.type` |
| View modes | List and form |

---

## 3. Views

### 3.1 Unit of Measure — list

Title: `Units & Packagings`.

| Column | Notes |
|---|---|
| Sequence | Shown as a drag handle, so the ordering is set by dragging rows. |
| Unit Name | |
| Contains | Hidden on a row whose contained quantity is one **and** which has no reference unit — that is, hidden on root units, because "contains one of nothing" is noise. Displayed with up to three decimal digits. |
| Reference Unit | |

### 3.2 Unit of Measure — form

Title: `Units of Measure`.

Layout: a single group named for the unit's details containing

- the **Unit Name**, made read-only when the form was opened from a product context **and** the
  record already exists — that is, a packaging reached from a product may not be renamed in
  place, because the rename would affect every other product using it;
- a label reading `Quantity`, followed by an inline row holding the **Contains** value (displayed
  with up to five decimal digits, width-limited) and the **Reference Unit** (with the placeholder
  text `Reference Unit`), both read-only under the same product-context condition.

Where the catalogue capability is installed, a button box is added to the form holding one
statistic button with a barcode icon and the caption `Packaging Barcodes`, which invokes the
operation of 2.2.

**Deliberate omissions.** The form shows neither the absolute quantity nor the rounding precision
nor the hierarchy path. A user configures only the contained quantity and the reference unit;
everything else is derived. A rebuild that exposes the absolute quantity as editable will allow
inconsistent data.

### 3.3 Unit of Measure — search

Title: `Search UOM` (the abbreviation in the title stands for *unit of measure*).

| Element | Behaviour |
|---|---|
| Text field | Matches the unit name. |
| Filter `Archived` | Shows units whose archival flag is cleared. |

### 3.4 Product Unit Barcode — list

Title: `Packaging Barcodes`. Editable in place, with new rows appended at the bottom.

| Column | Notes |
|---|---|
| Product | |
| Barcode | |
| Unit | Optional, hidden by default. Read-only when the list was opened with a default unit in context, which is the case when it was reached from a unit's form. |

### 3.5 Package Type — form

Title: `Package Type`. The name is the form's heading. Three pages:

| Page | Contents |
|---|---|
| `Configuration` | Two groups. The identification group holds the barcode, the reference sequence (visible only to technical readers), the sequence prefix and the routes (as removable tags, visible only where advanced locations are enabled, with creation disabled). The delivery group holds the company (visible only in a multi-company installation). |
| `Dimensions` | A label reading `Size` followed by an inline row of length, width and height separated by multiplication signs, then the length unit label with the help text `Size: Length × Width × Height`. Then the tare weight followed by the weight unit label, then the maximum weight followed by the weight unit label. |
| `Capacity` | Visible only where multiple locations are enabled. An editable list of storage category capacities, each row holding a storage category and a quantity. |

### 3.6 Package Type — list

Title: `Package Types`.

| Column | Notes |
|---|---|
| Sequence | Drag handle. |
| Package Type | |
| Height, Width, Length | |
| Max Weight | |
| Has Contents | Optional, shown by default. |
| Barcode | Optional, hidden by default. |

### 3.7 Product form — the unit controls

| Control | Where | Behaviour |
|---|---|---|
| Own unit, beside the sales price | Read as "…per *unit*" | Uses the unit-aware selector described in section 4. Hidden for a combination-offer product. Behind the multiple-units feature group. Quick creation of a unit from the field is disabled. |
| Own unit, beside the cost | Same rendering | Same, and additionally hidden when the form shows a template with more than one variant. |
| Additional units | On the sales page, in the upsell group | A removable-tag selector using the unit-aware autocomplete of section 4, with quick creation disabled, tag editing enabled, and a context carrying the single variant's identifier when the template has exactly one variant, the full list of variant identifiers otherwise, and a flag asking the barcode list to show the variant name when the template has several variants. Behind the multiple-units feature group. |
| Own unit, in the product list | A column | Read-only, shown by default, behind the multiple-units feature group. |

### 3.8 Where the unit appears on document lines

In every case the field is placed behind the multiple-units feature group, so that an
installation that does not use packagings never sees a unit column.

| Document | Field | Rendering |
|---|---|---|
| Sales order line | Line unit | Unit-aware selector, restricted to the line's allowed units. |
| Purchase order line | Line unit | Unit-aware selector, restricted to the line's allowed units, which include vendor units. |
| Invoice and bill line | Line unit | Unit-aware selector. |
| Stock move | Move unit | Unit-aware selector; read-only once the move is complete. |
| Stock move line | Picked unit | Unit-aware selector. |
| Bill of materials and its lines | Yield unit and component units | Unit-aware selector, each configured with the name of the quantity field beside it. |
| Production order, its raw-material moves and its finished moves | Order unit and move units | Unit-aware selector. |
| Production split wizard, consumption warning wizard | Unit | Unit-aware selector. |
| Unbuild order | Unit | Unit-aware selector. |
| Work centre alternative products | Unit | Unit-aware selector. |
| Batch transfer report and picking batch view | Unit | Plain read-only field. |

---

## 4. The unit-aware selection controls

Two client-side controls exist purely to make unit selection comprehensible. They add no server
behaviour, but their displayed text is part of the specified user experience.

### 4.1 The unit selector

A single-value selector that knows which product it is beside.

**Configuration options.** The name of the field holding the product (defaulting to the product
link on the record) and the name of the field holding the quantity (defaulting to the line's
demand). When the control is used on a product form itself, the product is the record being
edited.

**Behaviour.**

1. On opening, the control reads the product's own unit and remembers its name, absolute
   quantity, hierarchy path and rounding precision. This is the **reference unit** for the
   annotations below.
2. When the user types, the control searches units by name within the field's domain and requests
   each candidate's contained quantity, absolute quantity, reference unit and hierarchy path.
3. Each candidate is annotated with a **relative information** string:
   - if the candidate shares the first element of its hierarchy path with the reference unit
     **and** has a reference unit of its own, the annotation is the current quantity (or one when
     the quantity is empty) multiplied by the candidate's *contained* quantity, rounded onto the
     reference unit's rounding precision, followed by the name of the candidate's own reference
     unit;
   - otherwise, if the candidate is not the reference unit, the annotation is the current quantity
     (or one) multiplied by the candidate's absolute quantity divided by the reference unit's
     absolute quantity, rounded onto the reference unit's rounding precision, followed by the
     reference unit's name;
   - the reference unit itself gets no annotation.
4. Candidates that share the first path element with the reference unit are sorted before those
   that do not.
5. The displayed label of a candidate is the first line of its name.

**Worked example.** A product whose own unit is `Units`, a line quantity of five, and a candidate
`Box of 12 Dozens` whose reference unit is `Dozens`: the candidate shares the counting root and
has a reference unit, so the annotation is five multiplied by twelve, rounded to one hundredth,
followed by `Dozens` — that is, `60 Dozens`. A candidate `Pallet of 40 Boxes` whose reference
unit is the box gives `200 Box of 12 Dozens`. A candidate `kg`, which shares no root, gives the
second form: five multiplied by one thousand divided by one, that is `5000 Units` — a meaningless
figure that the sorting pushes to the bottom of the list.

**Rebuild note.** The annotation is *not* the conversion result. For a unit that has a reference
unit it deliberately shows the quantity in terms of the candidate's **own** reference unit, not in
terms of the product's unit. Reproducing the exact text matters only for user-visible equivalence,
not for data equivalence.

### 4.2 The unit tag control

The multi-value form of the same control, used for a product's additional units. It carries the
same product-field and quantity-field options, the same annotation logic and the same sorting,
rendered as removable coloured tags.

### 4.3 The time input controls

Where the time-recording capability is installed, the unit carries a name of an input control.
Two values occur: one meaning a clock-style hours-and-minutes input, used on the working-hour
unit, and one meaning a stepping toggle, used on the day unit. The value selects the control the
desktop client uses when a duration is entered in that unit.

---

## 5. Named operations exposed over the remote transport

The domain exposes very few named operations. Everything else is ordinary record reading and
writing.

| Operation | Entity | Inputs | Output | Purpose |
|---|---|---|---|---|
| Open the packaging barcodes of a unit | `uom.uom` | Exactly one unit | A window action description filtered to that unit's barcode rows, carrying that unit as the default for new rows | The statistic button on the unit form. |
| Convert a quantity | `uom.uom` | A quantity, a destination unit, a rounding flag, a rounding method, a tolerate-failure flag | The converted quantity | Internal; not intended as a public contract, but a rebuild that exposes an equivalent must accept the same five inputs with the same defaults. |
| Convert a price | `uom.uom` | A price and a destination unit | The converted price | Internal; see above. |
| Round a value in a unit | `uom.uom` | A value and a rounding method | The rounded value | Internal. |
| Compare two values in a unit | `uom.uom` | Two values | Minus one, zero or plus one | Internal. |
| Test a value for zero in a unit | `uom.uom` | A value | True or false | Internal. |
| Snap a quantity to whole packagings | `uom.uom` | A quantity, a destination unit, a rounding method | The snapped quantity | Internal. |
| Test whether two units share an ancestor | `uom.uom` | Another unit | True or false | Internal. |
| Get the number of digits of a named precision | `decimal.precision` | A usage name | The number of digits, or two when unknown | Used by every client that must format or validate a quantity. |
| Get the next package name for a package type | `stock.package.type` | none | A generated name | Used when a package is created. |

**A caution for rebuilds.** The conversion, price conversion, rounding, comparison, zero-test and
snapping operations are *internal* in the system being specified: they are not part of a
documented external contract. They are nonetheless listed because any rebuild must offer
equivalent behaviour to its own layers, and because a client written against the original will
call them. Reproducing their names is optional; reproducing their signatures and defaults is not.

---

## 6. Routes

**This domain publishes no route of its own.** No path is served, no controller is registered.

Quantities and units cross routes that belong to other domains. The three that matter for
equivalence are:

| Route owner | What it carries from this domain |
|---|---|
| The storefront cart service | A unit identifier accompanying a product identifier and a quantity when a visitor adds to a cart, and the validation that refuses a unit the product is not available in. |
| The product configurator service | The unit chosen for a configured product and the price per that unit. |
| The counter-sale synchronisation service | The units of the lines being synchronised, and the conversion refusal when a conversion rounds to zero. |

Their paths, methods, authentication and payloads are specified in
[`../website-and-storefront/`](../website-and-storefront/),
[`../products-and-catalog/`](../products-and-catalog/) and
[`../point-of-sale/`](../point-of-sale/) respectively.

---

## 7. Printable documents

The domain produces no report of its own. It contributes a **unit column or unit suffix** to the
printable documents of other domains, and in every case the column is conditional on the
multiple-units feature group, so that an installation not using packagings prints a bare number.

| Printable document | What this domain contributes |
|---|---|
| Quotation and order confirmation | The line unit beside the ordered quantity. |
| Delivery note and picking operations | The move or move line unit beside each quantity; the packaging unit and packaging quantity where a document line used one. |
| Batch picking report | The move operation unit beside each quantity. |
| Customer invoice and vendor bill | The invoice line unit beside the invoiced quantity. |
| Purchase order | The line unit beside the ordered quantity. |
| Bill of materials structure report | The bill unit beside the produced quantity and the component unit beside each component quantity, together with the costs converted per those units. |
| Production order overview | The order unit, the component units and the quantities converted into each. |
| Package content label | The content description: the quantity, then the unit name **only when the reader holds the multiple-units group**, then the product name. |
| Product label | One label per item for a counting-tree unit, one label per line otherwise. |
| Lot and serial number label | The same rule applied to the move line's unit. |
| Package type label | The dimensions with the length unit label and the weights with the weight unit label. |

---

## 8. Notifications and warnings

The domain sends no message and creates no activity. It raises three client-side warnings and a
set of blocking errors; all are specified with their exact text in
[`business-rules.md`](business-rules.md).

| Kind | Trigger | Blocking? |
|---|---|---|
| Warning on a protected unit's contained quantity | Editing the contained quantity of a protected unit older than one day | No |
| Warning on a reduced decimal precision | Lowering the digits of a decimal precision | No |
| Warning on a product's unit change | Changing a product's own unit while historical records exist | No |
| Error on deleting a protected unit | Deleting | Yes |
| Error on a missing reference unit | Saving a root unit whose contained quantity is not one | Yes |
| Error on a zero contained quantity | Saving | Yes |
| Error on a duplicate barcode | Saving a packaging barcode or a product barcode | Yes |
| Error on a duplicate package type barcode | Saving | Yes |
| Error on changing a completed move's unit | Saving | Yes |
| Error on an off-grid picked quantity | Validating a transfer | Yes |
| Error on a serial-tracked quantity other than one | Editing a move line | Yes |
| Error on a conversion that rounds to zero | Completing a counter-sale transfer | Yes |
| Error on an unavailable storefront unit | Adding to a cart | Yes |
| Error on changing a product's unit when another is in use | Saving the product | Yes |
| Error on changing a product's unit when a posted document uses another | Saving the product | Yes |

---

## 9. External exchange formats

### 9.1 Outgoing exchanged invoices

Every exchanged invoice line carries a **standard international trade code** for its unit,
resolved from the unit's external identifier through the mapping in
[`configuration.md`](configuration.md) section 2.5, with a fallback to the code for a plain piece.

The mapping is by external identifier, not by name, so a unit created by a user always emits the
fallback code however it is named. A business that must emit a specific code for a user-created
unit has to give that unit the external identifier of a mapped one, which in practice means
reusing a delivered unit instead.

### 9.2 Incoming exchanged invoices

1. The trade code on the incoming line is translated back into a unit; an unrecognised code
   becomes the counting unit.
2. The matched product's own unit is compared with the translated unit using the shared-ancestor
   test.
3. **If they do not share an ancestor, the translated unit is discarded** and the product's own
   unit is used instead, with the quantity unchanged.
4. The same test is applied a second time, later in the mapping, when a unit has been resolved
   from a different element of the document.

### 9.3 National unit codes

Where a country capability is installed, the unit additionally carries a national code used in
that country's mandated format. The codes are tabulated in [`configuration.md`](configuration.md)
section 2.6. One country's format validation reports a failure with the message stating that the
invoice lines' unit codes should all be set up correctly when a required code is missing; the
abbreviation in that message stands for *unit of measure*.

### 9.4 Import and export of the entities of this domain

All four entities take part in the platform's generic import and export.

| Entity | Import notes |
|---|---|
| `uom.uom` | Import the reference unit by external identifier or by name. Because names are not unique, importing by name is ambiguous; a rebuild should prefer the external identifier. Import the contained quantity, never the absolute quantity: the absolute quantity is derived and any imported value is overwritten. Import parents before children. |
| `product.uom` | Import the product and the unit by external identifier, and the barcode as text. Both uniqueness rules are enforced on import. |
| `stock.package.type` | Import the dimensions, weights, barcode, reuse policy and sequence prefix. Supplying a prefix on import creates the sequence. |
| `decimal.precision` | Import the usage and the digits. Changing the digits of the `Product Unit` record by import has the same global effect as changing it by hand, and raises no warning because the warning is a user-interface behaviour. |

**Export note.** Exporting a unit exports its contained quantity and its reference unit, which
together are sufficient to reconstruct the absolute quantity. Exporting the absolute quantity as
well is harmless but redundant, and re-importing it has no effect.

---

## 10. Integration points with external services

The domain integrates with nothing directly. Three indirect integrations depend on it:

| Integration | Dependency |
|---|---|
| Shipping carriers | The weight of a transfer, computed by converting each picked quantity into the product's own unit and multiplying by the product's weight, then read as kilograms or pounds according to the weight system parameter. A carrier that expects one and receives the other will quote wrongly. |
| Barcode scanners | The shared barcode namespace between products and packagings, and the fact that one scan of a packaging code means one packaging, not one item. |
| Electronic document networks | The trade code mapping of 9.1 and 9.2. |

---

## 11. What a rebuild's user interface must get right

1. Every unit control is behind the multiple-units feature group, and hiding it must not change
   any stored value.
2. The unit list hides the contained quantity on root units.
3. The unit form exposes only the name, the contained quantity and the reference unit.
4. A unit reached from a product context has its name, contained quantity and reference unit
   read-only when it already exists.
5. The selector annotates candidates with a quantity in the candidate's own reference unit when
   the trees match, and in the product's unit otherwise, and sorts matching trees first.
6. The package type form shows the length unit label beside the dimensions and the weight unit
   label beside both weights, resolved from the system parameters.
7. The package content description omits the unit name for a reader without the feature group.
8. Label printing distinguishes counting-tree units from all others.
