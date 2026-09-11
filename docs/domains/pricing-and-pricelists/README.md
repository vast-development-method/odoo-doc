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
| Margins | The line cost field and its five computation variants (catalogue cost, stock valuation, manufacturing through the stock variant, timesheets, expenses), the line margin and margin percentage, the order margin and margin percentage, and the product margin analysis with all fifteen of its measures. |
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
| Loyalty programmes, coupons and promotional rewards | `../loyalty-and-promotions/` |
| Delivery charges and carrier price rules | `../delivery-and-shipping/` |
| The storefront pages, cart and checkout | `../website-and-storefront/` |

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

| Entity | Transport name | What this domain adds |
|---|---|---|
| Product Template | `product.template` | The price computation entry points, the contextual price and contextual price list helpers, the vendor relation, the list of available units. |
| Product Variant | `product.product` | The extra price from attributes, the catalogue price including that extra, the vendor selection algorithm, the contextual discount, the price list rules edited from the variant form, and the fifteen margin analysis measures. |
| Contact | `res.partner` | The effective price list and the specific price list assignment. |
| Sales Order Line | `sale.order.line` | The cached price list rule, the unit price derivation, the discount derivation, the cost, the margin and the margin percentage. |
| Sales Order | `sale.order` | The margin and margin percentage totals. |
| Purchase Order Line | `purchase.order.line` | The selected vendor price, the unit price and discount derivation, the planned date derivation, the allowed units. |
| Website | `website` | The published price lists and the visitor price list resolution. |

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
6. **[`state-machines.md`](state-machines.md)** — this domain has no document with a state field;
   the file explains what it has instead (validity windows, activation flags and derived
   applicability states) and diagrams them.
7. **[`configuration.md`](configuration.md)** — the feature flags, the decimal precision records,
   the configuration parameters, the security groups and the access matrix.
8. **[`interfaces.md`](interfaces.md)** — menus, views, named remote operations, routes, the
   printable report and the import templates.
9. **[`accounting-effects.md`](accounting-effects.md)** — this domain posts nothing; the file
   explains precisely how its outputs reach the ledger through other domains.
10. **[`acceptance-criteria.md`](acceptance-criteria.md)** — numbered Given / When / Then scenarios
    with concrete numbers, to verify a rebuild.

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
| Website and Storefront | Resolves a visitor's price list per request and displays struck-through prices using the storefront discount rule. |
| Point of Sale | Loads price lists and their rules into the terminal and re-implements the same algorithm client-side; the two implementations must agree. |
| Events | Ticket prices pass through the price list engine and use the same discount-display rule. |
| [Inventory Valuation and Costing](../inventory-valuation-and-costing/) | Supplies the delivered cost that the stock margin variant uses instead of the catalogue cost. |
| Loyalty and Promotions | Reads the price already computed and adds its own reward lines; it does not participate in rule selection. |

### 5.3 The one-way rule

The price engine never reads a document. It takes a product, a quantity, a unit, a currency and a
date, and returns a number and a rule identifier. It never mutates anything. Every document-shaped
concern — taxes, fiscal positions, down payments, combos, loyalty, delivery — is applied by the
caller **after** the engine returns. A rebuild that lets document concerns leak into the engine
will not be able to reproduce the storefront, the terminal and the order line from one code path.
