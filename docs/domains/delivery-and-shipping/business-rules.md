# Business rules

The complete rule catalogue of the Delivery and Shipping domain: validations, constraints,
invariants, guards, permission checks, locking rules, company-consistency rules and the exact text
of every message the user sees. Every rule carries a stable identifier of the form DSH-nnn, unique
within this file, so that the other files of the folder can cite it. Section 15 indexes every
identifier.

A note on message wording. User-facing messages are reproduced exactly as the system produces them,
in quotation marks. A placeholder inside a message is written between angle brackets and its
content is described in words. Where a message contains a product name that a compatible rebuild
must reproduce, the message is still reproduced verbatim, because support procedures and automated
tests key on the exact text.

---

## 1. The Delivery Method record

**DSH-001** A Product Tag may not appear both among a Delivery Method's required tags and among its
excluded tags. The check runs whenever either set is written. Violation message: "Carrier <name>
cannot have the same tag in both Must Have Tags and Excluded Tags.", where the placeholder is the
Delivery Method's name.

**DSH-002** A Delivery Method must carry a name. The name is translatable and is what the customer
sees on the quotation, on the invoice and in the storefront.

**DSH-003** A Delivery Method must carry a delivery product. The reference is restricted: the
product cannot be deleted while a Delivery Method points at it.

**DSH-004** The proportional margin may not be lower than −1, that is, minus one hundred per cent.
Enforced at the database level. Violation message: "Margin cannot be lower than -100%"

**DSH-005** The insurance percentage must lie between 0 and 100 inclusive. Enforced at the database
level. Violation message: "The shipping insurance must be a percentage between 0 and 100."

**DSH-006** The provider kind is required and defaults to the fixed kind. A rebuild must reject an
unknown value, because the whole dispatch of rating, sending, tracking, cancellation and
return-label production is keyed on it.

**DSH-007** The invoicing policy is required and defaults to `estimated`. The value `real` exists
only while the Delivery – Inventory bridge is present; removing that bridge resets every method
carrying `real` to `estimated`.

**DSH-008** Selecting the integration level `rate` resets the invoicing policy to `estimated`. This
is a screen-level adjustment: a programmatic write can leave the pair inconsistent, and the
consequence is described in DSH-055.

**DSH-009** Clearing the return capability clears the automatic-return flag, and clearing the
automatic-return flag clears the portal flag. Both are screen-level adjustments applied when the
user changes the field, not stored constraints.

**DSH-045** Duplicating a Delivery Method copies its Delivery Price Rules and renames the copy
"<name> (copy)", where the placeholder is the name of the original. No other field is altered, so
the copy points at the same delivery product and therefore inherits the same company and currency.

**DSH-048** A Delivery Method's company is a stored mirror of its delivery product's company and
may be overwritten afterwards. A method with no company is visible to every company.

**DSH-049** Clearing every country of a Delivery Method also clears every postal-code prefix. The
screen further makes the region field and the prefix field read-only while no country is chosen and
shows the note "Please select a country before choosing a state or a zip prefix."

**DSH-050** Changing the set of countries removes from the region set every region that does not
belong to one of the chosen countries.

**DSH-044** Only a Route flagged as applicable to shipping methods may be attached to a Delivery
Method; the selection list of the field is restricted to those routes. When an order carried by
such a method is confirmed, every order line that names no route of its own is procured through the
method's routes; a line that names its own routes keeps them. A line procured through the method's
routes is sourced from the location the route's rule names, which is how a Delivery Method can pull
goods from a particular shelf or a particular warehouse.

---

## 2. Availability of a Delivery Method

**DSH-010** A Delivery Method is available for a document only when all five tests of
[calculations.md](calculations.md) section 1 hold. Failing the filter is not an error: the method is
simply absent from the list the user can choose from.

**DSH-011** When a Delivery Method lists countries, a destination contact whose country is not one
of them, and a destination contact with no country at all, both fail the test.

**DSH-012** When a Delivery Method lists regions, a destination contact whose region is not one of
them, and a destination contact with no region at all, both fail the test.

**DSH-013** When a Delivery Method lists postal-code prefixes, a destination contact whose
upper-cased postal code does not start with one of them, and a destination contact with no postal
code at all, both fail the test.

**DSH-014** The weight and volume tests read the products of *every* line of the document, without
excluding services, display lines or the shipping charge line itself, and convert every quantity
into the product's reference unit before multiplying by the product's unit weight or unit volume. A
limit of zero means no limit.

**DSH-015** The required-tag test and the excluded-tag test read the complete tag set of the
products of the document, which includes the tags of the template and the tags of the variant. The
required test passes when the method requires no tag or at least one required tag is present; the
excluded test passes when no excluded tag is present. A service product's tags count: a service
line carrying an excluded tag makes the whole method unavailable.

**DSH-035** Passing anything other than a Sales Order or a Transfer as the source document of the
availability filter is refused with "Invalid source document type".

**DSH-036** In the storefront a rule-based Delivery Method is additionally required to produce a
successful rate before it is offered. A rule-based method whose rules match nothing for the current
basket is therefore never shown, rather than shown and then refusing.

**DSH-037** An archived Delivery Method is never offered, not even as the default method of a
Contact. The wizard's default is left empty when the contact's default method is archived or fails
the availability filter.

---

## 3. Delivery Price Rules

**DSH-016** A Delivery Method's rules are not validated against one another. Overlapping
conditions, contradictory conditions, gaps between conditions, negative amounts and duplicate
sequences are all accepted. The engine takes the first rule whose condition holds in the stored
order and reports failure when none does. A rebuild must reproduce this permissiveness, because
sellers rely on ordering rather than on exhaustiveness.

**DSH-018** A rule's condition variable, operator and comparison value are all required.

**DSH-019** A rule's base amount and factor amount are both required and default to zero.

**DSH-020** A rule's factor variable is required and defaults to the weight.

**DSH-021** When no rule of a rule-based Delivery Method matches, the rating fails with the error
message "Not available for current order".

**DSH-022** Deleting a Delivery Method deletes its rules. Deleting a rule does not affect the
method.

---

## 4. Rating

**DSH-023** The fixed kind ignores both margins. A fixed method carrying a proportional margin and
a fixed margin charges exactly the price the price list returns.

**DSH-024** A Delivery Method whose provider kind has no rating routine returns failure with the
error message "Error: this delivery method is not available." and a charge of zero.

**DSH-025** The fixed engine and the rule-based engine repeat only the destination-address test
before rating, not the tag, weight and volume tests. A failure returns the error message "Error:
this delivery method is not available for this address." and a charge of zero. The consequence is
that a caller which rates a method without first applying the availability filter can obtain a
charge for a basket that exceeds the method's weight or volume limit.

**DSH-038** The free-above-a-threshold waiver is never applied by the rating pipeline to a
rule-based Delivery Method. The exclusion is deliberate and the interface hides the waiver controls
for that kind. The shipment-time waiver of [calculations.md](calculations.md) section 7.7 carries no
such exclusion, so a rule-based method whose waiver flag was set programmatically does waive the
charge stored on the transfer while never having waived the charge on the order. This is recorded
as **compatibility finding** DSH-054.

**DSH-039** The waiver's comparison is *greater than or equal to*: an order whose total without
carriage equals the threshold exactly is waived.

**DSH-040** The waiver's threshold is expressed in the company currency and the order total is
converted into that currency before the comparison. The charge is set to zero in the order
currency.

---

## 5. The shipping charge line

**DSH-017** The selection wizard requires a Delivery Method before it can be confirmed. The field
is mandatory and is restricted to the methods of the available list.

**DSH-026** Replacing or removing the shipping charge lines of an order is refused when every
existing charge line has already been invoiced. Violation message, reproduced with its line breaks
and with the list of processed lines appended:

"You can not update the shipping costs on an order where it was already invoiced!

The following delivery lines (product, invoiced quantity and price) have already been processed:

"

Each processed line is then appended on its own line as a hyphen, a space, the product's display
name without its internal code, a colon, a space, the invoiced quantity, a space, a lowercase
letter x, a space and the unit price.

**DSH-027** When at least one shipping charge line is still uninvoiced, only the uninvoiced lines
are deleted and the operation succeeds.

**DSH-028** Deleting the last shipping charge line of an order clears the order's Delivery Method.

**DSH-029** A shipping charge line may be deleted from a confirmed order. The domain removes
shipping charge lines from the set of lines a confirmed order refuses to lose.

**DSH-030** A shipping charge line may not be invoiced on its own. An order whose only invoiceable
line is the carriage cannot produce an invoice.

**DSH-031** A shipping charge line never carries a price-list item and is excluded from the
price-list recomputation of the order, because its price comes from the rating pipeline.

**DSH-032** Writing the price or the description of a shipping charge line on a locked order is
refused, except during the write-back of the real carriage charge, which carries a flag that
removes those two fields from the protected set for that one write and only when every line being
written is a shipping charge line.

**DSH-033** The recomputation flag is set on the order whenever a line, the customer or the
delivery address changes while a shipping charge line exists. It is cleared when the selection
wizard is confirmed. The flag never blocks anything; it only drives the amber highlighting.

**DSH-061** The shipping charge line's taxes are the delivery product's sale taxes restricted to
the order's company, mapped through the order's fiscal position when the order carries both a
customer and a fiscal position. When the order's company is a branch and the delivery product
carries taxes of both the branch and its parent, only the branch's taxes are kept; when it carries
only the parent's taxes, those are kept.

**DSH-062** When a Delivery Method whose waiver flag is set produces a charge of zero, the charge
line's description is extended with a line break and the reproduced text "Free Shipping".

**DSH-063** When a Delivery Method's invoicing policy is `real`, the charge line is created at zero
and its description is extended with " (Estimated Cost: <charge formatted in the order currency>)".

---

## 6. Transfers and shipments

**DSH-043** A Transfer may only carry a Delivery Method whose company is compatible with the
transfer's company, and the choice is restricted to the methods that pass the availability filter
for the transfer's contact and content. The same company check applies to the Delivery Method of a
Sales Order.

**DSH-064** The destination contact of a Transfer becomes required as soon as the transfer carries
a Delivery Method whose integration level is `rate_and_ship`.

**DSH-065** A Delivery Method can no longer be changed on a Transfer that is done or cancelled.

**DSH-066** A shipment is never created twice for the same Transfer: the sending step is skipped
whenever the transfer already carries a tracking reference.

**DSH-067** A shipment is never created for an incoming Transfer.

**DSH-068** A shipment is only created when the operation type asks for shipping labels. The flag
is set by default on outgoing operation types and cleared on incoming and internal ones.

**DSH-069** When the sending routine refuses and no carrier transfer of the same validation batch
has been processed yet, the refusal aborts the whole validation and is shown to the user unchanged.

**DSH-070** When the sending routine refuses and at least one carrier transfer of the same batch
has already been processed, the refusal is *not* raised. The transfer stays validated, the refusal
text is posted as a message and a warning activity dated today is scheduled on the transfer for the
transfer's responsible user, or for the validating user when the transfer has none. The activity's
note reads "Exception occurred with respect to carrier on the transfer <a link carrying the
transfer's name>. Manual actions might be needed." followed by "Exception: <the refusal text>".

**DSH-071** The rule-based sending routine refuses with "There is no matching delivery rule." when
the transfer's contact does not pass the destination-address test of the method.

**DSH-072** A return Transfer is created with no Delivery Method and a shipping cost of zero,
whatever the source transfer carried. The domain has no return integration, so copying the carrier
would produce a second charge for a shipment nobody is making.

**DSH-073** A Transfer that is done keeps the Delivery Method it had; adding a shipping charge to an
already confirmed order writes the method only onto the transfers that are neither done nor
cancelled and that contain no move returning another move.

**DSH-074** Cancelling a shipment clears the tracking reference and posts "Shipment <tracking
reference> cancelled". It does not undo the shipping cost, the labels or the charge line.

**DSH-075** Opening the tracking page of a Transfer that has no tracking link is refused with "Your
delivery method has no redirect on courier provider's website to track this order."

---

## 7. Packing

**DSH-034** Packing move lines that name more than one Delivery Method, or of which at least one
names none, is refused with "You cannot pack products into the same package when they have
different carriers (i.e. check that all of their transfers have a carrier assigned and are using
the same carrier)."

**DSH-076** A transfer that carries a Delivery Method always shows the packing dialogue, whatever
the operation type's own setting, because the shipping weight and the container type must be
confirmed before the carrier is called.

**DSH-077** The packing dialogue restricts the selectable container types to those whose carrier
kind matches the kind derived from the lines, and the selectable existing packages to those whose
carrier kind and container type match as well.

**DSH-078** Exceeding the container type's maximum weight in the packing dialogue produces a
warning, not a refusal. Title: "Package too heavy!" Body when a container type was chosen: "The
weight of your package is higher than the maximum weight authorized for this package type. Please
choose another package type." Body when an existing package was chosen: "The weight of your package
is higher than the maximum weight authorized for its package type. Please choose another package."

**DSH-079** Packing without the dialogue still sets the package's shipping weight, to the weight the
package computes for the transfer, so that the carrier never receives a parcel of weight zero.

---

## 8. Building parcels

**DSH-080** Building parcels from a Sales Order whose total weight is exactly zero is refused with
"The package cannot be created because the total weight of the products in the picking is 0.0
<weight unit label>", where the placeholder is the display name of the weight unit named by the
weight system parameter.

**DSH-081** Building parcels from a Transfer that has no bulk weight and no package is refused with
the same message, the placeholder being the transfer's weight unit label.

**DSH-082** A commodity's whole-unit quantity is never below one, even when the exact quantity
rounds to zero. A commodity's exact quantity is kept beside it unrounded, so that a carrier that
accepts fractional quantities can use it.

**DSH-083** A commodity's country of origin falls back to the country of the source warehouse's
address when the product declares none, and is empty when neither is known.

---

## 9. Collection points

**DSH-046** The selection wizard refuses to confirm a parcel-point network method without a chosen
point. Violation message: "Please, choose a Parcel Point"

**DSH-047** Confirming a Sales Order is refused when the Delivery Method's parcel-point nature and
the delivery address's parcel-point nature disagree — a parcel-point method with an ordinary
address, or an ordinary method with a parcel-point address. Violation message: "Mondial Relay
mismatching between delivery method and shipping address." When several orders are confirmed
together, a space, an opening parenthesis, the comma-separated names of the disagreeing orders and
a closing parenthesis are appended. The message is reproduced verbatim because support procedures
and automated tests key on it.

**DSH-084** Asking for collection points from a Delivery Method whose provider kind declares no
close-locations routine answers with the error "No pick-up points are available for this delivery
address." The same error is returned when the routine answers with an empty list.

**DSH-085** A refusal raised by a close-locations routine is caught and returned as an error
carrying the routine's own text, so that the selector can show it instead of failing.

**DSH-086** Asking for collection points with a postal code but no country is a programming error.
The collection-in-store package resolves the country from the stored collection point or from the
geolocation of the request, and drops the postal code when neither yields a country.

**DSH-087** An address created from a collection point carries the pickup-point flag. Such an
address is excluded from the list of selectable delivery addresses and is reset to the customer
whenever an order's delivery address is recomputed onto it.

**DSH-088** An address created from a parcel-point network point may not be edited by the customer
in the portal.

---

## 10. Collection in store

**DSH-041** A Delivery Method of the in-store kind may not be published while it has no store.
Violation message: "The delivery method must have at least one warehouse to be published."

**DSH-042** Every store of an in-store Delivery Method that carries a company must carry that same
company. A store with no company is accepted. Violation message: "The delivery method and a
warehouse must share the same company"

**DSH-089** Creating a Delivery Method of the in-store kind forces the integration level to `rate`,
clears the cash-on-delivery flag and clears the three destination filters; it then attaches every
warehouse of the resolved company and publishes the method when at least one warehouse was found.
The company is resolved as the supplied company, failing that the delivery product's company,
failing that the current company.

**DSH-090** Writing the in-store kind onto an existing Delivery Method forces the same four values
but does **not** attach the warehouses, so such a method must have its stores added by hand before
it can be published. This is recorded as **compatibility finding** DSH-059.

**DSH-091** Reaching the payment step of the storefront with an in-store Delivery Method and
deliverable goods is refused when no store was chosen. The refusal is shown as a two-part message:
the heading "Sorry, we are unable to ship your order." and the body "Please choose a store to
collect your order."

**DSH-092** Reaching the payment step with an in-store Delivery Method, deliverable goods and a
chosen store that cannot supply every line is refused with the heading "Sorry, we are unable to
ship your order." and the body "Some products are not available in the selected store."

**DSH-093** The final readiness check performed immediately before payment repeats the stock test
and refuses with "Some products are not available in the selected store."

**DSH-094** A line the chosen store cannot supply carries the warning "<available>/<ordered>
available at this location", where the first placeholder is the greatest quantity the store can
supply expressed in the line's own unit and the second is the ordered quantity truncated to a whole
number.

**DSH-095** The stock check skips a product that is not storable and a product that is allowed to be
sold out of stock; both are always considered available.

**DSH-096** The stock check rounds the available quantity **downwards** when converting it into the
line's unit, because only whole units can be sold.

**DSH-097** When several lines of the same product exist, the check consumes the available quantity
line by line in the order the lines appear, so that a later line can be short while an earlier one
is not.

**DSH-098** The in-store kind is excluded from express checkout, together with every method that
offers collection points, because express checkout offers no way to choose a location.

**DSH-099** The click-and-collect availability widget is hidden for a product that carries one of
the in-store method's excluded tags.

---

## 11. Payment paths

**DSH-100** A custom payment provider whose mode is `cash_on_delivery` is removed from the
compatible providers unless the order's Delivery Method allows cash on delivery. The availability
report records the removal with the reason "cash on delivery not allowed by selected delivery
method".

**DSH-101** A custom payment provider whose mode is `on_site` is removed from the compatible
providers unless the order's Delivery Method is of the in-store kind **and** the order carries at
least one line whose product is goods. The availability report records the removal with the reason
"no in-store delivery methods available".

**DSH-102** Validating a transaction of a provider whose mode is `on_site` against an order whose
Delivery Method is not of the in-store kind is refused with "You can only pay on site when
selecting the pick up in store delivery method." The check exists to defeat a forged request and is
never reached through the interface.

**DSH-103** A pending transaction of a provider whose mode is `cash_on_delivery` confirms every
draft order it is attached to, with the confirmation message enabled. The same holds for a provider
whose mode is `on_site`.

**DSH-104** The payment methods whose codes are `cash_on_delivery` and `pay_on_site` are treated by
the payment form as pay-later methods, which changes the wording of the submit button.

---

## 12. Batching

**DSH-105** When the operation type groups batches by carrier, a transfer may only be batched with
transfers carrying the same Delivery Method; *no method* is a value that must match.

**DSH-106** When the operation type caps a batch by weight, a transfer is not added to a batch when
the sum of the batch's transfer weights plus the candidate's weight would exceed the cap, and two
transfers are not paired when the sum of their weights would exceed it. A line is not added to a
wave when the sum of the batch's move weights plus the line's weight would exceed it. A cap of zero
means no cap.

**DSH-107** The automatic batch's description is extended with the Delivery Method's name, preceded
by a comma and a space when the description was not empty.

---

## 13. Permissions and record visibility

**DSH-108** Reading a Delivery Method requires membership of one of: the sales user group, the
sales administrator group, the settings group, the contact-creation group, the inventory user group
or the inventory administrator group.

**DSH-109** Creating, changing and deleting a Delivery Method requires membership of the sales
administrator group or of the inventory administrator group. No other group may write one.

**DSH-110** Reading a Delivery Price Rule requires membership of the sales user group, the sales
administrator group, the inventory user group or the inventory administrator group. Creating,
changing and deleting one requires the sales administrator group or the inventory administrator
group.

**DSH-111** Reading a Delivery Postal Code Prefix requires membership of the sales user group, the
contact-creation group, the inventory user group or the inventory administrator group. Creating,
changing and deleting one requires the contact-creation group or the inventory administrator group.
The sales administrator group alone may not write a prefix.

**DSH-112** Creating, reading and changing a Delivery Method Selection Wizard record requires
membership of the sales user group or the inventory user group. Nobody may delete one; the
transient-record cleaner removes it.

**DSH-113** A global record rule limits every user to the Delivery Methods whose company is one of
the companies currently active for that user, plus the Delivery Methods with no company. The rule
applies to every operation and to every group, including administrators.

**DSH-114** The parcel-point container code is readable and writable only by members of the settings
group.

**DSH-115** The shipment-sending step runs with elevated rights, so that a warehouse operator who
cannot write a Sales Order can still push the real carriage charge onto it.

**DSH-116** The rule-based variable collection runs with elevated rights on both the method and the
order, so that a user who cannot read costs can still obtain a rate.

**DSH-117** The weight of a Stock Move and the weight of a Transfer are computed with elevated
rights, so that a user who cannot read a move can still see the transfer's weight.

**DSH-118** The shipping charge line is created with elevated rights, so that a user who may price
carriage but may not create order lines can still add one.

---

## 14. Compatibility findings and industry-standard defaults

**DSH-051** *(compatibility finding.)* The declared value of a parcel is computed by converting the
product's unit cost **from the company currency into the product's currency**, although the amount
starts life in the product's currency and the carrier expects the company currency. In the ordinary
configuration the two currencies are the same record and the conversion is skipped, so the defect is
invisible. A corrected behaviour would convert from the product's currency into the company
currency. A rebuild that wants byte-for-byte compatibility must reproduce the observed direction;
one that wants correctness should convert the other way and document the change.

**DSH-052** *(compatibility finding.)* When an order is split into several parcels, each
commodity's whole-unit quantity is divided by the number of parcels with the integer part taken and
a floor of one applied, and each parcel receives the *same* commodity list. Three parcels built
from seven units therefore declare two units each, six in total, and one unit is not declared. A
corrected behaviour would distribute the remainder across the parcels so that the declared
quantities sum to the shipped quantity.

**DSH-053** *(compatibility finding.)* The parcel that carries the loose goods of a transfer
declares a value computed over **every** move line of the transfer, including the lines already
inside a package, so a transfer that mixes packed and loose goods declares the packed goods' value
twice. A corrected behaviour would restrict the sum to the move lines that are not in a package,
mirroring the way the bulk weight itself is computed.

**DSH-054** *(compatibility finding.)* The free-above-a-threshold waiver is skipped for the
rule-based kind when the order is priced, but not when the shipment is sent. A rule-based method
whose waiver flag was set programmatically therefore charges the customer on the order and records
a shipping cost of zero on the transfer. A corrected behaviour would apply the same exclusion in
both places, or remove it from both and let the interface decide what to offer.

**DSH-055** *(compatibility finding.)* The combination of the integration level `rate` with the
invoicing policy `real` is unreachable through the interface but reachable programmatically. Such a
method creates no shipment, so the zero-priced charge line it produces is never replaced and the
customer is never charged for carriage. A corrected behaviour would reject the combination with a
record constraint.

**DSH-056** *(compatibility finding.)* The fixed kind and the rule-based kind declare a cancellation
routine but leave it unimplemented, so calling it fails without a user-facing message. The
interface hides the cancellation button for those two kinds, so the failure is only reachable
programmatically. A corrected behaviour would refuse with a message such as *this delivery method
cannot cancel a shipment*.

**DSH-057** *(compatibility finding.)* The shipment-sending dispatcher returns nothing when the
method's provider kind declares no sending routine, and the caller immediately reads the first
element of that answer, which fails without a user-facing message. In practice every kind that can
reach the sending step declares a routine, and the two core kinds do. A corrected behaviour would
refuse with a message such as *this delivery method cannot create a shipment*.

**DSH-058** *(compatibility finding.)* A Package Type bound to a carrier displays no length unit,
because the carrier integration expresses the dimensions in its own unit and the system performs no
conversion. The dimensions sent to the carrier are therefore the raw numbers typed by the user,
interpreted in whatever unit that carrier uses. A corrected behaviour would store the unit alongside
the dimensions and convert.

**DSH-059** *(compatibility finding.)* Switching an existing Delivery Method to the in-store kind
forces the four in-store defaults but does not attach the company's warehouses, so the method
cannot be published until a store is added by hand, and no message explains why. A corrected
behaviour would attach the warehouses on the write as it does on the creation.

**DSH-060** Deleting the shipped "Deliveries" product category is refused with "You cannot delete
the deliveries product category as it is used on the delivery carriers products." The guard runs on
an ordinary deletion and is suspended while the contributing package is being removed.

**DSH-119** *(industry-standard default.)* The system does not state what happens when two carrier
integrations return the same tracking reference for two transfers of the same chain. The observed
behaviour appends the reference a second time separated by a comma only when the existing reference
does not already contain the returned one, and the containment test is a plain substring test, so a
reference that is a substring of another is treated as already present. Where a rebuild needs a
decision, treat the stored value as a comma-separated set of references, compare whole elements
rather than substrings, and add a returned reference only when no element equals it.

**DSH-120** *(industry-standard default.)* The system does not state a precision for the sale value
of a move line when the move has no sales order line. Where a rebuild needs a decision, round that
value to the company currency's decimal places, half up, which is what the sales-order-line branch
already does with the line currency.

**DSH-121** *(industry-standard default.)* The system does not state what a carrier integration
should do when the destination address has no telephone number, no electronic mail address or no
region while the carrier requires them. Where a rebuild needs a decision, refuse the shipment in
the compliance hook so that the refusal becomes a warning activity rather than a failed
transmission, and word the refusal so that it names the missing field.

---

## 15. Rule index

| Identifier | Subject |
|---|---|
| DSH-001 | A tag may not be both required and excluded |
| DSH-002 | A Delivery Method must carry a name |
| DSH-003 | A Delivery Method must carry a delivery product |
| DSH-004 | The proportional margin may not be below minus one hundred per cent |
| DSH-005 | The insurance percentage lies between 0 and 100 |
| DSH-006 | The provider kind is required |
| DSH-007 | The invoicing policy is required and its real value depends on a package |
| DSH-008 | Selecting the rate-only level resets the invoicing policy |
| DSH-009 | The return flags cascade downwards when cleared |
| DSH-010 | The five tests of the availability filter |
| DSH-011 | The country test |
| DSH-012 | The region test |
| DSH-013 | The postal-code test |
| DSH-014 | What the weight and volume tests read |
| DSH-015 | What the tag tests read |
| DSH-016 | Delivery Price Rules are not validated against one another |
| DSH-017 | The wizard requires a Delivery Method |
| DSH-018 | A rule's condition is required |
| DSH-019 | A rule's amounts are required |
| DSH-020 | A rule's factor variable is required |
| DSH-021 | No matching rule fails the rating |
| DSH-022 | Deleting a method deletes its rules |
| DSH-023 | The fixed kind ignores margins |
| DSH-024 | A kind with no rating routine fails |
| DSH-025 | The engines repeat only the address test |
| DSH-026 | An invoiced charge line cannot be replaced |
| DSH-027 | Uninvoiced charge lines are deleted, invoiced ones are kept |
| DSH-028 | Deleting the last charge line clears the method |
| DSH-029 | A charge line may be deleted from a confirmed order |
| DSH-030 | A charge line may not be invoiced alone |
| DSH-031 | A charge line carries no price-list item |
| DSH-032 | Locking and the real-charge write-back |
| DSH-033 | The recomputation flag |
| DSH-034 | Packing with different carriers is refused |
| DSH-035 | An invalid source document is refused |
| DSH-036 | A rule-based method must rate successfully to be offered online |
| DSH-037 | An archived method is never offered |
| DSH-038 | The waiver excludes the rule-based kind when pricing |
| DSH-039 | The waiver's comparison is inclusive |
| DSH-040 | The waiver's currency handling |
| DSH-041 | A published in-store method needs a store |
| DSH-042 | A store must share the method's company |
| DSH-043 | Company consistency of the method on a document |
| DSH-044 | Only shipping-selectable routes may be attached, and they feed procurement |
| DSH-045 | Duplication copies the rules and renames the copy |
| DSH-046 | A parcel-point method needs a chosen point |
| DSH-047 | The parcel-point method and address must agree |
| DSH-048 | The method's company mirrors the product's |
| DSH-049 | Clearing the countries clears the postal-code prefixes |
| DSH-050 | Changing the countries prunes the regions |
| DSH-051 | Compatibility finding: the declared-value conversion direction |
| DSH-052 | Compatibility finding: commodity quantities lost when splitting parcels |
| DSH-053 | Compatibility finding: the loose-goods parcel double-counts packed goods |
| DSH-054 | Compatibility finding: the waiver's asymmetry between pricing and shipping |
| DSH-055 | Compatibility finding: rate-only combined with real cost |
| DSH-056 | Compatibility finding: cancellation unimplemented for the core kinds |
| DSH-057 | Compatibility finding: sending with no routine fails silently |
| DSH-058 | Compatibility finding: container dimensions carry no unit |
| DSH-059 | Compatibility finding: switching to the in-store kind attaches no store |
| DSH-060 | The shipped delivery product category cannot be deleted |
| DSH-061 | The taxes of the charge line |
| DSH-062 | The free-shipping annotation |
| DSH-063 | The estimated-cost annotation |
| DSH-064 | A rate-and-ship method makes the contact required on a transfer |
| DSH-065 | The method is frozen on a done or cancelled transfer |
| DSH-066 | A shipment is never created twice |
| DSH-067 | No shipment for an incoming transfer |
| DSH-068 | No shipment without the shipping-label flag |
| DSH-069 | A refusal before any success aborts the validation |
| DSH-070 | A refusal after a success becomes a warning activity |
| DSH-071 | The rule-based sending routine's address refusal |
| DSH-072 | A return transfer carries no carrier |
| DSH-073 | Which transfers receive a method added after confirmation |
| DSH-074 | What cancelling a shipment does and does not undo |
| DSH-075 | Opening a tracking page without a link is refused |
| DSH-076 | A carrier forces the packing dialogue |
| DSH-077 | The packing dialogue's restrictions |
| DSH-078 | The too-heavy warning |
| DSH-079 | Packing without the dialogue still sets a shipping weight |
| DSH-080 | A weightless order cannot be parcelled |
| DSH-081 | A weightless transfer cannot be parcelled |
| DSH-082 | A commodity's quantity floor |
| DSH-083 | A commodity's country of origin fallback |
| DSH-084 | No collection point available |
| DSH-085 | A refusal from a close-locations routine becomes an error message |
| DSH-086 | A postal code needs a country |
| DSH-087 | A pickup-point address is never a default delivery address |
| DSH-088 | A parcel-point address is not editable by the customer |
| DSH-089 | The in-store defaults applied on creation |
| DSH-090 | The in-store defaults applied on a later write |
| DSH-091 | No store chosen at payment |
| DSH-092 | The chosen store is short at payment |
| DSH-093 | The final readiness check |
| DSH-094 | The per-line shortage warning |
| DSH-095 | Which products the stock check skips |
| DSH-096 | The stock check rounds downwards |
| DSH-097 | Several lines of the same product |
| DSH-098 | Express checkout excludes location-based methods |
| DSH-099 | The availability widget and excluded tags |
| DSH-100 | Cash on delivery is filtered by the method |
| DSH-101 | Pay on site is filtered by the method and the content |
| DSH-102 | Pay on site is refused for a method that is not in-store |
| DSH-103 | A pending transaction of either mode confirms the order |
| DSH-104 | Both payment methods are pay-later methods |
| DSH-105 | Batching groups by carrier |
| DSH-106 | The three batch weight caps |
| DSH-107 | The batch description carries the carrier's name |
| DSH-108 | Who may read a Delivery Method |
| DSH-109 | Who may write a Delivery Method |
| DSH-110 | Who may read and write a Delivery Price Rule |
| DSH-111 | Who may read and write a Delivery Postal Code Prefix |
| DSH-112 | Who may use the selection wizard |
| DSH-113 | The multi-company record rule |
| DSH-114 | The parcel-point container code is restricted to the settings group |
| DSH-115 | The shipment step runs with elevated rights |
| DSH-116 | The rule-based collection runs with elevated rights |
| DSH-117 | The weight computations run with elevated rights |
| DSH-118 | The charge line is created with elevated rights |
| DSH-119 | Industry-standard default: comparing tracking references |
| DSH-120 | Industry-standard default: rounding a move line's sale value |
| DSH-121 | Industry-standard default: refusing an incomplete destination address |

---

## 16. Rules this domain relies on but does not own

| Rule | Owning domain |
|---|---|
| A Transfer may not be validated without quantities, and the backorder question | [`../inventory-operations/`](../inventory-operations/) |
| A Sales Order may not be confirmed without lines | [`../sales/`](../sales/) |
| A locked Sales Order refuses writes on its protected fields | [`../sales/`](../sales/) |
| A tax-inclusive price is adapted when a fiscal position maps its taxes | [`../taxes/`](../taxes/) |
| A quantity is converted between two units of the same category | [`../units-of-measure-and-packaging/`](../units-of-measure-and-packaging/) |
| An amount is converted between two currencies at a dated rate | [`../multi-currency/`](../multi-currency/) |
| A price list returns a product's price for a quantity | [`../pricing-and-pricelists/`](../pricing-and-pricelists/) |
| An address is geolocated into coordinates | [`../contacts-and-organizations/`](../contacts-and-organizations/) |
| A payment transaction moves between its states | [`../payment-providers/`](../payment-providers/) |
| A cart must be ready before payment | [`../website-and-storefront/`](../website-and-storefront/) |
