# Interfaces

Everything through which a person, another system or another part of this system reaches the
Delivery and Shipping domain: menus, screens, buttons, named operations, routes, printable
documents, portal fragments, the contract every carrier integration must satisfy, and the external
services the domain contacts.

| # | Subject |
|---|---|
| 1 | Menus |
| 2 | Screens and the controls on them |
| 3 | Named operations invoked from a screen |
| 4 | The carrier-integration contract |
| 5 | Operations other domains call on this one |
| 6 | Routes |
| 7 | Printable documents |
| 8 | Portal fragments |
| 9 | Client components |
| 10 | External services contacted |
| 11 | Import and export |

---

## 1. Menus

| Menu path | Opens | Sequence | Visible to |
|---|---|---|---|
| Sales → Configuration → Delivery Methods | The Delivery Method list, grouped by provider kind | 4 | Anyone who can read a Delivery Method |
| Inventory → Configuration → Delivery Methods | The same list | 1 | Anyone who can read a Delivery Method |
| Inventory → Configuration → Postal Code Prefix | The Delivery Postal Code Prefix list | 100 | Members of the technical group |
| Storefront → Configuration → Delivery Methods | The same Delivery Method list | 90 | Anyone who can read a Delivery Method |
| Storefront → Configuration → Postal Code Prefix | The Delivery Postal Code Prefix list | 100 | Members of the technical group |

The last two menu items are contributed by [`../website-and-storefront/`](../website-and-storefront/)
and are listed here because they open records this domain owns.

Two windows are opened by a button rather than by a menu item:

| Button | Opens |
|---|---|
| "Install more Providers", on a Delivery Method form | A gallery and a list of the capability packages whose technical name begins with the delivery prefix, excluding three packages that are bridges rather than carriers: the barcode bridge, the batch-transfer bridge and the connected-device bridge. When no such package is available the window shows an empty state inviting the reader to obtain further carrier integrations. |
| "Delivery Methods", on an installed package's card in that gallery | The Delivery Method list, filtered on the provider kind derived from the package's technical name by removing the delivery prefix. For the parcel-point companion the filter is the parcel-point flag instead, because that companion adds no provider kind of its own. A package whose technical name does not begin with the delivery prefix produces no window. |

---

## 2. Screens and the controls on them

### 2.1 The Delivery Method list

Columns: the drag handle for the sequence, the name, the provider kind, the company (for
multi-company users), the countries, the maximum weight, the maximum volume, the required tags and
the excluded tags. The last five are optional columns; the maximum weight is shown by default and
the others are hidden by default. The parcel-point companion makes the countries column shown by
default, because its three shipped methods differ only by country.

The list is opened grouped by provider kind.

### 2.2 The Delivery Method search panel

| Control | Effect |
|---|---|
| Search on "Carrier" | Matches the name |
| Search on the provider kind | Matches the kind |
| Filter "Archived" | Shows the archived methods |
| Grouping "Provider" | Groups by provider kind, keeping empty groups visible |
| Filter "Published" | Contributed by the storefront; shows the published methods |

### 2.3 The Delivery Method form

**Status buttons**, shown only for a provider kind other than the two core kinds and the in-store
kind:

| Button | Shown when | Effect |
|---|---|---|
| "Production / Environment" | The production flag is set | Clears the production flag |
| "Test / Environment" | The production flag is clear | Sets the production flag |
| "No debug" | The debug flag is clear | Sets the debug flag |
| "Debug requests" | The debug flag is set | Clears the debug flag |

An archived method carries the ribbon "Archived".

**The identity block** holds the name, with the placeholder "e.g. UPS Express".

**The provider block** holds the provider kind, the "Install more Providers" link, the
cash-on-delivery flag, the integration level as a radio control and, for multi-company users, the
company with the placeholder "Visible to all". The Delivery – Inventory bridge adds the routes
field to this block, shown only to members of the advanced-locations group.

**The charge block** holds the fixed charge (shown only for the fixed kind), the proportional margin
labelled "Margin on Rate" and shown as a percentage, the fixed margin labelled "Additional margin",
the waiver flag and its threshold, the delivery product, the tracking-link pattern with the
placeholder "i.e. https://ekartlogistics.com/shipmenttrack/<shipmenttrackingnumber>", the invoicing
policy as a radio control and the insurance percentage. Each control is hidden for the kinds it does
not apply to, as described in [entities.md](entities.md) section 1.2. The parcel-point companion
adds the brand code after the delivery product, shown and required only for a parcel-point method.

**The pricing page**, shown only for the rule-based kind, holds the ordered list of Delivery Price
Rules. Each row shows the drag handle and the rule's readable name. Opening a row shows the
condition as three side-by-side controls — variable, operator, comparison value — and the charge as
a base amount, a plus sign, a factor amount, a multiplication sign and a factor variable.

**The availability page** opens with the note "Filling this form allows you to make the shipping
method available according to the content of the order or its destination." It holds two groups:

- *Destination*: the countries, the regions restricted to those countries and read-only while no
  country is chosen, and the postal-code prefixes, likewise read-only and offering no inline
  creation form.
- *Content*: the maximum weight with its unit label, the maximum volume with its unit label, the
  required tags and the excluded tags.

While no country is chosen the page shows the italic note "Please select a country before choosing
a state or a zip prefix." The whole page except the two tag fields is hidden for the in-store kind.

**The stores page**, contributed by the collection-in-store package and shown only for the in-store
kind, lists the stores with their name, opening hours, stock location (for multi-location users),
address and company. Stores cannot be created from the list.

**The description page** holds the customer-facing description, with the placeholder "Shipping
method details to be included at bottom sales orders and their confirmation emails. E.g.
Instructions for customers to follow."

The empty state of the whole list reads: "Define a new delivery method", then "Each carrier (e.g.
UPS) can have several delivery methods (e.g. UPS Express, UPS Standard) with a set of pricing rules
attached to each method.", then "These methods allow to automatically compute the delivery price
according to your settings; on the sales order (based on the quotation) or the invoice (based on
the delivery orders)."

### 2.4 The Delivery Postal Code Prefix screens

A list and a form, each holding the single prefix field. The empty state reads "Manage delivery zip
prefixes", then "Delivery zip prefixes are assigned to delivery carriers to restrict which zips it
is available to."

### 2.5 The Sales Order form

| Control | Where | Shown when |
|---|---|---|
| "Add shipping" | Under the order lines | The order has lines, they are not all services, and no shipping charge line exists |
| "Update shipping cost", amber | Under the order lines | The order has lines, they are not all services, a shipping charge line exists and the recomputation flag is set |
| "Update shipping cost", plain | Under the order lines | As above but the recomputation flag is clear |
| Shipping Weight | Beside the commitment date | Always; read-only on the form |
| Amber highlighting of the charge line | The line list | The recomputation flag is set and the line is the shipping charge line |

### 2.6 The Delivery Method Selection Wizard

A dialogue holding the method, the total weight with its unit label, the cost, a "Get rate" button
shown for every kind but the two core kinds, an amber banner carrying the invoicing message when
there is one, a blue banner carrying the delivery message when the last rate succeeded, a red
banner carrying the same message when the carrier declined to quote, and three buttons.

| Button | Shown when | Label |
|---|---|---|
| Confirm, update path | The dialogue was opened to update | "Update" |
| Confirm, add path with a rate | The dialogue was opened to add, and either the kind is a core kind or a cost has been obtained | "Add" |
| Confirm, add path without a rate | The dialogue was opened to add, the kind is not a core kind and no cost has been obtained | "Add", preceded by the confirmation prompt "Are you sure you want the delivery to be free for this order? You might have forgotten to compute the rates." |
| Discard | Always | "Discard" |

The parcel-point companion adds the network's selector widget to the dialogue, shown only for a
parcel-point method.

### 2.7 The Transfer form

**The shipping information group**, inserted before the other-information group, holds the Delivery
Method — read-only once the transfer is done or cancelled, with no inline creation and no link
through to the method — the tracking reference with a "Cancel" button beside it, the computed
weight with its unit label and the shipping weight with its unit label.

**Header buttons:**

| Button | Shown when |
|---|---|
| "Send to Shipper" | The tracking reference is empty, the provider kind is set and is neither `fixed` nor `base_on_rule` nor `in_store`, the transfer is done and the operation kind is not incoming |
| "Print Return Label" | The transfer is a return, it is not done and the operation kind is incoming |

**Status button:** "Tracking", shown when the transfer carries a tracking reference and a Delivery
Method.

**The Cancel button** beside the tracking reference asks "Cancelling a delivery may not be undoable.
Are you sure you want to continue?" and is shown when the tracking reference is set, the provider
kind is set and is neither `fixed` nor `base_on_rule`, and the transfer is done.

**Required contact:** the destination contact becomes required when the transfer carries a Delivery
Method whose integration level is `rate_and_ship`.

### 2.8 The Transfer list

Five optional columns are added, all hidden by default: the tracking reference, the Delivery
Method, the destination country under the heading "Destination", the computed weight and the
shipping weight.

### 2.9 Other screens the domain changes

| Screen | Change |
|---|---|
| Stock Move form | The move's weight is shown after the destination location |
| Detailed operations list | The destination country and the Delivery Method are added as optional columns |
| Detailed operations search panel | The Delivery Method is added as a searchable field and as the grouping "Carrier" |
| Package form | The shipping weight is shown with its unit, followed by the computed weight in muted text between brackets and preceded by the word "computed" |
| Package Type form | The carrier and the carrier code are added to the delivery group |
| Package Type list | The carrier is added after the sequence and the carrier code after the maximum weight |
| Operation Type form | The shipping-label flag is added after the sequence code, shown only for internal and outgoing operation types |
| Operation Type form, batching section | The carrier grouping is added after the contact grouping, and the maximum weight before the automatic-confirmation flag; both are shown only while automatic batching is on |
| Route form | The shipping-selectable flag is added to the route-selector group under the label "Shipping Methods" |
| Rule form | The carrier-propagation flag is added after the cancellation-propagation flag |
| Contact form | The default Delivery Method is added to the sales group |
| Product form | The Harmonized System code and the country of origin are added to the lots-and-weight group |
| Warehouse form | The opening hours are added after the address |
| Packing dialogue | The shipping weight and its unit label are added after the result package, shown only when a carrier kind is present and either a container type or an existing package is chosen; the container-type field and the result-package field are restricted to the carrier kind |
| Payment provider form | The response-code control is hidden for the cash-on-delivery mode |
| Payment form | The submit button is told to adapt its label for cash on delivery |
| Sales settings page | A button labelled "Shipping Methods" opens the Delivery Method list, shown only while the Delivery Costs package is installed |
| Storefront settings page | A button labelled "Configure Pickup Locations" opens the in-store Delivery Methods |

### 2.10 The informational dialogue for several tracking links

A one-field dialogue whose body reads "You have multiple tracker links, they are available in the
chatter." and whose only button is "OK". It is opened by the tracking button when the stored
tracking link parses as a list of pairs.

---

## 3. Named operations invoked from a screen

| Operation | Invoked from | Takes | Does |
|---|---|---|---|
| Toggle the production environment | The method form | One or more Delivery Methods | Inverts the production flag on each |
| Toggle debug logging | The method form | One or more Delivery Methods | Inverts the debug flag on each |
| Install more providers | The method form | Nothing | Returns the package gallery window described in section 1 |
| Show the delivery methods of a package | The package gallery | One package | Returns the Delivery Method list filtered on the kind derived from the package's technical name |
| Open the delivery wizard | The Sales Order form | One Sales Order | Returns the selection-wizard dialogue, pre-filled with the order, the default method and the estimated weight; when opened in update mode the title becomes "Update shipping cost" and the method is the order's own |
| Update the price | The selection wizard | One wizard record | Re-requests the rate and re-opens the same dialogue; raises the engine's error message when the rate fails |
| Confirm | The selection wizard | One wizard record | Writes the shipping charge line, clears the recomputation flag and copies the delivery message |
| Send to shipper | The Transfer form | One Transfer | Runs the shipment procedure of [workflows.md](workflows.md) section 9 |
| Print the return label | The Transfer form | One Transfer | Calls the return-label routine of the method |
| Cancel the shipment | The Transfer form | One or more Transfers | Calls the cancellation routine, posts the cancellation message and clears the tracking reference |
| Open the tracking page | The Transfer form | One Transfer | Opens the tracking link, or posts the list of links and opens the informational dialogue, or refuses when there is no link |
| Show the in-store delivery methods | The storefront settings page | Nothing | Returns the single in-store method in form view when exactly one exists, and the filtered list otherwise |

---

## 4. The carrier-integration contract

A carrier integration is reached only through the six routines below, each named after the
integration's own provider-kind value. The core never calls a carrier in any other way, and a
rebuild must keep the same six shapes so that an integration written for one deployment works in
another.

### 4.1 Rate a shipment

| Aspect | Contract |
|---|---|
| Name | The provider kind followed by `_rate_shipment` |
| Called on | One Delivery Method |
| Takes | One Sales Order |
| Returns | A record with exactly four members: whether it succeeded, a charge, an error message and a warning message |
| Charge currency | The company currency; the pipeline converts and rounds afterwards |
| On success | The three following steps of the pipeline run: the fiscal adaptation, the margins and the waiver |
| On failure | The error message is shown to the user by whichever caller asked; the charge must be zero |
| When absent | The pipeline returns failure with "Error: this delivery method is not available." |
| Extra members | A caller may read a fifth member saying that the carrier declined to quote; the selection wizard uses it to colour the message banner red instead of blue |

The two core implementations are specified in [calculations.md](calculations.md) sections 4 and 6.
The in-store implementation ignores its argument entirely and always answers success with the
delivery product's sales price.

### 4.2 Create a shipment

| Aspect | Contract |
|---|---|
| Name | The provider kind followed by `_send_shipping` |
| Called on | One Delivery Method |
| Takes | A set of Transfers |
| Returns | One record per Transfer, in the same order, each carrying the exact charge and the tracking reference |
| Side effects expected of the integration | Attaching the shipping labels to each Transfer, named with the shipping prefix; attaching any accompanying document, named with the document prefix; producing the return label when the method asks for one at delivery |
| Charge currency | The company currency; the caller applies the shipment-time waiver, then the margins |
| Tracking reference | A single reference, or nothing when the carrier issues none |
| On failure | Raise a user-level refusal carrying the carrier's own text. The caller turns it either into an aborted validation or into a warning activity, as described in [workflows.md](workflows.md) section 9 |
| When absent | The dispatcher returns nothing and the caller fails without a user-facing message; see **compatibility finding** DSH-057 |

The fixed implementation returns the method's fixed charge and no tracking reference, for every
transfer. The rule-based implementation repeats the destination-address test for each transfer,
refusing with "There is no matching delivery rule." when it fails, and otherwise returns the
rule-based charge computed from the transfer's Sales Order, or zero when the transfer has none, and
no tracking reference.

### 4.3 Build a tracking link

| Aspect | Contract |
|---|---|
| Name | The provider kind followed by `_get_tracking_link` |
| Called on | One Delivery Method |
| Takes | One Transfer |
| Returns | A link, or nothing |
| When absent | The computed tracking link of the Transfer is empty and the tracking button refuses |

The two core implementations substitute the tracking reference into the method's pattern. An
integration may return a structured list of label-and-link pairs instead of a single link, in which
case the transfer's tracking button posts the list and opens the informational dialogue.

### 4.4 Cancel a shipment

| Aspect | Contract |
|---|---|
| Name | The provider kind followed by `_cancel_shipment` |
| Called on | One Delivery Method |
| Takes | A set of Transfers |
| Returns | Nothing |
| Side effects expected of the integration | Voiding the shipment with the carrier |
| When absent | Nothing happens and the caller proceeds to clear the tracking reference |

The two core kinds declare the routine but leave it unimplemented; see **compatibility finding**
DSH-056.

### 4.5 Produce a return label

| Aspect | Contract |
|---|---|
| Name | The provider kind followed by `_get_return_label` |
| Called on | One Delivery Method |
| Takes | A set of Transfers, an optional tracking number and an optional original date |
| Returns | Whatever the integration chooses; the caller ignores it |
| Side effects expected of the integration | Attaching the return labels, named with the return prefix |
| Guard | Called only when the method's return-capability computation answers true for its kind |
| Afterwards | When the method publishes return labels, the caller generates an access token on each return-label attachment |

### 4.6 Supply a default container code

| Aspect | Contract |
|---|---|
| Name | A leading underscore, the provider kind, then `_get_default_custom_package_code` |
| Called on | One Delivery Method |
| Takes | Nothing |
| Returns | The carrier's own code for a non-standard container, or nothing |
| Used by | The Package Type form: choosing a carrier looks up the first Delivery Method of that kind and fills the carrier code with the answer, clearing it when there is no such method |
| When absent | The answer is nothing and the carrier code is cleared |

### 4.7 Offer collection points

Two further members are optional and are read by name rather than called blindly:

| Member | Shape | Meaning |
|---|---|---|
| The provider kind followed by `_use_locations` | A boolean field on the Delivery Method | Whether the method offers collection points. The order reads it to decide whether to store a chosen point and whether express checkout should skip the method |
| A leading underscore, the provider kind, then `_get_close_locations` | A routine taking a destination address and any further named arguments | Returns the list of collection point records near that address, already sorted. An empty list produces the error "No pick-up points are available for this delivery address." |

The collection-in-store implementation is specified in [calculations.md](calculations.md)
section 15.

### 4.8 Two optional overrides

| Override | Effect |
|---|---|
| The return-capability computation | Set to true for the integration's own kind, which reveals the return-label controls and makes the return-label routine reachable |
| The insurance-support computation | Set to true for the integration's own kind, which reveals the insurance percentage |
| The compliance hook | Called for every Transfer after the shipment step; an integration raises a refusal from it when the transfer's data would be rejected by the carrier |

### 4.9 The debug log

When a Delivery Method's debug flag is set, an integration may record the body of a request or a
response. The record is written on a **separate connection**, so that it survives a failure that
undoes the surrounding work, and it carries: the entity name `delivery.carrier`, the kind `server`,
the database name, the level `DEBUG`, the body as the message, the provider kind as the path, the
name of the routine as the function and the line number 1. A failure of that separate connection is
swallowed, so debug logging can never break a shipment.

---

## 5. Operations other domains call on this one

| Operation | Called by | Takes | Returns |
|---|---|---|---|
| Rate a shipment through the pipeline | The selection wizard, the storefront, express checkout | A Sales Order | The rate result of section 4.1 plus the pre-waiver charge |
| Filter the available methods | The selection wizard, the Transfer form, the storefront | A destination contact and a source document | The subset of the methods that pass the availability filter |
| Test one method against one order | The storefront | A Sales Order | Whether the method is available, including the rating test for the rule-based kind |
| Set the shipping charge line | The selection wizard, the storefront, the collection-in-store routes | A Delivery Method and an amount | Removes the existing charge lines, writes the method on the order, creates the line and, for a confirmed order, writes the method on the pending transfers |
| Remove the shipping charge lines | The selection wizard, the storefront | Nothing | Deletes the uninvoiced charge lines or refuses |
| Store a collection point | The storefront and the back-office selector | The point's description | Stores it, and for the in-store kind sets the warehouse, the fiscal position and the taxes |
| List the collection points | The storefront and the back-office selector | An optional postal code and an optional country | The list of points, or an error |
| Estimate the order weight | The selection wizard, parcel building | Nothing | The estimated weight |
| Estimate the transfer weight | Parcel building | Nothing | The estimated weight |
| Build the parcels of an order | A carrier integration | The order and a default container type | The list of Delivery Parcels |
| Build the parcels of a transfer | A carrier integration | The transfer and a default container type | The list of Delivery Parcels |
| Build the commodities of an order | A carrier integration | The order | The list of Delivery Commodities |
| Build the commodities of a set of move lines | A carrier integration | The move lines | The list of Delivery Commodities |
| Decide whether a commercial invoice is needed | A carrier integration | Nothing | True when the transfer's warehouse address is in a different country from the destination contact |
| Send the shipment of a transfer | The validation step and the "Send to Shipper" button | Nothing | Performs the whole procedure of [workflows.md](workflows.md) section 9 |
| Read the tracking links of a transfer | The portal page | Nothing | The parsed list of label-and-link pairs, or nothing when the stored value is not such a list |

---

## 6. Routes

Four routes belong to this domain. Two more, owned by
[`../website-and-storefront/`](../website-and-storefront/), are listed because they call operations
of this domain and because the collection-in-store package overrides one of them.

| Path | Transport | Authentication | Owner | Takes | Returns |
|---|---|---|---|---|---|
| `/delivery/set_pickup_location` | Structured call | Signed-in user | This domain | The Sales Order identifier and the point's description as a structured text | Nothing |
| `/delivery/get_pickup_locations` | Structured call | Signed-in user | This domain | The Sales Order identifier and an optional postal code | The list of collection points, or an error. The country is taken from the geolocation of the request when it is known, and from the order's delivery address otherwise |
| `/shop/set_click_and_collect_location` | Structured call | Public | The collection-in-store package | The point's description as a structured text | Nothing. Creates a cart when there is none, sets the in-store method with the delivery product's sales price when the cart carries another method, then stores the point |
| `/website_sale/get_pickup_locations` | Structured call | Public | The storefront, overridden here | An optional postal code and, when called from a product page, the product identifier | The list of collection points. The override sets the in-store method on the cart when the call comes from a product page and the cart carries another method, and answers from a transient order when there is no cart at all, so that browsing never creates a cart |
| `/shop/delivery_methods` | Structured call | Public | The storefront | Nothing | The rendered delivery form. The collection-in-store package adds the default collection point of every single-store in-store method, together with that store's stock check |
| `/shop/get_delivery_rate` | Structured call | Public | The storefront | A Delivery Method identifier | The rate, with the taxes applied according to the website's tax display setting |

Two further storefront routes call this domain without being changed by it:
`/shop/set_delivery_method`, which sets the method on the cart, and the express-checkout address
route, which lists the methods sorted by ascending charge.

---

## 7. Printable documents

The domain creates no printable document of its own. It adds content to five documents owned
elsewhere.

### 7.1 The quotation and order document

Owned by [`../sales/`](../sales/). Immediately before the order note, when the order's Delivery
Method carries a customer-facing description, a paragraph headed "Shipping Description" prints that
description.

### 7.2 The transfer document

Owned by [`../inventory-operations/`](../inventory-operations/). After the date block:

- when the transfer carries a Delivery Method, a block headed "Carrier" showing the method;
- when the transfer carries a shipping weight, a block headed "Weight" showing the weight and its
  unit label;
- when the transfer carries a Delivery Method, a further column headed "Shipping Method" showing the
  method.

### 7.3 The delivery slip

Owned by [`../inventory-operations/`](../inventory-operations/). After the scheduled-date block:

- a column headed "Carrier" showing the Delivery Method, when there is one;
- a column headed "Total Weight" showing the shipping weight and its unit label, when there is one;
- a column headed "Tracking Number" showing the tracking reference, when there is one.

A "HS Code" column — the Harmonized System code — is added to the goods table, to the detailed
operations table, to the serial-number rows and to the aggregated rows, and is shown only when at
least one move of the transfer carries a product with such a code. The aggregation that feeds the
aggregated rows is extended so that each aggregated line carries the code of its product.

Each package section line is extended with the package's weight: " - Weight: <weight> <unit
label>" when a shipping weight was typed, and " - Weight (estimated): <weight> <unit label>"
otherwise. The section line for the goods that are in no package is extended with " - Weight:
<bulk weight> <unit label>".

### 7.4 The parcel labels

Owned by [`../inventory-operations/`](../inventory-operations/). Three documents are extended: the
printer-language parcel label, the full-page parcel barcode and the small parcel barcode.

Each of them appends the weight segment of [calculations.md](calculations.md) section 19 to the
package's barcode, and prints the weight in words:

| Document | Printed text |
|---|---|
| Printer-language label | "Shipping Weight: <weight> <unit label>" when a shipping weight was typed, otherwise "Weight: <weight> <unit label>" |
| Full-page barcode | When the package carries a valid serial shipping container code: "Shipping Weight: <weight> <unit label>" or, failing that, "Weight: <weight> <unit label>". When it does not: "Shipping Weight:" on its own line followed by the weight and the display name of the weight unit, and nothing at all when no shipping weight was typed |
| Small barcode | A row "Shipping Weight: <weight> <unit label>" or "Weight: <weight> <unit label>", and, under the container type, a large centred row repeating the shipping weight when one was typed |

### 7.5 The product labels

Owned by [`../products-and-catalog/`](../products-and-catalog/). Both printer-language product
labels print the Harmonized System code, prefixed by the two letters and a colon that the label
format uses, when the product carries one.

---

## 8. Portal fragments

Owned by [`../customer-portal/`](../customer-portal/) and extended here.

| Fragment | Where | Content |
|---|---|---|
| Tracking block | The order's portal page, after each transfer's delivery details | The word "Tracking:" followed by: every label-and-link pair as a link separated by a plus sign, when the stored tracking link parses as a list; or the tracking reference as one link, when a tracking link exists; or the Delivery Method's name and the tracking reference as plain text, when no link exists |
| Return-label link | The same place | A link labelled "Print Return Label", shown when the method publishes return labels and the transfer carries at least one, pointing at the first return-label attachment with its access token |
| Parcel-point badge | The address card | The parcel-point network's badge, shown on an address whose external reference marks it as a parcel point |
| Payment-confirmation block | The storefront's payment confirmation page | For a pay-on-site transaction, the Delivery Method's name, the annotation "(In-store pickup)" and the method's online description, replacing the order reference block |

---

## 9. Client components

The domain contributes one reusable client component and one specialised one; both are described
here as behaviour, not as implementation.

### 9.1 The collection point selector

A dialogue that shows the collection points near a postal code, as a list and on a map. Its
properties are the Sales Order identifier, the postal code to search around, the identifier of the
point already chosen, a save operation and a close operation.

| Behaviour | Detail |
|---|---|
| Fetch | Calls the collection-point route with the order and the postal code, replacing the list on every change of postal code, and debouncing the search button by three tenths of a second |
| Title | "Pickup Location" when exactly one point came back, "Choose a pick-up point" otherwise |
| Selection | The first point is preselected unless the point already chosen is still in the list; selecting is by identifier compared as text, because a carrier may identify a point by letters |
| List or map | The list is hidden when exactly one point came back. On a narrow screen the user switches between them with the buttons "List view" and "Map view" |
| Point card | The name, the street, the postal code and the city on one line, and the opening hours under the heading "Opening hours", with "Closed" for a day that has no range and the day names written in full in the reader's language, the week starting on Monday |
| Confirm | "Choose this location" calls the save operation with the whole point record and closes |
| Empty and busy states | "No result" when the route answered with an error, "Loading..." while the request is running |
| Postal code control | Placeholder "Your postal code" |
| Resize | The dialogue re-measures itself on a window resize, debounced by three tenths of a second |

### 9.2 The click-and-collect availability widget

Shown on a product page when collection in store is enabled. Its properties are the product variant
identifier, whether the current variant combination is possible, the postal code, the store already
chosen, the in-store stock figures, the ordinary-delivery stock figures and whether a "select
store" button should be offered. It listens for the event the product page raises when the variant
changes, and re-reads the product identifier, the two stock figures and the possibility flag from
it. Pressing the store button opens the selector of section 9.1 in product mode; confirming a store
calls the product-page route of section 6.

### 9.3 The parcel-point selector widget

A field widget shown on the selection wizard for a parcel-point method. It loads the network's own
selector from the network's public address and passes it the brand code, the container code, the
comma-separated upper-cased country codes of the method, the destination postal code, the
destination country code, a responsive flag, a flag asking for results on a map and the identifier
of the point already chosen. When the customer picks a point, the widget writes the point's number,
name, street, second street line, postal code, city and country into the wizard's field. When the
network reports no result, the widget suppresses the resulting third-party script error for ten
seconds, because the network's own selector fails on an invalid postal code.

### 9.4 The checkout adjustments for collection in store

| Behaviour | Detail |
|---|---|
| Titles | Selecting an in-store method retitles the address section "Contact Details" and the reuse-for-billing option "Same as contact details"; selecting any other method restores the original titles |
| Reload | Selecting an in-store method, and confirming a collection point for one, both reload the page, because the taxes, the warehouse and the titles all change |
| Warning block | Shown for an in-store method whose chosen store is short; hidden when the store is changed to one that is not |
| Readiness | The checkout refuses to proceed while the warning block is present |
| Quantity buttons | "Update cart with available stock" sets the line to the available quantity; "Remove" sets it to zero. Both reload the page |
| Method icons | An in-store method is marked with a location icon and every other method with a lorry icon, but only when collection in store is enabled for the website |

---

## 10. External services contacted

| Service | Contacted by | Address | Why |
|---|---|---|---|
| The parcel-point network's tracking page | The customer, through the link the domain builds | `https://www.mondialrelay.com/public/permanent/tracking.aspx?ens=<brand>&exp=<reference>&language=<language>` | To show the shipment's progress. The address is reproduced because it is part of the integration contract |
| The parcel-point network's selector | The selection wizard | `https://widget.mondialrelay.com/parcelshop-picker/jquery.plugin.mondialrelay.parcelshoppicker.min.js` | To let the customer choose a point. Reproduced for the same reason |
| A public map tile service | The collection point selector | `https://tile.openstreetmap.org/{z}/{x}/{y}.png`, where the three placeholders are the zoom level and the two tile coordinates | To draw the map behind the collection points. Reproduced for the same reason |
| A geolocation service | The preparation of a store's collection point record | Contacted through [`../contacts-and-organizations/`](../contacts-and-organizations/), which owns the address and the choice of service | To turn a store's address into coordinates |

No other external service is contacted by this domain. Every carrier's own service is contacted by
the carrier integration, through the six routines of section 4.

---

## 11. Import and export

The domain defines no import routine and no export routine of its own. Its records are imported and
exported through the platform's generic mechanism, described in
[`../../data/data-loading-and-exchange.md`](../../data/data-loading-and-exchange.md). Three
properties matter when a Delivery Method is imported:

1. The delivery product must exist first, because the reference is required and restricted.
2. The company is derived from the delivery product unless it is supplied, so an import that
   supplies neither produces a method visible to every company.
3. Importing a Delivery Method of the in-store kind runs the in-store defaults of
   [business-rules.md](business-rules.md) rule DSH-089, which overwrite the integration level, the
   cash-on-delivery flag and the three destination filters whatever the imported file said, and
   attach every warehouse of the resolved company.

A Delivery Price Rule is imported against its method and carries no other reference. A Delivery
Postal Code Prefix is imported by its value, which is upper-cased on the way in and must be unique.
