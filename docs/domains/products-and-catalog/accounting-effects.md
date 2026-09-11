# Products and Catalog — Accounting Effects

**This domain produces no journal entries.**

No entity specified in this folder — Product Template, Product Variant, Product Category, Product
Tag, Product Attribute, Attribute Value, Template Attribute Line, Template Attribute Value,
Template Attribute Exclusion, Attribute Custom Value, Product Combo, Product Combo Item, Product
Document, Packaging Barcode, Barcode Nomenclature, Barcode Rule, Label Layout, Attribute Value Bulk
Update, Expiry Delivery Confirmation — creates, changes, posts, reverses or reconciles any Journal
Entry or Journal Item. Creating a product, generating a variant, setting an extra price, defining an
exclusion, scanning a barcode, printing a label or recomputing an expiry date have no direct
financial consequence.

What the domain does is supply the **inputs** from which other domains compute financial
consequences. This file catalogues those inputs precisely, so that an implementer knows which
catalog values the accounting domains read and when a change to one of them will and will not
propagate.

---

## 1. Why the catalog posts nothing

The catalog describes what a thing *is*. A financial consequence arises only when something *happens*
to a thing: it is sold, bought, received, shipped, manufactured, scrapped, revalued or counted.
Each of those events belongs to an operational domain that owns its own journal entries.

Two consequences follow and must be respected by an implementation:

1. **Changing a catalog value never rewrites history.** Raising a product's cost does not
   re-value the stock already on hand; that is done by an explicit revaluation, specified in
   `../inventory-valuation-and-costing/`. Raising a sales price does not change an invoice already
   issued.
2. **Changing a catalog value changes only future computations.** The next document line created
   from that product picks up the new value.

---

## 2. Catalog values consumed by financial domains

### 2.1 The sales price

| Field | Read by | Used for |
|---|---|---|
| Template sales price (`list_price`) | `../pricing-and-pricelists/`, `../sales/`, `../point-of-sale/`, `../website-and-storefront/` | The base of every pricelist computation; the fallback when no pricelist rule matches |
| Variant price extra (`price_extra`, summed from the Template Attribute Values) | the same | Added to the base before any pricelist rule is applied |
| Variant sales price (`lst_price`) | `../sales/` margins, `../loyalty-and-promotions/` | The undiscounted reference price used to derive a displayed discount |

The sales price is expressed in the template's currency, which is the template's company's currency
or, for a shared template, the main company's currency. Conversion to a document's currency is the
document's responsibility.

### 2.2 The cost

| Field | Read by | Used for |
|---|---|---|
| Variant cost (`standard_price`), per company | `../inventory-valuation-and-costing/` | The unit value under the standard-cost method; the value of an inventory adjustment; the value of a receipt whose purchase cost is unknown; the running weighted average under the average-cost method, which **writes back** to this field |
| | `../sales/` and its margin companions | The cost side of a margin |
| | `../manufacturing/` | The component value in a production cost |
| | `../expenses/` | The unit price of an expense priced from a product |

The cost is company-dependent: each company stores its own value and reading returns the acting
company's. It is readable only by internal users; computations that need it for a non-internal
reader — deriving a sales price from a cost, for instance — re-read it as a privileged reader.

Two catalog behaviours interact with valuation and must be reproduced:

- writing the cost on a **template** with exactly one variant writes it on that variant;
- writing the cost on a template with several variants writes nothing, because the template's value
  is a computed mirror that falls back to zero in that case.

### 2.3 The category

The Product Category is the carrier of the valuation configuration: the costing method, the
valuation mode and the accounts. Those fields are **not** specified here — they belong to
`../inventory-valuation-and-costing/` — but the category record itself, its tree, its complete name
and its cycle protection are specified in [entities.md](entities.md).

Moving a product from one category to another therefore changes which accounts its future movements
hit. The catalog does not guard that change; the valuation domain does.

### 2.4 The product type

| Value | Financial meaning conferred by other domains |
|---|---|
| `consu` ("Goods") | May be stocked and valued; produces cost-of-goods-sold entries when delivered or invoiced, according to the recognition mode |
| `service` ("Service") | Never stocked, never valued; its revenue is recognised by the document alone |
| `combo` ("Combo") | Never stocked or valued in its own right; every financial consequence is carried by the child lines that name the chosen items. The combo line itself always carries a unit price of zero, so it contributes nothing to revenue. |

### 2.5 The unit of measure

The default unit and the additional packagings decide how a document's quantity is converted before
it is multiplied by a price or a cost. The conversion arithmetic belongs to
`../units-of-measure-and-packaging/`; the catalog only decides which unit is the default.

The unit-change behaviour matters financially: changing a product's unit **replaces the unit label
on existing documents without converting their quantities**. An implementation that converts
instead would silently restate historical amounts.

### 2.6 The tax fields

Taxes on a product — the customer taxes and the vendor taxes — are fields added to the Product
Template by `../taxes/`. This folder does not specify them. Their presence is noted here so that an
implementer does not look for them in [entities.md](entities.md).

### 2.7 Combo pricing

The proration of [calculations.md](calculations.md), section 8.2 decides how much of a combo
product's price each child line carries. Because those child lines are ordinary document lines, the
proration decides:

- how much revenue is attributed to each item's product, and therefore to each item's income
  account;
- how the tax is computed, since each child line carries the taxes of its own product;
- how the cost of goods sold is recognised, since each child line names a stockable product.

The proration is therefore financially material even though it produces no entry itself. Its two
invariants are worth restating here:

```formula
sum over choice groups of share(group) = combo_product_price     (exactly, in the document currency)
combo_product_line_unit_price = 0
```

The first holds because the rounding remainder is given to the last choice group.

---

## 3. Events in this domain and their financial consequence

| Event | Direct journal entry | Indirect consequence |
|---|---|---|
| A product is created | none | Future documents may reference it |
| A product's sales price changes | none | Future document lines price differently; existing lines are untouched |
| A product's cost changes | none | Future valuations use it. Under the standard-cost method an explicit revaluation, owned by `../inventory-valuation-and-costing/`, is what posts the difference |
| A product's category changes | none | Future movements hit the new category's accounts |
| A product's unit changes | none | Existing documents keep their numbers and change their unit label |
| Variants are generated | none | New records other domains may reference |
| A variant is archived | none | It can no longer be ordered; existing documents referencing it still resolve and still post |
| A variant is deleted | none | Deletion is refused, and archival substituted, precisely when a financial document references it |
| An extra price is set on a Template Attribute Value | none | Future lines for combinations containing that value price higher |
| An exclusion is created | none | Combinations disappear; their variants are archived rather than deleted when a financial document references them |
| A combo choice group's items change | none | The group's base price changes, so future prorations differ |
| A barcode is assigned or a scan is parsed | none | Identification only |
| A label is printed | none | none |
| An expiry date is computed or reached | none | Stock becomes unavailable, which changes **when** the valuation domain recognises an outgoing movement, not **what** it recognises |
| Expired lines are discarded before completing a transfer | none in this domain | The move lines disappear, so the movements they would have produced — and the entries the valuation domain would have posted for them — do not happen |
| A product message template fires on invoice posting | none | The invoice is already posted; the message is a notification only |

---

## 4. The one place a catalog record is written by an accounting flow

The relationship is not entirely one-way. Under the average-cost method, receiving goods **writes
back** onto the catalog: the variant's cost for the receiving company is updated to the new
weighted average. The formula and the entries are specified in
`../inventory-valuation-and-costing/`; what this domain guarantees is that the field exists, is
company-dependent, is stored on the variant and is mirrored onto the template when the template has
exactly one variant.

An implementation must not place any validation on the cost field that would block that write-back.
In particular, the "the cost of a product can't be negative" rule is an **interface warning only**
and must not become a record validation: a negative average cost can legitimately arise from a
correction pass over negative stock.

---

## 5. Analytic accounting

This domain contributes no analytic distribution and no analytic line. Products do not carry a
default analytic distribution; the distribution on a document line comes from the distribution
models of `../analytic-accounting/`, which may use the product or its category as a criterion. The
catalog supplies those criteria and nothing more.

---

## 6. Multi-company and multi-currency

- A template with no company is shared; its currency is the main company's currency. An
  implementation must resolve the currency that way and not from the acting company, or the sales
  price of a shared product would change meaning per company.
- A template with a company takes that company's currency.
- The **cost** currency resolves differently: the template's company's currency, or, when there is
  no company, the **acting** company's currency. This asymmetry is deliberate — a shared product's
  sales price is one number in one currency, while its cost is per company and therefore in the
  acting company's currency.
- The combo base price converts item prices into the choice group's currency, at the current moment,
  for the group's company or the acting company.
- The combo proration converts choice group base prices and item extra prices into the **document's**
  currency, for the **document's** company, at the **document's** date.
- The matrix header cell converts extra prices into the requested currency, for the requested
  company, at today's date.
- The vendor selection ordering converts discounted prices into the acting company's currency, at
  the requested date, **without rounding**.

All five conversions delegate to `../multi-currency/`.

---

## 7. Summary for an implementer

If you are building the accounting side of this system, take from this domain:

1. the sales price, the attribute extra prices and the combo proration, as the revenue inputs;
2. the cost, per company, as the valuation and margin input, and remember that valuation writes
   back to it;
3. the category, as the carrier of the valuation configuration;
4. the type, as the switch between stockable and non-stockable treatment;
5. the unit, as the quantity basis.

Take nothing else, and post nothing from here.
