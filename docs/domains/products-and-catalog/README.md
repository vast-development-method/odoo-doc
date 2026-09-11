# Products and Catalog

## Scope

This domain specifies the catalog of the system: the things that can be sold, bought, stocked,
manufactured, repaired, invoiced or served at a counter, together with everything that describes,
classifies, identifies, illustrates, prices-by-attribute, groups or dates them.

Concretely the domain covers:

- **Product Templates** — the commercial product as a user thinks of it ("Office Chair"), carrying
  the name, the type, the category, the sales price, the cost, the default unit of measure, the
  images, the descriptions, the tags, the documents, the properties, the company scoping and the
  archival flag.
- **Product Variants** — the individually stockable, orderable and barcode-bearing incarnations of
  a template ("Office Chair, Black, Large"). Every template always owns at least one variant unless
  its attributes are configured to be created on demand.
- **Attributes, attribute values, template attribute lines, template attribute values and
  exclusions** — the configuration data from which variants are generated, priced with extra
  amounts and restricted by incompatibility rules.
- **The variant generation algorithm** — the exact cartesian enumeration, exclusion filtering,
  reactivation, creation, archival and deletion sequence that keeps the set of variants in step
  with the attribute configuration, including the generation limit and its error message.
- **Combination possibility checks** — the predicate that decides whether a given set of attribute
  values may be ordered, including parent-product exclusions, archived combinations and the
  dynamic-creation case, plus the generators that walk possible combinations and repair impossible
  ones.
- **Extra price computation** — how per-value extra prices are summed into a variant's sales price
  and how values that never create a variant contribute their extra price through the evaluation
  context.
- **Product categories** — the hierarchical classification with its complete-name computation,
  cycle protection, product counting and per-category property definitions.
- **Product tags** — the flat, colour-coded, customer-visible labels attached to templates or to
  individual variants.
- **Product documents** — files and links attached to a template or a variant, delegating storage
  to the generic attachment record.
- **Product combos and combo items** — a product type whose price is split over several choice
  groups, with the base price of each group and the proration arithmetic that assigns a share of
  the combo product's price to each chosen item.
- **The product configurator data contract** — the dictionaries the configurator and the storefront
  receive (exclusions, archived combinations, parent exclusions, attribute-name mapping) and the
  free-text custom values a customer may type.
- **Product matrices** — the two-dimensional grid of a template's variants used to enter quantities
  for many variants at once, with its header, cells, possibility flags and extra-price badges.
- **Barcode nomenclatures and rules** — the ordered rule sets that classify a scanned string, the
  pattern grammar with its numeric-content braces, the check-digit arithmetic, the European Article
  Number and Universal Product Code conversions, and the parsing algorithm that yields a type, a
  value and a base code.
- **Global Standards One parsing** — the alternative nomenclature in which a barcode is a
  concatenation of application identifiers and values, the per-identifier content types (date,
  measure, numeric identifier, alphanumeric), the decimal-position digit, the separator handling and
  the decomposition loop, plus the search preprocessing that unpads codes.
- **Uniform resource identifier decoding** — the electronic-product-code style identifiers that are
  translated into product, batch and package data before nomenclature parsing.
- **Packaging barcodes** — the secondary barcodes that bind a unit of measure to a variant, and the
  uniqueness rules shared with product barcodes.
- **Expiry dates and alerts** — the four per-product day counts, the four resulting dates on a lot
  or serial number, their recomputation on change, the expiry alert flag, the scheduled reminder
  activity, the delivery-time confirmation dialog and the effect on available quantity.
- **The catalog mixin data contract** — the shape of the payload any order-like document must
  produce so that products can be added to it from the catalog view.
- **Product email templates** — the per-product message sent to the customer when an invoice is
  posted.
- **Images** — the five stored resolutions on templates and variants, the variant-to-template
  fallback, the zoomability flag and the placeholder.
- **Labels** — the print wizard, its layouts, and the printable label and packaging-barcode
  documents.
- **Access groups, access rights, record rules and settings** of all of the above.

The domain does **not** cover: units of measure themselves, their factors and their conversion
arithmetic (see `../units-of-measure-and-packaging/`); pricelists, pricelist rules, vendor
pricelists and the price computation engine (see `../pricing-and-pricelists/`); stock quantities,
lots as inventory objects, removal strategies and transfers (see `../inventory-operations/`);
product valuation and costing (see `../inventory-valuation-and-costing/`); the sales and purchase
documents that consume the catalog (see `../sales/` and `../purchasing/`); taxes on products (see
`../taxes/`); and the storefront presentation of products (see `../website-and-storefront/`).

Where a field of a catalog entity belongs to another domain (for example the vendor list, the
pricelist rules, the tracking mode or the reordering rules), this folder names the field, says what
the catalog does with it, and points at the owning domain.

## Capabilities covered

| Capability | Where specified |
|---|---|
| Product Template and Product Variant data model, delegation, related-field propagation | `entities.md` |
| Product types (goods, service, combo) and what each enables | `entities.md`, `business-rules.md` |
| Naming and display rules, code prefixing, seller-specific names, search heuristics | `calculations.md`, `interfaces.md` |
| Attribute definitions, variant-creation modes, display types, values, free-text values | `entities.md`, `business-rules.md` |
| Template attribute lines and template attribute values, their creation, reactivation, archival and deletion | `entities.md`, `workflows.md` |
| Exclusions and their inversion | `entities.md`, `calculations.md` |
| Variant generation algorithm, generation limit, archival versus deletion | `calculations.md`, `workflows.md` |
| Combination possibility, first possible combination, closest possible combination, pruned cartesian product | `calculations.md` |
| Extra price arithmetic on variants and on values that never create a variant | `calculations.md` |
| Dynamic variant creation on demand | `workflows.md`, `calculations.md` |
| Categories, complete names, cycle protection, counting, property definitions | `entities.md`, `calculations.md` |
| Tags on templates and on variants, merged tag set | `entities.md`, `calculations.md` |
| Documents on templates and variants, upload route, automatic creation from the message thread | `entities.md`, `interfaces.md`, `workflows.md` |
| Combos, combo items, base price, proration of the combo product price, rounding remainder | `entities.md`, `calculations.md` |
| Configurator payload, custom values | `interfaces.md`, `entities.md` |
| Product matrix grid construction and cell contract | `calculations.md`, `interfaces.md` |
| Barcode nomenclature, rules, pattern grammar, pattern validation messages | `entities.md`, `business-rules.md`, `calculations.md` |
| Check-digit arithmetic and encoding checks | `calculations.md` |
| Nomenclature parsing algorithm with European Article Number and Universal Product Code conversion | `calculations.md` |
| Global Standards One decomposition, application identifiers, content types, decimal position, date conversion | `calculations.md` |
| Uniform resource identifier decoding into product, batch and package data | `calculations.md` |
| Barcode search preprocessing (unpadding) | `calculations.md`, `business-rules.md` |
| Product barcode and packaging barcode uniqueness | `business-rules.md` |
| Expiry day counts, lot dates, alert flag, reminder activity, delivery confirmation, fresh quantity | `entities.md`, `calculations.md`, `workflows.md`, `state-machines.md` |
| Catalog mixin payload and its two remote operations | `interfaces.md` |
| Product email template and the message posted on invoice posting | `workflows.md`, `interfaces.md` |
| Images, resolutions, fallback, zoomability, placeholder | `entities.md`, `calculations.md` |
| Label printing wizard, layouts, copies, printable documents | `interfaces.md`, `workflows.md` |
| Import of products with attribute values from a single file | `workflows.md`, `business-rules.md` |
| Groups, access rights, record rules, settings, decimal precisions, scheduled work | `configuration.md` |
| Every validation and its exact message | `business-rules.md` |
| Numbered acceptance scenarios | `acceptance-criteria.md` |

## Entities

| Entity | Transport name | Storage name | Purpose |
|---|---|---|---|
| Product Template | `product.template` | `product_template` | The commercial product: name, type, category, prices, unit, images, descriptions, tags, documents |
| Product Variant | `product.product` | `product_product` | One concrete combination of attribute values of a template; the record every other domain points at |
| Product Category | `product.category` | `product_category` | Hierarchical classification of templates, and the holder of per-category property definitions |
| Product Tag | `product.tag` | `product_tag` | Flat, colour-coded label attached to templates and to variants |
| Product Attribute | `product.attribute` | `product_attribute` | A dimension of variation (size, colour, legs) with its variant-creation mode and display type |
| Attribute Value | `product.attribute.value` | `product_attribute_value` | One possible value of an attribute, with default extra price, colour, image and free-text flag |
| Template Attribute Line | `product.template.attribute.line` | `product_template_attribute_line` | The use of one attribute on one template, with the subset of values selected |
| Template Attribute Value | `product.template.attribute.value` | `product_template_attribute_value` | The materialised pair (template attribute line, attribute value) carrying the per-template extra price and exclusions |
| Template Attribute Exclusion | `product.template.attribute.exclusion` | `product_template_attribute_exclusion` | A rule making one template attribute value incompatible with a set of others on a given template |
| Attribute Custom Value | `product.attribute.custom.value` | `product_attribute_custom_value` | The free text a customer typed for a value flagged as free text |
| Product Combo | `product.combo` | `product_combo` | One choice group of a combo product, with its items and its base price |
| Product Combo Item | `product.combo.item` | `product_combo_item` | One selectable variant inside a choice group, with its extra price |
| Product Document | `product.document` | `product_document` | A file or link attached to a template or a variant, delegating to the generic attachment record |
| Packaging Barcode | `product.uom` | `product_uom` | The binding of one barcode to one (variant, unit of measure) pair |
| Product Catalog Mixin | `product.catalog.mixin` | none (abstract) | The contract an order-like document implements to be fed from the catalog view |
| Barcode Nomenclature | `barcode.nomenclature` | `barcode_nomenclature` | An ordered rule set interpreting scanned strings, in classic or Global Standards One mode |
| Barcode Rule | `barcode.rule` | `barcode_rule` | One pattern of a nomenclature with its encoding, result type and numeric-content configuration |
| Barcode Event Mixin | `barcodes.barcode_events_mixin` | none (abstract) | The contract a form implements to react to a scan |
| Label Layout | `product.label.layout` | none (transient) | The print wizard: format, number of copies, extra content, pricelist |
| Attribute Value Bulk Update | `update.product.attribute.value` | none (transient) | The wizard adding a value to every template using its attribute, or pushing its default extra price |
| Expiry Delivery Confirmation | `expiry.picking.confirmation` | none (transient) | The dialog shown when a delivery contains expired lots |

Entities owned by other domains that this domain extends with catalog fields, and whose extension
is specified here: Company (`res.company`, barcode nomenclature and pricelist bootstrapping),
Configuration Settings (`res.config.settings`), Currency (`res.currency`), Country Group
(`res.country.group`), Partner (`res.partner`), Unit of Measure (`uom.uom`), Attachment
(`ir.attachment`), Lot or Serial Number (`stock.lot`, expiry dates), Stock Move (`stock.move`),
Stock Move Line (`stock.move.line`), Stock Quantity (`stock.quant`), Transfer (`stock.picking`),
Removal Strategy (`product.removal`), Journal Entry (`account.move`, product email template) and
Message Template (`mail.template`).

## Reading order

1. `glossary.md` — the vocabulary (template, variant, combination, template attribute value,
   exclusion, dynamic attribute, no-variant attribute, nomenclature, application identifier,
   check digit, expiry alert, combo base price).
2. `entities.md` — the data model, field by field.
3. `calculations.md` — the algorithms: variant generation, combination possibility, extra prices,
   combo proration, barcode parsing, check digits, Global Standards One decomposition, expiry dates.
   Read it in order; later sections depend on earlier ones.
4. `state-machines.md` — the small number of state-like fields in this domain and their transitions.
5. `workflows.md` — the end-to-end operational sequences.
6. `business-rules.md` — every validation, constraint and message.
7. `configuration.md` — groups, access rights, record rules, settings, precisions, default records.
8. `interfaces.md` — navigation, views, remote operations, routes, printable documents, templates.
9. `accounting-effects.md` — why this domain posts nothing itself and what it feeds.
10. `acceptance-criteria.md` — the numbered scenarios an implementation must satisfy.

## Dependencies on other domains

| Depends on | For what |
|---|---|
| `../units-of-measure-and-packaging/` | The unit of measure record, the relative and absolute factors, quantity and price conversion, the multiple-units group, the `Product Unit` precision used to compare quantities |
| `../pricing-and-pricelists/` | Pricelists, pricelist rules, vendor pricelists and the price computation the catalog delegates to; the catalog only supplies the base price, the cost and the attribute extra price |
| `../multi-currency/` | Currency conversion used by the combo base price, by the matrix header cells and by the vendor selection ordering |
| `../messaging-and-activities/` | The message thread and activity mixins on templates and variants, the activity scheduled by the expiry reminder, and the message template posted on invoice posting |
| `../inventory-operations/` | Lots and serial numbers, stock quantities, transfers and moves that the expiry capability extends; also the barcode rules the inventory domain registers into the default nomenclature |
| `../accounts-receivable/` | The customer invoice whose posting triggers the per-product email |
| `../automation-and-integration/` | The generic attachment record behind product documents, the properties mechanism, and the scheduled-work runner that triggers the expiry reminder |

## Dependents

Every operational domain reads this one. The most tightly coupled are `../sales/` (configurator,
combos, matrices, optional products), `../purchasing/` (matrices, vendor references),
`../point-of-sale/` (barcode scanning, combos), `../inventory-operations/` (variants, barcodes,
packaging barcodes, expiry) and `../manufacturing/` (attribute-restricted component lines).

## Document map

| File | What it settles |
|---|---|
| [glossary.md](glossary.md) | Every term of the domain, defined in full, with the transport and storage names of every entity |
| [entities.md](entities.md) | The complete data model: twenty-one own entities plus fourteen extensions of entities owned elsewhere, each with a full field table, lifecycle, ordering, display rule, uniqueness rules and company behaviour |
| [calculations.md](calculations.md) | Fifteen groups of algorithms and formulas: combinations, value materialisation, variant generation, the four possibility predicates, the pruned cartesian product, extra prices, combo proration, the matrix, the classic barcode parser, the Global Standards One decomposer, names and displays, expiry dates, vendor selection, and the rounding summary |
| [state-machines.md](state-machines.md) | The ten state-like fields of the domain, each with a state table, a transition table and a diagram, plus the two interaction state machines (transfer completion with expired goods; the barcode parse outcome) |
| [workflows.md](workflows.md) | Fourteen end-to-end operational sequences, from creating a product to importing a catalogue with attribute values from one file |
| [business-rules.md](business-rules.md) | Every validation classified by when it fires and what it does, with the exact message; the complete possibility rule set; the permission model; the ordering invariants; fifteen explicitly stated edge cases |
| [configuration.md](configuration.md) | Settings, four system parameters, four decimal precisions, every shipped default record including the twenty-six Global Standards One rules, five security groups, the full access matrix, six record rules, two installation hooks and the one scheduled task |
| [interfaces.md](interfaces.md) | Window actions, views field by field, sixteen named remote operations, four web routes, the catalog data contract, seven printable documents, the message hook, the five client-side contracts, and the import and export formats |
| [accounting-effects.md](accounting-effects.md) | Why this domain posts nothing, and the exact list of catalog values the financial domains consume |
| [acceptance-criteria.md](acceptance-criteria.md) | Twenty-two groups of numbered Given/When/Then scenarios with concrete numbers, ending in seven cross-cutting invariants |

## Worked examples index

The numeric examples an implementer should reproduce first, and where they are:

| Example | Where |
|---|---|
| A template with a size attribute and a colour attribute generating six variants with extra prices | [calculations.md](calculations.md) §1.2 and §3.2; [acceptance-criteria.md](acceptance-criteria.md) A-3, A-4 |
| An excluded combination, and the inverse-completed exclusion map | [calculations.md](calculations.md) §3.3 and §4.3; [acceptance-criteria.md](acceptance-criteria.md) A-5, A-7, C-9 |
| A dynamically created variant, including the archived case | [calculations.md](calculations.md) §5.3; [acceptance-criteria.md](acceptance-criteria.md) B-1 to B-8 |
| Extra prices of values that never create a variant | [calculations.md](calculations.md) §7.2; [acceptance-criteria.md](acceptance-criteria.md) D-2, D-3 |
| A combination product's price, with and without a rounding remainder | [calculations.md](calculations.md) §8.3; [acceptance-criteria.md](acceptance-criteria.md) E-3 to E-5 |
| The closest possible combination when the requested one is excluded | [calculations.md](calculations.md) §6.3; [acceptance-criteria.md](acceptance-criteria.md) C-11 |
| A matrix grid with an unavailable cell | [calculations.md](calculations.md) §9; [acceptance-criteria.md](acceptance-criteria.md) R-1, R-2 |
| A check digit computed digit by digit | [calculations.md](calculations.md) §10.1; [acceptance-criteria.md](acceptance-criteria.md) F-1 |
| Parsing a weight barcode, from the printed label to the stored zeroed base code | [calculations.md](calculations.md) §10.6; [acceptance-criteria.md](acceptance-criteria.md) F-3, F-4 |
| Decoding the four uniform resource identifier shapes | [calculations.md](calculations.md) §10.7; [acceptance-criteria.md](acceptance-criteria.md) F-12 to F-15 |
| Parsing a barcode carrying the product, batch and expiry application identifiers | [calculations.md](calculations.md) §11.7; [acceptance-criteria.md](acceptance-criteria.md) G-2 to G-4 |
| The decimal-position digit of a weight application identifier | [calculations.md](calculations.md) §11.8; [acceptance-criteria.md](acceptance-criteria.md) G-7 |
| The century-determination rule for a six-digit date | [calculations.md](calculations.md) §11.5; [acceptance-criteria.md](acceptance-criteria.md) G-10 |
| Search unpadding under a Global Standards One nomenclature | [calculations.md](calculations.md) §11.9; [acceptance-criteria.md](acceptance-criteria.md) G-13 to G-18 |
| An expiry date computation, including a postponement that preserves a manual adjustment | [calculations.md](calculations.md) §13.4; [acceptance-criteria.md](acceptance-criteria.md) O-1 to O-3 |
| The label sheet page count | [calculations.md](calculations.md) §14.2; [acceptance-criteria.md](acceptance-criteria.md) N-4 |

## Invariants at a glance

1. Within one template, no two **active** variants share the same combination.
2. Every active, non-dynamic template has at least one variant.
3. A dynamic template may legitimately have none.
4. Within one company, a barcode identifies at most one product, and never both a product and a
   packaging binding; across the whole installation, a barcode identifies at most one packaging
   binding.
5. The sum of the prorated shares of a combination product's choice groups equals the combination
   product's price exactly, and the combination product's own line carries a unit price of zero.
6. A Global Standards One decomposition either consumes the whole scanned string or returns nothing.
7. Postponing a lot's expiration date moves every non-empty derived date by the same amount, so the
   offsets between the four dates are preserved.
8. Nothing in this domain writes a journal entry.
