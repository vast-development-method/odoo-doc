# Delivery and Shipping

## Scope

This domain specifies how the system decides **who carries the goods, what the customer is charged for carriage, and what is exchanged with the carrier's computer system**.

Concretely it covers:

- the Delivery Method record, which is both a commercial offer ("Standard delivery, nine euros ninety-five") and the configuration of an integration with an external carrier's computer system;
- the two pricing engines shipped in the core: the **fixed charge** engine and the **rule-based** engine, the latter with its ordered rule list, its five rule variables, its five comparison operators, its base-plus-factor charge formula, and the exact order in which the free-above-a-threshold waiver, the percentage margin, the fixed margin, the tax-inclusive price adaptation and the currency conversion are applied;
- the **availability filter** that decides whether a Delivery Method may be offered at all for a given destination address and a given basket (countries, regions, postal-code prefixes, maximum weight, maximum volume, required product tags, excluded product tags);
- the **contract that an external carrier integration must satisfy**: rate request, shipment creation with label production, tracking-link construction, shipment cancellation, return-label production, and the default container-code lookup — each one stated as what is sent, what comes back, and what happens when it fails;
- the **shipping charge line** that is added to a sales order, how it is taxed, when it is recomputed, when it may be deleted, and how it is invoiced;
- the **package model used for shipping**: how the goods of an order or of a transfer are grouped into one or more parcels, how each parcel's weight, dimensions, container code, declared value and contained commodities are derived, and how parcels are split when a container type has a maximum weight;
- the **shipment sending step** performed when an outgoing transfer is validated: which transfers trigger it, what is stored back on the transfer (real charge, tracking reference), how the tracking reference is propagated along the chain of transfers, what is written in the transfer's message history, and how a failure is turned into a warning activity instead of an aborted validation;
- **real-cost invoicing**: how the charge returned by the carrier replaces the estimate on the sales order;
- **pickup points**: the generic contract by which a Delivery Method offers a list of collection points near an address, how one is chosen, how it becomes a delivery address on the order, and the parcel-point network companion that implements it;
- **collection from a store** ("click and collect"): the in-store Delivery Method, the warehouse list attached to it, the distance sort, the per-store stock check, the fiscal position taken from the store, and the pay-on-site payment path;
- **cash on delivery** as a payment path enabled per Delivery Method;
- **batch delivery**: grouping outgoing transfers into batches by carrier and capping a batch by total weight;
- the **carrier propagation rules** that copy a Delivery Method and a tracking reference from one transfer of a multi-step delivery route to the next;
- the printable documents, portal pages, named remote operations, routes, security groups and access rights that belong to all of the above.

Everything in this folder is derived from the behavior of the following capability packages: the core **Delivery Costs** package (delivery methods, price rules, postal-code prefixes, the method-selection wizard, the cash-on-delivery payment path, the partner default method), the **Delivery – Inventory bridge** package (shipment sending, labels, tracking, packages for shipping, return labels, real-cost invoicing, weight on moves and transfers), the **parcel-point network companion** package (an example pickup-point integration), the **Delivery – Batch Transfers bridge** package (batching by carrier and by weight), the **Collection in store** package (in-store delivery methods, store stock checks, pay on site), and the delivery-method hooks that the **Sales – Inventory bridge** contributes (carrier propagation on rules, route selection from the method).

The domain does **not** cover:

| Subject | Owning domain |
|---|---|
| Transfers themselves, their states, reservation, validation, backorders, put in pack, packages, package types | [`../inventory-operations/`](../inventory-operations/) |
| Weight and volume fields on products, the weight unit of measure system parameter, unit conversion arithmetic, package type dimensions and base weight | [`../units-of-measure-and-packaging/`](../units-of-measure-and-packaging/) |
| Sales orders, their state machine, order lines, invoicing policy, the invoice created from an order | [`../sales/`](../sales/) |
| Tax computation, tax-included and tax-excluded price adaptation, fiscal position mapping | [`../taxes/`](../taxes/) |
| Price list price lookup used by the fixed charge engine | [`../pricing-and-pricelists/`](../pricing-and-pricelists/) |
| Currency rates and the conversion routine | [`../multi-currency/`](../multi-currency/) |
| Payment providers, payment methods, payment transactions and their state machine | [`../payment-providers/`](../payment-providers/) |
| The storefront cart, checkout steps, address forms and express checkout mechanics | [`../website-and-storefront/`](../website-and-storefront/) |
| Customer invoices and the journal entries they produce | [`../accounts-receivable/`](../accounts-receivable/) |
| Warehouses, their addresses and their locations | [`../inventory-operations/`](../inventory-operations/) |
| Product tags, product templates and product variants | [`../products-and-catalog/`](../products-and-catalog/) |

Where this domain has to name a mechanism owned by another domain — a unit conversion, a tax mapping, a currency conversion, the validation of a transfer — it states exactly which inputs it passes and which result it expects, and links to the owning domain rather than restating the rule.

## Capabilities covered

| Capability | Where specified |
|---|---|
| Delivery Method: identity, provider kind, integration level, environment flag, debug logging flag, company scoping, linked delivery product | [`entities.md`](entities.md), [`configuration.md`](configuration.md) |
| Availability filter: countries, regions, postal-code prefixes with their regular-expression semantics, maximum weight, maximum volume, required tags, excluded tags | [`calculations.md`](calculations.md), [`business-rules.md`](business-rules.md) |
| Fixed charge engine: the price list lookup, the absence of margins, the free-above-threshold waiver | [`calculations.md`](calculations.md) |
| Rule-based charge engine: variable collection, rule ordering, operator evaluation, first-match rule, base-plus-factor formula, the no-match error | [`calculations.md`](calculations.md) |
| The complete charge pipeline: engine, tax-inclusive adaptation, percentage margin, fixed margin, rounding, waiver above a threshold, currency conversion at each step | [`calculations.md`](calculations.md) |
| Rate request contract for external carrier integrations: inputs, the result record, the four result keys, warning handling, failure handling | [`workflows.md`](workflows.md), [`interfaces.md`](interfaces.md) |
| Shipment sending contract: inputs, the per-transfer result record, real charge, tracking reference, label attachment, message posted, failure turned into an activity | [`workflows.md`](workflows.md), [`interfaces.md`](interfaces.md), [`business-rules.md`](business-rules.md) |
| Tracking-link contract and the placeholder substitution; multiple tracking references on one transfer | [`calculations.md`](calculations.md), [`interfaces.md`](interfaces.md) |
| Shipment cancellation contract and the voiding of the tracking reference | [`workflows.md`](workflows.md), [`state-machines.md`](state-machines.md) |
| Return-label contract, the attachment naming prefixes, and portal access to the return label | [`workflows.md`](workflows.md), [`interfaces.md`](interfaces.md) |
| Parcel construction from an order and from a transfer: splitting by maximum container weight, commodity lists, declared values, country of origin, Harmonized System code | [`calculations.md`](calculations.md) |
| The shipping charge line on a sales order: creation, naming, taxes, sequence, free-shipping annotation, deletion rules, recomputation triggers | [`workflows.md`](workflows.md), [`business-rules.md`](business-rules.md) |
| Estimated order weight and the shipping weight override used for rating | [`calculations.md`](calculations.md) |
| Real-cost invoicing: zero-priced estimate line, replacement at shipment time, protected-field bypass | [`workflows.md`](workflows.md), [`accounting-effects.md`](accounting-effects.md) |
| Carrier propagation across multi-step delivery routes and the grouping of moves into transfers by carrier | [`calculations.md`](calculations.md), [`workflows.md`](workflows.md) |
| Pickup points: the two named remote operations, the location record shape, the delivery address created at confirmation | [`interfaces.md`](interfaces.md), [`workflows.md`](workflows.md) |
| Collection in store: store list, distance sort, opening hours, per-store stock check, warehouse and fiscal position selection, pay on site, the blocking checkout errors | [`workflows.md`](workflows.md), [`calculations.md`](calculations.md), [`business-rules.md`](business-rules.md) |
| Cash on delivery: the payment method, the provider, the compatibility filter, order confirmation on a pending transaction | [`workflows.md`](workflows.md), [`configuration.md`](configuration.md) |
| Batch delivery: grouping key by carrier, maximum batch weight, wave weight | [`calculations.md`](calculations.md), [`configuration.md`](configuration.md) |
| Package weights for shipping: the computed weight, the shipping weight override, the maximum-weight warning, the weight in the parcel barcode | [`calculations.md`](calculations.md), [`interfaces.md`](interfaces.md) |
| Printable documents: the delivery slip additions, the transfer document additions, the parcel barcode label additions, the quotation shipping description | [`interfaces.md`](interfaces.md) |
| Shipped records, security groups, access rights, record rules, the neutralisation statements | [`configuration.md`](configuration.md) |
| Accounting consequences: this domain posts nothing itself; it feeds the sales order and therefore the customer invoice | [`accounting-effects.md`](accounting-effects.md) |

## Entities

| Entity | Transport name | Storage name | Purpose |
|---|---|---|---|
| Delivery Method | `delivery.carrier` | `delivery_carrier` | One purchasable carriage offer and, when it is an integration, the configuration of the link to an external carrier's computer system |
| Delivery Price Rule | `delivery.price.rule` | `delivery_price_rule` | One ordered condition-and-charge line of the rule-based pricing engine of one Delivery Method |
| Delivery Postal Code Prefix | `delivery.zip.prefix` | `delivery_zip_prefix` | One reusable postal-code prefix pattern that restricts the destinations a Delivery Method serves |
| Delivery Method Selection Wizard | `choose.delivery.carrier` | `choose_delivery_carrier` | Transient record used by a salesperson to pick a Delivery Method, request its rate and add the shipping charge line to a sales order |

Two further structures are not stored records but are part of the contract every carrier integration must honour; they are specified in `entities.md` alongside the stored entities:

| Structure | Purpose |
|---|---|
| Delivery Parcel | The description of one physical parcel handed to a carrier integration: weight, dimensions, container code, name, declared value, currency, contained commodities, and the source document |
| Delivery Commodity | The description of one product line inside a parcel: product, whole-unit quantity, exact quantity, declared unit value, country of origin |

The domain also adds fields to entities owned by other domains. They are listed in full in `entities.md` and summarised here:

| Host entity | Owning domain | Fields added by this domain |
|---|---|---|
| Sales Order (`sale.order`) | [`../sales/`](../sales/) | `carrier_id`, `delivery_message`, `delivery_set`, `recompute_delivery_price`, `is_all_service`, `shipping_weight`, `pickup_location_data` |
| Sales Order Line (`sale.order.line`) | [`../sales/`](../sales/) | `is_delivery`, `product_qty`, `recompute_delivery_price` |
| Transfer (`stock.picking`) | [`../inventory-operations/`](../inventory-operations/) | `carrier_id`, `carrier_price`, `carrier_tracking_ref`, `carrier_tracking_url`, `delivery_type`, `integration_level`, `allowed_carrier_ids`, `weight`, `weight_uom_name`, `is_return_picking`, `return_label_ids`, `destination_country_code` |
| Stock Move (`stock.move`) | [`../inventory-operations/`](../inventory-operations/) | `weight` |
| Stock Move Line (`stock.move.line`) | [`../inventory-operations/`](../inventory-operations/) | `sale_price`, `destination_country_code`, `carrier_id` |
| Package (`stock.package`) | [`../inventory-operations/`](../inventory-operations/) | `weight`, `weight_uom_name`, `weight_is_kg`, `weight_uom_rounding`, `package_carrier_type` |
| Package Type (`stock.package.type`) | [`../units-of-measure-and-packaging/`](../units-of-measure-and-packaging/) | `shipper_package_code`, `package_carrier_type` |
| Operation Type (`stock.picking.type`) | [`../inventory-operations/`](../inventory-operations/) | `batch_group_by_carrier`, `batch_max_weight`, `weight_uom_name` |
| Route (`stock.route`) | [`../inventory-operations/`](../inventory-operations/) | `shipping_selectable` |
| Product Template (`product.template`) | [`../products-and-catalog/`](../products-and-catalog/) | `hs_code`, `country_of_origin` |
| Contact (`res.partner`) | [`../contacts-and-organizations/`](../contacts-and-organizations/) | `property_delivery_carrier_id`, `is_pickup_location` |
| Warehouse (`stock.warehouse`) | [`../inventory-operations/`](../inventory-operations/) | `opening_hours` |
| Website (`website`) | [`../website-and-storefront/`](../website-and-storefront/) | `in_store_dm_id` |
| Put-in-pack Wizard (`stock.put.in.pack`) | [`../inventory-operations/`](../inventory-operations/) | `shipping_weight`, `weight_uom_name`, `package_carrier_type` |

## Reading order

1. **[`README.md`](README.md)** (this file) — the map.
2. **[`entities.md`](entities.md)** — the Delivery Method, the Delivery Price Rule, the Delivery Postal Code Prefix, the selection wizard, the two parcel structures, and every field added to entities owned elsewhere. Read the Delivery Method field table before anything else; almost every rule in the other files is a rule about one of its fields.
3. **[`calculations.md`](calculations.md)** — **the heart of the domain.** The availability filter, the two pricing engines, the complete charge pipeline with its evaluation order, the estimated weight, the parcel-splitting algorithm, the tracking-link construction, the store distance sort, the store stock check, the batch weight caps. Every formula has a worked numeric example.
4. **[`state-machines.md`](state-machines.md)** — the shipment lifecycle of a transfer (no carrier, carrier assigned, sent to carrier, tracked, cancelled), the shipping-charge lifecycle on a sales order, the pickup-location lifecycle, and the provider-kind and integration-level configuration states.
5. **[`workflows.md`](workflows.md)** — the end-to-end procedures: adding a shipping charge from the back office, choosing a method in the storefront, choosing a pickup point, confirming an order that has a pickup point, validating an outgoing transfer and sending the shipment, printing a return label, cancelling a shipment, invoicing the delivery, collecting in a store, paying cash on delivery, and batching by carrier.
6. **[`business-rules.md`](business-rules.md)** — every validation, every constraint, every exact error message, every permission check and every edge case.
7. **[`interfaces.md`](interfaces.md)** — menus, views, the named remote operations of each integration contract, the routes, the printable documents and the portal fragments.
8. **[`accounting-effects.md`](accounting-effects.md)** — why this domain posts no journal entry of its own and exactly how it changes the entries that other domains post.
9. **[`configuration.md`](configuration.md)** — shipped records, settings, groups, the access-rights matrix, record rules, and the neutralisation statements.
10. **[`acceptance-criteria.md`](acceptance-criteria.md)** — one hundred and eighty-two numbered Given/When/Then scenarios with concrete numbers, including the eight mandatory ones listed below.
11. **[`glossary.md`](glossary.md)** — every term.

## The eight reference scenarios

These eight cases are specified end to end and appear as numbered scenarios in [`acceptance-criteria.md`](acceptance-criteria.md). They are the acceptance gate for a reimplementation of this domain.

| # | Scenario | Primary file |
|---|---|---|
| 1 | A fixed charge of nine point nine five is added to a quotation and taxed | [`acceptance-criteria.md`](acceptance-criteria.md) §A |
| 2 | A weight-banded charge is computed for a shipment of seven point five kilograms | [`acceptance-criteria.md`](acceptance-criteria.md) §B |
| 3 | A charge is waived because the order total without delivery reaches one hundred | [`acceptance-criteria.md`](acceptance-criteria.md) §C |
| 4 | A rate is requested from an external carrier integration and applied to the order | [`acceptance-criteria.md`](acceptance-criteria.md) §D |
| 5 | A label is produced at transfer validation and a tracking reference is stored | [`acceptance-criteria.md`](acceptance-criteria.md) §E |
| 6 | A shipment is cancelled and the label voided | [`acceptance-criteria.md`](acceptance-criteria.md) §F |
| 7 | The delivery is charged on the customer invoice | [`acceptance-criteria.md`](acceptance-criteria.md) §G |
| 8 | An order is collected from a store after a per-store stock check | [`acceptance-criteria.md`](acceptance-criteria.md) §H |

## Dependencies on other domains

### This domain needs

| From | What it needs |
|---|---|
| [`../products-and-catalog/`](../products-and-catalog/) | A product of kind service to carry the charge; product weight, volume, tags, country of origin |
| [`../units-of-measure-and-packaging/`](../units-of-measure-and-packaging/) | The weight and volume unit-of-measure system parameters, the conversion of a line quantity to the product's reference unit, package types with their dimensions, base weight and maximum weight |
| [`../pricing-and-pricelists/`](../pricing-and-pricelists/) | The price of the delivery product under the order's price list, used by the fixed charge engine |
| [`../taxes/`](../taxes/) | The taxes of the delivery product, the fiscal position mapping, and the adaptation of a price from one tax set to another |
| [`../multi-currency/`](../multi-currency/) | Conversion between the company currency and the order currency at the order date |
| [`../sales/`](../sales/) | The sales order, its lines, its totals, its confirmation, its invoicing |
| [`../inventory-operations/`](../inventory-operations/) | Transfers, their validation, their move lines, packages, the put-in-pack operation, return transfers, the multi-step delivery routes and their rules |
| [`../contacts-and-organizations/`](../contacts-and-organizations/) | The delivery address, its country, region and postal code; the creation of a child delivery address for a pickup point; geolocation coordinates |
| [`../payment-providers/`](../payment-providers/) | The custom payment provider mechanism used by cash on delivery and pay on site |

### This domain provides

| To | What it provides |
|---|---|
| [`../sales/`](../sales/) | The shipping charge line, the delivery method on the order, the estimated shipping weight, the shipping description printed on the quotation, and the recomputation trigger when the basket changes |
| [`../inventory-operations/`](../inventory-operations/) | The carrier on a transfer, the weight of a move and of a transfer, the shipping weight of a package, the tracking reference, the send-to-shipper step at validation, and the carrier grouping key used when moves are assigned to transfers |
| [`../accounts-receivable/`](../accounts-receivable/) | A taxable order line whose amount reaches the customer invoice like any other line |
| [`../website-and-storefront/`](../website-and-storefront/) | The list of delivery methods offered at checkout, their rates, the pickup-location selector data and the click-and-collect availability widget data |
| [`../payment-providers/`](../payment-providers/) | The compatibility filters that hide the cash-on-delivery provider and the pay-on-site provider when the chosen method does not allow them |
| [`../replenishment-and-procurement/`](../replenishment-and-procurement/) | The routes selected by a Delivery Method, which are pushed into the procurement values of every order line |

## Files in this folder

| File | Contents |
|---|---|
| [`README.md`](README.md) | This file: the scope, the capabilities, the entities, the reading order, the dependencies and the conventions |
| [`entities.md`](entities.md) | Every entity in full, the two transport structures of the carrier contract, and every field added to entities owned elsewhere |
| [`state-machines.md`](state-machines.md) | The seven lifecycles of the domain, each with its states, its transition table, its guards and a diagram |
| [`workflows.md`](workflows.md) | Twenty end-to-end procedures, step by step, with the records each step creates or changes |
| [`business-rules.md`](business-rules.md) | Every validation, constraint, permission and message, numbered from DSH-001, with an index |
| [`calculations.md`](calculations.md) | Twenty formulas and algorithms with their rounding rules and worked numeric examples |
| [`accounting-effects.md`](accounting-effects.md) | Why the domain posts no journal entry of its own, and the four entries it changes elsewhere |
| [`configuration.md`](configuration.md) | Settings, parameters, shipped records, groups, access rights, record rules and the neutralisation statements |
| [`interfaces.md`](interfaces.md) | Menus, screens, named operations, the carrier-integration contract, routes, printable documents and portal fragments |
| [`acceptance-criteria.md`](acceptance-criteria.md) | One hundred and eighty-two numbered Given / When / Then scenarios with concrete numbers |
| [`glossary.md`](glossary.md) | Every term of the domain, defined |

There is no extra topic file in this folder: the eleven documents above are the whole deliverable.

## Conventions used in this folder

- Entity names are written in full words and in title case: Delivery Method, Delivery Price Rule, Sales Order Line, Transfer, Package Type. On first mention in each file the transport name and storage name are given in code font.
- Field names, selection values and route paths are reproduced exactly in code font because external contracts depend on them; each is accompanied by its full name in words on first use in a document.
- Formulas are plain mathematics in fenced blocks labelled `formula`, with every quantity named in words, an explicit rounding rule, and at least one worked numeric example.
- Monetary amounts in examples use a currency whose smallest unit is one hundredth, and a rounding of one hundredth, unless a scenario says otherwise. Weights are in kilograms and volumes in cubic metres unless a scenario says otherwise.
- Error messages are reproduced exactly as the system produces them, with placeholders translated into words.
