# Acceptance criteria

Given / When / Then scenarios that a replacement must pass. Each scenario is independently verifiable and uses concrete values. Every one of the numbered rules of `business-rules.md` is cited by at least one scenario below, and a scenario that cites a rule number in its title verifies that rule. Every formula of `calculations.md`, every workflow of `workflows.md` and every state transition of the entities of `entities.md` is exercised by at least one scenario as well, either directly, when the scenario title names the section, or through the numbered rule that states it. Scenario numbers are allocated in blocks, one block per section, so the sequence has deliberate gaps between sections; a gap means room left for a future scenario in that section, never a removed scenario.

Unless a scenario says otherwise, the starting point is: one company whose currency is the euro (`EUR`), one warehouse named `Main` with the short name `WH` that receives in one step and delivers in one step, the replenishment horizon set to 0 days, the days to purchase set to 0, the sales safety days set to 0, and one storable product `P` whose unit of measure is `Units` with a rounding of 0.01 and whose "Product Unit" precision is two decimal places.

---

## 1. Routes

**AC-001 A route may be shared by every company (`RP-RULE-001`).**
Given a route with no company.
When a user of any company opens the routes list.
Then the route is visible to that user.

**AC-002 A rule of another company than its route is refused (`RP-RULE-002`).**
Given a route `R` whose company is `Company A`, and a rule `S` of that route.
When a user writes `Company B` on `S`.
Then the write is refused with `Rule S belongs to Company B while the route belongs to Company A.`

**AC-003 Changing a route's company filters its warehouses (`RP-RULE-003`).**
Given a route whose company is `Company A` and whose warehouses are `WH-A` (of `Company A`) and `WH-B` (of `Company B`).
When the user changes the company to `Company B` in the form.
Then the warehouse list holds only `WH-B`.

**AC-004 Turning off warehouse selectability empties the warehouse list (`RP-RULE-005`).**
Given a route with `warehouse_selectable` true and two warehouses selected.
When the user unticks `warehouse_selectable`.
Then the warehouse list is empty.

**AC-005 Routes are tried by ascending sequence (`RP-RULE-007`, `RP-RULE-063`).**
Given product `P` carrying route `Route 1` of sequence 10 with a rule of sequence 20, and route `Route 2` of sequence 5 with a rule of sequence 20; both rules pull from `WH/Stock` to `WH/Stock` with the delivery operation type.
When rule selection runs for `P` at `WH/Stock` with both routes passed as the request routes.
Then the rule of `Route 2` is returned.

**AC-006 Archiving a route archives its rules (`RP-RULE-011`).**
Given an active route with three active rules whose destination locations are all active.
When the route is archived.
Then the three rules are archived; and when the route is unarchived, the three rules are active again.

**AC-007 Deleting a route deletes its rules (`RP-RULE-010`).**
Given a route with two rules.
When the route is deleted.
Then both rules no longer exist.

**AC-008 Duplicating a route clears its assignments (`RP-RULE-009`).**
Given a route named `Two-steps reception` selected on two products, one category and one warehouse, holding one rule named `Input to Stock`.
When the route is duplicated.
Then the copy is named `Two-steps reception (copy)`, holds one rule named `Input to Stock (copy)`, and has no product, no category, no warehouse, no supplied warehouse and no supplying warehouse.

**AC-009 A buy route is a valid resupply route only with a vendor (`RP-RULE-013`).**
Given a route holding one rule with action `buy`, and product `P` with no Vendor Price.
When the resupply eligibility of the route for `P` is evaluated.
Then the answer is false; and after a Vendor Price is added to `P`, the answer is true.

**AC-010 A manufacture route is a valid resupply route only with a bill (`RP-RULE-013`).**
Given a route holding one rule with action `manufacture` and no `buy` rule, and product `P` with no bill of materials.
When the resupply eligibility of the route for `P` is evaluated.
Then the answer is false; and after a bill of materials of type `normal` is created for `P`, the answer is true.

---

## 2. Stock Rules

**AC-020 Choosing the buy action clears the source location (`RP-RULE-025`).**
Given a rule form with action `pull` and source location `WH/Stock`.
When the user sets the action to `buy`.
Then the source location is empty.

**AC-021 Choosing an operation type fills both locations (`RP-RULE-026`).**
Given a rule form and an operation type whose default source location is `Partners/Vendors` and whose default destination location is `WH/Input`.
When the user selects that operation type.
Then the source location is `Partners/Vendors` and the destination location is `WH/Input`.

**AC-022 Only allowed operation type codes are offered for a buy rule (`RP-RULE-024`).**
Given the Drop Shipping capability package is not installed.
When a rule's action is set to `buy`.
Then only operation types whose code is `incoming` may be selected; and after the Drop Shipping capability package is installed, operation types whose code is `dropship` are offered as well.

**AC-023 A pull rule without a source location fails at run time (`RP-RULE-028`).**
Given a pull rule named `Stock to Output` with no source location, selected for a need.
When the request is run.
Then the run fails with `No source location defined on stock rule: Stock to Output!` and no stock move is created.

**AC-024 The destination of the created move depends on the rule flag (`RP-RULE-032`).**
Given a pull rule whose destination location is `WH/Stock`, whose operation type's default destination location is `WH/Input`, and whose `location_destination_from_rule` is false; and a need for 12 units at `WH/Stock`.
When the request is run.
Then the created move has destination location `WH/Input` and final location `WH/Stock`; and when `location_destination_from_rule` is true instead, the created move has destination location `WH/Stock`.

**AC-025 The last rule of a cancel-propagating chain does not propagate (`RP-RULE-034`).**
Given a warehouse configured to receive in three steps with the Purchase Inventory capability package absent.
When the reception route is generated.
Then the rules `Vendors to Stock` and `Input to Quality Control` have `propagate_cancel` true and the rule `Quality Control to Stock` has `propagate_cancel` false.

**AC-026 An archived rule is reused instead of duplicated (`RP-RULE-035`).**
Given a warehouse switched from two-step to one-step reception and back to two-step reception.
When the reception route is rebuilt the second time.
Then the route holds exactly one rule `Input to Stock`, which was unarchived rather than duplicated.

**AC-027 A circular rule chain is refused when walked (`RP-RULE-036`, `RP-RULE-160`).**
Given the reception route of `Main` whose rules are all archived, plus a new rule `Looping Rule` in that route with action `pull_push`, supply method `make_to_order`, source `WH/Stock` and destination `WH/Stock`; and an outgoing demand of 10 units of `P` from `WH/Stock`.
When the replenishment screen is opened.
Then the operation fails with `Invalid rule's configuration, the following rule causes an endless loop: Looping Rule`.

---

## 3. Reordering Rule validations and lifecycle

**AC-040 A second rule for the same product and location is refused (`RP-RULE-040`).**
Given a reordering rule for `P` at `WH/Stock` in the company.
When a second rule is created for the same product, location and company.
Then the creation is refused with `A replenishment rule already exists for this product on this location.`

**AC-041 An archived rule still blocks the pair (`RP-RULE-040`).**
Given the rule of AC-040 archived.
When a new rule is created for the same product and location.
Then the creation is still refused with the same message.

**AC-042 The minimum may not exceed the maximum (`RP-RULE-041`).**
Given a warehouse `Main`.
When a reordering rule is created for `P` with a minimum of 2 and a maximum of 1.
Then the creation is refused with `The minimum quantity must be less than or equal to the maximum quantity.`

**AC-043 The maximum follows the minimum upwards (`RP-RULE-049`).**
Given a reordering rule form with a minimum of 0 and a maximum of 0.
When the user types 15 into the minimum.
Then the maximum becomes 15; and when the user then types 30 into the maximum and lowers the minimum to 10, the maximum stays 30.

**AC-044 A kit product may not have a reordering rule (`RP-RULE-042`).**
Given product `K` with a bill of materials of type "kit".
When a reordering rule is created for `K`.
Then the creation is refused with `A product with a kit-type bill of materials can not have a reordering rule.`

**AC-045 A snoozed automatic rule may not be created (`RP-RULE-043`).**
When a reordering rule is created with a snooze date and no explicit trigger.
Then the creation is refused with `You can not create a snoozed orderpoint that is not manually triggered.`

**AC-046 An automatic rule may not be snoozed (`RP-RULE-044`).**
Given an existing reordering rule whose trigger is `auto`.
When the user snoozes it from the replenishment screen.
Then the operation is refused with `You can only snooze manual orderpoints. You should rather archive 'auto-trigger' orderpoints if you do not want them to be triggered.`

**AC-047 The company of a rule is immutable (`RP-RULE-045`).**
Given a reordering rule of `Company A`.
When a user writes `Company B` on it.
Then the write is refused with `Changing the company of this record is forbidden at this point, you should rather archive it and create a new one.`

**AC-048 A rule cannot be created without a warehouse (`RP-RULE-046`).**
Given a company with no warehouse.
When a reordering rule is created for a product of that company.
Then an inventory administrator is shown `Please create a warehouse for company <company name>.` with a button `Go to Warehouses`, and a user who is not an inventory administrator is shown `Please contact your administrator to configure your warehouse.`

**AC-049 Choosing a warehouse fills the location, and the reverse (`entities.md` section 3.4).**
Given a reordering rule form.
When the user selects the warehouse `Main`.
Then the location becomes `WH/Stock`; and when the user selects the location `WH/Shelf 1` instead, the warehouse becomes the warehouse of `WH/Shelf 1`.

**AC-050 Setting a vendor price on a rule with no route sets a buy route (`RP-RULE-052`).**
Given a reordering rule with no route and a Vendor Price for `P`.
When the user sets that Vendor Price on the rule.
Then the rule's route becomes the route of the first `buy` rule, and clearing the route afterwards clears the vendor price again (`RP-RULE-051`).

**AC-051 A manual override is discarded on an automatic rule (`RP-RULE-056`).**
Given an automatic reordering rule whose computed quantity to order is 5.
When a user writes 12 into the quantity to order.
Then the manual override is 0 and the quantity to order is 5 again.

**AC-052 A manual override is kept on a manual rule (`RP-RULE-057`).**
Given a manual reordering rule whose computed quantity to order is 5.
When a user writes 12 into the quantity to order.
Then the manual override is 12 and the quantity to order is 12; and pressing the undo button clears the override and the quantity to order is 5 again.

**AC-053 Archiving a product archives its reordering rules (`RP-RULE-058`).**
Given product `P` with one active reordering rule.
When `P` is archived.
Then the rule is archived; and when `P` is unarchived, the rule is active again.

**AC-054 The reference is taken from the sequence (`RP-RULE-060`).**
Given no reordering rule has ever been created.
When a user creates one.
Then its reference is `OP/00001`, and the next one created is `OP/00002`.

**AC-055 The quantity to order is recomputed when a demand appears (`entities.md` section 3.6).**
Given an automatic reordering rule for `P` at `WH/Stock` with minimum 5, maximum 5 and a quantity to order of 5.
When an outgoing move of 1 unit from `WH/Stock` to the customer location is confirmed.
Then the quantity to order becomes 6.

**AC-056 The quantity to order follows a deleted receipt (`entities.md` section 3.6).**
Given a manual reordering rule for `P` at `WH/Stock` with minimum 10 and maximum 10, and a confirmed incoming move of 10 units from the vendor location to `WH/Stock`.
Then the forecast is 10 and the quantity to order is 0.
When the incoming move is deleted.
Then the forecast is 0 and the quantity to order is 10.

---

## 4. Rule selection

**AC-060 No rule found fails the request (`RP-RULE-072`).**
Given product `P` with no route at all, and a warehouse with no route selected on it.
When a need for 5 units of `P` at a location outside any warehouse is run.
Then the run fails with `No rule has been found to replenish "P" in "<location display name>".` followed by a new line and `Verify the routes configuration on the product.`, and no document is created.

**AC-061 A warehouse buy route is ignored without a vendor (`RP-RULE-068`).**
Given a warehouse whose routes include the global Buy route, and product `P` with no Vendor Price and no route of its own.
When rule selection runs for `P` at `WH/Stock`.
Then the Buy route is not considered.

**AC-062 The request routes win over the product routes (`RP-RULE-062`).**
Given product `P` carrying route `Product route` with a rule for `WH/Stock`, and a need that carries route `Request route`, which also has a rule for `WH/Stock`.
When rule selection runs.
Then the rule of `Request route` is returned.

**AC-063 The product's own routes sort before the category's routes (`RP-RULE-063`).**
Given product `P` carrying route `A` of sequence 30 and its category carrying route `B` of sequence 10, both with a rule for `WH/Stock`.
When rule selection runs for `P` at `WH/Stock` with no request route.
Then the rule of route `A` is returned, because the product's own routes sort first whatever their sequence.

**AC-064 Selection walks up the location hierarchy (`RP-RULE-061`).**
Given a rule whose destination location is `WH/Stock` and a sub-location `WH/Stock/Shelf 1` with no rule of its own.
When rule selection runs for a need at `WH/Stock/Shelf 1`.
Then the rule whose destination is `WH/Stock` is returned.

**AC-065 A warehouse-specific rule wins over a warehouse-less rule (`RP-RULE-065`).**
Given one route holding two rules for `WH/Stock`: rule `X` with warehouse `Main` and rule `Y` with no warehouse.
When rule selection runs with warehouse `Main`.
Then rule `X` is returned; and when it runs with warehouse `Second`, rule `Y` is returned.

---

## 5. The pull action and chaining

**AC-070 A make-to-order move creates the supply need (`RP-RULE-140`).**
Given product `P` carrying a route with a rule `Stock to Output` that pulls from `WH/Stock` to `WH/Output` with the internal operation type and the flag "destination location from rule"; and a delivery of 10 units from `WH/Output` to the customer location whose supply method is make to order.
When the delivery is confirmed and the scheduler runs.
Then exactly one move from `WH/Stock` to `WH/Output` exists, and the delivery move is listed among its downstream moves.

**AC-071 The pull rule lead time shifts the scheduled date (`RP-RULE-220`).**
Given a pull rule with a lead time of 2 days, and a need whose planned date is day 20 and whose deadline is day 20.
When the request is run.
Then the created move's scheduled date is day 18 and its deadline is day 18.

**AC-072 Negative requests are processed first (`RP-RULE-080`).**
Given two requests for the same product, rule and location: one of −4 units and one of +10 units.
When they are run in one batch.
Then the negative request is handled before the positive one and the created moves can merge.

**AC-073 A negative request marks the move as a refund (`RP-RULE-085`).**
Given a request for −3 units.
When it is run through a pull rule.
Then the created move's "update quantities on the order" flag is true.

**AC-074 A move is created even when the requester has no inventory rights (`RP-RULE-086`).**
Given a salesperson with no inventory rights who confirms a sales order for a make-to-order product.
When the order is confirmed.
Then the delivery move and the origin move are created and no access-rights error is raised.

**AC-075 The take-from-stock-otherwise split orders only the missing part (`RP-RULE-142`).**
Given a rule whose supply method is `mts_else_mto` (make to stock, else make to order), and 12 free units of `P` at its source location, and two moves of 10 units each confirmed in the same batch.
When the batch is confirmed.
Then one supply need for 8 units is created, and neither move is linked to the resulting document.

**AC-076 A two-step reception chain pushes the received quantity forward (`workflows.md` section 9).**
Given a warehouse that receives in two steps, and a purchase order for 12 units of `P` confirmed.
Then a receipt move from the vendor location to `WH/Input` exists with final location `WH/Stock`.
When the receipt is validated.
Then a second move `WH/Input` to `WH/Stock` for 12 units exists, with supply method make to order, and it is listed among the receipt's downstream moves.
When the purchase order line is raised from 12 to 15 after validation.
Then a second receipt move for 3 units is created, and validating it raises the downstream internal transfer to 15 units in total.

**AC-077 A transparent push rule rewrites the move instead of adding a step (`RP-RULE-133`).**
Given a push rule whose automatic mode is "Automatic No Step Added", whose source is `WH/Input`, whose destination is `WH/Stock` and whose lead time is 1 day; and a receipt move landing in `WH/Input` scheduled on day 5.
When push application runs on that move.
Then no new move is created, the move's destination location becomes `WH/Stock` and its scheduled date becomes day 6.

**AC-078 A push rule is not applied twice (`RP-RULE-130`).**
Given a move into `WH/Input` that already has a downstream move whose source location is `WH/Input`.
When push application runs.
Then no second move is created.

**AC-079 A return is not pushed back where it came from (`RP-RULE-132`).**
Given a completed receipt into `WH/Input` and a return move created from it whose destination is the vendor location, and a push rule from the return's destination back to `WH/Input`.
When push application runs on the return.
Then no move is created.

---

## 6. The buy action

**AC-090 A need for a bought product creates a draft purchase order (`workflows.md` section 5).**
Given product `P` carrying the Buy route and one Vendor Price for vendor `Smith` at 100.00 per unit with a minimum quantity of 1; and a reordering rule at `WH/Stock` with minimum 0 and maximum 0; and a confirmed outgoing demand of 10 units.
When the scheduler runs.
Then one draft purchase order exists for `Smith` with one line of 10 units of `P` at 100.00, whose source document is the reordering rule's reference.

**AC-091 Two needs for the same vendor and reference share one order (`RP-RULE-097`, `RP-RULE-098`).**
Given vendor `Smith` whose grouping mode is `On Order`, and two needs for two different products both carrying the same stock reference.
When both are run.
Then one purchase order holds two lines.

**AC-092 A need with no reference never joins an order that has one (`RP-RULE-098`).**
Given vendor `Smith` whose grouping mode is `On Order`, one draft purchase order created from a need carrying reference `S00001`, and a new need with no reference.
When the new need is run.
Then a second draft purchase order is created.

**AC-093 The Always grouping mode merges everything (`RP-RULE-099`).**
Given vendor `Smith` whose grouping mode is `Always`, one need carrying reference `S00001` and one need with no reference.
When both are run.
Then one purchase order holds both lines.

**AC-094 The Daily grouping mode groups by expected arrival day (`RP-RULE-100`).**
Given vendor `Smith` whose grouping mode is `Daily`, and two needs whose planned dates fall on the same calendar day but carry different references.
When both are run.
Then one purchase order holds two lines whose planned dates are that day; and the order's deadline is that day minus the greatest vendor lead time among the two products.

**AC-095 The Weekly grouping mode on a named weekday moves the dates (`RP-RULE-102`, `RP-RULE-118`).**
Given today is Wednesday 10 September at 10:00; vendor `Smith` with grouping mode `Weekly` and week day `Tuesday`; product `P1` with a vendor lead time of 0 and product `P2` with a vendor lead time of 2; a need for `P1` planned on Friday 12 September and a need for `P2` planned on Saturday 13 September, each with its own reference.
When both are run and the order is confirmed.
Then one purchase order exists whose expected arrival is Tuesday 16 September at 10:00 and whose order deadline is Sunday 14 September at 10:00, and both of its lines are planned on Tuesday 16 September at 10:00.

**AC-096 Two needs for the same product merge into one line (`RP-RULE-106`).**
Given product `P` carrying the Buy route with a vendor whose lead time is 5, and a make-to-order move of 10 units confirmed, which creates one purchase order line of 10.
When a second make-to-order move of 5 units is confirmed.
Then still exactly one purchase order line exists and its quantity is 15.

**AC-097 Different custom descriptions do not merge (`RP-RULE-106`).**
Given product `T` carrying the Buy route, a vendor with a product name and product code, and three needs: 5 units with the description `Color (Red)`, 10 units with `Color (Red)` and 10 units with `Color (Green)`.
When the three are run.
Then the purchase order holds two lines: one of 15 units named `<product display name>` then a new line then `Color (Red)`, and one of 10 units named `<product display name>` then a new line then `Color (Green)`.

**AC-098 Two presses of Order on a temporary rule reuse the same line (`RP-RULE-109`).**
Given a confirmed outgoing demand of 5 units of `P`, which creates a temporary reordering rule; the user presses Order, creating one purchase order line of 5.
When a further demand of 3 units is confirmed, the replenishment screen is reopened and the user presses Order again.
Then still exactly one purchase order line exists and its quantity is 8.

**AC-099 The cheapest bracket that the quantity reaches is chosen (`RP-RULE-091`).**
Given product `P` with two Vendor Prices for the same vendor: sequence 1 with a minimum quantity of 1 at 140.00, and sequence 2 with a minimum quantity of 10 at 100.00; and a replenishment of 10 units.
When the replenishment is launched.
Then the created purchase order line's unit price is 100.00.

**AC-100 The discounted price decides (`RP-RULE-091`).**
Given product `P` with two Vendor Prices for the same vendor: 100.00 with a 10 percent discount, and 110.00 with a 20 percent discount; and a replenishment of 1 unit.
When the replenishment is launched.
Then the Vendor Price of 110.00 with the 20 percent discount is chosen, because its discounted price of 88.00 is lower than 90.00.

**AC-101 Merging into a line re-selects the price bracket (`RP-RULE-111`).**
Given the two Vendor Prices of AC-099 and a first need for 5 units, creating a line of 5 units at 140.00.
When a second need for 10 units merges into that line.
Then the line's quantity is 15 and its unit price is 100.00.

**AC-102 The vendor unit is used unless the caller forces the request unit (`RP-RULE-113`, `RP-RULE-114`).**
Given product `P` stocked in `Units` with a Vendor Price expressed in `Dozens`, and a need for 12 units with no forced unit.
When the need is run.
Then the created line carries 1 `Dozen`; and when the need is run with the forced unit flag, the line carries 12 `Units`.

**AC-103 A missing vendor blocks a reordering rule (`RP-RULE-094`).**
Given product `P` carrying the Buy route with no Vendor Price, and an automatic reordering rule with a positive quantity to order.
When the scheduler runs.
Then no purchase order is created, and a warning activity is scheduled on the product template whose note is `There is no matching vendor price to generate the purchase order for product P (no vendor defined, minimum quantity not reached, dates not valid, ...). Go on the product form and complete the list of vendors.`, assigned to the product's responsible person.

**AC-104 The same failure does not create a second activity (`RP-RULE-169`).**
Given the activity of AC-103 already exists.
When the scheduler runs again.
Then no second activity is created.

**AC-105 A missing vendor does not block a make-to-order chain (`RP-RULE-095`, `RP-RULE-096`).**
Given a sales order line for a make-to-order product carrying the Buy route and having no Vendor Price.
When the order is confirmed.
Then no purchase order is created; the delivery move's supply method becomes take from stock; the delivery move is cancelled instead when its rule propagates cancellation; and a message is posted on the sales order mentioning the salesperson, reading `No supplier has been found to replenish` then the product display name in bold then `this product should be manually replenished.`

**AC-106 A new line is never created with a non-positive quantity (`RP-RULE-110`).**
Given a merged request whose net quantity is −4 units and no existing line for that product on the order.
When the buy action runs.
Then no line is created and the order is left unchanged.

**AC-107 A group made only of negative requests creates no order (`RP-RULE-103`).**
Given no draft purchase order for vendor `Smith` and a single request of −4 units.
When the buy action runs.
Then no purchase order is created.

**AC-108 The order deadline is moved earlier, never later (`RP-RULE-117`).**
Given a draft purchase order whose order deadline is 10 March, and a new need for the same vendor whose planned date minus the vendor lead time is 7 March.
When the need is run.
Then the order deadline becomes 7 March; and a further need whose computed date is 12 March leaves the order deadline at 7 March.

---

## 7. The manufacture action

**AC-120 A manufacture rule creates a manufacturing order (`workflows.md` section 6).**
Given product `M` carrying a Manufacture route and a bill of materials of type `normal` with a manufacturing lead time of 0 days, and a need for 5 units at `WH/Stock` planned on day 10.
When the need is run.
Then one manufacturing order for 5 units exists whose start date is day 10 minus one hour and whose deadline is day 10.

**AC-121 A manufacturing lead time moves the start date (`RP-RULE-126`).**
Given the same setup with a manufacturing lead time of 3 days.
When the need is run.
Then the manufacturing order's start date is day 7 and its deadline is day 10.

**AC-122 A zero or negative manufacture request is skipped (`RP-RULE-120`).**
Given a need for 0 units of `M`.
When the need is run.
Then no manufacturing order is created and no error is raised.

**AC-123 A draft manufacturing order from a reordering rule is confirmed after the batch (`RP-RULE-175`).**
Given two automatic reordering rules, one for a finished product `M` and one for its component `C`, both with a positive quantity to order.
When the scheduler runs.
Then the manufacturing order for `M` is confirmed only after both reordering rules have been processed, so that the component need it generates does not interfere with the rule for `C`.

---

## 8. Quantity to order

**AC-140 The required worked example (`calculations.md` section 7).**
Given a reordering rule for `P` at `WH/Stock` with a minimum of 10, a maximum of 50 and a replenishment multiple worth 12 product units, and a forecast of 4 units at the rule's forecast date.
When the quantity to order is computed.
Then it is 48, and the rule reports that the replenishment is unwanted because 4 plus 48 is above the maximum of 50.

**AC-141 No multiple gives the exact difference (`calculations.md` section 7).**
Given 14.5 units on hand, a minimum of 15, a maximum of 30 and no replenishment multiple.
When the quantity to order is computed.
Then it is 15.5.

**AC-142 A pack of ten rounds up (`calculations.md` section 8).**
Given the same rule with a replenishment multiple worth 10 product units.
When the quantity to order is computed.
Then it is 20.

**AC-143 The product unit as a multiple rounds to a whole unit (`calculations.md` section 8).**
Given the same rule with the product's own unit as the replenishment multiple.
When the quantity to order is computed.
Then it is 16.

**AC-144 A fractional multiple is honoured (`calculations.md` section 7).**
Given a reordering rule with a minimum of 4, a maximum of 5.1, a forecast of 0 and a replenishment multiple worth 0.1 product unit.
When the quantity to order is computed.
Then it is exactly 5.1.

**AC-145 A forecast equal to the minimum orders nothing (`RP-RULE-162`).**
Given a reordering rule with a minimum of 10 and a forecast of exactly 10.
When the quantity to order is computed.
Then it is 0.

**AC-146 The quantity in progress is deducted (`calculations.md` section 5).**
Given a reordering rule for `P` at `WH/Stock` with a minimum of 0 and a maximum of 0, and a draft purchase order line for 10 units of `P` attached to that rule.
When the forecast is computed.
Then it is 10 higher than the pure stock forecast, and the quantity to order is 0.

**AC-147 Cancelling the purchase order restores the shortage (`calculations.md` section 5).**
Given the confirmed purchase order of AC-146.
Then the rule's forecast is 10.
When the purchase order is cancelled.
Then the rule's forecast is 0 again.

**AC-148 Setting a supplier raises the quantity to the vendor minimum (`RP-RULE-204`).**
Given a reordering rule with a quantity to order of 3 and a Vendor Price whose minimum quantity is 10 units.
When the user presses "Set as Supplier" on that Vendor Price.
Then the rule's quantity to order becomes 10 and its route becomes a Buy route.

---

## 9. Lead times, the horizon and dates

**AC-160 The horizon widens the forecast window without moving the documents (`calculations.md` sections 3 and 13.6).**
Given the replenishment horizon set to 365, the days to purchase set to 0, a vendor lead time of 7 days, a reordering rule with a minimum of 10 and a maximum of 50 whose quantity to order is positive, and today being 1 March.
When the scheduler runs.
Then the created purchase order's order deadline is 1 March at 12:00 and its expected arrival is 8 March at 12:00.

**AC-161 The horizon can hide a future demand (`calculations.md` section 6).**
Given the replenishment horizon set to 4 days, a vendor lead time of 1 day, today being 14 January, a reordering rule with a minimum of 0 and a maximum of 0, and a confirmed outgoing move of 1 unit scheduled on 20 January.
Then the quantity to order is 0, because the forecast date is 19 January.
When a second outgoing move of 1 unit is confirmed for today, making the current forecast −1.
Then the quantity to order is 1.

**AC-162 The days to purchase widens the window but not the order deadline (`calculations.md` section 2).**
Given the replenishment horizon set to 0, the days to purchase set to 2, a vendor lead time of 1 day and today being 21 April.
When a reordering rule is evaluated.
Then its forecast date is 24 April; and the request for quotation it creates shows an order deadline of 23 April and an expected arrival of 24 April.

**AC-163 A missing vendor adds a year to the lead time (`calculations.md` section 2).**
Given product `P` carrying the Buy route with no Vendor Price, and a reordering rule for it.
When the lead days are computed.
Then they are 365.

**AC-164 The scheduler does not move an existing order deadline (`RP-RULE-117`).**
Given the days to purchase set to 2, a vendor lead time of 1 day, a reordering rule that the scheduler processed today, creating a purchase order line of 25 units with an order deadline of today plus 2.
When two days pass and the scheduler runs again with 5 more units of demand.
Then the same purchase order line is updated to 30 units and the order deadline is still today plus 2 counted from the first run.

**AC-165 A deadline change propagates along the chain (`RP-RULE-223`).**
Given a delivery move and its origin internal move, both with a deadline of day 30.
When the delivery move's deadline is written as day 24.
Then the origin move's deadline becomes day 24; and completing the origin move afterwards leaves both deadlines at day 24 while setting the origin move's scheduled date to the moment of completion.

**AC-166 A late origin move raises a delay alert (`RP-RULE-224`).**
Given a delivery scheduled on day 10 and an origin receipt scheduled on day 12 that is not completed.
Then the delivery's delay alert date is day 12.
When the receipt is completed.
Then the delivery's delay alert date is empty.

**AC-167 A rule lead time positions the receipt date (`calculations.md` section 12).**
Given the reception rule of `Main` with a lead time of 9 days, a reordering rule for `P` with a minimum of 0 and a maximum of 5, and a confirmed outgoing move of 12 units scheduled five days from today.
When the scheduler runs.
Then one receipt move from the vendor location exists whose scheduled date is today and whose quantity is 17.

**AC-168 A vendor lead time moves the replenish wizard's scheduled date (`entities.md` section 25).**
Given today is 1 January 2023, product `P` carrying the Buy route, and two Vendor Prices with lead times of 0 and 3 days.
When the Product Replenish wizard is opened and the zero-lead-time vendor is chosen.
Then the scheduled date is 1 January 2023.
When the three-day vendor is chosen.
Then the scheduled date is 4 January 2023.

**AC-169 The days to purchase moves the replenish wizard's scheduled date (`entities.md` section 25).**
Given today is 1 January 2023, the days to purchase set to 0 and a vendor with a lead time of 0.
Then the wizard's scheduled date is 1 January 2023.
When the days to purchase is set to 5 and the vendor is chosen again.
Then the wizard's scheduled date is 6 January 2023.

---

## 10. The deadline date

**AC-180 The deadline is today when the stock is already short (`calculations.md` section 11).**
Given a reordering rule with a minimum of 10 and 5 units on hand.
Then the deadline date is today.

**AC-181 The deadline is the day of the first dip (`calculations.md` section 11).**
Given today is 2 September, a replenishment horizon of 365, a lead time of 0, 20 units on hand, a minimum of 10, a confirmed outgoing move of 15 units on 17 September and a confirmed incoming move of 10 units on 27 September.
Then the deadline date is 17 September.

**AC-182 An incoming move can push the dip later (`calculations.md` section 11).**
Given the same setup with a confirmed outgoing move of 15 on 17 September, a confirmed incoming move of 15 on 17 September and a confirmed outgoing move of 15 on 27 September.
Then the deadline date is 27 September.

**AC-183 No dip inside the horizon means no deadline (`calculations.md` section 11).**
Given the same setup with a confirmed outgoing move of 10 on 27 September and one of 5 on 7 October, and a replenishment horizon of 30 days.
Then the deadline date is empty, because the dip on 7 October falls outside the horizon of 2 October.

**AC-184 Unconfirmed moves are not counted (`calculations.md` section 11).**
Given the moves of AC-181 created but not confirmed.
Then the deadline date is empty; and confirming them sets it to 17 September.

**AC-185 The lead time moves the deadline earlier (`calculations.md` section 11).**
Given the setup of AC-181 and a chain lead time of 5 days.
Then the deadline date is 12 September.

---

## 11. The scheduler

**AC-200 Only automatic rules whose product is active are run (`RP-RULE-163`).**
Given one automatic rule for an active product, one automatic rule for an archived product and one manual rule, all with a positive quantity to order.
When the scheduler runs.
Then only the first creates a document.

**AC-201 A failing rule does not stop the batch (`RP-RULE-167`).**
Given two automatic reordering rules with a positive quantity to order, one for a product with a vendor and one for a product carrying the Buy route with no vendor.
When the scheduler runs.
Then the first rule creates a purchase order, and the second produces a warning activity.

**AC-202 Waiting moves are reserved in the right order (`RP-RULE-173`).**
Given three moves in state `confirmed` with reservation dates today, today and tomorrow, priorities urgent, normal and normal.
When the scheduler runs.
Then the two moves whose reservation date is today are reserved, the urgent one first; the move whose reservation date is tomorrow is not reserved.

**AC-203 A confirmed transfer triggers its reordering rules immediately (`RP-RULE-171`).**
Given an automatic reordering rule for `P` at `WH/Stock` with a minimum of 0 and a maximum of 5, a Vendor Price, and no scheduled run pending.
When a delivery of 12 units of `P` from `WH/Stock` is confirmed and reserved.
Then a receipt move from the vendor location for 17 units scheduled today already exists, without the daily job having run.

**AC-204 An internal transfer inside the watched location creates no need (`RP-RULE-171`).**
Given an automatic reordering rule at `WH/Stock` and a transfer from `WH/Stock/Shelf 1` to `WH/Stock/Shelf 2`.
When the transfer is confirmed.
Then no need is created for that rule.

**AC-205 A move added to a confirmed transfer triggers its own rule (`RP-RULE-171`).**
Given the setup of AC-203 and a second product `Q` with its own automatic rule with a minimum of 0 and a maximum of 5.
When a move of 5 units of `Q` is added to the already confirmed delivery.
Then a receipt move for 10 units of `Q` scheduled today exists.

**AC-206 The event-driven trigger can be switched off (`RP-RULE-172`).**
Given the stored parameter `inventory.disable_automatic_scheduler` set.
When the delivery of AC-203 is confirmed.
Then no receipt move is created until the daily job runs.

---

## 12. Manual replenishment and the replenishment report

**AC-220 A negative forecast creates a temporary rule (`RP-RULE-191`, `RP-RULE-192`).**
Given `P` with no stock, and a confirmed internal transfer of 3 units from `WH/Stock` to a replenishment sub-location.
When the replenishment screen is opened.
Then exactly one reordering rule exists for `P` at `WH/Stock`, named `Replenishment Report`, with a minimum of 0, a maximum of 0, the trigger `manual`, created by the superuser, and a quantity to order of 3.

**AC-221 A satisfied temporary rule is deleted (`RP-RULE-059`, `RP-RULE-183`).**
Given the temporary rule of AC-220 and a press of Order that covers the shortage.
When the replenishment screen is reopened.
Then the temporary rule no longer exists.

**AC-222 An existing rule absorbs the shortage instead of a new rule being created (`RP-RULE-191`).**
Given an existing reordering rule for `P` at `WH/Stock` and a further negative forecast of −5 at that location.
When the replenishment screen is opened.
Then no second rule is created and the existing rule's forecast is 5 lower.

**AC-223 An archived location does not break the report (`RP-RULE-188`).**
Given a confirmed outgoing move from a sub-location that is afterwards archived.
When the replenishment screen is opened.
Then the screen opens without error.

**AC-224 Pressing Order above the minimum changes nothing (`RP-RULE-180`).**
Given 10 units of `P` on hand and a reordering rule with a minimum of 5 and a maximum of 200.
Then the forecast is 10 and the quantity to order is 0.
When the user presses Order.
Then the forecast is still 10 and no document is created.

**AC-225 Order to Max fills up to the maximum (`RP-RULE-185`).**
Given the rule of AC-224.
When the user presses Order to Max.
Then a document for 190 units is created and the forecast becomes 200.

**AC-226 Order to Max honours the multiple (`calculations.md` section 8).**
Given the rule of AC-225 now with a replenishment multiple of `Dozens` and a maximum of 240.
When the user presses Order to Max.
Then 4 dozens, that is 48 units, are ordered and the forecast becomes 248.

**AC-227 A single-rule failure offers a shortcut to the product (`RP-RULE-181`).**
Given one selected reordering rule whose product carries the Buy route with no vendor.
When the user presses Order.
Then the error is shown with a button labelled `Edit Product` that opens the product form.

**AC-228 Ordering reports the created purchase order (`RP-RULE-184`).**
Given one selected reordering rule that buys.
When the user presses Order.
Then a notification titled `The following replenishment order has been generated` is shown with a link labelled with the purchase order's display name.

**AC-229 Ordering across warehouses reports the transfer (`RP-RULE-184`).**
Given one selected reordering rule whose route is an inter-warehouse resupply route.
When the user presses Order.
Then a notification titled `The inter-warehouse transfers have been generated` is shown with a link labelled with the transfer's name.

**AC-230 Snoozing hides a manual rule (`RP-RULE-193`).**
Given a manual reordering rule with a positive quantity to order.
When the user snoozes it for one week and the "Not Snoozed" filter is applied.
Then the rule is not listed; and after the snooze date has passed it is listed again.

**AC-231 "Automate" switches the trigger and orders (`RP-RULE-186`).**
Given a manual rule with a positive quantity to order.
When the user presses Automate.
Then the rule's trigger is `auto` and the documents are created.

---

## 13. The Replenishment Information wizard

**AC-240 The lead time breakdown is read backwards from today (`interfaces.md` section 3).**
Given today is 1 March, a vendor lead time of 3 days, a days to purchase of 2 days and a replenishment horizon of 365 days.
When the dialog is opened for that rule.
Then it shows a row `Order Deadline` with the date 3 March and a row `Receipt Date` with the date 6 March, plus the captions `Vendor Lead Time + 3 day(s)`, `Days to Purchase + 2 day(s)` and `Time Horizon + 365 day(s)`.

**AC-241 The forecast date is shown (`interfaces.md` section 3).**
Given a replenishment horizon of 3 days and a reordering rule whose lead days are 0.
When the dialog is opened.
Then the forecast date shown is today plus 3 days.

**AC-242 The demand graph, thirty-day basis (`calculations.md` section 20).**
Given today is 14 August, a reordering rule with a minimum of 10 and a maximum of 50, one completed delivery of 15 units in the past 31 days and no return; the period basis is `Last 30 days` and the factor is 100.
When the dialog is opened.
Then the daily demand is 0.48, the average stock is 30.0, the ordering period is 82, the horizontal axis reads `''`, `In 82 day(s)`, `In 164 day(s)`, `In 246 day(s)`, and the saw-tooth curve is 50, 10, 50, 10, 50, 10.

**AC-243 The demand graph, seven-day basis with a factor (`calculations.md` section 20).**
Given the same data with the basis `Last 7 days`, a factor of 200, a minimum of 20 and a maximum of 40.
Then the daily demand is 4.29, the average stock is 30.0, the ordering period is 4, the horizontal axis reads `''`, `In 4 day(s)`, `In 8 day(s)`, `In 12 day(s)`, and the curve is 40, 20, 40, 20, 40, 20.

**AC-244 Confirmed moves count in the demand graph (`calculations.md` section 20).**
Given the setup of AC-243 plus a confirmed outgoing move of 15 units dated five days ago.
Then the daily demand is 8.57 and the ordering period is 2.

**AC-245 A supplying warehouse that cannot cover the need warns first (`RP-RULE-201`).**
Given a reordering rule with a quantity to order of 40 and a supplying warehouse holding 12 free units.
When the user presses Select Route on that option.
Then a form titled `Quantity available too low` is shown reading `<warehouse name> can only provide 12.0 Units, while the quantity to order is 40.0 Units.`
When the user presses "order the available quantity".
Then the rule's route is the resupply route and its quantity to order is 12.

**AC-246 Ordering everything keeps the quantity (`RP-RULE-202`).**
Given the same warning.
When the user presses "order everything".
Then the rule's route is the resupply route and its quantity to order is still 40.

**AC-247 The last purchase date is shown per vendor price (`entities.md` section 20).**
Given a confirmed purchase order placed with vendor `Smith` on 3 May for a variant of `P`.
When the Vendors tab is opened for a reordering rule of `P`.
Then the row of `Smith` shows a last purchase date of 3 May.

---

## 14. The Product Replenish wizard

**AC-260 A buy replenishment creates a purchase order (`workflows.md` section 17).**
Given product `P` carrying the Buy route with a Vendor Price at 100.00 and a warehouse `Main`.
When the wizard is launched for 10 units.
Then one purchase order for 10 units exists and the wizard returns a notification titled `The following replenishment order have been generated` linking to it.

**AC-261 The Dropship route is never offered (`RP-RULE-207`).**
Given the Drop Shipping capability package installed and product `P` carrying the Dropship route.
When the wizard is opened for `P`.
Then the Dropship route is not among the routes offered.

**AC-262 A deleted Buy route does not break the wizard (`RP-RULE-205`).**
Given the global Buy route deleted.
When the wizard is opened and launched with another route.
Then the operation completes without error.

**AC-263 The typed unit is kept (`RP-RULE-208`).**
Given product `P` stocked in `Units` with a Vendor Price expressed in `Dozens`.
When the wizard is launched for 12 `Units`.
Then the created purchase order line carries 12 `Units`, not 1 `Dozen`.

**AC-264 An inter-warehouse replenishment picks the right supplier (`workflows.md` section 19).**
Given warehouse `WH2` resupplied from `Main`.
When the wizard is launched for `P` in `WH2` with the resupply route.
Then the created transfers show `Main` and `WH2` as counterparties through the transit location.

---

## 15. Drop shipping

**AC-280 A drop-shipped sales order creates a purchase order to the customer (`workflows.md` section 18).**
Given product `D` carrying the Dropship route with vendor `Supplier`, and a sales order for customer `Customer` for 200 units.
When the sales order is confirmed.
Then a draft purchase order for `Supplier` exists whose operation type is `Dropship` and whose destination address is `Customer`.
When the purchase order is confirmed.
Then the sales order shows a drop shipment count of 1 and a delivery count of 0, and the purchase order shows a drop shipment count of 1 and an incoming shipment count of 0.
When the drop shipment is validated for 200 units.
Then the purchase order line's received quantity is 200 and the sales order line's delivered quantity is 200.

**AC-281 The drop shipment flag is computed from the two locations (`RP-RULE-250`).**
Given a transfer from the shared vendor location to the shared customer location.
Then its drop shipment flag is true; and a transfer from the vendor location to `WH/Stock` has the flag false.

**AC-282 Vendor selection ignores the customer (`RP-RULE-092`).**
Given a drop-shipped need whose request partner is the customer `Customer`, and a product whose only Vendor Price is for `Supplier`.
When the need is run.
Then the purchase order is placed with `Supplier`.

**AC-283 Needs from different sales orders do not share a line (`RP-RULE-106`).**
Given two sales orders for the same drop-shipped product and the same vendor.
When both are confirmed.
Then the purchase order holds two lines, one per sales order line.

**AC-284 A drop shipping order covering two sales orders is split into two transfers (`RP-RULE-256`).**
Given a confirmed drop shipping purchase order holding lines of two different sales orders and no live transfer.
When the transfers are created.
Then two transfers exist, one per sales order, each holding only that order's lines, each with a note linking back to the purchase order.

**AC-285 A return of a drop shipment into stock is an ordinary receipt (`accounting-effects.md` section 3.4).**
Given a validated drop shipment of 10 units.
When 4 units are returned into `WH/Stock` rather than to the vendor.
Then the on-hand quantity at `WH/Stock` rises by 4 and the movement is valued as an entry into stock.

**AC-286 A drop shipment does not change the average cost (`accounting-effects.md` section 3.2).**
Given product `D` valued at average cost with 10 units in stock at 5.00.
When 100 units are drop-shipped at a purchase price of 9.00.
Then the average cost of `D` is still 5.00 and the on-hand quantity is still 10.

**AC-287 The lot of a drop-shipped delivery gets the sales order's shipping address (`entities.md` section 24).**
Given a serial-tracked product drop-shipped to `Customer`.
When the drop shipment is validated.
Then the lot's customer contacts contain the shipping address of the sales order, not the transfer's partner.

---

## 16. Inter-warehouse resupply

**AC-300 Adding a supplying warehouse creates a route (`workflows.md` section 19).**
Given warehouse `WH2`.
When `Main` is added to its resupply warehouses.
Then a route named `WH2: Supply Product from Main` exists, selectable on warehouses, products and product categories, whose supplied warehouse is `WH2` and whose supplying warehouse is `Main`.

**AC-301 The route's rules pull back to the supplying stock (`RP-RULE-272`).**
Given `Main` delivering in one step and the route of AC-300.
Then the route holds two rules: `WH/Stock` to the transit location, supply method take from stock; and the transit location to `WH2/Stock`, supply method trigger another rule.

**AC-302 A two-step supplying warehouse adds a third rule (`RP-RULE-272`).**
Given `Main` delivering in two steps.
Then the route holds three rules: `WH/Stock` to `WH/Output` with supply method take from stock; `WH/Output` to the transit location with supply method trigger another rule; and the transit location to `WH2/Stock` with supply method trigger another rule.

**AC-303 Removing a supplying warehouse archives the route (`RP-RULE-274`).**
Given the route of AC-300.
When `Main` is removed from `WH2`'s resupply warehouses.
Then the route is archived; and adding `Main` back unarchives the same route rather than creating a second one.

**AC-304 Switching a supplying warehouse to several delivery steps rewires the routes (`RP-RULE-275`, `RP-RULE-277`).**
Given the route of AC-301 and `Main` delivering in one step.
When `Main` is switched to two delivery steps.
Then the rule that ends at the transit location now starts at `WH/Output` with supply method trigger another rule, a rule `WH/Stock` to `WH/Output` is created, and the rule of the Replenish on Order route that went from `WH/Stock` to the transit location is archived.

**AC-305 Both warehouses appear as counterparties (`RP-RULE-084`).**
Given the route of AC-301 and a need of 10 units at `WH2/Stock`.
When the need is run.
Then the move into the transit location carries `WH2`'s partner and the move out of the transit location carries `Main`'s partner.

---

## 17. Warehouse configuration

**AC-320 A one-step reception warehouse has one reception rule (`configuration.md` section 8.1).**
Given the Purchase Inventory capability package absent and a new warehouse `Main` receiving in one step.
Then its reception route holds one rule `WH: Vendors → Stock` with action pull, supply method take from stock and the Receipt operation type.

**AC-321 Installing the purchasing package removes the first reception rule (`configuration.md` section 8.1).**
Given the Purchase Inventory capability package installed and a warehouse receiving in one step.
Then its reception route holds no rule at all, its sequence is 9, and the chain starts at the `buy` rule `WH: Stock (Buy)`.

**AC-322 Three-step reception generates the right chain (`configuration.md` section 8.1).**
Given a warehouse `Main` with the short name `WH` receiving in three steps and the Purchase Inventory capability package absent.
Then its reception route is named `Main: Receive in 3 steps (input + quality + stock)` and holds `WH: Vendors → Stock` (pull, take from stock, cancel propagation true), `WH: Input → Quality Control` (push, trigger another rule, cancel propagation true) and `WH: Quality Control → Stock` (push, trigger another rule, cancel propagation false).

**AC-323 Unticking "Buy to Resupply" archives the buy rule (`RP-RULE-279`, `configuration.md` section 8.3).**
Given `Main` with "Buy to Resupply" ticked.
When it is unticked.
Then `Main` is no longer listed on the global Buy route and its `buy` rule is archived; and the flag stays unticked after the warehouse is written again.

**AC-324 Renaming a warehouse rewrites its route and rule names (`RP-RULE-279`).**
Given warehouse `Main` with a route named `Main: Receive in 1 step (stock)`.
When the warehouse is renamed to `Central`.
Then the route is named `Central: Receive in 1 step (stock)`.

**AC-325 A deleted global route is skipped silently (`configuration.md` section 7.2).**
Given the global Buy route deleted.
When a warehouse is written.
Then no error is raised and the warehouse simply keeps no `buy` rule.

**AC-326 A warehouse with ongoing operations cannot be archived (`configuration.md` section 8.3).**
Given a warehouse with a confirmed transfer.
When it is archived.
Then the operation is refused with `You still have ongoing operations for operation types <operation type names> in warehouse <warehouse name>`.

---

## 18. Cancellation

**AC-340 Cancelling a completed move is refused (`RP-RULE-240`).**
Given a completed move whose destination is `WH/Stock`.
When it is cancelled.
Then the operation is refused with `You cannot cancel a stock move that has been set to 'Done'. Create a return in order to reverse the moves which took place.`

**AC-341 Cancel propagation cancels the next step (`RP-RULE-241`).**
Given a two-step reception chain whose first rule propagates cancellation, a confirmed receipt and its downstream internal move.
When the receipt is cancelled.
Then the internal move is cancelled too.

**AC-342 No propagation falls back to stock (`RP-RULE-242`).**
Given the same chain with cancel propagation off on the first rule.
When the receipt is cancelled.
Then the internal move is not cancelled, its supply method becomes take from stock, and it is no longer linked to the receipt.

**AC-343 Cancelling a draft purchase order cancels its chain (`RP-RULE-245`).**
Given a draft purchase order created for a make-to-order sales order line, in a two-step reception warehouse.
When the purchase order is cancelled.
Then its moves are cancelled and the customer-side delivery move survives with supply method take from stock.

**AC-344 A completed receipt is notified when its order is cancelled (`RP-RULE-246`).**
Given a purchase order with one completed transfer.
When the purchase order is cancelled.
Then a note is posted on that transfer reading `The purchase order <link to the order> this receipt is linked to was cancelled.`

**AC-345 Deleting a purchase order line with propagation off keeps the chain (`RP-RULE-247`).**
Given a purchase order line feeding a downstream move, with `propagate_cancel` false.
When the line is deleted.
Then the line's own moves are cancelled and the downstream move survives with supply method take from stock.

---

## 19. The forecast report

**AC-360 Reserved, free and incoming quantities are reconciled in order (`calculations.md` section 19).**
Given 5 units of `P` in `WH/Stock`, a confirmed delivery `D1` of 3 units on day 5 that is reserved, a confirmed delivery `D2` of 6 units on day 8 that is not reserved, and a confirmed receipt `R1` of 4 units on day 7.
When the forecast report is opened for `P` in `Main`.
Then the lines are: 3 units for `D1` with a reservation; 2 units for `D2` taken from stock; 4 units for `D2` matched with `R1`, with the lateness flag false because day 7 is not later than day 8.

**AC-361 Unsatisfied demand is flagged (`calculations.md` section 19).**
Given the setup of AC-360 without the receipt `R1`.
Then the last line carries 4 units for `D2` with the "replenishment filled" flag false.

**AC-362 An unused incoming move appears on its own line (`calculations.md` section 19).**
Given no outgoing move and a confirmed receipt of 4 units.
Then one line carries 4 units with the incoming move alone.

**AC-363 Reserving the origin chain from the report (`interfaces.md` section 1.19).**
Given a delivery whose origin internal move is in state `confirmed`.
When the reserve operation is run on the delivery from the report.
Then the internal move is reserved; and running the release operation afterwards releases it.

**AC-364 The header shows the lead time breakdown (`interfaces.md` section 4.2).**
Given product `P` with a vendor lead time of 7 days and a days to purchase of 0.
When the forecast report is opened.
Then the header shows a total lead time of 7 days and a breakdown holding `Vendor Lead Time + 7 day(s)`.

---

## 20. Reporting on vendors and suggestions

**AC-380 The on-time rate is a weighted average (`calculations.md` section 22.2).**
Given a purchase order line of 12 units received 8 on time and 4 late, and a second line of 10 units received entirely on time, both for vendor `Smith`.
When the Vendor Delay Report is grouped by vendor.
Then the on-time delivery rate for `Smith` is 81.82 percent, not 83.33 percent.

**AC-381 A vendor with no qualifying line reports no data (`calculations.md` section 22.1).**
Given vendor `Smith` with no confirmed purchase order line in the measurement window.
Then the contact's on-time delivery rate is −1, which the screens present as "no data".

**AC-382 A partially cancelled order is measured on what remains (`calculations.md` section 22.2).**
Given a purchase order of two lines, one cancelled and one received on time.
Then the Vendor Delay Report holds one row only, for the line that has moves, and its rate is 100 percent.

**AC-383 The suggested quantity deducts stock and incoming goods (`calculations.md` section 23.2).**
Given a monthly demand of 60 units, a contact whose `suggest_days` is 7 and whose `suggest_percent` is 100, 5 units on hand and 3 units incoming.
Then the suggested quantity is 6.

**AC-384 The actual-demand basis uses the negative forecast (`calculations.md` section 23.2).**
Given a contact whose basis is `actual_demand` and a product whose forecast in the chosen warehouse is −7.4, with a percentage of 100.
Then the suggested quantity is 8.

**AC-385 The suggestion respects the vendor unit (`calculations.md` section 23.3).**
Given a Vendor Price expressed in `Dozens` and a suggested quantity of 24 units.
When the suggestion is added to a purchase order.
Then the created line carries 2 `Dozens`.

---

## 21. Access and multi-company

**AC-400 An inventory user may not create a route (`RP-RULE-291`).**
Given a user of the inventory user group only.
When they try to create a route.
Then the operation is refused with the access-rights message.

**AC-401 An inventory administrator may create a route (`RP-RULE-292`).**
Given a user of the inventory administrator group.
When they create a route.
Then the route is created.

**AC-402 An inventory user may validate a receipt that updates a purchase order line (`RP-RULE-297`).**
Given a user of the inventory user group with no purchasing rights and a receipt linked to a purchase order line.
When they validate the receipt.
Then the received quantity of the purchase order line is updated and no access-rights error is raised.

**AC-403 A rule with no company is visible to every company (`RP-RULE-290`).**
Given the global Dropship route and its rule with no company.
When a user whose active company is `Company B` runs rule selection for a drop-shipped product.
Then the rule is found.

**AC-404 A reordering rule is invisible outside its company (`RP-RULE-290`).**
Given a reordering rule of `Company A`.
When a user whose active companies are only `Company B` opens the replenishment screen.
Then the rule is not listed.

**AC-405 An inter-company need uses the right company's rules (`RP-RULE-067`).**
Given two companies, a custom route of `Company A` and a need of `Company A` at a location of `Company A`.
When the need is run with elevated rights.
Then only rules with no company or of `Company A` or of its descendants are considered.

**AC-406 A foreign-currency vendor gives a foreign-currency order (`RP-RULE-299`).**
Given a Vendor Price expressed in the United States dollar (`USD`) and a company whose currency is the euro (`EUR`).
When a reordering rule orders from that vendor.
Then the created purchase order's currency is `USD` and its line price is the Vendor Price converted at today's rate when the two differ.

---

## 22. Robustness

**AC-420 Two runs of the same rule in one scheduler pass produce one line (`RP-RULE-109`).**
Given a manual reordering rule with a quantity to order of 1 for product `P` with a vendor.
When the rule is triggered twice in a row.
Then exactly one purchase order line exists and its quantity is the sum of both triggers.

**AC-421 A rounding of one unit is honoured on the purchase line (`RP-RULE-312`).**
Given the unit `Units` with a rounding of 1.0 and a need for 1.2 units.
When the need is run.
Then the created purchase order line carries 1 unit.

**AC-422 A rounding of one dozen is honoured (`RP-RULE-312`).**
Given the units `Units` and `Dozens` both with a rounding of 1.0, and a need for 1.3 dozens.
When the need is run.
Then the created line carries 1 dozen.

**AC-423 A serialization failure retries the batch (`RP-RULE-168`, `RP-RULE-304`).**
Given two concurrent scheduler runs touching the same reordering rule.
When one of them fails with a serialization error in batch mode.
Then that batch is rolled back and retried, and exactly one document is created in the end.

**AC-424 A whole batch fails before creating anything when one rule is missing (`RP-RULE-077`).**
Given two requests in one batch, one that finds a rule and one that does not.
When the batch is run.
Then no document is created for either request and the error names the failing product and location.

**AC-425 A run with user errors disabled isolates the failures (`RP-RULE-079`).**
Given the same batch run with user-facing errors disabled.
When the batch is run.
Then a procurement exception carrying the failing pair is raised, and the caller can remove that request and run the rest successfully.

---

## 23. Route and Stock Rule details

**AC-426 The warehouse list of a route is restricted to the route company (`RP-RULE-004`).**
Given a route whose company is `Company A`, and warehouses `WH-A` of `Company A` and `WH-B` of `Company B`.
When the user opens the warehouse list of that route.
Then only `WH-A` is offered; and after the route company is emptied, both `WH-A` and `WH-B` are offered.

**AC-427 A route is offered only where it is marked selectable (`RP-RULE-006`).**
Given a route `R` whose `product_selectable` is true and whose `product_category_selectable`, `warehouse_selectable`, `package_type_selectable`, `sale_selectable` and `shipping_selectable` are all false.
Then `R` is offered on a product form; and it is offered on none of a product category, a warehouse, a package type, a sales order line or a shipping method.
When `sale_selectable` is set to true.
Then `R` becomes selectable on a sales order line and stays unavailable on the other four.

**AC-428 Two routes may carry the same name (`RP-RULE-008`).**
Given a route named `Buy`.
When a second route named `Buy` is created.
Then the creation succeeds and the two routes are distinguished by their surrogate keys.

**AC-429 Unarchiving a route brings its rules back (`RP-RULE-011`, `RP-RULE-012`).**
Given an active route holding rule `S1` whose destination location is active and rule `S2` whose destination location is archived.
When the route is archived.
Then `S1` is archived and `S2` is left as it was.
When the route is unarchived.
Then `S1` is active again and `S2` is still untouched.

**AC-430 A rule cannot be saved without its mandatory fields (`RP-RULE-020`).**
Given a new stock rule form with only a name typed.
When the user saves.
Then the save is refused, and naming `action`, `destination_location`, `route`, `procure_method`, `operation_type` and `auto` makes the save succeed.

**AC-431 A rule of the wrong company is refused (`RP-RULE-021`).**
Given a route `R` whose company is `Company A` and a rule `S` of that route.
When `Company B` is written on `S`.
Then the write is refused with `Rule S belongs to Company B while the route belongs to Company A.`

**AC-432 A new rule takes the active company (`RP-RULE-022`).**
Given a user whose active company is `Company B` and a route with no company.
When the user creates a rule on that route without naming a company.
Then the rule's company is `Company B`.

**AC-433 Rule references must belong to the rule company (`RP-RULE-023`).**
Given a rule of `Company A`.
When a location of `Company B` is written as its `destination_location`.
Then the write is refused by the shared company-consistency check; and a location with no company is accepted.

**AC-434 Choosing a route realigns the company and the operation type (`RP-RULE-027`).**
Given a rule form with an operation type whose warehouse belongs to `Company B`.
When a route of `Company A` is chosen.
Then the rule's company becomes `Company A` and its `operation_type` is emptied.

**AC-435 The denormalized route sequence follows the route (`RP-RULE-029`).**
Given a route with sequence 10 holding one rule whose `route_sequence` is 10.
When the route sequence is changed to 5.
Then the rule's `route_sequence` becomes 5 without the rule being edited.

**AC-436 Rules are ordered by sequence then by key (`RP-RULE-030`).**
Given three rules of one route with sequences 20, 10 and 20, created in that order.
When the rule list of the route is read.
Then the order is: the rule with sequence 10, then the first rule with sequence 20, then the second.

**AC-437 Duplicating a rule marks the copy (`RP-RULE-031`).**
Given a rule named `WH: Vendors to Stock` with a lead time of 3 days.
When it is duplicated.
Then the copy is named `WH: Vendors to Stock (copy)` and its lead time is 3 days.

**AC-438 A conditional push rule is skipped for a move that does not match (`RP-RULE-033`, `RP-RULE-131`).**
Given two push rules at `WH/Input`: `Pa` with sequence 10 and the condition `priority = "1"`, and `Pb` with sequence 20 and no condition.
When a move of normal priority arrives at `WH/Input`.
Then `Pa` is excluded and `Pb` is applied.
When a move of urgent priority arrives at `WH/Input`.
Then `Pa` is applied.

---

## 24. Reordering Rule field restrictions and the replenishment location

**AC-439 A reordering rule only watches storable goods (`RP-RULE-047`).**
Given a service product `ServiceProduct` and a consumable product `ConsumableProduct`.
When either is chosen on a reordering rule.
Then the choice is refused by the shared "value not allowed" check; and choosing storable product `P` is accepted.

**AC-440 The location list of a reordering rule is restricted (`RP-RULE-048`).**
Given a reordering rule whose warehouse is `Main` of `Company A`.
Then the locations offered are the internal and view locations that belong to `Main` or to no warehouse and to `Company A` or to no company; the shared vendor location, the shared customer location and a location of `WH-B` are not offered.

**AC-441 The route list of a reordering rule is restricted (`RP-RULE-050`).**
Given a route `R1` selectable on products, a route `R2` not selectable on products but holding a `buy` rule, and a route `R3` selectable on nothing and holding only pull rules.
Then `R1` and `R2` are offered on a reordering rule and `R3` is not.

**AC-442 Setting a bill of materials sets the route (`RP-RULE-053`).**
Given a reordering rule for `P` with no preferred route, and a Manufacture route holding a rule with action `manufacture`.
When a bill of materials of `P` is written on the rule.
Then the rule's `route` becomes that Manufacture route.

**AC-443 The vendor price list of a reordering rule is restricted (`RP-RULE-054`).**
Given a reordering rule for variant `P1` of template `T` in `Company A`.
Then the vendor prices offered are those of `P1` and those of `T` with no variant, restricted to `Company A` or to no company; a vendor price of another template is not offered.

**AC-444 The bill list of a reordering rule is restricted (`RP-RULE-055`).**
Given bills `B1` of type `normal` for `P`, `B2` of type `kit` for `P` and `B3` of type `normal` for another product.
Then only `B1` is offered on a reordering rule for `P`.

**AC-445 Two replenishment locations may not nest (`RP-RULE-189`).**
Given `WH/Stock` flagged as a replenishment location and a child location `WH/Stock/Shelf 1`.
When `replenish_location` is set on `WH/Stock/Shelf 1`.
Then the write is refused with `Another parent/sub replenish location WH/Stock exists, if you wish to change it, uncheck it first`

**AC-446 Only an internal location may be a replenishment location (`RP-RULE-190`).**
Given an internal location whose `replenish_location` is true.
When its usage is changed to `transit` and the record is saved.
Then `replenish_location` is forced to false, and setting it to true again on that location has no effect while the usage is not `internal`.

**AC-447 The Stock Rules Report needs a warehouse (`RP-RULE-209`, `RP-RULE-046`).**
Given a company with no warehouse.
When a user opens the Stock Rules Report wizard.
Then the wizard is refused with the redirect warning `Please create a warehouse for company <company name>.`

---

## 25. Rule selection details

**AC-448 The lowest route sequence wins inside one source (`RP-RULE-064`).**
Given two routes on product `P`: `R1` with sequence 5 holding a rule `S1` to `WH/Stock` with sequence 20, and `R2` with sequence 10 holding a rule `S2` to `WH/Stock` with sequence 1.
When a need for `P` appears at `WH/Stock`.
Then `S1` is selected, because the route sequence is compared before the rule sequence.

**AC-449 A rule of another warehouse is discarded (`RP-RULE-066`).**
Given a rule `S1` whose `warehouse` is `WH-B` and a rule `S2` with no warehouse, both targeting `WH-A/Stock` on the same route.
When a need whose warehouse is `WH-A` appears at `WH-A/Stock`.
Then `S2` is selected and `S1` is discarded.

**AC-450 An inter-company transit need also accepts customer rules (`RP-RULE-069`).**
Given a need at the shared inter-company transit location and a delivery rule whose destination is the shared customer location.
When the need is run.
Then that delivery rule is selected, without a duplicate rule having to be created for the transit location.

**AC-451 A drop-shipped sales need is restricted to its own company (`RP-RULE-070`).**
Given the Drop Shipping capability package installed, a parent company `Group` holding a Dropship rule and a child company `Child` holding its own Dropship rule, and a sales order line of `Child`.
When the line is confirmed.
Then the rule of `Child` is selected and the rule of `Group` is not considered.

**AC-452 Push selection reads only push rules and ignores the warehouse route filter (`RP-RULE-071`).**
Given at `WH/Input` a pull rule `S1` and a push rule `S2` on a route that is not attached to the warehouse.
When goods arrive at `WH/Input`.
Then `S2` is applied and `S1` is never considered.

---

## 26. The run algorithm details

**AC-453 The run operation fills three defaults (`RP-RULE-073`).**
Given a request for 5 units of `P` at `WH/Stock` whose values carry neither company, nor priority, nor planned date, submitted on 4 March at 09:00.
When the run operation is called.
Then the request is processed with company = the company of `WH/Stock`, priority = `"0"` and planned date = 4 March at 09:00.

**AC-454 A request for a service or for zero is skipped silently (`RP-RULE-074`).**
Given one request for 5 units of the service product `ServiceProduct` and one request for 0.004 units of `P` whose unit rounds to 0.01.
When the batch is run.
Then no document is created, no error is raised, and no message is collected.

**AC-455 A kit request is exploded before rule selection (`RP-RULE-075`).**
Given product `K` with a bill of materials of type `kit` made of 2 units of `C1` and 3 units of `C2`, and a request for 4 units of `K` at `WH/Stock` whose origin is `SO001`.
When the batch is run.
Then rule selection runs for 8 units of `C1` and 12 units of `C2`, both at `WH/Stock`, both with origin `SO001`, and no rule is ever selected for `K`.

**AC-456 A buy route pulls in the reception route of the warehouse (`RP-RULE-076`).**
Given a warehouse receiving in two steps and a request carrying only the Buy route.
When the batch is run.
Then the reception route of the warehouse is added to the request routes, the rule "Vendors to Stock" of that route is selected first, and the chain reaches the Buy rule at the vendor location.

**AC-457 One failing action handler does not stop the others (`RP-RULE-078`).**
Given a batch holding one `buy` request that finds no vendor price and one `pull` request that is valid.
When the batch is run.
Then the pull request creates its move, the buy request fails, and the collected failure is raised after both handlers have run.

---

## 27. The pull action details

**AC-458 The supply method of a split rule is written as take-from-stock (`RP-RULE-081`).**
Given a rule whose `procure_method` is `mts_else_mto`.
When it creates a move.
Then the move's `procure_method` is `make_to_stock`.

**AC-459 The company of a created move follows the rule (`RP-RULE-082`, `RP-RULE-301`).**
Given a rule with no company whose source location belongs to `Company A`, and a request whose company is `Company B`, run by a user whose active company is `Company C`.
When the move is created.
Then the move's company is `Company A`.

**AC-460 The rule address overrides the request partner (`RP-RULE-083`).**
Given a rule whose `partner_address` is `Warehouse B Address` and a request whose `values.partner` is `Customer`.
When the move is created.
Then the move's partner is `Warehouse B Address`.

**AC-461 A created move is confirmed at once (`RP-RULE-087`).**
Given a two-step reception route.
When a need at `WH/Stock` creates the "Input to Stock" move.
Then that move is immediately confirmed, which is what makes it create the supply need at `WH/Input`.

---

## 28. The buy action details

**AC-462 The company of a buy request follows the rule (`RP-RULE-090`).**
Given a `buy` rule of `Company A` and a request whose company is `Company B`.
When the buy action runs.
Then the purchase order is created in `Company A`.

**AC-463 Vendor selection uses the later of the planned date and today (`RP-RULE-093`).**
Given today is 10 March, a request planned for 5 March, and a Vendor Price valid from 8 March.
When the buy action runs.
Then vendor selection is asked for 10 March and the Vendor Price is found.
Given a second request planned for 20 March.
Then vendor selection is asked for 20 March.

**AC-464 A weekly vendor groups on a window around the planned date (`RP-RULE-101`).**
Given a vendor whose grouping mode is `week` and whose `grouping_weekday` is `default`, and a need planned for Wednesday 10 September, which is weekday 3.
When the buy action runs.
Then the window runs from the start of Sunday 7 September to the end of Saturday 13 September, and a second need planned for Friday 12 September joins the same order.
Given a third need planned for Monday 15 September.
Then a second purchase order is created.

**AC-465 A new purchase order is created with elevated rights (`RP-RULE-104`).**
Given a salesperson with no purchasing rights who confirms a make-to-order sales order.
When the buy action runs.
Then the purchase order is created, and the salesperson is not among its followers.

**AC-466 Merging into an existing order accumulates the origins and the references (`RP-RULE-105`).**
Given a draft purchase order whose origin is `SO001` and which carries the reference of `SO001`.
When a second need whose origin is `SO002` and whose reference is that of `SO002` merges into it.
Then the order's origin becomes `SO001, SO002` and its references hold both.

**AC-467 Merged requests sum their quantities and union their downstream moves (`RP-RULE-107`).**
Given two requests for the same product, unit, vendor and merge key, one for 6 units linked to move `M1` and one for 4 units linked to move `M2`, only the second carrying a reordering rule.
When the batch is run.
Then one line of 10 units is created, linked to both `M1` and `M2`, and carrying the reordering rule of the second request.

**AC-468 The unit price of an updated line follows the new bracket (`RP-RULE-112`, `RP-RULE-300`).**
Given a product with two vendor prices for one vendor in United States dollars: 10 units at 30.00 and 20 units at 25.00, an order in euros, and a rate of 0.50 euro per dollar today.
Given a draft line of 15 units priced at 15.00 euros, which is 30.00 dollars converted.
When a second need for 10 units merges into it.
Then the line holds 25 units and its unit price becomes 12.50 euros, which is 25.00 dollars converted at today's rate.

**AC-469 A variant description is appended to the line name (`RP-RULE-115`).**
Given a request whose `values.product_description_variants` is `Color: Red` and a product whose display name is `Chair`.
When the line is created.
Then the line name is `Chair` followed by a new line and `Color: Red`.

**AC-470 The order date is the earliest of the requests (`RP-RULE-116`).**
Given two positive requests for one new purchase order, one planned for 20 March with a vendor lead time of 3 days and one planned for 25 March with the same vendor.
When the order is created.
Then its order date is 17 March, the earlier of 17 March and 22 March.

---

## 29. The manufacture action

**AC-471 The bill of materials is chosen in a fixed order (`RP-RULE-121`).**
Given product `M` with bill `B1` of type `normal` tied to the manufacturing operation type and bill `B2` of type `normal` tied to no operation type, and a reordering rule for `M` whose `bill_of_materials` is `B2`.
When the reordering rule is run without a bill being forced in the values.
Then `B2` is used.
When the same need is run with `values.bill_of_materials` = `B1`.
Then `B1` is used.
When a need with neither is run through the rule whose operation type is the manufacturing operation type.
Then `B1` is used.

**AC-472 An existing draft manufacturing order is extended (`RP-RULE-122`).**
Given a draft manufacturing order for 10 units of `M` with bill `B1`, the manufacturing operation type, no responsible user, not planned, and no references.
When a second need for 5 units of `M` with the same bill, operation type, company and no references is run.
Then no second order is created and the existing order holds 15 units.

**AC-473 A different reference set forbids the merge (`RP-RULE-122`).**
Given the same draft order, now carrying the reference of `SO001`.
When a need carrying the reference of `SO002` is run.
Then a second manufacturing order is created.

**AC-474 A production group must be an ancestor of the candidate's own (`RP-RULE-123`).**
Given a draft manufacturing order whose production group is `G2`, whose ancestors are `G1` and `G2`.
When a need whose `values.production_group` is `G1` is run.
Then that order is extended.
When a need whose `values.production_group` is `G3` is run.
Then a new order is created.

**AC-475 A reordering rule only extends an order that is early enough (`RP-RULE-124`).**
Given bill `B1` with a manufacturing lead time of 2 days, a need from a reordering rule planned for 20 March, so a procurement date at the end of 18 March, and a draft manufacturing order whose deadline is 17 March.
When the need is run.
Then that order is extended.
Given instead a draft order whose deadline is 19 March.
Then a new order is created.
Given instead a confirmed order whose start date is 18 March.
Then that order is extended.

**AC-476 Batch sizes always create new orders (`RP-RULE-125`).**
Given bill `B1` with batch sizes enabled at 4 units per batch, and a need for 10 units.
When the need is run.
Then three manufacturing orders are created, for 4, 4 and 2 units, even though a matching draft order already existed.

**AC-477 A manufacturing order from a reordering rule with components waits for the batch (`RP-RULE-127`, `RP-RULE-175`).**
Given a reordering rule batch holding a rule for `M`, whose bill has one component `C`, and a rule for `C`.
When the batch is run.
Then the manufacturing order for `M` is created as a draft and is confirmed only after every rule of the batch has been processed.
Given instead a manufacturing order created from a sales order line and having component moves.
Then it is confirmed immediately.

---

## 30. Push application and manual push moves

**AC-478 A transparent push loops only while the destination changes (`RP-RULE-134`).**
Given two transparent push rules, the first rewriting `WH/Input` to `WH/Quality Control` and the second rewriting `WH/Quality Control` to `WH/Stock`.
When a move arriving at `WH/Input` is pushed.
Then its destination becomes `WH/Stock` in one pass, and the loop stops because the third pass finds no further change.

**AC-479 A manual push rule copies the completed quantity (`RP-RULE-135`).**
Given a move demanding 10 units of which 7 were completed, and a manual push rule at its destination.
When push application runs.
Then the created move demands 7 units.
Given instead a move whose demand quantity is −4.
Then the created move demands −4 units.

**AC-480 The destination of a pushed move follows the final location and the partner (`RP-RULE-136`).**
Given a manual push rule from `WH/Output` to the shared customer location, and a source move whose partner is `Customer` whose delivery address location is `Customers/Customer`.
When push application runs.
Then the created move's destination is `Customers/Customer`.
Given instead a push rule from `WH/Input` to `WH/Stock` and a source move whose final location is `WH/Stock/Shelf 1`.
Then the created move's destination is `WH/Stock/Shelf 1`.

**AC-481 A pushed move carries the final location only when it is still useful (`RP-RULE-137`).**
Given a source move whose destination is `WH/Input` and whose final location is `WH/Stock`.
When push application creates the next move.
Then the created move carries `WH/Stock` as final location.
Given instead a source move whose destination is `WH/Stock/Shelf 1` and whose final location is `WH/Stock`.
Then the created move carries no final location.

**AC-482 A pushed move is chained only when its source holds stock (`RP-RULE-138`).**
Given a manual push rule from `WH/Output` to the shared customer location.
When push application runs on a move arriving at `WH/Output`.
Then the created move's supply method is `make_to_order` and it is registered as a downstream move of the source move.
Given instead a push rule whose source is the shared vendor location.
Then the created move's supply method is `make_to_stock` and no downstream link is created.

**AC-483 Downstream moves are rewired after a push (`RP-RULE-139`).**
Given a move `A` from `WH/Input` to `WH/Stock` with final location `WH/Packing Zone`, a downstream move `B` whose source is `WH/Packing Zone`, and a push rule that creates move `N` from `WH/Stock` to `WH/Packing Zone`.
When push application runs.
Then `B` is detached from `A` and attached to `N`.
Given instead a downstream move `C` whose source is neither `WH/Stock` nor a descendant of it and no new move was created.
Then the make-to-order link between `A` and `C` is broken and `C` becomes `make_to_stock`.

**AC-484 A push rule shifts the scheduled date and copies the deadline (`RP-RULE-221`).**
Given a source move scheduled for 10 March at 08:00 with a deadline of 12 March, and a push rule with a lead time of 2 days.
When push application creates the next move.
Then that move is scheduled for 12 March at 08:00 and its deadline is 12 March.

---

## 31. The values written on a confirm-time need

**AC-485 A non-split move procures its whole demand (`RP-RULE-141`).**
Given a move of 10 units whose rule's supply method is `make_to_order`, and 12 free units at its source location.
When the move is confirmed.
Then the need created origin is for 10 units, not for zero.

**AC-486 Only a make-to-order move links its need (`RP-RULE-143`).**
Given a move `M` of 10 units whose rule's supply method is `make_to_order`.
When `M` is confirmed.
Then the created need carries `M` in `values.move_destinations`.
Given instead a move whose rule's supply method is `mts_else_mto` and a shortage of 8 units.
Then the created need for 8 units carries no downstream move.

**AC-487 The partner of a need comes from the transit warehouse (`RP-RULE-144`).**
Given a move from the company's internal transit location to `WH-A/Stock`, whose rule's supply method is `make_to_order`, and `WH-A` whose partner is `Main Address A`.
When the move is confirmed.
Then the need carries `values.partner` = `Main Address A`.
Given instead a move whose rule's supply method is `make_to_stock`.
Then the need carries no partner.

**AC-488 The warehouse of a need falls back to the route's supplier warehouse (`RP-RULE-145`).**
Given a move whose `warehouse` is empty, whose operation type has no warehouse, whose source location is the shared inter-company transit location which belongs to no warehouse, and whose rule belongs to the resupply route `A: Supply Product from B`.
When the move is confirmed.
Then the need carries `values.warehouse` = `B`.

**AC-489 The routes of a need fall back to the package type (`RP-RULE-146`).**
Given a move that carries no route and whose result package has a package type carrying route `R`.
When the move is confirmed.
Then the need carries `values.routes` = `R`.

**AC-490 The name and origin of a need come from the rule and the reference (`RP-RULE-147`).**
Given a move created by rule `WH: Stock to Output` and carrying the reference named `SO001`.
When the move is confirmed.
Then the need is named `WH: Stock to Output` and its origin is `SO001`.
Given instead a move with no rule, no reference and no origin, inside transfer `WH/OUT/00007`.
Then the need is named `/` and its origin is `WH/OUT/00007`.

**AC-491 The dates of a need are taken from the move (`RP-RULE-148`).**
Given a move scheduled for 20 March whose source location is `WH/Input`, which is not a descendant of the warehouse stock location `WH/Stock`.
When the move is confirmed.
Then the need carries `values.date_planned` = 20 March and no `values.date_order`.
Given instead a move scheduled for 20 March whose source location is `WH/Stock/Shelf 1`, which is a descendant of `WH/Stock`, and whose rule chain from that location accumulates a purchase delay of 3 days.
Then the need carries `values.date_planned` = 20 March and `values.date_order` = 17 March.

**AC-492 A move reserved at confirmation gets today as reservation date (`RP-RULE-149`).**
Given an operation type that reserves at confirmation, and today is 4 March.
When a move of that operation type is confirmed.
Then its `reservation_date` is 4 March.

**AC-493 A negative merged move is turned around (`RP-RULE-150`).**
Given a merged move of −4 units from `WH/Stock` to the shared customer location whose final location is the customer location.
When the move is confirmed.
Then its source becomes the customer location, its destination becomes `WH/Stock`, its final location becomes `WH/Stock`, its demand becomes +4 units, its operation type becomes the return operation type of its own operation type, and its supply method becomes `make_to_stock`.

---

## 32. Adjusting the supply method of an existing move

**AC-494 The rule search for an existing move matches both locations (`RP-RULE-151`).**
Given a move from `WH/Input` to `WH/Stock`, and a rule `S` whose `location_source` is `WH/Input`, whose `destination_location` is `WH/Stock` and whose action is `pull`.
When the supply method of the move is adjusted.
Then `S` is found and written on the move.
Given instead only a push rule between the same two locations.
Then it is not considered.

**AC-495 No rule means take from stock (`RP-RULE-152`).**
Given a move from an isolated location `Spare` to `WH/Stock` with no rule anywhere between them.
When the supply method of the move is adjusted.
Then the move's supply method becomes `make_to_stock` and its `rule` stays empty.

**AC-496 A split rule is written as take from stock on an existing move (`RP-RULE-153`).**
Given a move whose adjusted rule has supply method `mts_else_mto`.
When the supply method is adjusted.
Then the rule is written on the move and the move's supply method becomes `make_to_stock`.
Given instead a rule with supply method `make_to_order`.
Then the move's supply method becomes `make_to_order`.

---

## 33. Reordering rule evaluation and the scheduler details

**AC-497 The forecast is read at the end of the horizon day and corrected (`RP-RULE-161`).**
Given a reordering rule for `P` at `WH/Stock` whose `lead_horizon_date` is 20 March, 4 units on hand, an outgoing move of 10 units on 20 March at 18:00 and a draft purchase order line of 3 units for that location.
When the rule is evaluated.
Then `quantity_forecast` is 4 − 10 + 3 = −3.

**AC-498 Snoozing never affects the scheduler (`RP-RULE-164`).**
Given a manual reordering rule snoozed until 30 March and an automatic reordering rule for the same product at another location.
When the scheduler runs on 20 March.
Then the automatic rule is processed and the manual rule is neither processed nor shown on the replenishment report.

**AC-499 Rules are processed in batches of one thousand (`RP-RULE-165`).**
Given 2500 automatic reordering rules with a positive quantity to order.
When the scheduler runs in batch mode.
Then three batches are processed and committed, of 1000, 1000 and 500 rules, each with its own transaction.

**AC-500 Only a strictly positive quantity produces a request (`RP-RULE-166`).**
Given three rules whose quantities to order are 0, 0.004 at a unit rounding of 0.01, and 5.
When the scheduler runs.
Then only the third rule produces a request.

**AC-501 A reordering-rule run fails hard when no vendor exists (`RP-RULE-170`, `RP-RULE-094`, `RP-RULE-095`).**
Given a reordering rule for `P` that must buy, and `P` has no Vendor Price.
When the rule is run.
Then the run fails with `There is no matching vendor price to generate the purchase order for product P (no vendor defined, minimum quantity not reached, dates not valid, ...). Go on the product form and complete the list of vendors.` and a warning activity is created for the product's responsible user.
Given instead the same need coming from a confirmed stock move rather than from a reordering rule.
Then no error is raised, the downstream move falls back to taking from stock or is cancelled, and the responsible user is notified.

**AC-502 A scheduler failure aborts the run and is logged (`RP-RULE-174`).**
Given a scheduler run whose second task raises an unexpected error.
When the scheduled action runs.
Then the error is logged with its stack trace, the scheduled run stops, and the next daily run starts from scratch.

**AC-503 Ordering clears the manual override (`RP-RULE-182`).**
Given a reordering rule whose `quantity_to_order_manual` is 12 and whose computed quantity is 7.
When the user presses Order.
Then 12 units are ordered, `quantity_to_order_manual` becomes 0, and `quantity_to_order` falls back to the recomputed value.

**AC-504 Opening the report cleans up and recomputes (`RP-RULE-187`, `RP-RULE-059`).**
Given a temporary reordering rule whose quantity to order has fallen to zero and an ordinary rule with a stale computed quantity.
When the replenishment report is opened with the recompute flag in its context.
Then the temporary rule is deleted and the ordinary rule's `quantity_to_order_computed` and `deadline_date` are recomputed.

---

## 34. The wizards

**AC-505 Editing the minimum in the information dialog uses the reader's own rights (`RP-RULE-200`).**
Given an inventory user who is not an administrator.
When that user opens the Replenishment Information dialog.
Then the dialog is refused by the shared access-rights check, because only an administrator may read it.
Given an administrator who edits the minimum quantity in the dialog.
Then the reordering rule is written with that administrator's rights, not with elevated rights.

**AC-506 A supplying warehouse chosen from the replenish wizard writes back to the wizard (`RP-RULE-203`).**
Given the Product Replenish wizard for `P` in `WH-A`, from which the Replenishment Information dialog was opened.
When the user presses "Order all" on the option that resupplies from `WH-B`.
Then the route `A: Supply Product from B` is written on the Product Replenish wizard, the reordering rule is left unchanged, and the Product Replenish wizard is reopened.

**AC-507 Buy routes appear in the replenish wizard only with a vendor price (`RP-RULE-206`).**
Given product `P` with no Vendor Price and a Buy route whose rule's operation type has code `incoming`.
Then that route is not offered in the Product Replenish wizard.
When a Vendor Price is added to `P`.
Then the route is offered.

---

## 35. Dates, deadlines and lateness

**AC-508 A purchase order line dates its receipt moves (`RP-RULE-222`).**
Given a purchase order line planned for 20 March inside an order planned for 25 March.
When the order is confirmed.
Then the receipt move is scheduled for 20 March and its deadline is 20 March.
Given instead a line with no planned date.
Then the receipt move is scheduled for 25 March with a deadline of 25 March.

**AC-509 A deadline change posts one note per document (`RP-RULE-225`).**
Given a chain of a receipt and a delivery, the delivery being the origin of the change.
When the deadline of the delivery moves by two days.
Then a note is posted on the receipt with subject `Deadline updated due to delay on WH/OUT/00003` and body `The deadline has been automatically updated due to a delay on <link to WH/OUT/00003>.`, authored by the system account.
When the same propagation happens again immediately.
Then no second note is posted, because the most recent message already carries that subject.

**AC-510 Writing a planned date on a line moves the deadline of its moves (`RP-RULE-226`).**
Given a purchase order line with one receipt move that is not completed.
When the line's `date_planned` is set to 22 March.
Then the receipt move's deadline becomes 22 March.
Given instead a line whose receipt move is already completed but which feeds a downstream delivery move.
Then the deadline is written on that downstream move instead.

**AC-511 A fully received line keeps its planned date (`RP-RULE-227`).**
Given a purchase order line whose only move is completed.
When the shared date-update operation asks for 22 March.
Then the line's `date_planned` is unchanged.
Given instead a line with no move at all.
Then the line's `date_planned` becomes 22 March.

**AC-512 Lead times are counted in calendar days (`RP-RULE-228`).**
Given a vendor lead time of 3 days and a need planned for Monday 23 March.
When the order date is computed.
Then it is Friday 20 March, three calendar days earlier, and the intervening weekend is not skipped.

**AC-513 The procurement date of a reordering rule subtracts the horizon again (`RP-RULE-229`).**
Given a company whose partner declares no time zone, a replenishment horizon of 365 days, and a rule whose `lead_horizon_date` is 4 March of next year.
When the rule is run.
Then the procurement date is 4 March of next year at 12:00 in coordinated universal time, minus 365 days, that is 4 March of this year at 12:00, so the documents created are dated this year and not next year.

---

## 36. Cancellation details

**AC-514 The parameter cancels origin moves too (`RP-RULE-243`).**
Given the stored parameter `inventory.cancel_originating_moves` present, and a chain of an origin receipt move and a downstream delivery move whose `propagate_cancel` is true.
When the delivery move is cancelled.
Then the receipt move is cancelled as well.
Given the parameter absent.
Then the receipt move stays open.

**AC-515 Breaking a make-to-order link recomputes the downstream state (`RP-RULE-244`).**
Given a waiting delivery move fed by one origin move.
When the link is broken.
Then the origin move leaves the delivery move's origin list, the delivery move's supply method becomes `make_to_stock`, and its state is recomputed to `confirmed` or `assigned` from its own reservation.

---

## 37. Drop shipping consequences

**AC-516 A dropship operation type always keeps its locations (`RP-RULE-251`).**
Given an operation type whose code is `dropship`.
Then its default source location is the shared vendor location, its default destination location is the shared customer location, its warehouse is empty and it appears in the operations overview.
When a user writes `WH/Input` as its default source location.
Then the default source location reverts to the shared vendor location and the warehouse stays empty, because both are derived from the code.

**AC-517 A drop shipping purchase order always carries a destination address (`RP-RULE-252`).**
Given a sales order for `Customer` whose shipping address is `Customer Delivery`.
When the drop-shipped line is confirmed.
Then the purchase order's `destination_address` is `Customer Delivery`.

**AC-518 The delivered quantity of a drop-shipped line follows the ordered quantity (`RP-RULE-253`).**
Given a sales order line for 200 units of `D` drop-shipped, and a confirmed purchase order line for 200 units.
Then the sales order line's delivered quantity is 200 even before the drop shipment is validated.
When the purchase order line is raised to 250 units.
Then the sales order line's delivered quantity becomes 250.
When the purchase order is cancelled.
Then the delivered quantity falls back to 0.

**AC-519 A kit with drop-shipped components is measured from its moves (`RP-RULE-253`).**
Given a kit `K` whose bill of materials holds one drop-shipped component `C1` at 1 per kit and one stocked component `C2` at 1 per kit, and a sales order for 10 kits.
When the order is confirmed.
Then the purchase order line created for `C1` names `C1`, not `K`, so the drop-shipped rule does not apply and the delivered quantity of the kit line stays 0.
When the drop shipment of 10 units of `C1` and the delivery of 10 units of `C2` are both validated.
Then the delivered quantity of the kit line is 10.
When only the drop shipment is validated.
Then the delivered quantity of the kit line is 0, because a kit is delivered only when every component is.

**AC-520 A partial return of a kit reduces the delivered quantity proportionally.**
Given the same kit `K` sold in 4 units, with a drop-shipped component `C1` at 4 per kit, fully delivered, so the delivered quantity is 4.
When 4 units of `C1` are returned to the vendor, one quarter of the 16 units shipped.
Then the delivered quantity of the kit line becomes 3.

**AC-521 Cancelled transfers leave a kit undelivered.**
Given the same kit `K` sold in 10 units with both component transfers created.
When every one of those transfers is cancelled.
Then the delivered quantity of the kit line is 0.

**AC-522 A drop-shipped kit move carries its bill of materials line.**
Given the kit `K` sold with the drop shipping route.
When the drop shipment is created.
Then each of its move lines carries the bill of materials line of the component it corresponds to, so the kit can be reconstituted from the transfer.

**AC-523 A purchased sales order line may not change product (`RP-RULE-254`).**
Given a sales order line with one purchase order line attached.
When a user of the purchase user group tries to change the product on the sales order line.
Then the change is refused by the shared read-only check.

**AC-524 A drop-shipped line is flagged make to order (`RP-RULE-255`).**
Given product `D` whose only route is Dropship, whose rule's operation type has the shared vendor location as default source and the shared customer location as default destination.
When a sales order line for `D` is created.
Then the line is flagged make to order.

**AC-525 Counters split drop shipments from receipts and deliveries (`RP-RULE-257`).**
Given a purchase order with one ordinary receipt and one drop shipment, and a sales order with one delivery and one drop shipment.
Then the purchase order shows a drop shipment count of 1 and an incoming shipment count of 1; and the sales order shows a delivery count of 1 and a drop shipment count of 1.

**AC-526 A drop shipping reference links the sales order when there is exactly one (`RP-RULE-258`).**
Given a drop shipping purchase order whose every line belongs to sales order `SO001`.
Then the stock reference of that purchase order also links `SO001`.
Given instead an order whose lines belong to `SO001` and `SO002`.
Then the reference links neither sales order.

**AC-527 A drop-shipped move uses the outgoing description (`RP-RULE-259`).**
Given product `D` whose incoming picking description is `Check the seals` and whose outgoing picking description is `Fragile, handle with care`.
When the drop shipment move is created.
Then its description is `Fragile, handle with care`.

---

## 38. Drop shipping to a subcontractor

**AC-528 Installing the capability package creates the per-company records (`workflows.md` section 20.2).**
Given a company `Company A` whose subcontracting location is `Subcontracting/A`.
When the Dropship and Subcontracting Management capability package is installed.
Then a sequence with code `dropship_subcontractor_transfer` named `Dropship Subcontractor (Company A)` with prefix `DSC/` and five digits exists; an operation type named `Dropship Subcontractor` with code `dropship`, no warehouse, prefix `DSC`, source = the shared vendor location, destination = `Subcontracting/A` and "use existing lots" off exists and is remembered on the company; and a `buy` rule in the global Dropship route from the shared vendor location to `Subcontracting/A` with supply method `make_to_stock` and that operation type exists.

**AC-529 A warehouse that resupplies subcontractors holds the return pull rule (`workflows.md` section 20.2).**
Given warehouse `Main` whose "Resupply Subcontractors" flag is on.
Then the global Dropship route holds a pull rule from `Subcontracting/A` to the production location with supply method `make_to_order` and the warehouse's subcontracting operation type.
When the flag is switched off.
Then that rule is archived.

**AC-530 The route and the operation type follow the last active rule (`RP-RULE-263`).**
Given one company and one warehouse whose "Resupply Subcontractors" flag is on.
When the flag is switched off.
Then the company's drop-ship-to-subcontractor operation type becomes inactive and the global Dropship route becomes inactive.
Given a second company whose own warehouse still resupplies subcontractors.
Then the global Dropship route stays active while the first company's operation type is inactive.

**AC-531 The subcontractor becomes the destination address (`RP-RULE-260`).**
Given a subcontracted product `S` whose bill of materials names subcontractor `Sub` and whose component `C` carries the Dropship route, and a manufacturing order for `S` whose subcontractor is `Sub`.
When the component need is run through the `buy` rule whose destination is the subcontracting location and whose values carry no partner.
Then the created purchase order's `destination_address` is `Sub`.

**AC-532 Components for two subcontractors never share an order (`RP-RULE-261`).**
Given the same vendor `V` supplying component `C` to subcontractors `Sub1` and `Sub2`, and two needs, one carrying `Sub1` as partner and one carrying `Sub2`.
When both are run.
Then two purchase orders are created for `V`, one whose destination address is `Sub1` and one whose destination address is `Sub2`.
Given instead two needs both carrying `Sub1`.
Then one purchase order with one merged line is created.

**AC-533 A reordering rule at a subcontracting location carries the subcontractor (`RP-RULE-262`).**
Given a reordering rule for `C` whose location is `Subcontracting/A`, a location whose single subcontractor is `Sub1`.
When the rule is run.
Then the values carry `partner` = `Sub1`, and the purchase order created for `C` has `Sub1` as destination address.
Given instead a subcontracting location with two subcontractors.
Then no partner is carried and the purchase order is delivered to the company.

**AC-534 A single drop-ship order is created for several subcontracted products of one subcontractor.**
Given two subcontracted products whose bills both name subcontractor `Sub` and whose components are both drop-shipped from vendor `V`.
When both are ordered in one purchase order to `Sub` and confirmed.
Then exactly one drop-ship purchase order to `V` is created, whose destination address is `Sub`, holding one line per component.

**AC-535 The operation type of a drop-ship-to-subcontractor transfer is the dedicated one.**
Given the need of `AC-531`.
When the purchase order to `V` is confirmed.
Then the created transfer's operation type is `Dropship Subcontractor`, its name begins with `DSC/`, its source is the shared vendor location and its destination is `Subcontracting/A`.

**AC-536 A component bought for a customer keeps the customer operation type.**
Given a subcontracted product whose finished goods are drop-shipped to the customer.
When the purchase order for the finished product is confirmed.
Then its transfer uses the `Dropship` operation type, whose destination is the shared customer location, and not the `Dropship Subcontractor` one.

**AC-537 A portal subcontractor may record production on a drop-shipped component.**
Given a subcontractor `Sub` with portal access only, and a serial-tracked component drop-shipped straight to `Subcontracting/A`.
When `Sub` records the production through the portal.
Then the serial numbers are accepted and the component moves are completed, without `Sub` being granted any inventory right.

---

## 39. Inter-warehouse resupply details

**AC-538 The transit location depends on the companies (`RP-RULE-270`).**
Given `WH-A` and `WH-B` of the same company.
When `WH-B` is added to the resupply list of `WH-A`.
Then the route uses the company's internal transit location, and that location is activated.
Given instead `WH-A` of `Company A` and `WH-C` of `Company C`.
Then the route uses the shared inter-company transit location, and that location is activated.
Given instead two warehouses of one company whose internal transit location does not exist.
Then no route is created and no error is raised.

**AC-539 The output location of the supplying warehouse depends on its delivery steps (`RP-RULE-271`).**
Given `WH-B` delivering in one step.
Then the first rule of the resupply route starts at `WH-B/Stock`.
Given `WH-B` delivering in two steps.
Then it starts at `WH-B/Output`.

**AC-540 A one-step supplying warehouse gets an extra make-to-order rule (`RP-RULE-273`).**
Given `WH-B` delivering in one step and resupplying `WH-A`.
Then the global "Replenish on Order" route holds an extra rule from `WH-B/Stock` to the transit location with `WH-B`'s outgoing operation type and the name suffix `Make To Order`.

**AC-541 Moving back to one delivery step rewires the resupply routes (`RP-RULE-276`).**
Given `WH-B` delivering in two steps and resupplying `WH-A`, so the resupply route holds a "stock to Output" rule.
When `WH-B` is changed to deliver in one step.
Then that "stock to Output" rule is archived and a new make-to-order rule from `WH-B/Stock` to the transit location with the outgoing operation type and the name suffix `Make To Order` is created.

**AC-542 The company of a resupply route is the intersection (`RP-RULE-278`).**
Given `WH-A` and `WH-B` of `Company A`.
Then the resupply route's company is `Company A`.
Given instead `WH-A` of `Company A` and `WH-C` of `Company C`.
Then the resupply route has no company, which is what makes it visible to both.

---

## 40. Access, rounding and robustness

**AC-543 Any internal user may read routes and rules (`RP-RULE-293`).**
Given a salesperson with no inventory right at all.
When that salesperson confirms a sales order whose line must be procured.
Then rule selection reads the routes and the rules successfully and the procurement is created.

**AC-544 An inventory user may drive the wizards (`RP-RULE-294`).**
Given an inventory user who is not an administrator.
Then that user may create, read and change the Stock Rules Report wizard, the Product Replenish wizard and the Replenishment Option wizard, and may create, read, change and delete the Reordering Rule Snooze wizard.

**AC-545 Only an administrator opens the Replenishment Information dialog (`RP-RULE-295`).**
Given an inventory user who is not an administrator.
When that user presses the forecast-detail button on a reordering rule.
Then the dialog is refused by the shared access-rights check.

**AC-546 A purchase user may work on transfers and moves (`RP-RULE-296`, `RP-RULE-298`).**
Given a purchase user.
Then that user may read Locations, Warehouses, Reordering Rules and the Vendor Delay Report; may create, read, change and delete Transfers; and may create, read and change Stock Moves but not delete them.
Given a purchase manager.
Then that user may additionally delete Stock Moves.

**AC-547 Quantity comparisons use the unit rounding (`RP-RULE-302`).**
Given a unit whose rounding is 0.01 and a forecast of 9.999 against a minimum of 10.
When the reordering rule is evaluated.
Then the forecast rounds to 10.00, which is not lower than the minimum, and nothing is ordered.

**AC-548 A batch is written all or nothing (`RP-RULE-303`).**
Given a batch of three reordering rules whose third request fails.
When the batch is run.
Then none of the three documents exists after the failure, and re-running the reduced batch creates exactly one document per surviving rule.

**AC-549 Quantities carry the Product Unit precision (`RP-RULE-310`).**
Given the "Product Unit" precision set to two decimal places.
When a quantity to order of 7.126 is stored.
Then the stored value is 7.13.

**AC-550 The quantity to order is rounded up to the multiple (`RP-RULE-311`).**
Given a reordering rule for `P` whose replenishment multiple is `Dozens` of 12 units, a minimum of 0, a maximum of 0 and a shortage of 46 units.
When the rule is evaluated.
Then 46 ÷ 12 = 3.83 is rounded up to 4 dozens, that is 48 units.
Given instead no replenishment multiple and no fallback multiple.
Then the quantity to order stays 46 units.

**AC-551 A quantity in progress is converted without rounding (`RP-RULE-313`).**
Given a reordering rule in `Units` and a draft purchase order line of 1 pack of 6 units where only 4 units remain to be received.
When the quantity in progress is read.
Then 4 units are counted, not 6 and not 0.

**AC-552 The demand graph rounds its values (`RP-RULE-314`).**
Given a daily demand of 1.9333 units at a unit rounding of 0.01 and an ordering period of 81.6 days.
When the graph is built.
Then the daily demand is shown as 1.93 and the ordering period as 82 days.

**AC-553 A vendor with nothing ordered reports no data (`RP-RULE-315`).**
Given a vendor with no qualifying purchase order line in the window.
When the on-time delivery rate is read.
Then the value is −1 and the screen shows "no data" rather than 0 percent.

---

## 41. Multi-lingual procurement

**AC-554 A vendor's language decides the line name (`RP-RULE-108`, `RP-RULE-115`).**
Given product `P` whose name is `Chair` in English and `Chaise` in French, whose purchase description is `Solid oak` in English and `Chene massif` in French, and vendor `V` whose language is French.
When a need for 5 units of `P` is run through the Buy route.
Then the created purchase order line is named `Chaise` followed by a new line and `Chene massif`.

**AC-555 A second need in the vendor's language merges into the same line (`RP-RULE-108`).**
Given the line of `AC-554`.
When a second need for 5 units of `P` for the same vendor, same unit and same merge key is run.
Then the line name still matches the product display name in French followed by the description, the line is reused, and it holds 10 units.

**AC-556 A need whose description differs does not merge (`RP-RULE-108`).**
Given the line of `AC-554`.
When a need carrying `values.product_description_variants` = `Color: Red` is run for the same vendor.
Then the candidate line is rejected because its name does not match `Chaise` followed by `Color: Red`, and a second line is created.

**AC-557 Two reordering rules in different languages keep their own lines (`RP-RULE-108`).**
Given two reordering rules for `P`, one at `WH-A/Stock` and one at `WH-B/Stock`, and vendor `V` whose language is French.
When the scheduler runs both.
Then each rule produces its own purchase order line, because the candidate-line test keeps only lines whose `orderpoint` is that rule or is empty, and both lines are named in French.

---

## 42. The Vendor Delay Report

**AC-558 A receipt in another unit is converted into the reference unit (`entities.md` section 9).**
Given a purchase order line for 12 `Units` of `P` and a receipt recorded as 1 `Dozen`.
When the report is read grouped by product.
Then `quantity_total` is 12, `quantity_on_time` is 12 and the on-time delivery rate is 100 percent.

**AC-559 Several destination locations are summed (`entities.md` section 9).**
Given a purchase order line for 10 units received on time as 6 units into `WH/Stock/Shelf 1` and 4 units into `WH/Stock/Shelf 2`.
When the report is read grouped by product.
Then `quantity_total` is 10, `quantity_on_time` is 10 and the rate is 100 percent.

**AC-560 A backorder starts at a partial rate and completes it (`entities.md` section 9).**
Given a purchase order line for 10 units received on time as 6 units with a backorder.
When the report is read.
Then `quantity_total` is 10, `quantity_on_time` is 6 and the rate is 60 percent.
When the backorder of 4 units is received on time as well.
Then `quantity_on_time` is 10 and the rate is 100 percent.

**AC-561 Cancelling the backorder leaves the rate partial (`entities.md` section 9).**
Given a purchase order with two lines of 10 units each, one product carrying a category and one carrying none, both received as 6 units with the backorder cancelled.
When the report is read grouped by product.
Then two rows are returned, each with `quantity_total` 10, `quantity_on_time` 6 and a rate of 60 percent, and the product without a category is present in the result.

**AC-562 A duplicated receipt completes the rate (`entities.md` section 9).**
Given a purchase order line for 10 units received as 6 units with the backorder cancelled, then a copy of that receipt validated for the remaining 4 units on time.
When the report is read.
Then `quantity_total` is 10, `quantity_on_time` is 10 and the rate is 100 percent.

**AC-563 A partially cancelled order still reports (`calculations.md` section 22.2).**
Given a purchase order line for 10 units of which the receipt of 6 units is completed on time and the remaining move is cancelled.
When the report is read.
Then `quantity_total` is 10, `quantity_on_time` is 6 and the rate is 60 percent, because the cancelled move contributes zero and does not reduce the ordered quantity.

---

## 43. The purchase analysis view

**AC-564 The effective days to arrival counts from the order date (`RP-RULE-351`, `calculations.md` section 30).**
Given a purchase order dated ten days ago for 1 unit, whose line is planned five days from now.
When the order is confirmed and its receipt is validated today.
Then the average of `days_to_arrival` for that order is 10.

**AC-565 An unreceived order reports its planned distance (`calculations.md` section 30).**
Given a purchase order dated 1 March at 08:00 whose line is planned for 11 March at 08:00, with nothing received.
When the view is read for that order.
Then `days_to_arrival` is 10.00.

**AC-566 A return to the vendor does not move the effective date (`RP-RULE-350`).**
Given an order received on 6 March at 20:00 and ordered on 1 March at 08:00, so `days_to_arrival` is 5.50.
When part of the goods is returned to the vendor on 12 March.
Then `effective_date` is still 6 March at 20:00 and `days_to_arrival` is still 5.50.

**AC-567 The warehouse column is empty for a drop shipping order (`RP-RULE-352`).**
Given one ordinary purchase order whose operation type belongs to `Main` and one drop shipping purchase order whose operation type has no warehouse.
When the view is grouped by warehouse.
Then the first order is grouped under `Main` and the second under "None".

**AC-568 Two arrival days are not merged into one row (`RP-RULE-353`).**
Given an order whose two lines arrived on different days.
When the view is read grouped by order.
Then the two lines stay on separate rows, because the earliest completion date is part of the grouping.

**AC-569 The rate is an unweighted average of the rows (`RP-RULE-351`).**
Given three rows whose `days_to_arrival` values are 5.50, 10.00 and 3.00, for quantities of 100, 1 and 1.
When the average is read.
Then it is 6.17, not a quantity-weighted value.

**AC-570 The view is read-only (`RP-RULE-354`).**
Given a user with every access group of this domain.
When that user tries to write a value on a row of the purchase analysis view.
Then the write is refused, and the value changes only when the underlying purchase order line or its transfers change.

---

## 44. Returning goods to a vendor

**AC-571 The Return button appears only on a receipt of a purchase order (`RP-RULE-338`).**
Given a completed receipt created by a purchase order and a completed internal transfer created by hand.
Then the Return button is shown on the receipt and hidden on the internal transfer.

**AC-572 A return to the vendor reduces the received quantity (`RP-RULE-332`, `RP-RULE-333`, `RP-RULE-334`).**
Given a purchase order line for 10 units whose receipt has been validated for 10, so the received quantity is 10.
When 2 units are returned to the vendor with "Update Quantities on Purchase Order" ticked and the return is validated.
Then the received quantity is 8.

**AC-573 A return that has not been validated changes nothing (`RP-RULE-335`).**
Given the return of `AC-572` created but not validated.
Then the received quantity is still 10, because the return move is not completed and its demand quantity is not counted on a not-yet-completed purchase return.

**AC-574 Unticking the flag leaves the received quantity alone (`RP-RULE-336`).**
Given the line of `AC-572` at a received quantity of 10.
When 2 units are returned to the vendor with the flag unticked and the return is validated.
Then the received quantity is still 10.

**AC-575 Receiving returned goods again without the flag keeps the quantity (`RP-RULE-334`, `RP-RULE-336`).**
Given the line of `AC-572` at a received quantity of 8 after a return with the flag ticked.
When those 2 units are received again through a return of that return with the flag unticked.
Then the received quantity is still 8.
Given instead the flag ticked on that second return.
Then the received quantity returns to 10.

**AC-576 A multi-step return stamps the purchase order line on the pushed move (`RP-RULE-330`, `RP-RULE-331`, `calculations.md` section 31).**
Given a warehouse receiving in three steps, an extra push rule from an internal location `Vendor returns processing` to the shared vendor location, and a purchase order line for 10 units fully received, so the received quantity is 10.
When 2 units are returned from the storage transfer into `Vendor returns processing` and that return is validated.
Then the received quantity is still 10.
When the pushed transfer from `Vendor returns processing` to the vendor location is validated.
Then the received quantity is 8 and that transfer's counterparty is the vendor of the purchase order.

**AC-577 A return transfer takes the vendor as counterparty (`RP-RULE-331`).**
Given a receipt whose counterparty is empty and a return of it straight to the vendor location.
When the return is created.
Then every move of the return resolves to the same vendor, and the return transfer's counterparty is rewritten to that vendor.
Given instead a return whose moves resolve to two different counterparties.
Then the transfer keeps the counterparty it was created with.

**AC-578 Changing the return operation type to a delivery restores the received quantity (`RP-RULE-337`).**
Given 1 unit bought, received and returned to the vendor, so the received quantity is 0.
When the return's operation type is changed to a delivery whose default destination location is the shared customer location, before validation.
Then the return's destination is no longer a vendor location, the move is no longer a purchase return, and the received quantity is 1.

**AC-579 "Return all" proposes the net received quantity (`RP-RULE-341`).**
Given a receipt of 10 units of which 3 have already been returned.
When the user presses "Return all".
Then the line is filled with 7 units.
When the user instead types 9 units by hand.
Then the wizard refuses with `You cannot return more than what has been received.`

**AC-580 A return needs a non-zero quantity (`workflows.md` section 25.1).**
Given the Return dialog on a completed receipt with every line still at zero.
When the user presses "Return".
Then the operation is refused with `Please specify at least one non-zero quantity.`

**AC-581 Only a completed transfer may be returned (`workflows.md` section 25.1).**
Given a receipt that is still in state `assigned`.
When a user opens the Return dialog on it.
Then the dialog is refused with `You may only return Done pickings.`

**AC-582 Only one transfer may be returned at a time (`workflows.md` section 25.1).**
Given two completed receipts selected together.
When the Return dialog is opened.
Then it is refused with `You may only return one picking at a time.`

**AC-583 An exchange of a receipt shows three transfers (`RP-RULE-339`).**
Given a purchase order line for 10 units whose receipt has been validated.
When the user presses "Exchange" for 10 units.
Then a return transfer and an exchange transfer are created, the purchase order shows three transfers in total, the exchange moves carry no originating returned move, and after both are validated the received quantity is 10.

**AC-584 An exchange may start from a return that has no origin (`RP-RULE-340`).**
Given a return transfer created by hand, with no originating transfer.
When the user presses "Exchange" on it.
Then a new transfer is built from that return's own operation type, source location and destination location, no purchase order line is attached to its moves, and no received quantity changes.
