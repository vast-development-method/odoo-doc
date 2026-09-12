# Storefront engagement and merchandising

The storefront features that surround the catalogue and the cart: wish lists, product comparison,
cross-selling and upselling blocks, dynamic product and category content blocks, visitor product tracking,
promotions on the storefront, newsletter subscription, donation pages, course products and print-on-demand
products.

---

## 1. Wish lists

A wish list is a flat list of saved variants. It works for anonymous visitors, bound to the browsing
session, and for signed-in shoppers, bound to their contact, and it migrates from the first form to the
second at sign-in.

### 1.1 Adding

The add endpoint is public and takes a variant.

1. Load the variant.
2. The price is the price of its own combination information payload, that is the price the shopper
   currently sees, in the display currency and tax mode.
3. When the visitor is the public user, create the row with elevated privileges and with no owner;
   otherwise create it for the user's contact. The row also records the current price list, the display
   currency and the site.
4. When the row has no owner, append its identifier to the session's anonymous row list.
5. Return the created row.

Capturing the price at insertion time is what allows the wish list page to show a price drop: the stored
price is compared with the current one ([calculations.md](calculations.md) §28.2).

### 1.2 Reading

1. When there is no request, there is nothing to read.
2. When the visitor is the public user, the rows are those whose identifier is in the session's anonymous
   row list, read with elevated privileges.
3. Otherwise the rows are those whose owner is the user's contact and whose site is the current one.
4. The result keeps only the rows whose product's template is published and can still be added to the cart.

The final filter silently hides a saved product that was unpublished or archived, without deleting the row
(WS-446).

### 1.3 Removing

1. When the visitor is the public user, the row is deleted only when its identifier is present in the
   session list; the identifier is removed from the list first and the session is marked as modified.
2. Otherwise the row is deleted with the user's own access rights, which the record rule restricts to their
   own rows.
3. The answer is a success flag.

### 1.4 The page and the identifier endpoint

The wish list page renders the saved products with their internal references suppressed. The identifier
endpoint returns the identifiers of the currently saved variants, which lets the shop grid mark the products
that are already saved. The shop listing also receives, as an additional rendering value, the templates of
the saved products, which is what fills the wish list marker on a product card.

The wish list page layout is controlled by the four site settings of [entities.md](entities.md) §5.1:
the tile design, the number of columns, the number of columns on narrow screens and the gap, all written
through the same editor endpoint that writes the shop and product page settings.

### 1.5 Migration at sign-in

On a successful credential verification, when the session holds wish list identifiers:

1. Read the session rows with elevated privileges and the rows already owned by the signing-in user's
   contact.
2. Delete the session rows whose product the contact already owns.
3. Assign the remaining session rows to the contact.
4. Clear the session key.

### 1.6 Cleanup

A periodic cleanup deletes, archived rows included, every row that has no owner and was created more than
five weeks ago. Anonymous wish lists are therefore bounded in time even though the sessions that own them
are gone. The recommended additional bound is WS-905.

### 1.7 Access

The model access and the record rules are in [configuration.md](configuration.md) §4.1 and §4.2. A portal or
internal user sees only the rows whose owner is their own contact; a sales manager sees every row; the
public group has no access at all, so anonymous rows are created, read and deleted with elevated privileges
under the protection of the session list.

### 1.8 Back-in-stock toggle

See [storefront-stock-and-pickup.md](storefront-stock-and-pickup.md) §3.3.

---

## 2. Product comparison

### 2.1 The page

The comparison page is public and takes a comma-separated list of variant identifiers in the query string.
Entries that are not numbers are ignored. With no valid identifier the shopper is redirected to the shop.
The variants are then **searched** rather than browsed, which applies the reader's access rules and silently
drops the ones they may not see. The page renders them with their internal references suppressed.

The functional guidance limits a comparison to four products at a time; the limit is enforced by the page
that collects the selection, not by the endpoint.

### 2.2 The data endpoint

The data endpoint is public and takes a list of variants. It returns one entry per readable variant,
carrying the variant identifier; the display name from the combination information payload; the variant
address; the address of the 1024-pixel variant picture; the price from the same payload; the zero-price
flag; the display currency; and a strikethrough price, which is the reference price when the payload reports
a discount, otherwise the comparison price when it is greater than the price, and otherwise absent.

### 2.3 The comparison matrix

1. The attributes are the attributes of the valid attribute lines of the compared templates, sorted.
2. The sections are one per attribute category of those attributes, in category order, plus a trailing
   section with no category when at least one attribute has none.
3. For each attribute and each compared product, the cell holds the attribute values of that product for
   that attribute or, when the product has no value because the attribute does not create variants, every
   value offered by that product's attribute line for the attribute.

The result is read as: section, then attribute row, then one cell per product.

On a product page the same grouping is applied to the attribute lines alone, and a specification variant of
it first removes the attribute lines that have exactly one value which is a free-text custom value, because
printing an empty custom field as a specification is meaningless.

---

## 3. Cross-selling and upselling blocks

The three links and where they surface are described in
[storefront-catalogue.md](storefront-catalogue.md) §15. This section specifies the dynamic content blocks
that render them.

### 3.1 Shipped filters

The seven shipped filters, their providers, the fields they request and their limits are in
[configuration.md](configuration.md) §5.5.

### 3.2 The common product condition

```formula
base condition = the publication flag is true, for a public or portal reader
             AND the site condition
             AND ( the company is empty OR the company is the current site's company )
             AND the block's own search condition
```

Every provider narrows this condition with its own identifier set and applies the block's limit.

### 3.3 The providers

**Recently sold.** The last eight confirmed orders of this site and company, ordered by order date
descending; their lines are counted per variant, or per the template's first variant in variant-hiding mode;
the most common variants, up to the limit, are searched against the base condition and then re-sorted by
decreasing count.

**Recently viewed.** Requires a known visitor. The visit entries of that visitor that carry a published
product not already in the cart are grouped per product and ordered by the most recent visit descending,
limited. The resulting identifiers are filtered through the base condition and then re-ordered to the visit
order, because a search would lose it. The rendered entries carry a flag that makes the add-to-cart control
re-render.

**Sold with.** Requires a product. The last eight confirmed orders of this site and company that contain the
given template; the products of their lines, minus the products already in the cart and minus the variants
of the given template, are searched against the base condition.

**Accessories.** Requires a product. The accessory variants of the given template that pass the storefront
filter, minus the products already in the cart and minus the variants of the given template.

**Alternatives.** Requires a product. The variants of the alternative templates of the given template, minus
the products already in the cart and minus the variants of the given template.

**Category list.** The parent category and its children when a parent is given, and otherwise the top-level
categories, all matching the availability condition of [entities.md](entities.md) §5.2. Each entry carries
the identifier, the name, whether it has no published product, and the cover picture address, falling back
to the product placeholder picture.

### 3.4 Variant-hiding mode

A product block may be configured to show one entry per product rather than one per variant.

1. The limit is temporarily raised to the square of the requested limit, because several variants may
   collapse onto one template and the block must still be able to fill itself
   ([calculations.md](calculations.md) §20.3).
2. The records are mapped onto their templates and truncated to the requested limit.
3. For each entry: when the template is not configurable, the entry falls back to the single variant, so
   that the add-to-cart action stays available; the price payload is the variant payload for a variant and
   the template-only payload for a template, and in the latter case the template's first variant is
   re-attached to the entry.

The squared limit is an approximation and may still be insufficient when a product has very many variants; a
rebuild may choose a different strategy as long as the block ends up with at most the requested number of
distinct products.

### 3.5 Access and samples

A product or category block renders nothing at all when the current visitor has no shop access. In the
editor, when no real record matches, sample entries are produced: six sample products — a chair, a lamp, a
whiteboard, a drawer unit, a box and a bin, each with a picture and a one-line description — and four sample
categories — desks, furniture, boxes and drawers, each with a cover picture — merged with the generic sample
payload of WS-260.

### 3.6 Currency

Product blocks display amounts in the current site's display currency.

---

## 4. Visitor product tracking

| Endpoint | Behaviour |
|---|---|
| Record a view | Resolves the visitor of the request, creating one when needed, and records a view of the given variant. The view is skipped when the variant is not a possible combination of its template. |
| Forget a view | Deletes the visit entries of the current visitor for one variant, or for every variant of one template. Does nothing when no visitor exists. |

The product page carries a tracking marker that triggers the recording call once the page is displayed.

The visitor record then reports the total number of product views, the number of distinct products viewed
and the list of those products, all restricted to products of the companies the reader may see. A dedicated
history action opens the visit entries of one visitor that carry a product, with a graph view of the same
data.

---

## 5. Promotions on the storefront

The promotion engine itself is owned by
[loyalty and promotions](../loyalty-and-promotions/README.md). The storefront adds the following.

### 5.1 Entry points

| Endpoint | Behaviour |
|---|---|
| The promotional code form | Tries the entered text as a loyalty or coupon code first. When the engine reports "not found", the text is tried as a price list promotional code ([storefront-catalogue.md](storefront-catalogue.md) §13.3). When the engine reports an error, the message is kept in the session. On success, when the code yields exactly one reward, or the requested reward among several, and that reward is not a multi-product reward or a product was specified, the reward is applied; the code is then kept in the session as the successful code. The shopper is redirected to the return address, the cart page by default. |
| The coupon address | Records the code in the session as the pending coupon. When a cart exists, the code is applied at once: on failure the error is put in the redirect query, and on success the code is put there as a notification. When no cart exists, the redirect carries the warning `The coupon will be automatically applied when you add something in your cart.` Any pre-existing error keys in the redirect address are discarded, so that only genuine messages are shown. |
| The reward claim | Applies a claimable reward to the cart. A reward identifier that is not a number, an unknown reward or a missing cart redirects to the return address. A multi-product reward requires a product. When the reward's program is code-driven and the supplied code matches the coupon, the promotional code form is invoked instead, which both applies the code and claims the reward. |
| The wallet top-up | Signed-in users only: adds one unit of the electronic wallet's trigger product to the cart and redirects to the cart. |

Applying a reward:

1. Try to apply the program reward, for the product held in the context when there is one.
2. On a user error or an engine error, keep the message in the session and report failure.
3. Refresh the programs and rewards.
4. When the order's delivery method offers free shipping above a threshold and the reward's program is not a
   payment program, rate the method again: on success rewrite the delivery line, on failure remove it.
5. Report success.

### 5.2 Automatic rewards

After every cart update and on the cart page, the claimable rewards are re-evaluated and applied
automatically, except when the reward's program carries more than one reward, or the program is nominative,
or the reward is a free-product reward with several product choices, or the reward is in the order's
disabled list, or it is already on an order line.

Removing a reward line from the cart adds that reward to the order's disabled list, which prevents it from
reappearing on the next update. Nominative programs are not offered to anonymous visitors.

### 5.3 Display

* The discount lines produced by one reward across several tax groups are shown as a single aggregate line
  in the cart.
* Reward lines do not count in the cart quantity.
* Reward lines never show a strikethrough price.
* Only free-product reward lines are clickable, that is treated as sellable.
* Free-shipping reward lines are excluded from the amount used by the free-shipping threshold, which
  prevents a free-shipping reward from covering its own threshold.
* The order summary of the delivery step carries the discounted delivery amount and the list of formatted
  discount amounts.

### 5.4 Claimable and showable rewards

The cart shows, besides the rewards the shopper can claim right now, the rewards attached to their loyalty
cards.

1. Start with the claimable rewards.
2. Take the loyalty cards of the order's contact whose program matches the storefront program condition and
   which are either code-driven or automatic programs that apply to future orders.
3. For each card, read the real point balance for this order, and for each reward of the card's program that
   is not already on a line: skip a global discount when a better global discount is already applied; skip a
   discount reward when the order total is zero; skip the card when it has expired; and otherwise keep the
   reward when the balance reaches its required points.

### 5.5 Cleanup

A periodic cleanup releases the coupons attached to storefront carts untouched for longer than the number of
days given by the coupon validity parameter, whose default is 4, and re-evaluates those carts.

### 5.6 Program configuration on the storefront

* The availability flag, true by default, decides whether a program applies to storefront orders; the
  storefront substitutes this flag for the generic "available in sales" flag in the program and trigger
  conditions, and additionally requires the program to be generic or attached to the current site.
* An electronic wallet program whose trigger products are not all published raises a configuration warning,
  because the shopper would not be able to top up the wallet online.
* A promotional code must be unique among the rules reachable from one site; a coupon and a program may
  never share a code (WS-458).
* The portal page of a loyalty card additionally lists the published trigger products of the program with
  their formatted price.

---

## 6. Newsletter subscription at checkout

When the newsletter page option is active, the address form shows a subscription box. The submitted value is
not a contact field, so it arrives as extra form data. On submission, when the box is ticked and the
submitted address carries an electronic mail address, that address is subscribed to the site's newsletter
list under the submitted name, through the ordinary newsletter subscription operation.

The setting is stored as the active state of the newsletter page option, not as a field: writing the setting
toggles the page option of the selected site, and reading it reports whether that option is active.

---

## 7. Donations

The donation feature is a payment page that is not a shop order.

| Endpoint | Behaviour |
|---|---|
| The donation page | Behaves like the generic payment page, with the donation flag set. A submission by form stores the amount, the currency, the options and the descriptions in the session and answers with a redirect to the same address, which turns the submission into a normal page load. On a page load, the values are taken from the session when they are not in the address. The currency defaults to the company currency and the amount to 25.0. For an anonymous visitor the site's public contact is used and an access token is generated for the amount and the currency. |
| The donation transaction | Creates the donation transaction. An amount below the minimum is refused with `Donation amount must be at least %.2f.` For an anonymous visitor the donor details are mandatory: `Name is required.`, `Email is required.` and `Country is required.`; the transaction is then attached to the site's public contact, tokenisation is disabled, and the donor name, address, country and language are written on the transaction. For a signed-in shopper the transaction is attached to their own contact and the country is taken from the supplied details when the contact has none. The access token is generated again from the final amount, because the donor may have changed it on the page. An internal notification message is sent immediately. |
| The advertised payment methods block | Returns the payment methods advertised on the site as pairs of a name and a picture address: the brands of every primary method that has at least one compatible published provider, plus the primary methods that have no brand. The providers are resolved as the site's public user, in the site's company, which makes an editor see exactly what a visitor sees. The answer is not cached for internal users and is cached for one week, with one further day of stale reuse, for everybody else. |

On completion of a donation transaction the donation confirmation message is sent to the donor and the donor
details are logged on the resulting payment: the company, the contact, the donor name, the donor country and
the donor address, each prefixed by its label. The "save my payment details" option is hidden on a donation
page for an anonymous visitor.

---

## 8. Course products

A course may be sold: its enrolment mode becomes the payment mode and it names the product whose purchase
grants access.

| Rule | Behaviour |
|---|---|
| Product kind | The linked product must carry the course service tracking value. A paid course without a product is refused with `Product is required for on payment channels.` |
| Add to cart | A course product may always be added to the cart when at least one published course uses it, even when the product itself would fail the ordinary storefront checks. |
| Quantity | A course may be bought only once per cart: a request above 1 is reduced to 1 with the warning `You can only add a course once in your cart.` |
| Reorder | Course lines are excluded from reordering. |
| Zero price | The course tracking value is exempt from the zero-price rule, so a free course may be added even when the site forbids zero-price sales. |
| Line description | The sales description of a course product is replaced by `Access to: ` followed by the names of the paid courses it grants, one per line when there are several. |
| Confirmation | On order confirmation the customer is added as a member of every paid course whose product appears on the order. |
| Publication | Publishing a paid course publishes its product; unpublishing a course unpublishes its product unless another published course still uses it. |
| Confirmation page | The page receives the enrolment records created for the buyer, which lets it link straight to the purchased courses. |
| Course page | A paid course page shows the combination information payload of its product, resolved through a search, which applies the access rules. |
| Reporting | A course reports the summed line total of the completed sales of its product, in the product currency, visible to the sales user group. |

---

## 9. Print-on-demand products

Products fulfilled by an external print-on-demand service impose three storefront rules.

| Rule | Trigger | Message |
|---|---|---|
| Print pictures before publication | Publishing a product of a print-on-demand template that still lacks print pictures | `Print images must be set on products before they can be published.` |
| No removal of print pictures while published | Emptying a print picture of a published product | `Products must be unpublished before print images can be removed.` |
| No mixing in one cart | Adding a goods product whose print-on-demand nature differs from a goods product already in the cart | `The product %(product_name)s cannot be added to the cart as it requires separate shipping. Please place your order for the current cart first.` The added quantity becomes zero. |

Service products are ignored by the mixing rule on both sides. Synchronising a print-on-demand template that
creates new print pictures unpublishes the product, so that it cannot be sold while its artwork is
incomplete. The storefront description of such a product is filled from the external template description
when the variants are created. Express checkout is allowed for a cart containing print-on-demand products
only when the customer address is already complete, because the external service requires a full address.

---

## 10. Ratings and reviews

The product page can show a discussion and rating block. When that block is inactive, an external user may
not post a message on a product: the required permission becomes write access, which they do not have. Only
ratings by users who are not internal are counted in the average and in the review count, and the rating
statistics of a product may be published.

---

## 11. Merchandising controls available to an editor

| Control | Written values |
|---|---|
| Reorder a product on the grid | The shop ordering value through the four move operations, or the tile width and tile height together. |
| Change an attribute's display type | The attribute display type; the page template cache is cleared afterwards. |
| Change the page style settings | Only the fields of the writable set: the shop container, the products per page, the products per row, the default sort, the gap, the shop design tokens, the product page container, the media layout, the media width, the grid columns, the media spacing, the media ratio, the media ratio on narrow screens, the column order, the media rounding and the call-to-action design; plus the four wish list settings when the wish list capability is installed; plus the extra-step flag. A products-per-page value of zero is stored as 1. |
| Change a category's page options | Only the three category page options: show the title, show the description and centre the content. |
| Set a category cover picture | The cover picture, taken from an attachment. |

All of these require the Restricted Editor group, except the category page options, which require only that
the category exists and that the caller has write access to it. Failing the group check answers "not found"
for the product, attribute and site endpoints and "forbidden" for the cover picture endpoint (WS-298).
