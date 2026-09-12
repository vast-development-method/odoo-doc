# Pricing and Price Lists

This domain owns **the answer to one question**: given a product, a quantity, a unit of measure, a
currency, a date and a price list, what is the unit price?

Every selling document in the platform — sales quotations and orders, storefront carts, point of
sale tickets, event ticket lines, subscription lines — asks that question, and every buying
document asks its mirror image through the vendor price list. The computation is small in code and
large in consequence: a single misplaced rounding step or a rule chosen in the wrong order changes
the amount a customer is charged, the margin reported to management, and the revenue posted to the
ledger.

The domain also owns the **margin** computations that compare what was charged against what the
goods cost, in each of the five variants the platform ships.

---

## 1. Scope

### 1.1 In scope

| Area | What is specified here |
|---|---|
| Price lists | The Price List entity, its currency, its company, its country groups, its rules, its ordering, its display name, its deletion guard. |
| Price list rules | The Price List Rule entity in full: every field, every selection value, the four applicability levels, the three computation kinds, the three bases, the validity window, the minimum quantity, the discount, the surcharge, the rounding step, the two margin bounds, the markup mirror field. |
| The price computation algorithm | Rule gathering, rule ordering and specificity, quantity conversion into the product's own unit, date validity, applicability tests, the base price, recursion into another price list with cycle protection, currency conversion and the date used, the formula order (discount, then rounding, then surcharge, then minimum margin, then maximum margin), unit conversion of every monetary rule parameter, and the rule identifier returned alongside the price. |
| Price list selection | For a partner (the specific assignment, the country group match, the fallback chain, the configuration parameter fallback), and for a storefront visitor (geolocated country, session memory, promotional code, selectable flag, website and company scoping). |
| The discount policy | How a sales line turns a price list result into a displayed unit price plus a percentage discount, which rule kinds are allowed to reveal a discount, and the storefront variant of that rule. |
| Vendor price lists | The Vendor Price entity, the seller filtering algorithm, the seller ordering algorithm, the discounted price, the lead time, the vendor's own unit, and how a purchase line derives its unit price and discount from the selected seller — including the fall back to the product cost. |
| Vendor price learning | The rule that adds a vendor to a product when a purchase order is confirmed. |
| Margins | The line cost field and its five computation variants (catalogue cost, stock valuation, manufacturing through the stock variant, timesheets, expenses), the line margin and margin percentage, the order margin and margin percentage, and the product margin analysis with all fourteen of its measures and the three echoes of its calling context. |
| Rounding and precision | Which decimal precision record governs each stored amount, where rounding happens and where it deliberately does not. |
| The price list report | The comparison report that prints several price lists side by side at several quantities. |

### 1.2 Out of scope, and where it lives

| Subject | Where it is specified |
|---|---|
| The catalogue price field, the cost field, attribute extra prices, product categories, variants | [`../products-and-catalog/`](../products-and-catalog/) |
| Unit conversion, price conversion, the rounding function, unit trees and factors | [`../units-of-measure-and-packaging/calculations.md`](../units-of-measure-and-packaging/calculations.md) |
| Currency rates, the rate lookup for a date and company, the conversion arithmetic | [`../multi-currency/`](../multi-currency/) |
| Taxes, tax-included prices, fiscal position tax mapping | [`../taxes/`](../taxes/) |
| Sales order and sales order line lifecycle, amounts, invoicing | [`../sales/`](../sales/) |
| Purchase order and purchase order line lifecycle, receipt and bill statuses | [`../purchasing/`](../purchasing/) |
| Stock valuation layers, the cost methods, cost of goods sold | [`../inventory-valuation-and-costing/`](../inventory-valuation-and-costing/) |
| Loyalty programmes, coupons and promotional rewards | [`../loyalty-and-promotions/`](../loyalty-and-promotions/) |
| Delivery charges and carrier price rules | [`../delivery-and-shipping/`](../delivery-and-shipping/) |
| The storefront pages, cart and checkout | [`../website-and-storefront/`](../website-and-storefront/) |

Loyalty and delivery both compute money, but neither uses the price list engine: they are separate
engines and are documented with their own domains. This domain is cited by both.

---

## 2. Capabilities covered

1. Define any number of price lists, each in one currency, each optionally restricted to one
   company and to a set of country groups.
2. Define, inside a price list, an ordered set of rules that each target all products, one product
   category and its descendants, one product template, or one product variant.
3. Restrict a rule to a minimum quantity, expressed in the product's own unit, and to a validity
   window with a start instant and an end instant.
4. Compute a rule's price as a fixed amount, as a percentage discount off a base, or with the full
   formula: a base, a percentage discount or markup, a rounding step, an additive surcharge, a
   minimum margin and a maximum margin.
5. Choose the base of a rule: the catalogue sales price, the product cost, or the price another
   price list would give — recursively, with a guard that forbids cycles.
6. Return, along with the price, the identifier of the rule that produced it, so that documents can
   explain and re-derive the price.
7. Express the result in any currency and any unit of measure, converting both, using an explicit
   date for the currency rate.
8. Attach a price list to a partner, directly or by country group, with a documented fallback
   chain, and inherit it down a partner's company hierarchy.
9. Select a price list for an anonymous storefront visitor from the visitor's geolocated country,
   the session, or a promotional code, restricted to the price lists the website publishes.
10. Turn a price list result into a displayed unit price and a percentage discount on a sales line,
    under a configurable discount policy.
11. Record vendor prices per product, per vendor, per vendor unit, per currency, per validity
    window and per minimum quantity, with a per-line percentage discount and a lead time.
12. Select the applicable vendor price for a product, a vendor, a quantity, a unit and a date, and
    order candidates so that the cheapest wins.
13. Learn a vendor price automatically from a confirmed purchase order.
14. Compute the cost, margin and margin percentage of a sales line and of a sales order, under five
    different cost sources.
15. Analyse the realised and expected margin of a product over a date range from posted invoices
    and bills.
16. Print a comparison report of several price lists at several quantities.

---

## 3. Entities

| Entity | Transport name | Storage table | One-line purpose |
|---|---|---|---|
| Price List | `product.pricelist` | `product_pricelist` | A named, currency-bearing container of pricing rules, optionally scoped to a company, a set of country groups and a website. |
| Price List Rule | `product.pricelist.item` | `product_pricelist_item` | One rule of a price list: what it applies to, when it is valid, from what quantity, and how it computes a price. |
| Vendor Price | `product.supplierinfo` | `product_supplierinfo` | One vendor's offer for one product: price, currency, unit, minimum quantity, validity window, discount and lead time. |
| Country Group | `res.country.group` | `res_country_group` | A named set of countries; used here to restrict a price list to a geography. Owned by the contacts domain, extended here with the reverse relation to price lists. |
| Product Margin Wizard | `product.margin` | *(transient, no table)* | A short-lived form that collects a date range and an invoice-state filter and opens the product margin analysis. |

Entities this domain **extends** rather than owns:

| Entity | Transport name | Owner | What this domain adds |
|---|---|---|---|
| Product Template | `product.template` | [products and catalog](../products-and-catalog/) | The price computation entry points, the contextual price and contextual price list helpers, the vendor relation, the price list rule relation, the list of available units. |
| Product Variant | `product.product` | [products and catalog](../products-and-catalog/) | The extra price from attributes, the catalogue price including that extra, the vendor selection algorithm, the contextual discount, the price list rules edited from the variant form, and the seventeen margin analysis fields. |
| Contact | `res.partner` | [contacts and organizations](../contacts-and-organizations/) | The effective price list and the specific price list assignment. |
| Company | `res.company` | [platform foundation](../platform-foundation/) | No field: the provisioning of a default price list on creation and the deferral of that step across a currency change. |
| Currency | `res.currency` | [multi-currency](../multi-currency/) | No field: archiving a currency archives its price lists, and granting multi-currency grants the price list capability. |
| Sales Order | `sale.order` | [sales](../sales/) | The price list, the currency derived from it, the two indicator flags, and the margin totals. |
| Sales Order Line | `sale.order.line` | [sales](../sales/) | The cached price list rule, the unit price derivation, the discount derivation, the cost, the margin and the margin percentage. |
| Purchase Order Line | `purchase.order.line` | [purchasing](../purchasing/) | The selected vendor price, the unit price and discount derivation, the planned date derivation, the allowed units, the suggested quantity. |
| Reordering Rule | `stock.warehouse.orderpoint` | [replenishment and procurement](../replenishment-and-procurement/) | The pinned vendor price, the effective vendor, the placeholder showing the automatic choice, and the vendor search helpers. |
| Product Replenish Wizard | `product.replenish` | [replenishment and procurement](../replenishment-and-procurement/) | The vendor price chosen for a one-off replenishment. |
| Point of Sale Configuration | `pos.config` | [point of sale](../point-of-sale/) | The default price list, the available price lists and the flag that offers a choice, with their company and currency constraints. |
| Website | `website` | [website and storefront](../website-and-storefront/) | The published price lists and the visitor price list resolution. |
| Loyalty Programme | `loyalty.program` | [loyalty and promotions](../loyalty-and-promotions/) | No field: the guard that refuses to archive a price list an active programme names. |

### 3.1 Candidate entities this folder does **not** own

The working scope of this domain listed twenty-seven candidate entities, because they are shipped by
the same capability package as the Price List. Only the five above belong here. The rest are the
catalogue itself and its printing, and they are specified by
[products and catalog](../products-and-catalog/):

| Transport name | Full name | Where it is specified |
|---|---|---|
| `product.template` | Product Template | [products and catalog](../products-and-catalog/) |
| `product.product` | Product Variant | [products and catalog](../products-and-catalog/) |
| `product.category` | Product Category | [products and catalog](../products-and-catalog/) |
| `product.attribute` | Product Attribute | [products and catalog](../products-and-catalog/) |
| `product.attribute.value` | Product Attribute Value | [products and catalog](../products-and-catalog/) |
| `product.attribute.custom.value` | Product Attribute Custom Value | [products and catalog](../products-and-catalog/) |
| `product.template.attribute.line` | Product Template Attribute Line | [products and catalog](../products-and-catalog/) |
| `product.template.attribute.value` | Product Template Attribute Value | [products and catalog](../products-and-catalog/) |
| `product.template.attribute.exclusion` | Product Template Attribute Exclusion | [products and catalog](../products-and-catalog/) |
| `update.product.attribute.value` | Update Product Attribute Value Wizard | [products and catalog](../products-and-catalog/) |
| `product.combo` | Product Combo | [products and catalog](../products-and-catalog/) |
| `product.combo.item` | Product Combo Item | [products and catalog](../products-and-catalog/) |
| `product.document` | Product Document | [products and catalog](../products-and-catalog/) |
| `product.tag` | Product Tag | [products and catalog](../products-and-catalog/) |
| `product.catalog.mixin` | Product Catalogue Mixin | [products and catalog](../products-and-catalog/) |
| `product.uom` | Product Unit Line | [units of measure and packaging](../units-of-measure-and-packaging/) |
| `product.label.layout` | Product Label Layout Wizard | [products and catalog](../products-and-catalog/) |
| `report.product.report_pricelist` | Price List Grid Report | this folder — see [`interfaces.md`](interfaces.md#3-printable-documents); the report carries no stored record |
| `report.product.report_producttemplatelabel2x7` | Product Label Sheet, two by seven | [products and catalog](../products-and-catalog/) |
| `report.product.report_producttemplatelabel4x7` | Product Label Sheet, four by seven | [products and catalog](../products-and-catalog/) |
| `report.product.report_producttemplatelabel4x12` | Product Label Sheet, four by twelve | [products and catalog](../products-and-catalog/) |
| `report.product.report_producttemplatelabel4x12noprice` | Product Label Sheet, four by twelve, priceless | [products and catalog](../products-and-catalog/) |
| `report.product.report_producttemplatelabel_dymo` | Product Label Sheet, single-roll | [products and catalog](../products-and-catalog/) |

The price a label sheet prints is the contextual price of this domain; the sheet itself is not.

---

## 3.2 Who does what

| Actor | Role in this domain |
|---|---|
| Sales administrator | Creates, edits and deletes price lists and price list rules; switches the Basic Price Lists and Discounts capabilities on and off. |
| Product manager | Creates and edits price lists, price list rules and vendor prices from the catalogue; the only role besides the purchase manager that may delete a vendor price. |
| Salesperson | Reads price lists, applies one to a quotation, triggers *Update Prices*, overrides a unit price by hand, reads the line cost and margin. |
| Purchase administrator | Creates, edits and deletes vendor prices; maintains the vendor side of the catalogue. |
| Purchase user | Reads vendor prices through purchase order lines and replenishment screens. |
| Internal user | Reads every price list, rule and vendor price the record rules allow, and sees the price list's promotional code and the product's cost. |
| Point of sale manager and cashier | Configures the price lists of a terminal, or switches between the available ones during a session. |
| Portal and public visitor | Reads price lists and rules only where the storefront capability is installed; is priced correctly in every case, because resolution and computation run with elevated rights. |
| The platform itself | Provisions default price lists, archives price lists when a currency is archived or the capability is switched off, learns vendor prices at purchase confirmation, and selects vendor prices for replenishment. |

---

## 4. Reading order

1. **[`glossary.md`](glossary.md)** — read first if any term below is unfamiliar. Every term used
   in this folder is defined there in full words.
2. **[`entities.md`](entities.md)** — the three stored entities field by field, with defaults,
   computations, relations, ordering and multi-company behaviour.
3. **[`calculations.md`](calculations.md)** — **the heart of the domain.** The price computation
   algorithm end to end, with every formula, every rounding decision and eleven worked numeric
   examples. Nothing else in this folder makes sense without it.
4. **[`business-rules.md`](business-rules.md)** — every validation, every constraint, every error
   message, every permission check and every edge case.
5. **[`workflows.md`](workflows.md)** — the operational sequences: configuring a price list,
   pricing a sales line, pricing a purchase line, learning a vendor price, running the comparison
   report.
6. **[`state-machines.md`](state-machines.md)** — this domain has no document with a stored state
   field; the file explains what it has instead (validity windows, activation flags, derived
   applicability states, price provenance and the storefront session) and diagrams each of them.
7. **[`configuration.md`](configuration.md)** — the feature flags, the decimal precision records,
   the configuration parameters, the security groups and the access matrix.
8. **[`interfaces.md`](interfaces.md)** — menus, views, named remote operations, routes, the
   printable report and the import templates.
9. **[`accounting-effects.md`](accounting-effects.md)** — this domain posts nothing; the file
   explains precisely how its outputs reach the ledger through other domains.
10. **[`acceptance-criteria.md`](acceptance-criteria.md)** — numbered Given / When / Then scenarios
    with concrete numbers, to verify a rebuild.

### 4.1 Every file in this folder

| File | Content |
|---|---|
| [`README.md`](README.md) | This page: scope, capabilities, entities, reading order, dependencies, actors and the file list. |
| [`entities.md`](entities.md) | Price List, Price List Rule, Vendor Price and the Product Margin Wizard field by field; every field this domain adds to entities owned elsewhere; uniqueness, ordering, display names, indexes, archival and multi-company behaviour. |
| [`state-machines.md`](state-machines.md) | The four derived lifecycles, the two archival lifecycles, the storefront session lifecycle and the invoice-state filter, each with states, transitions, guards and a diagram. |
| [`workflows.md`](workflows.md) | Twenty-three end-to-end procedures, from switching the capability on to reviewing the margin of a quotation. |
| [`business-rules.md`](business-rules.md) | Every validation, constraint, invariant, permission check and edge case, numbered `PR-nnn`, with the exact refusal text; the index of rule identifiers; the mapping from the two earlier numbering schemes; the reconciliation notes. |
| [`calculations.md`](calculations.md) | The price engine end to end, the vendor selection, the margins and the product margin analysis, with every formula, every rounding decision and the worked numeric examples. |
| [`accounting-effects.md`](accounting-effects.md) | Why this domain posts nothing, and exactly how its outputs reach the ledger through other domains. |
| [`configuration.md`](configuration.md) | Capabilities, settings, decimal precisions, configuration parameters, shipped records, access rights, record rules and scheduled jobs. |
| [`interfaces.md`](interfaces.md) | Named operations, the request route, the printable grid, the exports, the messages, the screens and the terminal payload. |
| [`acceptance-criteria.md`](acceptance-criteria.md) | Numbered Given / When / Then scenarios with concrete records, inputs and results. |
| [`glossary.md`](glossary.md) | Every term of the domain, defined. |

---

## 5. Dependencies on other domains

### 5.1 Hard dependencies — the engine cannot run without them

| Domain | What is needed |
|---|---|
| [Units of Measure and Packaging](../units-of-measure-and-packaging/) | The quantity conversion operation and the price conversion operation, with their exact rounding behaviour, and the rounding function used for the rule's rounding step. The price engine calls quantity conversion once per priced product and price conversion up to four times per formula rule. |
| [Multi-currency](../multi-currency/) | The currency record with its rounding, and the conversion operation for an amount, a source currency, a destination currency, a company and a date. The price engine always converts **unrounded**. |
| [Products and Catalog](../products-and-catalog/) | The catalogue sales price, the cost, the product's own unit, the product category tree with its materialised path, the variant extra prices, and the template-to-variant relation. |
| [Contacts and Organizations](../contacts-and-organizations/) | The contact record, the parent-child company hierarchy used to propagate a price list, and the country and country group records. |

### 5.2 Consumers — domains that call this one

| Domain | How it calls |
|---|---|
| [Sales](../sales/) | Every sales order line asks for a price and a rule, then derives a displayed price and a discount. |
| [Purchasing](../purchasing/) | Every purchase order line selects a vendor price and derives a unit price, a discount and a planned date. Confirmation writes vendor prices back. |
| [Website and Storefront](../website-and-storefront/) | Resolves a visitor's price list per request and displays struck-through prices using the storefront discount rule. |
| [Point of Sale](../point-of-sale/) | Loads price lists and their rules into the terminal and re-implements the same algorithm client-side; the two implementations must agree. |
| [Events](../events/) | Ticket prices pass through the price list engine and use the same discount-display rule. |
| [Replenishment and Procurement](../replenishment-and-procurement/) | The buy rule selects a vendor price for every procurement, takes its lead time, and pins one on a reordering rule. |
| [Inventory Valuation and Costing](../inventory-valuation-and-costing/) | Supplies the delivered cost that the stock margin variant uses instead of the catalogue cost. |
| [Manufacturing](../manufacturing/) | Subcontracting reads the vendor prices of the subcontractors of a product; the manufacturing margin bridge routes production cost into the line cost. |
| [Timesheets](../timesheets/) | Supplies the analytic cost that the timesheet margin variant uses instead of the catalogue cost. |
| [Loyalty and Promotions](../loyalty-and-promotions/) | Reads the price already computed and adds its own reward lines; it does not participate in rule selection, and it forbids archiving a price list an active programme names. |
| [Taxes](../taxes/) | Supplies the tax-inclusion correction the sales line and the purchase line apply to the engine's answer. |

### 5.3 The one-way rule

The price engine never reads a document. It takes a product, a quantity, a unit, a currency and a
date, and returns a number and a rule identifier. It never mutates anything. Every document-shaped
concern — taxes, fiscal positions, down payments, combos, loyalty, delivery — is applied by the
caller **after** the engine returns. A rebuild that lets document concerns leak into the engine
will not be able to reproduce the storefront, the terminal and the order line from one code path.

---

## 6. Reconciliation notes

This folder was assembled from two independently written descriptions of the same domain. Where they
disagreed, the platform's behaviour decided, and each resolution is recorded at the end of the file
it affects: [`entities.md`](entities.md#7-reconciliation-notes),
[`state-machines.md`](state-machines.md#8-reconciliation-notes),
[`business-rules.md`](business-rules.md#24-reconciliation-notes),
[`calculations.md`](calculations.md#22-reconciliation-notes),
[`workflows.md`](workflows.md#24-reconciliation-notes),
[`interfaces.md`](interfaces.md#9-reconciliation-notes),
[`configuration.md`](configuration.md#13-reconciliation-notes),
[`glossary.md`](glossary.md#11-reconciliation-notes) and
[`acceptance-criteria.md`](acceptance-criteria.md#25-reconciliation-notes). Three resolutions change
the shape of this page and are recorded here:

1. **The name of the domain.** One description called it "Pricing, Pricelists and Discounts". The
   discount policy is one paragraph of behaviour, not a third subject, and the folder key is
   `pricing-and-pricelists`; the title used everywhere is **Pricing and Price Lists**, and the two
   words "price list" are always written apart in prose because the platform's own compound spelling
   is an abbreviation of them. Stored identifiers keep the compound spelling, as they must.

2. **Which entities the folder owns.** One description listed four owned entities and counted the
   comparison grid among them; the other listed five and counted the Product Margin Wizard. The grid
   has no record of any kind and is a printable document, specified in
   [`interfaces.md`](interfaces.md#3-printable-documents); the wizard is a real, transient entity and
   is specified in [`entities.md`](entities.md#5-product-margin-wizard). Section 3 above lists five
   owned entities on that basis.

3. **The count of margin measures.** Both descriptions said "fifteen". There are fourteen numeric
   measures and three echoes of the calling context; thirteen of the fourteen can be summed in a
   grouped list. Sections 1.1 and 3 use the corrected counts.
