# Storefront availability, pickup and collection

How product availability is shown and enforced on the storefront, how back-in-stock notifications work, how
a warehouse is chosen, and how a shopper may collect an order in a store or at an external parcel shop.
Quantities themselves — the quantity free to use, the reservation, the transfers — are owned by
[inventory operations](../inventory-operations/README.md); this folder reads them and decides what the
shopper sees and may do.

---

## 1. The availability model

Three product settings drive everything.

| Setting | Where | Meaning |
|---|---|---|
| Tracked inventory | Product Template | Only tracked products have a quantity at all. A service or a good that is not tracked is never limited and never shows availability. |
| Sell when out of stock, default true | Product Template | When true, the storefront never limits the quantity and never declares the product sold out. |
| Show the remaining quantity and the availability threshold, defaults false and 5.0 | Product Template | Whether and when the remaining quantity is printed. |

Plus one site setting, the storefront warehouse, and one product text, the out-of-stock message.

### 1.1 The storefront quantity free to use

```formula
storefront quantity free to use = the quantity free to use of the product,
                                  evaluated with the site warehouse in context
```

With no warehouse on the site, the quantity free to use is the sum over every warehouse of the company.
This is the quantity used on the shop grid and on the product page, that is **before** checkout. During
checkout the cart-specific rule of §4.2 is used instead, because the chosen delivery method may pin a
warehouse.

With collection in store installed the rule becomes:

1. Start with the quantity free to use evaluated with the site warehouse in context.
2. When the site names a warehouse and has a published collect-in-store method, read the current cart, when
   a storefront request is in progress.
   1. When there is no cart, or the cart has no delivery method, the quantity becomes the greater of the
      quantity computed in step 1 and the largest quantity free to use of the product across the stores of
      the collect-in-store method.
   2. Otherwise, when the cart's method is a collect-in-store method and a pickup location is chosen, the
      quantity becomes the quantity free to use of the product in that store.

The first branch is what lets a product that is out of stock in the central warehouse still be offered,
because one shop has it. The second branch pins the availability to the store the shopper selected.

### 1.2 Sold out

```formula
a variant is sold out  = false                     when the product is not tracked
                       = false                     when out-of-stock ordering is allowed
                       = the storefront quantity free to use ≤ 0    otherwise
a template is sold out = false                     when the product is not tracked
                       = false                     when out-of-stock ordering is allowed
                       = true                      when the template has no variant
                       = whether its first variant is sold out      otherwise
```

A sold-out product is hidden from the quick-add action of the shop grid and of the product page, which
removes the add-to-cart button while leaving the product visible and browsable.

### 1.3 Kit availability

A product whose composition is a kit has no stock of its own: its availability is derived from its
components. The derivation is owned by [manufacturing](../manufacturing/README.md) and is reproduced, with
two worked examples, in [calculations.md](calculations.md) §18.1, because the storefront depends on it.

This is why a kit is the recommended composition for a storefront product that must map onto a single
tracked reference: the published product remains a catalogue entry while the reservation happens on the
components.

---

## 2. What the product page shows

The combination information payload carries the stock block only when the caller asked for quantities; the
variant information endpoint sets that flag and the plain price endpoints do not.

1. When the product is a combo, compute the maximum quantity of each of its combos (§2.2), keep the known
   values and report the smallest of them as the combo maximum.
2. When the product is not tracked, stop here.
3. Report the storable flag as true, the out-of-stock ordering flag and the availability threshold.
4. When a variant is resolved:
   1. Read the storefront quantity free to use.
   2. Convert it from the product unit to the requested unit, without rounding, then round it **down** to a
      whole number: the storefront never offers a fraction of a sellable unit.
   3. The cart quantity is zero when out-of-stock ordering is allowed, and otherwise the cart quantity of
      that variant converted from the product unit to the requested unit.
   4. Report the quantity free to use, the cart quantity, the requested unit's name and rounding precision,
      the availability display flag, the out-of-stock message, whether the current contact is subscribed to
      the back-in-stock notification of the product — or whether the anonymous session recorded a
      subscription for it — and the address held in the session for the subscription form.
5. When no variant is resolved, report a quantity free to use and a cart quantity of zero.

### 2.1 The availability line

| Situation | What the shopper sees |
|---|---|
| Not tracked, or out-of-stock ordering allowed | Nothing. |
| The availability display flag is false | Nothing, but the quantity input is still capped. |
| The flag is true and the quantity free to use minus the cart quantity is above the threshold | Nothing; the product is comfortably available. |
| The flag is true and the quantity free to use minus the cart quantity is above zero and at most the threshold | The remaining quantity, in the selected unit. |
| The quantity free to use minus the cart quantity is at most zero | The out-of-stock message when one is set, and the back-in-stock subscription form. |

The remaining quantity always subtracts what the shopper already put in the cart, which is why the cart
quantity is part of the payload.

### 2.2 Maximum quantities

```formula
maximum quantity of a variant = unknown        when the product is not tracked
                                               or out-of-stock ordering is allowed
maximum quantity of a variant = the storefront quantity free to use − the cart quantity of that variant
maximum quantity of a combo   = unknown        when at least one item of the combo has an unknown maximum
maximum quantity of a combo   = the largest maximum quantity among the items of the combo
maximum combo quantity of a product = the smallest maximum quantity among the combos of the product whose
                                      maximum is known; absent when none is known
```

The asymmetry is intentional: a combo can be sold as long as **one** of its choices is available, hence the
largest over items, while a product made of several combos needs **all** of them, hence the smallest over
combos.

---

## 3. Back-in-stock notifications

### 3.1 Subscribing

The subscription endpoint is public and takes an electronic mail address and a variant.

1. When the address does not match the address pattern, raise `Invalid Email`.
2. Load the variant. When it does not exist, or it is archived, or it cannot be sold, or it is not
   published, raise `This product is not eligible for stock notifications.`
3. Find or create the contact for that address.
4. When the visitor is the public user and that contact already has a user account, raise
   `Please sign in to proceed.` This prevents an anonymous visitor from subscribing somebody else's account
   address.
5. When the contact is not already subscribed to the variant, add it to the variant's subscriber list, with
   elevated privileges.
6. When the visitor is the public user, record the variant and the address in the session, which makes the
   page show the subscribed state on the next visit.

### 3.2 Notifying

An hourly job scans every variant that has at least one subscriber.

1. When the variant is still sold out, continue with the next variant: there is nothing to announce.
2. For each subscribed contact:
   1. Render the availability message body in that contact's language.
   2. Wrap it in the light notification layout with the entity description `Product`.
   3. Create and send a message whose subject is `The product '%(product_name)s' is now available`, with the
      product name in the contact's language; whose sender is the formatted address of the site company's
      contact, or, failing that, the site salesperson's formatted address; and whose recipient is the
      contact's formatted address.
   4. Remove the contact from the variant's subscriber list.

Sending failures do not abort the job. The subscription is single-shot: the contact is removed whether or
not the message was accepted (WS-405). The evaluation uses the current site, therefore the availability that
triggers a notification is the storefront availability of §1.1, not the raw warehouse quantity.

### 3.3 Wish list integration

A wish list row exposes a back-in-stock subscription flag, computed as "the row owner is subscribed to the
row's product". Writing it to true subscribes the owner; writing it to false does not unsubscribe. This is
what lets the wish list page offer a notification toggle per saved product.

---

## 4. Enforcing availability in the cart

### 4.1 The quantity cap

The stock capability overrides the quantity verification hook of
[storefront-checkout.md](storefront-checkout.md) §2.9.

1. When the product is not tracked or out-of-stock ordering is allowed, fall through to the base behaviour,
   which accepts the requested quantity.
2. Read the cart quantity and the quantity free to use of the product (§4.2).
3. Convert the cart quantity to the requested unit; convert the quantity free to use to the requested unit
   without rounding and then round it down to a whole number.
4. Compute the previous quantity of the line, which is zero for a new line; the added quantity; and the
   total quantity that would then be in the cart. The arithmetic is [calculations.md](calculations.md)
   §14.2.
5. When the available quantity is at least the total in the cart, accept the requested quantity with no
   warning.
6. Otherwise compute what this line may hold and produce the matching warning of WS-400: the
   existing-line warning when the allowed quantity is above zero and a line exists; the new-line warning
   when the allowed quantity is above zero and no line exists; the "became unavailable" warning when the
   allowed quantity is at most zero and a line exists; and the "not added" warning when no line could be
   created.

Quantities are printed as whole numbers when they are integral and with decimals otherwise. The allowed
quantity may be zero or negative; the caller then deletes the line, which is exactly what the third warning
announces.

### 4.2 The quantities used

```formula
cart quantity of a product        = the sum, over the order lines carrying that product, of the line
                                    quantity converted from the line unit to the product unit
quantity free to use of a product = the quantity free to use evaluated with the shop warehouse of the
                                    order in context
shop warehouse of an order        = the site warehouse
```

The cart quantity sums **every** line of the same product, combo item lines and lines added through
different units included, which is what prevents a shopper from exceeding the stock by splitting a product
across lines.

With collection in store installed, the shop warehouse of an order is the order's own warehouse when the
order's method is a collect-in-store method and the site warehouse otherwise; and the quantity free to use
becomes the largest quantity across the stores of the site's collect-in-store method when the site has a
warehouse, has a published collect-in-store method and the order has no delivery method yet.

### 4.3 Line checks before payment

1. When the product is not tracked or out-of-stock ordering is allowed, the line passes.
2. Read the cart quantity and the quantity free to use.
3. When the cart quantity is greater than the quantity free to use, write the warning
   `You ask for %(desired_qty)s %(product_name)s but only %(new_qty)s is available` on the line, with the
   available quantity floored at zero, and the line fails.
4. Otherwise the line passes.

```formula
maximum available quantity of a line = the smallest, over the priced lines of the line group — the line
                                       and its linked lines — of ( the quantity free to use − the cart
                                       quantity ), considering only tracked products whose out-of-stock
                                       ordering is disabled; unknown when there is none
maximum line quantity                = the line quantity + the maximum available quantity, or unknown
```

The payment readiness check collects the warning of every line that fails and raises them joined by single
spaces (WS-401).

### 4.4 Abandoned carts

A cart is not mailed when any of its products is sold out (§1.2). This is applied on top of the general
eligibility filter of [storefront-checkout.md](storefront-checkout.md) §11.3.

### 4.5 Ribbon, feed and structured description

* A ribbon whose assignment mode is the out-of-stock mode applies to a product whose template disallows
  out-of-stock ordering and whose variant is sold out.
* The product feed publishes the out-of-stock availability value for a sold-out variant and the in-stock
  value otherwise.
* The structured product description publishes the corresponding availability property for a tracked
  variant.

---

## 5. The warehouse of a storefront order

1. When the order has no site, the standard computation applies.
2. Otherwise, when the site names a warehouse, that warehouse is used; failing that, the standard
   computation applies.
3. When the result is still empty, the current user's default warehouse is used.

With collection in store, an order that already carries a pickup location keeps the warehouse named by that
location and is not recomputed. Selecting a method that is not a collect-in-store method after one that was
recomputes both the warehouse and the fiscal position and, when the fiscal position changed, recomputes the
taxes.

The site warehouse is also the default for new products through the storefront inventory defaults of
[configuration.md](configuration.md) §1.4.

---

## 6. Collection in store

Collection in store is a delivery method whose kind is the collect-in-store kind and whose stores are
warehouses. The shopper picks a store, the order is pinned to that store's warehouse, the taxes follow the
store's address, and the shopper may pay on site.

### 6.1 The method

A shipping method of the collect-in-store kind is forced to rate-only integration, cash on delivery
disabled, and no country, state or postal-code restriction. A created method receives every warehouse of its
company and is published when at least one was found. A published collect-in-store method must have at least
one store, and every store must share the method's company (WS-388, WS-389, WS-390).

Its rate is always the sales price of its delivery product, with no error and no warning; by default that
product is a free service named `Pick up in store`.

### 6.2 The store payload

Built by the warehouse operation of [entities.md](entities.md) §6.13: the identifier, the name, the street,
the city, the state code, the postal code, the country code, the latitude, the longitude and the opening
hours. A store whose address cannot produce a payload is silently omitted from the selector, which is why a
complete warehouse address is a functional prerequisite.

Geolocation is attempted once: when both coordinates are zero the address is geolocated; when the lookup
fails, the coordinates are set to 1000 and 1000, an impossible pair that marks the address as invalid and
prevents the platform from calling the geolocation service again on every page view.

### 6.3 Distance and ordering

Stores are ordered by increasing great-circle distance from the reference address. The formula and a worked
example are [calculations.md](calculations.md) §18.2. A store with the impossible coordinates therefore
sorts last, which is the intended consequence.

### 6.4 Fetching the stores

The pickup location endpoint is public and takes a postal code and optionally a product.

1. When a product was supplied, the call comes from a product page:
   1. When there is no cart, build a temporary, unsaved order carrying the collect-in-store method and ask
      it for the locations, without creating a cart (WS-393). Not creating a cart when the shopper is only
      inspecting availability keeps the abandoned-cart statistics meaningful.
   2. Otherwise, when the cart's method is not a collect-in-store method, write the collect-in-store method
      and its delivery line on the cart.
2. The country is the visitor's geolocated country, or, failing that, the delivery address country.
3. When a postal code was supplied but no country could be resolved, the postal code is dropped and the
   delivery address is used instead.
4. The reference is an unsaved contact carrying the country and the postal code when both are known, and the
   order's delivery address otherwise.
5. Ask the method for its close locations for that reference.
6. When the method has no close-locations operation, or it returns nothing, answer with the error
   `No pick-up points are available for this delivery address.`
7. Otherwise answer with the locations.

Each returned location carries an additional block: when the call comes from a product page, the formatted
stock of that product in that store; when it comes from the checkout, whether every storable product of the
cart is available in that store.

**Formatted stock of one product in one store.**

```formula
in stock       = the quantity free to use in that store > 0
show quantity  = the availability display flag is true
             AND in stock
             AND the availability threshold ≥ the quantity free to use
reported in stock = in stock OR out-of-stock ordering is allowed
```

The block is empty when the record is not a variant.

### 6.5 Choosing a store

Two endpoints write the choice. The one called from the product page resolves or creates the cart, writes
the collect-in-store method and its delivery line when the cart does not already carry a collect-in-store
method, and then sets the pickup location. The one called from the checkout page sets the pickup location on
the current cart after a method has been selected.

Setting a pickup location:

1. Base behaviour: when the current method declares that it uses locations, write the parsed payload on the
   order, or an empty value when the payload is empty.
2. With collection in store: when the method is not a collect-in-store method, stop.
3. Remember the current fiscal position.
4. Write the parsed payload.
5. When the payload is not empty, set the order warehouse to the store named by the payload and recompute
   the fiscal position; otherwise recompute the warehouse.
6. When the fiscal position changed, recompute the taxes.

The fiscal position of an order collected in store is resolved for the customer using the **store address**
as the delivery address, which is what makes a purchase collected in another region carry that region's
taxes.

### 6.6 Defaults and the checkout page

When a collect-in-store method has exactly one store and it is not already the order's method, the checkout
page receives a default location for it, together with the shortage data for that store. This lets the page
preselect the only possible store.

The checkout page of an order that is not services-only also receives, for the currently selected location,
the shortage mapping.

**The shortage mapping.**

1. Start with an empty result.
2. For each product of the order's lines, grouped per product:
   1. Skip the product when it is not tracked or when out-of-stock ordering is allowed.
   2. Read its quantity free to use in that store.
   3. For each line carrying that product, in line order:
      1. Convert the remaining quantity to the line unit, floor it and floor it again at zero.
      2. When the line quantity is greater than that value, record the pair in the result and write on the
         line the warning `%(available_qty)s/%(line_qty)s available at this location`.
      3. Subtract the line quantity, converted to the product unit, from the remaining quantity.
3. The order is in stock at that store when the result is empty.

The running subtraction matters when the same product appears on several lines: the second line is evaluated
against what the first one left.

### 6.7 Payment on site

The shipped pay-on-site provider is a custom provider whose mode is the pay-on-site value, published,
enabled, carrying the payment method `Pay on site`, which supports no tokenisation, no express checkout, no
manual capture and no refund. It is offered only when the order's method is a collect-in-store method and
the order contains at least one goods line; otherwise it is filtered out with the reason
`no in-store delivery methods available`. Attempting to pay on site without a collect-in-store method is
refused with `You can only pay on site when selecting the pick up in store delivery method.`

A pay-on-site transaction stays pending; the order is nevertheless confirmed at that point, which creates
the transfer so that the store can prepare the goods.

### 6.8 Exclusions

* Collect-in-store methods are never offered in express checkout, because the express flow cannot present a
  store selector.
* A product carrying one of the method's excluded tags does not show the collection block on its page.
* Services are never eligible: the block is shown only for tracked variants.

### 6.9 What the product page shows

When the site has a published collect-in-store method, the product is a tracked variant and none of the
method's excluded tags is on the product, the payload reports that the collection availability block must be
shown, plus two blocks:

* the delivery stock block: the formatted stock of the product in the site warehouse, when at least one
  published method that is not a collect-in-store method exists for this site; an empty block otherwise;
* the collection stock block: the formatted stock of the product in the selected store, when the cart has a
  collect-in-store method and a selected location; otherwise the formatted stock computed from the best
  quantity free to use across the stores of the collect-in-store method.

The page then shows two lines, one for delivery and one for collection at a named store, each with a
quantity when the threshold rule allows it.

---

## 7. External parcel-shop pickup

The external parcel-shop network is integrated as its own delivery method with its own relay-point
selector.

### 7.1 Choosing a relay point

The relay endpoint is public and receives the relay payload: the relay identifier, the name, two address
lines, the postal code, the city and the country.

1. Resolve the cart. When it is still anonymous, refuse with
   `Customer of the order cannot be the public user at this step.`
2. When the current method restricts countries and the relay's country is not among them, refuse with
   `%s is not allowed for this delivery carrier.`
3. Find or create the relay contact under the order's customer, carrying the relay identifier, the name, the
   street, the second street line, the postal code, the city, the lower-cased country code and the
   customer's telephone number.
4. Write it as the order's delivery address when it differs from the current one.
5. Answer with the re-rendered address block and the new delivery contact.

### 7.2 Rules

* A relay contact may never be edited from the address form: the attempt raises
  `You cannot edit the address of a Point Relais®.`
* The completeness check of a relay address always succeeds, because the shopper cannot correct it.
* Payment is refused when a relay delivery address is used without the relay method
  (`Point Relais® can only be used with the delivery method Mondial Relay.`) and when the relay method is
  used without a relay delivery address
  (`Delivery method Mondial Relay can only ship to Point Relais®.`).
* When the delivery address is recomputed and it is a relay contact while the chosen method is not the relay
  method, the delivery address falls back to the customer's own address.
* The order summary returned after selecting the relay method carries the brand, the package kind, the
  delivery postal code and country, the list of allowed country codes and, when a relay is already chosen,
  its reference.

---

## 8. Configuration defaults

| Setting | Applies to | Default | Effect |
|---|---|---|---|
| Continue selling when out of stock | new Product Templates | true | The default out-of-stock ordering flag. |
| Show available quantity | new Product Templates | false | The default availability display flag. |
| Show threshold | new Product Templates | 5.0 | The default availability threshold. |
| Warehouse | the Website | none | The warehouse used for storefront availability and assigned to carts. Restricted to warehouses of the site's company. |

Changing a default does not change existing products; it applies only to products created afterwards.
