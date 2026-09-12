# Glossary

Every term this folder uses in a sense that is specific to the Delivery and Shipping domain, or
that a reader coming from another domain would read differently. Terms owned by another domain are
defined only far enough to be used here, and the owning domain is named.

Entries are alphabetical. A reproduced identifier is given in code font beside the term it belongs
to.

---

**Availability filter.** The five tests — destination address, required tags, excluded tags,
maximum weight, maximum volume — that decide whether a Delivery Method may be offered for a given
Sales Order or Transfer. Failing the filter is never an error: the method is simply not listed.
Specified in [calculations.md](calculations.md) section 1.

**Base amount** (`list_base_price`, on-screen label "Sale Base Price"). The constant part of the
charge a Delivery Price Rule produces. Expressed in the Delivery Method's currency.

**Batch Transfer.** A group of Transfers prepared together, owned by
[`../inventory-operations/`](../inventory-operations/). This domain adds two grouping decisions to
it: group only transfers of the same Delivery Method, and cap the batch's total weight.

**Brand code** (`mondialrelay_brand`). The account identifier a parcel-point network issues to a
seller. It is sent to the network's selector widget and appears in the network's tracking address.
Its shipped default is the reproduced test value `BDTEST  `, which includes two trailing spaces.

**Bulk weight.** The weight of the goods of a Transfer that are in no package. Computed by
[`../inventory-operations/`](../inventory-operations/) and used here to build the loose-goods parcel
and to compute the shipping weight.

**Carriage.** The service of moving goods from the seller to the customer. This folder uses
*carriage* for the service and *shipment* for one concrete instance of it handed to a carrier.

**Carrier.** The organisation that moves the goods. The system never stores a carrier as such: it
stores a Delivery Method, which is one offer from one carrier.

**Carrier code** (`shipper_package_code`). A carrier's own code for a container type, sent with
every parcel built from that type so that the carrier knows which box it is being asked to carry.

**Carrier integration.** A capability package that adds one value to the provider kind and
implements the six routines of [interfaces.md](interfaces.md) section 4, so that the system can ask
an external carrier's computer system for a price, for a shipment, for a tracking link, for a
cancellation, for a return label and for a default container code.

**Carrier kind of a package** (`package_carrier_type`). The value on a Package Type that reserves it
for one carrier integration. The reproduced value `none` means *no carrier integration*, and the
two core provider kinds both map onto it.

**Cash on delivery** (`cash_on_delivery`). A payment path in which the carrier collects the money
when the goods are handed over. Enabled per Delivery Method; the payment provider is hidden for an
order whose method does not allow it.

**Charge.** The amount the customer is asked to pay for carriage. It travels from the rating
pipeline to a Sales Order Line and from there to the customer invoice. Distinguished throughout
from the *cost*, which is what the carrier asks the seller for.

**Click and collect.** The informal name of collection in store. See *Collection in store*.

**Collection in store** (`in_store`). The provider kind in which the customer collects the goods
from one of the seller's own stores rather than having them carried. It brings a store list, a
distance sort, a per-store stock check, a warehouse and fiscal position taken from the chosen
store, and the pay-on-site payment path.

**Collection point.** A place other than the customer's own address where the goods are handed
over: a point of a carrier's network, or one of the seller's stores. Stored on a Sales Order as a
structured value and turned into a delivery address when the order is confirmed.

**Commodity.** See *Delivery Commodity*.

**Company currency.** The currency of the Delivery Method's company, and, when the method carries
no company, the currency of the main company. Every engine expresses its charge in it, and the
waiver's threshold is compared in it.

**Compliance hook.** The extension point a carrier integration uses to refuse a Transfer whose data
the carrier will not accept. It runs immediately after the shipment step, so its refusal is turned
into a warning activity rather than an aborted validation when another shipment of the same batch
has already been accepted.

**Container code.** See *Carrier code*.

**Container type.** This folder's name for the Package Type record owned by
[`../units-of-measure-and-packaging/`](../units-of-measure-and-packaging/): a reusable description
of a box, carrying dimensions, a base weight, a maximum weight, a carrier and a carrier code.

**Cost.** What the carrier asks the seller to pay. Stored on a Transfer as the shipping cost. It
becomes the customer's charge only when the Delivery Method's invoicing policy is the real cost.

**Declared value.** The value of the goods in a parcel, computed from the products' unit costs and
sent to the carrier for customs and insurance. It is never posted to the ledger.

**Delivery address.** The Contact to which the goods are sent. This domain reads its country, its
region and its postal code for the availability filter, and replaces it with a collection-point
address when the customer chose one.

**Delivery Commodity.** One product line inside a Delivery Parcel: the product, a whole-unit
quantity that is never below one, the exact quantity, the declared unit value and the country of
origin. It is a transport structure, not a stored record.

**Delivery message** (`delivery_message`). The warning the rating routine returned when the method
was chosen, kept on the Sales Order so that the salesperson can see why the charge is what it is.
The commonest value is the free-above-a-threshold notice.

**Delivery Method** (`delivery.carrier`, table `delivery_carrier`). The central record of this
domain: one purchasable carriage offer and, when it is an integration, the configuration of the
link to an external carrier's computer system. Called a *shipping method* in some screens.

**Delivery Package.** See *Delivery Parcel*.

**Delivery Parcel.** One physical parcel handed to a carrier integration: its weight, its
dimensions, its container code, its name, its declared value, its currency, its commodities and the
document it came from. It is a transport structure, not a stored record.

**Delivery Postal Code Prefix** (`delivery.zip.prefix`, table `delivery_zip_prefix`). One reusable
postal-code pattern that restricts the destinations a Delivery Method serves. Stored upper-cased,
unique, and used as a regular-expression fragment anchored at the start of the postal code.

**Delivery Price Rule** (`delivery.price.rule`, table `delivery_price_rule`). One line of the
ordered list the rule-based engine evaluates: a condition over one of five variables, and a charge
of the form *base amount plus factor amount times one of the five variables*.

**Delivery product** (`product_id`). The service product through which a Delivery Method charges.
It carries the price the fixed engine reads, the taxes the charge line carries, the income account
the invoice selects and the currency in which the method's amounts are expressed.

**Delivery slip.** The printable document that accompanies the goods, owned by
[`../inventory-operations/`](../inventory-operations/). This domain adds the carrier, the total
weight, the tracking reference, the customs code column and the per-package weights.

**Estimated cost.** The invoicing policy (`estimated`) in which the customer pays the charge
computed when the method was chosen, whatever the carrier later asks for.

**Estimated weight.** The weight of the goods of a Sales Order, summed from the line quantities and
the products' unit weights, skipping services, combinations, display lines, the shipping charge
line and lines whose quantity is not strictly positive.

**Exact quantity** (`real_qty`). The unrounded quantity of a Delivery Commodity, in the product's
reference unit, kept beside the whole-unit quantity for carriers that accept fractions.

**Excluded tag** (`excluded_tag_ids`). A Product Tag whose presence on any product of a document
makes a Delivery Method unavailable for it.

**Express checkout.** The storefront path in which a payment wallet supplies the address and the
delivery choice in one step. It excludes the in-store kind and every method that offers collection
points, because it offers no way to choose a location.

**Factor amount** (`list_price`, on-screen label "Sale Price"). The per-unit part of the charge a
Delivery Price Rule produces, multiplied by the factor variable.

**Factor variable** (`variable_factor`). Which of the five variables the factor amount is
multiplied by. It is chosen independently of the condition variable.

**Fixed kind** (`fixed`). The provider kind in which the charge is the price of the delivery
product under the order's price list. It ignores both margins.

**Fixed margin** (`fixed_margin`). An absolute amount added to the charge after the proportional
margin, converted from the company currency into the order currency. Ignored by the fixed kind.

**Free above a threshold.** See *Waiver*.

**Harmonized System code** (`hs_code`). The standardised customs classification of a product.
Printed on the delivery slip and on the product label, and sent to a carrier with the commodity
list.

**In-store kind.** See *Collection in store*.

**Integration level** (`integration_level`). Whether the system only asks the carrier for a price
(`rate`) or also creates the shipment when an outgoing Transfer is validated (`rate_and_ship`).

**Invoicing policy** (`invoice_policy`). Whether the customer is charged the estimate (`estimated`)
or the cost the carrier actually asked for (`real`). The second value exists only while the
Delivery – Inventory bridge is installed.

**Label prefix.** One of three reproduced naming conventions for the documents a carrier
integration attaches to a Transfer: `LabelShipping-` for a shipping label, `LabelReturn-` for the
document the customer uses to send goods back, and `ShippingDoc-` for an accompanying document,
each followed by the provider kind. The return-label list of a Transfer searches attachments by the
second prefix.

**Loose goods.** The goods of a Transfer that are in no package. They become one Delivery Parcel
named with the reproduced value `Bulk Content`.

**Margin.** See *Fixed margin* and *Proportional margin*.

**Maximum value** (`max_value`). The comparison value of a Delivery Price Rule's condition. Despite
the name it is a lower bound whenever the operator is `>=` or `>`.

**Order currency.** The currency of the Sales Order's price list. The charge is rounded by it and
written on the charge line in it.

**Order total without carriage.** The Sales Order's total including taxes minus the tax-inclusive
totals of its shipping charge lines. The quantity the waiver compares against its threshold and the
quantity the rule-based engine's price variable holds.

**Pay on site** (`pay_on_site`, provider mode `on_site`). The payment path in which the customer
pays at the counter when collecting. Offered only for an order carried by the in-store kind and
containing goods.

**Package.** A physical container of goods inside the warehouse, owned by
[`../inventory-operations/`](../inventory-operations/). This domain adds its weight, its weight unit
label, whether that unit is the kilogram, that unit's rounding step and its carrier kind.

**Parcel.** See *Delivery Parcel*. A parcel is what is handed to the carrier; a Package is what the
warehouse holds. A parcel is built from a package, from the loose goods of a transfer, or from a
whole order.

**Parcel point.** A collection point of a carrier's own network, as opposed to one of the seller's
stores.

**Parcel-point network.** The example collection-point integration this folder describes: a
companion package that marks a Delivery Method by the reproduced code `MR` on its delivery product,
adds a brand code and a container code, embeds the network's own selector widget, builds the
network's tracking address, and creates a delivery address whose external reference begins with the
reproduced prefix `MR#`.

**Pickup location.** See *Collection point*.

**Pickup-point address** (`is_pickup_location`). A Contact created from a collection point. It is
excluded from the customer's selectable delivery addresses and is never chosen as the default
delivery address of an order.

**Production environment** (`prod_environment`). Whether a carrier integration talks to the
carrier's production service or to its test service. Cleared on every method when a copy of a
database is neutralised.

**Proportional margin** (`margin`). A fraction added to the charge, shown on screen as a
percentage. Constrained to be at least minus one. Ignored by the fixed kind.

**Provider kind** (`delivery_type`). Which pricing engine, which sending routine, which tracking
routine, which cancellation routine and which return-label routine apply. Two values are core,
`fixed` and `base_on_rule`; one is added by the collection-in-store package, `in_store`; each
carrier integration adds one of its own.

**Rate.** The four-member answer a rating routine returns: whether it succeeded, a charge, an error
message and a warning message. The pipeline adds a fifth member, the charge before the waiver.

**Rate request.** The act of asking a Delivery Method what it would charge for a given Sales Order.
It runs the whole pipeline of [calculations.md](calculations.md) section 7.

**Real cost.** The invoicing policy (`real`) in which the charge line is created at zero and the
cost the carrier asked for at shipment time replaces it. Each further shipment of the same order
produces a further charge line.

**Recomputation flag** (`recompute_delivery_price`). The marker set on a Sales Order when its
basket, its customer or its delivery address changed after the charge was computed. It highlights
the charge line and the update button in amber and blocks nothing.

**Reference unit.** A product's own unit of measure, into which every quantity is converted before
being multiplied by a unit weight, a unit volume or a unit value.

**Required tag** (`must_have_tag_ids`). A Product Tag of which at least one must be present on the
products of a document for the Delivery Method to be available. A method that requires no tag is
available to every content.

**Return label.** A carrier document that lets the customer send the goods back. Produced
automatically at shipment time when the method asks for it, or on demand from an incoming return
transfer, and optionally published to the customer portal with an access token.

**Rule-based kind** (`base_on_rule`). The provider kind in which the charge is produced by the
ordered Delivery Price Rules. Margins apply; the waiver does not, at pricing time.

**Selection wizard.** See *Delivery Method Selection Wizard* in [entities.md](entities.md)
section 4. The dialogue through which a salesperson chooses a Delivery Method, requests its rate
and writes the shipping charge line.

**Sending routine.** The routine a carrier integration supplies to create a shipment. It returns one
result per Transfer, each carrying the exact charge and the tracking reference, and it attaches the
labels itself.

**Shipment.** One concrete handover of goods to a carrier, identified by a tracking reference and
paid for by the shipping cost. A Transfer carries at most one shipment at a time; cancelling it
frees the transfer to carry another.

**Shipping charge line.** The Sales Order Line that carries the carriage charge. Marked by
`is_delivery`, priced by the rating pipeline, taxed by the delivery product's taxes, excluded from
price-list recomputation and deletable even from a confirmed order.

**Shipping cost** (`carrier_price`). What the carrier asked for one Transfer, after the
shipment-time waiver and after the margins. Never posted to the ledger.

**Shipping insurance** (`shipping_insurance`). The percentage of the declared value the carrier is
asked to insure. Between 0 and 100.

**Shipping weight.** The weight actually handed to a carrier: the loose goods plus, for each
outermost package, the weight typed on it or, failing that, the weight it computes. Writable, so an
operator may override it.

**Shipping-label flag** (`print_label`). The Operation Type setting that decides whether validating
a Transfer creates the shipment. Set by default on outgoing operation types, cleared on incoming
and internal ones.

**Store.** A Warehouse listed on an in-store Delivery Method as a place the customer may collect
from. It brings its address, its coordinates, its opening hours, its stock and, through its
address, the order's fiscal position.

**Tracking link** (`tracking_url` on the method, `carrier_tracking_url` on the transfer). The
address at which the customer may follow the shipment. Built by substituting the tracking reference
into the method's pattern, or by the carrier integration's own routine. A carrier may return a list
of label-and-link pairs instead of a single link.

**Tracking reference** (`carrier_tracking_ref`). The identifier the carrier issued for the
shipment. Propagated along every transfer of the same chain, and accumulated as a comma-separated
list when a transfer is covered by more than one shipment.

**Transfer** (`stock.picking`). The record that moves goods from one place to another, owned by
[`../inventory-operations/`](../inventory-operations/). This domain adds the carrier, the shipping
cost, the tracking reference, the tracking link, the weight, the return flag and the return-label
list to it.

**Variable.** One of the five quantities a Delivery Price Rule may test or multiply by: `weight`,
`volume`, `wv` (weight times volume), `price` and `quantity`. The first four are summed over the
order's goods lines; the price is the order total without carriage, converted into the company
currency.

**Waiver.** The rule that sets the charge to zero when the order total without carriage reaches the
Delivery Method's threshold. Configured by the waiver flag (`free_over`) and the threshold
(`amount`). Applied at pricing time to every kind but the rule-based one, and at shipment time to
every kind.

**Warning activity.** The follow-up the system creates when a carrier refuses a shipment after
another shipment of the same validation batch has already been accepted. Dated today, assigned to
the transfer's responsible user, carrying the carrier's refusal text.

**Weight system parameter** (`product.weight_in_lbs`). The parameter that chooses the unit in which
every weight of this domain is expressed. The reproduced value `1` selects the pound; anything else
selects the kilogram. Owned by
[`../units-of-measure-and-packaging/`](../units-of-measure-and-packaging/).

**Weight times volume** (`wv`). The variable that accumulates, per line, the product's unit weight
times its unit volume times the line's reference-unit quantity. When the accumulation is zero it
falls back to the total volume times the total weight.

**Whole-unit quantity** (`qty`). The quantity of a Delivery Commodity rounded to a whole number and
never below one. Carriers that only accept whole units read it; the exact quantity is kept beside
it.

**Volume system parameter** (`product.volume_in_cubic_feet`). The parameter that chooses the unit
in which every volume of this domain is expressed, and the unit in which a container type's
dimensions are typed. The reproduced value `1` selects the cubic foot and the foot; anything else
selects the cubic metre and the millimetre. Owned by
[`../units-of-measure-and-packaging/`](../units-of-measure-and-packaging/).

---

## Terms this folder deliberately avoids

| Avoided | Used instead | Why |
|---|---|---|
| Carrier, as a record | Delivery Method | The stored record is one offer, not one organisation; a carrier normally has several |
| Shipping method | Delivery Method | The two names appear in different screens for the same record; this folder uses one |
| Picking | Transfer | The record's canonical name in this repository |
| Packaging | Container type, or Package | The word is used in the sources for two different records |
| Zip | Postal code | An abbreviation, except inside a reproduced identifier |
| Free shipping | Waiver, or a waived charge | *Free shipping* is a reproduced annotation on a charge line, not the name of the mechanism |
