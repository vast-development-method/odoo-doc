# Products and Catalog — Configuration

Settings, system parameters, decimal precisions, shipped default records, security groups, the
complete access-rights matrix, record rules, installation hooks and scheduled work of this domain.

---

## 1. Settings

Settings live on the configuration screen. Each setting either toggles a security group — in which
case granting it reveals a part of the interface to every internal user — or writes a system
parameter.

| Setting (storage name) | Kind | Values and default | Effect |
|---|---|---|---|
| Units of Measure and Packagings (`group_uom`) | group toggle | off | Grants `uom.group_uom`. Reveals the unit fields on products and documents, and the additional-packaging list. Owned by `../units-of-measure-and-packaging/`. |
| Variants (`group_product_variant`) | group toggle | off | Grants `product.group_product_variant`. Reveals the attribute configuration on the product form, the variant list, the attribute menu and the attribute-value columns of the catalog and matrix views. |
| Promotions, Coupons, Gift Card and Loyalty Program (`module_loyalty`) | capability toggle | off | Installs the loyalty capability. Owned by `../loyalty-and-promotions/`. |
| Pricelists (`group_product_pricelist`) | group toggle | off | Grants `product.group_product_pricelist`. Owned by `../pricing-and-pricelists/`, but its side effects on companies are specified in section 6 below. |
| Weight unit of measure (`product_weight_in_lbs`) | parameter | `0` "Kilograms (kg)" (default), `1` "Pounds (lb)" | Writes the system parameter `product.weight_in_lbs`. Chooses the unit in which the weight field is interpreted. |
| Volume unit of measure (`product_volume_volume_in_cubic_feet`) | parameter | `0` "Cubic Meters (m³)" (default), `1` "Cubic Feet (ft³)" | Writes the system parameter `product.volume_in_cubic_feet`. Chooses the unit in which the volume field is interpreted, **and** the unit in which the length, width and height fields are interpreted. |
| Display Expiration Dates on Delivery Slips (`group_expiry_date_on_delivery_slip`) | group toggle | off | Grants `product_expiry.group_expiry_date_on_delivery_slip`. Adds the expiry columns to the printed delivery document. |

### 1.1 Coupled settings

- Turning **off** the lot-on-delivery-slip setting (owned by `../inventory-operations/`) turns off
  the expiry-date-on-delivery-slip setting.
- Turning **on** the lot-and-serial-number setting turns on the expiry capability toggle.
- Turning **off** the expiry capability toggle turns off the expiry-date-on-delivery-slip setting.
- Turning **off** the pricelist setting while at least one active pricelist exists produces the
  warning "You are deactivating the pricelist feature. Every active pricelist will be archived.",
  and on saving, every pricelist is archived.
- Turning **on** the pricelist setting, when it was off, runs the default-pricelist bootstrap of
  section 6.

---

## 2. System parameters

| Parameter | Default | Read by | Meaning |
|---|---|---|---|
| `product.dynamic_variant_limit` | `1000` | Variant generation | The maximum number of variants one generation pass may create for one template. The comparison is strictly greater than the limit, so exactly the limit is allowed. Exceeding it raises the generation-ceiling error. |
| `product.weight_in_lbs` | unset, read as not `1` | The weight-unit resolution | When it is exactly the string `1`, weights are interpreted in pounds; otherwise in kilograms. |
| `product.volume_in_cubic_feet` | unset, read as not `1` | The volume-unit and length-unit resolutions | When it is exactly the string `1`, volumes are interpreted in cubic feet and lengths in feet; otherwise in cubic metres and millimetres. |
| `barcode.max_time_between_keys_in_ms` | `150` | The scan collector in the client | Two keystrokes at most this many milliseconds apart belong to the same scan. The value is placed in the session payload, but only for internal users. |

### 2.1 Unit resolution

```
weight unit  = the shipped unit "lb"  when product.weight_in_lbs is "1"
             = the shipped unit "kg"  otherwise

length unit  = the shipped unit "ft"  when product.volume_in_cubic_feet is "1"
             = the shipped unit "mm"  otherwise

volume unit  = the shipped unit "ft³" when product.volume_in_cubic_feet is "1"
             = the shipped unit "m³"  otherwise
```

The label fields on the product show the display name of the resolved unit. Note that one parameter
governs both the volume unit and the length unit.

---

## 3. Decimal precisions

Four named precisions are shipped by this domain, each with a default digit count. A precision is a
record, so an administrator may change the digit count, and every field declared against that
precision changes with it.

| Precision name | Default digits | Fields that use it |
|---|---|---|
| `Product Price` | 2 | Template sales price; template and variant cost; variant price extra; variant sales price; Template Attribute Value extra price; combo base price; combo item original price and extra price; vendor unit price |
| `Discount` | 2 | Pricelist and vendor discount percentages (owned by `../pricing-and-pricelists/`) |
| `Stock Weight` | 2 | Template and variant weight |
| `Volume` | 2 | Template and variant volume |

One further precision, `Product Unit`, is shipped by `../units-of-measure-and-packaging/` and is
used here for the minimum-quantity comparison in vendor selection.

The `Product Price` precision is declared on most price fields as a **minimum** display precision:
the field is shown with at least that many digits and with more when the value needs them.

---

## 4. Shipped default records

### 4.1 Categories

Three categories, protected against update after installation:

| Name | Parent |
|---|---|
| Goods | none |
| Expenses | none |
| Services | none |

### 4.2 Removal strategy

One removal-strategy record, shipped by the expiry capability:

| Name | Method |
|---|---|
| `First Expiry First Out (FEFO)` — the shipped name; the parenthesised four letters abbreviate the words before them | `fefo` |

### 4.3 Barcode nomenclatures

**Default Nomenclature** — classic mode, conversion policy `always`, protected against update. One
rule:

| Sequence | Name | Type | Encoding | Pattern |
|---|---|---|---|---|
| 90 | Product Barcodes | `product` | `any` | `.*` |

Because the encoding is `any` and the pattern matches anything, this rule classifies every
otherwise-unmatched scan as a product barcode. Other domains insert rules with lower sequences ahead
of it.

**`Default GS1 Nomenclature`** — the shipped name, in which the three characters before the space
stand for Global Standards One. Global Standards One mode, protected against update. Twenty-six
rules, all with the `gs1-128` encoding:

| Sequence | Name | Application identifier pattern | Value pattern | Type | Content type | Decimal | Associated unit |
|---|---|---|---|---|---|---|---|
| 100 | Serial Shipping Container Code | `(00)` | `(\d{18})` | `package` | identifier | — | — |
| 101 | Global Trade Item Number | `(01)` | `(\d{14})` | `product` | identifier | — | — |
| 102 | Trade item number of contained items | `(02)` | `(\d{14})` | `product` | identifier | — | — |
| 110 | Ship-to / deliver-to Global Location Number | `(410)` | `(\d{13})` | `location_dest` | identifier | — | — |
| 113 | Ship-for / deliver-for Global Location Number | `(413)` | `(\d{13})` | `location_dest` | identifier | — | — |
| 114 | Physical location Global Location Number | `(414)` | `(\d{13})` | `location` | identifier | — | — |
| 125 | Batch or lot number | `(10)` | `([!"%-/0-9:-?A-Z_a-z]{0,20})` | `lot` | alpha | — | — |
| 126 | Serial number | `(21)` | `([!"%-/0-9:-?A-Z_a-z]{0,20})` | `lot` | alpha | — | — |
| 137 | Pack date | `(13)` | `(\d{6})` | `pack_date` | date | — | — |
| 138 | Best before date | `(15)` | `(\d{6})` | `use_date` | date | — | — |
| 139 | Expiration date | `(17)` | `(\d{6})` | `expiration_date` | date | — | — |
| 300 | Variable count of items | `(30)` | `(\d{0,8})` | `quantity` | measure | no | Units |
| 305 | Count of trade items in a logistic unit | `(37)` | `(\d{0,8})` | `quantity` | measure | no | Units |
| 310 | Net weight, kilograms | `(310[0-5])` | `(\d{6})` | `quantity` | measure | yes | kilogram |
| 311 | Length or first dimension, metres | `(311[0-5])` | `(\d{6})` | `quantity` | measure | yes | metre |
| 314 | Area, square metres | `(314[0-5])` | `(\d{6})` | `quantity` | measure | yes | square metre |
| 315 | Net volume, litres | `(315[0-5])` | `(\d{6})` | `quantity` | measure | yes | litre |
| 316 | Net volume, cubic metres | `(316[0-5])` | `(\d{6})` | `quantity` | measure | yes | cubic metre |
| 320 | Net weight, pounds | `(320[0-5])` | `(\d{6})` | `quantity` | measure | yes | pound |
| 321 | Length or first dimension, inches | `(321[0-5])` | `(\d{6})` | `quantity` | measure | yes | inch |
| 322 | Length or first dimension, feet | `(322[0-5])` | `(\d{6})` | `quantity` | measure | yes | foot |
| 323 | Length or first dimension, yards | `(322[0-5])` | `(\d{6})` | `quantity` | measure | yes | yard |
| 351 | Area, square feet | `(351[0-5])` | `(\d{6})` | `quantity` | measure | yes | square foot |
| 357 | Net weight or volume, ounces | `(357[0-5])` | `(\d{6})` | `quantity` | measure | yes | ounce |
| 360 | Net volume, quarts | `(360[0-5])` | `(\d{6})` | `quantity` | measure | yes | quart |
| 361 | Net volume, gallons | `(361[0-5])` | `(\d{6})` | `quantity` | measure | yes | gallon |
| 364 | Net volume, cubic inches | `(364[0-5])` | `(\d{6})` | `quantity` | measure | yes | cubic inch |
| 365 | Net volume, cubic feet | `(365[0-5])` | `(\d{6})` | `quantity` | measure | yes | cubic foot |
| 500 | Package type | `(91)` | `([!"%-/0-9:-?A-Z_a-z]{0,90})` | `package_type` | alpha | — | — |

Two remarks an implementation must reproduce:

1. The rule at sequence 323, named for yards, carries the application identifier pattern
   `(322[0-5])` — the same as the feet rule at sequence 322. Because the feet rule has the lower
   sequence, the yards rule is unreachable through the decomposition loop. This is the shipped
   state.
2. The character class `[!"%-/0-9:-?A-Z_a-z]` used by the alphanumeric rules is the Global Standards
   One character set: the exclamation mark, the quotation mark, the range from the percent sign to
   the solidus, the digits, the range from the colon to the question mark, the uppercase letters,
   the low line and the lowercase letters. Notably it does **not** include the space or the
   non-printing group-separator character, which is what allows the separator to terminate a
   variable-length field.

The separator expression of both shipped nomenclatures is the default `(Alt029|#|\x1D)`.

### 4.4 Paper formats and printable documents

| Record | Kind | Details |
|---|---|---|
| A4 Label Sheet | paper format | A4, portrait, zero margins on all four sides, shrinking disabled, ninety-six dots per inch, marked as a default |
| Dymo Label Sheet | paper format | custom, fifty-seven millimetres high by thirty-two wide, landscape, zero margins, shrinking disabled, ninety-six dots per inch, marked as a default |

The printable documents themselves are catalogued in [interfaces.md](interfaces.md), section 6.

---

## 5. Security groups

| Group | Full name | Privilege category | Implied by | Granted to |
|---|---|---|---|---|
| `product.group_product_manager` | Products — Create | "Products" privilege, in the master-data category, sequence 9, placeholder "View", described as "Helps you manage your product catalog." | The system administration group | the root user and the administrator user |
| `product.group_product_variant` | Manage Product Variants | none | — | every internal user, as soon as the product matrix capability is installed |
| `product.group_product_pricelist` | Basic Pricelists | none | — | granted by the pricelist setting, and automatically when multi-currency is activated |
| `product_expiry.group_expiry_date_on_delivery_slip` | Include expiration dates on delivery slip | none | — | granted by the corresponding setting |

The "Products" privilege is a display grouping: in the user form it appears as a single choice
between "View" (no group) and "Create" (the product manager group).

---

## 6. Company bootstrapping

Two hooks run on companies and touch this domain.

### 6.1 Barcode nomenclature

- A new company defaults its nomenclature to the shipped Default Nomenclature.
- When the barcode capability is installed, every existing company whose nomenclature is unset is
  written with the shipped Default Nomenclature.

### 6.2 Default pricelist

Owned by `../pricing-and-pricelists/` but triggered from here, because the trigger points are
company creation and the pricelist setting.

When a company is created, or when the pricelist setting is turned on, and the acting user holds
the pricelist group:

1. Take the companies concerned — the newly created ones, or every company when the call comes from
   the setting.
2. Find, as a privileged reader and including archived records, the pricelists that have **no
   rules** and belong to one of those companies, and whose currency equals their company's
   currency. Unarchive them.
3. For every company that did not get one that way, create a pricelist named "Default", with the
   company's currency, the company, and sequence 10.

When a company's currency is written, the pricelist bootstrap is **suppressed for the duration of
the write** and re-run afterwards only when the pricelist group has just become granted, so that
the pricelist created is created with the new currency rather than the old one.

Archiving a currency archives every pricelist that uses it.

---

## 7. Access rights matrix

Read as: for the named group, on the named entity, which of create, read, update and delete are
allowed. A group not listed for an entity has no rights on it beyond what another of its groups
grants.

### 7.1 Catalog entities

| Entity | Every internal user | Product manager |
|---|---|---|
| Product Template (`product.template`) | read | create, read, update, delete |
| Product Variant (`product.product`) | read | create, read, update, delete |
| Product Category (`product.category`) | read | create, read, update, delete |
| Product Tag (`product.tag`) | read | create, read, update, delete |
| Product Attribute (`product.attribute`) | read | create, read, update, delete |
| Attribute Value (`product.attribute.value`) | read | create, read, update, delete |
| Template Attribute Line (`product.template.attribute.line`) | read | create, read, update, delete |
| Template Attribute Value (`product.template.attribute.value`) | read | create, read, update, delete |
| Template Attribute Exclusion (`product.template.attribute.exclusion`) | read | create, read, update, delete |
| Attribute Custom Value (`product.attribute.custom.value`) | read | create, read, update, delete |
| Product Combo (`product.combo`) | read | create, read, update, delete |
| Product Combo Item (`product.combo.item`) | read | create, read, update, delete |
| Product Document (`product.document`) | read | create, read, update, delete |
| Packaging Barcode (`product.uom`) | read | create, read, update, delete |
| Unit of Measure (`uom.uom`) | (granted by the unit-of-measure domain) | create, read, update, delete |

### 7.2 Wizards

| Entity | Group | Rights |
|---|---|---|
| Label Layout (`product.label.layout`) | every internal user | create, read, update, delete |
| Attribute Value Bulk Update (`update.product.attribute.value`) | product manager | create, read, update — **not** delete |
| Expiry Delivery Confirmation (`expiry.picking.confirmation`) | the inventory user group | create, read, update — **not** delete |

### 7.3 Pricing entities bootstrapped from here

| Entity | Every internal user | Contact manager | Product manager |
|---|---|---|---|
| Pricelist (`product.pricelist`) | read | read | create, read, update, delete |
| Pricelist Rule (`product.pricelist.item`) | read | — | create, read, update, delete |
| Vendor Pricelist Line (`product.supplierinfo`) | read | — | create, read, update, delete |

### 7.4 Barcode entities

| Entity | Every internal user | System administration |
|---|---|---|
| Barcode Nomenclature (`barcode.nomenclature`) | read | create, read, update, delete |
| Barcode Rule (`barcode.rule`) | read | create, read, update, delete |

Note that barcode nomenclatures are administered by the **system administration** group, not by the
product manager group: a nomenclature change affects every scan in the installation.

---

## 8. Record rules

A record rule narrows what a group may see. The rules below apply to every group unless stated
otherwise.

| Entity | Rule | Condition |
|---|---|---|
| Product Template | Product multi-company | the record's company is a parent of one of the acting companies, **or** the record has no company |
| Product Document | Product multi-company | the record's company is a parent of one of the acting companies, **or** the record has no company |
| Product Combo | Product combo multi-company rule | the record has no company, **or** its company is a parent of one of the acting companies |
| Pricelist | product pricelist company rule | the record's company is a parent of one of the acting companies, **or** the record has no company |
| Pricelist Rule | product pricelist item company rule | same |
| Vendor Pricelist Line | product supplierinfo company rule | the record has no company, **or** its company is a parent of one of the acting companies |

All six rules are declared with update protection, so an upgrade of the installation does not
overwrite an administrator's changes to them.

Product Variants have **no** record rule of their own. They are protected by two other mechanisms:
the variant entity declares that its company domain follows the parent-of relation, and reading a
variant requires reading its template, which is protected by the rule above.

---

## 9. Installation hooks

| Hook | Capability | Effect |
|---|---|---|
| Assign the default nomenclature | barcode | Every company whose nomenclature is unset gets the shipped Default Nomenclature. |
| Enable tracking numbers | expiry | The lot-and-serial-number group is added to the implied groups of the internal-user group and of the portal-user group, so that installing the expiry capability on its own still shows lots. |

---

## 10. Scheduled work

This domain defines no scheduled job of its own. It contributes one **task** to a job owned by
another domain.

| Task | Host job | Owner of the host job | What it does |
|---|---|---|---|
| Expiry alert reminder | The replenishment scheduler | `../replenishment-and-procurement/` | Runs the alert-reminder algorithm of [workflows.md](workflows.md), section 12.4, after the scheduler's own tasks. It also increases the scheduler's declared task count by one, so that progress reporting stays accurate, and reports one unit of progress when the scheduler is running with its own transaction. |

Because the reminder rides on the replenishment scheduler, its frequency is that job's frequency,
and an installation that disables the replenishment scheduler gets no expiry reminders.

---

## 11. Import templates

Two downloadable starting files are offered from the import screen:

| Entity | Label | Path |
|---|---|---|
| Product Template | Import Template for Products | `/product/static/xls/product_product.xls` |
| Vendor Pricelist Line | Import Template for Vendor Pricelists | `/product/static/xls/product_supplierinfo.xls` |

The product file carries the product-values column, which drives the combined
template-and-variant import of [workflows.md](workflows.md), section 9.

---

## 12. Capability dependencies

| Capability | Depends on | Provides |
|---|---|---|
| Products and pricelists | the framework core, messaging, units of measure | Templates, variants, attributes, categories, tags, documents, combos, packaging barcodes, catalog contract, label printing, pricelists |
| Product matrix | accounting (for the section-and-note widget only) | The matrix contract; grants the variant group to every internal user |
| Barcode | the web client | Nomenclatures, rules, the scan contract, the company nomenclature link |
| Barcode — Global Standards One nomenclature | barcode, units of measure | The Global Standards One mode, the content types, the shipped rule set |
| Products expiration date | inventory | The four day counts, the four lot dates, the alert flag, the reminder, the confirmation dialog, the first-expiry-first-out strategy |
| Product e-mail template | accounting, messaging | The per-product message template and the send on invoice posting |
