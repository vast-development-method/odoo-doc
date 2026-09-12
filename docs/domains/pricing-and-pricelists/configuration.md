# Pricing and Price Lists — Configuration

Every setting, capability, decimal precision, configuration parameter, shipped record, access right,
record rule, menu and scheduled job the domain needs or defines, with the data type, the default and
the effect of each.

---

## 1. Feature settings

These appear in the sales settings screen, in the pricing block. They are **not** stored as rows of
a settings table: each is a granted capability, and the settings form is only a convenient way to
grant or withdraw it.

| Setting label | Type | Shipped state | Backing capability | Effect when granted | Effect when withdrawn |
|---|---|---|---|---|---|
| "Pricelists" | boolean | off in a database without demonstration data; forced on as soon as the multi-currency capability is granted | Basic Price Lists | The price list menus appear; the price list field appears on quotations, on contacts and on point of sale configurations; the pricing tab appears on products; the default price list of every company is provisioned | **Every active price list is archived**; the contact resolution returns nothing; the storefront resolution returns nothing |
| "Discounts" | boolean | off | Discount per Sales Order Line | The discount column appears on sales order lines and on quotation print-outs; percentage rules present themselves as a discount percentage instead of a reduced unit price. Turning it on also turns the Pricelists setting on | No line ever shows a price list discount; the whole reduction is folded into the unit price |
| "Units of Measure & Packagings" | boolean | off | Units of Measure | The unit fields this domain uses become visible: the unit on a vendor price, the unit column on purchase order lines, the unit column of the comparison report | The fields are hidden. Prices are still converted between units, because every product always has an own unit |
| "Variants" | boolean | off | Manage Product Variants | The product variant field appears on a price list rule and the variant column on the vendor price list, which is what allows rules and offers scoped to one variant | Variant-scoped rules already stored keep working; they simply cannot be edited in the interface |
| "Multi-Currency" | boolean | off | Multi-Currency | The currency field appears on a price list and on a vendor price. Granting it also grants Basic Price Lists and runs the provisioning of every company | The fields are hidden; every price list keeps its currency |
| "Multi-Company" | boolean | off | Multi Company | The company field appears on a price list, on a price list rule and on a vendor price | The fields are hidden; the record rules still apply |

**How "enabled" is decided, and why it differs between the two main flags.**

| Flag | Test performed | Consequence |
|---|---|---|
| Basic Price Lists, in the **contact resolution** and in the **storefront resolution** | whether the **superuser** holds the capability — that is, "is this capability switched on in this database" | The answer does not depend on who is acting, so a portal visitor and a salesperson resolve the same way |
| Basic Price Lists, in the **provisioning** of company default price lists | whether the **acting user** holds the capability | Provisioning triggered by a user who does not hold it does nothing, even in a database where the capability is on for others |
| Discount per Sales Order Line | whether the **superuser** holds the capability | A portal visitor and a salesperson see the same split between price and discount |

A rebuild must reproduce both tests. Unifying them changes storefront prices.

### 1.1 Settings owned by other domains that change behaviour specified here

| Setting | Owner | Effect on pricing |
|---|---|---|
| Fiscal positions and their tax mappings | [taxes](../taxes/) | Decide whether a sales line's unit price is re-expressed after the price list produced it, and which purchase taxes are stripped from a vendor price |
| Currency rates and their provider | [multi-currency](../multi-currency/) | Decide every cross-currency price and every vendor ranking across currencies |
| The costing method of a product category | [inventory valuation and costing](../inventory-valuation-and-costing/) | Decides how the cost a cost-based rule reads is maintained, and whether the stock margin variant replaces the line cost |
| The project time unit of a company | [projects and tasks](../projects-and-tasks/) | The unit the timesheet margin variant converts a cost into |

---

## 2. Capabilities

| Capability | Held by default | What it allows in this domain |
|---|---|---|
| Basic Price Lists | nobody until the setting is turned on, then every internal user | See and use price lists at all |
| Manage Product Variants | nobody until the setting is turned on | Scope a rule or an offer to one variant through the interface |
| Products / Create | the system administrator and the root user; implied by the System capability | Create, modify and delete Price List, Price List Rule and Vendor Price |
| Discount per Sales Order Line | nobody until the setting is turned on | See and edit the discount percentage on sales lines, and receive percentage rules as a visible discount |
| Salesperson | sales users | Read price lists |
| Sales Manager | sales managers | Full rights on Price List and Price List Rule |
| Purchase Manager | purchase managers | Full rights on Vendor Price and Price List Rule |
| Point of Sale User | cashiers | Read Price List and Vendor Price |
| Point of Sale Manager | point of sale managers | Full rights on Price List |
| Inventory Manager | inventory managers | Full rights on Price List |
| Manufacturing Manager | manufacturing managers | Read Vendor Price, full rights on Price List Rule |
| Contacts Manager | contacts managers | Read Price List, in order to assign one on a contact form |
| Accounting User | accounting users | Create, read and update the Product Margin Wizard |
| Portal | portal users | Read Price List and Price List Rule, so that storefront prices can be computed for a signed-in customer |
| Public | anonymous storefront visitors | Read Price List and Price List Rule, so that storefront prices can be computed for an anonymous visitor |
| Internal User | every employee | Read Price List, Price List Rule and Vendor Price |
| Technical Features | opt-in | Show the two margin fields and the company-settings group on a price list rule form |

---

## 3. Access rights matrix

Read, create, update and delete per entity and capability. A blank cell means the capability grants
nothing on that entity; rights accumulate across the capabilities a user holds.

| Capability | Price List | Price List Rule | Vendor Price | Product Margin Wizard |
|---|---|---|---|---|
| Internal User | read | read | read | |
| Contacts Manager | read | | | |
| Salesperson | read | | | |
| Sales Manager | read, create, update, delete | read, create, update, delete | | |
| Purchase Manager | | read, create, update, delete | read, create, update, delete | |
| Products / Create | read, create, update, delete | read, create, update, delete | read, create, update, delete | |
| Point of Sale User | read | | read | |
| Point of Sale Manager | read, create, update, delete | | | |
| Inventory Manager | read, create, update, delete | | | |
| Manufacturing Manager | | read, create, update, delete | read | |
| Portal | read | read | | |
| Public | read | read | | |
| Accounting User | | | | read, create, update |

Consequences a rebuild must accept:

- Any internal user may **read** every price list, every rule and every vendor price the record
  rules let them see. Pricing is not confidential from internal users.
- A salesperson who is not a sales manager may read price **lists** but not price list **rules**
  through the ordinary path. Orders are nevertheless priced correctly, because the rule search is
  performed inside the platform's own computation and the resulting price is a computed field on
  the line. A rebuild must not let the rule-read restriction block the engine.
- The Product Margin Wizard grants no delete right to anybody. A short-lived record is discarded,
  never deleted by a user.
- Nobody may delete a vendor price except a holder of Products / Create or a purchase manager;
  sales managers cannot.

---

## 4. Record rules

All three are shipped as non-updatable data, so reinstalling the capability does not overwrite a
customised version.

| Rule name | Entity | Condition kept |
|---|---|---|
| "product pricelist company rule" | Price List | the record's company is an ancestor-or-self of one of the acting user's allowed companies, **or** the record has no company |
| "product pricelist item company rule" | Price List Rule | the same test on the rule's computed company |
| "product supplierinfo company rule" | Vendor Price | the record has no company, **or** its company is an ancestor-or-self of one of the allowed companies |

The vendor selection algorithm applies a **stricter** test of its own on top of the record rule:
an offer is a candidate only when it has no company or its company **is exactly** the buying
company. See [`calculations.md`](calculations.md#151-preparation).

---

## 5. Field-level access

| Entity | Field | Restriction |
|---|---|---|
| Price List | `code` (promotional code) | Readable and writable only by internal users |
| Price List Rule | `price_min_margin` (price minimum margin), `price_max_margin` (price maximum margin) | Shown on a form only with the Technical Features capability |
| Price List Rule | `pricelist_id` (price list), `company_id` (company), `currency_id` (currency), grouped as the company settings | Shown on a form only with the Technical Features capability, and each additionally gated by the multi-company and multi-currency capabilities |
| Product Template and Product Variant | `standard_price` (cost) | Readable only by internal users; read with elevated rights inside the price engine |
| Sales Order Line | `purchase_price` (cost), `margin`, `margin_percent` (margin percent) | Readable only by internal users |

---

## 6. Decimal precisions

Three shared, editable precision records govern this domain. Changing one changes rounding and
comparison everywhere it is used.

| Record name | Shipped digits | Used here for |
|---|---|---|
| `Product Price` | 2 | The display precision of the fixed price, the rounding step, the surcharge, the two margin bounds, the vendor unit price, the sales line unit price, the sales line cost and the sales line margin |
| `Discount` | 2 | The discount percentage of a vendor price, of a sales order line and of a purchase order line |
| `Product Unit` | 2 | The minimum quantity of a price list rule and of a vendor price, the rounding of every quantity conversion, and the tolerance of the quantity comparison in the vendor selection |

Two consequences that are easy to miss:

- `Product Price` may be **finer** than the rounding of the currency in use. The vendor ranking
  compares discounted prices at their stored precision after an unrounded currency conversion, so
  three offers at twenty-five, twenty-two and twenty thousandths order correctly even in a currency
  rounded to hundredths.
- The `Product Unit` precision changes **which** vendor offers survive: the minimum-quantity test on
  a vendor price is a precision-aware comparison, so at two digits a quantity of two and nine
  hundred ninety-nine thousandths counts as reaching a minimum of three. The same test on a price
  list rule is a plain comparison and is unaffected. See
  [`calculations.md`](calculations.md#192-the-product-unit-precision-and-quantity-breaks).

---

## 7. Configuration parameters

| Parameter key | Type | Shipped value | Effect |
|---|---|---|---|
| `res.partner.property_product_pricelist_` followed by the company identifier | text holding a price list identifier | unset | The third step of the contact price list fallback, tried when no price list without a country group matches |
| `res.partner.property_product_pricelist` | text holding a price list identifier | unset | The fourth step of the same fallback |

Both are read with elevated rights. A value that is empty, not a whole number, or that names a price
list which no longer exists is ignored and the next step of the fallback is taken. These parameters
exist **instead of** a per-field default value, because a per-field default would silently become
the value written on every contact created without an explicit price list, turning a fallback into
a pinned assignment.

---

## 8. Shipped records

The domain ships **no** price list and **no** price list rule. The only price list that exists in a
fresh database is the one the provisioning step creates for each company:

| Field | Value |
|---|---|
| `name` (name) | "Default" |
| `currency_id` (currency) | the company's currency |
| `company_id` (company) | the company |
| `sequence` (sequence) | 10 |
| `active` (active) | true |
| `item_ids` (rules) | none |

It ships the three decimal precision records of section 6 with the digits shown there, the three
record rules of section 4, the access entries of section 3, and two record-loading templates:

| Entity | Template label |
|---|---|
| Price List | "Import Template for Pricelists" |
| Vendor Price | "Import Template for Vendor Pricelists" |

It ships no message template, no activity type and no scheduled job.

---

## 9. Master-data prerequisites

Before a pricing configuration can be built, these records must exist. All are owned by other
folders.

| Prerequisite | Owner | Why it is needed |
|---|---|---|
| At least one company with a currency | [platform foundation](../platform-foundation/) | A price list needs a currency and, usually, a company |
| Currencies and their rates | [multi-currency](../multi-currency/) | Any price list expressed in a currency other than the product's needs a rate at the pricing date |
| Countries and country groups | [contacts and organizations](../contacts-and-organizations/) | Country-group defaults for contacts, and storefront availability by country |
| Units of measure with their absolute factors | [units of measure and packaging](../units-of-measure-and-packaging/) | Every quantity and price conversion |
| Product categories arranged as a tree with a materialised path | [products and catalog](../products-and-catalog/) | Category-scoped rules and their descendant matching |
| Products with a sales price, a cost and an own unit | [products and catalog](../products-and-catalog/) | The two catalogue bases of the formula |
| Sales taxes and purchase taxes on products, and fiscal positions | [taxes](../taxes/) | The tax-inclusion correction on a sales line and the stripping of price-included purchase taxes on a purchase line |
| Vendors as contacts | [contacts and organizations](../contacts-and-organizations/) | Vendor prices |
| Routes containing a buy rule | [replenishment and procurement](../replenishment-and-procurement/) | The vendor price chosen on a reordering rule |

---

## 10. Configuration recipes

Every configuration a rebuild must support out of the box, expressed as the rule fields to set.

| Goal | Rule configuration |
|---|---|
| A flat price for one product | level product template, kind fixed price, fixed price set |
| A flat price for one variant | level product variant, kind fixed price, fixed price set |
| A quantity break | two rules of the same level, one with minimum quantity zero, one with the break quantity; the higher minimum wins once it is reached |
| A customer discount the customer can see | level all products, kind percentage, positive percentage, Discounts capability granted |
| A customer discount the customer must not see | level all products, kind formula, base sales price, positive discount, everything else zero |
| A surcharge | kind percentage or formula with a negative percentage; the surcharge is never displayed as a discount |
| A cost-plus price | kind formula, base cost, positive markup |
| A cost-plus price with a floor expressed as an absolute margin | as above, plus a minimum margin; the margin is measured against the cost |
| Prices ending in ninety-nine hundredths | kind formula, rounding step 1.00, surcharge −0.01 |
| Prices ending in nine and ninety-nine hundredths | kind formula, rounding step 10.00, surcharge −0.01 |
| A doubled sales price with a floor of five currency units | kind formula, base sales price, discount −100, minimum margin 5.00 |
| A seasonal price | any kind, plus a validity window |
| A price list in another currency | a separate price list with the target currency; the base prices are converted automatically at the pricing date, and the fixed prices, surcharges and margins are read as amounts in the new currency without conversion |
| A reseller price list derived from a retail one | kind percentage or formula with base other price list pointing at the retail price list |
| A country default | a price list carrying the country groups; contacts in those countries resolve to it with no specific assignment |
| A storefront-only price list | set the website, or tick selectable, or set a promotional code |
| A price list a cashier can switch to | add it to the available price lists of the point of sale configuration, in the point of sale currency and in its company or in none |

---

## 11. Menu placement

| Menu entry | Parent | Opens | Visible with |
|---|---|---|---|
| Pricelists | Sales, then Products | the price list list, with the sales price base as the creation default | Basic Price Lists |
| Pricelists | Website, then Configuration | the same list | Basic Price Lists |
| Vendor Pricelists | Purchase, then Configuration | the vendor price list, filtered to active products and with the product column shown | Purchase Manager |
| Product Margins | Inventory or Accounting reporting, then Products | the Product Margin Wizard dialogue, whose button opens the product list in margin mode | Accounting User |
| Price Rules | no menu of its own; reached from a price list, from a product or from a settings link | the price list rule list | Basic Price Lists |

---

## 12. Scheduled jobs

The domain defines **none**. Prices are computed on demand and never materialised, so there is
nothing to refresh on a schedule. Two recurring computations owned elsewhere call into this domain:

| Job | Owner | What it calls here |
|---|---|---|
| The replenishment scheduler | [replenishment and procurement](../replenishment-and-procurement/) | The vendor price selection, for the lead time and for the price of the purchase order line it creates |
| The point of sale data load, on session opening and on each incremental refresh | [point of sale](../point-of-sale/) | The price lists and rules to be re-implemented client-side |
