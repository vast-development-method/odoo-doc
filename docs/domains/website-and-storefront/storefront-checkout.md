# Storefront cart, checkout and payment

The shopping cart, the checkout step machinery, the address step, the delivery step, the payment step, the
confirmation, the express checkout contract and the abandoned cart flow. Prices and taxes are computed by
the rules of [storefront-catalogue.md](storefront-catalogue.md) and
[calculations.md](calculations.md); availability checks are in
[storefront-stock-and-pickup.md](storefront-stock-and-pickup.md).

---

## 1. The cart

A cart is a Sales Order in the draft state whose site is set. It is created lazily, the first time the
shopper adds something.

### 1.1 Creating a cart

1. The contact is the current user's contact, which is the site's public contact for an anonymous visitor.
2. The creation values are: the site's company; that contact; the request fiscal position, written
   explicitly **only** for the public user, because a signed-in contact's fiscal position is derived from
   their own address by the standard computation; the request price list; the site's sales team; and the
   site.
3. The order is created as the platform system user, in the company of the site. A supplied company that
   differs from the site's company is refused by WS-343.
4. The order is read back as the current user with elevated privileges.
5. The session records the order identifier and the cart quantity, and the order is bound to the request.

### 1.2 Resolving the cart of a request

On every storefront request the cart is resolved lazily and cached in the session.

1. When the session holds a cart identifier, that order is read with elevated privileges.
   1. When the order no longer exists, the storefront session is reset and there is no cart.
   2. When the order exists but its state is not draft, or its last portal transaction is pending,
      authorised or completed, or it belongs to another site, the storefront session is reset and there is
      no cart (WS-340).
   3. When the order exists, the user is not the public user, the user's contact differs from the order's
      contact and the request may write, the customer is rewritten through the address propagation of §6.6.
2. Otherwise, when the user is not the public user and their contact is allowed to trade with the site's
   company, the first draft order of that contact on this site is adopted. When one is found and the request
   may write, the address propagation runs for that contact and the archived-product cleanup of §2.7 is
   applied.
3. When an order was found, or the user is not the public user, and the resolved identifier differs from the
   session value, the session records it, storing the value "no cart" for a signed-in user who has none.
   Storing that value is deliberate: it prevents a search for an abandoned cart on every subsequent request.
4. When the session has no cart quantity yet, it is set from the order.

The storefront session reset clears the cart key, the cart quantity, the price list, the selected price
list and the fiscal position keys.

### 1.3 The cart state machine

The states, the transitions, their guards and their side effects are
[state-machines.md](state-machines.md) §11. The transaction states themselves are owned by
[payment providers](../payment-providers/README.md).

---

## 2. Adding to the cart

### 2.1 The endpoint

**Input.** The template, the variant, a quantity defaulting to 1, an optional unit of measure, a list of
custom attribute values each carrying its template attribute value and its text, a list of no-variant
attribute values, and a list of linked products — optional products and combo items — each carrying its own
template, variant, quantity, custom values, no-variant values, parent template and, for a combo item, the
combo item itself.

**Steps.**

1. Resolve the cart, creating one when none exists.
2. Truncate the quantity to a whole number. The storefront never adds fractional quantities.
3. Load the variant; when no-variant values were supplied, evaluate it with those values in its price
   context.
4. Guard: when the variant does not exist or the add-to-cart permission check fails, raise
   `The given product does not exist therefore it cannot be added to cart.`
5. Add the main line with the cart verification suppressed (§2.2).
6. For each linked product, in the order received:
   1. Load the linked variant with elevated privileges.
   2. When the linked quantity is not zero and the variant does not exist, or the permission check fails
      and the entry is not a combo item, raise the message of step 4. Combo items are validated instead by
      the line constraint that the combo item belongs to the product's combo.
   3. Add it with the parent line taken from the map of lines already created in this request, keyed by the
      parent template. A missing key is a hard failure, which is how malformed optional-product payloads are
      rejected (WS-350).
   4. When the main product is a combo and a linked add returns a quantity of zero, the whole combo is
      abandoned: the main line is deleted, its children cascading with it, and the answer reports a quantity
      of zero with the warning produced by the failing child.
7. When the main product is a combo, equalise the combo quantities (§2.5). When that changed the
   quantities, the added quantities per line and the reported quantity are restated from the final line
   quantity, and the reported warning becomes the warning stored on the combo line.
8. Run the post-update verification once (§2.6).
9. For a combo line, run the line validity check now, because it can only be evaluated once every child
   line exists.
10. Build the answer.

**Answer.** The cart quantity after the update; the resulting quantity of the main line; the notification
payload; and the analytics payload.

The notification payload carries the display currency, a warning, and one entry per line whose added
quantity is strictly positive. Each entry carries the line identifier, the address of the 128-pixel product
picture, the quantity that was just added, the line header, the combination name, the multi-line variant
description, the amount for the added quantity — the unit price with tax when the site shows tax-included
prices and without tax otherwise, multiplied by the added quantity — the parent line for a combo item, and
the unit name when the product offers several units. When a combo item's product is not readable by the
current user but has a picture, the picture address is replaced by an inline picture payload.

The analytics payload describes every touched line: the item identifier, which is the barcode when there is
one and the identifier otherwise; the item name; the item category; the currency; the unit price excluding
tax after discount; the discount, that is the unit price minus that; and the quantity.

### 2.2 The add operation

1. Work in the company of the order.
2. When no unit was given, or the product does not support several units, use the product's own unit.
   Otherwise, when the requested unit is not among the product's available units, raise
   `This product is not available (anymore) in this unit of measure.`
3. Look for a matching line (§2.3). When one is found, update it to its current quantity plus the requested
   quantity (§2.4) and stop.
4. Verify the quantity for a new line (§2.9), which returns the allowed quantity and a warning.
5. Create the line with the prepared values (§2.8).
6. Write the warning on the created line, or on the order when no line was created.
7. Unless verification is suppressed, run the post-update verification (§2.6).
8. Return the added quantity, the line and the warning.

The product passed in must not be reused after the line is created: the line may carry a different variant,
because the combination is normalised and a dynamic variant may have been created.

### 2.3 Finding a matching line

1. An order with no lines has no match.
2. A combo product never matches: a combo always creates a new line.
3. The candidates are the lines whose product and unit equal the requested ones, that carry no custom
   attribute values, whose parent line equals the given parent line — possibly none — and that are not combo
   item lines.
4. When there are no candidates, there is no match.
5. When the product has at least one no-variant attribute that offers a real choice — more than one value,
   or a multiple-choice display — the candidates are narrowed to those whose no-variant attribute values are
   exactly the given list.
6. The first remaining candidate is the match.

A line carrying custom free-text values is therefore never merged, and two selections of different
no-variant values stay on separate lines. With the promotion capability, reward lines are excluded from the
candidates, because they are managed by the promotion engine and not by cart edits.

### 2.4 Updating a line

1. Work in the company of the order and take the order's line with that identifier.
2. When it is not found, answer with the warning
   `We weren't able to update your cart. Please refresh your page before trying again.` and change nothing.
3. When the requested quantity is greater than zero, verify it (§2.9); otherwise the warning is empty,
   because the line is going away anyway.
4. The added quantity is the resulting quantity minus the line's current quantity.
5. Apply the new quantity:
   1. A quantity of zero or less deletes the line; its child lines cascade.
   2. Otherwise, when the line is a combo line with combo item children and the quantity changes, the
      combo quantity starts at the requested quantity and is lowered, for each child whose quantity differs,
      to the quantity that child may hold; every child whose quantity differs is then updated to the combo
      quantity with verification suppressed, and the combo line takes the same value.
   3. The values are written and the line validity check runs.
6. Unless verification is suppressed, run the post-update verification.
7. Write the warning on the line when it still exists, and on the order otherwise.
8. Return the added quantity, the line, the resulting quantity and the warning.

### 2.5 Combo quantity equalisation

1. Read the linked lines of the combo line; when there are none, nothing changed.
2. The available quantity is the smallest quantity among the children.
3. When the available quantity is smaller than the combo line's quantity, write the stock warning of
   WS-400 on the combo line, set the quantity of the combo line and of every child to the available
   quantity, and report that something changed.
4. Otherwise report that nothing changed.

A combo and its items always carry the same quantity. When one item is short, the whole combo is reduced.

### 2.6 Post-update verification

1. When the order is services-only, remove the delivery line and clear the pickup location unless it must
   be kept.
2. Otherwise, when a shipping method is set, rate it: on success write the rate on the delivery line, and
   on failure remove the delivery line.
3. When this is a storefront request, refresh the session cart quantity.
4. With the promotion capability: update the applicable programs and rewards, apply the claimable rewards
   automatically, and refresh the session cart quantity again, because rewards may change it.

This runs once per request, not once per line, which is why the per-line operations accept a suppression
flag (WS-356).

### 2.7 Cart cleanup

Every line whose product exists and is archived is deleted. This runs when an abandoned cart is revived at
sign-in, and on the cart page, where lines with archived products are removed before rendering.

### 2.8 Preparing the values of a new line

1. The received combination is the variant's attribute values plus the given no-variant values.
2. The combination is normalised to the closest possible combination of the product's template.
3. The variant for that combination is resolved, and created when the template allows dynamic creation and
   it does not exist yet. When no variant could be resolved, raise
   `The given combination does not exist therefore it cannot be added to cart.`
4. When a parent line was given and it does not belong to this order, raise `Invalid request parameters.`
5. The values are the product, the quantity, the unit — the product unit when none was given — the order,
   the parent line and the combo item.
6. The no-variant values are the given ones plus every no-variant value of the normalised combination; when
   there are any, they are written as exactly that set.
7. For every value of the combination that accepts a custom value and was not supplied, an empty custom
   value is appended; when there are any, one custom value record is created per entry.

Normalising to the closest possible combination is what makes a partial or contradictory selection land on a
valid variant instead of failing.

### 2.9 Quantity verification hooks

The verification takes the line, the product, the requested quantity and the unit, and returns the allowed
quantity and a warning text. The base implementation returns the requested quantity and no warning. It is
overridden, in this order of installation:

| Capability | Behaviour |
|---|---|
| Storefront stock | Caps the quantity at the available quantity; see [storefront-stock-and-pickup.md](storefront-stock-and-pickup.md) §4 and [calculations.md](calculations.md) §14.2. |
| Print on demand | When the cart already contains a goods line whose print-on-demand nature differs from the product being added, returns the quantity zero and the warning of WS-360. Service products are ignored on both sides of the comparison. |
| Courses | A course product may be bought only once: a request above 1 returns the quantity 1 and the warning `You can only add a course once in your cart.` |

### 2.10 Line validity

After a line is written, and after a combo line has all of its children, the validity check runs: when the
line is not a combo item, the sum of the unit prices of the line and its priced linked lines is zero, the
site forbids zero-price sales and the product's service tracking value is not exempt, raise
`The given product does not have a price therefore it cannot be added to cart.`

The exempt list is empty by default; the course capability adds the course tracking value, which lets a
free course be added.

---

## 3. The cart page

1. Shop-access guard; on failure, redirect to the sign-in page.
2. Resolve the cart.
3. Abandoned-cart revival, when the address carries an order identifier and an access token (§3.1).
4. Delete the lines whose product is archived.
5. Compute the accessory suggestions ([storefront-catalogue.md](storefront-catalogue.md) §15).
6. Compute the express-checkout payment values (§9.1).
7. Compute the checkout step values (§5.2).
8. Compute the quick reorder history (§3.2).
9. Render.

With the promotion capability installed, the applicable programs and rewards are refreshed and the
automatically claimable rewards applied before rendering.

### 3.1 Reviving an abandoned cart

The recovery message links to the cart page carrying the order identifier and the order access token.

1. Read the order with elevated privileges.
2. When it does not exist, or the token does not match in a constant-time comparison, answer "not found".
3. When the order is no longer in draft, render the cart with the "already completed" flag.
4. Otherwise, when the revival method is "squash", or it is "merge" and the session holds no cart, the
   session cart becomes the old order and the browser is redirected to the cart page.
5. Otherwise, when the revival method is "merge", every line of the old order is moved onto the session
   cart and the old order is cancelled.
6. Otherwise, when the old order is not already the session cart, the cart is rendered with a prompt
   offering the two revival methods.

### 3.2 Quick reorder history

The cart page offers the products of the shopper's recent orders for re-adding in one action.

1. The recent lines are the lines of the last ten confirmed orders of the current contact on this site,
   ordered by order date descending.
2. For each line, in that order:
   1. Skip it when its parent line is a combo line.
   2. Skip it when the line is not sellable.
   3. Skip it when the site forbids zero-price sales and the variant's combination price is zero.
   4. Skip it when a line for the same product already exists in the cart or has already been kept, two
      combo lines being considered the same only when their linked products match exactly.
   5. Compute the day label of [calculations.md](calculations.md) §20.2 and append the line to that group.
3. Return the groups in insertion order.

### 3.3 Cart endpoints

| Endpoint | Behaviour |
|---|---|
| Add | §2.1. |
| Quick add | Signed-in users only. Performs the add, then returns the re-rendered cart lines block, the short cart summary block — which includes the express checkout values and the checkout step values — the re-rendered reorder history and the readiness flag. |
| Update | Updates one line. When no line identifier is supplied, the line is found from the product identifier, which is needed for promotion lines that are displayed as unsaved aggregates. Returns the cart quantity, the readiness flag, the order total, the total in minor currency units and the re-rendered cart lines, totals and reorder history blocks. |
| Quantity | Returns the session cart quantity when present, and otherwise the cart quantity read from the order. |
| Clear | Deletes every line of the cart. |

---

## 4. Checkout readiness checks

Two guards protect every checkout page.

**The cart guard.**

1. When the order does not exist or its state is not draft, clear the cart and transaction session keys and
   redirect to the shop.
2. When the order has no lines, redirect to the cart page.
3. When the current user is the public user and the site requires an account, redirect to the sign-in page
   with the checkout page as the return path.
4. When the cart contains zero-priced lines, write the per-line warning and the order warning of WS-365 and
   redirect to the cart page.

**The address guard.**

1. When the cart is anonymous, that is when its contact is still the public contact, redirect to the address
   form.
2. When the order is not services-only, the delivery address is incomplete and the current customer is
   allowed to edit it, redirect to the address form for that contact with the delivery kind.
3. When the billing address is incomplete and the current customer is allowed to edit it, redirect to the
   address form for that contact with the billing kind.

**Zero-priced lines.** When the site allows zero-price sales there are none. Otherwise they are the lines
that carry a product, that are neither a section nor a note, that are not delivery lines, whose product's
template is not a combo — combos are priced by their items — that are not combo item lines, whose unit price
is exactly 0, and whose product's service tracking value is not exempt. With the promotion capability,
reward lines are excluded.

**Readiness.** A cart is ready when it has at least one line and no zero-priced line.

---

## 5. Checkout steps

### 5.1 The shipped steps

| Ordering value | Label | Path | Forward label | Backward label |
|---|---|---|---|---|
| 0 | `Order` | `/shop/cart` | none | `Back to cart` |
| 250 | `Address` | `/shop/checkout` | `Checkout` | `Back to address` |
| 500 | `Extra Info` | `/shop/extra_info` | `Confirm` | `Back to extra info` |
| 999 | `Payment` | `/shop/payment` | `Confirm` | none |

Each site receives its own copy of every step at creation. The copy of the extra-information step is
published only when the extra-information page option is active for that site (WS-465).

### 5.2 Navigation values

On every checkout page:

1. The current address is the request path with the address rewrite rules applied.
2. When it equals the rewritten address form path, it becomes the rewritten checkout path, because the
   address form belongs to the address step.
3. The allowed steps are the site's published steps.
4. The current step is the allowed step whose rewritten path equals the current address, or none.
5. The next step is the first allowed step after the current one by ordering value; the previous step is the
   last allowed step before it.
6. The forward address is the next step's path; when that path is the checkout path, the skip flag is
   appended to it.
7. When the current request path is the address form, the forward address is suppressed, because the
   redirect after a successful submission is decided by the submission endpoint.

Comparing the paths after rewriting is what makes a site rewrite rule on a checkout path harmless.

### 5.3 Enabling or disabling the extra step

The editor endpoint that writes the page settings also accepts the extra-step flag. Writing it sets both the
page option and the publication flag of the site's extra-information step to the same value, which keeps the
step list and the page in agreement.

---

## 6. The address step

### 6.1 The combined checkout page

1. Resolve the cart and record its identifier in the session's last-order key.
2. Run the cart and address guards; redirect when they fail.
3. Build the page values: the order; whether the delivery address is also the billing address; the
   services-only flag; the shopper's billing and delivery address lists; and the address form path.
4. When the order has deliverable products: list the available delivery methods; resolve the preferred one;
   rate it; and, when the order has no method yet, or the rating failed, or the rated price differs from the
   current delivery amount, set the method on the order, which recreates the delivery line.
5. Add the checkout step values.
6. When the request asked to skip the step and the order has no deliverable products, redirect to the next
   step instead of rendering.

The address lists are built as follows: the billing candidates are the contact itself, every descendant of
its commercial contact whose kind is invoice or other, and the commercial contact itself; the delivery
candidates are the contact itself and every contact matching the commercial contact's delivery address
condition. When the shopper is a child of the commercial contact and the commercial contact's own address is
incomplete, that address is removed from the corresponding list, because a child may not edit it.

### 6.2 The address form

**Parameters.** The contact whose address is edited, the address kind — billing or delivery — and whether
the address must serve as both.

**Preparing the update.**

1. When the cart is anonymous, no contact is resolved and a new address will be created.
2. Otherwise the contact with that identifier is taken; when it is neither the order's contact, nor its
   billing contact, nor its delivery contact, it is kept only when it still exists.
3. When a contact was resolved and no address kind was given, the kind is billing when it is the order's
   billing contact, delivery when it is the order's delivery contact, and billing otherwise.
4. When a contact was resolved and the current customer may not edit it, the answer is "forbidden"
   (WS-375).

When an existing contact is being edited, the "use as both" flag is recomputed as "this contact is at once
the order's delivery and billing contact", ignoring the value supplied in the address.

**Form values.** The standard portal address values, plus the anonymous-cart flag, the order, the
services-only flag, the business-to-business field block flag — true when the portal already enables it or
when the corresponding page option is active on the site — the discard address, which is the callback when
one was given, the cart page for an anonymous cart and the checkout page otherwise, and the
commercial-address update address rewritten to the address form for the order's contact.

The default country of a new address on an anonymous cart is the country resolved from the visitor's
network address, when it is known.

### 6.3 Mandatory fields

```formula
mandatory address fields  = street, city, country
                          + state    when the country requires a state
                          + postal code when the country requires a postal code
mandatory billing fields  = name, electronic mail address
                          + telephone + the mandatory address fields
                            when the order needs a customer address
mandatory delivery fields = the same set
needs a customer address  = the order is not services-only
                         OR the parameter website_sale.require_billing_details_for_services is true,
                            which is its default
```

An order that contains only services, on an installation where that parameter has been set to a false
value, therefore requires only the name and the address.

The set actually enforced on a submission is built as follows.

1. Start with the extra fields requested by the form.
2. When the address kind is delivery, or the address serves as both, add the mandatory delivery fields of
   the submitted country.
3. When the address kind is billing, or the address serves as both, add the mandatory billing fields of the
   submitted country; and, when the address being created is not the commercial address, remove from the
   set every commercial field that is absent from the submission.
4. When the submission fills any of the common address fields, add the mandatory address fields of the
   submitted country.

The last step means that a shopper who starts typing an address must finish it, even when the order would
not have required an address at all (WS-369).

### 6.4 Validation of submitted address values

Evaluated in this order; every failure adds the field to the highlighted set and appends a message.

| Check | Condition | Message |
|---|---|---|
| Country change | The contact already has a country, the submitted country differs, and documents have been issued for the contact | `Changing your country is not allowed once document(s) have been issued for your account. Please contact us directly for this operation.` |
| Name or address change on an internal user | The name or the electronic mail address changes and at least one linked user is not an external user | `If you are ordering for an external person, please place your order via the backend. If you wish to change your name or email address, please do so in the account settings or contact your administrator.` |
| Commercial field on a child address | The contact is not its own commercial contact and a commercial field is submitted with a different value | `The %(field_name)s is managed on your company account.` when the commercial contact is a company, otherwise `The %(field_name)s is managed on your main account address.` A commercial field submitted with the same value is silently dropped from the submission. |
| Company name on a child address | The contact being edited is not the current shopper's own contact | The company name is silently dropped. |
| Tax identification number change | The contact is its own commercial contact, already carries a number, the submitted one differs, and documents have been issued | `Changing VAT number is not allowed once document(s) have been issued for your account. Please contact us directly for this operation.` The three capital letters abbreviate value-added tax. |
| Address format | The submitted electronic mail address does not match a single valid address | `Invalid Email! Please enter a valid email address.` |
| Tax identification number format | A number is submitted, the accounting capability is installed and the number was not already rejected | The message raised by the number validation of the country, reproduced verbatim. |
| Missing required fields | Any field of the required set is empty | `Some required fields are empty.` |

The answer to a failed submission carries the invalid fields and the messages; the page highlights those
inputs.

### 6.5 Writing the address

1. Run the cart guard; when it redirects, answer with that target.
2. Prepare the update (§6.2).
3. The callback is the supplied one, or the checkout path with the skip flag when the contact is new or the
   order is services-only, and the plain checkout path otherwise.
4. Create or update the contact with the validated values; when the answer reports invalid fields, return it
   unchanged.
5. Decide which order fields to write: when the cart is anonymous or the order's contact equals this
   contact, the customer field is written, which forces a full re-resolution; for the billing kind, the
   billing field, plus the delivery field when the contact is new and the order is services-only; for the
   delivery kind, the delivery field, plus the billing field when the address serves as both.
6. Propagate the change to the order (§6.6).
7. When the cart was still anonymous, unsubscribe the site's public contact from the order's followers.
8. Return the feedback.

Creating a new address on an anonymous cart forces the contact kind to "contact" rather than a child address
kind, because that contact becomes the order's customer. The language written on a new contact is dropped
when it is not one of the site's languages. The company of a new contact is the site's company, its
salesperson is the site salesperson, and, when the site keeps accounts separate per site, the new contact
records that site.

### 6.6 Propagating an address change to the order

1. When no field is requested, nothing happens.
2. Remember the current fiscal position and price list.
3. Build the values as the requested fields, each set to the contact.
4. When the customer field is requested, for each of the billing and delivery fields that was not requested
   and whose current value belongs to the same commercial contact as the new customer, keep that value
   explicitly, so that it is not reset by the customer change.
5. Write the values on the order.
6. When the fiscal position changed, recompute the order taxes and store the new fiscal position in the
   session and on the request.
7. When the session holds an explicitly selected price list: when it still exists, is available on the site
   and is available in the new country, write it on the order; otherwise forget the selection.
8. When the price list changed, or the fiscal position changed, recompute the order prices and store the new
   price list in the session and on the request.
9. When the order has a delivery method, the delivery address was among the written fields and the order has
   deliverable products, recompute the available methods, pick the preferred one and set it on the order.

### 6.7 Choosing an existing address

The address selection endpoint takes a contact and an address kind. The contact must be the order's
customer, the customer's commercial contact, or a descendant of the commercial contact whose kind is
invoice, delivery or other; otherwise the answer is "forbidden". The matching order field is then updated
through §6.6, but only when it actually changes.

### 6.8 Account modes and sign-up

| Account policy | Effect |
|---|---|
| `optional` | An anonymous shopper may complete the checkout as a guest. Sign-up is free, so the confirmation message can invite the shopper to create an account and follow the order. |
| `disabled` | The same guest checkout, but sign-up is invitation-only, so no account can be created from the storefront. |
| `mandatory` | The cart guard redirects an anonymous shopper to the sign-in page with the checkout page as the return path. |

Writing the setting also writes the site sign-up policy: free sign-up for `optional` and `mandatory`,
invitation-only for `disabled`.

The newsletter box of the address form is handled as extra form data: when it is ticked and the submitted
address carries an electronic mail address, that address is subscribed to the site's newsletter list under
the submitted name.

The address autocompletion service is used when the site carries an access key; the storefront supplies the
site key instead of the back-office key, and exposes a check that tells the page whether an autocompletion
key exists at all.

An address that is an external relay point may not be edited: the attempt raises
`You cannot edit the address of a Point Relais®.` and the completeness check of such an address always
succeeds, because the shopper cannot fix it.

---

## 7. The extra information step

When the extra-information page option is inactive, the request is redirected to the payment page. The cart
guard applies, except when the page is opened by an editor with the editing flag in the address, which lets
an administrator design the page without a cart.

The page renders a public form whose target entity is the Sales Order. Submitting it:

1. Extracts the submitted data against the Sales Order definition. A validation failure answers with the
   list of failed fields.
2. When there is no cart, answers with the error `No order found; please add a product to your cart.`
3. Writes the recognised fields on the order.
4. Logs the unrecognised fields as a message on the order, each line wrapped as a paragraph.
5. Attaches the uploaded files to the order.
6. Answers with the order identifier.

---

## 8. The delivery step

### 8.1 Which methods are offered

Every shipping method is read with elevated privileges and kept when it is published for this site,
evaluated through the site-aware publication search, and when it belongs to the order's company or to no
company. It is then kept only when it is available for the order.

Availability of one method for one order requires: the destination address matches the method's country,
state and postal-code prefix restrictions; the order's products satisfy the method's required tags and carry
none of its excluded tags; the order weight and volume are within the method's bounds. For a rule-based
method, availability additionally requires that the rate computation succeeds.

### 8.2 The preferred method

1. Start with the order's current method.
2. When the available list is not empty and the current method is not in it, take the delivery contact's
   preferred method when that one is available.
3. Otherwise take the first available method.

### 8.3 Setting a method

1. Remove the delivery line, and clear the pickup location unless it must be kept.
2. When no method is given, or the order has no deliverable products, stop.
3. Take the supplied rate, or compute it now.
4. When the rate succeeds, create the delivery line at the rated price.

With collection in store installed, switching away from a collect-in-store method recomputes the warehouse
and the fiscal position and, when the fiscal position changed, recomputes the taxes. With the promotion
capability installed, setting or removing a delivery method re-evaluates the programs and rewards, which is
what makes a free-shipping reward appear or disappear.

The selection endpoint refuses to change the method when the order already has a transaction whose state is
not draft, cancelled or in error, with the message of WS-383. It then returns the order summary values: a
success flag, whether the delivery amount is zero, whether the method invoices the real cost after shipping,
and the formatted delivery amount, untaxed amount, tax amount and total.

With the promotion capability the summary additionally carries the discounted delivery amount and the list
of formatted discount amounts, grouped per reward — one entry per reward for ordinary discounts and one
entry per line for gift-card and electronic-wallet lines. With the external relay network it carries the
relay brand, the package kind, the delivery postal code and country, the allowed countries and, when the
current delivery address is already a relay point, its reference.

### 8.4 Rating a method for display

The rate itself is produced by the shipping method. The tax treatment, the margin, the rounding and the
free-shipping override are [calculations.md](calculations.md) §17, with its worked example. In express mode
only the tax-included value is used, and only the partial address fields are required.

The rating endpoint returns the rate for one method without selecting it: it raises `Your cart is empty.`
when there is no cart, and
`It seems that a delivery method is not compatible with your address. Please refresh the page and try again.`
when the method is not in the available list; on success it adds the formatted amount, the free-delivery
flag and the invoice-after-shipping flag; on failure it returns a formatted zero amount together with the
failure message.

### 8.5 Pickup points

Specified in [storefront-stock-and-pickup.md](storefront-stock-and-pickup.md) §6.4 and §6.5. The two
endpoints return the close points for a postal code, resolving the country from the visitor's network
address or from the delivery address, and write the chosen point on the order.

---

## 9. Express checkout

Express checkout lets a shopper pay from the cart page with a wallet that supplies the addresses.

### 9.1 Values handed to the cart page

The standard payment values computed for the order, in express mode, plus:

| Value | Content |
|---|---|
| Payment access token | The order access token, under the name the wallet expects. |
| Minor amount | The order total excluding delivery, in minor currency units. |
| Merchant name | The site name. |
| Transaction address | The transaction endpoint of this order. |
| Express checkout address | The main express callback. |
| Landing address | The payment validation endpoint. |
| Generic payment method | The identifier of the payment method used when the wallet does not name one. |
| Shipping information required | True when the order has deliverable products. |
| Delivery amount | The order total minus the total excluding delivery, in minor currency units. |
| Delivery address update address | The delivery-address callback. |
| Customer | The value −1 when the visitor is the public user. |

No stored payment token is offered in express mode (WS-417). Express checkout is allowed by default; with
the print-on-demand capability, an order containing print-on-demand products allows it only when the
customer address is already complete, because that service requires a full address before the order can be
sent.

### 9.2 The delivery-address callback

The callback receives a partial address, typically the country, the state, the city and the postal code.

1. Resolve the country and the state from their codes and put the records in the payload.
2. Parse the payload into contact values.
3. When the cart is anonymous, create a contact named after the order — the name carries the order name,
   which is how a later call recognises the placeholder — and write it as the order's customer, while
   protecting the price list field from recomputation, because the shopper has already been shown an
   amount.
4. Otherwise, when the current delivery contact's name contains the order name, that is when it is a
   placeholder created by a previous call, write the new values on it.
5. Otherwise, when the payload differs from the current delivery address, look for an existing descendant of
   the commercial contact with the same address; use it, or create a new placeholder-named delivery contact.
6. Compute the available methods and their rates in express mode, skipping methods that require a pickup
   point, because express checkout cannot show a map, and, with collection in store installed, skipping
   collect-in-store methods.
7. Sort the methods by increasing price and answer with one entry per method: the method identifier under
   the key the wallet expects, the method name, the storefront description and the price in minor currency
   units.
8. Preselect the cheapest method on the order when it differs from the current one.

With the promotion capability, the answer additionally carries the delivery discount in minor units when a
free-shipping reward applies.

### 9.3 The tax recomputation callback

The callback recomputes the order taxes in express mode and returns the order total excluding delivery in
minor currency units. When the tax computation fails, for example because an external tax service refuses
the partial address, it returns the external tax error flag.

### 9.4 The main express callback

The callback receives a billing address, optionally a delivery address and optionally the chosen shipping
option.

1. Resolve the country and the state and parse the billing payload.
2. When the cart is anonymous, create the contact and write it as the order's customer, protecting the price
   list.
3. Otherwise, when the billing payload differs from the order's billing contact, reuse a descendant with the
   same address or create one, and write it as the billing contact.
4. Record the order identifier in the session's last-order key, because the express flow skips the pages
   that would normally do it.
5. When a delivery address was supplied, update the placeholder contact, or reuse a matching descendant, or
   create one, and write it as the delivery contact.
6. When a shipping option was supplied, set that method on the order when it is among the available ones.
7. Answer with the order's customer identifier.

Two addresses are considered the same when every key of the supplied payload equals the corresponding field
of the contact. The telephone number is not always part of the comparison, because some wallets do not
return it.

---

## 10. The payment step

### 10.1 The page

1. Resolve the cart and run the cart and address guards.
2. Recompute the cart: recompute the taxes, recompute the prices and, when a delivery method is set, rate it
   again while keeping the pickup location. With the promotion capability, the programs and rewards are
   refreshed and applied first.
3. Build the page values: the order; the blocking errors (§10.2); the billing contact; the submit label
   `Pay now`; and the standard payment form values for the order with this site, with the form's own submit
   button suppressed, the transaction address, the landing address and the order identifier, which lets a
   provider decide whether tokenisation is required.
4. When errors exist, the payment methods and the stored tokens are removed from the values, which hides the
   payment form.
5. Add the checkout step values and render.

### 10.2 Blocking errors

1. When the order has deliverable products and no delivery method is available, append the pair
   `Sorry, we are unable to ship your order.` and
   `No shipping method is available for your current order and shipping address. Please contact us for more information.`
2. With collection in store, when the order has deliverable products and the method is a collect-in-store
   method: when no pickup location is selected, append `Sorry, we are unable to ship your order.` and
   `Please choose a store to collect your order.`; otherwise, when some products are short at the selected
   store, append `Sorry, we are unable to ship your order.` and
   `Some products are not available in the selected store.`

### 10.3 Which providers and methods are offered

The compatible providers are computed by the payment folder and then narrowed:

1. Only providers with no site, or with this site (WS-414).
2. With collection in store, pay-on-site providers are removed unless the order's method is a
   collect-in-store method and the order contains at least one goods line (WS-415).
3. In express mode, stored tokens are not offered.

The availability report records, for each excluded provider, the reason: the incompatible-site reason, or
`no in-store delivery methods available`.

### 10.4 Creating the transaction

1. Check the access token; a failure raises `The access token is invalid.` A missing order raises the
   standard missing-record error.
2. Take a non-blocking exclusive lock on the order row. When the lock is unavailable, raise
   `Payment is already being processed.`
3. When the order is cancelled, raise `The order has been cancelled.`
4. Run the payment readiness check (§10.5).
5. Validate the transaction arguments; unexpected arguments are rejected.
6. Force the counterparty to the order's billing contact and the currency to the order currency, and link
   the order to the transaction.
7. When no amount was supplied, use the order total.
8. When the supplied amount differs from the order total at currency precision, raise
   `The cart has been updated. Please refresh the page.`
9. When the amount already paid equals the order total, raise
   `The cart has already been paid. Please refresh the page.`
10. For a token payment, delay the charge until after validation.
11. Create the transaction and record its identifier in the session, replacing any previous one.
12. Run the order-specific transaction validation (§10.6).
13. For a delayed token payment, charge the token now.
14. Return the provider processing values.

### 10.5 Payment readiness

1. When the cart is not ready, raise `Your cart is not ready to be paid, please verify previous steps.`
2. When the order is not services-only: when no delivery method is set, raise
   `No shipping method is selected.`; when the method is not among the available methods, raise
   `The delivery method is not compatible with your delivery address.`

Extensions, applied before the base rules:

| Capability | Additional rule | Message |
|---|---|---|
| Storefront stock | Every line must pass the availability check | The per-line stock warnings, joined by single spaces. |
| Collection in store | For a collect-in-store method with deliverable products, every storable product must be in stock at the selected store | `Some products are not available in the selected store.` |
| External relay network | A relay delivery address requires the relay method | `Point Relais® can only be used with the delivery method Mondial Relay.` |
| External relay network | The relay method requires a relay delivery address | `Delivery method Mondial Relay can only ship to Point Relais®.` |

### 10.6 Order-specific transaction validation

| Capability | Rule | Message on failure |
|---|---|---|
| Collection in store | A pay-on-site provider may only be used with a collect-in-store method | `You can only pay on site when selecting the pick up in store delivery method.` |
| Promotion | The programs and rewards are re-evaluated; when the order total changes as a result, the payment is refused | `Cannot process payment: applied reward was changed or has expired.\nPlease refresh the page and try again.` |

### 10.7 Returning from the payment provider

The validation endpoint may be called with an order identifier or without one.

1. When an order identifier was supplied, that order is taken and must equal the session's last order;
   otherwise the session cart is taken, or the session's last order when the cart key was already cleared.
2. When there is no order, redirect to the shop.
3. When the order is not yet confirmed, compute the blocking errors; when there are any, raise a validation
   error carrying the first error's title and message, joined by a line break.
4. Read the order's last portal transaction.
5. When the order has a non-zero total and there is no transaction, redirect to the shop.
6. When the order has a zero total, there is no transaction and the order is not yet confirmed, run the
   readiness check and confirm the order with message sending enabled.
7. Reset the storefront session.
8. When a transaction exists and it is still in draft, redirect to the shop.
9. Otherwise redirect to the confirmation page.

### 10.8 The confirmation page

The confirmation page reads the session's last-order key; without it the shopper is redirected to the shop.
It renders the order summary and an analytics payload carrying the order identifier, the company name, the
order total, the order tax total, the currency code, one entry per line that is not a delivery line — the
item identifier, which is the internal reference or the identifier, the item name, the item category, the
line unit price and the quantity — and the delivery line unit price when a delivery line exists.

With the course capability, the page additionally receives the enrolment records created for the buyer for
every course product of the order, which lets it link directly to the purchased courses.

The print address renders the order document of the session's last order as a printable document.

### 10.9 What confirmation does

Order confirmation is owned by [sales](../sales/README.md). The storefront adds:

* Before the standard confirmation, the salesperson is assigned with a forced recomputation, running as the
  platform system user, so that the assignment notification is authored by the system rather than by the
  shopper (WS-419).
* The confirmation message template is the site's own template when one is set, and otherwise the platform
  default.
* Wire transfer: the transaction stays pending, therefore the order only moves to the sent state. The
  shopper sees the payment instructions and the order appears in the unpaid list; a person confirms it after
  the funds arrive. No stock is reserved before that.
* Automatic invoicing: when the platform parameter is true, a completed transaction also creates and posts
  the invoice for its orders, even for a partial payment, and sends it, either immediately or through the
  deferred sending job when asynchronous messages are enabled.
* Pay on site: the transaction stays pending, but the order is confirmed anyway, which creates the transfer
  so that the store can prepare it.

---

## 11. The abandoned cart flow

### 11.1 Detection

The condition and its worked example are [calculations.md](calculations.md) §16.

### 11.2 The scheduled job

Runs hourly. For each site:

1. When the site does not send recovery messages, continue with the next site.
2. The candidates are the orders that are abandoned carts, have not been mailed, belong to this site and
   whose order date is at or after the site's recovery activation moment. The activation condition is what
   implements the rule that existing abandoned carts are not mailed when the feature is switched on.
3. When there are no candidates, continue with the next site.
4. Apply the eligibility filter (§11.3).
5. Mark every candidate that is not eligible as already mailed, so that it is never examined again.
6. For each eligible order, send the recovery template — addressing the message to the customer's formatted
   address when the template has no recipient configuration at all — and mark the order as mailed.

### 11.3 The eligibility filter

All the orders of one call share one site.

1. The threshold is the present moment minus the site's abandoned-cart delay.
2. The later orders are the confirmed orders of the same customers on the same site created at or after the
   threshold.
3. For each customer, remember the latest creation moment among the candidate carts.
4. A customer "has a later order" when, for at least one of their confirmed orders, that customer's latest
   candidate creation moment is at or before the confirmed order's order date.
5. Keep an order when its customer has an electronic mail address, no transaction of the order is in error,
   at least one line has a non-zero unit price, and the customer does not have a later order.

With the stock capability, an additional condition applies: no product of the order may be sold out. There
is no point inviting a shopper back to buy something that is gone.

The three business reasons behind the filter are: a shopper whose payment failed must be handled differently
from one who simply left; a cart of only free products is not worth a reminder; and a shopper who has
meanwhile bought must not be chased.

### 11.4 The recovery message

The shipped template is named `Ecommerce: Cart Recovery`, its entity is the Sales Order and its subject is
`You left items in your cart!`. The sender is the order salesperson's formatted address, else the company's,
else the current user's. It uses the default recipients of the record rather than an explicit recipient, and
it is not deleted after sending. The portal button inside the notification is relabelled `Resume Order` and
points at the revival address of §3.1.

### 11.5 The manual action

A salesperson may select carts and open the recovery composer. The action first ensures that every selected
order has a portal access token, then opens the message composer in mass-mailing mode when several orders
are selected and in single-message mode otherwise, pre-loaded with the recovery template resolved as
follows: the site's own template when every order belongs to the same site and that site names one,
otherwise the shipped template, otherwise no template.

The composer context carries the recovery flag, which makes the post-send step mark the orders as mailed:
after a mass mailing, every order of the batch that is still an abandoned cart and not yet mailed; after a
single message, the order unconditionally.

The programmatic variant, meant for automation rules, sends the site-specific template per order, ensuring
the access token first, and marks the orders as mailed.

### 11.6 Counters

A sales team that has at least one site reports the number and the summed total of the abandoned carts of
that team which have not been mailed. Teams without a site report zero. The formula is
[calculations.md](calculations.md) §28.1.

---

## 12. Reordering

### 12.1 From the portal

1. Resolve the order through the portal access check; on failure redirect to the portal home.
2. Keep the lines for which reordering is allowed: the line has a product, the product may be added to the
   cart, and the line is shown in the cart. Section and note lines, delivery lines, event tickets, promotion
   rewards and course lines are therefore skipped.
3. When nothing remains, raise `Nothing can be reordered in this order`.
4. Resolve or create the cart.
5. For each kept line, call the add operation with the line's quantity, its custom attribute values, its
   no-variant attribute values and, for a combo line, the full description of each combo item child: the
   template, the variant, the combination, the no-variant values, the custom values, the quantity, the combo
   item and the parent template.
6. Collect the order-level warnings of the adds that produced a zero quantity and write them, joined by line
   breaks, on the cart.
7. Return the accumulated analytics payload and the new cart quantity.

Reordering is allowed for an order when it is confirmed and at least one of its lines allows it.

### 12.2 From the cart

The quick reorder block of §3.2 adds one product at a time through the ordinary add endpoint.

---

## 13. Interaction with promotions

With the promotion capability installed:

* The programs applicable to a storefront order are those marked available on the site and either generic or
  attached to that site; the same substitution is applied to the trigger conditions.
* The program time zone is the site salesperson's time zone.
* A coupon code opened before the shopper had a cart is remembered in the session and applied as soon as a
  cart exists; when exactly one reward results and it is not a multi-product reward, it is applied
  immediately.
* Rewards are applied automatically after every cart update, except those the shopper explicitly removed —
  removing a reward line records the reward in the order's disabled list — and those belonging to nominative
  programs, multi-product rewards, and programs that carry more than one reward.
* Nominative programs are refused for anonymous visitors.
* Discount lines generated by one program for several tax groups are displayed as a single aggregate line.
* Reward lines never count towards the cart quantity, never appear in the reorder history, never show a
  strikethrough price, and only free-product rewards are treated as sellable.
* Free-shipping reward lines are excluded from the amount excluding delivery used by the free-shipping
  threshold.
* A cleanup job releases the coupons attached to storefront carts that have not been touched for the number
  of days given by the coupon validity parameter, whose default is 4, and then re-evaluates those carts.
* Applying a reward on an order whose method offers free shipping above a threshold rates the method again
  and rewrites or removes the delivery line.

---

## 14. Sequence summary of a complete purchase

| Step | Actor | Records written |
|---|---|---|
| 1. Browse and open a product | Visitor | A visit tracking entry carrying the viewed product. |
| 2. Add to the cart | Visitor | The Sales Order, the first time; one Sales Order Line per product and per linked product; the session keys. |
| 3. Review the cart | Visitor | Possibly line quantity changes, promotion reward lines, and the delivery line when a method was already chosen. |
| 4. Enter the address | Visitor | One or two Contact records; the order's customer, billing and delivery contacts; recomputed taxes and prices; possibly a newsletter subscriber. |
| 5. Choose a delivery method | Visitor | The order's shipping method; one delivery Sales Order Line; possibly the pickup location payload and the warehouse. |
| 6. Extra information | Visitor | Fields written on the order; a message on the order; attachments. |
| 7. Pay | Visitor | A Payment Transaction linked to the order; the session keys. |
| 8. The transaction succeeds | The provider callback | The order is confirmed; a salesperson is assigned; delivery work is created; the confirmation message is sent; optionally an invoice is created, posted and sent. |
| 9. Confirmation page | Visitor | Nothing; the session is reset. |
| 10. Fulfilment | Internal user | Owned by [inventory operations](../inventory-operations/README.md) and [accounts receivable](../accounts-receivable/README.md). |
