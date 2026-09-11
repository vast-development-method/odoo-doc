# Products and Catalog — Interfaces

Everything this domain exposes: user-visible navigation, the views and what each shows, named
remote operations with their inputs and outputs, web routes, printable documents, message
templates, external data contracts, and import and export formats.

---

## 1. Navigation

This domain ships **no menu entries of its own**. Every window action it defines is placed in a
menu by a consuming domain — the sales, purchasing, inventory or point-of-sale application decides
where "Products", "Product Variants", "Attributes" and "Categories" appear. The actions themselves
are listed below and are addressable by their identifiers.

| Action | Entity | View sequence | Default filters and context | Typical placement |
|---|---|---|---|---|
| Products (`product.product_template_action_all`) | Product Template | cards, list, form | none | The Products menu of every application that sells or buys |
| Product Variants (`product.product_normal_action`) | Product Variant | list, form, cards, activity | The variant search view; no forced first view | Under Products, revealed by the variant group |
| Product Variants, sellable (`product.product_normal_action_sell`) | Product Variant | cards, list, form, activity | Filter "Sales" pre-applied; the variant list view; the variant search view | The sales application |
| Product Variants of one template (`product.product_variant_action`) | Product Variant | list, form (simplified), cards | Grouped and defaulted on the template in context; creation disabled | The "Variants" button on the product form |
| Attributes (`product.attribute_action`) | Product Attribute | list, form | none | Configuration, revealed by the variant group |
| Categories (`product.product_category_action_form`) | Product Category | list, form | none | Configuration |
| Product Tags (`product.product_tag_action`) | Product Tag | list, form | none | Configuration |
| Combo Choices (`product.product_combo_action`) | Product Combo | list, form | none | Configuration, in applications that sell combos |
| Barcode Nomenclatures (`barcodes.action_barcode_nomenclature_form`) | Barcode Nomenclature | list, cards, form | none | Technical configuration |

### 1.1 Empty-state help

| Action | Text shown when the list is empty |
|---|---|
| Products | "Create a new product" |
| Product Variants | "Create a new product variant" followed by "You must define a product for everything you sell or purchase, whether it's a storable product, a consumable or a service." |
| Product Variants, sellable | "Create a new product variant" followed by "You must define a product for everything you sell, whether it's a physical product, a consumable or a service you offer to customers. The product form contains information to simplify the sale process: price, notes in the quotation, accounting data, procurement methods, etc." |
| Barcode Nomenclatures | "Add a new barcode nomenclature" followed by "A barcode nomenclature defines how the point of sale identify and interprets barcodes" |

Both product actions place the word "product" into the generic empty-state helper, so the framework's
standard "no product found" phrasing is used elsewhere.

---

## 2. Views

### 2.1 Product Template — form

One sheet with a picture, a title block, tab pages and a message thread. The blocks the domain
itself contributes:

- **Header buttons** — Print Labels; Documents (with a count badge); Variants (with a count badge,
  shown only under the variant group).
- **Title** — the favourite toggle, the name, the internal reference, the tags.
- **General information** — the type with its tooltip, the sellable and purchasable flags, the
  sales price, the cost, the category, the internal reference, the barcode, the unit and the
  additional packagings.
- **Combo choices** — shown only when the type is `combo`: the list of choice groups with their
  item counts and base prices.
- **Attributes and variants** — shown under the variant group: the attribute lines, each with its
  attribute, its values and a button opening the materialised values so that extra prices and
  exclusions can be set.
- **Sales** and **Purchase** description pages.
- **Properties** — the custom fields whose schema comes from the category.
- The message thread and the activity list.

### 2.2 Product Template — search

| Element | Behaviour |
|---|---|
| Product field | Matches when the template's internal reference contains the term, **or** any of its variants' internal references does, **or** the name does, **or** the barcode does |
| Category field | Matches the category **and all its descendants** |
| Tags field | Matches the template tags |
| Attributes field | Matches the attribute lines; shown only under the variant group |
| Filters | Goods (type is `consu`); Services (type is `service`); Combo (type is `combo`); Favorites; Sales (sellable flag true); Warnings (an activity exception is set); Archived |
| Activity filters | My Activities, Late Activities, Today Activities, Future Activities — all hidden by default |
| Groupings | Product Type; Product Category; Product Properties |

### 2.3 Product Variant — search

Inherits the template search and changes four things: the Tags field searches the **merged** tag set
(template tags plus variant tags); the Product field drops the variant-reference clause (it is
already searching variants); the Attributes field is replaced by the attribute-value field plus a
template field; and a "Product Template" grouping is added after the category grouping.

### 2.4 Product Variant — simplified form

Used by the "Variants" button. It shows the variant image with its fallback to the template image,
the combination as read-only tags, the internal reference, the barcode, the cost, the weight and
the volume, and a button opening the full template. Duplication is disabled on this view.

### 2.5 Product Variant — catalog card view

The view the catalog grid uses. Records are not draggable and the default order is favourite
descending, then internal reference, then name, then identifier. Each card shows:

- a context menu with an "Edit" entry and the favourite toggle;
- the name in bold, with the favourite marker beside it when set;
- an empty, identified block into which the client injects the price returned by the catalog
  contract;
- the attribute values as coloured tags, shown only under the variant group and only when there are
  any;
- the one-hundred-and-twenty-eight-pixel image, when there is one.

### 2.6 Product Variant — catalog search view

| Element | Behaviour |
|---|---|
| Product field | internal reference, name or barcode contains the term |
| Category field | the category and all its descendants |
| Attribute values field | shown under the variant group |
| Product Template field | |
| Filters | Favorites; Services; Goods |
| Groupings | Product Type; Product Category |
| Side panel | Categories (single select, list icon); Tags (multiple select, list icon, with counters) |

### 2.7 Barcode Nomenclature — form

Two groups and an explanatory block, then the rule list.

- **General** — the name and the conversion policy. The conversion policy is hidden when the
  nomenclature is a Global Standards One nomenclature.
- **Global Standards One** — the mode flag, and the separator expression which is shown only when
  the mode flag is set.
- **Explanatory text**, shown to the user verbatim:

  > *Barcodes Nomenclatures* define how barcodes are recognized and categorized. When a barcode is
  > scanned it is associated to the *first* rule with a matching pattern. The pattern syntax is that
  > of regular expression, and a barcode is matched if the regular expression matches a prefix of
  > the barcode.
  >
  > Patterns can also define how numerical values, such as weight or price, can be encoded into the
  > barcode. They are indicated by `{NNN}` where the N's define where the number's digits are
  > encoded. Floats are also supported with the decimals indicated with D's, such as `{NNNDD}`. In
  > these cases, the barcode field on the associated records *must* show these digits as zeroes.

- **Rules** — a reorderable list with a drag handle on the sequence, the name, the result type, the
  encoding (hidden when the nomenclature is a Global Standards One nomenclature, because it is
  forced) and the pattern. Under the Global Standards One mode the list also shows the content type,
  the decimal-usage flag (only for measure rules) and the associated unit (only for measure rules).
  New rules created from this list inherit the Global Standards One encoding through a context flag
  named `is_gs1` ("is Global Standards One").

### 2.8 Barcode Rule — form

The name, the sequence, the result type, the encoding (hidden for alias rules and for Global
Standards One rules), the pattern, the alias (shown only for alias rules), and, for Global Standards
One rules, the content type, the decimal-usage flag and the associated unit.

### 2.9 Product Document — views

- **Card view** — one card per document, with a ribbon reading "Variant" when the owner is a
  variant, the file preview, the name, the owner's name shown in italics for variant-owned
  documents, and an upload button.
- **List view** — sequence, name, owner model (hidden), kind.
- **Form view** — the name, the owner model (hidden), the owner reference shown as "Product
  Variant" when the owner is a variant, the kind, the content or the address, the company, the
  sequence.
- **Search view** — a filter selecting the documents of the variant named in the context, applied by
  default when the document list is opened from a variant.

### 2.10 Expiry views

| View | What it adds |
|---|---|
| Product Template form | A block with the use-expiration-date flag and, when it is set, the four day counts |
| Lot or Serial Number form | The four dates, made writable, plus the expiry alert indicator |
| Stock Quantity list | The removal date column, revealed when the product in context uses expiration dates |
| Stock Move form | The expiration date and removal date columns on the detailed operations |
| Configuration screen | The expiry-on-delivery-slip setting |

---

## 3. Named remote operations

These are the operations a caller outside the domain invokes by name. Each is listed with its
receiver, its inputs and its output.

| Operation | Receiver | Inputs | Output |
|---|---|---|---|
| Get the single-variant shortcut | one Product Template | none | An empty record when the template has more than one variant or is configurable; otherwise a record with the variant's identifier under `product_id` and its display name under `product_name` |
| Get the attribute exclusions | one Product Template | an optional parent combination, an optional parent product name, an optional list of the combination's value identifiers | The six-entry payload of [calculations.md](calculations.md), section 4.6 |
| Get the template matrix | one Product Template | an optional company, an optional currency, a flag saying whether extra prices are displayed | A record with `header` (the header row) and `matrix` (the list of rows) |
| Get the contextual price | one Product Template or one Product Variant | optionally an explicit variant; the pricelist, quantity, unit and date are read from the context | The unit price under the contextual pricelist |
| Open the label dialog | a set of Product Templates or Product Variants | none | A window action opening the label layout dialog, pre-filled with the selection; raises for services |
| Open the documents | one Product Template or one Product Variant | none | A window action over the document entity, filtered to the product's documents, with the product pre-filled as the default owner |
| Open the template | one Product Variant | none | A window action opening the template's form in a dialog |
| Open the attribute's product lines | one Product Attribute | none | A window action over the template attribute lines of this attribute on active templates |
| Open the line's materialised values | one Template Attribute Line | none | A window action over the line's Template Attribute Values, using the dedicated list and form views, with the active filter pre-applied and the product column hidden |
| Add the value to all products | one Attribute Value | none | A window action opening the bulk-update dialog in add mode |
| Update the extra prices | one Attribute Value | none | A window action opening the bulk-update dialog in price mode |
| Open the packaging barcodes | one Unit of Measure | none | A window action over the packaging barcodes of that unit |
| Add from catalog | one document implementing the catalog contract | none | A window action opening the catalog grid (see section 5) |
| Render the pricelist report as markup | the pricelist report builder | a data record carrying the quantities, the pricelist identifier, the active model, the active identifiers and the title flag | Rendered markup for the on-screen report |
| Get the import templates | Product Template, Vendor Pricelist Line | none | A list of label-and-path records (see [configuration.md](configuration.md), section 11) |

### 3.1 Server actions bound to the variant entity

Two actions appear in the contextual action menu of a variant list:

| Action | Group required | Effect |
|---|---|---|
| Print Labels | every internal user | Opens the label dialog for the selection |
| Pricelist Report | the pricelist group | Opens the client-side pricelist report, carrying the current context |

---

## 4. Web routes

| Path | Method | Authentication | Read-only | Purpose |
|---|---|---|---|---|
| `/product/catalog/order_lines_info` | remote procedure call over the web transport | authenticated user | yes | Returns the catalog payload for a set of products on a given document. Inputs: the document's transport name under `res_model`, the document identifier under `order_id`, the list of variant identifiers under `product_ids`, plus any keywords the document understands. Output: a map from variant identifier to the payload of section 5.2. Executed with the document's own company as the acting company. |
| `/product/catalog/update_order_line_info` | remote procedure call over the web transport | authenticated user | no | Creates, changes or removes the document's line for one product. Inputs: `res_model`, `order_id`, `product_id`, `quantity` (defaulting to zero), plus any keywords. Output: the resulting unit price. Executed with the document's own company as the acting company. |
| `/product/document/upload` | hypertext transfer protocol, post only | authenticated user | no | Uploads one or more files as product documents. Inputs: the files under the form field `ufile`, the owner transport name under `res_model`, the owner identifier under `res_id`. Rejects any owner model other than the product variant or the product template, a missing record, or a caller without write access — in each case by returning an empty response. Output: a structured object holding either `success` with the text "All files uploaded" or `error` with the failure text. |
| `/product/export/pricelist/` | hypertext transfer protocol | authenticated user | yes | Exports the pricelist report. Inputs: `report_data`, a structured string carrying the report parameters, and `export_format`, either the comma-separated-values format or the spreadsheet format. Output: a file download named "Pricelist - *the pricelist name*" with the matching extension. |

### 4.1 The pricelist export file

Both formats have the same shape:

| Column | Content |
|---|---|
| Product | the product name for a template, the display name for a variant |
| `UOM` (the literal column header; it abbreviates "unit of measure") | the product's default unit name |
| Quantity (*q* UoM) | one column per requested quantity, holding the pricelist price at that quantity |

Rows are one per product; when a template has more than one variant, the template row is replaced by
one row per variant. In the spreadsheet format each column is widened to the longest cell it
contains.

---

## 5. The product catalog data contract

Any order-like document that wants to be filled from the catalog grid implements this contract. It
is the contract, not the implementation, that other domains depend on.

### 5.1 What the document must provide

| Question | Default answer |
|---|---|
| Which products may be shown? | Products whose company is empty or is a parent of the document's company, and whose type is not `combo` |
| What extra context does the grid need? | Whether the unit column is shown (the acting user holds the units group), the document's identifier, the document's transport name |
| Which of my lines correspond to these products? | Nothing — the document must answer |
| What does a product with no line look like? | Quantity zero and a read-only flag from the document's own read-only test |
| What happens when a quantity is typed? | Nothing, returning a price of zero — the document must answer |
| Am I read-only? | No — the document may answer otherwise |

The document's line entity must be able to produce its own per-line payload.

### 5.2 The per-product payload

| Key | Type | Meaning |
|---|---|---|
| `productId` | whole number | The variant's identifier |
| `quantity` | decimal, optional | The quantity currently on the document for this product |
| `productType` | text | The variant's type: `consu`, `service` or `combo` |
| `price` | decimal | The unit price for this product on this document |
| `uomDisplayName` | text | The unit shown on the card; filled from the product's own unit when the line did not supply one |
| `code` | text, optional | The variant's reference, or the empty string |
| `readOnly` | flag, optional | Whether the card accepts input |

The payload is assembled in two passes: first for the products that already have lines, then for the
remaining products from the defaults. A product that has a line is never overwritten by the default
pass.

### 5.3 The catalog action

The action opens the variant entity with the catalog card view and the catalog search view, with
the document's filter and with the merged context. Any pre-existing form-view override in the
context is removed, so that opening a card always opens the standard product form.

---

## 6. Printable documents

| Document | Entity | Paper format | File name | Content |
|---|---|---|---|---|
| Product Label 2x7 (portable document format) | Product Template | A4 Label Sheet | "Products Labels - *the product name*" | Two columns by seven rows, with the price |
| Product Label 4x7 (portable document format) | Product Template | A4 Label Sheet | same | Four by seven, with the price |
| Product Label 4x12 (portable document format) | Product Template | A4 Label Sheet | same | Four by twelve, with the price |
| Product Label 4x12 No Price (portable document format) | Product Template | A4 Label Sheet | same | Four by twelve, without the price |
| Product Label (portable document format) | Product Template | Dymo Label Sheet | same | One label per page, sized for a single-label printer |
| Packaging Barcodes (portable document format) | Packaging Barcode | default | "Products packaging - *the unit name*" | Per packaging barcode: the unit name, the product's display name, a "Qty:" line holding the unit's relative factor and its reference unit (the reference unit shown only under the units group), the barcode rendered as a bar symbol with automatic symbology selection, and the barcode string in text |
| Pricelist | Product Variant | default | — | The pricelist report; also rendered on screen as markup |

### 6.1 Label content

Each label carries the product's name with internal-reference display switched **off**, the barcode
rendered as a bar symbol, and — for the price-bearing formats — the price under the dialog's chosen
pricelist. The dialog's extra markup is placed on every label. The products are re-read ordered by
name descending, because the template consumes the product map from its end; the printed order is
therefore ascending by name.

### 6.2 The pricelist report data

The report builder receives: the quantities to price (defaulting to a single quantity of one), the
pricelist identifier (falling back to the first pricelist when the given one does not exist), the
active model (defaulting to the template entity), the active identifiers, and whether the pricelist
title is displayed. It returns, per product: the identifier; the name — the plain name for a
template, the display name for a variant; a map from quantity to the pricelist price at that
quantity; the unit name; and, for a template with more than one variant, the same structure
recursively for each variant.

---

## 7. Message templates and notifications

This domain defines **no message template of its own**. It provides the hook that sends one.

| Hook | Trigger | What is sent |
|---|---|---|
| Product e-mail on invoice posting | A customer invoice is posted | For each invoice line whose product carries a message template, a message rendered from that template is posted on the invoice with the light notification layout and the comment subtype, which notifies the invoice's followers. One message per line. When the posting is performed by the system rather than a user, the send is re-attributed to the superuser. |

The expiry capability schedules an **activity**, not a message: a to-do activity with the summary
"Alert Date Reached" and the note "The alert date has been reached for this lot/serial number".

---

## 8. Client-side contracts

### 8.1 The scan collector

The client assembles keystrokes into a scan. Two keystrokes belong to the same scan when they are
at most `barcode.max_time_between_keys_in_ms` milliseconds apart; the value is delivered in the
session payload and only to internal users, defaulting to one hundred and fifty.

A form participates by declaring the scan field and implementing the scan handler. The handler
receives the assembled string; the field is blanked before the handler runs, so a second scan is
never confused with the first.

A manual-entry component lets a user type a barcode when no scanner is available, and a camera
component lets a mobile device read one; both feed the same handler.

### 8.2 The client-side parser

The client carries its own implementation of the parsing algorithms of
[calculations.md](calculations.md), sections 10 and 11, so that a scan can be interpreted without a
round trip. The two implementations must agree exactly; in particular:

- the check-digit arithmetic;
- the encoding check including the "a thirteen-digit code starting with zero is not a thirteen-digit
  European Article Number" clause;
- the brace grammar and the base-code zeroing;
- the conversion policy;
- the Global Standards One decomposition, including the symbology-identifier stripping list, the
  separator group, the content-type interpretations and the century-determination rule;
- the abandon-everything behaviour when a Global Standards One string cannot be fully decomposed.

Because the client's pattern engine and the server's pattern engine are different, patterns must be
written in the subset both accept. This is why the Global Standards One rules use positional groups
rather than named groups.

### 8.3 The matrix dialog

The client opens a grid from the matrix payload. It renders the header row, the row headers, and one
input per cell; cells whose possibility flag is false are rendered as unavailable. On confirmation
it returns the list of (value identifiers, quantity) pairs with a non-zero quantity.

### 8.4 The catalog grid

The client renders the catalog card view, calls the order-lines-information route for the visible
page, injects the returned price into each card's price block, and calls the
update-order-line-information route whenever a quantity changes.

### 8.5 The scannable decimal field

A decimal field may be marked as scannable, in which case scanning a value-carrying barcode writes
the decoded numeric value into it rather than the barcode string.

---

## 9. Import and export formats

### 9.1 Importing products

The import of the template entity accepts an extra column matched to the product-values field. Its
cells hold comma-separated `attribute:value` pairs, for example:

```
Size:L,Colour:White
```

Rules:

- the attribute name and the value name are separated by the **first** colon, and both are trimmed;
- an attribute that does not exist is created with variant creation **dynamic** and display type
  **radio**;
- a value that does not exist is created under that attribute;
- a template that does not exist is created from the row's required fields only;
- empty cells are filled from the first variant of the template;
- rows with an empty product-values cell are imported as plain templates.

The full sequence is in [workflows.md](workflows.md), section 9, and the error messages are in
[business-rules.md](business-rules.md), section 3.5.

### 9.2 Exporting products

Exporting a variant with the product-values column produces the comma-joined, **alphabetically
sorted** list of its `attribute:value` pairs, which is exactly the form the import accepts. A
round trip is therefore lossless for the combination.

### 9.3 The pricelist export

Specified in section 4.1.

---

## 10. External service integrations

This domain integrates with no external service. It defines the two contracts that make external
identification work — the barcode nomenclature and the Global Standards One parser — but it never
calls out.

Where a consuming domain needs a global identifier, it uses the barcode field and a nomenclature;
where it needs a party identifier in the Global Standards One scheme, that is specified in
`../electronic-invoicing-and-document-exchange/`.

---

## 11. Fields other domains rely on

An implementation must keep these names and meanings stable, because contracts outside the domain
depend on them.

| Name | On | Depended on by |
|---|---|---|
| `product.template`, `product.product` | the two product entities | every operational domain |
| `product_tmpl_id` | Product Variant | every domain that groups variants |
| `product_template_attribute_value_ids` | Product Variant | the configurator, the matrix, the storefront |
| `product_no_variant_attribute_value_ids` | order lines (owned elsewhere) | the extra-price computation of this domain |
| `combination_indices` | Product Variant | the combination lookup |
| `barcode` | Product Variant, Packaging Barcode | scanning, labels, the uniqueness checks |
| `default_code` | Product Template, Product Variant | every printed document and display name |
| `list_price`, `lst_price`, `standard_price`, `price_extra` | the two product entities | pricing, margins, valuation |
| `uom_id`, `uom_ids` | Product Template | unit conversion and packaging |
| `type` with the values `consu`, `service`, `combo` | Product Template | every domain that branches on what a product is |
| `categ_id` | Product Template | valuation, reporting, properties |
| `use_expiration_date`, `expiration_time`, `use_time`, `removal_time`, `alert_time` | Product Template | the expiry behaviour of inventory |
| `expiration_date`, `use_date`, `removal_date`, `alert_date`, `product_expiry_alert` | Lot or Serial Number | first-expiry-first-out, the forecast report, the delivery guard |
| `nomenclature_id` | Company | every scanning surface |
