# Pricing and Price Lists — Glossary

Every term this folder uses, defined. Terms owned by another domain are marked with the folder that
defines them. Reproduced identifiers are given in code font beside the full name, because the field
tables of [`entities.md`](entities.md) and the algorithms of [`calculations.md`](calculations.md)
use them.

British spelling is used throughout this folder.

---

## 1. Entities and records

**Price List** (`product.pricelist`, the transport name). A named collection of pricing rules bound
to exactly one currency, optionally bound to one company, to one or more country groups and to one
storefront. Applying a price list to a product, a quantity, a unit of measure, a date and a target
currency yields one unit price and the identity of the rule that produced it. A price list with no
rules yields the product's catalogue sales price converted into the price list's currency.

**Price List Rule** (`product.pricelist.item`). One pricing decision inside one price list: the
products it covers, the minimum quantity and validity window it requires, the amount it starts
from, and the method and parameters by which it turns that amount into a price. A rule has no state
field and no archive flag.

**Vendor Price** (`product.supplierinfo`, whose stored description is "Supplier Pricelist"). The
price one vendor charges for one product above a minimum quantity, in a unit of measure and a
currency, optionally only between two calendar dates, together with the vendor's lead time in days
and the vendor's own product code and product name.

**Product Margin Wizard** (`product.margin`). A short-lived form that collects a date range and an
invoice-state filter and opens the product margin analysis. It stores nothing beyond the life of
the dialogue.

**Price List Report** (`report.product.report_pricelist`). The comparison grid that lists selected
products in rows and requested quantities in columns, filled with the price one price list gives at
each quantity. It has no stored record.

**Default price list.** The price list named "Default" that the platform provisions for each
company, in the company's own currency, with sequence ten and no rules. Provisioning un-archives it
rather than creating a duplicate.

**Untouched default price list.** A price list with **no rules at all** whose currency equals its
company's currency. This is the exact test the provisioning step applies when deciding whether a
company already has a default price list.

**Country Group** (`res.country.group`). A named set of countries, owned by
[contacts and organizations](../contacts-and-organizations/). A price list that names country
groups becomes the default price list of contacts located in their member countries.

---

## 2. Scope, candidacy and selection

**Applicability level**, also called the **scope**. The breadth of a rule's application, one of:
all products, a product category and its descendants, a product template, a product variant. Stored
in `applied_on` (apply on) as a value whose leading digit makes ascending alphabetic order run from
most specific to least specific.

**Specificity.** The ordering of applicability levels, from a product variant (most specific) to
all products (least specific). The most specific applicable rule always wins, even when it is less
favourable to the customer.

**Candidate rule.** A rule of the price list whose price list, target fields and validity window do
not exclude the products being priced. Candidate retrieval ignores the minimum quantity and the
applicability level; both are evaluated afterwards, per product.

**Applicable rule.** A candidate rule that also passes the minimum-quantity test and the coverage
test for one specific product and quantity.

**Suitable rule**, also called the **applied rule**. The first applicable rule in the selection
order. It is returned to the caller alongside the price, and it is what a sales order line caches
in order to decide whether to display a discount.

**Empty rule.** The absence of an applicable rule, modelled as a rule record with no identity. It
always catches, its computation kind is nothing, its base is unset, and it prices at the catalogue
price. Treating it as a real object rather than as a special case is what lets every downstream
consumer ask it questions safely.

**Selection order.** Applicability level ascending, then minimum quantity descending, then product
category identifier descending, then rule identifier descending. A Price List Rule has **no**
sequence field; this ordering is the whole of it.

**Specificity ladder.** The mental model of the selection order: four rungs, one per applicability
level, descended until one rung catches, with the empty rule as the terminal rung.

**Quantity break.** Two or more rules of the same applicability level differing only by their
minimum quantity, which produce a lower unit price once the ordered quantity reaches the higher
threshold. Descending order on the minimum quantity is what makes the largest satisfied break win.

**Matching quantity**, also called the **quantity in the product unit**. The requested quantity
converted into the product's own unit of measure. This, and not the requested quantity, is what the
minimum quantity is compared against.

**Requested quantity.** The quantity the caller supplies, expressed in the document unit.

**Document unit**, called the *target unit* inside the algorithm. The unit of measure the caller
asks the price in. It defaults to the product's own unit.

**Product own unit.** The unit of measure stored on the product template. Every monetary parameter
of a price list rule is expressed per one of this unit.

**Pricing date.** The instant used both for rule validity and for the currency rate. It defaults to
the current instant and is resolved once for a whole call.

**Target currency.** The currency the returned price must be expressed in: the supplied currency,
else the price list's currency, else the acting company's currency.

---

## 3. Computation

**Computation kind**, stored in `compute_price` (compute price). One of `fixed` (Fixed Price),
`percentage` (Discount) and `formula` (Formula). The kind decides which parameters of the rule are
read and whether the result can be presented as a visible discount.

**Computation base**, stored in `base`. The amount a rule starts from: `list_price` (the product's
catalogue sales price including attribute extra prices), `standard_price` (the product's cost), or
`pricelist` (the price another price list gives for the same request).

**Base price.** The value of the computation base for one specific request, already converted into
the document unit and into the target currency, unrounded.

**Chained price list.** A price list used as the computation base of a rule of another price list.
Chaining may be arbitrarily deep as long as the graph of price lists stays acyclic.

**Recursion guard.** The write-time depth-first walk that refuses a rule which would close a cycle
in the graph of price lists. The engine itself performs no cycle check.

**Rounding step**, stored in `price_round` (price round). The multiple the discounted price is
snapped to, applied after the percentage and before the surcharge. A step of zero disables
rounding. The step is never converted between units of measure.

**Surcharge**, stored in `price_surcharge` (price surcharge) and labelled "Extra Fee". A fixed
amount added after the rounding step, expressed per one product unit and therefore converted into
the document unit. A negative surcharge subtracts, which is how prices ending in a chosen pattern
are produced.

**Minimum margin**, stored in `price_min_margin` (price minimum margin). The smallest allowed
distance between the final price and the base price. When the computed price falls below the base
price plus the minimum margin, it is raised to that floor and is **not** rounded again.

**Maximum margin**, stored in `price_max_margin` (price maximum margin). The largest allowed
distance between the final price and the base price. When the computed price rises above the base
price plus the maximum margin, it is lowered to that ceiling.

**Markup**, stored in `price_markup` (price markup). The presentation of the formula percentage used
when the base is the cost. It is maintained as the exact negation of the discount percentage, so a
markup of sixty per cent multiplies the cost by one and six tenths.

**Discount percentage of a rule**, stored in `price_discount` (price discount) for the formula kind
and in `percent_price` (percent price) for the percentage kind. A dimensionless number subtracted
from the base; a negative value raises the price.

**Attribute extra price.** An amount attached to an attribute value that is added to a product's
sales price when that value is part of the chosen combination. Inside the engine it is added
**before** the unit conversion, therefore it scales with the unit.

**Price before discount.** The base price reached by descending from the applied rule through
chained percentage rules until a rule that is not a percentage rule, or no rule at all, is met. It
is the amount shown as the unit price when a discount is displayed, and the denominator of the
displayed discount percentage.

**Price list price.** The price the applied rule actually produces, before any presentation
decision.

**Displayed price.** The larger of the price before discount and the price list price when a
discount is to be shown; the price list price otherwise. Taking the larger value is what prevents a
surcharge from appearing as a negative discount.

**Rule tip.** The live worked example of a formula rule on a base amount of one hundred, shown on
the rule form. It ignores the two margin bounds and is therefore a hint, not a specification.

**Price label.** The one-line human description of what a rule charges, computed from the kind, the
base and the parameters.

---

## 4. Presentation on a document

**Discounts capability.** The granted capability that decides whether a percentage rule presents
itself as a visible discount percentage on a document line or is folded into the unit price. There
is no per-price-list setting for this, and the test asks whether the capability is switched on in
the database rather than whether the acting user holds it.

**Shown discount.** A discount percentage written on a document line and printed on the document.
On a sales line it exists only when the Discounts capability is on and the applied rule's kind is
`percentage`.

**Folded discount.** A reduction that appears only as a lower unit price, with no discount column.
This is what a formula rule and a fixed rule always produce, and what a percentage rule produces
when the Discounts capability is off.

**Surcharge presentation.** A negative percentage raises the unit price and is never shown as a
negative discount on a sales document.

**Shadow unit price**, stored in `technical_price_unit` (technical price unit). A copy of the last
automatically computed unit price kept beside the stored unit price. When the two differ by more
than the line currency's rounding, the price is treated as manually set and is not recomputed by a
quantity or unit change.

**Manual price.** A unit price typed by a user, detected by the shadow price test. It survives every
automatic recomputation until an explicit repricing operation forces it out.

**Update Prices.** The explicit repricing operation on a sales order: it reprices the eligible
lines with recomputation forced, resets and recomputes their discounts, clears the indicator and
posts a message in the order's conversation.

**Tax-inclusion correction.** The re-expression of a unit price when the product's own taxes are
price-included and a fiscal position maps them onto other taxes. Owned by [taxes](../taxes/); this
domain only states where it is applied.

**Combo product and combo item line.** A product sold as a bundle of choices, and the lines that
carry the chosen items. The combo line displays a price of zero and each item line takes a prorated
share of the combo product's price.

---

## 5. Contact resolution

**Effective price list**, also called the **resolved price list**. The price list that applies to a
contact in the acting company, computed rather than stored: the specific assignment when there is
one, otherwise the country-group price list, otherwise the fallback.

**Specific assignment**, stored in `specific_property_product_pricelist` (specific property product
price list). The price list pinned on a contact, stored per company, written only when the user
picks a price list that differs from the country default.

**Country default.** The first active price list of the acting company or of no company that has a
country group containing the contact's country.

**Fallback price list.** The price list used when no country default applies: the first active
price list with no country group, otherwise the one named by the company-scoped configuration
parameter, otherwise the one named by the global configuration parameter, otherwise the first active
price list whatever its country groups.

**Context country.** A country supplied through the calling context under the key `country_code`
(country code), used to resolve the entry for contacts that have no country of their own.

**Commercial parent.** The company contact a contact belongs to, owned by
[contacts and organizations](../contacts-and-organizations/). The specific price list assignment is
one of the fields synchronised from it onto its child addresses, per company.

---

## 6. Storefront

**Publishable.** A price list is publishable on a storefront when its company is empty or equals the
storefront's company and either its website is that storefront, or it has no website and is either
selectable or carries a promotional code.

**Back-office price list.** A price list with no website, not selectable and with no promotional
code. It never reaches a storefront.

**Selectable**, stored in `selectable`. When true, a storefront visitor may pick this price list
from the price list chooser.

**Promotional code**, stored in `code` and labelled "E-commerce Promotional Code". A code a visitor
may type to activate a price list that is not otherwise offered. Readable only by internal users.

**Session price list.** The identifier of the resolved price list remembered for the visitor's
session under the key `website_sale_current_pl` (website sale current price list). It is the one
place in this domain where a resolution result is remembered rather than recomputed.

**Available in a country.** A price list is available in a country when it has no country groups at
all, or the country's code is among the codes of the countries of its country groups. A missing
country code makes every price list available.

---

## 7. Vendor side

**Vendor price ordering.** Sequence ascending, then minimum quantity descending, then unit price
ascending, then identifier ascending. This preparation ordering decides which **vendor** wins when
several vendors qualify.

**Discounted price**, stored as the computed `price_discounted` (price discounted). A vendor price's
unit price converted into the product's own unit of measure and then reduced by its discount
percentage. Converted into the company currency at the pricing date, it is the ranking key used to
choose between the surviving offers of the winning vendor.

**Forced unit.** A requirement, passed by the purchase order line, that a vendor price be stated
either in the line's unit of measure or in the product's own unit; vendor prices in any other unit
are excluded from selection.

**Grouping by vendor.** The step that keeps only the offers of the first vendor encountered in the
preparation ordering, and therefore never compares prices across vendors.

**Lead time in days**, stored in `delay`. The number of days between the confirmation of a purchase
order and the expected receipt, carried by a vendor price and used both to set the expected arrival
of a purchase order line and to plan replenishment.

**Vendor product code and vendor product name**, stored in `product_code` and `product_name`. The
vendor's own reference and name for the product, substituted for the internal ones on documents
sent to that vendor.

**Vendor registration**, also called **vendor price learning**. The automatic creation of a vendor
price on a product when a purchase order is confirmed with a vendor that is not yet a vendor of
that product, capped at ten existing offers.

**Retired offer.** A vendor price whose end date lies in the past. There is no archive flag on a
Vendor Price; an end date in the past is the retirement idiom.

---

## 8. Margins

**Line cost**, stored in `purchase_price` (purchase price) and labelled "Cost". The unit cost a
sales order line uses for its margin, expressed in the line's unit and the line's currency. It is
writable, so a salesperson may override it.

**Line margin**, stored in `margin`. The line's subtotal minus the line cost times the ordered
quantity; on a line that exists only because of a delivery, the unit price times the delivered
quantity minus the line cost times the delivered quantity.

**Line margin percentage**, stored in `margin_percent` (margin percent). The line margin divided by
the same subtotal, expressed as a **fraction** and displayed multiplied by one hundred.

**Order margin and order margin percentage.** The sum of the line margins, and that sum divided by
the order's untaxed amount. The percentage aggregates as an **average** in grouped lists.

**Stock-valuation cost variant.** The margin variant that blends the real unit value of what has
been delivered with the standard cost of what has not, weighted by quantity, for products whose
category cost method is not the standard one.

**Timesheet cost variant.** The margin variant that derives the line cost from the analytic lines of
the project, as the negated sum of their amounts divided by the sum of their unit amounts.

**Product margin analysis.** The fourteen measures computed on a product variant, over a date range
and an invoice-state filter, from posted and optionally draft invoice lines.

**Turnover.** The net sales value of a product over the range, taken from the accounting balance of
the customer invoice lines and therefore already in the company currency.

**Total cost (product margin analysis).** The net purchase value of a product over the range, taken
the same way from vendor bill lines.

**Expected sale.** The net invoiced sales quantity valued at the product's **current** catalogue
price.

**Normal cost.** The net invoiced purchase quantity valued at the product's **current** cost.

**Sales gap and purchase gap.** Expected sale minus turnover, and normal cost minus total cost.

**Total margin and expected margin.** Turnover minus total cost, and expected sale minus normal
cost.

**Total margin rate and expected margin rate.** Those two margins expressed **out of one hundred**,
not as fractions — the opposite convention from the sales line margin percentage.

---

## 9. Precisions, conversions and rounding

**Decimal precision record.** A named, editable setting that fixes the number of decimal places at
which one kind of value is displayed and compared. This domain uses `Product Price`, `Discount`
and `Product Unit`.

**Minimum display precision.** A declaration that a stored number is shown with at least the digits
of a named precision record. It does not round the stored value, and the price engine never rounds
to it.

**Quantity conversion.** Converting a quantity between units: multiply by the source unit's absolute
factor, divide by the destination unit's absolute factor, then round onto the `Product Unit` step
away from zero. Owned by
[units of measure and packaging](../units-of-measure-and-packaging/calculations.md).

**Price conversion.** Converting a price per unit between units: multiply by the **destination**
unit's absolute factor, divide by the **source** unit's absolute factor, never round. Owned by the
same folder, and pulling in the opposite direction from quantity conversion.

**Absolute factor.** How many reference units of its category one unit of measure contains. Owned by
[units of measure and packaging](../units-of-measure-and-packaging/).

**Currency conversion.** Multiplying an amount by the rate of the destination currency on a date and
dividing by the rate of the source currency on that date, rounding onto the destination currency
only when the caller asks for it. Inside the price engine it is always asked for **unrounded**.
Owned by [multi-currency](../multi-currency/).

**Rounding onto a step.** Dividing by the step, rounding half away from zero, and multiplying back.
The only rounding the engine itself performs.

---

## 10. Cross-domain terms used in this folder

**Company currency.** The currency of a company, owned by
[platform foundation](../platform-foundation/). The default currency of a price list and of a vendor
price, and the currency a document falls back to when it has no price list.

**Product currency.** The currency in which a product's catalogue sales price is expressed: its
company's currency when the product has a company, and the **main** company's currency otherwise.
Owned by [products and catalog](../products-and-catalog/).

**Cost currency.** The currency in which a product's cost is expressed: its company's currency when
the product has a company, and the **acting** company's currency otherwise. Owned by the same
folder. The asymmetry with the product currency is deliberate and observable.

**Fiscal position.** A mapping of taxes and accounts applied to a document because of the customer's
or the vendor's situation. Owned by [taxes](../taxes/). This domain reads only its tax mapping, and
only to re-express a price-included price.

**Price-included tax.** A tax whose amount is contained in the stated price rather than added to it.
Owned by [taxes](../taxes/). The two price adaptations of this domain act only on price-included
taxes.

**Reordering rule.** A rule that keeps a product's stock between a minimum and a maximum, owned by
[replenishment and procurement](../replenishment-and-procurement/). It may name a vendor price,
which then overrides the automatic vendor selection.

**Buy rule.** The procurement rule that satisfies a need by purchasing, owned by the same folder. It
calls the vendor price selection and contributes the selected lead time to the planning.

**Valued stock move.** A stock movement that carries a value, owned by
[inventory valuation and costing](../inventory-valuation-and-costing/). The stock margin variant
reads the unit value of the done moves of a sales line.

---

## 11. Reconciliation notes

1. **"Price list" as two words.** One of the two descriptions this glossary was merged from wrote the
   term as one compound word throughout, following the platform's own stored identifiers. Prose in
   this repository never abbreviates, and the compound spelling is an abbreviation of two words, so
   every entry here writes **price list**. The stored identifiers keep the compound spelling, in code
   font, because they are contractual.

2. **"Seller" against "vendor price".** One description used the word "seller" for a stored offer,
   following the storage name `seller_ids` (vendors). The record is one vendor's offer for one
   product, and the entity's own label calls it a supplier price list. This folder calls it a **Vendor
   Price** everywhere and reproduces the storage name only in code font.

3. **"Pricelist item" against "price list rule".** One description used the storage name as the term.
   The entity is called a **Price List Rule** here; `product.pricelist.item` appears only as a
   reproduced identifier.

4. **"Discount" as a single term.** The two descriptions used the one word for three different things:
   the percentage a rule subtracts, the percentage a document line displays, and the percentage a
   vendor offer subtracts. Section 3 and section 4 define the three separately, and every other file
   uses the qualified term.

5. **"Margin" as a single term.** Likewise the one word covered the two bounds of a formula rule and
   the difference between a price and a cost. Section 3 defines the bounds as the **minimum price
   margin** and the **maximum price margin**; section 8 defines the reporting measure.
