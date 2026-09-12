# Workflows

Replenishment turns a need into a supplying document. The workflows below carry that from end to end: the algorithm that picks the applicable stock rule, the run algorithm that turns needs into documents, the three actions (pull, buy, manufacture), push propagation, chained move creation, the reordering rule life cycle and the scheduler, manual replenishment, the replenishment report, drop shipping, inter-warehouse resupply, subcontractor resupply, warehouse route generation, return to the vendor, cancellation propagation and date propagation. Every step states the records it reads, the decisions it makes, the records it writes with the field values written, and the messages it emits.

## Table of contents

1. Rule selection for a need
2. Rule selection for an arrival (push)
3. The run algorithm
4. The pull action
5. The buy action
6. The manufacture action
7. Push application
8. Confirming a stock move and creating the supply need
9. Make to order versus multi-step routes
10. Adjusting the supply method of an existing move
11. Reordering rule evaluation
12. The scheduler
13. Manual replenishment: Order and Order to Max
14. Snoozing a reordering rule
15. The replenishment report
16. The Replenishment Information wizard
17. The Product Replenish wizard
18. Drop shipping
19. Inter-warehouse resupply
20. Subcontractor resupply and drop shipping to a subcontractor
21. Warehouse creation and reconfiguration
22. Cancellation propagation
23. Date and deadline propagation
24. The forecast report
25. Return to the vendor
26. State tables

---

## 1. Rule selection for a need

**Purpose.** Given a product, a location where the product is needed, and a values map, return exactly one Stock Rule, or nothing.

**Actor.** The system, on behalf of whatever created the need.

**Preconditions.** The location exists. The values map may carry `routes`, `packaging_unit_of_measure`, `warehouse` and `company`.

**Steps.**

1. If the location is empty, return nothing.
2. **Build the location chain.** Start with the location. Repeatedly append the parent of the last element until an element has no parent. The result is an ordered list from the requested location up to the root location. Call it the *location chain*.
3. **Build the base condition.** The base condition is:
   - `destination_location IN <location chain>` AND `action ≠ "push"`.
   - When the location chain contains the shared inter-company location, the shared customer location is added to the list of destinations as well. This avoids having to duplicate every rule that delivers to customers.
   - When the search is running with elevated rights and the values map carries a company, the condition is further narrowed to `company IS EMPTY OR company IS A DESCENDANT OF (the request company, plus the companies of the routes in `routes`)`. When the search runs with a normal user's rights, the record rules of the domain already restrict the result to the user's companies and no extra narrowing is applied.
4. **Restrict by warehouse.** Let *warehouse* be `values.warehouse` when present, otherwise the warehouse of the location chain. When a warehouse is known, add `warehouse IS EMPTY OR warehouse = <warehouse>`.
5. **Collect the candidate route set.** The union of:
   - the routes in `values.routes`, if any;
   - the routes of the package type of `values.packaging_unit_of_measure`, if any;
   - the routes of the product plus the total routes of the product's category (which includes the routes inherited from ancestor categories);
   - the routes of the warehouse, filtered by the *warehouse route filter* below.

   When that union is not empty, add `route IN <candidate route set>` to the condition.

   *Warehouse route filter.* A warehouse route is kept only when it is usable for this product:
   - a route containing a rule with action `buy` is kept only when the product has at least one Vendor Price;
   - a route containing a rule with action `manufacture` is kept only when the product has at least one bill of materials of type `normal`;
   - every other route is kept.
6. **Group the matching rules.** Read every rule matching the condition, grouped by (`destination_location`, `warehouse`, `route`). Inside each group keep the single rule with the lowest (`route_sequence`, `sequence`) pair. Build a map: (destination location, route) → warehouse → rule. The map preserves the read order, which is by minimum `route_sequence` ascending, then minimum `sequence` ascending.
7. **Walk the location chain again**, this time from the requested location upwards, stopping at the first location for which a rule is found. For each candidate location:
   1. If the candidate location is the shared inter-company location and it has not yet been handled, also consider the shared customer location at this step (in the same iteration, after the candidate location itself).
   2. Apply the *route precedence* below to the candidate location.
   3. If a rule was found, stop and return it.
   4. Otherwise move to the parent location and repeat.
8. If the walk reaches the root without finding a rule, return nothing.

**Route precedence (applied at one candidate location).** Try each of the following sources in order, stopping at the first that yields a rule:

1. `values.routes` (the routes explicitly attached to the need: a sales order line route, a reordering rule route, a move route, a replenish-wizard route).
2. The routes of the package type of `values.packaging_unit_of_measure`.
3. The product's own routes, unioned with the total routes of the product's category.
4. The warehouse's routes.

**Rule extraction inside one source.** Given a set of routes and a candidate location:

1. Sort the routes with the following key, ascending: first a boolean that is false when the route is one of the product's own routes and true otherwise (product routes therefore come first), then the route `sequence`.
2. For each route in that order, look up (candidate location, route) in the map.
   - If there is no entry, continue with the next route.
   - If no warehouse is known, take the first entry of the warehouse sub-map in its stored order.
   - If a warehouse is known, take the entry for that warehouse; when there is none, take the entry whose warehouse is empty.
3. Return the first rule found; if none, return nothing.

**Postcondition.** At most one rule is returned. The rule's `destination_location` is the requested location or one of its ancestors.

**Worked example (route precedence by sequence).** A product carries two routes: "Route 1" with sequence 10 holding a rule with sequence 20, and "Route 2" with sequence 5 holding a rule with sequence 20. Both rules pull from the warehouse stock location to the warehouse stock location with the delivery operation type. Rule selection for that product at the warehouse stock location, with both routes passed as `values.routes`, returns the rule of "Route 2", because 5 is lower than 10.

---

## 2. Rule selection for an arrival (push)

**Purpose.** Given a product and a destination location where goods have just arrived (or are about to), return the push rule that says where the goods go next.

**Steps.**

1. Set *location* to the destination location.
2. While no rule has been found and *location* is not empty:
   1. Build the condition `location_source = <location>` AND `action IN ("push", "pull_push")`, combined with any extra condition supplied by the caller (used to exclude rules already rejected, see step 4 of "Push application").
   2. Restrict by warehouse exactly as in rule selection for a need: `warehouse IS EMPTY OR warehouse = <warehouse>` when a warehouse is known.
   3. Search the rules in four passes, in this order, stopping at the first pass that returns a rule. Each pass reads at most one rule, ordered by `route_sequence` ascending then `sequence` ascending:
      1. rules whose route is in `values.routes`;
      2. rules whose route is in the routes of the package type of `values.packaging_unit_of_measure`;
      3. rules whose route is in the product's routes unioned with the total routes of the product's category;
      4. rules whose route is in the warehouse's routes.
   4. If no rule was found, set *location* to its parent and repeat.
3. Return the rule found, or nothing.

**Note on the difference with need selection.** Push selection does not group by warehouse and does not apply the warehouse route filter; it simply reads the single best-ordered rule per source.

---

## 3. The run algorithm

**Purpose.** Turn a list of procurement requests into documents.

**Actors.** Any producer of needs: a confirmed sales order line, a confirmed make-to-order stock move, a reordering rule (manually or through the scheduler), the Product Replenish wizard, a manufacturing order, a subcontracting flow.

**Inputs.** A list of procurement requests (structure in `entities.md`, section 10), and a flag `raise_user_error` (default true).

**Steps.**

1. **Explode kits.** Before anything else, when the Manufacturing capability package is installed: for every request, look for a bill of materials of type "kit" for the request's product in the request's company. When one is found, the request is replaced by one request per component of the exploded kit:
   - the requested quantity is first converted into the kit bill's unit, then divided by the bill's quantity to obtain the number of kits to produce;
   - the kit is exploded for that number of kits, excluding the attribute values listed in `never_product_template_attribute_values`;
   - for each resulting component line, the component quantity is adjusted to the component's reference unit (a conversion that may change the unit as well as the number), and a new request is built with the component product, the adjusted quantity and unit, the same location, name, origin, company and values, plus `bill_of_materials_line` set to the exploded line.
   Requests whose product is not a kit pass through unchanged.
2. **Widen the route set for buy routes.** When the Purchase Inventory capability package is installed: for every request whose `values.routes` contains a route holding a rule with action `buy`, the reception route of every warehouse of the request's company is added to `values.routes`. This makes the receipt steps of the warehouse available to the chain that the buy rule will start.
3. **Apply the three defaults.** For every request, set `values.company` to the company of the request location when absent, `values.priority` to `"0"` when absent, and `values.date_planned` to the current moment when absent or empty.
4. **Skip irrelevant requests.** A request is skipped, silently, when the product is not a goods product, or when the requested quantity is zero at the rounding of the request unit.
5. **Select a rule** for each remaining request, using the algorithm of section 1.
   - If no rule is found, record the error `No rule has been found to replenish "<product display name>" in "<location display name>".` followed by a new line and `Verify the routes configuration on the product.`, paired with the request.
   - Otherwise file the request under the rule's action, mapping `pull_push` to `pull`.
6. **Fail early on unresolvable requests.** If any request produced an error in step 5, raise now (see "Error handling" below). No document is created for any request in this call.
7. **Dispatch.** For each action present, in the order the actions were first encountered, call the matching action handler with the complete list of (request, rule) pairs of that action: the pull action for `pull`, the buy action for `buy`, the manufacture action for `manufacture`. If an action handler raises a procurement exception, its faulty (request, message) pairs are collected and the remaining actions still run.
8. **Fail late.** If any action handler produced errors, raise now.
9. Return success.

**Error handling.** When `raise_user_error` is true, the collected messages are joined with new lines and raised as a single user-facing error, which aborts the surrounding transaction. When it is false, a procurement exception carrying the list of (request, message) pairs is raised instead, so that the caller (the scheduler) can isolate the failing requests and continue.

---

## 4. The pull action

**Purpose.** Create stock moves for a list of (request, pull rule) pairs.

**Steps.**

1. **Validate the rules.** For every pair, if the rule has no `location_source`, raise a procurement exception carrying the single pair and the message `No source location defined on stock rule: <rule name>!`. Nothing is created.
2. **Order the pairs.** Sort the pairs so that requests with a negative or zero quantity are handled before requests with a positive quantity. (The sort key is the boolean "quantity is greater than zero", ascending.) This matters because a negative request expresses a return and must be able to merge with its sibling.
3. **Compute the supply method to write.** For each pair, the move's supply method is the rule's `procure_method`, except that `mts_else_mto` (make to stock, else make to order) is written as `make_to_stock`. The split between the part taken from stock and the part that triggers another rule has already happened before the request was built (see section 8, step 3).
4. **Build the move values** for each pair, as follows.

| Move field | Value |
|---|---|
| `company` | the first non-empty of: the rule company, the company of the rule's source location, the company of the rule's destination location, the request company |
| `product` | the request product |
| `unit_of_measure` | the request unit |
| `demand_quantity` | the request quantity |
| `partner` | the rule's `partner_address` when set, otherwise `values.partner` |
| `source_location` | the rule's `location_source` |
| `location_final` | the request location |
| `destination_location` | the rule's `destination_location`, but only when the rule's `location_destination_from_rule` is true; otherwise the field is left out and the move derives it from the operation type's default destination location |
| `move_destinations` | the moves in `values.move_destinations` |
| `rule` | the rule |
| `references` | the references in `values.references` |
| `procure_method` | the value computed in step 3 |
| `origin` | the request origin |
| `operation_type` | the rule's `operation_type` |
| `procurement_values` | the serialized `values` map |
| `routes` | the routes in `values.routes`, replacing any previous content |
| `never_product_template_attribute_values` | `values.never_product_template_attribute_values` |
| `warehouse` | the rule's `warehouse` |
| `date` | `values.date_planned` minus the rule's `lead_time_days` calendar days |
| `deadline` | `values.date_deadline` minus the rule's `lead_time_days` calendar days, or empty when there is no deadline |
| `propagate_cancel` | the rule's `propagate_cancel` |
| `priority` | `values.priority` |
| `orderpoint` | `values.orderpoint` |
| extra fields | every key listed by the capability packages' extension point that is present in `values`: `bill_of_materials_line` (Manufacturing), `production_group` (Manufacturing), `sales_order_line` (Sales Inventory) |

5. **Inter-warehouse partner stamping.** When `values.move_destinations` is not empty and the request location is the company's internal transit location:
   - when no partner has been determined yet and every downstream move's destination warehouse resolves to exactly one partner, that partner becomes the new move's `partner`;
   - the downstream moves' `partner` is set to the partner of the warehouse of the rule's source location, or, when that warehouse has none, to the partner of the rule's company.
6. **Mark negative requests as refunds.** When the requested quantity is negative at the rounding of the request unit, `values.to_refund` is set to true before the values are serialized.
7. **Create and confirm.** Group the move values by company. For each company, create the moves with elevated rights and in that company's context (the user who triggered the need may have no right to create stock moves, for example a salesperson or a portal user), then confirm them. Confirmation is what chains the next step of the route (see section 8).

**Postcondition.** One stock move per request, confirmed, linked to the requesting moves through `move_destinations`.

---

## 5. The buy action

**Purpose.** Create or extend draft purchase orders for a list of (request, buy rule) pairs.

**Actor.** The system, running with elevated rights for the purchase records.

### 5.1 Vendor selection and grouping

For each (request, rule) pair, in the order received:

1. **Determine the company**: the rule company when set, otherwise the request company.
2. **Select the vendor price** (the detailed selection algorithm is in `calculations.md`, section "Vendor selection"):
   1. When `values.forced_vendor_price` is set, use it.
   2. Otherwise, when `values.orderpoint` is set and that reordering rule has a `vendor_price`, use that.
   3. Otherwise run the shared vendor-selection operation of `../pricing-and-pricelists/` with: the contact restriction (`values.vendor_contact`, or `values.partner` when `values.force_unit_of_measure` is true, otherwise no restriction; for the drop shipping route the restriction is always removed), the requested quantity, the date (the later of the request's planned date and today, or no date when there is no planned date), the request unit, and the parameter `force_unit_of_measure` taken from `values.force_unit_of_measure`.
   4. When that still yields nothing, fall back to the first vendor price of the product that belongs to this company or to no company, regardless of price, minimum quantity or validity dates.
3. **Handle the absence of a vendor.**
   - When no vendor price at all could be found **and** the request came from a reordering rule (the run was started with the reordering-rule marker), record the error:
     `There is no matching vendor price to generate the purchase order for product <product display name> (no vendor defined, minimum quantity not reached, dates not valid, ...). Go on the product form and complete the list of vendors.`
   - When no vendor price could be found and the request did **not** come from a reordering rule: do not fail. Instead:
     1. take the downstream moves in `values.move_destinations`;
     2. cancel those whose `propagate_cancel` is true;
     3. set the supply method of the downstream moves to `make_to_stock`, so that they wait for stock instead of waiting for a purchase;
     4. notify the responsible person (see 5.5);
     5. continue with the next pair; no purchase order is created for this request.
4. **Record the chosen vendor** in `values.chosen_vendor_price`, and copy the rule's `propagate_cancel` into `values.propagate_cancel`.
5. **Compute the purchase order grouping key** (the "purchase order condition") and file the pair under it. The condition is an exact-match condition on the following fields; two requests share a purchase order only when every component matches:

| Component | Value | Always present |
|---|---|---|
| `partner` | the contact of the chosen vendor price | yes |
| `state` | `draft` | yes |
| `operation_type` | the rule's `operation_type` | yes |
| `company` | the company from step 1 | yes |
| `user` | the buyer of the vendor contact | yes |
| `currency` | the currency of the chosen vendor price, or the vendor's purchase currency, or the company currency | yes |
| `references` | when the vendor's grouping mode is `default`, or the operation type has code `dropship`: `references` contains one of `values.references` when that is not empty; otherwise, for grouping mode `default` only, `references IS EMPTY` | conditional |
| `date_planned` window | see below | conditional |

   Date window by grouping mode:
   - `default` ("On Order"): no date window. Needs are grouped by reference, therefore a need with no reference never joins an order that has one.
   - `all` ("Always"): no date window and no reference component at all; every need for this vendor joins the same draft order.
   - `day` ("Daily"): `date_planned` must fall on the same calendar day as the request's planned date (from the start of that day to the end of that day).
   - `week` ("Weekly"): when the vendor's `grouping_weekday` is `default`, `date_planned` must fall in the window that runs from the start of the day *n* days before the request's planned date to the end of the day 6 minus *n* days after it, where *n* is the weekday number of the planned date (Monday = 1, Tuesday = 2, Wednesday = 3, Thursday = 4, Friday = 5, Saturday = 6, Sunday = 7). When `grouping_weekday` names a weekday, the window is the single target day: the planned date shifted forward by `(7 + target weekday − planned weekday) modulo 7` days, from the start to the end of that day.

### 5.2 Finding or creating the purchase order

For each grouping key, with its list of (request, rule) pairs:

1. Collect the set of non-empty request origins.
2. Search, with elevated rights, for one purchase order matching the grouping key.
3. **If none exists:**
   1. Keep only the requests whose quantity is not negative.
   2. If that list is empty, skip this key entirely (a negative-only group creates no order).
   3. Compute the order date: the minimum, over those requests, of `values.date_order` when set, otherwise `values.date_planned` minus the chosen vendor's lead time in calendar days.
   4. Take the first of those requests as the source of the common values and create the purchase order with elevated rights, in the company context, with:

| Purchase order field | Value |
|---|---|
| `partner` | the contact of the chosen vendor price |
| `user` | the buyer of that contact |
| `operation_type` | the rule's `operation_type` |
| `company` | the company determined in 5.1 |
| `currency` | the currency of the vendor price, else the vendor's purchase currency, else the company currency |
| `destination_address` | `values.partner` when set, otherwise empty |
| `origin` | the collected origins, joined with ", " |
| `payment_term` | the vendor's supplier payment term |
| `date_order` | the order date computed above |
| `fiscal_position` | the fiscal position resolved for the vendor in that company (owned by `../taxes/`) |
| `references` | `values.references` |

   The order is created under the superuser account so that the user who triggered the need does not become a follower of the purchase order; that user may have no access to purchasing at all.
4. **If one exists:**
   1. Add every reference from every request's `values.references` to the order's `references`.
   2. Adjust the order origin: when the order already has an origin, append the origins that are not already present, comma separated; when it has none, set it to the collected origins joined with ", ".

### 5.3 Merging the requests into lines

1. **Group the requests that could share a line.** Two requests are grouped together when every component of the following key is equal:
   - the product;
   - the request unit;
   - `values.propagate_cancel`;
   - `values.product_description_variants`;
   - the value `values.orderpoint` when `values.move_destinations` is empty, and the empty value otherwise (needs that feed a downstream move are grouped by their downstream moves, not by their reordering rule);
   - `values.sales_order_line`, when the Drop Shipping capability package is installed (needs coming from different sales order lines never share a line, because the delivered quantity of each sales order line is read back from its own purchase order line).
2. **Merge each group into one request:** the quantities are summed; the `move_destinations` of every member are unioned; the `orderpoint` is the first non-empty one found; every other value is taken from an arbitrary member (they are equal by construction, except for the three just listed).
3. **Index the existing order lines** of the purchase order by product, ignoring lines that are section or note lines.
4. **For each merged request**, find a candidate line among the existing lines for that product:
   1. Keep lines whose `propagate_cancel` equals `values.propagate_cancel`.
   2. When the request has an `orderpoint`, has no `move_destinations`, and that reordering rule is **not** a temporary rule (a temporary rule is one created by the superuser with trigger `manual`), keep only lines whose `orderpoint` is that reordering rule or is empty.
   3. When `values.force_unit_of_measure` is true, keep only lines whose unit equals the request unit.
   4. When `values.product_description_variants` is set, keep only lines whose name matches the product display name in the vendor's language followed by a new line and the description, or lines whose name is exactly the product display name while the description equals the product name.
   5. Among the remaining lines, take the first when sorted by `orderpoint` (lines without a reordering rule sort first).
5. **If a candidate line was found, update it** (see 5.4).
6. **If no candidate was found:**
   1. If the merged quantity is zero or negative, skip: a new line is never created with a non-positive quantity.
   2. Otherwise build the create values for a new line (see 5.4) and add them to a batch.
   3. Advance the order date if needed: let *planned* be the order's `date_planned` when set, otherwise the minimum planned date among the batch; compute *candidate order date* as *planned* minus the chosen vendor's lead time in calendar days; when the date part of *candidate order date* is earlier than the date part of the order's `date_order`, write *candidate order date* onto the order's `date_order`.
7. Create the batched lines with elevated rights.

### 5.4 Line values

**Updating an existing line.**

1. Convert the request quantity into the line's unit with half-up rounding; call it *added quantity*.
2. Re-run vendor selection for the contact of the chosen vendor, the quantity (existing line quantity plus *added quantity*), the date part of the order's `date_order`, the line's unit, and the `force_unit_of_measure` parameter. This can select a different price bracket now that the total is larger.
3. Compute the unit price: when a vendor price was found, take its price, correct it for tax inclusion against the product's vendor taxes and the line's taxes in the order company, and convert it from the vendor price currency into the order currency at today's rate when the currencies differ. When no vendor price was found, keep the line's current unit price.
4. Write on the line: `product_quantity` = existing quantity plus *added quantity*; `price_unit` = the price from step 3; `move_destinations` extended with `values.move_destinations`.
5. When the selected vendor price uses a different unit than the line and `values.force_unit_of_measure` is not true, convert the whole new quantity into the vendor's unit with half-up rounding and write both the converted quantity and the vendor unit onto the line.
6. When `values.orderpoint` is set, write it onto the line's `orderpoint`.

**Creating a new line.**

1. When `values.force_unit_of_measure` is not true and the vendor's unit differs from the request unit, convert the quantity into the vendor's unit and use the vendor unit for the line.
2. Build the base line values with the shared purchase-line preparation operation of `../purchasing/` (product, quantity, unit, company, vendor contact, order), which fills the name, taxes, price and analytic distribution.
3. When `values.product_description_variants` is set and differs from the product name, append a new line and that description to the line name.
4. Set `date_planned` to `values.date_planned`.
5. **Weekly grouping adjustment.** When the vendor's grouping mode is `week` and `grouping_weekday` names a weekday, shift the line's `date_planned` forward by `(7 + target weekday − planned weekday) modulo 7` days. When the order has no `date_planned` yet, or its `date_planned` is not earlier than the shifted line date, shift the order's `date_order` forward by the same number of days, so that the interval between the order deadline and the expected arrival is preserved.
6. Set `move_destinations` to `values.move_destinations`, `location_final` to the request location, `orderpoint` to `values.orderpoint`, `propagate_cancel` to `values.propagate_cancel`, `product_description_variants` to `values.product_description_variants`, and the excluded attribute values to `values.never_product_template_attribute_values`.

### 5.5 Notifying the responsible person when no vendor was found

The base behavior is to do nothing. Capability packages extend it:

- When the need came from a sales order line, the salesperson of that order is notified.
- When the need came from a manufacturing order, the responsible of that order is notified.

The notification is a message posted on the originating document, addressed to the users to notify, whose body is the mention of each user followed by a line break, then `No supplier has been found to replenish` then the product display name in bold, then `this product should be manually replenished.`

### 5.6 Worked example: make to order with a three-day vendor and two security days

Configuration: the product has a single vendor with a lead time of 3 days; the company's days to purchase is 2; the sales safety days is 0; the warehouse receives in one step; the product carries the Buy route and the Replenish on Order route.

A sales order line for 10 units is confirmed on day 0 with a promised delivery date on day 20.

1. The sales order line builds a procurement request with `date_deadline` = day 20 and `date_planned` = day 20 minus the sales safety days = day 20.
2. Rule selection at the customer location returns the delivery rule of the warehouse, supply method `make_to_order`. The pull action creates the delivery move with `date` = day 20 and `deadline` = day 20.
3. Confirming the delivery move creates a need at the warehouse stock location with `date_planned` = day 20 and `date_deadline` = day 20, and, because the source location is inside the warehouse, with `date_order` computed by the dates-information rule: `date_order` = day 20 minus the purchase delay carried by the rule chain = day 20 minus 3 = day 17.
4. Rule selection at the warehouse stock location returns the Buy rule. The buy action selects the vendor, and creates a purchase order with `date_order` = day 17 (taken from `values.date_order`) and one line with `date_planned` = day 20.
5. The company's days to purchase of 2 does not move the order deadline or the expected arrival; it widens the forecast window used by reordering rules (see section 11). The purchase order therefore reads: Order Deadline day 17, Expected Arrival day 20.
6. Confirming the purchase order creates the receipt move with `date` = day 20 and `deadline` = day 20, linked as the origin move of the delivery move.

Had the sales safety days been 2, step 1 would give `date_planned` = day 18 and `date_deadline` = day 20, step 4 would give `date_order` = day 15 and a line `date_planned` of day 18, and step 6 would give a receipt scheduled on day 18 with a deadline of day 20.

---

## 6. The manufacture action

**Purpose.** Create or extend manufacturing orders for a list of (request, manufacture rule) pairs. The manufacturing order itself is owned by `../manufacturing/`; this section specifies only what this domain contributes.

**Steps.** For each pair, in the order received:

1. Skip the request when its quantity is zero or negative at the rounding of the request unit.
2. **Find the bill of materials:** `values.bill_of_materials` when set; otherwise the `bill_of_materials` of `values.orderpoint` when set; otherwise the best matching bill of type `normal` for the product, the rule's operation type and the company; otherwise the best matching bill of type `normal` for the product and the company, ignoring the operation type.
3. **Look for an existing manufacturing order to extend**, unless the request origin is the master production schedule marker. The search condition is an exact match on: the bill of materials, the product, `state IN ("draft", "confirmed")`, "not planned", the rule's operation type, the request company, "no responsible user", and `references` equal to `values.references`. When `values.production_group` is set, the production group's ancestors must contain it. When `values.orderpoint` is set, the order must additionally satisfy: either it is a draft whose deadline is not later than *procurement date*, or it is confirmed and its start date is not later than *procurement date*, where *procurement date* is the end of the day `values.date_planned` minus the bill's manufacturing lead time.
4. **If no order is found, or the bill enables batch sizes:** create one or more manufacturing orders. The batch size is the bill's batch size converted into the request unit when batch sizes are enabled, otherwise the whole requested quantity. Orders are created repeatedly, each for one batch, until the remaining quantity is not greater than zero.
5. **If an order is found and batch sizes are not enabled:** increase its quantity through the shared "change production quantity" operation of `../manufacturing/`, by the request quantity converted into the product's reference unit and then into the order's unit, and link the order's finished-product moves for this product (excluding completed and cancelled ones) to `values.move_destinations`.
6. **Manufacturing order values** for a newly created order:

| Field | Value |
|---|---|
| `origin` | the request origin |
| `product` | the request product |
| `product_description_variants` | `values.product_description_variants` |
| `never_product_template_attribute_values` | `values.never_product_template_attribute_values` |
| `product_quantity` | the batch quantity converted into the bill's unit when a bill exists, otherwise the request quantity |
| `unit_of_measure` | the bill's unit when a bill exists, otherwise the request unit |
| `location_source` | the default source location of the effective operation type |
| `destination_location` | the default destination location of the effective operation type when set, otherwise the request location; overridden by the rule's `destination_location` when `location_destination_from_rule` is true |
| `location_final` | the request location |
| `bill_of_materials` | the bill of materials |
| `date_start` | `values.date_planned` minus the bill's manufacturing lead time in calendar days; when that subtraction leaves the date unchanged (a lead time of zero), one hour is subtracted instead, so that the production always starts strictly before the need |
| `date_deadline` | `values.date_deadline` when set, otherwise `date_start` plus the bill's manufacturing lead time |
| `references` | `values.references` |
| `propagate_cancel` | the rule's `propagate_cancel` |
| `orderpoint` | `values.orderpoint` |
| `operation_type` | the bill's operation type when set, otherwise the rule's operation type, otherwise the manufacturing operation type of `values.warehouse` |
| `company` | the request company |
| `move_destinations` | `values.move_destinations` |
| `responsible_user` | empty |

   The effective operation type is the bill's operation type when set, otherwise the rule's operation type.
7. **Create and confirm.** The orders are created with elevated rights under the superuser account and in the company context. An order is confirmed immediately when: it has no component moves and it has no work orders and (it came from a reordering rule or its downstream moves take from stock); or it has component moves and it did not come from a reordering rule. Otherwise it is left as a draft, to be confirmed after the whole scheduler batch has run (see section 12, step 5).

---

## 7. Push application

**Purpose.** When a stock move has been created or completed, send the goods onward according to the push rules.

**Actor.** The system, at move confirmation for negative remainders and after a move is completed for the normal case.

**Steps.** For each move:

1. Determine the warehouse: the move's `warehouse` when set, otherwise the warehouse of the transfer's operation type.
2. When the destination location belongs to a company that is not in the current company set, switch to elevated rights, widen the allowed companies to the user's companies, and drop the warehouse restriction.
3. Read the package types of the packages that contain the move's result packages (including ancestor packages).
4. **Find the push rule** using the algorithm of section 2, with `routes` = the move's `routes` unioned with the routes of those package types, `warehouse` = the warehouse from step 1, `packaging_unit_of_measure` = the move's packaging unit.
5. **Apply the applicability condition.** While the found rule has a non-empty `push_condition` and the move does not satisfy that condition, add the rule to the excluded set and search again, with the extra condition "rule identifier is not in the excluded set". This loop stops when a rule with no condition is found, when a rule whose condition the move satisfies is found, or when no rule is found at all.
6. **Refuse to push a return back where it came from.** If the move is a return (it has an origin returned move) and the found rule's destination location is the destination location of that origin returned move, no push happens.
7. **Run the rule** (see below). The result is the new move, if any.
8. **Rewire the downstream moves.** For each downstream move of the original move that is not the newly created move:
   - when a new move was created, the original move has a final location, and the downstream move's source location equals that final location: the downstream move is moved from the original move to the new move (unlinked from the original move's `move_destinations`, linked to the new move's `move_destinations`);
   - otherwise, when the downstream move's source location is not the destination location of the original move nor a descendant of it: the make-to-order link is broken (see section 22, "Breaking a make to order link").
9. Confirm the newly created moves with elevated rights.

**Running a push rule.**

- **When the rule's `auto` is `transparent` ("Automatic No Step Added"):**
  1. Remember the move's current destination location.
  2. Write on the move: `date` = the move's current `date` plus the rule's `lead_time_days` calendar days; `destination_location` = the rule's `destination_location`.
  3. When the move has move lines, set their destination location to the putaway location computed for the new destination location and the product, or to the new destination location when there is no putaway rule.
  4. When the destination location actually changed, run push application again on the same move, so that a chain of transparent rules collapses in one pass, and return the first move it produces. When it did not change, stop: this prevents an endless loop on a badly configured rule.
- **When the rule's `auto` is `manual` ("Manual Operation"):**
  1. Copy the move with elevated rights, with the following values:

| New move field | Value |
|---|---|
| `demand_quantity` | the completed quantity of the source move; but when the source move's demand quantity is negative, the source move's demand quantity |
| `origin` | the source move's origin, else the name of its transfer, else `/` |
| `source_location` | the source move's destination location |
| `destination_location` | the rule's `destination_location`; replaced by the source move's `location_final` when that final location is a descendant of the rule's destination location; replaced by the customer location of the source move's partner when the source move has a partner and the rule's destination location is a customer location |
| `location_final` | the source move's `location_final`, but only when the source move's destination location is not a descendant of that final location; empty otherwise |
| `rule` | the push rule |
| `date` | the source move's `date` plus the rule's `lead_time_days` calendar days |
| `deadline` | the source move's `deadline` |
| `company` | the rule's company; when the rule has no company, the company of the rule's warehouse, else the company of the rule's operation type's warehouse |
| `transfer` | empty (the move will be assigned to a transfer at confirmation) |
| `operation_type` | the rule's `operation_type` |
| `propagate_cancel` | the rule's `propagate_cancel` |
| `warehouse` | the rule's `warehouse`, else the warehouse of the source move's destination location |
| `procure_method` | `make_to_order` |
| `purchase_line` | cleared; when the rule's destination location is a vendor location, set to the purchase order line found by walking up the move chain, together with that line's vendor as partner |
| `production_group` | the source move's production group; `manufacturing_order` cleared |

  2. When the new move must not be pushed any further (see "the push-skip test" below), rewrite its destination location to its final location.
  3. When the new move's source location bypasses reservation (a vendor, customer, production, inventory-loss or transit location), set its supply method to `make_to_stock`.
  4. When the new move's source location does not bypass reservation, link the new move as a destination of the source move.
  5. Return the new move.

**The push-skip test.** A move is not pushed further when it is an inventory adjustment move, or when it already has a downstream move whose source location is the move's destination location or one of its ancestors or descendants. The second condition prevents creating a duplicate step when the chain has already been built by a pull rule.

---

## 8. Confirming a stock move and creating the supply need

**Purpose.** Move a draft stock move into its running state and, when required, create the need that will supply it.

**Steps.** For a batch of moves:

1. **Classify each draft move.**
   - If the move already has origin moves: it becomes *waiting*.
   - Else if its supply method is `make_to_order`: it becomes *waiting* and a procurement request is created for it.
   - Else if its rule's supply method is `mts_else_mto`: it becomes *confirmed* and a procurement request is created for it.
   - Else: it becomes *confirmed*.
   Moves that are not drafts are left alone.
2. **Compute the quantity to procure** for each move that needs a request:
   - For a move whose rule's supply method is not `mts_else_mto`, the quantity is the move's demand quantity.
   - For a move whose rule's supply method is `mts_else_mto`:
     - when the move's real quantity is zero or negative, or when the move's source location bypasses reservation, the quantity is the full demand quantity;
     - otherwise: read the free quantity of the product at the move's source location once per (location, product) pair; subtract from it the quantity already consumed by earlier moves of the same batch for that pair; the result, floored at zero, is the *available quantity*; the quantity to procure is `max(move real quantity − available quantity, 0)`, converted into the move's unit with half-up rounding; and the consumed amount for that pair is increased by `min(move real quantity, available quantity)`.
3. **Build the procurement request values** for each such move:

| Key | Value |
|---|---|
| `date_planned` | the dates-information result (see below) |
| `date_order` | the dates-information result |
| `date_deadline` | the move's `deadline` |
| `move_destinations` | the move itself, but only when the move's supply method is `make_to_order`; empty otherwise |
| `partner` | the move's partner, resolved as follows, and only when the rule's supply method is `make_to_order` or `mts_else_mto`: when the move's source location is the company's internal transit location, the partner of the destination location's warehouse; otherwise the move's own partner |
| `routes` | the move's `routes`; when the move has none, the routes of the package types of the packages holding the move's result packages |
| `warehouse` | the move's `warehouse`, else the warehouse of the move's operation type; when the move's source location belongs to no warehouse, the supplier warehouse of the move's rule's route instead |
| `priority` | the move's priority |
| `references` | the move's references |
| `orderpoint` | the move's reordering rule |
| `packaging_unit_of_measure` | the move's packaging unit |
| `procurement_values` | the move's carried procurement values |
| `product_description_variants` | the move's picking description with the product's generic description and picking description removed |
| `never_product_template_attribute_values` | the move's excluded attribute values |

   *Dates information.* When the move's source location belongs to a warehouse and the warehouse's stock location is an ancestor of the move's source location, the dates are computed by walking the rule chain from the move's source location with the move's routes: `date_planned` is the move's own `date`, and `date_order` is that date minus the *purchase delay* accumulated along the chain (the vendor lead time of any buy rule in the chain). Otherwise `date_planned` is the move's own `date` and there is no `date_order`.

   The request name is the rule's name, or `/` when the move has no rule. The request origin is the name of the move's first reference, else the move's origin, else the display name of its transfer. The request location is the move's source location, the quantity is the value computed in step 2, the unit is the move's unit, and the company is the move's company.
4. **Run** the requests. The run is executed with `raise_user_error` set to false when the confirmation is itself part of a reordering-rule run, and true otherwise.
5. **Write the states**: the moves classified *confirmed* get state `confirmed`, the moves classified *waiting* get state `waiting`; cancelled moves are excluded from both.
6. **Set the reservation date.** Every move just moved to `confirmed` or `waiting` whose operation type reserves at confirmation gets `reservation_date` = today.
7. **Assign transfers.** Moves that should be assigned are grouped by (set of references, source location, destination location) and each group is attached to a transfer.
8. **Check company consistency** across the batch.
9. **Merge** the moves with their siblings, unless merging was disabled by the caller.
10. **Handle negative moves.** For each merged move whose demand quantity is negative:
    1. When it has a final location that differs from its destination location, push it first (section 7), which produces new push moves.
    2. Swap its source and destination locations, set its final location to its (new) destination location, rebuild its origin and destination links by re-orienting each linked move according to the sign of its own quantity, flip the sign of its demand quantity, switch its operation type to the return operation type when one is configured, and set its supply method to `make_to_stock`.
    3. Assign it to a transfer.
11. **Reserve where possible.** Every move now in state `confirmed` or `partially_available` whose source location bypasses reservation, or which should be reserved at confirmation, is reserved.
12. **Confirm the new push moves.** Positive push moves are confirmed normally; negative push moves are confirmed with merging restricted to the siblings of their origin moves.

---

## 9. Make to order versus multi-step routes

Two different mechanisms create a chain of moves. A replacement must implement both, because they behave differently.

| | Make to order chain | Multi-step route chain |
|---|---|---|
| Trigger | A move whose supply method is `make_to_order` is confirmed. | A move is completed and a push rule applies to its destination location. |
| Direction | Backwards: the downstream move exists first, the origin document is created to supply it. | Forwards: the origin move exists first, the downstream move is created to continue. |
| Link | The created move lists the triggering move in `move_destinations`. | The created move is listed in the completing move's `move_destinations`. |
| Reservation | The downstream move waits for the origin move; it reserves nothing from general stock. | The downstream move is created with supply method `make_to_order` and is therefore bound to the move that created it. |
| Quantity | The full demand quantity, except for `mts_else_mto` where only the missing part is chained. | The quantity actually completed on the origin move (or the full negative demand for a negative move). |
| Cancellation | Governed by the rule's `propagate_cancel`, evaluated downstream. | Same. |
| Typical use | Buying or manufacturing specifically for one sales order. | Two-step reception, three-step reception, two-step and three-step delivery. |

**Worked example: a two-step reception chain.** A warehouse is configured to receive in two steps. Its reception route then holds:

1. a pull rule "Vendors → Input", supply method `make_to_stock`, operation type Receipt, source the vendor location, destination the warehouse stock location, `propagate_cancel` true, `location_destination_from_rule` false (the receipt therefore actually lands in the Input location, which is the operation type's default destination, while the rule's destination location, the stock location, is written on the move as its final location);
2. a push rule "Input → Stock", operation type Internal Transfer (storage), source the Input location, destination the stock location, `propagate_cancel` false (the last rule of a cancel-propagating chain always has propagation switched off, so that cancelling a receipt never cancels a delivery).

A purchase order for 12 units is confirmed:

1. The receipt move is created from the vendor location to the Input location, with `location_final` = the warehouse stock location, scheduled on the expected arrival date.
2. Confirming the receipt creates no supply need (supply method `make_to_stock`).
3. When the receipt is validated, push application finds the push rule at the Input location and creates a second move Input → Stock for the quantity actually received (12), with `procure_method` = `make_to_order`, `date` = the receipt date plus the push rule lead time, `deadline` = the receipt deadline, and `move_destinations` on the receipt pointing at it.
4. Increasing the purchase order line from 12 to 15 after validation creates a second receipt move for the extra 3; validating that second receipt pushes another 3 into the existing downstream transfer, which then shows 15.

---

## 10. Adjusting the supply method of an existing move

**Purpose.** Given moves that already exist (typically the moves of a manually created transfer, or of a transfer whose operation type has just changed), decide whether each of them should take from stock or trigger a rule.

**Steps.** For each move:

1. Set *location* to the move's source location.
2. While *location* is not empty:
   1. Build the condition `location_source = <location>` AND `destination_location = <the move destination location>` AND `action ≠ "push"`, plus `operation_type.code = <the requested code>` when the caller supplied one.
   2. Search for a rule with the four-pass source order of section 2, using no explicit routes, the move's packaging unit, the move's product, and the move's warehouse (or the warehouse of its operation type).
   3. If a rule is found, stop.
   4. Otherwise set *location* to its parent and repeat.
3. If no rule was found at all, set the move's supply method to `make_to_stock` and stop.
4. Otherwise write the rule on the move, and set the move's supply method to the rule's supply method when that is `make_to_stock` or `make_to_order`, or to `make_to_stock` when the rule's supply method is `mts_else_mto`.

---

## 11. Reordering rule evaluation

**Purpose.** Decide, for one reordering rule, whether anything must be ordered, and how much.

**Steps.**

1. **Compute the rule chain.** Run the rule-chain walk of `calculations.md`, section "The rule chain from a location". Set the current location to the rule's `source_location` and repeat the following until it stops: select the rule for the product at the current location with `routes` = the rule's `route` and `warehouse` = the rule's warehouse; when no rule is found, stop; when the found rule takes from stock, or its action is neither `pull` nor `pull_push`, add it to the chain and stop; otherwise add it to the chain, set the current location to the found rule's `location_source` and repeat. A rule that appears twice means a misconfiguration and raises `Invalid rule's configuration, the following rule causes an endless loop: <rule display name>`.
2. **Compute the lead days** from that chain (formula in `calculations.md`, section "Lead days"). Compute `lead_horizon_date` = today plus lead days plus the company's replenishment horizon.
3. **Read the forecast** of the product at `source_location` at the end of `lead_horizon_date`, and add the quantity in progress that the forecast does not see (draft, sent and to-approve purchase order lines targeted at this rule or at this location; pending manufacturing orders). The result is `quantity_forecast`.
4. **Decide.** When `quantity_forecast` is not lower than `product_minimum_quantity` at the product unit's rounding, nothing is ordered and `quantity_to_order_computed` is set to zero (treated as empty).
5. **Otherwise compute the quantity to order** (formula and worked examples in `calculations.md`, section "Quantity to order").
6. `quantity_to_order` is the manual override when one is set and non-zero, otherwise the computed quantity.

**Precondition for the rule to be executed at all.** The scheduler only evaluates rules whose `trigger` is `auto` and whose product is active. Manual rules are evaluated for display on the replenishment report but are only executed when a user presses Order.

---

## 12. The scheduler

**Purpose.** The daily automated pass that runs every automatic reordering rule, reserves what can be reserved, and cleans up the stock quantity records.

**Actor.** The scheduled action named "Procurement: run scheduler", running under the superuser account, every 1 day.

**Steps.**

1. **Announce three tasks** to the progress tracker when running in batch mode.
2. **Task 1: minimum stock rules.**
   1. Select every reordering rule matching `trigger = "auto" AND product.active = true`, restricted to one company when the caller named one.
   2. Recompute `quantity_to_order_computed` and `deadline_date` on all of them, with elevated rights.
   3. Commit the progress of task 1.
   4. Run the reordering-rule procurement pass over them (section 12.1), with `raise_user_error` set to false.
3. **Task 2: reserve waiting moves.**
   1. Select every stock move matching: the caller's company when one was named; `state IN ("confirmed", "partially_available")`; `demand_quantity ≠ 0`; and (`reservation_date ≤ today` OR the operation type reserves at confirmation). When the Manufacturing capability package is installed, moves that belong to a manufacturing order as a finished-product move are excluded.
   2. Order them by `reservation_date` ascending, then `priority` descending, then `date` ascending, then `identifier` ascending.
   3. Reserve them in chunks of one thousand, committing after each chunk when running in batch mode, and logging `A batch of <n> moves are assigned and committed`.
   4. Commit the progress of task 2.
4. **Task 3: merge duplicated stock quantity records** by calling the shared quantity maintenance operation of `../inventory-operations/`, then commit the progress of task 3.
5. Any exception during the three tasks is logged with its stack trace and re-raised, which aborts the scheduled run.

### 12.1 The reordering-rule procurement pass

**Input.** A set of reordering rules, a company, and the `raise_user_error` flag.

**Steps.**

1. Work in the named company's context.
2. Split the rules into batches of one thousand identifiers. When running in batch mode, each batch gets its own database cursor and is committed independently, logging `A batch of <n> orderpoints is processed and committed` afterwards.
3. For each batch, repeat the following until it succeeds or until no rule remains:
   1. **Build the requests.** For every rule in the batch whose `quantity_to_order` is strictly positive at the product unit's rounding:
      - *origin*: when the caller supplied a set of originating stock references for this rule, the rule display name, then " - ", then the reference names joined with commas; otherwise the rule's `name`;
      - *procurement date*: `lead_horizon_date` at 12:00 in the time zone of the company's partner (falling back to coordinated universal time when the partner has no time zone), converted to coordinated universal time; then, when the company's replenishment horizon is not zero, that date minus the horizon in days. This yields the date at which the goods are actually needed, as opposed to the date at which the forecast was read;
      - *values*: `routes` = the rule's `route`; `date_planned` and `date_order` from the dates-information rule applied to the procurement date, the rule's `source_location` and the rule's route; `date_deadline` = the procurement date; `warehouse` = the rule's warehouse; `orderpoint` = the rule; `forced_vendor_price` = the rule's `vendor_price`; `references` = the stock references supplied by the caller for this rule, when any; `partner` = the single subcontractor of the rule's location when that location is a subcontracting location with exactly one subcontractor;
      - the request itself: product, `quantity_to_order`, the rule's unit, the rule's `source_location`, the rule's `name`, the origin, the rule's company, the values.
   2. **Run** the requests inside a savepoint, with the reordering-rule marker set in the context (this marker is what makes a missing vendor a hard error rather than a silent fallback, and what makes a nested move confirmation tolerate failures).
   3. **On a procurement exception:** collect the (reordering rule, message) pairs; remove the failing rules from the batch; if no failing rule could be identified, log `Unable to process orderpoints` and abandon the batch; otherwise loop back to step 3.1 with the reduced batch, so that the rules that can be processed still are.
   4. **On a database serialization error:** when running in batch mode, roll the batch cursor back and retry the batch; otherwise re-raise.
   5. **On success:** run the post-processing step (see below) and leave the loop.
4. **Create warning activities.** For every collected (reordering rule, message) pair: search for an existing activity on the product template of the rule's product whose note contains the message. When none exists, schedule a warning activity on that product template, under the superuser account, with the message as its note, assigned to the product's responsible person, or to the superuser when the product has no responsible person.

**Post-processing.** The base behavior returns immediately. The Manufacturing capability package extends it: every manufacturing order created by the batch that was left as a draft is confirmed now, after all reordering rules have run. Confirming earlier would let the component needs of one manufacturing order interfere with the reordering rules still to be processed in the same batch.

### 12.2 The event-driven trigger

Besides the daily scheduled action, automatic reordering rules are also triggered by stock movements. When a batch of moves is confirmed through a transfer:

1. For each move, search for at most one reordering rule matching: the move's product; `trigger = "auto"`; `source_location` is the move's source location or one of its ancestors; the move's company; and `source_location` is **not** the move's destination location nor one of its ancestors (an internal transfer inside the watched location creates no need).
2. Collect the rules found, grouped by company.
3. For a move whose real quantity is greater than the rule's minimum quantity and which carries stock references, remember those references against the rule, so that the created documents can be traced back to the originating document.
4. For each company, run the reordering-rule procurement pass over the collected rules, with the remembered references in the context and with `raise_user_error` set to false.

This trigger is disabled entirely when the stored parameter `inventory.disable_automatic_scheduler` is set.

---

## 13. Manual replenishment: Order and Order to Max

**Actor.** An inventory user, on the replenishment report or on a reordering rule form.

**Preconditions.** At least one reordering rule is selected.

### 13.1 Order

1. Remember the current moment.
2. Run the reordering-rule procurement pass for the selected rules in the active company, with `raise_user_error` true.
3. **On a user-facing error:** when more than one rule was selected, the error is shown as is. When exactly one rule was selected, the error is shown as a redirect warning whose button is labelled `Edit Product` and opens the product form of the rule's product, so that the user can add a vendor or a route.
4. **On success, and only when exactly one rule was selected**, build the notification:
   - search for one purchase order line created or modified since the remembered moment and linked to that rule; when found, return a notification titled `The following replenishment order has been generated` whose single link is the purchase order display name and points at that order;
   - otherwise search for one stock move created or modified since the remembered moment and linked to that rule; when the move's source location belongs to a different warehouse than the rule's, or is a transit location, and the move has a transfer, return a notification titled `The inter-warehouse transfers have been generated` whose single link is the transfer name and points at that transfer;
   - otherwise return no notification.
5. Clear the manual quantity override on the selected rules and recompute `quantity_to_order`.
6. Delete every selected rule that was created by the superuser, has `trigger` equal to `manual`, and whose `quantity_to_order` is now zero or below.

### 13.2 Order and set to automatic

Sets `trigger` to `auto` on the selected rules, then performs Order exactly as above.

### 13.3 Order to Max

1. For every selected rule, force `quantity_to_order` to the multiple-rounded value of `product_maximum_quantity` minus `quantity_forecast`. Because writing `quantity_to_order` runs its inverse rule, this is stored as a manual override on manual rules and is ignored (reset to zero) on automatic rules.
2. Continue with steps 2 to 6 of Order.

**Worked example.** A product has 10 units on hand, a rule with minimum 5 and maximum 200. `quantity_forecast` is 10 and `quantity_to_order` is 0, because 10 is not below 5. Pressing Order changes nothing; the forecast stays at 10. Pressing Order to Max forces the quantity to `200 − 10 = 190` and creates the documents; the forecast becomes 200. Setting the multiple to "Dozens" and the maximum to 240 and pressing Order to Max again computes `240 − 200 = 40` units, converts to `40 ÷ 12 = 3.33` dozens, rounds up to 4 dozens, converts back to 48 units, and the forecast becomes 248.

---

## 14. Snoozing a reordering rule

**Actor.** An inventory user on the replenishment report.

1. The user selects one or more rules and opens the snooze wizard.
2. The wizard proposes `1 Day` and sets `snoozed_until` to tomorrow. Choosing `1 Week` sets it to today plus one week; `1 Month` sets it to today plus one month; `Custom` leaves the date for the user to type.
3. Confirming writes `snoozed_until` on every selected rule.
4. If any selected rule has `trigger` equal to `auto`, the write is refused with `You can only snooze manual orderpoints. You should rather archive 'auto-trigger' orderpoints if you do not want them to be triggered.`
5. A snoozed rule is filtered out of the replenishment report until the snooze date has passed. The scheduler does not consider snoozed rules either, because only automatic rules are scheduled and automatic rules can never be snoozed.

---

## 15. The replenishment report

**Purpose.** Show, for every replenishment location, the products whose forecast is negative, as temporary manual reordering rules that a user can adjust and order.

**Actor.** An inventory user opening Operations → Replenishment.

**Steps.**

1. Read every reordering rule, including archived ones.
2. **Delete satisfied temporary rules:** every rule created by the superuser with `trigger` equal to `manual` whose `quantity_to_order` is zero or below is deleted. The deleted rules are removed from the working set.
3. When the opening context asks for a forced recomputation, recompute `quantity_to_order_computed` and `deadline_date` on the remaining rules.
4. **Determine the scope:** every goods product that has at least one stock move ("all products"), and every location whose `replenish_location` flag is true ("all replenishment locations").
5. **Find the shortages, pass one (fast filter).** Read, for all products at once:
   - the stock quantity records inside those locations, summed per product and location;
   - the incoming moves in state `waiting`, `confirmed`, `assigned` or `partially_available` whose destination or final location is inside those locations, or which are internal to them, summed per product, destination location and final location;
   - the outgoing moves in the same states whose source location is inside those locations, or which are internal to them, summed per product and source location.
   For each (location, product) pair, compute `on hand + incoming − outgoing` counting only the records whose location is the replenishment location or a descendant of it. When that value is negative at the product unit's rounding, the pair is a candidate.
6. **Group the candidates by lead time.** For each candidate, compute the rule chain and the lead days at that location, and group the candidate products by (lead days plus replenishment horizon, location).
7. **Find the shortages, pass two (exact).** For each group, read the forecast of its products at that location at the end of the day `today + <the group's day count>`. Keep the pairs whose forecast is still negative; the shortage is the forecast value (a negative number).
8. **Deduct what is already in progress.** For the surviving pairs, read the quantity in progress per (product, location) from sources the forecast does not see (draft, sent and to-approve purchase order lines), and add the `quantity_to_order` of the existing reordering rules for the same pair. When that total is zero, the pair keeps its shortage unchanged. Otherwise the shortage becomes `shortage + quantity in progress`. Pairs whose result is no longer negative at the Product Unit precision are dropped.
9. **Create or top up rules.** For each remaining pair:
   - when a reordering rule already exists for it (including archived ones), that rule's `quantity_forecast` is increased by the (negative) shortage, which makes the report show the extra need on the existing rule;
   - otherwise create a rule, under the superuser account, with: the product; the location; `product_maximum_quantity` 0.0; `product_minimum_quantity` 0.0; `trigger` `manual`; `name` `Replenishment Report`; `warehouse` = the warehouse of the location, or the first warehouse of the location's company when the location has none; `company` = the company of the location.
10. Open the replenishment report view.

**Worked example.** A warehouse receives in one step; a delivery of 3 units of a product is confirmed from the stock location to a replenishment sub-location, and no other stock exists. Opening the replenishment report finds a negative forecast of −3 at the warehouse stock location (the source of the internal transfer) and creates one temporary rule at that location with a quantity to order of 3.

---

## 16. The Replenishment Information wizard

**Actor.** An inventory user clicking the information icon on a line of the replenishment report.

**Steps.**

1. The system creates a Replenishment Information record pointing at the reordering rule and opens it in a dialog titled `Replenishment Information for <product display name> in <warehouse display name>`.
2. The dialog shows:
   - the lead-time breakdown, built by running the lead-days computation on the rule chain **with** descriptions enabled, then converting it into a list of rows (see `interfaces.md`, "Replenishment Information payload") where each row is either a labelled number of days or a labelled date obtained by walking the description backwards from today;
   - the forecast date (`lead_horizon_date`), today's date, the trigger, the forecast quantity, the quantity to order, the minimum and maximum quantities formatted at the Product Unit precision, the unit name, and a flag saying whether the rule is a temporary rule (trigger `manual` and created by the superuser);
   - the demand graph (formulas in `calculations.md`, section "Replenishment demand graph");
   - a Warehouses tab listing one Replenishment Option per inter-warehouse resupply route of the rule's warehouse, sorted by free-to-use quantity descending, each showing the supplying warehouse, its free quantity, the unit, the quantity to order and the lead time;
   - a Vendors tab listing every Vendor Price of the product, shown when the rule has no preferred route or when the selected rules include a `buy` rule;
   - a Bills of Materials tab listing every bill that could produce the product, shown under the equivalent condition for `manufacture`.
3. Editing the minimum or maximum quantity in the dialog writes straight back onto the reordering rule, with the current user's rights.
4. Editing `based_on` or `percent_factor` recomputes the demand graph.
5. **Choosing a supplying warehouse.** Pressing Select Route on a Replenishment Option:
   - when the option's free quantity is lower than the quantity to order, opens a warning form titled `Quantity available too low`, showing `<warehouse name> can only provide <free quantity> <unit>, while the quantity to order is <quantity to order> <unit>.` and offering two buttons: "order the available quantity", which writes the route on the rule and sets the quantity to order to the free quantity, and "order everything", which writes the route only;
   - otherwise writes the route on the rule and closes.
   - When the wizard was opened from the Product Replenish wizard, the route is written on that wizard instead and the wizard is reopened.
6. **Choosing a vendor.** Pressing "Set as Supplier" on a Vendor Price row writes that vendor price on the rule, which also sets the rule's route to a Buy route when it had none, and recomputes the quantity to order in the vendor's unit.

---

## 17. The Product Replenish wizard

**Actor.** Any user with inventory rights, from a product form or from the forecast report.

**Steps.**

1. Opening the wizard fills: the product and template from the context; the company from the product template or the active company; the unit from the product template; the warehouse, the first warehouse of the company; and the route, from the reordering rule that already exists for that product and warehouse when there is one, otherwise the first route satisfying the allowed-route condition, otherwise the first route of the product template that belongs to the active company or to no company. When a reordering rule exists and carries a vendor price, that vendor price is proposed too.
2. **Allowed routes.** A route is offered when it is selectable on products, or is one of the warehouse routes that is a valid resupply route for the product; and none of its rules has the shared inter-company location as source or destination; and every one of its rules has a destination location that belongs to a warehouse. When the Purchase Inventory capability package is installed, the routes of `buy` rules of the company whose operation type has code `incoming` are additionally offered when the product has at least one vendor price. The global Dropship route is always excluded.
3. **Quantity default.** When the product's forecast in that warehouse is negative, the default quantity is the absolute value of the forecast; otherwise 1.
4. **Scheduled date.** The default is the current moment plus the sum of the `lead_time_days` of every rule of the chosen route. When the chosen route is a buy route and a vendor price is chosen, the vendor's lead time and the company's days to purchase are added on top.
5. **On-change on the route.** When the chosen route is a buy route and no vendor price is set, the first vendor price of the product template is proposed. When the chosen route is not a buy route, the vendor price is cleared.
6. **Launching the replenishment** builds a single procurement request: the product; the typed quantity; the typed unit; the warehouse's stock location; the name `Manual Replenishment`; the origin `Manual Replenishment`; the warehouse's company; and the values `warehouse` = the warehouse, `routes` = the chosen route, `date_planned` = the typed scheduled date, `force_unit_of_measure` = true, plus `forced_vendor_price` = the chosen vendor price when one is set. The request is run with a cleaned context.
7. **Notification.** After the run, the wizard searches for a purchase order line written since the start of the operation; when found it returns a notification titled `The following replenishment order have been generated` linking to that purchase order. Otherwise it searches for a stock move written since the start, and links to that move's transfer when it has one. When neither exists it simply closes.
8. The wizard closes with an indication that the operation completed.

---

## 18. Drop shipping

Drop shipping ships goods straight from the vendor to the customer, without ever entering the company's warehouse.

### 18.1 Configuration created by the capability package

Per company:

1. A sequence with code `dropship_transfer`, named `Dropship (<company name>)`, prefix `DS/`, five digits.
2. An operation type named `Dropship`, with code `dropship`, no warehouse, that sequence, prefix `DS`, default source location = the shared vendor location, default destination location = the shared customer location, and "use existing lots" switched off.
3. A `buy` rule inside the global Dropship route, named `<vendor location name> → <customer location name>`, with destination = the customer location, source = the vendor location, supply method `make_to_stock`, the company's drop shipping operation type, and the company.

Globally, a route named `Dropship`, sequence 20, no company, selectable on sales order lines, on products and on product categories.

### 18.2 Flow: a drop-shipped sales order

**Actors.** A salesperson, then a purchase user.

1. The salesperson creates a sales order line for the product and selects the Dropship route on the line (or the product carries the route).
2. Confirming the order creates a procurement request at the customer's delivery location, with `routes` = the line routes, `sales_order_line` = the line, `partner` = the order's shipping address, `date_deadline` = the promised delivery date, `date_planned` = that date minus the sales safety days.
3. Rule selection at the customer location returns the Dropship `buy` rule (the Dropship route has sequence 20 and is normally the only route whose rules have a customer destination that is selectable on the line).
4. **Vendor selection ignores the request partner**: for a rule belonging to the global Dropship route, the contact restriction passed to vendor selection is always cleared, because the request partner is the customer, not a vendor.
5. The buy action creates or extends a draft purchase order whose `operation_type` is the drop shipping operation type and whose `destination_address` is the customer shipping address. Because the operation type has code `dropship`, the grouping key always carries the reference component, so needs from different sales orders only merge when they share a stock reference.
6. Confirming the purchase order creates the transfer: source = the vendor's vendor location, destination = the customer location of the order's destination address, operation type = the drop shipping operation type. The transfer's `is_dropship` flag is therefore true.
7. **Multi-order split.** When a confirmed drop shipping purchase order covers lines from more than one sales order, has no live transfer yet, and holds at least one goods product, one transfer is created per sales order instead of a single one. Each transfer receives only that sales order's lines, its moves are confirmed, numbered by ascending date in steps of five, reserved, and a message linking back to the purchase order is posted. Every created transfer plus the transfers impacted by the moves is then confirmed.
8. Validating the drop shipment marks the purchase order line received and the sales order line delivered at the same time.

### 18.3 Consequences elsewhere

- A purchase order counts drop shipments separately: `dropship_transfer_count` counts the transfers whose `is_dropship` is true and `incoming_transfer_count` excludes them. The same split is applied on the sales order between deliveries and drop shipments.
- The delivered quantity of a sales order line whose purchase order lines are drop shipped is taken from those purchase order lines rather than from stock moves (see `entities.md`, section 18).
- A sales order line is flagged make to order when any rule of its routes has an operation type going from a vendor location to a customer location.
- The stock reference created for a drop shipping purchase order additionally links the single sales order when the order's lines all belong to one sales order.
- A drop-shipped move's product description uses the outgoing picking description of the product rather than the incoming one.
- The Dropship route is never offered by the Product Replenish wizard.

---

## 19. Inter-warehouse resupply

**Purpose.** Let one warehouse be supplied by another through a transit location.

**Trigger.** A user adds a warehouse to `resupply_warehouses` of another warehouse, or creates a warehouse with that field filled.

**Steps.** For the supplied warehouse and each supplying warehouse:

1. **Choose the transit location:** the company's internal transit location when both warehouses belong to the same company; the shared inter-company transit location otherwise. When neither exists, the pair is skipped. The chosen transit location is activated.
2. **Choose the supplying warehouse's output location:** its stock location when it delivers in one step, its Output location otherwise.
3. **Create an extra make-to-order rule when the supplying warehouse delivers in one step.** In that case the global "Replenish on Order" route gains a rule from the output location to the transit location, with the supplying warehouse's outgoing operation type, supply method `make_to_order`, name suffix "Make To Order", and the create values of the warehouse's make-to-order rule (active, pull, manual, carrier propagation on). Warehouses that deliver in two or three steps already have such a rule.
4. **Create the resupply route**, with: name `<supplied warehouse name>: Supply Product from <supplying warehouse name>`, selectable on warehouses, on products and on product categories, `supplied_warehouse` = the supplied warehouse, `supplier_warehouse` = the supplying warehouse, company = the intersection of the two warehouses' companies (empty when they differ).
5. **Create the pull rules of the route:**
   1. From the supplying warehouse's output location to the transit location, with the supplying warehouse's outgoing operation type, `location_destination_from_rule` true, and supply method computed by the resupply rule: `make_to_stock` when the source location is the supplying warehouse's stock location, `make_to_order` otherwise.
   2. When the supplying warehouse does not deliver in one step, a second rule from the supplying warehouse's stock location to its output location, with its picking operation type, supply method `make_to_stock` (the source is the stock location).
   3. From the transit location to the supplied warehouse's stock location, with the supplied warehouse's incoming operation type, supply method `make_to_order` (the source is not the supplied warehouse's stock location).
   Each rule is created active.
6. The route is now selectable on the product, on the category, or on a reordering rule. Selecting it makes the chain "supplying warehouse stock → (output) → transit → supplied warehouse stock" the way the product is replenished.

**Partner stamping.** When the pull action creates the move into the transit location and the request location is the company's internal transit location, it stamps the destination warehouse's partner on the new move and the source warehouse's partner (or the company partner) on the downstream moves, so that the two transfers show the two warehouses as counterparties.

**Changing the number of delivery steps of a supplying warehouse.** When the delivery steps of a warehouse change and the change crosses the boundary between one step and several steps:

- Every rule of every route that this warehouse supplies, which is not a push rule and whose destination is a transit location, has its source location rewritten to the new output location, and its supply method set to `make_to_order` when moving to several steps or `make_to_stock` when moving back to one step.
- **Moving back to one step:** the extra "stock → Output" rules of those routes (identified by destination = the Output location and the picking operation type) are archived; new make-to-order rules from the stock location to each transit destination are created with the outgoing operation type.
- **Moving to several steps:** those extra rules are unarchived; for every supplied route that had none, a new "stock → new output location" rule is created with the picking operation type; and every rule of the global "Replenish on Order" route whose destination is a transit location and whose source is this warehouse's stock location is archived, so that it can no longer be used.

---

## 20. Subcontractor resupply and drop shipping to a subcontractor

This section specifies only the routes and rules; the subcontracting production flow itself is owned by `../manufacturing/`.

### 20.1 Resupply subcontractor on order

Each warehouse carries a flag "Resupply Subcontractors". While it is on, the warehouse holds two rules in the global resupply-subcontractor route: a make-to-order pull rule and a make-to-stock pull rule that move components from the warehouse to the subcontracting location. Turning the flag off archives them.

### 20.2 Drop ship components straight to a subcontractor

The Dropship and Subcontracting Management capability package adds, per company:

1. A sequence with code `dropship_subcontractor_transfer`, named `Dropship Subcontractor (<company name>)`, prefix `DSC/`, five digits.
2. An operation type named `Dropship Subcontractor`, code `dropship`, no warehouse, that sequence, prefix `DSC`, default source location = the shared vendor location, default destination location = the company's subcontracting location, "use existing lots" off. It is remembered on the company as `dropship_subcontractor_operation_type`.
3. A `buy` rule in the global Dropship route named `<vendor location name> → <subcontracting location name>`, destination = the subcontracting location, source = the vendor location, supply method `make_to_stock`, that operation type, that company.

Per warehouse, while "Resupply Subcontractors" is on, a pull rule inside the same global route named after the subcontracting and production locations, from the subcontracting location to the production location, supply method `make_to_order`, with the warehouse's subcontracting operation type.

**Route and operation type activation.** Whenever a warehouse is created with resupply switched on, or the resupply flag or the active flag of a warehouse is written:

1. The warehouse's own subcontracting dropship pull rules are archived or unarchived to match the flag (archiving a warehouse is skipped, because archiving already archives its rules).
2. Per company, the company's drop-ship-subcontractor operation type is active exactly when that company still has at least one active pull rule in the route.
3. The route itself is active exactly when at least one active pull rule remains in it, across every company.

**Purchase order preparation.** When the buy action prepares a purchase order and no partner is carried in the values, and the rule's destination location is the company's subcontracting location or a subcontracting location, and the first downstream move belongs to a manufacturing order that has a subcontractor, that subcontractor becomes the purchase order's destination address.

**Grouping.** When the rule's source is a vendor location, its destination is a subcontracting location, and the values carry a partner, the purchase order grouping key gains the component `destination_address = <that partner>`, so that components for different subcontractors are never merged into one order.

**Reordering rules at a subcontracting location.** When a reordering rule's location is a subcontracting location with exactly one subcontractor, the procurement values gain `partner` = that subcontractor.

---

## 21. Warehouse creation and reconfiguration

**Actor.** An inventory administrator.

**Steps on creation.** After the warehouse locations, sequences and operation types have been created (owned by `../inventory-operations/`), this domain contributes:

1. **The reception and delivery routes.** For each of the two route slots:
   1. When the slot already holds a route, update it and archive all of its rules; otherwise create the route.
   2. Look up the routing list for the current step value (see the table below) and create or unarchive one rule per routing entry.
   3. When the route is selectable on warehouses, add it to the warehouse's `routes`.

| Step value | Routings (source → destination, operation type, action) |
|---|---|
| `one_step` (Receive and Store) | vendor location → warehouse stock, Receipt, pull |
| `two_steps` (Receive then Store) | vendor location → warehouse stock, Receipt, pull; Input → warehouse stock, Storage, push |
| `three_steps` (Receive, Quality Control, then Store) | vendor location → warehouse stock, Receipt, pull; Input → Quality Control, Quality Control, push; Quality Control → warehouse stock, Storage, push |
| `ship_only` (Deliver) | warehouse stock → customer location, Delivery, pull |
| `pick_ship` (Pick then Deliver) | warehouse stock → customer location, Pick, pull; Output → customer location, Delivery, push |
| `pick_pack_ship` (Pick, Pack, then Deliver) | warehouse stock → customer location, Pick, pull; Packing Zone → Output, Pack, push; Output → customer location, Delivery, push |

   Route values: the reception route is created with `product_category_selectable` true, `warehouse_selectable` true, `product_selectable` false, the warehouse company, sequence 50; its name is the label of the step value and its `active` follows the warehouse. The delivery route is identical except for sequence 60. When the Purchase Inventory capability package is installed, the reception route is created with sequence 9 instead of 50, its rules are created with supply method `make_to_order`, and the routing list loses its first (pull) entry, because the `buy` rule now starts the chain (see 21.1).

   Rule values: name built from the source and destination location names plus an optional suffix; the routing's source, destination, action and operation type; `auto` = `manual`; supply method `make_to_stock` for the **first** routing of a list and `make_to_order` for every following one; the warehouse; the warehouse company. The reception route's rules additionally get `propagate_cancel` true and the delivery route's rules get `propagate_carrier` true. **In any rule list where cancel propagation was requested, the last rule has `propagate_cancel` forced back to false**, so that cancelling a receipt never cascades past the end of the chain into an unrelated delivery.

   Rule reuse: before creating a rule, an archived rule with the same operation type, source location, destination location, route and action is looked for; when one exists it is unarchived instead of creating a duplicate.

2. **The global route rules.** For each global rule slot of the warehouse, when the slot is empty a rule is created, otherwise the existing rule is updated:

| Slot | Route | Create values | Update values |
|---|---|---|---|
| `make_to_order_pull` | "Replenish on Order" | active, supply method `make_to_order`, the company, action `pull`, `auto` `manual`, carrier propagation true | name from the stock location and the delivery destination with suffix "Make To Order"; destination = the destination of the first delivery routing that starts at the stock location; source = the stock location; operation type = that routing's operation type |
| `buy_pull` | "Buy" | action `buy`, the incoming operation type, the company, `propagate_cancel` = (reception steps ≠ one step) | active = `buy_to_resupply`; name from the stock location with suffix "Buy"; destination = the stock location; `propagate_cancel` = (reception steps ≠ one step) |
| `manufacture_pull`, `manufacture_make_to_order_pull`, `pick_before_manufacturing_make_to_order_pull`, `store_after_manufacturing_rule`, `subcontracting_pull`, `subcontracting_make_to_order_pull` | manufacturing and subcontracting global routes | owned by `../manufacturing/` | owned by `../manufacturing/` |
| `subcontracting_dropshipping_pull` | "Dropship" | supply method `make_to_order`, the company, action `pull`, `auto` `manual`, name from the subcontracting and production locations, destination = the production location, source = the subcontracting location, the subcontracting operation type | active = `subcontracting_to_resupply` |

   A slot whose route could not be resolved (the user deleted the global route) is skipped entirely.

   **Resolving a global route** works as follows: take the route registered under the well-known marker; when it does not exist, or it belongs to a different company than the warehouse's, search for an active or archived route whose name is like the given label and whose company is the warehouse company or empty, ordered by company, taking the first; when nothing is found and the marker route exists, duplicate it with the given name, the warehouse company and no rules.

3. **The resupply routes** for every warehouse listed in `resupply_warehouses` (section 19).

**Steps on reconfiguration.** Writing on a warehouse:

1. Changing `reception_steps` activates or deactivates the Input and Quality Control locations; changing `delivery_steps` activates or deactivates the Packing Zone and Output locations.
2. Changing either triggers the resupply check of section 19.
3. Every route slot whose declared dependencies include a written field is rebuilt: its rules are archived and the rules for the new step value are created or unarchived.
4. Every global rule slot whose declared dependencies include a written field is updated.
5. Changing `resupply_warehouses` archives the routes of removed warehouses and creates routes for added ones.
6. Changing the warehouse name rewrites, in every route name, every rule name of those routes, and the make-to-order rule name and the buy rule name, the first occurrence of the old warehouse name by the new one.
7. Changing `buy_to_resupply` to true adds the warehouse to the Buy route's `warehouses`; changing it to false removes it. The `buy_pull` rule's `active` follows the flag.

---

## 22. Cancellation propagation

**Cancelling a stock move.**

1. Refuse when any move in the batch is completed and its destination is not an inventory-loss location: `You cannot cancel a stock move that has been set to 'Done'. Create a return in order to reverse the moves which took place.`
2. Unmark the moves as picked and release their reservations.
3. Set their state to `cancel`.
4. For each cancelled move, look at the sibling moves (the other origin moves of its downstream moves):
   - **When the move has `propagate_cancel` true** and every sibling is already cancelled: cancel the downstream moves whose state is not `done` and whose source location equals this move's destination location; for the remaining downstream moves, set their supply method to `make_to_stock` and unlink this move from their origin moves. When the stored parameter `inventory.cancel_originating_moves` is set, also cancel the origin moves that are not completed.
   - **When the move has `propagate_cancel` false** and every sibling is completed or cancelled: set the downstream moves' supply method to `make_to_stock` and unlink this move from their origin moves.
5. Unless the caller asked to skip it, log a cancellation activity on the non-cancelled origin documents.
6. Clear the origin links of the cancelled moves and set their supply method to `make_to_stock`.

**Breaking a make to order link.** Unlinking one specific origin move from a downstream move: the origin move is removed from the downstream move's `move_origins`, the downstream move's supply method becomes `make_to_stock`, and its state is recomputed from its remaining links and reservation.

**Cancelling a purchase order.** For every line of every order being cancelled (in state draft, sent, to approve or purchase):

1. Every move of the line that is not completed is collected for cancellation.
2. For the line's downstream moves that are not completed and do not go to an inventory-loss location:
   - moves whose rule's route is not the reception route of their destination warehouse are collected to fall back to stock instead (these are the moves of the customer's chain, which must survive);
   - moves that are also fed by another created purchase order line are unlinked from this line rather than cancelled;
   - the remaining moves are cancelled when the line's `propagate_cancel` is true, and collected to fall back to stock otherwise.
3. The collected cancellations are executed, the collected fallbacks are set to `make_to_stock` and their states recomputed, and the order's non-completed transfers are cancelled.
4. For every already completed transfer, a note is posted: `The purchase order <link to the order> this receipt is linked to was cancelled.`

**Deleting a purchase order line.** Its moves are cancelled; downstream moves that are fed by more than one created purchase order line are unlinked from this line; when `propagate_cancel` is true the remaining downstream moves are cancelled, otherwise they fall back to `make_to_stock` and their states are recomputed.

---

## 23. Date and deadline propagation

**Two dates per move.** `date` is the scheduled date, the moment the operation is planned to happen. `deadline` is the latest moment by which it must be completed to keep a downstream promise. The scheduled date is what the warehouse plans against; the deadline is what "late" is measured against.

**How they are first set.**

- A pull rule sets `date` = `values.date_planned` minus the rule lead time, and `deadline` = `values.date_deadline` minus the same lead time.
- A push rule sets `date` = the source move's `date` plus the rule lead time, and copies the source move's `deadline` unchanged.
- A purchase order line sets both the `date` and the `deadline` of its receipt moves to the line's planned date (or the order's planned date when the line has none).

**Propagating a deadline change.** Writing a new deadline on a move:

1. Remember the set of moves already visited in this propagation, to prevent loops.
2. Compute `delta` = the move's current deadline minus the new deadline (zero when the move had no deadline).
3. For every origin and destination move that is neither completed nor cancelled and has not yet been visited, when it has a deadline and `delta` is not zero, subtract `delta` from its deadline. Writing that value propagates recursively.

The effect is that the whole chain shifts by the same amount while keeping the relative offsets that the rule lead times introduced.

**Worked example.** A delivery move and its origin internal move both have a deadline of day 30. Moving the delivery deadline 6 days earlier, to day 24, gives `delta` = 6 days; the origin move's deadline becomes day 24 as well. Completing the origin move afterwards sets its `date` to the moment of completion but leaves both deadlines at day 24.

**Delay alert.** For every move that is neither completed nor cancelled, `delay_alert_date` is the maximum scheduled date among its not-yet-completed origin moves, when that maximum is later than the move's own scheduled date; empty otherwise. A move with a non-empty `delay_alert_date` is shown as late and offers a pop-over listing the responsible origin documents.

**Deadline change notification.** When a deadline change is propagated from one document to another, a note is posted on each affected document, with subject `Deadline updated due to delay on <origin document name>` and body `The deadline has been automatically updated due to a delay on <link to the origin document>.`, authored by the system account. The note is skipped when the most recent message on the document already has the same subject.

**Purchase order line date changes.** Writing `date_planned` on a purchase order line writes the same value as the `deadline` of the line's not-yet-completed moves; when the line has no such moves, the deadline is written on the line's downstream moves instead. Writing `date_planned` through the shared date-update operation only changes the date when the line has no moves at all or has at least one move that is neither completed nor cancelled.

---

## 24. The forecast report

**Purpose.** Explain, for one product (or every variant of one template) in one warehouse, where every unit of forecast stock comes from and where it goes, and let a user reserve or release the origin chain of a specific outgoing document.

**Actor.** Any user with inventory read rights.

**Steps.**

1. **Determine the warehouse:** the warehouse named in the opening context, otherwise the first active warehouse.
2. **Determine the warehouse locations:** every location that is the warehouse's view location or a descendant of it. The warehouse's stock location is the location whose quantities count as free stock; quantities in the other warehouse locations count as in-transit stock.
3. **Build the header** (see `interfaces.md`, "Forecast report payload"): the product display names and variant names, the on-hand quantity, the forecast quantity, the free quantity, the incoming and outgoing quantities, the draft incoming and outgoing quantities, and the lead time with its breakdown.
4. **Build the lines** by running the reconciliation algorithm of `calculations.md`, section "Forecast reconciliation".
5. **Reserve or release a chain.** Two operations are offered on a line that carries an outgoing move:
   - reserve: take every origin move of that outgoing move, recursively, keep those whose state is not draft, cancelled, assigned or done, and reserve them;
   - release: take the same set, keep those whose state is not draft, cancelled or done, and release their reservations.

---

## 25. Return to the vendor

**Actors.** An inventory user or a purchase user, on a completed receipt.

**Preconditions.** The receipt is completed. It carries at least one move that is neither cancelled nor targeted at an inventory-loss location and that has not already been fully returned. The receipt belongs to a purchase order, which is what makes the Return button visible on a receipt in the first place.

**Entry point.** The Return Wizard, owned by `../inventory-operations/`, opened from the receipt. This section documents only what this domain adds: the link back to the purchase order line, the vendor as counterparty, and the effect on the received quantity. The generic mechanics of the wizard (building the line list, copying the transfer, linking the return move to the original move and to its siblings) belong to the owning domain.

### 25.1 Creating the return

1. The wizard offers one line per returnable move, each with a quantity of zero and the flag `to_refund` ("Update Quantities on Purchase Order") set to true.
2. The user types the quantities to return, or presses "Return all", which fills each line with the completed quantity of its move minus what has already been returned from that move.
3. The user presses "Return". The wizard refuses with `Please specify at least one non-zero quantity.` when every line is still zero, and with `You may only return Done pickings.` when the transfer is not completed.
4. A new transfer is created. Its operation type is the return operation type of the receipt's operation type when one is configured, and the receipt's own operation type otherwise. Its source location is the receipt's destination location. Its destination location is the default destination location of that return operation type when the return operation type receives goods, and the receipt's source location otherwise, which for a receipt from a vendor is the vendor location.
5. One move is created per non-zero line. **This domain adds:** when the source location of the return has usage `supplier`, that is when the goods go straight back to the vendor, the created move is stamped with the purchase order line and the counterparty found by the chain walk of `calculations.md`, section 31.
6. **This domain adds:** after the return transfer has been built, when every move of it resolves to exactly one counterparty and that counterparty differs from the transfer's own, the transfer's counterparty is rewritten to it. A return to the vendor therefore shows the vendor, not the warehouse address, even when the receipt itself carried no counterparty.
7. The return transfer is confirmed and reservation is attempted.

**Postconditions.** A new transfer exists in state `assigned` or `confirmed`. Its moves point back at the returned moves. The received quantity of the purchase order line has not moved yet, because nothing has been completed.

### 25.2 Completing the return

1. The user validates the return transfer.
2. The received quantity of every purchase order line whose moves changed is recomputed by the classification of `entities.md`, section 28.3.
3. When the return goes to an internal location first and a push rule then sends the goods on to a vendor location, the pushed move is stamped with the purchase order line and the vendor by the same chain walk (section 7 of this file), so the received quantity falls only when the goods actually leave for the vendor.

**Worked example (three-step reception).** A warehouse receives in three steps and an extra push rule sends goods from an internal "Vendor returns processing" location to the vendor location. A purchase order line orders 10 units; the receipt and the two internal transfers are validated, so the received quantity is 10. The user returns 2 units from the storage transfer, changes the return's destination to "Vendor returns processing" and validates it: the received quantity is still 10, because the goods have not left the company. The push rule creates a transfer from "Vendor returns processing" to the vendor location; the chain walk stamps it with the purchase order line and with the vendor. Validating it takes the received quantity to 8, and the transfer shows the vendor as counterparty.

### 25.3 Returning without updating the purchase order

When the user unticks "Update Quantities on Purchase Order" on a return line, the created move is excluded from both sides of the classification: it neither subtracts from nor adds to the received quantity. Two consequences follow.

- Returning 2 units out of 10 with the flag off leaves the received quantity at 10. The goods leave the company but the order still counts them as received, which is what a buyer wants when the vendor replaces the goods free of charge.
- Receiving those 2 units again, with the flag off on that second return, leaves the received quantity at 10 as well, because the new incoming move has an originating returned move and its flag is false. Ticking the flag on the second return instead would count it as incoming and take the received quantity from 8 back to 10.

### 25.4 Changing the operation type of the return

The operation type of a return transfer may be changed before it is validated. The classification of `entities.md`, section 28.3 reads the destination location and the originating returned move, never the operation type, so changing the operation type changes the received quantity only when it changes the destination location.

**Worked example.** One unit is bought, received and returned to the vendor, so the received quantity is 0. Changing the return's operation type to a delivery whose default destination location is the customer location makes the return's destination a customer location, which is not a vendor location. The move stops being a purchase return, it is not counted as outgoing, and the received quantity goes back to 1.

### 25.5 Exchanges

"Exchange" creates the return and then, for a receipt, immediately creates a second transfer that is the return of that return, so that the goods come back in. The exchange moves are detached from their originating moves, which is what stops them from being counted as returns. A purchase order that has been received once, returned once and exchanged once therefore shows three transfers: the receipt, the return and the exchange.

An exchange may also be created from a return that has no originating transfer at all, that is from a transfer created by hand rather than by the wizard. The wizard then builds the new transfer from the return's own operation type, source and destination, and no purchase order line is attached, so no received quantity moves.

---

## 26. State tables

### 25.1 Reordering Rule

There is no explicit state field. The observable state is the combination of `active`, `trigger`, `snoozed_until` and `quantity_to_order`.

| From | Trigger or operation | Guard | To | Side effects |
|---|---|---|---|---|
| Not existing | Create | Minimum not above maximum; product is not a kit; no rule exists for (product, location, company); when `snoozed_until` is set, `trigger` must be `manual` | Active | A sequence number is consumed for `name`. |
| Not existing | The replenishment report finds a negative forecast | No rule (active or archived) for that (product, location) | Active, manual, created by the superuser | `name` is `Replenishment Report`, minimum and maximum are 0. |
| Active, manual | Snooze | `trigger` is `manual` | Active, snoozed | The rule is hidden from the report until the snooze date. |
| Active, snoozed | The snooze date passes | | Active, manual | The rule reappears on the report. |
| Active, quantity to order greater than zero | Order | A rule chain exists; for a buy chain, a vendor price exists | Active, quantity to order recomputed | Documents are created; the manual override is cleared. |
| Active, quantity to order zero | Order to Max | | Active | The quantity is forced to maximum minus forecast, rounded to the multiple, then documents are created. |
| Active, manual, created by the superuser, quantity to order not above zero | The vacuum task, or opening the replenishment report | | Deleted | |
| Active | Archive, or archive the product | | Archived | The rule is no longer evaluated. |
| Archived | Unarchive, or unarchive the product | | Active | |
| Any | Write `company` with a different value | | Refused | `Changing the company of this record is forbidden at this point, you should rather archive it and create a new one.` |

### 25.2 Route

| From | Trigger | Guard | To | Side effects |
|---|---|---|---|---|
| Not existing | Create | Every rule's company matches the route company | Active | |
| Active | Archive | | Archived | Every rule of the route whose destination location is active is archived. |
| Archived | Unarchive | | Active | Those rules are unarchived. |
| Any | Delete | | Deleted | Every rule of the route is deleted (cascade). |

### 25.3 Stock Rule

| From | Trigger | Guard | To | Side effects |
|---|---|---|---|---|
| Not existing | Create | Rule company equals route company when the route has one | Active | |
| Active | Archive | | Archived | The rule is no longer selectable. |
| Active | The route is archived | The rule's destination location is active | Archived | |
| Archived | The warehouse is reconfigured and the same routing is needed again | Same operation type, source, destination, route and action | Active | The rule is reused instead of a duplicate being created. |
| Any | The route is deleted | | Deleted | |

### 25.4 The procurement request

A procurement request is not persistent, but it has a well-defined life:

| From | Trigger | Guard | To | Side effects |
|---|---|---|---|---|
| Built | Run, kit explosion | The product has a kit bill of materials | Replaced by one request per component | |
| Built | Run, skip test | The product is not a goods product, or the quantity is zero | Discarded silently | |
| Built | Run, rule selection | A rule is found | Assigned to an action | |
| Built | Run, rule selection | No rule is found | Failed | `No rule has been found to replenish "<product>" in "<location>".` plus `Verify the routes configuration on the product.` |
| Assigned to `pull` | The pull action | The rule has a source location | Realized as a stock move | The move is created and confirmed. |
| Assigned to `pull` | The pull action | The rule has no source location | Failed | `No source location defined on stock rule: <rule name>!` |
| Assigned to `buy` | The buy action | A vendor price is found | Realized as a purchase order line | A draft order is created or extended. |
| Assigned to `buy` | The buy action | No vendor price, request came from a reordering rule | Failed | `There is no matching vendor price to generate the purchase order for product <product> (no vendor defined, minimum quantity not reached, dates not valid, ...). Go on the product form and complete the list of vendors.` |
| Assigned to `buy` | The buy action | No vendor price, request came from elsewhere | Abandoned | Downstream moves are cancelled when they propagate cancellation, otherwise switched to take from stock; the responsible person is notified. |
| Assigned to `manufacture` | The manufacture action | Quantity greater than zero | Realized as one or more manufacturing orders | |
| Assigned to `manufacture` | The manufacture action | Quantity not greater than zero | Discarded silently | |
