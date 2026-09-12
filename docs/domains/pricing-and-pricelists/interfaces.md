# Pricing and Price Lists — Interfaces

The named operations a client or an integration invokes, the request routes, the printable and
exported documents, the notifications, the record-loading templates and the screens of the domain,
described as views over the behaviour specified in [`calculations.md`](calculations.md) and
[`business-rules.md`](business-rules.md).

---

## 1. Named operations

### 1.1 The price engine

Every operation in this table is **read-only**: it creates nothing, modifies nothing, posts no
message and emits no notification. All of them accept **at most one** price list; an empty price
list is legal and yields catalogue prices.

| Operation | Receiver | Inputs | Output | Failure |
|---|---|---|---|---|
| Price of one product | at most one price list | one product (a template or a variant), a quantity, optionally a unit, a date, a currency | one unrounded unit price, per one requested unit, in the requested currency, excluding taxes | refuses when more than one price list, product, unit or currency is supplied |
| Price and rule of one product | at most one price list | the same | the pair: the price and the identifier of the suitable rule, empty when none applied | the same |
| Rule of one product | at most one price list | the same | the identifier of the suitable rule alone; the price computation is skipped and a placeholder of zero is produced internally | the same |
| Prices of several products | at most one price list | a set of products, a quantity, optionally a unit, a date, a currency | a map from product identifier to unit price | the same |
| Price per price list | a set of price lists; an **empty** set means *every* price list | one product, a quantity, optionally a unit, a date | a map from product identifier to a map from price list identifier to the pair of price and rule | none |
| Price of one rule | at most one rule | a product, a quantity, a unit, a date, optionally a currency | the price this specific rule yields, with no rule selection at all | refuses when more than one rule, product, unit or currency is supplied |
| Base price of one rule | at most one rule | a product, a quantity, a unit, a date, a currency | the base price the rule starts from, in the given currency | the same |
| Price before discount of one rule | one rule | the same | the base price reached by descending through chained percentage rules | the same |
| Is this rule applicable | one rule | a product, a quantity **already expressed in the product's own unit** | true or false | refuses when more than one rule or product is supplied |
| Applicable rules of a price list | at most one price list | a set of products, a date | the candidate rules, already in selection order | none |

### 1.2 Contact price list resolution

| Operation | Inputs | Output | Side effects |
|---|---|---|---|
| Resolve the price lists of contacts | a set of contact identifiers | a map from contact identifier to price list, empty for every contact when the Basic Price Lists capability is off | none |
| Resolve the price lists of countries | a list of country identifiers | a map from country identifier to price list, plus one entry for "no country" | none |

Both are evaluated in the acting company, and, inside a storefront request, additionally restricted
to the price lists publishable on that storefront.

### 1.3 Vendor price selection

| Operation | Receiver | Inputs | Output | Side effects |
|---|---|---|---|---|
| Prepare the offers | one product variant | optionally: a purchase order that fixes the buying company, a forced-unit flag, a set of subcontractors, a purchase agreement | the company-filtered, active-vendor-filtered, variant-filtered offers in the stored ordering | none |
| Filter the offers | one product variant | optionally a vendor, a quantity, a date, a unit, the preparation options | the surviving offers, keeping the prepared order | none — but the quantity conversion **raises** when the units belong to different trees |
| Select one offer | one product variant | the same, plus an optional primary ranking key | at most one offer | the same |

Passing **no quantity at all** disables the minimum-quantity filter; passing a quantity of zero does
not.

### 1.4 Record operations

| Operation | Receiver | Effect | Refusal |
|---|---|---|---|
| Archive | price lists | clears the active flag | refused when an active loyalty or promotion programme names the price list |
| Un-archive | price lists | sets the active flag | none |
| Delete | price lists | deletes the price list and, by cascade, its rules | refused when a rule of **another** price list uses it as a base |
| Duplicate | price lists | copies the record, its rules, its country groups and its website; names the copy after the original followed by " (copy)" unless a name is supplied | none |
| Provision the company price lists | companies, or every company when none is named | un-archives untouched default price lists and creates the missing ones | silently does nothing when the calling context asks it to be skipped, or when the acting user does not hold Basic Price Lists |
| Open the comparison report preview | one price list | opens the price grid preview screen | none |
| Set the vendor on a reordering rule | one vendor price, with a reordering rule in the calling context | assigns a route containing a buy rule when the reordering rule has none, writes the offer on the reordering rule, raises the quantity to order to the offer's minimum quantity converted into the product's own unit, then reopens either the replenishment wizard or the replenishment information screen | returns nothing when no reordering rule is in context |
| Update prices | one sales order | reprices the eligible lines with recomputation forced, resets and recomputes their discounts, clears the indicator and posts a message in the conversation | the price list of a confirmed order cannot be changed at all |
| Open the product margins | one Product Margin Wizard | opens the product variant list, form and graph in margin mode, carrying the date range and the invoice-state filter in the calling context and disabling creation and editing | none |
| Get the record-loading templates | Price List, Vendor Price | returns the label and the location of the shipped template | none |

---

## 2. Request routes

| Path | Kind | Authentication | Read-only | Purpose |
|---|---|---|---|---|
| `/product/export/pricelist/` | form submission | a signed-in user | yes | Produces the comparison grid of a price list as a downloadable file |

**Request.** Two values.

| Value | Type | Meaning |
|---|---|---|
| `report_data` (report data) | a nested key-value document encoded as text | the same payload the preview screen uses: the price list identifier, the list of quantities, the entity of the selected records, the selected record identifiers, and the flag that decides whether the price list name is printed as a title |
| `export_format` (export format) | text | the chosen file format: `csv` for the delimited text file, anything else for the spreadsheet workbook |

**Response.** The file content, with the media type of the chosen format and a disposition header
naming the file "Pricelist - " followed by the price list's name and the extension of that format.

**File content, both formats.** A header row reading `Product`, then `UOM`, then one column per
requested quantity labelled `Quantity (<quantity> UoM)` — all four strings reproduced exactly,
including the shortened forms, because integrations read them. Then one row per selected product,
except that a template with more than one variant contributes **one row per variant instead of** a
row for itself. Each row holds the product name, the name of the product's own unit, and the price
at each quantity. The spreadsheet format additionally widens every column to the longest value it
contains, header included.

The preview screen calls a read-only operation with the same payload and receives the rendered grid
as a document fragment for display. It performs exactly the same computation as the printable
report.

---

## 3. Printable documents

### 3.1 The price list comparison grid

| Property | Value |
|---|---|
| Report name | "Pricelist" |
| Reached from | the print action on a price list form; the print menu of the product list and of the product variant list, with records selected |
| Inputs | a price list, a list of quantities, the selected products, and a flag deciding whether the price list name is printed |
| Fallback | when the requested price list no longer exists, the **first** price list in the standard price list ordering is used |
| Rows | one per selected product; a template with more than one variant gains nested, indented rows, one per variant |
| Columns | the product name; the name of the product's own unit, shown only with the units of measure capability; one column per requested quantity |
| Cell content | the price the price list gives for that product at that quantity, per the product's own unit, at the current instant, formatted in the price list's currency |
| Totals | none |
| Heading | the word "Pricelist", followed by a colon and the price list's display name when the title flag is set |

Default quantities: a single quantity of one. Default entity: product templates. With no selected
identifiers the grid is empty, which is a normal outcome and not an error.

The preview version renders each product name as a link that opens the product, and the price list
name as a link that opens the price list.

### 3.2 Documents of other domains that carry this domain's output

| Document | Owner | What this domain contributes |
|---|---|---|
| Quotation and sales order print-out | [sales](../sales/) | the unit price column and, with the Discounts capability, the discount percentage column |
| Request for quotation and purchase order print-out | [purchasing](../purchasing/) | the unit price, the discount percentage, the expected arrival date, and the vendor's own product code and product name inside the line description |
| Customer invoice print-out | [accounts receivable](../accounts-receivable/) | the unit price and discount inherited from the sales order line |
| Product label sheets | [products and catalog](../products-and-catalog/) | the price printed on a label, which is the contextual price of the product under the price list carried in the calling context |

---

## 4. Exported files

| File | Produced by | Content |
|---|---|---|
| Comparison grid, delimited text | the export route | as described in section 2 |
| Comparison grid, spreadsheet workbook | the export route | as described in section 2, with sized columns |
| Price list loading template | the data-loading screen of Price List | a spreadsheet skeleton carrying the price list columns |
| Vendor price loading template | the data-loading screen of Vendor Price | a spreadsheet skeleton carrying the vendor price columns |

The ordinary record export is available on all three entities. On Vendor Price, exporting **with**
the external identifier and loading the file back updates the existing rows; exporting **without**
it and loading the file back creates new rows, which is the usual cause of a vendor appearing twice
for the same product after a price update.

---

## 5. Notifications and messages

| Event | Channel | Text |
|---|---|---|
| Prices recomputed on a sales order that has a price list | the order's conversation | "Product prices have been recomputed according to pricelist" followed by the price list's display name rendered as a link, and a full stop |
| Prices recomputed on a sales order that has no price list | the order's conversation | "Product prices have been recomputed." |
| The currency, the company, the country groups or the website of a price list changed | the price list's conversation | a tracked-field entry naming the old and the new value |
| Replenishment found no vendor for a product | the conversation of the record that raised the procurement, addressed to the responsible users | "No supplier has been found to replenish" followed by the product's display name, then ", this product should be manually replenished." |
| The user is about to turn the Pricelists setting off while at least one active price list exists | a non-blocking warning in the settings screen | "You are deactivating the pricelist feature. Every active pricelist will be archived." |
| The user triggers *Update Prices* | a confirmation dialogue | "This will update the unit price of all products based on the new pricelist." |
| The user changes the company of a quotation that already has lines and is still a draft | a non-blocking warning | title "Warning for the change of your quotation's company", text "Changing the company of an existing quotation might need some manual adjustments in the details of the lines. You might consider updating the prices." |

Every refusal message of the domain is listed with its condition in
[`business-rules.md`](business-rules.md).

---

## 6. Scheduled jobs

The domain defines none. See [`configuration.md`](configuration.md#12-scheduled-jobs).

---

## 7. Screens

Screens are described as the fields they show and the operations they offer. Nothing here changes
behaviour; the behaviour is in the other files.

### 7.1 Price list list

- Columns: a drag handle bound to the sequence, the name, the country groups as removable tags with
  the placeholder "All countries", the currency (only with the multi-currency capability), the
  company (only with the multi-company capability).
- Ordering: the standard price list ordering. Dragging a row writes the sequence.
- Search: a text field matching the name, a field matching the currency, and an "Archived" filter.
- Operations: create, open, archive, un-archive, delete, export, load a data file, print the
  comparison grid.
- Empty-state guidance: "Create a new pricelist", followed by an explanation that a price list is a
  set of sales prices or of rules computing the price of sales order lines from products, product
  categories, dates and ordered quantities, and that price lists can be assigned to customers or
  chosen on a quotation.

### 7.2 Price list card view

One card per price list, showing the name in bold and the currency beside a money symbol. Used on
narrow screens.

### 7.3 Price list form

- Header: the print button, which opens the comparison grid preview.
- Ribbon: "Archived", shown when the price list is archived.
- Title: the name, with guidance text showing a sample price list name.
- Left group: the currency (with the multi-currency capability), the company (with the multi-company
  capability, with the placeholder "Visible to all" and no inline creation).
- Right group: the country groups as removable tags.
- Tab "Sales Prices": the list of rules, each row showing the computed scope label under the
  heading "Apply on", the computed price label under the heading "Price", the minimum quantity, the
  start date and the end date. Opening a row opens the rule form. New rules default to the sales
  price base.
- Storefront fields, present only with the storefront capability: the website, the selectable flag,
  the promotional code.
- The conversation at the bottom, recording the four tracked fields.

### 7.4 Price list rule form

- Left group, "Apply To" as two radio buttons: "Product" or "Category".
  - With "Product": the product template field, placeholder "All products"; and, when the chosen
    template has at least two variants, the variant field, placeholder "All variants".
  - With "Category": the category field, placeholder "All categories".
- "Price Type" as three radio buttons: "Discount", "Formula", "Fixed Price". Shown only with the
  Basic Price Lists capability.
  - With "Fixed Price": the amount, followed by the word "per" and the name of the product's own
    unit when a product is chosen.
  - With "Discount": the percentage, a percent sign, then the word "on" and a price list selector
    whose placeholder is "sales price". That selector excludes the rule's own price list.
- Right group: the minimum quantity, and the validity window as a date range written by one control.
- Information panel, shown only with the Discounts capability: with a percentage rule, "In sale
  order line original price is unit price and discount is in discount column."; with any other
  kind, "For formula or fixed pricing, the original price isn't shown in sale orders."
- Group "Based price", shown only with the formula kind:
  - the base selector, and the base price list selector, visible, editable and required only when
    the base is another price list;
  - the discount percentage, replaced by the markup percentage when the base is the cost;
  - "Round off to", the rounding step;
  - "Extra Fee", the surcharge;
  - "Margins", the minimum and the maximum with an arrow between them, shown only with the
    Technical Features capability;
  - two information panels: the live worked example of the formula on a base of one hundred, and
    the fixed hint "Tip: want to round at 9.99?" followed by "round off to 10.00 and set an extra
    at -0.01".
- Group "Company Settings", shown only with the Technical Features capability: the price list, the
  currency (with the multi-currency capability), the company (with the multi-company capability).
- Guards: the stored constraints run on save; the rounding step and the validity window are checked
  while typing.

Two derived forms exist. Opened from a product template, the "Apply To" radio buttons are hidden,
the product template is read-only and forced, and the price list selector moves beside the validity
window. Opened from a product variant, the variant is additionally read-only and forced.

### 7.5 Price list rule lists

Two variants.

The general list shows the price list (required), the computed scope label under the heading
"Applied On", the computed price label, the minimum quantity, and optionally the start date, the
end date and the company.

The list embedded in a product form is editable in place and shows the price list (only with the
Basic Price Lists capability, no inline creation, no navigation), the variant (only with the
variants capability, read-only when reached from a variant, hidden for a single-variant product,
required when the level is variant, placeholder "All variants"), the product template (only when
reached from a category), the fixed price under the heading "Price" and required, the minimum
quantity, the start and end dates, and optionally the company. Its create control is labelled "Add
a price".

### 7.6 Price list rule search

- Filters: "Product Rule" (the level is product template), "Variant Rule" (the level is variant,
  only with the variants capability), "Active" (the rule's price list is active).
- Searchable fields: the price list, the company, the currency.
- Groupings: by product template, by variant (only with the variants capability), by price list.

### 7.7 Vendor price list

Columns: a drag handle bound to the sequence; the vendor, read-only in place; the variant,
read-only, hidden when reached from a template that hides variants, restricted to the variants of
the template in context; the product template, read-only, hidden when the template is already the
context; the vendor's product name; the vendor's product code; the start date; the end date; the
company, read-only, only with the multi-company capability; the minimum quantity; the unit, only
with the units of measure capability; the unit price; the discount percentage; the currency, only
with the multi-currency capability; the lead time in days. Multi-record editing is enabled.

### 7.8 Vendor price card view

One card per offer, showing the vendor and the price in bold on the first line, and the minimum
quantity and the lead time on the second.

### 7.9 Vendor price form

- Group "Vendor": the vendor, searched among contacts marked as vendors; the vendor's product name;
  the vendor's product code; the lead time, followed by the word "days".
- Group "Pricelist": the product template, hidden when it is the context; the variant, only with the
  variants capability and with no inline creation; the minimum quantity followed by the unit, only
  with the units of measure capability; the unit price followed by the currency, only with the
  multi-currency capability; the validity as the start date, the word "to", and the end date; the
  discount percentage; the company.

### 7.10 Vendor price search

- Searchable fields: the vendor, the product template, the vendor's product name, the vendor's
  product code.
- Filters: "Active Products" (the product template or the variant is active); "Active" (no end
  date, or an end date at or after yesterday); "Archived" (an end date before yesterday).
- Groupings: by product template, by vendor.
- Empty-state guidance: "No vendor pricelist found", followed by "Register the prices requested by
  your vendors for each product, based on the quantity and the period."

### 7.11 Product form, pricing tab

Shown only with the Basic Price Lists capability and only for a product that can be sold. It holds
the embedded, editable rule list of section 7.5, scoped to the product. Opened on a product variant,
the same tab creates variant-scoped rules by default.

### 7.12 Product form, purchase tab

Shown only to users who may read vendor prices. It holds the vendor price list scoped to the
product, editable in place. For a product with a single variant the template-level list is shown;
for a product with several variants the variant-aware list is shown instead.

### 7.13 Contact form, sales and purchase tab

A price list field in the sales section, showing the resolved price list, with candidates restricted
to the price lists of the acting company and the shared ones. Writing it records a specific
assignment only when the chosen value differs from the country default.

### 7.14 Sales order form

- A price list field with no inline creation and no navigation, shown only with the Basic Price
  Lists capability and only when at least one active price list exists for the order's company or
  with no company; read-only once the order is confirmed or cancelled.
- Beside it, an *Update Prices* button with a refresh symbol, whose help text is "Recompute all
  prices based on this pricelist", visible only when the price list changed on an order that has
  lines and the order is neither confirmed nor cancelled, and guarded by the confirmation dialogue
  of section 5.
- On each line: the unit price, and the discount percentage column shown only with the Discounts
  capability.

### 7.15 Purchase order form

On each line: the unit price; the discount percentage; the expected arrival; the unit, restricted to
the product's own unit, its packaging units and the units used by the product's offers; and the line
description, which carries the vendor's own product code and product name.

### 7.16 Reordering rule list and replenishment information

- The reordering rule list shows a vendor column, visible only when the rule's effective route
  contains a buy rule, holding the chosen offer with the automatically selected one as its
  placeholder.
- The replenishment information screen lists the product's offers with a "Set Vendor" action on
  each, hidden on the offer already chosen.

### 7.17 Product margin screens

- The Product Margin Wizard dialogue: the range start, the range end, the invoice-state selector,
  and two buttons, "Open Margins" and "Cancel".
- The product margin list, form and graph, opened by that button, showing the fourteen measures of
  [`calculations.md`](calculations.md#18-margins-on-a-product--the-analysis-measures), with creation
  and editing disabled.

---

## 8. Data loaded into the point of sale terminal

The terminal re-implements the price engine client-side, so it must be given the same data. What is
loaded:

| What | Which records |
|---|---|
| Price lists | the available price lists of the configuration, the default price list, the price lists named by any preset, and every price list used as a base by a rule of any of those |
| Price list fields | the identifier, the name, the display name, the currency and the list of rule identifiers |
| Price list rules | the rules of the loaded price lists; on a first load, restricted to the loaded products and to rules valid at the moment of loading; on a later load, the rules changed since the previous load plus the rules whose start instant has passed since then |
| Price list rule fields | the product template, the product variant, the price list, the surcharge, the discount, the rounding step, the two margin bounds, the company, the currency, the start date, the end date, the computation kind, the fixed price, the percentage, the base price list, the base, the category and the minimum quantity |

The two implementations must agree exactly; the client-side engine itself is owned by
[point of sale](../point-of-sale/).

---

## 9. Reconciliation notes

1. **The export column headers.** One of the two descriptions this file was merged from expanded the
   grid's column headers into words; the other reproduced the shortened forms. The headers cross a
   system boundary and are read by integrations, so section 2 reproduces them exactly, in code font,
   with their meaning given in words beside them.

2. **The name of the export route's two values.** One description named the payload and the format in
   prose only. Both are part of the request contract and are reproduced in code font in section 2,
   with their meaning in words.

3. **The screens.** One description listed the screens as field lists; the other omitted them
   entirely. Section 7 keeps the field lists, because a rebuild needs to know which field appears
   where, and states plainly that nothing in that section changes behaviour.

4. **The vendor price display name.** One description gave only the vendor's display name; the other
   gave the enriched form and attributed it to the purchasing capability. The enriched form is
   contributed by the purchasing-and-inventory bridge; both forms are specified in
   [`entities.md`](entities.md#36-display-name) and the list views here show the fields that
   distinguish two offers of one vendor in either case.

5. **The scheduled jobs.** One description left the section empty; the other said the domain defines
   none. Section 6 says the second, and [`configuration.md`](configuration.md#12-scheduled-jobs) names
   the two jobs owned elsewhere that call into this domain.
