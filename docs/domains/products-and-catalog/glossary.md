# Products and Catalog — Glossary

Every term this folder uses, in full. Terms are listed alphabetically. Where a term names a stored
entity, the transport name and the storage name are given in code font; the transport name is the
string an external caller uses to address the entity, the storage name is the table that holds it.

Reproduced identifiers (field names, transport names, selection values, route paths) appear in code
font and are the literal strings an implementation must use so that external contracts keep
working. Every reproduced identifier is accompanied by its full name in words at least once.

---

## A

**Absolute factor** — the number of base units that one unit of a given unit of measure represents,
derived by walking the chain of relative factors up to the reference unit of the category. The
catalog never computes it; it asks the unit of measure domain (see
`../units-of-measure-and-packaging/`). It appears here only because price conversion between units
uses it.

**Alert date (on a lot or serial number)** — the moment from which the system considers the goods
close enough to their expiration that a human should be told. Stored on the Lot or Serial Number as
`alert_date` ("alert date"). Computed by subtracting the product's alert day count from the
expiration date. Reaching it causes a scheduled reminder activity to be created once.

**Alert day count** — the whole number of days before the expiration date at which the alert date
falls. Stored on the Product Template as `alert_time` ("alert time").

**Alias rule** — a barcode rule whose result type is `alias`. Matching such a rule does not end the
parse; instead the scanned string is replaced by the rule's alias string and the rule list is
scanned again from where it stopped.

**Application identifier** — in the Global Standards One barcode standard, a two- to four-digit
prefix that announces what the digits following it mean (a trade item number, a batch number, an
expiration date, a net weight, and so on). In this domain an application identifier is the first
captured group of a Global Standards One barcode rule's pattern.

**Archived** — a record whose activity flag is false. Archived records are excluded from ordinary
searches but remain in storage and remain referenced by historical documents. In this domain the
archival flag is `active` ("active") on most entities, but it is `ptav_active` ("template attribute
value active") on the Template Attribute Value, precisely because that entity must be displayed
even when archived in some views.

**Attribute** — a dimension along which a product may vary: size, colour, number of legs, engraving
text. Stored as Product Attribute (`product.attribute`, table `product_attribute`). An attribute
owns its possible values and decides, through its variant-creation mode, whether using it on a
template creates variants.

**Attribute value** — one possible value of an attribute ("Large", "Black"). Stored as Attribute
Value (`product.attribute.value`, table `product_attribute_value`). An attribute value is shared
across every template that uses the attribute; the per-template surcharge lives on the Template
Attribute Value, not here.

---

## B

**Barcode** — a machine-readable identifier printed on an item. In the catalog, the field `barcode`
("barcode") on a Product Variant, and the required field `barcode` on a Packaging Barcode.

**Barcode nomenclature** — an ordered set of barcode rules plus a conversion policy, used to turn a
scanned string into a typed piece of information. Stored as Barcode Nomenclature
(`barcode.nomenclature`, table `barcode_nomenclature`). A company points at exactly one
nomenclature.

**Barcode rule** — one pattern in a nomenclature, together with the encoding it demands, the result
type it produces and, in Global Standards One mode, the content type and decimal policy of the value
it captures. Stored as Barcode Rule (`barcode.rule`, table `barcode_rule`).

**Base code** — the result of a classic barcode parse in which every position that carried a
numeric value has been overwritten with the digit zero and, for the fixed-length encodings, the
check digit has been recomputed. The base code is the string that is actually stored on the product,
so that one stored barcode can stand for every weight-carrying or price-carrying variation of it.

**Base price (of a combo choice)** — the smallest sales price, expressed in the choice's currency,
among the items of that choice. Stored as `base_price` ("base price") on the Product Combo and
recomputed whenever its items change. It is the weight used to prorate the combo product's price
over the chosen items.

**Best-before date** — the moment from which the goods start deteriorating without yet being
dangerous. Stored on the Lot or Serial Number as `use_date` ("use date").

**Best-before day count** — the whole number of days before the expiration date at which the
best-before date falls. Stored on the Product Template as `use_time` ("use time").

---

## C

**Catalog view** — the grid of product cards from which a user adds products to an order-like
document by typing quantities. Fed by the Product Catalog Mixin contract.

**Category** — a node of the hierarchical classification of templates. Stored as Product Category
(`product.category`, table `product_category`). A category also carries the definition of the
per-category custom properties available on its products.

**Check digit** — the last digit of a fixed-length numeric barcode, computed from the preceding
digits so that a single mistyped digit is detected. The arithmetic is given in
[calculations.md](calculations.md).

**Closest possible combination** — the possible combination that keeps as many values of a
requested combination as possible, obtained by dropping values from the end of the requested
combination until at least one possible completion exists.

**Combination** — an ordered-by-attribute set of Template Attribute Values, one per attribute line
of the template (except for multi-checkbox lines, which may contribute zero, one or several). A
combination either designates an existing variant, may create one on demand, or is impossible.

**Combination indices** — the comma-joined ascending list of Template Attribute Value identifiers of
a variant, stored on the variant as `combination_indices` ("combination indices"). It is the key by
which a combination is looked up and the key of the uniqueness index that forbids two active
variants of the same template with the same combination.

**Combo** — a product type whose purchase entitles the buyer to pick one item from each of several
choice groups (a burger menu: one main, one side, one drink). Stored on the template as the type
value `combo`.

**Combo choice** — one group of alternatives inside a combo product. Stored as Product Combo
(`product.combo`, table `product_combo`).

**Combo item** — one selectable variant inside a combo choice, with the surcharge that picking it
adds. Stored as Product Combo Item (`product.combo.item`, table `product_combo_item`).

**Company scoping** — the rule that a record either belongs to one company or to none. A record
belonging to no company is visible to every company; a record belonging to a company is visible to
that company and to its descendants in the company tree.

**Complete name (of a category)** — the slash-joined chain of names from the root category down to
the category itself, for example `All / Saleable / Office Furniture`.

**Configurable template** — a template that must be passed through the configurator before it can
be ordered, because it has a dynamic attribute, or an attribute line with two or more values, or a
multi-checkbox attribute, or a free-text value.

**Cost** — the value of one unit of the product for valuation and margin purposes. Stored per
company on the Product Variant as `standard_price` ("standard price") and mirrored on the template.

**Custom value** — the free text a buyer typed for an attribute value flagged as free text. Stored
as Attribute Custom Value (`product.attribute.custom.value`, table
`product_attribute_custom_value`).

---

## D

**Decimal position digit** — in a Global Standards One measure rule whose decimal usage flag is set,
the last digit of the matched application identifier, read as the number of digits of the captured
value that lie after the decimal point.

**Default extra price** — the surcharge proposed for an attribute value whenever that value is
newly attached to a template. Stored on the Attribute Value as `default_extra_price` ("default extra
price"). It is copied once, at creation of the Template Attribute Value; later changes do not
propagate unless the bulk-update wizard is run.

**Display type** — how the configurator should render an attribute: as radio buttons, as pills, as a
dropdown, as colour swatches, as a multi-checkbox list, or as images. Stored on the Product
Attribute as `display_type` ("display type").

**Document** — a file or link attached to a template or to a variant. Stored as Product Document
(`product.document`, table `product_document`), delegating the file itself to the generic attachment
record.

**Dynamic attribute** — an attribute whose variant-creation mode is `dynamic`. Its use on a template
suppresses up-front variant generation; variants are created one at a time, the first time a
combination is actually ordered.

---

## E

**Encoding** — the fixed-length numeric format a barcode rule demands: any format, thirteen-digit
European Article Number, eight-digit European Article Number, twelve-digit Universal Product Code,
or the Global Standards One one-hundred-twenty-eight symbology. Stored on the Barcode Rule as
`encoding` ("encoding").

**European Article Number** — the thirteen-digit (or eight-digit) numeric article numbering scheme
with a trailing check digit. Reproduced as the selection values `ean13` and `ean8`.

**Exclusion** — a rule stating that one Template Attribute Value cannot be combined with a listed
set of other Template Attribute Values. Stored as Template Attribute Exclusion
(`product.template.attribute.exclusion`, table `product_template_attribute_exclusion`). An exclusion
whose value list is empty and whose owner is a value of another template means that the whole
template is incompatible with that parent value.

**Expiration date** — the moment from which the goods may become dangerous and must not be consumed.
Stored on the Lot or Serial Number as `expiration_date` ("expiration date").

**Expiration day count** — the whole number of days after receipt at which the expiration date falls.
Stored on the Product Template as `expiration_time` ("expiration time").

**Extra price** — the per-template surcharge that choosing a given attribute value adds to the sales
price. Stored on the Template Attribute Value as `price_extra` ("price extra").

---

## F

**Favourite** — a boolean flag on a template used only for ordering: favourite templates sort first.
Stored as `is_favorite` ("is favorite").

**First expiry first out** — the removal strategy that consumes the lots whose removal date is
earliest. Registered by the expiry capability as the removal-strategy method `fefo`.

**Free-text value** — an attribute value flagged with `is_custom` ("is custom"), for which the buyer
types a string instead of (or in addition to) selecting the value.

**Function code one separator** — the non-printing character, or one of the printable alternatives
configured on the nomenclature, that terminates a variable-length field inside a Global Standards
One barcode.

---

## G

**Global Location Number** — a thirteen-digit numeric identifier of a physical or legal location in
the Global Standards One scheme. Carried by the application identifiers `410`, `413` and `414`.

**Global Standards One** — the standards body whose barcode specification defines application
identifiers, check digits and the one-hundred-twenty-eight symbology. A nomenclature flagged
`is_gs1_nomenclature` ("is Global Standards One nomenclature") parses according to that
specification instead of the classic pattern grammar.

**Global Trade Item Number** — the fourteen-digit numeric identifier of a trade item, carried by
the application identifiers `01` and `02`.

**Goods** — the product type for tangible merchandise. Reproduced as the selection value `consu`.

---

## I

**Image resolution ladder** — the five stored sizes of a product image (one thousand nine hundred
and twenty, one thousand and twenty-four, five hundred and twelve, two hundred and fifty-six and one
hundred and twenty-eight pixels on the longest side), all derived from the largest one.

**Internal reference** — the short human code of a product, printed in square brackets before the
name in most displays. Stored on the Product Variant as `default_code` ("default code") and mirrored
on the template.

---

## L

**Label layout** — the transient record describing how product labels should be laid out on a
printed sheet: format, number of copies, extra content, pricelist for the printed price.

**Lot or Serial Number** — the inventory record identifying a batch or an individual item. Owned by
`../inventory-operations/`; extended here with the four expiry dates and the expiry alert flag.

---

## M

**Matrix** — the two-dimensional grid of a template's combinations, used to type quantities for many
variants at once. The first attribute line supplies the columns, the remaining lines supply the
rows.

**Multi-checkbox attribute** — an attribute whose display type is `multi`. Zero, one or several of
its values may be picked at once. The system forbids such an attribute from creating variants.

---

## N

**No-variant attribute** — an attribute whose variant-creation mode is `no_variant`. Choosing one of
its values never creates or selects a different variant; the choice is carried on the order line and
its extra price is added through the price context.

**Nomenclature** — see *Barcode nomenclature*.

---

## P

**Packaging Barcode** — the binding of one barcode to one pair (variant, unit of measure), so that
scanning a case barcode yields both the product and the quantity multiple. Stored as
`product.uom`, table `product_uom`.

**Parent combination** — the combination of the product from which the product being configured is
offered as an optional or accessory item. Exclusions attached to a parent combination make values of
the child product unavailable.

**Pattern** — the matching expression of a barcode rule. In classic mode it is a regular expression
that may contain at most one pair of braces delimiting the numeric content. In Global Standards One
mode it is a regular expression with exactly two capturing groups.

**Possible combination** — a combination that survives every check: the attributes match the
template's configuration, none of its values excludes another of its values, no parent exclusion
hits it, and the corresponding variant exists and is active (or, for a dynamic template, does not
exist yet and is therefore creatable).

**Product Template** — the commercial product as a person thinks of it. Stored as
`product.template`, table `product_template`.

**Product Variant** — one concrete, stockable, orderable, barcode-bearing combination of a template.
Stored as `product.product`, table `product_product`. Every other domain points at variants, not at
templates.

---

## S

**Sales price** — the catalogue price of one unit of the template before any pricelist rule. Stored
on the template as `list_price` ("list price"). The per-variant price including attribute extras is
`lst_price` ("list price with extras") on the variant.

**Sanitised European Article Number** — a thirteen-digit string obtained by truncating the input to
thirteen characters, left-padding it with zeros to thirteen characters, and replacing its last digit
with the freshly computed check digit.

**Serial Shipping Container Code** — the eighteen-digit numeric identifier of a logistic unit,
carried by the application identifier `00`.

**Service** — the product type for non-material offerings. Reproduced as the selection value
`service`.

**Single-value line** — a template attribute line that offers exactly one value. Such a line does
not multiply the number of variants and its value is hidden from the variant's display name.

---

## T

**Tag** — a flat, coloured, optionally customer-visible label on a template or on a variant. Stored
as Product Tag (`product.tag`, table `product_tag`).

**Template Attribute Line** — the use of one attribute on one template, together with the subset of
that attribute's values that the template offers. Stored as
`product.template.attribute.line`, table `product_template_attribute_line`.

**Template Attribute Value** — the materialised pair (template attribute line, attribute value),
carrying the per-template extra price, the exclusions and the per-template activity flag. Stored as
`product.template.attribute.value`, table `product_template_attribute_value`. This is the record
that appears in combinations and on order lines; the shared Attribute Value never does.

---

## U

**Uniform resource identifier barcode** — a scanned string beginning with `urn:`, encoding an
electronic product code. Decoded into a product identifier plus a batch number, or into a logistic
unit identifier, before any nomenclature rule is consulted.

**Universal Product Code** — the twelve-digit North American numeric article numbering scheme.
Reproduced as the selection value `upca`.

**Unpadding** — the preprocessing applied to a barcode search term under a Global Standards One
nomenclature: leading zeros are stripped and the remainder is matched with a contains-style
comparison, so that a fourteen-digit trade item number finds a thirteen-digit stored barcode.

---

## V

**Variant-creation mode** — the policy of an attribute about generating variants: `always`
("instantly"), `dynamic` ("dynamically") or `no_variant` ("never"). Stored on the Product Attribute
as `create_variant` ("create variant"). It cannot be changed once the attribute is used on an
active template.

**Variant generation** — the process that brings the set of a template's variants into agreement
with its attribute configuration: enumerate, filter, reactivate, create, archive or delete. Specified
step by step in [calculations.md](calculations.md).

**Variant limit** — the ceiling on how many variants one generation pass may create, read from the
system parameter `product.dynamic_variant_limit` and defaulting to one thousand.
