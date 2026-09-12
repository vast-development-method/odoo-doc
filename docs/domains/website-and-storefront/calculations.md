# Calculations

Every formula and every algorithm of this folder, with its inputs, its outputs, its evaluation order,
its rounding rule and at least one worked numeric example carried to the last decimal the rule
produces. Formulas are written over quantities named in words. Algorithms are written as numbered
procedures.

Tax computation itself is owned by [taxes](../taxes/calculations.md) and price computation by
[pricing and price lists](../pricing-and-pricelists/calculations.md); this file specifies how the
storefront calls them and what it does with the results.

---

## 1. Precision, rounding and comparison conventions

| Quantity | Precision | Rounding |
|---|---|---|
| Displayed prices, subtotals and totals | The number of decimal places of the display currency, two for most currencies | Half away from zero, applied by the tax computation |
| Comparison of two amounts | The display currency precision | The comparison returns 1 when the first is greater, −1 when it is smaller and 0 when the two are equal at that precision |
| Currency conversion inside a price payload | Not rounded | The conversion is requested with rounding disabled, so that a later tax computation rounds once |
| Storefront quantities free to use | Whole units | Rounded **down** |
| Shopper-entered quantities | Whole units | Truncated towards zero |
| Line quantities displayed | Product unit precision, two decimal places by default | Printed as a whole number when the value is integral |
| Base unit count | Unlimited decimal places | Stored exactly, so that a ratio such as one unit in a box of 10000 is representable |
| Distance between two addresses | Kilometres | Not rounded; used only for ordering |
| Reference measure in the product feed | Two decimal places | Half away from zero |
| Abandoned-cart delay | Hours, decimal | Compared without rounding |
| Site index priority | One decimal place | Half away from zero, then capped at 1 |
| Reputation score | Whole points | No rounding: every award is an integer |
| Relevance of a question | Not rounded | Used only for ordering |

The zero test used by the storefront is always "zero at the precision of the display currency", never an
exact equality with zero, except for the zero-priced line rule of the cart (WS-355 in
[business-rules.md](business-rules.md)), which compares the stored unit price with exactly 0.

---

## 2. Site resolution from the request host

**Inputs.** The request host as sent by the browser, including the port when one is present; the
configured domain of every site; the ordering value and identifier of every site.

**Output.** Exactly one site, or none.

1. When the session carries a forced site identifier and that site exists, that site is the answer.
2. Otherwise, when the execution context carries a site identifier, that site is the answer.
3. Otherwise, when the request is not a front-end request and no fallback was requested, the answer is
   "no site".
4. Otherwise the host is normalised: it is lower-cased and its scheme, if any, is removed. Two spellings
   are produced: the host converted to the ascii-compatible internationalized domain name encoding, and
   the host converted back to its readable form.
5. A first pass selects the candidate sites whose stored domain **contains** either spelling. This pass
   exists only to narrow the set cheaply.
6. The candidates are filtered to those whose stored domain, reduced to its host and port and
   lower-cased, is **equal** to one of the two spellings. The first candidate in ordering order wins.
7. When step 6 produced nothing, steps 5 and 6 are repeated after removing the port from both sides.
8. When step 7 produced nothing and a fallback was requested, the first site in ordering order wins;
   otherwise the answer is "no site".

The result of steps 4 to 8 is cached per host and per fallback flag.

**Worked example.** Three sites exist: site 1, ordering value 10, domain `https://shop.example.com`;
site 2, ordering value 20, domain `https://example.com:8069`; site 3, ordering value 30, no domain.

| Request host | Step that decides | Site |
|---|---|---|
| `shop.example.com` | 6, exact match on the host | 1 |
| `SHOP.EXAMPLE.COM` | 6, after lower-casing | 1 |
| `www.shop.example.com` | 6 fails (no exact equality), 7 fails, 8 with a fallback | 3 |
| `example.com:8069` | 6, exact match on host and port | 2 |
| `example.com` | 6 fails, 7 succeeds after removing the port from `example.com:8069` | 2 |
| `example.com:8070` | 6 fails, 7 succeeds, because both sides lose their port | 2 |
| `other.test` | 6 and 7 fail, 8 with a fallback | 3 |
| `other.test`, back-office request, no fallback | 3 | none |

The subdomain case is the one a rebuild most often gets wrong: `www.shop.example.com` does **not** match
`shop.example.com`, because the comparison is an equality and not a suffix test.

---

## 3. Slugs, unique addresses and unique template keys

### 3.1 The slug of a record

```formula
record slug = human part + "-" + record identifier
human part  = the slug name of the record when it is set, otherwise the display name of the record,
              reduced to a slug
```

Reducing a text to a slug: accents are removed, every character that is not a letter, a digit or a
hyphen becomes a hyphen, runs of hyphens collapse to one, leading and trailing hyphens are removed, and
the result is lower-cased. A human part that becomes empty is dropped together with its hyphen, so the
slug is then the bare identifier.

Parsing a slug back into an identifier reads the digits after the last hyphen; a segment made only of
digits is read as the identifier itself.

**Worked example.** A blog post named "Réunion d'équipe 2024" with the identifier 7 has the slug
`reunion-d-equipe-2024-7`. A post named "!!!" with the identifier 9 has the slug `9`.

### 3.2 The path slug of a page address

The path slug applies the same reduction to each path segment separately, keeping the slashes, with a
maximum of 1024 characters per segment. The result is always prefixed with one slash.

### 3.3 Uniqueness suffix

```formula
candidate 0 = the computed address
candidate n = the computed address + "-" + n          for n = 1, 2, 3, …
chosen      = the first candidate that no page of the same site uses, archived pages included
```

**Worked example.** A site already has `/contact` and `/contact-1`. Creating a third page named
"Contact" produces `/contact-2`.

### 3.4 Unique template key

```formula
base key    = the package name (or the supplied namespace) + "." + the slug of the page name
candidate 0 = base key
candidate n = base key + "-" + n                      for n = 1, 2, 3, …
chosen      = the first candidate that no template of the current site and no shared template uses,
              archived templates included
```

---

## 4. Approximate matching and the closest term

Used by the site search and by the shop listing when the exact term returns nothing.

### 4.1 When approximate matching is skipped

```formula
skip = ( number of characters of the term < 4 )
    OR ( the term contains a space )
    OR ( number of digits in the term ÷ number of characters of the term ≥ 0.8 )
```

When the search is skipped, the term is searched as it stands.

### 4.2 Bounded edit distance

The distance between two words is the number of single-character insertions, deletions and
substitutions needed to turn one into the other, computed with a band of at most 4 around the diagonal.
When the two lengths differ by more than 4, or when the computed distance exceeds 4, the distance is
reported as "beyond the limit" and the pair is treated as not similar.

### 4.3 Similarity score

```formula
common letters      = the number of distinct letters of the term that also occur in the candidate word
distinct letters    = the number of distinct letters of the term
symmetric letters   = the number of letters that occur in exactly one of the two words, counted once each
similarity score    = common letters ÷ distinct letters
                    − edit distance ÷ number of characters of the term
                    − symmetric letters ÷ ( number of characters of the term
                                            + number of characters of the candidate word )
```

A pair whose distance is beyond the limit scores −1 and is never chosen.

### 4.4 Choosing the closest term

1. The candidate words are the words occurring in the matched fields of the participating entities,
   limited to 1000 records per entity. When the storage engine offers similarity indexes, the candidate
   list is narrowed by an index lookup first.
2. A candidate that **contains** the term as a substring stops the search at once: the original term is
   used.
3. Only candidates whose first character equals the first character of the term are scored; each distinct
   candidate is scored once.
4. The candidate with the highest score wins. When it equals the term ignoring case, no substitution is
   reported to the page.

**Worked example.** The term is `chiar`, five characters, no space, no digits, so approximate matching
runs. Two candidates start with the same letter: `chair` and `chart`.

```formula
chair: edit distance = 2 ; distinct letters of the term = 5 ; common letters = 5 ;
       symmetric letters = 0
       score = 5 ÷ 5 − 2 ÷ 5 − 0 ÷ (5 + 5) = 1.0 − 0.4 − 0.0 = 0.6

chart: edit distance = 2 ; distinct letters of the term = 5 ; common letters = 4 ;
       symmetric letters = 2   (the letter i and the letter t)
       score = 4 ÷ 5 − 2 ÷ 5 − 2 ÷ (5 + 5) = 0.8 − 0.4 − 0.2 = 0.2
```

`chair` scores 0.6 against 0.2, so the search is re-run with `chair` and the page reports "showing
results for chair, search instead for chiar".

---

## 5. Pagination and the pager window

```formula
number of pages = ceiling( total number of records ÷ page size )
offset of page p = ( p − 1 ) × page size
first page of the window = maximum of 1 and ( current page − 2 )
last page of the window  = minimum of ( number of pages ) and ( first page of the window + 4 )
first page of the window = maximum of 1 and ( last page of the window − 4 )
```

The window therefore always shows at most five page links, and slides so that the current page is in the
middle wherever possible.

**Worked example.** 137 products, a page size of 21, the visitor is on page 4.
`ceiling(137 ÷ 21) = ceiling(6.5238) = 7` pages; the offset of page 4 is `(4 − 1) × 21 = 63`; the window
runs from `max(1, 2) = 2` to `min(7, 6) = 6`, then the first page is recomputed as `max(1, 2) = 2`, so
the links are 2, 3, 4, 5, 6.

The shipped page sizes are: 21 products on the shop listing, 12 posts on a blog listing, 20 contacts in
the reference directory, 50 results on the public search listing, and 45000 locations per site index
document.

---

## 6. Teaser extraction, truncation and highlighting

### 6.1 Teaser of a blog post

```formula
teaser = the manual teaser                                      when it is not empty
teaser = the first 200 characters of the plain text of the content, with runs of whitespace collapsed
         to one space, + "..."                                  otherwise
```

**Worked example.** A post whose content renders as 640 characters of plain text and that has no manual
teaser produces a teaser of exactly 203 characters: the first 200 characters, then the three dots. A post
whose plain text is 80 characters long still receives the three dots, because the rule appends them
unconditionally.

### 6.2 Truncation of a search result value

```formula
shown value = the value                                         when its length ≤ the maximum
shown value = the first ( maximum ) characters + "..."          otherwise
```

The maximum is 999 characters for the autocompletion answer and 200 characters for the public search
listing. A mapping may disable truncation for a value, in which case the whole value is returned.

### 6.3 Highlighting

Every whitespace-separated part of the search term is located in the value, case-insensitively, and each
occurrence is wrapped in the highlight template. The value is then treated as markup rather than as text.

---

## 7. Picture sizing, cache markers and site index priority

### 7.1 Derived picture sizes

A picture field stores the full resolution and derives four smaller renditions by downscaling to fit
inside a box of 1024, 512, 256 and 128 pixels on the longest side, preserving the aspect ratio.

```formula
chosen product tile picture = the 512-pixel rendition   when the tile is one cell wide and the grid has
                                                        at least three columns
chosen product tile picture = the 1024-pixel rendition  otherwise
```

### 7.2 Cache marker of a picture address

```formula
cache marker = the first eight characters of the checksum of the stored bytes
picture address = "/web/image/" + identifier + "-" + cache marker + "/" + the percent-encoded file name
```

When the attachment carries its own address, the marker is appended as a query parameter instead, using a
question mark when the address carries no query string yet and an ampersand otherwise.

### 7.3 Site index priority

```formula
contributed priority = minimum of 1 and round( template priority ÷ 32, 1 decimal place )
```

No priority is contributed when the template priority equals the neutral value 16.

**Worked example.** A template with the priority 40 contributes `round(40 ÷ 32, 1) = round(1.25, 1) = 1.3`,
capped at `1`. A template with the priority 24 contributes `round(0.75, 1) = 0.8`. A template with the
priority 16 contributes nothing.

---

## 8. Applying taxes to a storefront price

**Inputs.** A price in the display currency; the display currency; the product's own taxes filtered to
the current company; the same taxes mapped through the fiscal position; the product or template; the
site.

**Output.** The price to display, in the site's tax display mode.

1. Restate the price when the product taxes are price-included and the mapped taxes differ from them.
   Without this step a price stored tax-included under one tax would be reinterpreted under another tax.
2. Compute the mapped taxes for exactly one unit, with the current user's contact as the counterparty.
3. Return the total excluding tax when the site's tax display mode is tax excluded, and the total
   including tax otherwise.

Quantity one is used deliberately: the value returned is a **unit** price, and multiplying a rounded unit
price by a quantity is what the cart lines do afterwards.

**Worked example A — tax defined as excluded, display tax included.** Price 94.50, one tax of 21 percent
not included in the price, no fiscal position mapping.

```formula
step 1: the product taxes are not price-included, therefore the price is unchanged = 94.50
step 2: total excluding tax = 94.50 ; tax = 94.50 × 0.21 = 19.845 → rounded 19.85 ;
        total including tax = 94.50 + 19.85 = 114.35
step 3: display tax included → 114.35
```

**Worked example B — tax defined as included, display tax excluded.** Price 121.00 stored tax-included,
one tax of 21 percent price-included.

```formula
step 1: unchanged, because the mapped taxes equal the product taxes
step 2: total excluding tax = 121.00 ÷ 1.21 = 100.00 ; total including tax = 121.00
step 3: display tax excluded → 100.00
```

**Worked example C — mapping 21 percent price-included to 0 percent.** Price 121.00 stored tax-included.

```formula
step 1: the original tax is price-included and the new tax differs, therefore the price is restated
        onto the new basis: 121.00 ÷ 1.21 = 100.00
step 2: the new tax is 0 percent → total excluding tax = total including tax = 100.00
step 3: either display mode → 100.00
```

**Worked example D — mapping 15 percent price-included to 5 percent price-included.** Price 500.00 stored
tax-included at 15 percent.

```formula
step 1: 500.00 ÷ 1.15 = 434.782608…
step 2: total including tax = 434.782608… × 1.05 = 456.521739… → rounded 456.52
        total excluding tax = 434.782608… → rounded 434.78
step 3: display tax included → 456.52 ; display tax excluded → 434.78
```

---

## 9. The price of a combination

**Inputs.** A template, an optional variant, a combination, a quantity, a unit of measure, the request
price list, the site currency, the request fiscal position and today's date.

**Outputs.** The price, the reference price, the discount flag, the extra-price flag, optionally the
comparison price, and the tax context used by the extra-price computation.

1. Ask the price list for the price of the product or template, for the requested quantity — 1.0 when
   none was given — in the requested unit — the product unit when none was given — expressed in the site
   currency. The applied rule is returned with the price.
2. The price before discount equals the price list price, unless the applied rule shows a discount on the
   shop (§9.1), in which case the rule's own price before discount is requested for the same quantity,
   date, unit and currency.
3. The discount flag is true when the comparison of the price before discount with the price list price
   returns 1, that is when the former is strictly greater at currency precision.
4. The reference price is the greater of the price list price and the price before discount; the price is
   the price list price.
5. The extra-price flag is true when the applied rule is not a fixed-price rule.
6. When the product carries taxes for the current company, the taxes are mapped through the fiscal
   position and both the price and the reference price are passed through §8.
7. When no discount applies, the product carries a comparison price and the comparison-price feature is
   enabled, the comparison price is converted from the product currency to the site currency at today's
   rate, with rounding disabled, and is **not** taxed.
8. When the site forbids zero-priced sales and the price is zero at currency precision, the zero-price
   flag is set and the comparison price is forced to zero.

### 9.1 When a rule shows a discount on the shop

```formula
shows a discount on the shop =
      a rule was applied
  AND ( the rule computes a percentage
        OR ( the rule computes a formula
             AND its discount is not zero
             AND its base is the sales price or another price list ) )
```

This is deliberately broader than the rule used on the order: on the catalogue and in the configurator a
formula rule also shows a discount; on the cart and at checkout it does not.

### 9.2 Worked example: attribute extra, price list discount, tax included

**Setup.** A template whose sales price is 100.00 in the company currency, which is also the site
currency. One attribute value carries an extra price of 5.00. One sales tax of 21 percent, defined as
**not** included in the price. The site displays prices tax included. The applicable price list has one
rule: a percentage rule of 10 percent applied to the sales price. The shopper selects the attribute
value, quantity 1.

```formula
variant sales price             = 100.00 + 5.00               = 105.00
price list price (10 % off)     = 105.00 × (1 − 0.10)         =  94.50
shows a discount on the shop    = true, the rule is a percentage rule
price before discount           = 105.00
discount flag                   = compare(105.00, 94.50) = 1  → true
reference price before tax      = maximum of 94.50 and 105.00 = 105.00
extra-price flag                = true, the rule is not a fixed-price rule

taxes, display mode tax included:
price           = 94.50 × 1.21 = 114.345   → rounded 114.35
reference price = 105.00 × 1.21 = 127.05   → 127.05
attribute extra = 5.00 × 1.21 = 6.05
```

The product page shows **114.35**, with **127.05** struck through, and the attribute value labelled
**+6.05**.

The same setup on a site displaying tax-excluded prices shows **94.50**, struck through **105.00**, and
the extra **+5.00**.

The same setup with a fixed-price rule at 94.50 shows the same price, but the extra-price flag is false,
so the attribute value shows no extra at all, and the discount flag is false, because a fixed rule does
not show a discount on the shop, so there is no strikethrough.

If the product also declares a base unit count of 3 with the reference unit `100 g`, and the per-unit
price feature is enabled, the page additionally prints `38.12 / 100 g`, computed as
`114.345 ÷ 3 = 38.115`, rounded for display to 38.12. When the shopper switches the page to a unit of
measure holding 6 product units, the price is first converted to the product unit and then divided by the
base unit count.

---

## 10. Price list resolution, memoisation and freshness

### 10.1 Which price lists are offered

1. When the price list feature is disabled, the answer is empty.
2. The visitor's country code is resolved from the network address, or is unknown.
3. The signed-in contact's assigned price list is read with that country code in context; for the public
   user it is empty.
4. The candidate computation is then performed on exactly five inputs: the country code, the
   visible-only flag, the identifier of the price list currently in force, the tuple of the site's price
   list identifiers and the identifier of the contact's assigned price list. It returns identifiers, never
   records, so that the memoised value stays valid across requests.
5. Inside that computation, a price list is accepted when the visible-only flag is false, or when it is
   selectable, or when it is the one currently in force.
6. When a country code is known, the result is every price list of every country group containing that
   country, kept when it is available on the site (WS-324) and accepted by step 5.
7. When that result is empty, the result is the site's price lists, kept when they are accepted and, when
   a country code is known, when they carry no country group.
8. For a signed-in visitor, the contact's assigned price list is appended when it is available on the
   site, accepted, and available in the visitor's country.
9. The result is sorted by the price list default order and returned as identifiers.

Any create, write or delete on any price list clears every memoised entry.

### 10.2 Which price list is in force

1. When the session holds a price list identifier and that price list still exists, is available on the
   site and is available in the visitor's country, it is the answer.
2. Otherwise, when a cart exists, the cart's price list is recomputed — unless the request is read-only —
   and the cart's price list is the answer.
3. Otherwise the contact's assigned price list is taken; when the available list is not empty and the
   assigned one is not in it, the first available one is taken.
4. The answer is stored in the session together with the resolution moment.

### 10.3 Freshness

```formula
drop the cached price list when   the resolution moment < the present moment − 3600 seconds
```

The test runs on every shop listing request.

### 10.4 Worked example

A site offers two selectable price lists, `Public` with the identifier 1, the site default, in euro, and
`Retail Belgium` with the identifier 7, also in euro, and one price list that is not selectable,
`Wholesale`, with the identifier 9. A signed-in shopper whose contact carries no assigned price list opens
the shop at 10:00:00 from a Belgian network address.

1. The memoisation key is the country code `BE`, the visible-only flag true, no current price list, the
   site price lists 1, 7 and 9, and no assigned price list. `BE` is the two-letter country code of
   Belgium, used here only as a data value.
2. The candidate set keeps 1 and 7, because `Wholesale` is not selectable and no session price list forces
   it in, and returns the identifiers 1 and 7, not the records.
3. The session has no price list, so the site default 1 is resolved and stored, together with the
   resolution moment 10:00:00.
4. At 10:25:00 the shopper opens another page. `10:00:00` is not older than
   `10:25:00 − 3600 seconds = 09:25:00`, so the cached price list 1 is reused and no resolution runs.
5. At 11:05:00 the shopper opens a third page. `10:00:00 < 11:05:00 − 3600 seconds = 10:05:00`, so the
   cached price list is dropped, the resolution runs again and stores 11:05:00.
6. At 11:10:00 the shopper picks `Retail Belgium` in the selector. The session now holds 7 both as the
   price list in force and as the explicitly selected one, and the memoisation key of the next request
   carries 7 as the current price list, which is a different key and therefore a separate entry.
7. A configuration change to any price list clears every memoised entry, so the next request recomputes
   the candidate list from storage even inside the one-hour window.

---

## 11. The catalogue price payload

For each product template of the displayed slice, one payload is produced.

```formula
price shown       = §8 applied to the price list price for quantity 1
reference price   = §8 applied to the price before discount   when a discount rule applies (§9.1)
                                                              and the base is strictly greater
reference price   = the comparison price, converted to the display currency without rounding,
                                                              when no discount applies and the
                                                              comparison-price feature is enabled
reference price   = absent                                    otherwise
```

**Worked example.** A product stored at 61.98 with one tax of 21 percent not included; the site displays
tax included; the comparison feature is enabled; the price list applies 20 percent.

```formula
price list price      = 61.98 × 0.80 = 49.584
price before discount = 61.98
price shown           = 49.584 × 1.21 = 59.99664  → rounded 60.00
reference price       = 61.98  × 1.21 = 74.9958   → rounded 75.00
```

The shop tile shows **60.00** with **75.00** struck through.

---

## 12. Base unit price and attribute extra price

### 12.1 Base unit price

```formula
price per reference unit = 0                            when the base unit count = 0
price per reference unit = price ÷ base unit count      otherwise
```

On a page, the price fed into the formula is first converted from the selected unit of measure to the
product unit. In the cart, the price fed into the formula is the line's own price divided by the line
quantity.

**Worked example.** A two-litre bucket of paint sells at 60.00 with a base unit count of 2 and the
reference unit `L`. The page prints `30.00 / L`. The same product with a base unit count of 0 prints
nothing.

### 12.2 Attribute extra price

```formula
extra shown = 0                        when the value carries no extra price
extra shown = 0                        when the payload reports that extra prices are hidden
extra shown = the extra price, converted from the template currency to the payload currency at the
              payload date, then passed through the tax treatment of §8
```

**Worked example.** An extra of 5.00 in the company currency, a display currency at a rate of 2, a tax of
21 percent not included, display tax included: `5.00 × 2 = 10.00`, then `10.00 × 1.21 = 12.10`.

---

## 13. Price filter bounds and grid packing

### 13.1 Price filter bounds

```formula
rate = the conversion rate from the company currency to the site currency, today
condition = sales price ≥ minimum price ÷ rate AND sales price ≤ maximum price ÷ rate
available minimum = ( the smallest sales price over the listing condition, or 0 ) × rate
available maximum = ( the largest sales price over the listing condition, or 0 ) × rate
```

The two aggregates are obtained with one grouped query over the listing condition, not by searching for
the cheapest and the most expensive product.

Clamping, applied once the aggregates are known:

```formula
when a minimum was given and minimum > available maximum:  minimum = available minimum
when a maximum was given and maximum < available minimum:  maximum = available maximum
```

When the shopper switches to a price list in another currency, the bounds present in the previous address
are converted from the previous price list currency to the new one, without rounding.

**Worked example.** The company currency and the site currency are the same, so the rate is 1. The shopper
filters 50 to 200 in a category whose prices run from 10 to 80. The minimum 50 is at most 80, so it is
kept; the maximum 200 is at least 10, so it is kept; the result contains the products priced between 50
and 80. The shopper then switches to a narrower category whose prices run from 300 to 900 while keeping
the address: the minimum 50 is at most 900 and is kept, but the maximum 200 is below 300, so it is
replaced by 900, and the shopper sees the whole category instead of an empty page.

### 13.2 Grid packing

A product tile may span several grid cells: a width and a height in cells, each clamped between 1 and the
number of columns. The packer fills a sparse table of cells.

1. Set the minimum position to 0, the index to 0 and the highest row to 0.
2. For each product in listing order:
   1. The width and the height are the product's tile width and tile height, each clamped between 1 and
      the number of columns. When the index has reached the page size, both are forced to 1.
   2. Start at the minimum position. A position is expressed as a running cell number: its column is the
      position modulo the number of columns and its row is the position divided by the number of columns,
      discarding the remainder.
   3. A placement fits when, for every cell of the block, the column stays inside the grid and the cell is
      free. Rows are created empty as they are needed.
   4. While the placement does not fit, increase the position by one.
   5. When the index has reached the page size and the row after the placed block is beyond the highest
      row, stop packing: the page is complete.
   6. When the tile is one cell by one cell, the minimum position becomes the row of the placement.
   7. Mark every cell of the block as taken and write the product, its width, its height and its ribbon in
      the top-left cell.
   8. While the index is at most the page size, the highest row becomes the greater of itself and the row
      of the placement plus the height.
   9. Increase the index by one.
3. The rows are the table rows in ascending row order, each row being its non-empty cells in ascending
   column order.

The stop rule means that, once the page quota is reached, tiles continue to be placed only while they fill
holes in rows that are already started; the first tile that would open a new row ends the page. This keeps
large tiles from producing ragged pages.

**Worked example.** Four columns, a page size of 6, and the products, in order, have the sizes: A two by
two, B one by one, C one by one, D one by one, E one by one, F two by one, G one by one.

```formula
A two by two at position 0 → occupies the cells (0,0), (1,0), (0,1) and (1,1) ;
                             minimum position stays 0 ; highest row = 2
B one by one : positions 0 and 1 are taken, position 2 is free → cell (2,0) ; minimum position = 0
C one by one : cell (3,0)
D one by one : positions 4 and 5 are the cells (0,1) and (1,1), both taken → cell (2,1)
E one by one : cell (3,1)
F two by one : the first free pair on one row is (0,2) and (1,2) → row 2
G one by one : the index 6 has reached the page size 6, so its size is forced to one by one ;
               it fits at (2,2), and the row after it, 0.75, is not greater than the highest row 2,
               therefore it is kept
```

The page ends with three rows: A, B, C; then D, E; then F, G. A further one-by-one product would land at
(3,2) and still be kept; the next one would open row 3 and end the page.

---

## 14. Cart aggregates and the availability cap

### 14.1 Cart aggregates

```formula
cart quantity = truncate to a whole number of ( the sum of the demanded quantities of the displayed lines )
                − the quantities of reward lines, when promotions are installed
services only = every displayed line carries a service product ( true for an empty cart )
delivery amount = the sum of the subtotals of the delivery lines   when the tax display is tax excluded
delivery amount = the sum of the totals of the delivery lines      when the tax display is tax included
total excluding delivery = the order total − the sum of the totals of the delivery lines
lines counted for accessories = the lines that are not delivery lines,
                                minus the free-shipping reward lines when promotions are installed
```

**Worked example.** A cart holds two ordinary lines of quantity 2 and 1, a delivery line of quantity 1 and
a discount reward line of quantity 1. The delivery line is not displayed, so it never enters the sum:
`truncate(2 + 1 + 1) − 1 = 3`.

### 14.2 The availability cap

```formula
available     = round down to a whole number of
                ( the quantity free to use, converted from the product unit to the requested unit )
cart before   = the cart quantity of that product, converted from the product unit to the requested unit
added         = requested quantity − the previous quantity of the line ( 0 for a new line )
total in cart = cart before + added
allowed       = available − ( cart before − the previous quantity of the line )
```

When `available` is at least `total in cart`, the requested quantity is accepted with no warning.
Otherwise the line is set to `allowed`, which may be zero or negative, in which case the caller deletes
the line, and one of the four messages of WS-400 is produced.

**Worked example with one unit of measure.** A tracked product with out-of-stock ordering disabled has 3
units free. The cart already holds 2 of them on one line. The shopper types 5 on that line.

```formula
cart before = 2 ; available = 3 ; previous = 2 ; added = 5 − 2 = 3
total in cart = 2 + 3 = 5 > 3
allowed = 3 − ( 2 − 2 ) = 3
warning = "You ask for 5 <product> but only 3 is available"
```

The line is set to 3. If the same shopper had a second line of the same product holding 1 unit, then
`cart before = 3`, `allowed = 3 − (3 − 2) = 2`, and the edited line would be capped at 2, keeping the cart
total at 3.

**Worked example with two units of measure.** A product is stocked in units and also sold by the box of 6.
The quantity free to use is 20 units. The cart already holds one line of 1 box, that is 6 units. The
shopper asks for 4 boxes on that line.

```formula
available   = round down ( 20 ÷ 6 ) = round down ( 3.333… ) = 3 boxes
cart before = 6 units → 1 box
added       = 4 − 1 = 3
total       = 1 + 3 = 4 > 3
allowed     = 3 − ( 1 − 1 ) = 3
warning     = "You ask for 4 <product> but only 3 is available"
```

The line ends at 3 boxes, that is 18 of the 20 available units; the remaining 2 units cannot form a box.

---

## 15. Line display amounts

```formula
displayed unit price = the taxes of the line applied, for quantity one, to
                       ( the combo display price when the product is a combo,
                         otherwise the line unit price ),
                       returned tax excluded or tax included according to the site setting
cart display price   = the sum, over the line and its priced linked lines, of
                       the subtotal ( tax-excluded display ) or the total ( tax-included display )
displayed quantity   = round( the demanded quantity, product unit precision ),
                       printed as a whole number when it is integral
strikethrough shown  = the line discount is not zero
                       AND the line is sellable
                       AND the displayed unit price is not zero
```

**Worked example, tax-excluded display.** The site displays tax-excluded prices. The currency has two
decimal places; the product unit precision is two decimal places. A cart line holds 3 units of a desk lamp
whose unit price is 40.00, with a line discount of 10 percent and one tax of 21 percent. The stored line
amounts are `round(3 × 40.00 × (1 − 10 ÷ 100), 2) = 108.00` for the subtotal and
`round(108.00 × 1.21, 2) = 130.68` for the total.

```formula
displayed unit price : base = 40.00 ( not a combo )
                       for quantity one → total excluding tax 40.00, total including tax 48.40
                       the site displays tax excluded → 40.00
cart display price   : the line has no linked lines → the subtotal = 108.00
displayed quantity   : round(3.0, 2) = 3.0, integral → printed as 3
strikethrough shown  : the discount 10 is not zero, the line is sellable,
                       the displayed unit price 40.00 is not zero → true
```

**Worked example, tax-included display.** The same line on a site that displays tax-included prices. The
displayed unit price becomes `round(40.00 × 1.21, 2) = 48.40` and the cart display price becomes 130.68.
The displayed unit price is computed for quantity one and is therefore **not** `130.68 ÷ 3 = 43.56`: the
discount is not reflected in the struck-through unit price, which is exactly what the strikethrough is
meant to show.

**Worked example, a combo line.** A combo product sold at 90.00 tax excluded is made of two selected items,
a lamp line of 40.00 and a tray line of 50.00, both linked to the combo line. The combo line's own unit
price is 0.00, because the items carry the money. The displayed unit price therefore uses the combo display
price 90.00 rather than the line unit price 0.00, and the cart display price sums the combo line and its
two priced linked lines: `0.00 + 40.00 + 50.00 = 90.00`. The item lines are not displayed as cart lines of
their own; only the combo line is.

**Worked example, a fractional quantity.** A line holds 2.50 metres of cable. `round(2.50, 2) = 2.50`,
which is not integral, so it is printed as `2.50` and not as `2`.

---

## 16. Abandoned-cart threshold

```formula
threshold  = the present moment − ( the site's abandoned-cart delay, or 1 hour when it is zero )
abandoned  = the order has a site
         AND the order state is draft
         AND the order date is set
         AND the order date ≤ threshold
         AND the order contact is not the site's public contact
         AND the order has at least one line
```

### 16.1 Worked example: a ten-hour delay

**Setup.** The site's delay is 10.0 hours. The recovery feature was switched on at 07:00, so the activation
moment is 07:00. A signed-in shopper creates a cart at 08:00 with one line priced 45.00 and leaves.

| Moment | Threshold | Abandoned? | Job outcome |
|---|---|---|---|
| 12:00 | 02:00 | `08:00 ≤ 02:00` is false | Not selected. |
| 17:59 | 07:59 | `08:00 ≤ 07:59` is false | Not selected. |
| 18:01 | 08:01 | `08:00 ≤ 08:01` is true | Selected; the order date 08:00 is at or after the activation moment 07:00; the eligibility filter passes, because the customer has an address, no transaction is in error, the line price is not zero and the customer has placed no confirmed order since. The recovery message is sent and the recovery flag becomes true. |
| 19:00, next run | 09:00 | still true | Not selected, because the cart is already marked as mailed. |

Had the feature been switched on at 09:00 instead, the activation moment would be 09:00, the cart's order
date 08:00 would be earlier, and the cart would never be mailed. Had the cart contained only free products,
the eligibility filter would have rejected it at 18:01, and the cart would have been marked as mailed
anyway, which permanently removes it from the job.

---

## 17. Delivery rate, taxes on the rate and the free-shipping threshold

1. The method computes its own rate: a fixed price, a price computed from rules on weight, volume,
   quantity or price, or a value returned by an external carrier service.
2. The rate is restated as a tax-included unit price for the order's company, currency, date and fiscal
   position.
3. Any configured margin is applied.
4. The result is rounded to the order currency precision, and kept as the carrier price before any
   free-shipping override.
5. The free-shipping override is then applied:

```formula
amount without delivery = the order total − the sum of the totals of the delivery lines
free shipping applies   = the rate succeeded
                      AND the method offers free shipping above a threshold
                      AND the method is not a rule-based method
                      AND convert( amount without delivery, order currency → company currency ) ≥ threshold
when free shipping applies: the rate becomes 0.00 and the warning
   "The shipping is free since the order amount exceeds <threshold with two decimals>." is attached
```

6. For display the rate is passed through the taxes of the delivery product mapped by the order's fiscal
   position, for quantity one, and returned tax excluded when the request is not an express-checkout
   request and the site displays tax-excluded prices, and tax included otherwise.

### 17.1 Worked example: threshold 100, cart 95, delivery 7.50

**Setup.** The order currency and the company currency are the same. The method is a fixed-price method at
7.50 with free shipping above 100.00. The delivery product carries no tax. The site displays tax-included
prices. The cart holds goods for 95.00 including tax.

| Event | Amount without delivery | Threshold test | Rate | Order total |
|---|---|---|---|---|
| The method is selected on a cart of 95.00 | 95.00 | `95.00 ≥ 100.00` false | 7.50 | 95.00 + 7.50 = **102.50** |
| The shopper adds 10.00 of goods | 105.00 | `105.00 ≥ 100.00` true | 0.00, with the warning `The shipping is free since the order amount exceeds 100.00.` | 105.00 + 0.00 = **105.00** |
| The shopper removes the added item | 95.00 | false again | 7.50 | **102.50** |

The delivery line is never deleted by the threshold: it remains at a unit price of zero, which keeps the
free shipping visible on the order and on the invoice. With a free-shipping reward from a promotion instead
of a threshold, the reward line's total is excluded from the amount without delivery, which prevents the
reward from pushing the order over its own threshold.

---

## 18. Kit availability and the distance between a shopper and a store

### 18.1 Kit availability

```formula
for each component of the exploded kit, grouped per component:
    quantity per kit = the summed requirement for one kit, expressed in the component unit
    the component is skipped when it is not tracked or when the quantity per kit is zero
    ratio = round down ( the component's quantity free to use ÷ quantity per kit,
                         component unit precision )
quantity free to use of the kit = floor( round( the smallest ratio × the kit quantity of the
                                                bill of materials ) )
quantity free to use of the kit = 0    when no component qualifies
```

**Worked example A.** One bill of materials produces 2 gift boxes from 3 candles and 1 ribbon. Free: 26
candles, 9 ribbons.

```formula
ratio of the candle = floor( 26 ÷ 3 ) = 8
ratio of the ribbon = floor(  9 ÷ 1 ) = 9
quantity free to use of the kit = floor( minimum(8, 9) × 2 ) = 16 gift boxes
```

**Worked example B.** One bill of materials produces 1 desk from 1 table top and 4 legs. Free: 7 table
tops, 30 legs. `floor(7 ÷ 1) = 7`, `floor(30 ÷ 4) = 7`, therefore `minimum(7, 7) × 1 = 7` desks.

### 18.2 Distance between two addresses

Stores are ordered by increasing great-circle distance from the reference address, computed on a sphere of
radius 6371 kilometres.

```formula
latitude difference  = ( latitude of the store − latitude of the reference ) in radians
longitude difference = ( longitude of the store − longitude of the reference ) in radians
a = sine( latitude difference ÷ 2 ) squared
    + cosine( latitude of the reference in radians ) × cosine( latitude of the store in radians )
      × sine( longitude difference ÷ 2 ) squared
distance = 2 × 6371 × arc tangent of ( square root of a ÷ square root of ( 1 − a ) )
```

**Worked example.** The reference address is at latitude 50.8503 and longitude 4.3517; the store is at
latitude 51.2194 and longitude 4.4025.

```formula
latitude difference  = 0.3691 degrees = 0.0064421 radians
longitude difference = 0.0508 degrees = 0.0008867 radians
a = sine(0.00322105)² + cosine(0.887156) × cosine(0.893597) × sine(0.00044335)²
  = 1.03752 × 10⁻⁵ + 0.63099 × 0.62540 × 1.96560 × 10⁻⁷
  = 1.03752 × 10⁻⁵ + 7.7482 × 10⁻⁸
  = 1.04527 × 10⁻⁵
square root of a = 0.00323306 ; square root of (1 − a) = 0.99999477
arc tangent = 0.00323305
distance = 2 × 6371 × 0.00323305 = 41.19 kilometres
```

A store whose coordinates are the impossible pair 1000 and 1000 therefore sorts last, which is the
intended consequence of WS-391.

---

## 19. Feed reference measure

```formula
parse the reference unit label as ( optional digits )( letters )
base count = the digits when present, otherwise 1
base unit  = the letters, lower-cased
count      = base unit count × base count
publish the two measures only when the base unit is one of the supported measurement units
            and the count is not zero at two decimal places
reference measure      = round( count, 2 ) followed by the base unit
reference base measure = base count followed by the base unit
```

**Worked example A.** A base unit count of 6 and the reference unit `750ml`: the base count is 750, the
base unit is `ml`, the count is `6 × 750 = 4500`, therefore the feed publishes `4500ml` and `750ml`. The
external service then prints a price per 750 millilitres equal to `65.00 ÷ 6 = 10.83` for a pack sold at
65.00, which is the same value the storefront prints.

**Worked example B.** A base unit count of 0.5 and the reference unit `kg`: the base count is 1, the count
is 0.5, therefore the feed publishes `0.5kg` and `1kg`.

The supported measurement units are exactly: ounce, pound, milligram, gram, kilogram, fluid ounce, pint,
carat, quart, gallon, millilitre, centilitre, litre, cubic metre, inch, foot, yard, centimetre, metre,
square foot and square metre, written with the abbreviations the external service requires. A unit outside
this set suppresses both measures.

---

## 20. Ordering values, reorder grouping and the content block limit

### 20.1 Ordering values

```formula
default shop ordering value     = the highest existing shop ordering value + 5
default shop ordering value     = 10000    when no product exists
initial fill on installation    = the highest value + 5 × the row index, one row at a time
default category ordering value = the highest existing category ordering value + 5
default category ordering value = 10000    when no category exists
move to the top    : the ordering value becomes the smallest existing value − 5
move to the bottom : the ordering value becomes the largest existing value + 5
move up            : swap with the greatest value strictly below, among products of the same
                     publication state
move down          : swap with the smallest value strictly above, among products of the same
                     publication state
```

**Worked example.** Three published products with the ordering values 10000, 10005 and 10010. Moving the
third one up swaps it with the second: the values become 10000, 10010 and 10005, so the display order
becomes first, third, second. Moving the first one up finds no product below it and therefore sets its
value to `10000 − 5 = 9995`.

### 20.2 Reorder history grouping

```formula
days ago = today − the order date of the line, in whole days
label    = "Today"                    when days ago = 0
label    = "Yesterday"                when days ago = 1
label    = days ago + " days ago"     otherwise
```

Groups keep their insertion order, which is the order-date-descending order of the source orders.

### 20.3 Content block limit heuristic

When a product content block must show one entry per product instead of one per variant, the search limit
is temporarily raised to the square of the requested limit, the records are mapped onto their templates,
and the list is truncated to the requested limit.

**Worked example.** A block asks for 4 products. The search runs with a limit of 16 variants. When those 16
variants belong to only 3 templates, the block shows 3 entries; the heuristic does not guarantee 4.

---

## 21. The reputation economy of the forum

Every award is an integer number of points added to or subtracted from the reputation score of one
participant. The amounts are fields of the Forum record; the defaults are in [entities.md](entities.md)
§4.1 and are used in the examples below.

### 21.1 The awards

| Event | Beneficiary | Default amount |
|---|---|---|
| A question becomes active | its author | +2 |
| A question receives an up-vote | its author | +5 |
| A question receives a down-vote | its author | −2 |
| An answer receives an up-vote | its author | +10 |
| An answer receives a down-vote | its author | −2 |
| An answer is accepted | the author of the answer | +15 |
| An answer is accepted | the participant who accepts it | +2 |
| A post is marked offensive, or a question is closed as spam or offensive | the author | −100 |
| A validated address on the public profile | the account, only when its score is 0 | set to 3 |

### 21.2 Vote arithmetic

A vote row stores one of three values: `1`, `0` or `-1`. Every create and every change moves the
beneficiary's score by the **difference** between the award of the new value and the award of the old
value, where the award of `0` is zero.

```formula
award( "1" )  = the up-vote award of the question, or of the answer
award( "-1" ) = the down-vote award of the question, or of the answer
award( "0" )  = 0
reputation change = award( new value ) − award( old value )
```

**Worked example on an answer**, with the default awards +10 and −2.

| Action | Old value | New value | Change | Running score of the author |
|---|---|---|---|---|
| Start | | | | 100 |
| A participant up-votes | `0` (no row) | `1` | `+10 − 0 = +10` | 110 |
| The same participant up-votes again, which withdraws | `1` | `0` | `0 − 10 = −10` | 100 |
| The same participant down-votes | `0` | `-1` | `−2 − 0 = −2` | 98 |
| The same participant up-votes | `-1` | `1` | `+10 − (−2) = +12` | 110 |

**Worked example on a question**, with the default awards +5 and −2: an up-vote followed by a down-vote by
the same participant moves the author by `+5` and then by `−2 − 5 = −7`, ending 2 points below the start.

### 21.3 Acceptance arithmetic

Accepting an answer adds the acceptance award to the author of the answer and the acceptance bonus to the
participant who accepts, unless the two are the same person, in which case nothing is added at all.
Withdrawing the acceptance subtracts the same two amounts. Deleting an accepted answer subtracts them as
well.

**Worked example.** Anne asks a question, Ben answers, Anne accepts. Ben's score rises by 15 and Anne's by
2. Anne then withdraws the acceptance: Ben loses 15 and Anne loses 2. If Anne had answered her own question
and accepted her own answer, neither amount would have moved.

### 21.4 Closing deductions

Closing a question with the reason `Contains offensive or malicious remarks` subtracts the flagging award,
by default 100 points, from the author. Closing it with the reason `Spam or advertising` subtracts the same
amount, multiplied by ten when the question is the author's **first** question in that forum. Reopening
gives the deduction back with the same multiplier.

**Worked example.** A participant with 250 points posts their first question in a forum and it is closed as
spam: they lose `100 × 10 = 1000` points and end at −750. If it had been their second question, they would
have lost 100 and ended at 150.

### 21.5 Gates

Every operation of the forum compares the participant's score with a threshold of the forum; the complete
list of thresholds is in [entities.md](entities.md) §4.1 and the refusal messages are WS-581 to WS-594 in
[business-rules.md](business-rules.md). The comparison is "the score is greater than or equal to the
threshold". An administrator passes every gate.

---

## 22. The relevance score of a question

```formula
sign             = +1 when the vote count is zero or positive, −1 when it is negative
age in days      = the whole number of days between the creation moment and today
relevance = sign × ( | vote count − 1 | raised to the power of the first relevance parameter )
                 ÷ ( ( age in days + 2 ) raised to the power of the second relevance parameter )
relevance = 0    when the post has no creation moment
```

The two parameters are fields of the Forum record; their defaults are 0.8 for the vote exponent and 1.8 for
the time decay. The value is stored and recomputed whenever the vote count or either parameter changes.

**Worked example.** A question with 5 votes created 3 days ago, with the default parameters.

```formula
| 5 − 1 | raised to the power 0.8 = 4^0.8 = 3.0314331
( 3 + 2 ) raised to the power 1.8 = 5^1.8 = 18.119229
relevance = +1 × 3.0314331 ÷ 18.119229 = 0.1673050
```

A question with the same 5 votes created 30 days ago scores
`3.0314331 ÷ 32^1.8 = 3.0314331 ÷ 543.9188 = 0.0055733`, which is why the relevance ordering favours recent
activity so strongly. A question with a vote count of 1 scores exactly 0, because `|1 − 1| = 0` and zero
raised to a positive power is zero.

---

## 23. Tag similarity and related questions

The related questions of a question are the at most five questions with the highest tag similarity,
measured as the size of the intersection of the two tag sets divided by the size of their union.

```formula
tag similarity = the number of tags carried by both questions
               ÷ the number of tags carried by at least one of the two questions
```

The value runs from 0, no tag in common, to 1, identical tag sets. Questions are ordered by similarity
descending and then by last activity descending, and at most five are returned. A question with no tag has
no related questions.

**Worked example.** The current question carries the tags `install`, `upgrade` and `database`.

| Candidate | Its tags | Common | Union | Similarity |
|---|---|---|---|---|
| A | `install`, `upgrade`, `database` | 3 | 3 | `3 ÷ 3 = 1.0000` |
| B | `install`, `database` | 2 | 3 | `2 ÷ 3 = 0.6667` |
| C | `install`, `printing` | 1 | 4 | `1 ÷ 4 = 0.2500` |
| D | `printing` | 0 | 4 | not returned: it shares no tag |

The order is A, B, C.

---

## 24. Tracked link codes

```formula
initial length = 3
candidate      = a text of ( length ) characters drawn at random from the 26 lower-case letters,
                 the 26 upper-case letters and the 10 digits
```

For a batch of n links, n candidates are drawn. When the n candidates are not all distinct, or when at
least one of them already exists, the length is increased by one and the whole batch is drawn again. The
loop repeats until a batch of n distinct, unused codes is produced. The stored code is unique across the
whole catalogue; the message of the constraint is `Code must be unique.`

A tracked link itself is identified by the tuple of its target address, its campaign, its medium, its
source and its label; asking for a link with the same tuple returns the existing one rather than creating a
second.

**Worked example.** With three characters the space holds `62³ = 238328` codes. Asking for one code when
238000 already exist has a probability of about 0.14 of colliding on the first draw; on a collision the
length becomes 4, which offers `62⁴ = 14776336` codes, and the draw is repeated. The length never
decreases within one call.

---

## 25. The published-products condition of a storefront category

The flag "this category has published products" is both computed and searchable; the two use the same
condition, so that a listing and a record rule cannot disagree.

1. Select the products that are active, published and match the site sellable-product condition of
   [entities.md](entities.md) §6.7.
2. Select the categories that hold at least one of those products directly.
3. Expand that set to every identifier appearing in the materialised path of those categories, dropping the
   trailing empty segment. This adds all the ancestors.
4. The flag is true for the categories in the expanded set and false for every other category.

The evaluation is performed with elevated privileges, because the record rule that hides empty categories
reads the flag itself. Only the comparison "the flag is true" is supported by the search.

**Worked example.** `Furniture` has the child `Chairs`, which has the child `Office chairs`. One published
product is attached to `Office chairs` only.

| Category | Materialised path | Holds a published product directly | Flag |
|---|---|---|---|
| Furniture | `1/` | no | true, as an ancestor |
| Chairs | `1/2/` | no | true, as an ancestor |
| Office chairs | `1/2/3/` | yes | true |
| Lighting | `4/` | no | false |

A public visitor therefore sees `Furniture`, `Chairs` and `Office chairs` in the category tree and does not
see `Lighting`.

---

## 26. Visit counting, connection state and retention

```formula
a new visit is counted when   the previous last connection moment < the present moment − 8 hours
visit count at creation = 1
connected = the last connection moment > the present moment − 5 minutes
a track outside the page pipeline is created when
        no matching track exists for the visitor
     OR the most recent matching track is older than 30 minutes
deletion condition = the contact is empty
                 AND the last connection moment < the present moment − the retention period
retention period = the value of the parameter website.visitor.live.days, 60 days by default
```

**Worked example.** A visitor is created at 09:00 with the visit count 1. They browse at 09:05, 09:40 and
14:00: no new visit is counted, because each previous moment is less than eight hours old, but the last
connection moment is refreshed each time and a track row is added each time. They return at 23:30: the
previous moment 14:00 is more than eight hours earlier, so the visit count becomes 2. At 23:33 they are
still reported as connected, because 23:30 is less than five minutes ago; at 23:36 they are not. If they
never return and never sign in, the daily cleanup deletes them and their track rows on the sixtieth day
after 23:30.

---

## 27. Translation coverage of the configurator

Before asking the text generation service for content, the configurator measures how much of the rendered
content blocks exists in the site's default language.

```formula
translation coverage = 1                                     when the target language code begins with
                                                             the two letters of English followed by an
                                                             underscore
translation coverage = the number of placeholder terms whose translated text differs from the source text
                     ÷ the total number of placeholder terms
translation coverage = 0                                     when there is no placeholder term
generated text is requested only when translation coverage > 0.8
```

**Worked example.** The chosen blocks contain 60 placeholder terms. In a site whose default language is
French, 50 of them have a French text that differs from the shipped source text:
`50 ÷ 60 = 0.8333`, which is greater than 0.8, so the generation runs. With only 48 translated terms the
coverage is `48 ÷ 60 = 0.8000`, which is **not** greater than 0.8, so the shipped texts are kept and the
decision is recorded. In a site whose default language is a variant of English the coverage is 1 by
construction and the generation always runs.

Each generated text then has every occurrence of the marker `XXXX` replaced by the site name.

---

## 28. Reporting measures, wish list price drop and the display currency

### 28.1 Reporting measures

```formula
abandoned cart, in the sales analysis row =
        the order date ≤ the present moment − ( the site's abandoned-cart delay, or 1 hour )
    AND the order has a site
    AND the order state is draft
    AND the order contact is not the public contact
abandoned cart count of a team  = the number of orders that are abandoned carts, have not been mailed
                                  and belong to that team
abandoned cart amount of a team = the sum of the order totals over the same set
online sales measure of the periodic digest =
        the sum of the line subtotals over the sales analysis rows of the period whose state is not
        draft, cancelled or sent and whose site is set, per company
```

**Worked example.** Two abandoned carts of one team, totalling 120.00 and 80.00, neither mailed: the team
reports a count of 2 and an amount of 200.00. Marking one as mailed leaves a count of 1 and an amount of
80.00.

### 28.2 Wish list price drop

```formula
price dropped = compare( the price stored on the row, the current price of the product ) = 1
```

**Worked example.** A shopper saves a product at 114.35. A promotion later brings it to 99.00. The
comparison of 114.35 with 99.00 returns 1, so the wish list page shows the drop, with 114.35 struck
through.

### 28.3 The display currency

```formula
display currency = the currency of the request price list, during a storefront request
display currency = the currency of the site's company, otherwise
```

Because the price list can change during a session — through the selector, a promotional code or an address
change — the displayed currency changes with it, and every amount on the page is recomputed in the new
currency. Amounts stored on the cart are rewritten by the price recomputation that follows a price list
change.

**Worked example.** The site's company keeps its books in euro, currency code `EUR`, and the site offers a
second price list in pound sterling, currency code `GBP`, at a rate of 0.85 pound per euro. A background
job that has no request running reads the display currency and gets the company currency, the euro, which
is why scheduled amounts such as the abandoned-cart report are expressed in euro. A shopper whose session
resolved the pound price list opens a product priced at 100.00 euro: the page renders 85.00 with the pound
symbol, because inside that request the display currency is the price list currency. The same shopper
switches back to the euro price list in the selector; the cart is re-priced and the line that stored 85.00
in pound sterling is rewritten as 100.00 in euro, so no amount is ever displayed in one currency while
stored in another.

---

## 29. Cache keys and lifetimes

| Cache | Key | Lifetime | Invalidated by |
|---|---|---|---|
| Page response | The site identifier, the language code, the request path, the diagnostic flag and the all-consents-granted flag (WS-070) | 3600 seconds | A change of page address, visibility or authorised groups; the deletion of a page or model page; any change to a menu entry |
| Site resolution from a host | The request host and the fallback flag | The lifetime of the process registry | Any write on a Website |
| Site index document | The site identifier and an eight-character digest of the request root | 12 hours | Regeneration deletes every document of that site and host |
| Price list candidates | The five inputs of §10.1 | The lifetime of the process registry | Any create, write or delete on a price list |
| Product feed document | The feed | Until the start of tomorrow | A change of site, price list, language or categories, and the manual reset |
| Advertised payment methods block | The site | One week, with one further day of stale reuse | Not cached at all for internal users |
| Menu cache flag | The user and the site | Until the template cache is cleared | The template cache |
| Routing table | The site | The lifetime of the process registry | A create, write or delete of a rewrite rule of type `308` or `404` |

**Worked example of the page cache.** A public visitor requests `/about` in French on site 2 with the
diagnostic flag off and all consents granted. The key is the tuple (2, French, `/about`, off, granted). A
second visitor with the same profile is served the stored response after its request forgery token and its
cart counter have been rewritten. A third visitor who refused optional cookies has a different
all-consents-granted flag and therefore a different key, so they receive a separately rendered and
separately stored response, which is what keeps the neutralised third-party content out of the response
served to the first two.
