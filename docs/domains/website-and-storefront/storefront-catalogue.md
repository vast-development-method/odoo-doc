# Storefront catalogue

The public catalogue: how the shop listing is built, how categories, attributes, tags and prices filter
it, how it is sorted and paginated, how the product page is assembled, how an attribute selection is
resolved into a priced variant, and what the combination information service returns. It also covers
ribbons, media, documents, extra fields, base unit prices, cross-selling links, the price list selector,
the tax display mode and the product syndication feed.

Fields are specified in [entities.md](entities.md) Part 5 and §6.7 to §6.10; the arithmetic is in
[calculations.md](calculations.md); the numbered rules are in [business-rules.md](business-rules.md).

---

## 1. Reaching the shop

### 1.1 Access guard

Every shop page, product page and cart page first evaluates the shop-access check:

```formula
shop access = NOT ( the current user is the site's public user
                    AND the site's shop access setting is "logged_in" )
```

When the check fails, a shop listing page or a product page is redirected to the sign-in page carrying the
requested path as the return path, and the cart page is redirected to the sign-in page without a return
path. In addition, every site menu whose address starts with the shop path is hidden from the navigation
of anonymous visitors on such a site, and the site index generator emits no shop or category entry unless a
query string is supplied.

### 1.2 Addresses of the catalogue

| Path pattern | Renders |
|---|---|
| `/shop` | The full listing, first page. |
| `/shop/page/<page number>` | The full listing, page n. |
| `/shop/category/<category slug>` | The listing restricted to a category and its descendants, first page. |
| `/shop/category/<category slug>/page/<page number>` | The same, page n. |
| `/shop/<product slug>` | The product page. |
| `/shop/<category slug>/<product slug>` | The product page reached through a category, which changes only the breadcrumb. |
| `/shop/product/<product slug>` | A compatibility address; always answers with a permanent redirect to the canonical product address. |

Redirect rules:

1. A category supplied as a query parameter instead of a path segment produces a permanent redirect to the
   path form, with the category parameter stripped from the query string.
2. A product page reached with a category to which the product does not belong, directly or through a
   descendant category, produces a permanent redirect to the canonical product address, keeping the
   remaining query string.
3. A product or category that no longer exists or is not readable produces: a redirect to the category
   listing when the category is readable but the product is not; a redirect to the product address when the
   product is readable and no category was given; and otherwise "not found", never "forbidden". The listing
   endpoint always answers "not found" on an access failure (WS-461).
4. The canonical address advertised for search engines strips the category segment from a product address
   and preserves the language prefix when the request language is not the default one.

---

## 2. Building the listing

The listing endpoint performs the following steps in order.

1. **Access guard** (§1.1).
2. **Category resolution.** The category parameter is validated: a value that is neither a number nor a
   record raises `Invalid category.`; a value that does not exist or is not reachable from the current site
   yields no category; otherwise the category record is used. When the value arrived as a query parameter,
   a permanent redirect to the path form is issued.
3. **Price bounds parsing.** The minimum and maximum prices are parsed as decimals; an unparsable value
   becomes zero.
4. **Layout parameters.** The page size defaults to 21 when unset or zero, the number of columns defaults
   to 4 when unset or zero, and the gap defaults to `16px`.
5. **Attribute filter parsing.** Every repetition of the attribute filter parameter is parsed (§3.2). When
   at least one is present the raw list is stored in the session; when none is present that session key is
   cleared.
6. **Tag filter parsing.** Only when the tag filter page option is active: the tag parameter is split on
   commas and each element is read back into a tag identifier.
7. **Price list freshness.** When the session holds a price list resolution moment older than 3600 seconds,
   the cached price list identifier is dropped and the moment is restarted, which forces a fresh
   resolution.
8. **Currency conversion rate.** Only when the price filter page option is active: the rate from the
   company currency to the display currency at today's date is read; otherwise the rate is 1.
9. **Search.** The approximate search is executed with no limit, the computed order and the search options
   (§3).
10. **Condition rebuild.** The listing condition is rebuilt from the possibly corrected search term, the
    category and the attribute filter, because the price aggregate and the category tree need it.
11. **Price bounds and clamping.** Only when the price filter is active (§3.5).
12. **Tag list.** Only when the tag filter is active and the search returned at least one product: every
    tag that is visible to customers, is attached to at least one published template or published variant
    and matches the site condition.
13. **Category tree.** §4.
14. **Pagination.** The pager is built from the total count, the requested page, the page size and a window
    of five page links ([calculations.md](calculations.md) §5).
15. **Product slice.** The products of the current page are read, with binary fields fetched as sizes
    rather than as content.
16. **Variant mapping.** For each product of the slice, its first possible variant is resolved with
    elevated privileges and prefetched.
17. **Attribute panel.** §3.2.
18. **Layout mode.** The list mode when the list-view page option is active, and otherwise the grid mode. A
    shopper's explicit choice is kept in the session and is cleared whenever the list-view or the
    grid-or-list page option is toggled.
19. **Prices.** The catalogue price payload of the slice is computed once for every product (§6).
20. **Grid packing.** §5.
21. **Preview values.** The attribute-value previews of the slice are computed lazily, that is only when
    the page template actually asks for them.

The rendering values handed to the page are: the corrected search term; the original term when it was
corrected; the sort order; the category; the parsed attribute filter and the flat set of selected attribute
value identifiers; the pager; the product slice; the variant mapping; the full search result; the result
count; the packed grid rows; the layout parameters; the top-level categories; the category entries; the
filter attributes; the address builder; the identifiers of the categories that contain search results; the
layout mode; a price accessor; the shop path; the product page query parameters; the grouped selected
attribute values; the lazy preview values; the automatically assigned ribbons; and, when the corresponding
filters are active, the price bounds and the tag list.

---

## 3. Searching and filtering

### 3.1 Search fields and approximate matching

The search reads the following fields of the Product Template: the name and the aggregated variant
references, plus the sales description and the storefront description when descriptions are displayed.

The aggregated variant references field is the concatenation of the internal references of all variants of
the template, joined by a rarely used control character, which makes a search on any variant reference
match its template. The four searched fields, plus the internal reference itself, carry similarity indexes,
accent-insensitive where the storage engine supports it, which is what makes similarity matching
affordable.

The search term is matched as follows: an exact containment match is attempted first; when it returns
nothing, the closest term by similarity is proposed and searched instead, and both the corrected term and
the original term are returned, which lets the page print the substitution. Approximate correction can be
disabled per request. The algorithm is [calculations.md](calculations.md) §4.

The non-approximate condition used to rebuild the listing condition splits the term on spaces and requires
every word to be contained, case-insensitively, in at least one of the search fields:

```formula
listing condition = the site sellable-product condition
                AND for each word of the term: the word is contained in at least one search field
                AND the storefront categories are the selected category or one of its descendants,
                    when a category is selected
                AND for each attribute of the attribute filter:
                        the product offers at least one of the selected values of that attribute
```

The shop listing and the site-wide search box use the same field set; a rebuild must keep them identical or
results will differ between the two entry points.

### 3.2 Attribute filter

The filter travels as a repeated query parameter. Each repetition is the attribute identifier, a hyphen and
one or more value identifiers separated by commas. Parsing produces a mapping from attribute identifier to
the list of selected value identifiers.

The condition adds one clause per attribute; the clauses are combined with a logical conjunction while the
values inside one clause are combined with a logical disjunction. Selecting two colours therefore widens
the result, and selecting a colour and a size narrows it.

The filter panel lists the attributes that actually occur in the current result set: the attribute lines of
every product matching the listing condition are grouped by attribute, keeping only the attributes whose
storefront visibility is visible, in attribute order. When the result set is empty, the panel falls back to
the attributes whose identifiers are present in the current filter, sorted, so that the shopper can still
clear their selection.

### 3.3 Category filter

A selected category filters on "the storefront categories are the category or one of its descendants",
evaluated through the materialised path, which includes the category itself.

### 3.4 Tag filter

Active only when the tag filter page option is enabled. The condition keeps a product when one of the
selected tags is among its template tags or among the additional tags of one of its variants. The offered
tag list is restricted to tags marked visible to customers, attached to at least one published template or
variant, and matching the site condition.

### 3.5 Price filter

Active only when the price filter page option is enabled. The bounds are expressed in the display currency
while the stored sales price is expressed in the company currency, so a conversion rate is applied on both
sides; the formulas, the aggregate and the clamping are [calculations.md](calculations.md) §13.1.

The bounds shown on the slider are the shopper's values when given and the available bounds otherwise; the
available bounds are rounded to two decimal places for display. When the shopper switches to a price list
in another currency, the bounds present in the previous address are converted, without rounding, and
written back into the address.

### 3.6 Sort order

```formula
effective order = "publication flag descending, " + the requested order or the site default sort
                  + ", identifier descending"
```

The five offered orders are: the shop ordering value ascending, which is the featured order; the
publication date descending, which is the newest-arrivals order; the name ascending; the sales price
ascending; and the sales price descending. The publication prefix keeps unpublished products, visible only
to internal users, at the end; the identifier suffix makes the order total and therefore the pagination
stable.

---

## 4. Category navigation

Two category collections are produced.

**The navigation tree** is the set of top-level categories matching the site condition. When a search term
is active it is additionally restricted to the ancestors-and-self of the categories that contain at least
one matching product, so that the tree only offers branches that lead somewhere.

**The category entries**, that is the strip of clickable categories shown above the grid, are built as
follows.

1. When a category is selected, the entries are the children of that category that are reachable from the
   current site.
2. When a search term is active, the entries are restricted to the categories containing results.
3. When the entries are then empty, the parent of the selected category is taken and its children,
   reachable from the current site, become the entries; step 2 is applied again.
4. When no search term is active and the user is not internal, the entries are restricted to categories
   whose published-products flag is true.
5. When no category is selected, the entries are the navigation tree.

The fallback to the siblings in step 3 is what makes a leaf category show its siblings instead of an empty
strip. Public and portal users are additionally restricted by a record rule to categories whose
published-products flag is true, which hides empty branches without the listing having to test for it.

---

## 5. Grid packing

Product tiles may span several grid cells: a width and a height in cells. The packing procedure, with its
stop rule and a worked example, is [calculations.md](calculations.md) §13.2.

The ribbon written in the cell is the template ribbon. The automatic ribbons are resolved at render time
from the list of non-manual ribbons passed to the page, using the applicability rules of
[entities.md](entities.md) §5.3, with the catalogue price payload as price data.

---

## 6. Catalogue prices

For a slice of product templates, one payload is produced per template. The price list is asked once for
the whole slice, for quantity one, and each answer is then processed:

1. The product taxes are filtered to the current company and mapped through the request fiscal position.
2. The price shown is the price list price passed through the tax treatment of
   [calculations.md](calculations.md) §8.
3. When the applied rule shows a discount on the shop ([calculations.md](calculations.md) §9.1), the rule's
   price before discount is requested for quantity one, today, in the product unit and in the display
   currency; when it is strictly greater than the price list price at currency precision, it becomes the
   reference price and is passed through the same tax treatment.
4. When no reference price resulted, the comparison-price feature is enabled and the template carries a
   non-zero comparison price, the comparison price is converted from the product currency to the display
   currency, without rounding, and becomes the reference price. It is never taxed.

The payload therefore has one mandatory value, the price the shopper pays for one unit in the display tax
mode, and one optional value, the strikethrough price. The strikethrough comes either from the price list
rule, a genuine discount, or from the manually entered comparison price, never from both, and the
comparison price is never shown when a price list discount already applies.

A worked example is [calculations.md](calculations.md) §11.

---

## 7. The product page

### 7.1 Values assembled

1. The structured product description for search engines (§12), as a list starting with the product's own
   description.
2. The category: the one in the address, otherwise the first of the product's storefront categories
   reachable from the current site. When a category is known, a breadcrumb description is appended to the
   structured description.
3. The address builder used by the "back to the shop" link, which carries the last attribute filter kept in
   the session.
4. The combination information (§8). When the address carries a comma-separated list of attribute value
   identifiers, the initial combination is built by walking the product's attribute lines and selecting,
   per line, the first active template value whose attribute value is in the requested set, failing that
   the first active template value, unless the attribute is a multiple-choice attribute, in which case the
   line contributes nothing. Otherwise the default combination is used.
5. The full list of top-level storefront categories.
6. The resolved variant of the combination.
7. The page view tracking marker used to record the product view.
8. The shop path and, when the stock capability is installed, the shopper's address for the back-in-stock
   form, taken from the user record or from the session.
9. When collection in store is enabled: the selected store payload, whether a store selector button must be
   shown — only when the collect-in-store method has more than one store — and a postal code taken from the
   delivery address, then from the selected store, then from the visitor's geolocated postal code, and
   finally the empty text.

### 7.2 Blocks rendered

| Block | Content and condition |
|---|---|
| Media | The carousel or the grid, built from the media list of the variant — variant picture, variant extra media, template extra media — or of the template. Suppressed entirely when the media width setting hides media. |
| Name and price | The display name of the combination, the price, the strikethrough price, the per-unit price and the tax indication when that page option is active: `(Tax excluded)` or `(Tax included)` according to the site setting. |
| Storefront description | The short storefront description, printed under the name. |
| Attribute selection | One control per attribute line that offers a choice, rendered as radio buttons, a select list, colour swatches, pills or pictures according to the attribute display type; multiple-choice attributes render as check boxes; a value flagged as accepting a custom value opens a free-text input. |
| Informative attributes | The attribute lines that have exactly one value, grouped by attribute, printed below the configurator. |
| Specification table | The attribute lines grouped by attribute category, with the single-value custom attributes removed. |
| Quantity and actions | The quantity input, the add-to-cart button, the optional buy-now button, the wish list button and the comparison button. |
| Contact button | Replaces the add-to-cart button when the price is hidden; its destination is the site's contact button address. |
| Documents | The active documents of the template flagged for the product page, each a download link. |
| Extra fields | The declarations of the site, printed as described in [entities.md](entities.md) §5.6. |
| Tags | The tags of the variant, or of the template when no variant is resolved, that are visible to customers. |
| Reviews | The rating and discussion block when that page option is active. Only ratings by users who are not internal count towards the average and the count. |
| Alternative products | A content block listing the alternative templates. |
| Cross-selling blocks | Content blocks driven by the accessory, sold-with and alternative filters. |
| Availability | The stock block described in [storefront-stock-and-pickup.md](storefront-stock-and-pickup.md) §2. |

### 7.3 Page geometry derivations

```formula
media columns and detail columns, out of twelve:
    hidden        → 0 and 12
    50 percent    → 6 and 6
    66 percent    → 8 and 4
    100 percent   → 12 and 12
grid spacing  : none → gap 0, small → gap 1, medium → gap 2, big → gap 3
grid rounding : one presentation token per value, none, small, medium and big
page container = the product page container, or the shop page container when it is unset
shop picture ratio = the first ratio token present in the product tile design tokens among
                     sixteen-by-nine, four-by-three, six-by-five, four-by-five and two-by-three,
                     otherwise one-by-one
shop picture height = 36 units for sixteen-by-nine, 48 for four-by-three, 53 for six-by-five,
                      96 for four-by-five and 64 otherwise
```

---

## 8. The combination information service

This is the contract that the product page, the configurator dialogues, the comparison page and every
storefront price display rely on. It answers the question: given a product template, an optional variant, a
set of chosen attribute values, a quantity and a unit of measure, what must the storefront show?

### 8.1 Inputs

| Input | Type | Default | Meaning |
|---|---|---|---|
| Combination | list of template attribute values | empty | The chosen values. |
| Variant | record | none | A known variant. |
| Quantity | decimal | 1.0 | The quantity for which the price must be computed, because price list rules may depend on it. |
| Unit of measure | record | none | The unit for which the price must be computed; falls back to the product's unit. |
| Template only | boolean | false | When true, ignore the combination and never resolve a variant. |

### 8.2 Variant resolution

1. When no variant, no combination and no template-only flag were given, the combination becomes the first
   possible combination of the template.
2. When the template-only flag is set, no variant is resolved.
3. Otherwise, when a variant was given, that variant is taken; when the requested combination contains
   values the variant does not carry, the variant matching the combination is taken instead.
4. Otherwise the variant matching the combination is taken.
5. The product or template is the resolved variant when there is one and the template otherwise.
6. The combination is the given one when there is one, and otherwise the attribute values of the resolved
   variant.

The service deliberately does **not** check that the combination is possible; it reports possibility
instead, as one of the returned values.

### 8.3 Display name

```formula
display name = the display name of the product or template, with the internal reference suppressed
display name = that name + " (" + the combination name + ")"
               when no variant was resolved and the combination has a name
```

### 8.4 Returned payload

Values always present:

| Value | Type | Meaning |
|---|---|---|
| Variant | integer | The resolved variant identifier, or 0 when none. |
| Template | integer | The template identifier. |
| Display name | text | §8.3. |
| Combination possible | boolean | Whether the requested combination is an allowed one for the template. |
| Price | decimal | The unit price the shopper pays, in the display currency and in the site tax display mode, for the requested quantity and unit. |
| Reference price | decimal | The strikethrough reference, the greater of the price and the price before discount, in the same currency and tax mode. |
| Discount flag | boolean | True when the price before discount is strictly greater than the price. |
| Extra-price flag | boolean | True when the applied price list rule is not a fixed-price rule, that is when adding an attribute extra still changes the price. |
| Zero-price flag | boolean | True when the site forbids zero-price sales and the computed price is zero. |
| Currency precision | integer | The number of decimal places of the display currency; added by the endpoint. |

Values present under conditions:

| Value | Condition | Meaning |
|---|---|---|
| Comparison price | No price list discount applies, the template carries a comparison price and the comparison-price feature is enabled. Forced to 0 when the zero-price flag is true. | The manually entered strikethrough price, converted to the display currency, never taxed. |
| Reference unit label and price per reference unit | The per-unit price feature is enabled. | §10. |
| Analytics payload | The site has an audience measurement identifier. | The item identifier, the item name, the item category, the currency and the reference price. |
| Tax disclaimer | The product is a combo, the site shows prices tax included, and at least one product of the combo choices carries a tax that is not price-included. | The text `Final price may vary based on selection. Tax will be calculated at checkout.` |
| Discount start and end moments | Always computed; removed before the payload leaves the service. | The validity window of the applied price list rule, used by the product feed. |
| The stock block: storable flag, out-of-stock ordering flag, availability threshold, quantity free to use, cart quantity, unit name, unit rounding, availability display flag, out-of-stock message, subscription flag, subscription address, combo maximum | The stock capability, and only when the caller asked for quantities. | See [storefront-stock-and-pickup.md](storefront-stock-and-pickup.md) §2. |
| In the wish list | The wish list and stock capabilities, and only when the caller asked for it. | Whether the variant is already saved. |
| Collection availability block | The collect-in-store capability, for a storable variant not excluded by tag. | See [storefront-stock-and-pickup.md](storefront-stock-and-pickup.md) §6.9. |
| No product change | The resolved variant equals the requested one. | Tells the page that the media and tag blocks need not be re-rendered. |
| Picture presence and carousel | The media width setting does not hide media, media rendering was not suppressed, and the variant changed. | Whether a picture exists, and the re-rendered media block. |
| Product tags | The tag page option is active and the variant changed. | The re-rendered tag block. |

Values that exist only inside the service and are stripped before the payload is returned to a client: the
combination itself, the currency, the date, the product taxes, the mapped taxes and the two discount
moments.

### 8.5 Price computation inside the service

The steps are those of [calculations.md](calculations.md) §9, evaluated with the request price list, the
display currency, the request fiscal position and today's date. The price list price is **not** converted:
the price list is asked for a price in the display currency directly. The comparison price, in contrast, is
converted from the product currency and is never taxed, because it is meant to be printed exactly as
entered.

### 8.6 Attribute extra prices

Each selectable attribute value shows an extra amount next to its label, computed by
[calculations.md](calculations.md) §12.2: zero when the value has no extra or when the extra-price flag is
false, otherwise the extra converted into the display currency and passed through the same tax treatment as
the price. A fixed-price rule sets the extra-price flag to false, because in that case the extra would not
change what the shopper pays.

### 8.7 Variant creation from a combination

The storefront may create a variant on demand for templates whose attributes are configured for dynamic
variant creation.

* Input: the list of template attribute value identifiers.
* Output: the identifier of the variant matching the combination, or 0.
* The operation returns the same value whether the variant already existed or was created, and returns 0 for
  every failure, which prevents a caller from probing which templates use dynamic creation (WS-462).
* The operation is reachable by anonymous visitors. A rebuild must accept that this allows the creation of
  unused variants and must not add an error channel that leaks configuration; the recommended mitigation is
  WS-906.

### 8.8 When the configurator dialogue is shown

```formula
show the configurator =
       the template has at least one optional product that may be shown
    OR NOT ( the template resolves to a single variant OR the caller says it is already configured )
    OR ( the template offers more than one unit of measure
         AND the caller does not say it is already configured )
```

An optional product may be shown when it passes the standard configurator test, can be added to the cart
under the parent combination, and matches the site condition.

---

## 9. Ribbons on the storefront

The ribbon shown for a product is resolved by the rule of [entities.md](entities.md) §5.3. The precedence
is: the variant ribbon, then the template ribbon, then the first applicable automatic ribbon in ordering
order. The shipped automatic candidates, in ordering order, are `Sold out` at 1, `Out of stock` at 2,
`Sale` at 3 and `New!` at 4. None of them carries an automatic assignment mode by default; an administrator
sets the mode on the one they want automated, and at most one ribbon may carry each mode (WS-303).

**Worked example.** A product was published 12 days ago, has no manual ribbon, and its catalogue payload
carries a reference price of 120.00 and a price of 99.00. The `Sale` ribbon carries the sale mode at the
ordering value 3; the `New!` ribbon carries the new mode with a period of 30 days at the ordering value 4.
Both are applicable; the candidates are evaluated in ordering order, therefore `Sale` wins.

---

## 10. Base unit price

A product may declare how many reference units one sales unit contains, and the label of that reference
unit.

```formula
price per reference unit = 0                          when the base unit count = 0
price per reference unit = price ÷ base unit count    otherwise
reference unit label = the reference unit name when set, otherwise the unit of measure name
```

On a template, the base unit count and the reference unit mirror the single variant when the template has
exactly one variant, and are zero and empty otherwise; writing them writes through to that single variant.
On the storefront, the value shown next to the price is computed from the price actually displayed, after
converting it from the requested unit to the product unit. In the cart, the per-unit price of a line is
computed from the line's own price divided by the line quantity, then divided by the base unit count.

The count is stored with unlimited decimal precision on purpose, so that ratios such as one unit inside a
box of 10000 are representable. A worked example is [calculations.md](calculations.md) §12.1.

---

## 11. Product media

### 11.1 Ordered media list

```formula
media of a variant  = the variant itself, then its own extra media, then the template extra media
media of a template = the template itself, then the template extra media
```

The first entry is the main picture: it is the variant or the template record itself, not a media row. The
remaining entries are media rows, ordered by their ordering value and then by their identifier.

### 11.2 Adding media

The add-media operation accepts either pictures or one video and requires the Restricted Editor group;
without it the answer is "not found".

* Pictures: each selected attachment produces a media row whose caption is the attachment name and whose
  picture is the attachment content; an attachment created from a remote address whose content is empty is
  fetched from that address first.
* Video: exactly one row is produced, carrying the video address, a caption defaulting to a generic video
  name and, when it can be fetched, the video thumbnail as the picture. A missing video address is rejected
  with `Invalid video URL provided.`, whose three capital letters abbreviate uniform resource locator.

Where the rows are attached:

1. When a variant was given and no template, the template is that variant's template.
2. When no variant was given, a template was given and the template has dynamic attributes, the variant
   matching the given combination is used, and created when it does not exist.
3. When the template has configurable attributes, a variant is known, and not every attribute of the
   template is a no-variant attribute, the rows are attached to the variant.
4. Otherwise the rows are attached to the template.

### 11.3 Clearing media

Requires the Restricted Editor group. When a variant is given and it has its own extra media, the variant
media are deleted; otherwise the template media are deleted.

### 11.4 Reordering media

The move directions are first, left, right and last; the new index is 0 for first, the current index minus
one but not below zero for left, the current index plus one but not beyond the last for right, and the last
index for last. A move that would not change the index returns immediately.

After the move, when the main picture — the template or variant record itself, not a media row — is no
longer first, the entry that is now first is swapped with it: the two records exchange positions, their
picture payloads are exchanged, and the caption of the former main entry is copied onto the media row,
while the product name is unchanged. Every media row is then renumbered with its index in the new list. A
video may never become the main picture.

These two rules are what make the operation non-trivial: a rebuild that only renumbers ordering values will
produce a different result. The messages are `Product not found` when neither a variant nor a template can
be resolved, `Invalid image` when the record to move is not part of the product media list, and
`You can't use a video as the product's main image.` when a video would land in the first position.

---

## 12. Structured product description for search engines

For a template with exactly one variant, the variant description is produced. Otherwise a product-group
description is produced, carrying the vocabulary address, the type name `ProductGroup`, the template name,
the absolute address of the 1920-pixel template picture, the absolute address of the product page, the
description of each variant, and the plain text of the storefront description when it is set.

The number of variants described may be capped by the parameter `website_sale.markup_data_limit_variants`,
which exists to bound the cost for templates with many variants and many price list rules.

For one variant the description carries: the vocabulary address; the type name `Product`; the variant
display name without the internal reference; the absolute address of the variant page; the absolute address
of the 1920-pixel variant picture; an offer holding the taxed price for quantity one in the display
currency, the display currency code and, with the stock capability and for a stored storable variant, the
availability property; the storefront meta description or, failing that, the sales description, when either
is set; the aggregate rating with its average and its count, when the review page option is active and the
variant has ratings; the internal reference when set; and the barcode when set.

The property names `url`, `sku` and `gtin` are defined by the external structured-data vocabulary, not by
this specification, so they are emitted with exactly that spelling: the first carries the web address of the
page, the second carries the internal reference — the three letters abbreviate stock keeping unit — and the
third carries the barcode — the four letters abbreviate global trade item number.

The breadcrumb description appended on a category-qualified product page is a three-element list: `All
Products` pointing at the shop path, the category name pointing at the category path, and the product name
with no address.

---

## 13. Price list selection on the storefront

### 13.1 Which price lists are offered

The procedure, its five memoisation inputs and its acceptance rule are
[calculations.md](calculations.md) §10.1. The contact's assigned price list is passed into the memoised
computation as an argument rather than read inside it, precisely because the computation is memoised:
reading it inside would return stale results when the assignment changes.

Whether one price list is available is the same computation with the visible-only flag cleared, testing
membership of the result.

### 13.2 Which price list is in force

The procedure is [calculations.md](calculations.md) §10.2, with the freshness rule of §10.3. On sign-in, the
price list, the selected price list and the fiscal position session keys are cleared before the redirect, so
that the new user's assignment is applied.

### 13.3 Selecting a price list explicitly

Two entry points exist.

**The selector** applies the price list when it is available on the site and when it is either selectable or
the contact's own assigned price list. On success, and when the price filter is active and the price list
actually changed, the price bounds in the referring address are converted to the new currency. The shopper
is redirected back to the referring page, or to the shop path when there is none.

**The promotional code form** looks the code up against the price list codes, with elevated privileges, and
exactly; when none is found or it is not available, the shopper is redirected to the return path carrying
the code-not-available marker. With an empty code the price list is reset. The default return path is the
cart page. With the promotion capability installed, the same form first tries the code as a loyalty or
coupon code and only falls through to the price list code when the loyalty attempt reports "not found".

Applying a price list:

1. When no price list is supplied, the session price list and the selected price list are cleared, the
   price list is resolved again lazily and, when a cart exists, its price list is recomputed and, when it
   changed, its prices are recomputed.
2. When the supplied price list is already the request price list, nothing happens.
3. Otherwise the supplied price list is stored as both the session price list and the selected price list,
   and, when a cart exists, it is written on the cart and the cart prices are recomputed.

The selected price list is remembered separately from the current price list because an address change
re-resolves the current price list but must not silently discard an explicit choice: after an address
update, the selected price list is re-applied when it is still available on the site and in the new country,
and forgotten otherwise.

### 13.4 Worked example

A visitor whose network address resolves to Belgium browses a site whose price lists are `Public`, generic,
selectable, no country group; `Euro Zone`, generic, not selectable, country group Europe containing
Belgium; and `United States`, generic, selectable, country group United States. The visitor is not signed
in.

1. The available price lists, with the visible-only flag cleared: the country branch matches `Euro Zone`, so
   the result is that one price list.
2. The price list in force: the session is empty, there is no cart, the public contact's assigned price list
   is `Public`, which is not in the available set, therefore the first available price list is used,
   `Euro Zone`.
3. The selector, which asks for visible price lists only, returns nothing from the country branch, because
   `Euro Zone` is not selectable, and the fallback branch keeps only price lists without a country group
   that are selectable, that is `Public`.

This is why mixing selectable price lists and country groups is discouraged: the visitor sees a selector
containing only `Public`, and choosing it replaces the country pricing.

---

## 14. Tax display and fiscal position

Every storefront price passes through [calculations.md](calculations.md) §8, which first restates the stored
price to a tax-excluded base when the product taxes are price-included and the mapped taxes differ, then
computes the mapped taxes for one unit and returns either side according to the site's tax display mode.

The fiscal position in force is resolved once per session:

1. When the session holds a fiscal position identifier that still exists, it is the answer.
2. When the visitor's network country is known and the current user is the site's public user, a temporary
   contact carrying only that country is built and the fiscal position automatically detected for it is
   taken.
3. When nothing resulted, the fiscal position automatically detected for the real contact is taken.
4. The answer is stored in the session.

The fiscal position is resolved again and written back into the session whenever an address update changes
it, and the cart taxes are then recomputed.

---

## 15. Cross-selling and upselling links

| Link | Direction | Where it is shown | Filter applied |
|---|---|---|---|
| Alternative products | template to template | Bottom of the product page | The site sellable-product condition. |
| Accessory products | template to variant | The cart, as suggested accessories | The site sellable-product condition; for users who are not internal, additionally published. |
| Optional products | template to template | The configurator dialogue after an add to cart | Must be showable, addable to the cart under the parent combination and matching the site condition. |

The accessory suggestions of a cart are computed as follows.

1. Start with an empty suggestion list.
2. For each displayed cart line that carries a product, take the accessory variants of its template; when
   there are none, continue with the next line.
3. Build the combination of the line: the variant's attribute values plus its no-variant attribute values.
4. Keep a candidate when it is not already a product of the cart, when it may be quick-added, when it
   belongs to the line's company or to no company, when it is a possible variant under that combination,
   and, when the site forbids zero-price sales, when its contextual price is not zero.
5. Append the kept candidates to the suggestion list.
6. Return the list in a random order, which rotates the suggestions between page loads.

---

## 16. The product syndication feed

### 16.1 Endpoint

The feed path is public and takes the feed identifier and the access token. The path is a fixed literal with
no variable part: a slash, the three lower-case letters that abbreviate the name of the Google Merchant
Center product listing service, a dot, and the three lower-case letters that abbreviate extensible markup
language. The path and the two query parameter names are external wire literals: the external service
stores the whole address and replays it unchanged, so a rebuild must answer that exact path with those
exact parameter names. It answers:

| Condition | Response |
|---|---|
| The site has the feed feature disabled | not found |
| The feed identifier is not a number | bad request |
| No feed carries that identifier | not found |
| The access token does not match, compared in constant time | forbidden |
| The feed belongs to another site | bad request, with `Website does not match.` |
| Otherwise | the compressed document, with the content type declaring the extensible markup language format and the character set, and the content encoding declaring compression |

### 16.2 Document structure

The document carries a channel header and one item per product: the title is the site home page meta title
or, failing that, the site name; the link is the absolute, language-qualified address of the site home page;
the description is the site home page meta description or, failing that, the site name.

### 16.3 Item fields

One item is produced per variant returned by the feed product selection, in the feed language, and only
when the variant is a possible combination and its price payload is not empty. Every field name in the
table below is an external wire literal defined by the syndication service's product data specification,
not by this specification, and must be emitted with exactly that spelling. The shortened ones read in full
words as follows: `id` is the item identifier; `gtin` is the global trade item number carried by the
barcode; `identifier_exists` states whether such a number exists; `item_group_id` is the identifier of the
group that holds the variants of one product together; and `image_link` and `additional_image_link` carry
web addresses of pictures.

| Field | Value |
|---|---|
| `id` | The internal reference when set, otherwise the variant identifier. |
| `title` | The variant display name without the internal reference. |
| `description` | The storefront meta description, otherwise the sales description. |
| `link` | The absolute, language-qualified product address; when the feed names a price list, the price list query parameter is appended, which makes the landing page show the same price as the feed. |
| `gtin`, `identifier_exists` | The barcode when set, with the existence flag set to `yes`; when there is no barcode, only the existence flag is emitted, with the value `no`. |
| `image_link` | The absolute address of the 1920-pixel variant picture, or an empty value when the variant has no picture. A placeholder is never published. |
| `additional_image_link` | The absolute addresses of at most ten extra pictures, variant media first and then template media, videos excluded. |
| `price` | The reference price of the combination payload, formatted as the amount rounded to the currency, a space, and the currency code. |
| `sale_price` | Present only when the payload reports a discount: the discounted price, in the same format. |
| `sale_price_effective_date` | Present only when the applied price list rule has both a start and an end moment: the two moments in coordinated universal time, at minute precision, joined by a slash. |
| `unit_pricing_measure`, `unit_pricing_base_measure` | Present only under the conditions and with the computation of [calculations.md](calculations.md) §19. |
| `product_detail` | One pair of attribute name and value name per attribute value of the variant. |
| `is_bundle` | `yes` when the product is a combo, otherwise `no`. |
| `product_type` | At most five category display names, ordered by category ordering value, with the path separator replaced by a greater-than sign. |
| `custom_label` | At most five pairs of an indexed label name and a tag name, built from the variant tags ordered by their ordering value. |
| `item_group_id` | The template identifier, present only when the template has more than one variant. |
| `availability` | `in_stock`, or `out_of_stock` when the stock capability is installed and the variant is sold out. |

### 16.4 Product selection and limits

```formula
feed products = the variants matching
        the publication flag is true
    AND the product kind is goods or combo
    AND the site condition
    AND the storefront categories are the feed categories or their descendants,
        when categories are selected
limited to 6000 rows
```

A soft limit of 5000 is enforced as a validation on the feed record (WS-437). When a rendering returns more
than 5500 products and no warning was sent in the last week, the site salesperson is notified (WS-439).

### 16.5 Caching

The document is rendered at most once per day per feed. Rendering takes an exclusive lock on the feed row,
so two concurrent fetches do not render twice. Changing the site, the price list, the language or the
categories of the feed invalidates the cache immediately; an administrator may also reset it by hand.

### 16.6 Price localisation

When the feed names a price list, the request price list is replaced by it for the duration of the
rendering, and every product address receives the price list query parameter. This guarantees that the
price advertised in the feed and the price shown on the landing page are the same, including the currency.
