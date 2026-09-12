# Workflows

The end-to-end operational procedures of the Delivery and Shipping domain. Each procedure is
written as a numbered sequence of steps; each step states what it reads, what it creates or
changes, which named operation it invokes and how it can fail. Failures reference the rule
identifiers of [business-rules.md](business-rules.md); arithmetic references the sections of
[calculations.md](calculations.md).

| # | Workflow |
|---|---|
| 1 | Configuring a Delivery Method |
| 2 | Adding a shipping charge to a quotation from the back office |
| 3 | Updating a shipping charge after the basket changed |
| 4 | Removing a shipping charge |
| 5 | Choosing a delivery method in the storefront |
| 6 | Choosing a collection point and confirming the order |
| 7 | Confirming an order and creating the transfers |
| 8 | Packing goods for a carrier |
| 9 | Validating an outgoing transfer and sending the shipment |
| 10 | Writing the real carriage charge back onto the order |
| 11 | Cancelling a shipment |
| 12 | Printing and publishing a return label |
| 13 | Following a shipment from the customer portal |
| 14 | Collecting from a store |
| 15 | Paying cash on delivery |
| 16 | Paying on site |
| 17 | Batching outgoing transfers by carrier |
| 18 | Choosing a parcel point of an external network |
| 19 | Invoicing the carriage |
| 20 | Adding a new carrier integration |

---

## 1. Configuring a Delivery Method

**Actor.** A sales administrator or an inventory administrator.

**Preconditions.** A service product exists, or will be created from the form.

1. Open Delivery Methods from the sales configuration menu, from the inventory configuration menu
   or from the storefront configuration menu, and create a record.
2. Type the commercial name. It is required and translatable (DSH-002).
3. Choose the provider kind. The default is the fixed kind. Choosing the in-store kind at creation
   forces the integration level to `rate`, clears the cash-on-delivery flag, clears the three
   destination filters, attaches every warehouse of the resolved company as a store and publishes
   the method when at least one warehouse was found (state machine 5, transition K1).
4. Choose the delivery product. It is required (DSH-003). Creating it from the field pre-fills it
   as a service, not sellable, not purchasable and invoiced on ordered quantities. The product's
   company becomes the method's company and the product's currency becomes the method's currency.
5. For the fixed kind, type the charge. Writing it writes the delivery product's sales price.
6. For the rule-based kind, open the pricing page and create Delivery Price Rules. Each rule needs
   a condition variable, an operator, a comparison value, a base amount, a factor amount and a
   factor variable; all six are required (DSH-018 to DSH-020). Order them with the drag handle.
7. For a carrier integration, fill the credentials the integration declares, choose the integration
   level and, at level `rate_and_ship`, the invoicing policy.
8. Optionally set the margins. They are ignored for the fixed kind (DSH-023). The proportional
   margin may not go below −1 (DSH-004).
9. Optionally set the waiver: tick the flag and type the threshold, which is expressed in the
   company currency. The controls are hidden for the rule-based kind, whose charges are waived with
   a Delivery Price Rule instead.
10. Open the availability page and set the destination filters and the content filters. A region
    can only be chosen once a country is chosen, and so can a postal-code prefix; the screen shows
    "Please select a country before choosing a state or a zip prefix." while no country is chosen.
    Clearing every country clears every postal-code prefix.
11. A tag may not be listed both as required and as excluded; the write is refused with the message
    of DSH-001.
12. Optionally type the customer-facing description. It is printed at the foot of the quotation and
    repeated in the order confirmation message.
13. Optionally attach procurement routes. Only routes flagged as applicable to shipping methods can
    be chosen (DSH-044).
14. For the storefront, publish the method. An in-store method with no store cannot be published
    (DSH-041).
15. Save. The method becomes available wherever it passes the availability filter.

**Failure conditions.** Every one of DSH-001 to DSH-009 and DSH-041 to DSH-044 can refuse the save.

---

## 2. Adding a shipping charge to a quotation from the back office

**Actor.** A salesperson.

**Preconditions.** A quotation with at least one line, not all of them services.

1. The "Add shipping" button is shown only when the order has lines, they are not all services and
   the order carries no shipping charge line yet.
2. Pressing it opens the selection wizard. Three values are pre-filled:
   - the order;
   - the default method: the delivery address's own default method when it is active, failing that
     the commercial entity's default method when it is active, and in either case only when that
     method is in the order's available list; otherwise nothing;
   - the total weight: the order's estimated weight (calculations section 8.1).
3. The wizard computes the available methods: every method of the order's company that passes the
   availability filter for the order's delivery address and the order's content (calculations
   section 1). When the order has no customer, every method of the company is offered.
4. Choosing a method or changing the weight requests a rate for the two core kinds:
   - the pipeline of calculations section 7 runs;
   - on success the charge to apply and the cost to display are stored on the wizard, and the
     warning, if any, is shown in an information banner;
   - on failure a blocking dialogue carries the engine's own message: "Error: this delivery method
     is not available for this address." for a destination mismatch, "Not available for current
     order" when no Delivery Price Rule matches, "Error: this delivery method is not available."
     when the method's kind has no rating routine.
5. For every other kind both amounts are set to zero and the user presses "Get rate", which runs
   the same pipeline and re-opens the wizard. A failure is raised as an error carrying the same
   messages.
6. When the method's invoicing policy is `real`, the wizard shows the warning "The shipping price
   will be set once the delivery is done."
7. Pressing "Add" confirms. When the displayed cost is zero and the kind is neither fixed nor
   rule-based, the button first asks "Are you sure you want the delivery to be free for this order?
   You might have forgotten to compute the rates."
8. Confirmation performs, in this order:
   1. Remove the existing shipping charge lines (DSH-026).
   2. Write the Delivery Method onto the order.
   3. Create one Sales Order Line:
      - description: the method's name, read in the customer's language; when the delivery product
        carries a sales description, the method's name, a colon, a space and that description;
      - unit price: the charge to apply;
      - quantity: one;
      - product: the delivery product;
      - unit of measure: the delivery product's reference unit;
      - taxes: the delivery product's sale taxes restricted to the order's company, mapped through
        the order's fiscal position when the order carries one and a customer;
      - the shipping-charge flag;
      - sequence: one more than the last line's sequence, when the order has lines;
      - when the method's waiver flag is set and the charge is zero in the order currency, the
        description is extended with a line break and the reproduced text "Free Shipping";
      - when the method's invoicing policy is `real`, the price is then forced to zero and the
        description is extended with " (Estimated Cost: <charge formatted in the order currency>)".
   4. Clear the order's recomputation flag and copy the wizard's delivery message onto the order.
   5. When the order is already confirmed, write the Delivery Method onto every transfer of the
      order that is neither done nor cancelled and that contains no move returning another move.
9. The charge line appears at the foot of the order and is included in every total.

**Failure conditions.** DSH-026 refuses the replacement when every existing charge line has been
invoiced. DSH-017 refuses a confirmation without a method. The parcel-point network refuses a
confirmation without a chosen point (DSH-046).

---

## 3. Updating a shipping charge after the basket changed

1. Changing a line, the customer or the delivery address on an order that already carries a
   shipping charge line sets the order's recomputation flag.
2. The charge line is highlighted in amber in the list and the "Update shipping cost" button turns
   amber.
3. Pressing the button opens the same wizard with the order's current method pre-selected and the
   title "Update shipping cost".
4. The rate is requested again with the current basket and the current weight.
5. Pressing "Update" runs the same confirmation as workflow 2 step 8, which removes the old charge
   line and creates a new one.

The flag is only an invitation: nothing prevents confirming an order whose charge is out of date.

---

## 4. Removing a shipping charge

1. Delete the shipping charge line from the order's line list. A shipping charge line may be
   deleted even from a confirmed order, unlike an ordinary line.
2. Deleting the last shipping charge line clears the order's Delivery Method.
3. In the storefront the collection point stored on the order is also cleared, unless the caller
   asked to keep it — which the storefront does when it is only re-pricing the cart.
4. A charge line that has been invoiced cannot be deleted through the replacement path; the
   deletion itself is governed by the order's own rules, owned by [`../sales/`](../sales/).

---

## 5. Choosing a delivery method in the storefront

**Actor.** A shopper, signed in or anonymous.

1. The checkout page asks for the delivery form. The available methods are every published method
   compatible with the order's company that passes the availability filter, and, for the rule-based
   kind, whose rating succeeds.
2. Each method is rendered with its name, its online description and a rate. The rate is the
   pipeline of calculations section 7 followed by a tax adjustment owned by
   [`../website-and-storefront/`](../website-and-storefront/): the delivery product's taxes of the
   order's company are mapped through the fiscal position and applied to the charge; the website
   setting decides whether the tax-excluded or the tax-inclusive amount is shown.
3. The preferred method is chosen as follows: the method already on the order when it is still
   compatible; failing that the delivery address's default method when it is compatible; failing
   that the first compatible method in sequence order.
4. Selecting a method calls the storefront's set-method operation, which:
   1. refuses with "It seems that there is already a transaction for your order; you can't change
      the delivery method anymore." when the order has a payment attempt that is not draft,
      cancelled or in error;
   2. removes the existing shipping charge lines and the stored collection point;
   3. requests a rate and, on success, writes the Delivery Method and the charge line;
   4. returns the recomputed order totals, together with a flag saying whether the charge is zero
      and a flag saying whether the final charge will only be known after the delivery.
5. Asking for a single method's rate outside the list refuses with "It seems that a delivery method
   is not compatible with your address. Please refresh the page and try again." when the method is
   not in the available list, and with "Your cart is empty." when there is no cart.
6. A method that offers collection points shows the selector; see workflow 6.
7. Express checkout lists the methods sorted by ascending charge, excludes the in-store kind and
   excludes every method that offers collection points, and pre-selects the cheapest.

---

## 6. Choosing a collection point and confirming the order

**Actor.** A shopper or a salesperson.

1. The selector is offered when the order's Delivery Method declares a pickup-location flag named
   after its provider kind and that flag is true.
2. The selector asks for the close collection points, passing a postal code. The country is taken
   from the geolocation of the request when it is known, and from the order's delivery address
   otherwise.
3. The order builds the address to search around: a transient address carrying the given postal code
   and country when a postal code was given, and the order's delivery address otherwise. A postal
   code without a country is a programming error and is refused before the search; the
   collection-in-store package guards against it by resolving the country from the stored collection
   point or from the geolocation, and by dropping the postal code when neither yields a country.
4. The order looks for the routine named after the method's provider kind. When the method declares
   none, the answer is the error "No pick-up points are available for this delivery address."
5. The routine returns a list of collection point records. An empty list produces the same error. A
   refusal raised by the routine is returned as an error carrying the routine's own text.
6. The dialogue shows the points as a list and on a map, sorted by the routine. Its title is
   "Pickup Location" when exactly one point came back and "Choose a pick-up point" otherwise. Each
   point shows its name, its street, its postal code and city, and its opening hours under the
   heading "Opening hours", with "Closed" for a day that has no range. When no point matches, the
   dialogue shows "No result"; while the request is running it shows "Loading...".
7. The first point is preselected, unless the point already chosen is still in the list.
8. Pressing "Choose this location" stores the point on the order through the set-collection-point
   operation. For the in-store kind the operation also sets the order's warehouse to the chosen
   store, recomputes the fiscal position from the store's address and recomputes every tax when the
   fiscal position changed.
9. At confirmation the stored point becomes a delivery address (state machine 3, transition P5):
   1. read the name, street, city, postal code, country code and region code from the stored
      record;
   2. resolve the country by its code and the region by its code within that country;
   3. search for a child address of the order's current delivery address with the same street,
      city, region and country and of kind delivery;
   4. reuse it when found, otherwise create one carrying the point's name — or the delivery
      address's name when the point has none — the point's street, city, postal code, region and
      country, the delivery address's electronic mail address and telephone number, and the
      pickup-point flag;
   5. write it as the order's delivery address.
10. Because the created address carries the pickup-point flag, it is excluded from the customer's
    selectable delivery addresses and is never chosen as the default delivery address of a later
    order.

---

## 7. Confirming an order and creating the transfers

1. The order is confirmed. Everything owned by [`../sales/`](../sales/) runs unchanged.
2. Before it runs, the collection point is turned into a delivery address (workflow 6 step 9).
3. For the parcel-point network the confirmation is refused when the method and the delivery
   address disagree about being parcel-point records (DSH-047).
4. Procurement is launched for every line. When the line names no route of its own and the order's
   Delivery Method names routes, those routes are used (DSH-044).
5. Moves are grouped into transfers. The grouping key includes the Delivery Method of the move's
   sales order, so two orders carried differently never share a transfer.
6. Each new transfer receives a Delivery Method and a tracking reference by the propagation rule of
   calculations section 13.1, when at least one of its moves' rules carries the propagation flag.
7. The outgoing transfer created directly from the order carries the order's Delivery Method.

---

## 8. Packing goods for a carrier

**Actor.** A warehouse operator.

1. On a transfer that carries a Delivery Method, the packing action always opens the packing
   dialogue, because a carrier is present.
2. Before the dialogue opens, the carrier kind is derived from the lines being packed
   (calculations section 12.4). When the lines name more than one Delivery Method, or any line
   names none, the packing is refused with the message of DSH-034.
3. The dialogue proposes:
   - the container types whose carrier kind matches the derived kind;
   - the existing packages whose carrier kind and container type match and whose location is the
     destination of the lines, or which are already among the lines' source packages, or which have
     no location and no move line, or whose move lines end at the same destination;
   - a shipping weight computed by calculations section 12.2.
4. Changing the container type, the existing package or the weight may raise the too-heavy warning
   of calculations section 12.3. The warning does not block.
5. Confirming the dialogue creates or reuses the package, assigns the move lines to it and writes
   the typed shipping weight onto it.
6. When the packing is performed without the dialogue — from a scanning client, for instance — the
   weight is not supplied, so the package's shipping weight is set to the weight the package would
   compute for this transfer.
7. The transfer's shipping weight is recomputed from the packages and the loose goods
   (calculations section 8.6).

---

## 9. Validating an outgoing transfer and sending the shipment

**Actor.** A warehouse operator.

1. The operator validates one or several transfers. Everything owned by
   [`../inventory-operations/`](../inventory-operations/) runs first: reservation checks, quantity
   checks, backorder handling, the writing of the stock moves.
2. When that succeeds, the carrier propagation of calculations section 13.2 runs for every
   validated transfer that carries a Delivery Method.
3. The confirmation-message step then runs, transfer by transfer, and it is where the shipment is
   created. For each transfer:
   1. Evaluate the six guards of state machine 1, transition T6. When any of them fails, skip to
      step 3.3.
   2. Send the shipment:
      1. Call the sending routine named after the method's provider kind, with this one transfer.
         It returns one result per transfer; the first result is used. The result carries the exact
         charge and the tracking reference.
      2. Apply the shipment-time waiver of calculations section 7.7.
      3. Apply the margins of calculations section 7.1 step 5 and store the result as the
         transfer's shipping cost.
      4. When the result carries a tracking reference, propagate it: build the set made of this
         transfer — unless its existing reference already contains the returned one — plus every
         transfer reachable by following the origin moves transitively, plus every transfer
         reachable by following the destination moves transitively. Every transfer of that set with
         no reference receives the returned one; every transfer of that set that already has one
         receives a comma and the returned one appended.
      5. Post a message on the transfer: "Shipment sent to carrier <method name> for shipping with
         tracking number <the transfer's tracking reference>", a line break, then "Cost:
         <shipping cost with two decimals> <currency name>", where the currency is the order's
         currency, or the company's currency when the transfer has no order.
      6. Write the real charge back onto the order (workflow 10).
   3. Run the compliance hook. It does nothing in the core; a carrier integration uses it to refuse
      a transfer whose data the carrier will not accept — a missing weight, a missing customs code,
      a missing telephone number.
   4. Remember that a carrier transfer has been processed.
   5. When step 3.2 or step 3.3 refuses:
      - and no carrier transfer of this batch has been processed yet, re-raise the refusal. The
        whole validation is undone and the operator sees the carrier's message.
      - and at least one carrier transfer has already been processed, do **not** re-raise. Post the
        refusal text as a message on the transfer, and schedule a warning activity dated today,
        assigned to the transfer's responsible user or, when it has none, to the validating user.
        The activity's note reads "Exception occurred with respect to carrier on the transfer
        <a link to the transfer>. Manual actions might be needed." followed by "Exception:
        <the refusal text>".

   The asymmetry exists because the carrier's own computer system has already accepted the earlier
   transfers of the batch; undoing the local transaction would leave those shipments booked with
   the carrier and cancelled in the system.
4. The ordinary confirmation message of the transfer is then sent.
5. A transfer that was not sent automatically — because the operation type does not print labels,
   or because the reference already existed — can be sent later with the "Send to Shipper" button,
   which is offered on a done outgoing transfer whose kind is neither `fixed` nor `base_on_rule`,
   which carries no reference yet and which is not an in-store method.

---

## 10. Writing the real carriage charge back onto the order

This runs at the end of every successful shipment.

1. Stop unless the transfer has a Sales Order, the method's invoicing policy is `real` and the
   shipping cost stored on the transfer is not zero.
2. Look for a matching charge line on the order: a shipping charge line whose price is zero in the
   order currency and whose product is the method's delivery product.
3. When none is found, create one with the transfer's shipping cost, which — because the policy is
   `real` — is written at zero and given the bracketed estimate in its description.
4. Overwrite the first matching line with:
   - price: the transfer's shipping cost;
   - description: the method's name alone, read in the destination contact's language, so that the
     bracketed estimate disappears.
5. The write is performed with the flag that removes the price and the description from the
   protected fields of a shipping charge line, so it succeeds even on a locked order. An ordinary
   write of the same fields on the same line is still refused.

**Backorders.** The first shipment consumes the zero-priced line. When the backorder is validated,
no line matches any more — the first line now carries a price — so step 3 creates a second charge
line for the backorder's own cost. An order shipped in three parts therefore carries three charge
lines.

---

## 11. Cancelling a shipment

**Actor.** A warehouse operator.

1. The "Cancel" button appears beside the tracking reference on a done transfer whose provider kind
   is set and is neither `fixed` nor `base_on_rule`, and which carries a tracking reference.
2. Pressing it asks "Cancelling a delivery may not be undoable. Are you sure you want to continue?"
3. On confirmation, the cancellation routine named after the provider kind is called with the
   transfer.
4. A message "Shipment <tracking reference> cancelled" is posted on the transfer.
5. The tracking reference is cleared. The shipping cost, the labels and the charge line on the
   order are **not** undone; a seller who must also refund the carriage does so on the order.
6. Calling the operation on a fixed or rule-based method fails without a user-facing message,
   because those two kinds declare the routine but leave it unimplemented (DSH-056).

---

## 12. Printing and publishing a return label

1. A return label is produced automatically at shipment time when the method's automatic-return
   flag is set. The return-label routine named after the provider kind is called with the outgoing
   transfer.
2. It can also be produced on demand: the "Print Return Label" button appears on an incoming
   transfer that is a return — the method supports returns and at least one move both originates in
   a returned move and ends in an internal location — and that is not yet done.
3. In both cases the routine receives the transfer, an optional tracking number and an optional
   original date, and attaches one or more documents whose names begin with `LabelReturn-` followed
   by the provider kind.
4. When the method's portal flag is set, an access token is generated on each of those attachments.
5. The customer sees a "Print Return Label" link on the order's portal page for every transfer
   whose method publishes return labels and which carries at least one return-label attachment.
6. Two further naming prefixes are reserved for the outgoing direction, so that a carrier
   integration can distinguish its documents: `LabelShipping-` followed by the provider kind for a
   shipping label, and `ShippingDoc-` followed by the provider kind for an accompanying document
   such as a commercial invoice.

---

## 13. Following a shipment from the customer portal

1. The customer opens the order's portal page. Each transfer of the order is listed with its
   delivery details.
2. When the transfer carries a tracking reference, a tracking block is added:
   - when the stored tracking link parses as a list of label-and-link pairs, every pair is rendered
     as a link, separated by a plus sign;
   - when it does not parse but a tracking link exists, the reference is rendered as one link;
   - when no tracking link exists, the method's name and the reference are shown as plain text.
3. When the method publishes return labels and the transfer carries at least one, a "Print Return
   Label" link is added, pointing at the first return-label attachment with its access token.
4. In the back office the "Tracking" button opens the same links. When the stored value is a list,
   the links are posted in the transfer's history under "Tracking links for shipment:" and an
   informational dialogue explains "You have multiple tracker links, they are available in the
   chatter." When the transfer has no tracking link at all, the button refuses with "Your delivery
   method has no redirect on courier provider's website to track this order."

---

## 14. Collecting from a store

**Actor.** A shopper.

**Preconditions.** The website names a warehouse and a published in-store Delivery Method exists
for that website and company.

1. **On the product page.** The availability widget is shown when collection in store is enabled,
   the record shown is a variant, the product is storable and the product carries none of the
   in-store method's excluded tags. It shows two figures: the stock for ordinary delivery, taken
   from the website's warehouse, and the stock for collection, taken from the chosen store when one
   is chosen and from the best store otherwise. When the in-store method has exactly one store, the
   store is shown and no "select store" button is offered.
2. Pressing the store button opens the collection point selector in product mode. Confirming a
   store calls the product-page route, which creates a cart when there is none, sets the in-store
   method with the delivery product's sales price when the cart carries another method, and stores
   the chosen store.
3. **On the checkout page.** The in-store method is listed among the delivery methods, marked with
   a location icon rather than a lorry icon. When it has exactly one store, that store is proposed
   as the default collection point together with the stock check for that store.
4. Selecting the in-store method reloads the page, because the taxes, the warehouse and the address
   labels all change. The address section is retitled "Contact Details" and the option to reuse the
   delivery address for billing is retitled "Same as contact details".
5. When the chosen store cannot supply every line, a warning block lists each short line with its
   picture, its name, its variant attributes, its unit when the product has several, and the
   sentence "<available>/<ordered> available at this location". The block is headed "Some of the
   products are not available at <store name>." Each line offers either "Update cart with available
   stock" or, when nothing is available, "Remove". Pressing either updates the cart and reloads the
   page.
6. The checkout cannot proceed while the warning block is present.
7. Reaching the payment step is refused when no store was chosen — "Sorry, we are unable to ship
   your order." and "Please choose a store to collect your order." — or when the chosen store is
   short — "Sorry, we are unable to ship your order." and "Some products are not available in the
   selected store."
8. The final check immediately before payment repeats the stock test and refuses with "Some
   products are not available in the selected store."
9. On confirmation the store becomes the order's warehouse and a delivery address carrying the
   pickup-point flag is created from the store's address (workflow 6 step 9). The transfer is
   prepared in that store and the customer is notified when the transfer is validated.

---

## 15. Paying cash on delivery

1. The seller enables the cash-on-delivery provider, which is shipped disabled in demonstration
   data and enabled otherwise, and ticks the cash-on-delivery flag on the Delivery Methods that
   allow it.
2. At the payment step the compatible providers are filtered: when the order's Delivery Method does
   not allow cash on delivery, every custom provider whose mode is `cash_on_delivery` is removed,
   and the availability report records them as unavailable with the reason "cash on delivery not
   allowed by selected delivery method".
3. The payment form treats the payment method whose code is `cash_on_delivery` as a pay-later
   method, so the submit button carries the pay-later wording instead of the pay-now wording.
4. Choosing it creates a transaction in the pending state and shows the provider's pending message:
   "The delivery staff will collect payment upon delivery."
5. The post-processing of a pending transaction whose provider mode is `cash_on_delivery` confirms
   every draft order of that transaction, with the confirmation message enabled, which creates the
   transfers.
6. The money is collected by the carrier and reconciled outside this domain; see
   [`../payments-and-bank-reconciliation/`](../payments-and-bank-reconciliation/).

---

## 16. Paying on site

1. The seller enables the pay-on-site provider, which is shipped enabled and published and is
   disabled by the demonstration data.
2. At the payment step the compatible providers are filtered: unless the order's Delivery Method is
   of the in-store kind **and** the order carries at least one line whose product is goods, every
   custom provider whose mode is `on_site` is removed, and the availability report records them as
   unavailable with the reason "no in-store delivery methods available".
3. The payment form treats the payment method whose code is `pay_on_site` as a pay-later method.
4. Choosing it creates a transaction in the pending state and shows the provider's pending message:
   "Your order has been confirmed." followed by "Please come to the store to pay for your
   products."
5. Validating the transaction against the order refuses with "You can only pay on site when
   selecting the pick up in store delivery method." when the order's method is not of the in-store
   kind. The check exists to defeat a forged request.
6. The post-processing of a pending transaction whose provider mode is `on_site` confirms every
   draft order of that transaction, with the confirmation message enabled.
7. The payment-confirmation page replaces the order reference block with the method's name, the
   annotation "(In-store pickup)" and the method's online description.

---

## 17. Batching outgoing transfers by carrier

1. The seller ticks automatic batching on the outgoing operation type, ticks the carrier grouping
   and optionally types a maximum weight.
2. When a transfer becomes ready, the automatic batching looks for a batch to join or another
   transfer to pair with.
3. The candidate transfers are narrowed to those carrying the same Delivery Method; *no method* is
   itself a value that must match.
4. The candidate batches are narrowed to batches at least one of whose transfers carries the same
   Delivery Method.
5. The three weight guards of calculations section 14 are applied.
6. The automatic batch's description is extended with the Delivery Method's name.
7. Packing across the transfers of one batch is allowed only when they all carry the same method
   (DSH-034); the weight proposed by the packing dialogue then covers every transfer of the batch.

---

## 18. Choosing a parcel point of an external network

1. The seller installs the parcel-point network companion. It ships a delivery product whose code is
   `MR`, and three Delivery Methods, one per group of countries, each of the rule-based kind and
   each at integration level `rate`.
2. A Delivery Method is recognised as a parcel-point network method when its delivery product's code
   is exactly `MR`.
3. The brand code is required on such a method. Its default is the reproduced test value
   `BDTEST  `; the neutralisation statement of the companion resets every method to that value.
4. In the selection wizard, a parcel-point method shows the network's own selector widget, which is
   loaded from the network's public address and receives the brand code, the container code, the
   comma-separated upper-cased country codes of the method, the destination postal code, the
   destination country code and the identifier of the point already chosen.
5. Choosing a point stores its number, name, street, second street line, postal code, city and
   country on the wizard.
6. Confirming the wizard refuses with "Please, choose a Parcel Point" when no point was chosen.
   Otherwise the point becomes a delivery address:
   - the external reference is `MR#` followed by the point's number;
   - an existing address is reused when it is a descendant of the customer's commercial entity,
     carries that reference and has the same street and postal code;
   - otherwise an address of kind delivery is created under the customer with the point's name,
     street, second street line, postal code, city and country, resolved from the first two
     characters of the point's country value in lower case;
   - the order's delivery address is replaced by it.
7. Confirming the order is refused when the method's parcel-point nature and the delivery address's
   parcel-point nature disagree (DSH-047).
8. In the portal such an address shows the network's badge and may not be edited by the customer.
9. The tracking link of such a method is the network's own address (calculations section 18.2).

---

## 19. Invoicing the carriage

1. The shipping charge line is an ordinary Sales Order Line. It is invoiced with the rest of the
   order by [`../sales/`](../sales/), following the delivery product's own invoicing policy, which
   the shipped products set to *ordered quantities*.
2. A shipping charge line may not be invoiced on its own: the order must carry at least one other
   invoiceable line.
3. Adding a shipping charge line to an order whose goods are all invoiced does not make the order
   invoiceable again, because the charge line follows its product's policy and that policy is
   already satisfied.
4. Once the charge line carries an invoiced quantity, it can no longer be removed by the
   replacement path (DSH-026).
5. What reaches the ledger is described in [accounting-effects.md](accounting-effects.md).

---

## 20. Adding a new carrier integration

This is the procedure a rebuild must support so that a carrier can be added without changing the
core.

1. Add one value to the provider kind, with a label, and declare what happens to records carrying
   that value when the package is removed.
2. Add the configuration fields the carrier needs — credentials, account number, service level,
   container defaults — and show them on the method's form for that kind only.
3. Implement the rating routine. It takes a Sales Order and returns the four values of the rate
   result. It must express its charge in the company currency, because the pipeline treats it as
   such.
4. Implement the sending routine. It takes a set of transfers and returns one result per transfer,
   each carrying the exact charge and the tracking reference, and it attaches the labels itself.
5. Implement the tracking-link routine, taking a transfer and returning a link or nothing.
6. Implement the cancellation routine, taking a set of transfers.
7. Optionally implement the return-label routine, taking a set of transfers, a tracking number and
   an original date, and override the return-capability computation to true for the new kind.
8. Optionally implement the default container-code routine, so that a Package Type created for the
   new carrier receives the carrier's own code for a non-standard container.
9. Optionally override the insurance-support computation to true for the new kind.
10. Optionally override the compliance hook to refuse a transfer whose data the carrier will not
    accept, and the pickup-location routines when the carrier operates collection points: a boolean
    named after the kind followed by `_use_locations`, and a routine named with a leading underscore,
    the kind and `_get_close_locations`.
11. Add one value to the Package Type's carrier field, equal to the value added in step 1, so that
    container types can be reserved for the new carrier.
12. Add a neutralisation statement that switches the new methods out of the production environment.
