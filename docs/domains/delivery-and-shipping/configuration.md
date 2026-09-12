# Configuration

Everything a running system carries for this domain that is not a business transaction: the
settings a user can change, the system parameters, the records shipped with the capability
packages, the demonstration records, the security groups, the access-rights matrix, the record
rules, the scheduled jobs, the message templates and the statements that make a copy of a database
safe to work on.

| # | Subject |
|---|---|
| 1 | Settings and system parameters |
| 2 | Records shipped with the packages |
| 3 | Demonstration records |
| 4 | Security groups |
| 5 | Access rights and record rules |
| 6 | Scheduled jobs |
| 7 | Message templates and activity types |
| 8 | Sequences |
| 9 | Neutralising a copy of a database |
| 10 | Configuration checklist for a working installation |

---

## 1. Settings and system parameters

### 1.1 Settings the user sees

| Setting | Where | Effect |
|---|---|---|
| Delivery Methods | Sales configuration → Delivery Methods | Opens the list of Delivery Methods. A button in the sales settings page opens the same list under the label "Shipping Methods", shown only while the Delivery Costs package is installed. |
| Delivery Methods | Inventory configuration → Delivery Methods | The same list, reached from the warehouse side. |
| Postal Code Prefix | Inventory configuration → Postal Code Prefix | The list of Delivery Postal Code Prefixes. Visible only to members of the technical group. |
| Click and Collect | Storefront settings | A switch that installs the collection-in-store package. Beside it, a button labelled "Configure Pickup Locations" opens the in-store Delivery Methods: the single one in form view when exactly one exists, and the filtered list otherwise. |
| Generate Shipping Labels | Inventory configuration → Operation Types | Whether validating a transfer of this operation type creates the shipment. Computed as false for incoming and internal operation types and true for outgoing ones, and writable afterwards. |
| Carrier | Inventory configuration → Operation Types, batching section | Whether automatic batching groups by Delivery Method. Shown only while automatic batching is on. |
| Maximum weight | Inventory configuration → Operation Types, batching section | The weight cap of an automatic batch. Zero means no cap. Shown only while automatic batching is on. |
| Applicable on Shipping Methods | Inventory configuration → Routes | Whether the route may be attached to a Delivery Method. |
| Propagation of carrier | Inventory configuration → Rules | Whether the Delivery Method and the tracking reference are copied onto the transfer this rule creates. |
| Opening Hours | Inventory configuration → Warehouses | The working schedule whose attendance lines become the store's opening hours in the collection point selector. |
| Delivery Method | A Contact's sales page | The method proposed by default for that contact. Company-dependent. |
| Harmonized System Code, Origin of Goods | A product's inventory page | The customs classification and the country of origin sent to a carrier and printed on the delivery slip. |
| Carrier, Carrier Code | A Package Type's form | Which carrier the container type is reserved for, and that carrier's own code for it. |

### 1.2 System parameters

The domain reads two parameters and owns neither. Both belong to
[`../units-of-measure-and-packaging/`](../units-of-measure-and-packaging/) and are listed here
because every weight and every volume of this domain depends on them.

| Parameter key | Value | Effect on this domain |
|---|---|---|
| `product.weight_in_lbs` | `1` selects the pound; any other value or the absence of the parameter selects the kilogram | Chooses the unit in which every weight of this domain is expressed and displayed: the method's maximum weight, the order's shipping weight, the move's weight, the transfer's weight, the package's weight, the parcel's weight and the batch's weight cap. It also chooses the container-content marker of the parcel barcode: the kilogram marker when the parameter selects the kilogram, and the pound marker otherwise. |
| `product.volume_in_cubic_feet` | `1` selects the cubic foot; any other value or the absence of the parameter selects the cubic metre | Chooses the unit in which the method's maximum volume and the rule-based engine's volume variable are expressed. The same parameter also selects the length unit — the foot instead of the millimetre — which is what a container type's dimensions are typed in. |

Changing either parameter does **not** convert any stored value. A maximum weight of 10 typed while
the parameter said kilograms means 10 pounds the moment the parameter says pounds.

### 1.3 Behaviour that has no setting

Three behaviours a reader might look for a setting for have none, and a rebuild should not invent
one:

- there is no setting that turns the free-above-a-threshold waiver on globally; it is a per-method
  flag;
- there is no setting that chooses between the estimated and the real invoicing policy globally; it
  is a per-method selection, and it only exists while the Delivery – Inventory bridge is installed;
- there is no setting that limits how many tracking references a transfer may accumulate.

---

## 2. Records shipped with the packages

Every record below is shipped as a non-updating record: a later installation of the same package
does not overwrite a value the user has changed.

### 2.1 The core Delivery Costs package

| Record | Kind | Values |
|---|---|---|
| Deliveries | Product Category | Name "Deliveries". Protected against deletion by rule DSH-060. |
| Standard delivery | Product Variant | Name "Standard delivery"; internal code `Delivery_007`; kind service; category "Deliveries"; not sellable; not purchasable; sales price 0.0; invoiced on ordered quantities. |
| Standard delivery | Delivery Method | Name "Standard delivery"; fixed charge 0.0; sequence 1; provider kind `fixed`; delivery product "Standard delivery". |
| Cash on Delivery | Payment Method | Name "Cash on Delivery"; code `cash_on_delivery`; sequence 1000; carries an image; no tokenisation; no express checkout; refunds not supported. |
| Cash on Delivery | Payment Provider | Name "Cash on Delivery"; code `custom`; custom mode `cash_on_delivery`; the main company; the single payment method above; the custom redirect form; state enabled; published. Pending message: "The delivery staff will collect payment upon delivery." |

Installing the package additionally activates the custom provider in the cash-on-delivery mode, and
removing the package resets it.

### 2.2 The Delivery – Inventory bridge

The bridge ships no data record. It contributes the invoicing policy value `real`, the route
relation, the report additions and the security lines of section 5. Installing it also installs the
sales application when that is not already present.

### 2.3 The parcel-point network companion

| Record | Kind | Values |
|---|---|---|
| Parcel-point delivery product | Product Variant | Name "Mondial Relay"; internal code `MR`; kind service; category "Deliveries"; not sellable; not purchasable; sales price 5; invoiced on ordered quantities. The code is what makes a Delivery Method a parcel-point method. |
| Parcel-point method, first country group | Delivery Method | Sequence 30; provider kind `base_on_rule`; integration level `rate`; two countries; the delivery product above. |
| Its two rules | Delivery Price Rule | Condition `quantity` `<=` 10 charging a base of 4; condition `quantity` `>=` 10 charging a base of 5. |
| Parcel-point method, second country group | Delivery Method | Sequence 31; provider kind `base_on_rule`; integration level `rate`; two countries; the same delivery product. |
| Its three rules | Delivery Price Rule | Condition `quantity` `<=` 1 charging a base of 6; condition `quantity` `<=` 10 charging a base of 10; condition `quantity` `>=` 10 charging a base of 20. |
| Parcel-point method, third country group | Delivery Method | Sequence 32; provider kind `base_on_rule`; integration level `rate`; one country; the same delivery product. |
| Its three rules | Delivery Price Rule | Condition `quantity` `<=` 3 charging a base of 10; condition `quantity` `<=` 7 charging a base of 15; condition `quantity` `>=` 10 charging a base of 28. |

Every rule uses the default condition variable `quantity`, the default factor variable `weight` and
a factor amount of zero, so each is a flat charge for a quantity band. The third group's rules leave
a gap: a basket of more than 7 and fewer than 10 units matches no rule, and the rating fails with
"Not available for current order". This is deliberate in shipped data that the seller is expected to
replace; the companion's own description says the pricing is an example that must be adapted.

### 2.4 The collection-in-store package

| Record | Kind | Values |
|---|---|---|
| Pick up in store | Product Variant | Name "Pick up in store"; kind service; sales price 0; not purchasable; not sellable. |
| Pick up in store | Delivery Method | Name "Pick up in store"; provider kind `in_store`; the delivery product above; the default website; production environment set. Created only when it does not already exist. |
| Pay on site | Payment Method | Name "Pay on site"; code `pay_on_site`; sequence 1000; carries an image; no tokenisation; no express checkout; manual capture not supported; refunds not supported. |
| Pay on Site | Payment Provider | Name "Pay on Site"; code `custom`; custom mode `on_site`; state enabled; published; the single payment method above; the custom redirect form. Pending message: "Your order has been confirmed." followed by "Please come to the store to pay for your products." |

Installing the package additionally activates the custom provider in the on-site mode, and removing
the package resets it.

Because the shipped in-store Delivery Method is created by the ordinary creation routine, the
routine's in-store defaults apply: the integration level becomes `rate`, cash on delivery is
cleared, the three destination filters are cleared, every warehouse of the resolved company becomes
a store and the method is published when at least one warehouse was found.

### 2.5 The Delivery – Batch Transfers bridge

The bridge ships no data record. It contributes the two operation-type fields, the batching guards
and the view addition.

---

## 3. Demonstration records

Demonstration records are installed only when a database is created with demonstration data. They
are listed because acceptance scenarios refer to them and because a rebuild's demonstration data
should be equivalent.

| Record | Kind | Values |
|---|---|---|
| The Poste | Product Variant | Name "The Poste"; internal code `Delivery_009`; kind service; category "Deliveries"; not sellable; not purchasable; sales price 20.0; invoiced on ordered quantities. |
| The Poste | Delivery Method | Name "The Poste"; fixed charge 20.0; sequence 2; provider kind `base_on_rule`; the product above. |
| Its three rules | Delivery Price Rule | Condition `quantity` `<=` 5 charging a base of 20; condition `quantity` `>=` 5 charging a base of 50; condition `price` `>=` 300 charging a base of 0. Because the rules are walked in order and the first two both test the quantity, the third is only reached by a basket of fewer than five units and at least 300 in value — which the first rule already matched. The third rule is therefore unreachable, which is a property of the demonstration data and not of the engine. |
| Local Delivery | Delivery Method | Name "Local Delivery"; fixed charge 5.0; waiver flag set; threshold 50; sequence 4; provider kind `fixed`; a local-delivery product moved into the "Deliveries" category. |
| Default delivery method | Default value | "Local Delivery" is set as the default Delivery Method of every Contact. |
| Cash on Delivery | Payment Provider | Set to the disabled state, so that a demonstration database does not offer it. |
| Pay on Site | Payment Provider | Set to the disabled state, for the same reason. |
| Three outgoing transfers | Transfer | One confirmed, one ready and one done, each carrying "The Poste" as its Delivery Method, one unit of a demonstration product, a scheduled date three or eight days in the past, and a demonstration contact. |

---

## 4. Security groups

The domain defines no group of its own. It uses six groups defined elsewhere:

| Group | Owning domain | Role in this domain |
|---|---|---|
| Sales / User | [`../sales/`](../sales/) | Reads Delivery Methods, Delivery Price Rules and Delivery Postal Code Prefixes; uses the selection wizard |
| Sales / Administrator | [`../sales/`](../sales/) | Full control of Delivery Methods and Delivery Price Rules |
| Inventory / User | [`../inventory-operations/`](../inventory-operations/) | Reads Delivery Methods, Delivery Price Rules and Delivery Postal Code Prefixes; uses the selection wizard |
| Inventory / Administrator | [`../inventory-operations/`](../inventory-operations/) | Full control of Delivery Methods, Delivery Price Rules and Delivery Postal Code Prefixes |
| Contact creation | [`../contacts-and-organizations/`](../contacts-and-organizations/) | Reads Delivery Methods; full control of Delivery Postal Code Prefixes |
| Settings | [`../identity-and-access/`](../identity-and-access/) | Reads Delivery Methods; is the only group that may read and write the parcel-point container code |

Two further groups gate parts of the interface without gating data:

| Group | Effect |
|---|---|
| Technical | Shows the Postal Code Prefix menu |
| Advanced locations | Shows the Routes field on a Delivery Method |
| Multi-company | Shows the Company field on a Delivery Method |
| Units of measure | Shows the weight control on the selection wizard when the Delivery – Inventory bridge is absent; the bridge removes the restriction so that the weight is always shown |

---

## 5. Access rights and record rules

### 5.1 The access-rights matrix

Each row is one access-rights line. A user obtains the union of the rights of the groups they
belong to.

| Entity | Group | Read | Write | Create | Delete |
|---|---|---|---|---|---|
| Delivery Method | Sales / User | yes | no | no | no |
| Delivery Method | Settings | yes | no | no | no |
| Delivery Method | Sales / Administrator | yes | yes | yes | yes |
| Delivery Method | Contact creation | yes | no | no | no |
| Delivery Method | Inventory / User | yes | no | no | no |
| Delivery Method | Inventory / Administrator | yes | yes | yes | yes |
| Delivery Price Rule | Sales / User | yes | no | no | no |
| Delivery Price Rule | Sales / Administrator | yes | yes | yes | yes |
| Delivery Price Rule | Inventory / User | yes | no | no | no |
| Delivery Price Rule | Inventory / Administrator | yes | yes | yes | yes |
| Delivery Postal Code Prefix | Sales / User | yes | no | no | no |
| Delivery Postal Code Prefix | Contact creation | yes | yes | yes | yes |
| Delivery Postal Code Prefix | Inventory / User | yes | no | no | no |
| Delivery Postal Code Prefix | Inventory / Administrator | yes | yes | yes | yes |
| Delivery Method Selection Wizard | Sales / User | yes | yes | yes | no |
| Delivery Method Selection Wizard | Inventory / User | yes | yes | yes | no |

Two properties of the matrix are worth stating because they surprise readers:

1. The sales administrator group may not create a Delivery Postal Code Prefix. Only the
   contact-creation group and the inventory administrator group may. A sales administrator who
   needs a new prefix must therefore hold one of those two groups as well, or create the prefix
   inline from the availability page while holding one of them.
2. Nobody may delete a Delivery Method Selection Wizard record. The transient-record cleaner removes
   it, which is the ordinary treatment of a dialogue record.

### 5.2 Record rules

| Rule | Entity | Applies to | Condition | Operations |
|---|---|---|---|---|
| Delivery Carrier multi-company | Delivery Method | Every user, including administrators | The method's company is one of the companies currently active for the user, or the method has no company | Read, write, create, delete |

There is no record rule on a Delivery Price Rule, on a Delivery Postal Code Prefix or on the
selection wizard. A price rule is protected only by the visibility of its method; a postal-code
prefix is global by design, because the same prefix is meant to be shared.

### 5.3 Elevated-rights operations

Five operations of this domain deliberately run with elevated rights. They are listed because a
rebuild that omits any of them will produce access failures in ordinary use.

| Operation | Why |
|---|---|
| Collecting the rule-based engine's variables from an order | A salesperson who may not read product costs must still be able to obtain a rate |
| Computing the weight of a Stock Move and of a Transfer | A user who may not read a move must still see the transfer's weight |
| Creating the shipping charge line | A user who may price carriage may not be allowed to create order lines |
| Sending the shipment when a transfer is validated | A warehouse operator may not be allowed to write on a Sales Order, yet the real carriage charge must reach it |
| Computing the weight of a package while packing | A package may be shared between companies, and reading its contents must not fail |

---

## 6. Scheduled jobs

**The domain defines no scheduled job.** Nothing in it is polled, retried on a timer or swept.
Three consequences follow and a rebuild should not add jobs for them:

- a shipment that could not be created is not retried; the warning activity is the only follow-up,
  and a human presses "Send to Shipper" again;
- a tracking reference is never refreshed from the carrier; what the carrier returned once is what
  the system keeps;
- a collection point's coordinates are never re-geolocated on a timer; they are computed on first
  use and, when the geolocation fails, replaced by the impossible pair that stops further attempts.

The transient records of the selection wizard are removed by the platform's own transient-record
cleaner, described in [`../../runtime/scheduled-jobs.md`](../../runtime/scheduled-jobs.md).

---

## 7. Message templates and activity types

### 7.1 Message templates

**The domain defines no message template.** Every message it produces is composed in place and
posted in the transfer's discussion thread. The four messages are:

| Message | When | Text |
|---|---|---|
| Shipment sent | A shipment is created | "Shipment sent to carrier <method name> for shipping with tracking number <tracking reference>", a line break, then "Cost: <shipping cost with two decimals> <currency name>" |
| Shipment cancelled | A shipment is cancelled | "Shipment <tracking reference> cancelled" |
| Tracking links | The tracking button is pressed and the stored link is a list | "Tracking links for shipment:", a line break, then one link per pair, each rendered as the pair's label pointing at the pair's address, each followed by a line break |
| Carrier refusal | A shipment is refused after another was accepted | The carrier's own refusal text, posted as a notification |

The domain does contribute to one template it does not own: the order confirmation message of
[`../sales/`](../sales/) repeats the Delivery Method's customer-facing description.

### 7.2 Activity types

**The domain defines no activity type.** It schedules an activity of the platform's *warning* type,
owned by [`../messaging-and-activities/`](../messaging-and-activities/), when a shipment is refused
after another shipment of the same batch was accepted. The activity carries:

| Member | Value |
|---|---|
| Type | The platform's warning activity type |
| Due date | Today |
| Assigned to | The transfer's responsible user, or the validating user when the transfer has none |
| Attached to | The Transfer |
| Note | "Exception occurred with respect to carrier on the transfer <a link carrying the transfer's name>. Manual actions might be needed." followed by "Exception: <the carrier's refusal text>" |

---

## 8. Sequences

**The domain defines no sequence.** No record it owns is numbered: a Delivery Method is identified
by its name, a Delivery Price Rule by its position in the list, a Delivery Postal Code Prefix by
its own value. The tracking reference is not a sequence either — it is whatever the carrier
returned.

Three reproduced string constants act as naming conventions and must be reproduced exactly, because
the return-label computation searches attachments by prefix:

| Constant | Meaning |
|---|---|
| `LabelShipping-` followed by the provider kind | The name prefix of a shipping label attached to a Transfer |
| `LabelReturn-` followed by the provider kind | The name prefix of a return label attached to a Transfer; the return-label list searches for exactly this prefix |
| `ShippingDoc-` followed by the provider kind | The name prefix of an accompanying document, such as a commercial invoice |

Two further reproduced constants are part of the contract:

| Constant | Meaning |
|---|---|
| `Bulk Content` | The name given to the parcel that carries the goods of a transfer that are not in any package |
| `<shipmenttrackingnumber>` | The placeholder replaced by the tracking reference inside a Delivery Method's tracking-link pattern |

---

## 9. Neutralising a copy of a database

Two statements make a copy of a production database safe to work on, by preventing it from talking
to a carrier's production service.

| Statement | Effect |
|---|---|
| First statement of the core package | Every Delivery Method leaves the production environment. |
| Second statement of the core package | Every Delivery Method whose provider kind is neither `fixed` nor `base_on_rule` is archived. This includes the in-store kind and every carrier integration. |
| Statement of the parcel-point companion | Every Delivery Method's brand code is reset to the reproduced test value `BDTEST  `. |

The consequence of the second statement is that a neutralised copy offers only the two core kinds.
A rebuild must provide the equivalent, and every carrier integration must add its own statement, as
listed in [workflows.md](workflows.md) section 20 step 12.

---

## 10. Configuration checklist for a working installation

A rebuild is configured for this domain when all of the following hold. The list is an acceptance
aid, not a procedure.

1. The "Deliveries" product category exists and is protected against deletion.
2. At least one service product exists for carriage, not sellable, not purchasable, invoiced on
   ordered quantities.
3. At least one Delivery Method exists, and its company matches the company of its delivery
   product.
4. The weight system parameter and the volume system parameter resolve to a unit, whether or not
   the parameters are present.
5. Outgoing operation types ask for shipping labels; incoming and internal ones do not.
6. The rules of every multi-step delivery route carry the carrier-propagation flag where the seller
   wants the method carried forward, and the make-to-order rules carry it by construction.
7. The six groups of section 4 exist, and the sixteen access-rights lines of section 5.1 are in
   place.
8. The multi-company record rule of section 5.2 is in place and applies to administrators as well.
9. Both custom payment providers exist, each bound to its own payment method, each carrying its
   pending message.
10. For collection in store: at least one warehouse has an address that can be geolocated, and the
    in-store Delivery Method is published and carries at least one store.
11. For the storefront: every Delivery Method meant to be offered online is published, and its
    online description is filled.
12. The neutralisation statements of section 9 exist and run when a copy is made.
