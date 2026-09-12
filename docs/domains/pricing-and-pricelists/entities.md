# Pricing and Price Lists — Entities

This file specifies every entity the domain owns, and every field this domain adds to entities
owned elsewhere. For each entity: purpose, lifecycle, the complete field table, relations,
uniqueness, defaults, computed fields with their rules, ordering, display name, archival and
multi-company behaviour.

Field tables use three columns: the field with its storage name in code font, its type, and the
meaning and rules. Every storage name is given its full name in words on first use.

**Generated reference pages.** Every entity named in this file has a generated reference page that
lists its raw field inventory. This file is the authority on behaviour; the reference page is the
authority on the exact spelling of each identifier.

| Entity | Transport name | Reference page |
|---|---|---|
| Price List | `product.pricelist` | [`../../references/entities/product.pricelist.md`](../../references/entities/product.pricelist.md) |
| Price List Rule | `product.pricelist.item` | [`../../references/entities/product.pricelist.item.md`](../../references/entities/product.pricelist.item.md) |
| Vendor Price | `product.supplierinfo` | [`../../references/entities/product.supplierinfo.md`](../../references/entities/product.supplierinfo.md) |
| Product Margin Wizard | `product.margin` | [`../../references/entities/product.margin.md`](../../references/entities/product.margin.md) |
| Country Group | `res.country.group` | [`../../references/entities/res.country.group.md`](../../references/entities/res.country.group.md) |
| Product Template | `product.template` | [`../../references/entities/product.template.md`](../../references/entities/product.template.md) |
| Product Variant | `product.product` | [`../../references/entities/product.product.md`](../../references/entities/product.product.md) |
| Contact | `res.partner` | [`../../references/entities/res.partner.md`](../../references/entities/res.partner.md) |
| Company | `res.company` | [`../../references/entities/res.company.md`](../../references/entities/res.company.md) |
| Currency | `res.currency` | [`../../references/entities/res.currency.md`](../../references/entities/res.currency.md) |
| Sales Order | `sale.order` | [`../../references/entities/sale.order.md`](../../references/entities/sale.order.md) |
| Sales Order Line | `sale.order.line` | [`../../references/entities/sale.order.line.md`](../../references/entities/sale.order.line.md) |
| Purchase Order Line | `purchase.order.line` | [`../../references/entities/purchase.order.line.md`](../../references/entities/purchase.order.line.md) |
| Reordering Rule | `stock.warehouse.orderpoint` | [`../../references/entities/stock.warehouse.orderpoint.md`](../../references/entities/stock.warehouse.orderpoint.md) |
| Product Replenish Wizard | `product.replenish` | [`../../references/entities/product.replenish.md`](../../references/entities/product.replenish.md) |
| Point of Sale Configuration | `pos.config` | [`../../references/entities/pos.config.md`](../../references/entities/pos.config.md) |
| Website | `website` | [`../../references/entities/website.md`](../../references/entities/website.md) |
| Loyalty Programme | `loyalty.program` | [`../../references/entities/loyalty.program.md`](../../references/entities/loyalty.program.md) |

---

## 1. Price List

**Price List** (`product.pricelist`, table `product_pricelist`).

### 1.1 Purpose

A Price List is a named container of pricing rules, denominated in exactly one currency. It is the
unit of pricing policy: "Retail in euros", "Wholesale in euros", "Retail in United States dollars",
"Black Friday". A Price List holds no prices of its own; all pricing behaviour lives in its rules.
An empty Price List is legal and meaningful — it returns the product's catalogue price converted
into the Price List's currency.

A Price List is attached to a customer, to a country group, to a storefront, or to nothing at all.
It can also serve as the **base** of a rule in another Price List, which is how tiered pricing
policies are layered.

### 1.2 Lifecycle

| Stage | Trigger | Effect |
|---|---|---|
| Created automatically | A company is created, or the basic price list feature is switched on | One Price List named "Default", in the company's currency, with sequence ten and no rules, is created for every company that has none. An existing rule-less Price List whose currency equals its company's currency is un-archived instead of a new one being created. |
| Created manually | A user with the create privilege saves the form | The Price List exists with its currency, company, country groups and rules. |
| Copied | Duplicate | Name becomes the original name followed by a space and the word "(copy)" in parentheses, unless the caller supplies a name. Rules are copied. |
| Archived | The active flag is cleared, the Price List's currency is archived, or the basic price list capability is switched off | The Price List is hidden from selection lists everywhere but keeps its identity and its rules. A partner whose price list is archived falls back through the selection chain. |
| Archive refused | The active flag is cleared while an **active** loyalty or promotion programme names the Price List | Nothing is written. The message is "This pricelist may not be archived. It is being used for active promotion programs: " followed by the comma-separated programme names. Because archiving a currency archives its price lists, one such programme also aborts the archiving of the currency. |
| Un-archived | A user clears the archive, or the provisioning step of section 1.10 finds a rule-less Price List in the company's own currency | Reappears in every selection list and in the country resolution chain. |
| Deleted | Delete | Refused if any rule **in another Price List** uses it as a base. See the deletion guard below. |

### 1.3 Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Name (`name`) | single-line text | Required. Translatable. The human name of the price list. Not unique — two price lists may share a name. |
| Active (`active`) | boolean | Default true. When false the price list is hidden from every selection list and from the storefront, but existing documents keep their reference. Archiving is the supported way to retire a price list, because deletion is often refused. |
| Sequence (`sequence`) | integer | Default sixteen. Primary sort key. Lower sorts first. The automatically created company default is given sequence **ten**, so it sorts before manually created price lists, which default to sixteen. |
| Currency (`currency_id`) | link to Currency | Required. Default: the currency of the acting user's company. Tracked (tracking level one). Deleting the currency is restricted while a price list references it. **Every price computed by this price list is expressed in this currency** unless the caller explicitly asks for another. |
| Company (`company_id`) | link to Company | Default: the acting user's company. May be emptied, which makes the price list shared across all companies. Tracked (tracking level five). On delete: set to nothing. Changing it re-checks every rule for company consistency (see business rules). |
| Country Groups (`country_group_ids`) | many-to-many to Country Group | The set of country groups this price list serves. Empty means "no geographic restriction". Tracked (tracking level ten). Stored in the association table `res_country_group_pricelist_rel`, with the price list in column `pricelist_id` and the country group in column `res_country_group_id`. Used by partner price list selection and by the storefront. |
| Rules (`item_ids`) | one-to-many to Price List Rule | The rules of this price list, filtered by a domain so that rules pointing at an archived product template or an archived product variant are hidden. Copied when the price list is duplicated. |
| Promotional Code (`code`) | single-line text | Present only when the storefront capability is installed. Readable only by internal users. A storefront visitor who enters this code gets this price list even when it is not selectable. |
| Selectable (`selectable`) | boolean | Present only when the storefront capability is installed. When true, a storefront visitor may pick this price list from the price list chooser. |
| Website (`website_id`) | link to Website | Present only when the storefront capability is installed. On delete: restricted. Tracked (tracking level twenty). Default: the first website of the acting company. Restricted by a domain to websites of the price list's company, or to every website when the price list has no company. A price list with a website belongs to that website only. |
| Contacts Count (`partners_count`) | integer | Computed, never stored. Present only when the contact-partnership capability is installed. The number of contacts whose specific price list assignment names this price list, counted in the acting company. Displayed as a button that opens those contacts. |
| Contacts Label (`partners_label`) | single-line text | Related, read-only. Present only when the contact-partnership capability is installed: the acting company's own word for the group of contacts attached to a price list. It labels the count button; it never affects pricing. |
| Display Name (`display_name`) | single-line text | Computed, not stored. See the display rule below. |

The messaging mixin adds the usual conversation, follower, activity and rating fields; they behave
exactly as in every other threaded entity and are specified in the messaging domain. Price list
changes to the currency, the company and the country groups are logged in the conversation because
those three fields are tracked.

### 1.4 Display name rule

```formula
display_name = name_or_the_word_New + " (" + currency_code + ")"
```

The name falls back to the single word "New" when the name is empty (an unsaved record). The
currency code is the currency's short name, for example `EUR`. Two price lists named "Retail", one
in euros and one in United States dollars, therefore display as `Retail (EUR)` and `Retail (USD)`.

The display name depends on the currency, so changing the currency changes every place the price
list is shown.

### 1.5 Ordering

```
sequence ascending, then identifier ascending, then name ascending
```

The identifier tie-break before the name is deliberate: price lists created earlier sort before
price lists created later at the same sequence, which makes "the first price list" a stable notion
for the fallback chains described in [`calculations.md`](calculations.md).

### 1.6 The search-by-name behaviour

A name search matches either the name or the currency. Typing a currency code into a price list
selection field therefore finds every price list in that currency.

### 1.7 The rule visibility domain

The one-to-many relation to rules does **not** show every stored rule. It applies:

- keep the rule if it has no product template, or its product template is active; **and**
- keep the rule if it has no product variant, or its product variant is active.

So archiving a product hides its rules from the price list form without deleting them. The rules
remain in storage and — this is important — **remain effective**, because the price computation
searches rules directly and does not go through this relation. Archiving a product does not
disable its pricing rules; it only hides them from that one view. See
[`business-rules.md`](business-rules.md) section on archival.

### 1.8 Deletion guard

Before a price list is deleted, the platform searches for rules that satisfy all three of:

1. the rule's base is "other price list";
2. the rule's base price list is among the price lists being deleted;
3. the rule's own price list is **not** among the price lists being deleted.

If any such rule exists, deletion is refused with this message:

```
You cannot delete pricelist(s):
(the display names of the base price lists, one per line)
They are used within pricelist(s):
the display names of the price lists holding the rules, one per line
```

The third condition means a set of mutually dependent price lists can be deleted together: only
references **from outside the deleted set** block the deletion.

### 1.9 Multi-company behaviour

- A price list with no company is visible to every company.
- A price list with a company is visible to that company and to its descendant companies, through
  the record rule: keep the record when its company is an ancestor-or-self of one of the acting
  user's allowed companies, or when it has no company.
- A rule may only reference a base price list that is compatible with the rule's own company.
  Changing a price list's company re-runs that check on every rule and fails loudly if any rule
  would become inconsistent.

### 1.10 Automatic creation and the currency link

- Creating a company creates that company's "Default" price list, in the company's currency, with
  sequence ten — but **only** when the basic price list feature is enabled. When it is not, no
  price list is created.
- Changing a company's currency defers the automatic creation until after the currency is written,
  so that the created price list carries the new currency.
- Enabling multi-currency also enables the basic price list feature and then runs the automatic
  creation for every company.
- Archiving a currency archives every price list denominated in it.

### 1.11 Uniqueness, stored constraints and indexes

- **Identity** is the surrogate primary key alone. There is **no** natural key: two price lists may
  share the same name, the same currency, the same company and the same country groups. The display
  name disambiguates them only visually.
- There is **no** unique constraint and **no** stored check constraint on this entity. Every
  refusal it produces comes from a constraint method, a deletion guard or an archive guard; they are
  listed in [`business-rules.md`](business-rules.md#3-price-list).
- **Indexes:** the primary key; the index on `pricelist_id` (price list) carried by the rule side;
  and the association table `res_country_group_pricelist_rel` (price list to country group), which
  carries an index on each of its two columns.

---

## 2. Price List Rule

**Price List Rule** (`product.pricelist.item`, table `product_pricelist_item`).

### 2.1 Purpose

One rule of one price list. A rule answers three questions:

1. **To what does it apply?** — an applicability level (all products, a category, a template, a
   variant), plus a minimum quantity and a validity window.
2. **How does it compute?** — one of three computation kinds (fixed price, percentage discount,
   full formula).
3. **On what does it compute?** — a base (the catalogue sales price, the cost, or another price
   list).

### 2.2 Lifecycle

A rule has no state field. It comes into existence, it is edited, it is deleted with its price list
(the link cascades), or it is deleted on its own. Its *effectiveness* varies over time through the
validity window and over quantity through the minimum quantity; see
[`state-machines.md`](state-machines.md).

### 2.3 Field table

#### Ownership and scoping

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Price List (`pricelist_id`) | link to Price List | Indexed. On delete: cascade — deleting a price list deletes its rules. **Not required at the storage level**, because extension packages may create rules that belong to no price list; standard flows always set it. Default: the first price list whose company is the acting company or which has no company, in the standard ordering. |
| Price List Required (`is_pricelist_required`) | boolean | Computed, never stored, always true in the base platform. A pure user-interface flag that makes the price list field mandatory on the form. Extension packages override it to allow rules without a price list. |
| Company (`company_id`) | link to Company | Computed and stored. Read-only. Value: the price list's company if there is one, otherwise the product template's company, otherwise nothing. Recomputed when the price list's company or the rule's product template changes. On delete: set to nothing. Drives the company record rule and the company-consistency checks on the product, the variant and the base price list. |
| Currency (`currency_id`) | link to Currency | Computed and stored. Read-only. Value: the price list's currency if there is one, otherwise the computed company's currency, otherwise the acting company's currency. Recomputed when the price list's currency or the rule's company changes. Used to format the monetary fields on the form and in the rule description; it is **not** used by the price computation, which takes its currency from the caller or from the price list. |

#### Applicability

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Apply On (`applied_on`) | selection | Required. Default `3_global`. The applicability level. Values and labels: `0_product_variant` "Product Variant", `1_product` "Product", `2_product_category` "Product Category", `3_global` "All Products". **The stored values begin with a digit precisely so that ascending alphabetic ordering puts the most specific level first.** |
| Display Apply On (`display_applied_on`) | selection | Required. Default `1_product`. A user-interface-only field that chooses which half of the form is shown. Values: `1_product` "Product", `2_product_category` "Category". It never affects pricing; it only drives the change handlers that keep `applied_on` consistent with what the user filled in. |
| Category (`categ_id`) | link to Product Category | On delete: cascade. Meaningful only when the level is `2_product_category`. The rule then applies to this category **and every descendant category**, tested through the category's materialised path. Cleared automatically when the level is set to anything other than `2_product_category`. |
| Product (`product_tmpl_id`) | link to Product Template | On delete: cascade. Indexed (only non-empty values are indexed). Company-checked against the rule's company. Meaningful when the level is `1_product`; also carried, for user-interface convenience, when the level is `0_product_variant`. Cleared automatically when the level is `3_global` or `2_product_category`. |
| Variant (`product_id`) | link to Product Variant | On delete: cascade. Indexed (only non-empty values are indexed). Company-checked. Restricted by a domain to variants of the chosen product template. Meaningful only when the level is `0_product_variant`. Cleared automatically at every other level. |
| Product Unit Name (`product_uom_name`) | single-line text | Related, read-only: the unit name of the chosen product template. Shown next to the minimum quantity so the user can see the unit the minimum quantity is expressed in. |
| Variant Count (`product_variant_count`) | integer | Related, read-only: how many variants the chosen product template has. Used to decide whether the variant field is worth showing. |
| Min. Quantity (`min_quantity`) | decimal, `Product Unit` precision | Default zero. **Expressed in the product's own unit of measure, never in the document's unit.** A rule with a minimum quantity of zero has no quantity condition at all. A rule with a positive minimum quantity applies only when the requested quantity, converted into the product's own unit, is greater than or equal to it. |
| Start Date (`date_start`) | date and time | Optional. The first instant at which the rule is valid. Empty means "valid from the beginning of time". Compared against the pricing date as an instant, not as a day — a rule starting at nine in the morning does not apply at eight. |
| End Date (`date_end`) | date and time | Optional. The last instant at which the rule is valid. Empty means "valid forever". Compared as an instant. |

#### Computation

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Compute Price (`compute_price`) | selection | Required. Default `fixed`. Indexed. Values and labels: `fixed` "Fixed Price", `percentage` "Discount", `formula` "Formula". Determines which of the parameter fields below are read. |
| Fixed Price (`fixed_price`) | decimal, at least `Product Price` display precision | Read **only** when the computation kind is `fixed`. The price per the **product's own unit**, in the rule's price list currency. It is converted to the document's unit by price conversion, and it is **not** converted between currencies. |
| Percentage Price (`percent_price`) | decimal | Read **only** when the computation kind is `percentage`. A percentage subtracted from the base. A negative value is a mark-up. |
| Price Discount (`price_discount`) | decimal, two digits | Default zero. Read only when the computation kind is `formula`, and only when the base is **not** the cost. A percentage subtracted from the base price. A negative value is a mark-up. |
| Markup (`price_markup`) | decimal, two digits | Computed and stored, with an inverse. Always exactly the negation of the discount: writing one writes the other. Read instead of the discount when the base **is** the cost, so that a rule on cost is expressed as "cost plus twenty per cent" rather than "cost minus minus twenty per cent". |
| Price Rounding (`price_round`) | decimal, at least `Product Price` display precision | Read only when the computation kind is `formula`. The step the discounted price is rounded onto: a value of five hundredths rounds to the nearest multiple of five hundredths, a value of ten rounds to the nearest ten. Zero or empty means no rounding step. **Must be strictly positive** when set. Applied **after the discount and before the surcharge** — this ordering is what lets a rounding of ten with a surcharge of minus one hundredth produce prices ending in nine and ninety-nine hundredths. |
| Extra Fee (`price_surcharge`) | decimal, at least `Product Price` display precision | Read only when the computation kind is `formula`. A fixed amount added after rounding. Negative values subtract. Expressed **per the product's own unit** and converted to the document's unit by price conversion. |
| Min. Price Margin (`price_min_margin`) | decimal, at least `Product Price` display precision | Read only when the computation kind is `formula`. A floor expressed as an amount **above the base price**: the final price is never below the base price plus this amount. Expressed per the product's own unit and converted. |
| Max. Price Margin (`price_max_margin`) | decimal, at least `Product Price` display precision | Read only when the computation kind is `formula`. A ceiling expressed as an amount above the base price: the final price is never above the base price plus this amount. Expressed per the product's own unit and converted. Must not be below the minimum margin. |
| Based on (`base`) | selection | Required. Default `list_price`. Values and labels: `list_price` "Sales Price", `standard_price` "Cost", `pricelist` "Other Pricelist". Read by the percentage and formula kinds; ignored by the fixed kind. |
| Other Pricelist (`base_pricelist_id`) | link to Price List | On delete: set to nothing. Company-checked. Required when the base is `pricelist`. The price list whose result becomes this rule's base price. |

#### Descriptive, computed, never stored

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Name (`name`) | single-line text | Computed from the level and the target. "Category: " followed by the category display name when the level is category and a category is set; the product template display name when the level is template and a template is set; "Variant: " followed by the variant display name when the level is variant and a variant is set; otherwise "All Categories" when the display level is category; otherwise "All Products". |
| Price (`price`) | single-line text | Computed. A human sentence describing what the rule charges. See the three shapes below. |
| Rule Tip (`rule_tip`) | single-line text | Computed, only for formula rules with a base. A worked example on a base of one hundred. See below. |
| Display Name (`display_name`) | single-line text | The computed Name. |

**The three shapes of the Price description.**

1. Fixed kind: the fixed price, formatted with the `Product Price` precision and the rule's
   currency.
2. Percentage kind: "*p* % discount on *the base price list display name*" when a base price list
   is set, otherwise "*p* % discount on sales price". The percentage is printed without decimals
   when it is a whole number.
3. Formula kind: "*p* % *discount-or-markup* on *base* *extra*", where
   - *discount-or-markup* is the word "markup" and the markup value when the base is the cost, and
     the word "discount" and the discount value otherwise;
   - *base* is the base price list display name when the base is another price list, the words
     "product cost" when the base is the cost, and the words "sales price" otherwise;
   - *extra* is "+ *amount* extra fee" when the surcharge is positive, "− *amount* rebate" when it
     is negative (the amount printed as an absolute value in the rule's currency), and empty when
     the surcharge is zero.

**The Rule Tip.** Computed only when the computation kind is formula and a base is set. It shows:

```
<base label> with a <d> % <discount|markup> and <surcharge> extra fee
Example: <100 formatted> * <factor> + <surcharge> → <result formatted>
```

where

```formula
d      = price_discount            if the base is not the cost
d      = − price_markup            if the base is the cost
factor = ( 100 − d ) ÷ 100
result = round_to_step( 100 × factor , price_round )  +  price_surcharge      if a rounding step is set
result = 100 × factor + price_surcharge                                        otherwise
```

Note that the rule tip **ignores the two margin bounds**. It is a hint, not a specification; the
authoritative computation is in [`calculations.md`](calculations.md).

### 2.4 Ordering — and why it is the specificity order

```
applied_on ascending, min_quantity descending, categ_id descending, id descending
```

This single ordering clause **is** the rule selection order. It is read as follows.

| Sort key | Direction | Effect |
|---|---|---|
| `applied_on` | ascending | Because the selection values are `0_product_variant`, `1_product`, `2_product_category`, `3_global`, ascending alphabetic order puts variant rules first, then template rules, then category rules, then global rules. **Specificity wins over everything else.** |
| `min_quantity` | descending | Within one level, the highest minimum quantity is tried first. A rule "from one hundred units, twenty per cent off" is therefore tried before "from ten units, ten per cent off", which is tried before "from any quantity, no discount". This is what makes quantity breaks work: the engine takes the **first** applicable rule, so the largest break that the quantity satisfies wins. |
| `categ_id` | descending | Within one level and one minimum quantity, the category with the larger identifier wins. Since child categories are normally created after their parents, this **usually** prefers the more specific category — but it is an identifier comparison, not a depth comparison, and a rebuild must reproduce the identifier comparison, not invent a depth comparison. |
| `id` | descending | Final tie-break: the most recently created rule wins. |

### 2.5 Consistency enforced on write

Both creation and modification normalise the rule so that later searches are correct:

- On creation, if a variant is given but no template, the template is filled in from the variant.
- On creation, if no level is given, it is inferred: variant if a variant is given, otherwise
  template if a template is given, otherwise category if a category is given, otherwise global.
- On creation **and** on modification, whenever the level is set, the irrelevant targets are
  cleared:

| Level set to | Cleared |
|---|---|
| `3_global` | variant, template, category |
| `2_product_category` | variant, template |
| `1_product` | variant, category |
| `0_product_variant` | category |

Note that `1_product` clears the variant but keeps the template, and `0_product_variant` keeps
both the variant and the template. A variant rule therefore always carries its template, which is
what lets the applicable-rule search filter by template.

### 2.6 Form change handlers

These run only in an interactive form; they never run on a programmatic write.

| Change | Effect |
|---|---|
| Base changed | The discount and the markup are both reset to zero. |
| Base price list changed | If the computation kind is percentage, the base becomes `pricelist` when a base price list is now set and `list_price` when it is cleared. |
| Computation kind changed | The base price list is cleared. If the kind is not fixed, the fixed price is zeroed. If the kind is not percentage, the percentage price is zeroed. If the kind is not formula, the base is reset to `list_price` and the discount, surcharge, markup, rounding step, minimum margin and maximum margin are all zeroed. |
| Display level changed | If neither a template nor a category is set, the level becomes global. If the display level is product, the level becomes variant when the form was opened from a variant and template otherwise, and the category is cleared. If the display level is category, the variant and template are cleared, the level becomes category, and the unit name is cleared. |
| Variant changed | The template is filled in from the variant. If the form was opened with a default level of template, rules that now have a variant become variant rules and rules that lost their variant become template rules. |
| Template changed | A variant that does not belong to the new template is cleared. |
| Variant, template or category changed, with no default level | Rules with both a variant and a template become variant rules; rules with only a template become template rules; rules with a category whose name is not the single word "All" become category rules; everything else becomes global. |
| Rounding step changed | A strictly negative rounding step is refused immediately with the message *The rounding method must be strictly positive.* |
| Start or end date changed | The date-range check is run immediately. |
| Level, variant, template or minimum quantity changed, with the event-ticketing capability installed | A **non-blocking** warning appears when the minimum quantity is strictly positive. Its title is "Warning". Its text is "A pricelist item with a positive min. quantity will not be applied to the event tickets products." when the level is `3_global` or `2_product_category`, and "A pricelist item with a positive min. quantity cannot be applied to this event tickets product." when the level is `1_product` and the template's service tracking is event registration, or the level is `0_product_variant` and the variant's service tracking is event registration. Both texts are reproduced exactly, including the shortened form of the words "minimum quantity", because a rebuild must show the same sentence. No other combination produces a warning, and the value is never changed. |

### 2.7 Multi-company behaviour

The record rule keeps a rule when its computed company is an ancestor-or-self of one of the acting
user's allowed companies, or when it has no company. Because the company is computed from the
price list and the product template, a rule inherits the visibility of whichever of those two
carries a company.

The automatic company check refuses a rule whose product template, product variant or base price
list belongs to a different company than the rule itself.

### 2.8 Uniqueness, stored constraints and indexes

- **Identity** is the surrogate primary key. Several rules with the same level, the same target, the
  same minimum quantity, the same window and the same computation may coexist inside one price list;
  the ordering of section 2.4 decides which one is selected and the others never apply.
- There is **no** unique constraint and **no** stored check constraint. The five constraint methods
  — the base price list, the recursion guard, the date range, the margin ordering and the target
  consistency — are listed with their messages in
  [`business-rules.md`](business-rules.md#4-price-list-rule).
- **Indexes:** `pricelist_id` (price list), a plain index; `product_tmpl_id` (product template),
  indexed over non-empty values only; `product_id` (product variant), indexed over non-empty values
  only; `compute_price` (computation kind), a plain index. The rule ordering itself is not indexed.

### 2.9 Lifecycle in one place

A rule is created from the price list form, from the product template form, from the product variant
form, from the rule list or by a data load; it is edited freely at any time; and it disappears when
it is deleted directly or when its price list, its product template, its product variant or its
product category is deleted, because all four links cascade. Deleting a rule is never refused.
Changing a rule takes effect on the **next** price computation: document lines already written keep
the price they carry until they are repriced (see [`workflows.md`](workflows.md#12-change-the-price-list-on-a-quotation-and-update-prices)).

---

## 3. Vendor Price

**Vendor Price** (`product.supplierinfo`, table `product_supplierinfo`). The user-visible label is
"Supplier Pricelist"; a single record is one vendor's offer for one product.

### 3.1 Purpose

A Vendor Price records what one vendor charges for one product: the price, the currency, the unit
the vendor quotes in, the minimum quantity that unlocks the price, the validity window, a
percentage discount, a lead time in days, and the vendor's own name and code for the product.

Several Vendor Prices may exist for the same product and the same vendor — that is how quantity
breaks and successive validity windows are expressed.

### 3.2 Lifecycle

| Stage | Trigger |
|---|---|
| Created manually | On the product form, on the vendor form, or through the import template. |
| Created automatically | A purchase order is confirmed for a product whose vendor is not yet recorded and which has ten or fewer recorded vendors. See [`workflows.md`](workflows.md). |
| Edited | Freely. |
| Deleted | Freely; also cascaded when the product template or the vendor is deleted. |

There is no state field and no archival flag. A Vendor Price is retired by giving it an end date in
the past or by deleting it.

### 3.3 Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Vendor (`partner_id`) | link to Contact | Required. On delete: cascade. Company-checked. The vendor. A purchase line matches this against the order's vendor **or that vendor's parent contact**, so an offer recorded on a parent company also serves its child addresses. |
| Vendor Product Name (`product_name`) | single-line text | Optional. Printed on a request for quotation instead of the internal product name when set. |
| Vendor Product Code (`product_code`) | single-line text | Optional. Printed on a request for quotation instead of the internal reference when set. |
| Sequence (`sequence`) | integer | Default one. Primary sort key of the ordering and a secondary key of the selection algorithm. Lower means higher priority. |
| Unit (`product_uom_id`) | link to Unit of Measure | Required. Computed with a default, but freely writable and stored. On delete: restricted. Default: the variant's unit when a variant is set, otherwise the product template's unit. Once set it is never recomputed. **The price and the minimum quantity are both expressed in this unit.** |
| Quantity (`min_qty`) | decimal, `Product Unit` precision | Required. Default zero. The quantity that must be bought to obtain this price, expressed **in the vendor's unit**. Zero means no minimum. |
| Unit Price (`price`) | decimal, at least `Product Price` display precision | Default zero. The price per one of the vendor's unit, in the vendor's currency, before the discount. |
| Discounted Price (`price_discounted`) | decimal | Computed, never stored. See the formula below. Used as the primary sort key of the selection algorithm. |
| Discount (%) (`discount`) | decimal, `Discount` precision | A percentage taken off the unit price. Copied onto the purchase order line as the line discount. |
| Company (`company_id`) | link to Company | Indexed. Default: the acting company. May be emptied, which makes the offer valid for every company. On delete: set to nothing. |
| Currency (`currency_id`) | link to Currency | Required. Default: the acting company's currency. On delete: restricted. The currency the unit price is quoted in. |
| Start Date (`date_start`) | date | Optional. The offer is ignored on dates strictly before this day. A **date**, not an instant — unlike the price list rule window. |
| End Date (`date_end`) | date | Optional. The offer is ignored on dates strictly after this day. |
| Product Variant (`product_id`) | link to Product Variant | Optional. Computed with a default, writable, stored. On delete: set to nothing. Restricted by a domain to variants of the chosen template. **Empty means the offer applies to every variant of the template**; set means it applies to that variant only. |
| Product Template (`product_tmpl_id`) | link to Product Template | Required. Indexed. On delete: cascade. Computed from the variant when a variant is given, writable, stored. |
| Variant Count (`product_variant_count`) | integer | Related, read-only: the number of variants of the template. |
| Lead Time (`delay`) | integer | Required. Default one. Days between confirming a purchase order and receiving the goods. Used to derive the planned date of a purchase line and by the replenishment scheduler. |

Five further fields are contributed by extension packages. They are listed in full because the
selection algorithm, the replenishment screens and the subcontracting flows all read them.

| Field (storage name) | Type | Contributed by | Meaning and rules |
|---|---|---|---|
| Subcontracted (`is_subcontractor`) | boolean | the subcontracting capability | Computed, never stored. True when the vendor is registered as a subcontractor of the product. It is what allows the product to be subcontracted to that vendor, and it is the field the subcontracting narrowing of the candidate set tests. |
| Last Purchase (`last_purchase_date`) | date | the purchasing-and-inventory bridge | Computed, never stored. The order date of the most recent **confirmed** purchase order placed with this vendor that contains any variant of this product template. Informational; it takes no part in the selection. |
| Agreement (`purchase_requisition_id`) | link to Purchase Agreement | the purchase-agreement capability | Related, read-only: the agreement of the agreement line below. |
| *(technical, no label)* (`purchase_requisition_line_id`) | link to Purchase Agreement Line | the purchase-agreement capability | Indexed over non-empty values. Set when the offer comes from a purchase agreement rather than being maintained by hand. The agreement narrowing of the candidate set keeps only the offers with no agreement line or with the agreement of the order in context. |
| Show Set Supplier Button (`show_set_supplier_button`) | boolean | the purchasing-and-inventory bridge | Computed, never stored. True except when this offer is already the one named on the reordering rule carried in the calling context. It drives the visibility of the *Set Vendor* action on the replenishment information screen. |

### 3.4 The discounted price formula

```formula
price_discounted = convert_price( price , vendor_unit → product_own_unit ) × ( 1 − discount ÷ 100 )
```

where the product's own unit is the variant's unit when the record names a variant, and the
template's unit otherwise, and `convert_price` is the price conversion of
[`../units-of-measure-and-packaging/calculations.md`](../units-of-measure-and-packaging/calculations.md#10-price-conversion).

Two properties matter for a rebuild:

- The conversion is into the **product's own unit**, not into the purchase line's unit. This makes
  the value comparable across vendors who quote in different units, which is exactly what the
  selection algorithm needs.
- No currency conversion happens here. The selection algorithm converts separately, at sort time,
  using the company currency and the pricing date.

**Worked example.** A vendor quotes ninety-six per dozen with a five per cent discount, on a
product whose own unit is units.

```formula
convert_price( 96 , Dozens → Units ) = 96 × 1 ÷ 12 = 8
price_discounted = 8 × ( 1 − 5 ÷ 100 ) = 8 × 0.95 = 7.6
```

### 3.5 Ordering

```
sequence ascending, min_qty descending, price ascending, id ascending
```

Note this is the **stored** ordering, used whenever a set of Vendor Prices is read. The selection
algorithm of [`calculations.md`](calculations.md) applies a *different*, richer ordering on top of
it; the stored ordering is what the preparation step uses to produce a deterministic candidate
list.

### 3.6 Display name

In the base platform the display name is the **vendor's** display name, and nothing else. Two offers
from the same vendor for the same product therefore display identically; they are distinguished in
lists by the quantity, the price and the dates, which the list view always shows.

With the purchasing-and-inventory bridge installed the display name is enriched:

```formula
display_name = vendor_display_name + " (" + minimum_quantity + " " + unit_name + " - " + unit_price_formatted_in_the_offer_currency + ")"
```

unless the calling context asks for the simplified vendor name, in which case the vendor's display
name alone is used. The minimum quantity is printed with the `Product Unit` precision and the unit
price with the offer's currency.

**Worked example.** The vendor "Wood Corner", a minimum quantity of three, the unit named "Units"
and a unit price of seven hundred eighty-five United States dollars display as
`Wood Corner (3.0 Units - $ 785.00)`.

### 3.7 Normalisation and form change handlers

Whenever a variant is written without a template, the template is filled in from the variant. This
runs on creation and on modification.

Two handlers run only inside an interactive form:

| Change | Effect |
|---|---|
| Product template changed | A variant that no longer belongs to the chosen template is cleared. |
| Vendor changed | Present only with the purchasing capability. The currency becomes the vendor's preferred purchase currency when the vendor has one, and the acting company's currency otherwise. |

### 3.8 Multi-company behaviour

The record rule keeps the offer when it has no company, or when its company is an ancestor-or-self
of one of the acting user's allowed companies. The filtering helper used by the selection algorithm
repeats the check explicitly: an offer is a candidate only when it has no company or its company
**is exactly** the acting company — note that this is an equality test, stricter than the record
rule's ancestor test.

The vendor, the product variant and the product template are company-checked against the offer's own
company.

### 3.9 Uniqueness, stored constraints and indexes

- **Identity** is the surrogate primary key. The same vendor may hold several offers for the same
  product, differing by minimum quantity, by validity window, by unit, by currency or by company;
  that is precisely how quantity breaks and seasonal prices are expressed.
- There is **no** unique constraint, **no** stored check constraint and **no** constraint method at
  all on this entity. Every rule that governs it is a default, a normalisation or a filter applied
  by a consumer; they are listed in [`business-rules.md`](business-rules.md#5-vendor-price).
- **Indexes:** `product_tmpl_id` (product template), a plain index; `company_id` (company), a plain
  index; `purchase_requisition_line_id` (purchase agreement line), indexed over non-empty values
  only.

---

## 4. Extensions to entities owned by other domains

### 4.1 Product Template

**Product Template** (`product.template`, table `product_template`) — owned by
[products and catalog](../products-and-catalog/). This domain relies on and extends:

| Field (storage name) | Type | Role in this domain |
|---|---|---|
| Sales Price (`list_price`) | decimal, at least `Product Price` display precision | Default one. Tracked. The catalogue price, per the template's own unit, in the template's currency. The `list_price` base of a rule reads this. |
| Cost (`standard_price`) | decimal | Computed from the single variant, with an inverse back to it, searchable. Visible only to internal users. The `standard_price` base of a rule reads this. Per the template's own unit, in the template's **cost** currency. |
| Currency (`currency_id`) | link to Currency | Computed, not stored: the template's company's currency, or the main company's currency when the template has no company. The currency the catalogue price is expressed in. |
| Cost Currency (`cost_currency_id`) | link to Currency | Computed, not stored, depends on the acting company: the template's company's currency, or the acting company's currency when the template has no company. The currency the cost is expressed in. |
| Unit (`uom_id`) | link to Unit of Measure | Required. Tracked. **The unit every price list rule parameter is expressed in.** |
| Packagings (`uom_ids`) | many-to-many to Unit of Measure | Additional units the product may be sold in. Together with the own unit, these are the units a document may choose. |
| Vendors (`seller_ids`) | one-to-many to Vendor Price | The offers recorded for this template. Depends on the acting company. |
| Price List Rules (`pricelist_rule_ids`) | one-to-many to Price List Rule | The rules targeting this template, filtered to rules with no price list or with an active price list. Lets a user manage pricing from the product form. |

This domain adds three business operations to the template, specified in
[`calculations.md`](calculations.md): the raw price reader, the contextual price and the
contextual price list.

### 4.2 Product Variant

**Product Variant** (`product.product`, table `product_product`).

| Field (storage name) | Type | Role in this domain |
|---|---|---|
| Variant Price Extra (`price_extra`) | decimal | Computed, not stored: the sum of the extra prices of the variant's attribute values. Added to the catalogue price by the `list_price` base. |
| Sales Price (`lst_price`) | decimal | Computed, not stored, depends on a requested unit supplied through the calling context. Value: the template's catalogue price converted into the requested unit (or left as is when none is requested), **plus** the variant extra price — note the extra is added *after* the conversion and is therefore **not** converted. Writing it writes back the template's catalogue price, after converting the written value into the template's own unit and subtracting the extra. |
| Cost (`standard_price`) | decimal | Per company. Visible only to internal users. Never negative (guarded). |
| Price List Rules (`pricelist_rule_ids`) | one-to-many to Price List Rule | Computed from the template's rules, keeping the rules that target no variant or this variant; writing it writes the template's set, preserving rules that target other variants. |

### 4.3 Contact

**Contact** (`res.partner`, table `res_partner`) — owned by
[contacts and organizations](../contacts-and-organizations/).

| Field (storage name) | Type | Role in this domain |
|---|---|---|
| Pricelist (`property_product_pricelist`) | link to Price List | Computed with an inverse, never stored. Restricted by a domain to price lists of the acting company or with no company. Depends on the contact's country and on the specific assignment, and on two context values: the acting company and a country code override. **This is the effective price list of the contact** and is what a sales order reads. |
| *(no label)* (`specific_property_product_pricelist`) | link to Price List | Per company, stored. The deliberate assignment made by a user. Empty means "use the fallback chain". Part of the commercial field set, so it is propagated from a parent company contact to its child contacts. |

The inverse writes the specific assignment only when the chosen price list differs from what the
fallback chain would have produced for the contact's country; choosing exactly the fallback clears
the specific assignment. This keeps a contact following the policy unless a user deliberately
overrode it.

### 4.4 Country Group

**Country Group** (`res.country.group`, table `res_country_group`) gains the reverse relation
`pricelist_ids`, the many-to-many back to price lists through `res_country_group_pricelist_rel`.

### 4.5 Sales Order Line

**Sales Order Line** (`sale.order.line`, table `sale_order_line`) — owned by [sales](../sales/).

| Field (storage name) | Type | Role in this domain |
|---|---|---|
| *(technical, no label)* (`pricelist_item_id`) | link to Price List Rule | Computed, never stored. The rule the price list selected for this line's product, quantity and unit. Recomputed whenever the product, the unit or the quantity changes. Empty when there is no product, when the line is a display-only line, or when the order has no price list. |
| Unit Price (`price_unit`) | decimal, at least `Product Price` display precision | Computed, stored, writable, required, pre-computed. |
| *(technical, no label)* (`technical_price_unit`) | decimal | Stored, not computed. A shadow copy of the last automatically computed unit price. When it differs from the unit price, the user has edited the price by hand and automatic recomputation stops. |
| Discount (%) (`discount`) | decimal, `Discount` precision | Computed, stored, writable, pre-computed. |
| Cost (`purchase_price`) | decimal | Computed, stored, writable, not copied, pre-computed. Visible only to internal users. The unit cost used for the margin. |
| Margin (`margin`) | decimal | Computed and stored. Visible only to internal users. |
| Margin (%) (`margin_percent`) | decimal | Computed and stored. Visible only to internal users. |

### 4.6 Sales Order

**Sales Order** (`sale.order`, table `sale_order`) — owned by [sales](../sales/).

| Field (storage name) | Type | Role in this domain |
|---|---|---|
| Pricelist (`pricelist_id`) | link to Price List | Computed, stored, writable, pre-computed, company-checked, tracked (tracking level one). Derived from the customer's effective price list read in the order's company, and only while the order is a draft; cleared when there is no customer. Restricted by a domain to price lists of the order's company or with no company. Its help text reads "If you change the pricelist, only newly added lines will be affected." |
| Currency (`currency_id`) | link to Currency | Computed, stored, pre-computed: the price list's currency when a price list is set, and the company's currency otherwise. This is why choosing a price list in another currency re-expresses the whole quotation. |
| *(no label)* (`has_active_pricelist`) | boolean | Computed, never stored: true when at least one **active** price list exists whose company is the order's company or nothing. It decides whether the price list field is worth showing at all. |
| Has Pricelist Changed (`show_update_pricelist`) | boolean | Neither stored nor computed. Set to true when the user changes the price list on an order that already has lines, and when the user changes the order's company. It makes the *Update Prices* operation visible, and that operation sets it back to false. |
| Margin (`margin`) | monetary | Computed and stored. Visible only to internal users. The sum of the line margins. |
| Margin (%) (`margin_percent`) | decimal | Computed and stored. Visible only to internal users. Its aggregation in a grouped list is the **average**, not the sum. |

### 4.7 Purchase Order Line

**Purchase Order Line** (`purchase.order.line`, table `purchase_order_line`) — owned by
[purchasing](../purchasing/).

| Field (storage name) | Type | Role in this domain |
|---|---|---|
| *(technical, no label)* (`selected_seller_id`) | link to Vendor Price | Computed, never stored. The offer the selection algorithm chose for this line. |
| Unit Price (`price_unit`) | decimal | Computed, stored, writable. |
| *(technical, no label)* (`technical_price_unit`) | decimal | The shadow copy that detects a manual override, exactly as on the sales line. |
| Discount (%) (`discount`) | decimal | Computed, stored, writable. Copied from the chosen offer's discount, or set to zero when no offer applies. |
| Unit Price in the Product Unit (`price_unit_product_uom`) | decimal | Computed, read-only display: the line's unit price converted into the product's own unit, and zero for display-only lines and down payments. |
| Unit Price (Discounted) (`price_unit_discounted`) | decimal | Computed: the unit price times one minus the discount over one hundred. |
| Allowed Units (`allowed_uom_ids`) | many-to-many to Unit of Measure | Computed: the product's own unit, plus its packaging units, plus the units of every offer for this product or for no particular variant. This is why a buyer can pick the vendor's unit even when the product does not otherwise list it. |

### 4.8 Website

**Website** (`website`, table `website`) gains a computed, never-stored **Price list available for
this Ecommerce/Website** (`pricelist_ids`), the set of price lists the website publishes, and a
computed **Default Currency** (`currency_id`) which is the currency of the price list resolved for
the current request, falling back to the website's company currency. No field is added on the
website side to bind a price list to it: that binding is carried on the price list, by `website_id`
(website), `selectable` (selectable) and `code` (promotional code), and by the publishability tests
of [`business-rules.md`](business-rules.md#13-storefront).

### 4.9 Reordering Rule

**Reordering Rule** (`stock.warehouse.orderpoint`, table `stock_warehouse_orderpoint`) — owned by
[replenishment and procurement](../replenishment-and-procurement/). The fields below exist only when
the purchasing-and-inventory bridge is installed.

| Field (storage name) | Type | Role in this domain |
|---|---|---|
| Vendor Pricelist (`supplier_id`) | link to Vendor Price | Stored, writable, company-checked, with an inverse. Restricted by a domain to the offers of this variant, or of this template with no variant. The offer the buy rule must use instead of running the automatic selection. Writing it while the reordering rule has **no** route assigns the first route that contains a buy rule. |
| Show supplier column (`show_supplier`) | boolean | Computed, never stored: true when the reordering rule's effective route contains a buy rule. It decides whether the vendor column is shown. |
| *(no label)* (`supplier_id_placeholder`) | single-line text | Computed, never stored: the display name of the offer the automatic selection *would* pick, shown as the placeholder of the vendor price list field so that the buyer can see the default without pinning it. |
| Vendors (`vendor_ids`) | one-to-many to Vendor Price | Related, read-only: the offers of the variant. |
| *(no label)* (`effective_vendor_id`) | link to Contact | Computed, never stored, searchable: the vendor of the offer named on the rule when one is named, otherwise the vendor of the offer the automatic selection would pick. |
| Available Vendor (`available_vendor`) | link to Contact | Neither stored nor computed; a search helper only. It matches any vendor that has an offer for the rule's product. |

Clearing the route of a reordering rule clears its vendor price list. Setting the vendor price list
also raises the quantity to order to the offer's minimum quantity converted into the product's own
unit whenever the quantity to order is below it; see
[`business-rules.md`](business-rules.md#12-replenishment).

### 4.10 Product Replenish Wizard

**Product Replenish Wizard** (`product.replenish`) — owned by
[replenishment and procurement](../replenishment-and-procurement/). It gains, from the same bridge,
a transient **Vendor** (`supplier_id`) link to Vendor Price. Opening the wizard from a reordering
rule copies that rule's offer into it. When the wizard is opened with the vendor column shown and
no offer chosen, the **first** offer of the product template is proposed; clearing the vendor column
clears the field. The chosen offer is handed to the buy rule, which then skips the automatic
selection entirely, and it also drives the wizard's planned date through the offer's lead time.

### 4.11 Point of Sale Configuration

**Point of Sale Configuration** (`pos.config`, table `pos_config`) — owned by
[point of sale](../point-of-sale/).

| Field (storage name) | Type | Role in this domain |
|---|---|---|
| Use a pricelist. (`use_pricelist`) | boolean | Whether the terminal offers a choice of price lists. The label ends with a full stop; it is reproduced as the platform shows it. |
| Default Pricelist (`pricelist_id`) | link to Price List | The price list used when no customer is selected, or when the selected customer has no price list. |
| Available Pricelists (`available_pricelist_ids`) | many-to-many to Price List | The price lists a cashier may switch to. |

The four validations these three fields carry — the default must be among the available ones, every
available one must be in the terminal's currency, and both must belong to no company or to the
terminal's company — are in [`business-rules.md`](business-rules.md#14-point-of-sale). They
constrain configuration, never the computation.

### 4.12 Company

**Company** (`res.company`, table `res_company`) — owned by
[platform foundation](../platform-foundation/). This domain adds **no field**. It adds two
behaviours:

1. Creating a company provisions that company's default price list, unless the calling context
   carries the flag that suppresses the step or the acting user does not hold the basic price list
   capability.
2. Writing a company's currency performs the write with provisioning suppressed and re-runs the
   provisioning afterwards, so that a price list created in the same transaction would carry the new
   currency. See the compatibility finding recorded in
   [`business-rules.md`](business-rules.md#3-price-list) on why the re-run does not in fact happen.

The values used for a company's default price list are fixed:

| Field | Value |
|---|---|
| `name` (name) | "Default" |
| `currency_id` (currency) | the company's currency |
| `company_id` (company) | the company |
| `sequence` (sequence) | ten |

### 4.13 Currency

**Currency** (`res.currency`, table `res_currency`) — owned by
[multi-currency](../multi-currency/). This domain adds **no field**. It adds two behaviours:
archiving a currency archives every price list denominated in it, and granting the multi-currency
capability also grants the basic price list capability to internal users and then runs the
provisioning for every company.

### 4.14 Loyalty Programme

**Loyalty Programme** (`loyalty.program`, table `loyalty_program`) — owned by
[loyalty and promotions](../loyalty-and-promotions/). This domain adds no field to it. It adds one
guard in the opposite direction: a price list named by an **active** programme cannot be archived,
with the message quoted in section 1.2. The programme side carries the mirror constraint that a
programme's currency must equal the currency of every price list it names.

---

## 5. Product Margin Wizard

**Product Margin Wizard** (`product.margin`) is a transient form with three fields and one button.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| From (`from_date`) | date | Default: the first day of January of the current year. |
| To (`to_date`) | date | Default: the thirty-first day of December of the current year. |
| Invoice State (`invoice_state`) | selection | Required. Default `open_paid`. Values and labels: `paid` "Paid", `open_paid` "Open and Paid", `draft_open_paid` "Draft, Open and Paid". |

The button opens the product list in margin mode, carrying the three values in the calling context
under the keys `date_from`, `date_to` and `invoice_state`, and disabling creation and editing. The
seventeen margin analysis fields on the product variant are computed from that context; see
[`calculations.md`](calculations.md).

---

## 6. The margin analysis fields on the Product Variant

Seventeen fields are computed together by one operation, never stored, and read only when the
calling context supplies the date range and the invoice-state filter: **three echoes** of that
context and **fourteen numeric measures**. They are grouped here; the arithmetic is in
[`calculations.md`](calculations.md).

| Field (storage name) | Type | Meaning |
|---|---|---|
| Margin Date From (`date_from`) | date | Echo of the context date range start; defaults to the first of January of the current year. |
| Margin Date To (`date_to`) | date | Echo of the context date range end; defaults to the thirty-first of December of the current year. |
| Invoice State (`invoice_state`) | selection | Echo of the context filter; defaults to "Open and Paid". |
| Avg. Sale Unit Price (`sale_avg_price`) | decimal | Average unit price across customer invoices and credit notes. |
| Avg. Purchase Unit Price (`purchase_avg_price`) | decimal | Average unit price across vendor bills and refunds. |
| # Invoiced in Sale (`sale_num_invoiced`) | decimal | Net invoiced quantity on the sales side. |
| # Invoiced in Purchase (`purchase_num_invoiced`) | decimal | Net invoiced quantity on the purchase side. |
| Turnover (`turnover`) | decimal | Net sales value. |
| Total Cost (`total_cost`) | decimal | Net purchase value. |
| Expected Sale (`sale_expected`) | decimal | Net invoiced sales quantity valued at the current catalogue price. |
| Normal Cost (`normal_cost`) | decimal | Net invoiced purchase quantity valued at the current cost. |
| Sales Gap (`sales_gap`) | decimal | Expected sale minus turnover. |
| Purchase Gap (`purchase_gap`) | decimal | Normal cost minus total cost. |
| Total Margin (`total_margin`) | decimal | Turnover minus total cost. |
| Expected Margin (`expected_margin`) | decimal | Expected sale minus normal cost. |
| Total Margin Rate (%) (`total_margin_rate`) | decimal | Total margin times one hundred divided by turnover. |
| Expected Margin (%) (`expected_margin_rate`) | decimal | Expected margin times one hundred divided by expected sale. |

Thirteen of the fourteen numeric measures are summable in grouped lists, through a special
aggregation path that computes every group's total by summing the per-record values rather than by
asking the database. The exception is the average purchase unit price (`purchase_avg_price`), which
carries no such aggregation and cannot be summed. The three context echoes are not measures and are
not aggregated.

The button that opens this analysis is on the Product Margin Wizard of section 5; the list, form and
graph it opens are described in [`interfaces.md`](interfaces.md#7-screens).

---

## 7. Reconciliation notes

Two independently written descriptions of this domain were merged into this file. Where they
disagreed the platform's own behaviour was consulted and the correct statement kept; each resolution
is recorded here.

1. **The identifiers used.** One description named fields by a readable canonical name — for
   instance a "vendor price" field called `unit_of_measure` — and the other reproduced the storage
   names. Storage names are contractual: a rebuild that must import an existing database or serve an
   existing integration depends on them character for character. This file therefore reproduces the
   storage names, in code font, each with its full name in words, and the readable names appear only
   as prose. The affected fields are the vendor price's unit (`product_uom_id`), minimum quantity
   (`min_qty`), discount (`discount`), lead time (`delay`) and vendor (`partner_id`); the rule's
   minimum quantity (`min_quantity`), category (`categ_id`), template (`product_tmpl_id`), variant
   (`product_id`) and margins (`price_min_margin`, `price_max_margin`); the price list's rules
   (`item_ids`) and country groups (`country_group_ids`); and the sales and purchase line fields
   (`price_unit`, `technical_price_unit`, `discount`, `pricelist_item_id`, `selected_seller_id`,
   `date_planned`, `allowed_uom_ids`).

2. **The price description field of a rule.** One description called the derived one-line price
   sentence `price_label`, to distinguish it from the numeric notion of a price. The stored name is
   `price` (price description) and it is reproduced here; the field is text, never a number, and the
   table says so.

3. **The display name of a Vendor Price.** One description said the display name is the vendor's
   display name; the other said it is the vendor's display name followed by the quantity, the unit
   and the price, and attributed the difference to the purchasing capability. Both halves are true
   of different installations, and the attribution was wrong: the enriched form is contributed by
   the purchasing-and-inventory bridge, not by purchasing alone. Section 3.6 states both forms and
   the correct contributor.

4. **The event-ticket warning on a rule.** One description recorded a non-blocking form warning when
   a positive minimum quantity is set on a rule while the event-ticketing capability is installed;
   the other did not mention it, and an intermediate draft of this folder withdrew it as
   non-existent. The warning does exist, with two distinct texts. It is restored in section 2.6 and
   is numbered in [`business-rules.md`](business-rules.md#4-price-list-rule).

5. **How many margin measures the product analysis has.** Both descriptions spoke of "fifteen"
   measures. The operation computes **seventeen** fields: three echoes of the calling context and
   fourteen numeric measures, of which **thirteen** can be summed in a grouped list. Section 6
   states the corrected counts.

6. **The unit recorded when a vendor price is learned from a confirmed order.** One description said
   the unit is copied from the offer the line had selected; the other said it is the order line's
   unit. Neither was complete: when the line had selected an offer, the vendor's product name and
   product code are copied from that offer while the unit is taken from the **line**; when the line
   had selected no offer, none of the three is written and the unit falls back to its ordinary
   default. [`calculations.md`](calculations.md#1510-learning-a-vendor-price-from-a-confirmed-order)
   states it.

7. **Whether a rule can be archived.** One description described the rule as "not archivable, in the
   sense that it disappears when its price list is archived". A rule has no active flag at all and
   nothing about it changes when its price list is archived; only the price list stops being
   selected. Section 2.9 states it plainly.
