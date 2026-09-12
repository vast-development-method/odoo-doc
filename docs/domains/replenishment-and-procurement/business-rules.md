# Business rules

Every validation, guard, permission, company and currency consistency rule, locking rule, uniqueness rule, rounding rule and date rule of the Replenishment and Procurement domain is catalogued below, with the exact message each one shows. Rules are numbered with the prefix `RP-RULE-` so that the other documents of this folder and of sibling domains can cite them. A rule that is an industry-standard completion (that is, a rule that fills a gap the reference behavior leaves implicit) is marked **Industry-standard completion** in its statement.

Notation used throughout:

- "compare *a* with *b* in unit *u*" means: round both values to the rounding step of unit *u* and compare the rounded values. Quantities are never compared with exact equality.
- "the request" means one procurement request as specified in `entities.md`, section 10.
- "elevated rights" means the operation runs under the superuser account, bypassing access rights and record rules.
- A message shown between backticks is reproduced verbatim; placeholders are written between angle brackets.

---

## 1. Routes

| Rule | Statement | Message on violation |
|---|---|---|
| `RP-RULE-001` | A Route may have an empty company. An empty company means the route is shared by every company and is visible to every user. | none |
| `RP-RULE-002` | When a Route has a company, every Stock Rule of that route must have exactly that company. The check runs whenever the route's `company` is written and whenever a rule's `company` is written. | `Rule <rule name> belongs to <rule company> while the route belongs to <route company>.` |
| `RP-RULE-003` | Changing the `company` of a Route removes from `warehouses` every warehouse that does not belong to the new company. This happens while the user edits the form, before saving. | none |
| `RP-RULE-004` | The warehouses that may be selected in `warehouses` are the warehouses of the route `company`, or every warehouse when the route has no company. | none |
| `RP-RULE-005` | Setting `warehouse_selectable` to false empties `warehouses`. | none |
| `RP-RULE-006` | A Route is offered for selection on a product only when `product_selectable` is true, on a product category only when `product_category_selectable` is true, on a warehouse only when `warehouse_selectable` is true, on a package type only when `package_type_selectable` is true, on a sales order line only when `sale_selectable` is true, and on a shipping method only when `shipping_selectable` is true. | none |
| `RP-RULE-007` | Routes are ordered by `sequence` ascending. The sequence is the primary tie-breaker of rule selection (see `RP-RULE-062`). | none |
| `RP-RULE-008` | Route names are not unique. Two routes may carry the same name; they are distinguished by their surrogate key. | none |
| `RP-RULE-009` | Duplicating a Route duplicates its rules, appends " (copy)" to the route name and to each rule name, and clears `products`, `product_categories`, `warehouses`, `supplied_warehouse` and `supplier_warehouse`. | none |
| `RP-RULE-010` | Deleting a Route deletes every Stock Rule of that route (cascade). | none |
| `RP-RULE-011` | Archiving a Route archives every Stock Rule of that route whose destination location is still active. Rules whose destination location is already archived are left untouched, because archiving the location already archived them. | none |
| `RP-RULE-012` | Unarchiving a Route unarchives the rules that were archived with it, under the same destination-location condition. | none |
| `RP-RULE-013` | A Route is a valid resupply route for a product when: it contains a rule with action `buy` and the product has at least one Vendor Price; or it contains no `buy` rule but contains a rule with action `manufacture` and the product has at least one bill of materials of type `normal`. In every other case the answer is false. The test is evaluated in that exact order: a route containing both a `buy` rule and a `manufacture` rule is judged on its `buy` rule only. | none |

---

## 2. Stock Rules

| Rule | Statement | Message on violation |
|---|---|---|
| `RP-RULE-020` | `action`, `name`, `destination_location`, `route`, `procure_method`, `operation_type` and `auto` are mandatory. A rule cannot be saved without them. | The shared "required field" message of the platform. |
| `RP-RULE-021` | A rule whose `company` differs from the company of its `route` is refused, when the route has a company. | `Rule <rule name> belongs to <rule company> while the route belongs to <route company>.` |
| `RP-RULE-022` | The `company` of a newly created rule defaults to the active company when nothing else is supplied. | none |
| `RP-RULE-023` | `destination_location`, `location_source`, `operation_type`, `partner_address` and `warehouse` must all belong to the rule's company or to no company. | The shared company-consistency message of the platform. |
| `RP-RULE-024` | The operation types offered for `operation_type` depend on `action`: for `pull`, `push` and `pull_push`, every operation type; for `buy`, operation types whose code is `incoming`, plus `dropship` when the Drop Shipping capability package is installed; for `manufacture`, operation types whose code is `manufacturing`. | The shared "value not allowed" message of the platform. |
| `RP-RULE-025` | Setting `action` to `buy` in a form empties `location_source`. A `buy` rule has no source location: the source of the receipt it eventually produces is the vendor location of the vendor chosen at run time. | none |
| `RP-RULE-026` | Choosing an `operation_type` in a form sets `location_source` to the operation type's default source location and `destination_location` to the operation type's default destination location. | none |
| `RP-RULE-027` | Choosing a `route` in a form sets `company` to the route's company when the route has one, and empties `operation_type` when the warehouse of the currently selected operation type belongs to a different company than the route. | none |
| `RP-RULE-028` | A rule with `action` equal to `pull` or `pull_push` and no `location_source` cannot fulfil a need. The failure is detected at run time, not at save time. | `No source location defined on stock rule: <rule name>!` |
| `RP-RULE-029` | `route_sequence` is a stored copy of the sequence of the rule's route and is kept in step with it automatically. It exists so that candidate rules can be ordered without reading the route. | none |
| `RP-RULE-030` | Rules are ordered by `sequence` ascending, then by surrogate key ascending. | none |
| `RP-RULE-031` | Duplicating a rule appends " (copy)" to its name and copies every other field unchanged. | none |
| `RP-RULE-032` | `location_destination_from_rule` controls where the created move lands: when true, the move's destination location is the rule's `destination_location`; when false, the move's destination location comes from the operation type's default destination location and the rule's `destination_location` is written on the move as its final location instead. | none |
| `RP-RULE-033` | A push rule whose `push_condition` is not empty applies only to moves that satisfy that condition. A move that does not satisfy it causes the search to continue with the next candidate push rule (see `RP-RULE-131`). | none |
| `RP-RULE-034` | The last rule of a generated rule chain that requests cancellation propagation always has `propagate_cancel` forced to false, so that cancelling the first step of a receipt chain never cascades past the end of the chain into an unrelated delivery. | none |
| `RP-RULE-035` | When a warehouse configuration needs a rule that already exists in archived form with the same `operation_type`, `location_source`, `destination_location`, `route` and `action`, that archived rule is unarchived instead of a duplicate being created. | none |
| `RP-RULE-036` | **Industry-standard completion.** A rule whose `location_source` equals its `destination_location` and whose `procure_method` is `make_to_order` produces an endless supply loop. The loop is not blocked at save time; it is detected when a rule chain is walked (see `RP-RULE-160`). A replacement should apply the same late detection so that partially built configurations remain editable. | `Invalid rule's configuration, the following rule causes an endless loop: <rule display name>` |

---

## 3. Reordering Rules

| Rule | Statement | Message on violation |
|---|---|---|
| `RP-RULE-040` | At most one Reordering Rule may exist for a given triple (`product`, `source_location`, `company`). The uniqueness is enforced at database level and counts archived rules as well: an archived rule for the same product and location blocks the creation of a new one. | `A replenishment rule already exists for this product on this location.` |
| `RP-RULE-041` | `product_minimum_quantity` must be lower than or equal to `product_maximum_quantity`. Checked whenever either field is written. | `The minimum quantity must be less than or equal to the maximum quantity.` |
| `RP-RULE-042` | A rule may not be created for a product that has a bill of materials of type "kit". Checked whenever `product` is written. | `A product with a kit-type bill of materials can not have a reordering rule.` |
| `RP-RULE-043` | A record may not be created with a non-empty `snoozed_until` while `trigger` is `auto`. The check also fires when `trigger` is not supplied at all, because it then defaults to `auto`. | `You can not create a snoozed orderpoint that is not manually triggered.` |
| `RP-RULE-044` | `snoozed_until` may not be written on any rule whose `trigger` is `auto`, including a batch write where only some of the rules are automatic. | `You can only snooze manual orderpoints. You should rather archive 'auto-trigger' orderpoints if you do not want them to be triggered.` |
| `RP-RULE-045` | `company` may never be changed after creation. Writing a different value is refused. | `Changing the company of this record is forbidden at this point, you should rather archive it and create a new one.` |
| `RP-RULE-046` | When `warehouse` must be derived and no warehouse exists for the company, the operation is refused with the shared "no warehouse configured" redirect warning of `../inventory-operations/`: an inventory administrator is redirected to the warehouse list, a user who is not an inventory administrator is told to contact the administrator. | `Please create a warehouse for company <company name>.` with the button `Go to Warehouses`, or `Please contact your administrator to configure your warehouse.` |
| `RP-RULE-047` | `product` is restricted to inventory-tracked goods products. When the form is opened from a product template, only variants of that template may be chosen; when the opening context names a default product, only that product may be chosen. | The shared "value not allowed" message of the platform. |
| `RP-RULE-048` | `source_location` is restricted to internal and view locations that either belong to the rule's warehouse or belong to no warehouse at all, and that belong to the rule's company or to no company. | The shared "value not allowed" message of the platform. |
| `RP-RULE-049` | Whenever `product_minimum_quantity` changes, `product_maximum_quantity` is raised to the new minimum when it is empty or lower than the new minimum. This keeps `RP-RULE-041` satisfied without the user having to act. | none |
| `RP-RULE-050` | `route` is restricted to routes that are selectable on products, or that contain at least one rule with action `buy` or `manufacture`. | The shared "value not allowed" message of the platform. |
| `RP-RULE-051` | Writing an empty `route` also empties `vendor_price`. A vendor price without a buy route would never be used. | none |
| `RP-RULE-052` | Writing a non-empty `vendor_price` while `route` is empty sets `route` to the route of the first rule with action `buy` that belongs to the rule's company or to no company. | none |
| `RP-RULE-053` | Writing a non-empty `bill_of_materials` while `route` is empty sets `route` to the route of the first rule with action `manufacture`. | none |
| `RP-RULE-054` | `vendor_price` is restricted to vendor prices of the rule's product or of its product template, and to the rule's company. | The shared "value not allowed" message of the platform. |
| `RP-RULE-055` | `bill_of_materials` is restricted to bills of type `normal`, of the rule's company or of no company, and for the rule's product or its product template. | The shared "value not allowed" message of the platform. |
| `RP-RULE-056` | `quantity_to_order` is a derived, writable value. Writing it on a rule whose `trigger` is `auto` resets `quantity_to_order_manual` to zero: an automatic rule may never carry a manual override. | none |
| `RP-RULE-057` | Writing `quantity_to_order` on a manual rule sets `quantity_to_order_manual` to the written value when that value differs from `quantity_to_order_computed`; when both the current manual override and the written value are empty, the field falls back to `quantity_to_order_computed`. | none |
| `RP-RULE-058` | Archiving a product archives every Reordering Rule of that product; unarchiving the product unarchives them. | none |
| `RP-RULE-059` | A Reordering Rule created by the superuser with `trigger` equal to `manual` and a `quantity_to_order` that is zero or negative is deleted automatically: by the periodic cleanup task, and every time the replenishment report is opened. These are the temporary rules that the replenishment report itself created. | none |
| `RP-RULE-060` | `name` is read-only and is taken from the sequence with code `reordering_rule` (prefix `OP/`, five digits, first value `OP/00001`, increment 1, shared by every company). Rules created by the replenishment report are named `Replenishment Report` instead of consuming a sequence value. | none |

---

## 4. Rule selection

| Rule | Statement | Message on violation |
|---|---|---|
| `RP-RULE-061` | Rule selection for a need only ever considers rules whose `action` is not `push`, and whose `destination_location` is the requested location or one of its ancestors. | none |
| `RP-RULE-062` | Inside one candidate location, the sources of candidate routes are tried in this exact order, stopping at the first source that yields a rule: (1) the routes carried by the request in `routes`; (2) the routes of the package type of the request's packaging unit; (3) the routes of the product unioned with the total routes of the product's category (which include the routes inherited from ancestor categories); (4) the routes of the warehouse. | none |
| `RP-RULE-063` | Inside one source, routes are sorted by: first the routes that are the product's own routes, then the other routes; and inside each of those two blocks, by route `sequence` ascending. The first route that has a rule for the candidate location wins. | none |
| `RP-RULE-064` | When several rules of the same route target the same candidate location and warehouse, the rule with the lowest (`route_sequence`, `sequence`) pair wins. | none |
| `RP-RULE-065` | When a warehouse is known, the rule chosen inside a route is the one whose `warehouse` equals that warehouse; when there is none, the rule whose `warehouse` is empty is used. When no warehouse is known, the first rule of the route in stored order is used. | none |
| `RP-RULE-066` | A candidate rule is discarded when its `warehouse` is set and differs from the warehouse of the need. A rule with an empty `warehouse` always passes this test. | none |
| `RP-RULE-067` | When the search runs with elevated rights and the request carries a company, only rules with no company or with a company that is the request company or a descendant of it are considered; the companies of the routes carried by the request are added to that set. When the search runs with a normal user's rights, the record rules already restrict the result and no extra narrowing is applied. | none |
| `RP-RULE-068` | A warehouse route containing a rule with action `buy` is considered only when the product has at least one Vendor Price. A warehouse route containing a rule with action `manufacture` is considered only when the product has at least one bill of materials of type `normal`. Every other warehouse route is always considered. This filter applies to warehouse routes only, never to routes carried by the request, by the product or by the category. | none |
| `RP-RULE-069` | When the location chain contains the shared inter-company transit location, the shared customer location is added to the set of acceptable destinations, and the shared customer location is examined as a candidate location in the same step as the inter-company location, immediately after it. This avoids having to duplicate every customer-delivery rule for inter-company flows. | none |
| `RP-RULE-070` | When a request carries a sales order line and a company, and the Drop Shipping capability package is installed, candidate rules are further restricted to rules of exactly that company (not of its descendants). | none |
| `RP-RULE-071` | Rule selection for an arrival (push) only ever considers rules whose `action` is `push` or `pull_push` and whose `location_source` is the arrival location or one of its ancestors. It uses the same four-source order as `RP-RULE-062`, but reads only the single best-ordered rule of each source and does not apply the warehouse route filter of `RP-RULE-068`. | none |
| `RP-RULE-072` | When no rule can be found for a need, the request fails. | `No rule has been found to replenish "<product display name>" in "<location display name>".` followed by a new line and `Verify the routes configuration on the product.` |

---

## 5. Running procurement requests

| Rule | Statement | Message on violation |
|---|---|---|
| `RP-RULE-073` | Before anything else, the run operation applies three defaults to every request: `values.company` becomes the company of the request location when absent; `values.priority` becomes `"0"` when absent; `values.date_planned` becomes the current moment when absent or empty. | none |
| `RP-RULE-074` | A request is skipped silently, creating nothing and reporting nothing, when the request product is not a goods product, or when the requested quantity is zero at the rounding of the request unit. | none |
| `RP-RULE-075` | When the Manufacturing capability package is installed, a request whose product has a bill of materials of type "kit" is replaced, before rule selection, by one request per exploded component. The replacement requests keep the location, name, origin, company and values of the original request. | none |
| `RP-RULE-076` | When the Purchase Inventory capability package is installed and a request carries a route that contains a rule with action `buy`, the reception route of every warehouse of the request company is added to the request's `routes` before rule selection, so that the receipt steps of the warehouse are available to the chain the buy rule starts. | none |
| `RP-RULE-077` | Rule selection failures are collected for the whole batch and raised together before any document is created. No request of the batch produces a document when at least one request cannot find a rule. | see `RP-RULE-072` |
| `RP-RULE-078` | Action handlers run in the order the actions were first encountered while classifying the requests. `pull_push` is handled by the pull handler. When one action handler fails, the remaining handlers still run, and the collected failures are raised after all of them have run. | the joined messages |
| `RP-RULE-079` | When the run operation is asked to raise user-facing errors (the default), the collected messages are joined with new lines and raised as one error, which aborts the surrounding transaction. When it is asked not to, a procurement exception carrying the list of (request, message) pairs is raised instead, so that the caller can isolate the failing requests and continue with the others. | the joined messages |

---

## 6. The pull action

| Rule | Statement | Message on violation |
|---|---|---|
| `RP-RULE-080` | Requests whose quantity is negative or zero are processed before requests whose quantity is positive, so that a return can merge with its sibling. | none |
| `RP-RULE-081` | The supply method written on the created move is the rule's `procure_method`, except that `mts_else_mto` (make to stock, else make to order) is written as `make_to_stock`. The split between the part taken from stock and the part that triggers another rule has already happened before the request was built. | none |
| `RP-RULE-082` | The company of the created move is the first non-empty of: the rule company, the company of the rule's source location, the company of the rule's destination location, the request company. | none |
| `RP-RULE-083` | The partner of the created move is the rule's `partner_address` when set, otherwise `values.partner`. | none |
| `RP-RULE-084` | When the request has downstream moves and the request location is the company's internal transit location: when no partner has been determined and every downstream move's destination warehouse resolves to exactly one partner, that partner becomes the new move's partner; and the downstream moves receive as partner the partner of the warehouse of the rule's source location, or, when that warehouse has none, the partner of the rule's company. | none |
| `RP-RULE-085` | When the requested quantity is negative at the rounding of the request unit, `values.to_refund` is set to true before the values are serialized onto the move. | none |
| `RP-RULE-086` | Moves are created with elevated rights and in the company context of the move, because the user who triggered the need may have no right to create stock moves (for example a salesperson or a portal user). | none |
| `RP-RULE-087` | Every created move is confirmed immediately. Confirmation is what chains the next step of the route. | none |

---

## 7. The buy action

| Rule | Statement | Message on violation |
|---|---|---|
| `RP-RULE-090` | The company used for a buy request is the rule company when set, otherwise the request company. | none |
| `RP-RULE-091` | Vendor selection order: (1) `values.forced_vendor_price` when set; (2) the `vendor_price` of `values.orderpoint` when both are set; (3) the shared vendor-selection operation of `../pricing-and-pricelists/`; (4) as a last resort, the first Vendor Price of the product that belongs to this company or to no company, regardless of price, minimum quantity or validity dates. | none |
| `RP-RULE-092` | The contact restriction passed to vendor selection is `values.vendor_contact`, or `values.partner` when `values.force_unit_of_measure` is true, otherwise no restriction. For a rule that belongs to the global Dropship route the restriction is always removed, because the partner carried by such a request is the customer, not a vendor. | none |
| `RP-RULE-093` | The date passed to vendor selection is the later of the date part of `values.date_planned` and today; when the request carries no planned date, no date is passed. | none |
| `RP-RULE-094` | When no Vendor Price at all can be found and the request came from a reordering rule, the request fails. | `There is no matching vendor price to generate the purchase order for product <product display name> (no vendor defined, minimum quantity not reached, dates not valid, ...). Go on the product form and complete the list of vendors.` |
| `RP-RULE-095` | When no Vendor Price can be found and the request did not come from a reordering rule, nothing fails. Instead: the downstream moves whose `propagate_cancel` is true are cancelled; all downstream moves have their supply method set to `make_to_stock`; the responsible person is notified; and no purchase order is created for that request. | the notification of `RP-RULE-096` |
| `RP-RULE-096` | The vendor-not-found notification is a message posted on the originating document, addressed to the users to notify. Its body is the mention of each user, then a line break, then `No supplier has been found to replenish`, then the product display name in bold, then `this product should be manually replenished.` The users notified are the salesperson of the originating sales order, or the responsible person of the originating manufacturing order. When the need came from neither, nothing is posted. | none |
| `RP-RULE-097` | Two requests may share a draft purchase order only when every component of the grouping key matches: the vendor contact, `state` equal to `draft`, the rule's operation type, the company, the buyer of the vendor contact, and the order currency (the currency of the chosen Vendor Price, else the vendor's purchase currency, else the company currency). | none |
| `RP-RULE-098` | When the vendor's grouping mode is `default` ("On Order"), or the rule's operation type has code `dropship`, the grouping key additionally requires the order's `references` to contain one of the request's `references`; when the request carries no reference and the grouping mode is `default`, the key instead requires the order to have no reference at all. A need with no reference therefore never joins an order that has one. | none |
| `RP-RULE-099` | When the vendor's grouping mode is `all` ("Always"), the grouping key carries neither a reference component nor a date window: every need for that vendor, company, operation type, buyer and currency joins the same draft order. | none |
| `RP-RULE-100` | When the vendor's grouping mode is `day` ("Daily"), the order's `date_planned` must fall within the calendar day of the request's planned date, from the start of that day to the end of that day. | none |
| `RP-RULE-101` | When the vendor's grouping mode is `week` ("Weekly") and the vendor's `grouping_weekday` is `default` ("Expected Date"), the order's `date_planned` must fall in the window running from the start of the day *n* days before the request's planned date to the end of the day (6 − *n*) days after it, where *n* is the weekday number of the planned date with Monday = 1 and Sunday = 7. | none |
| `RP-RULE-102` | When the vendor's grouping mode is `week` and `grouping_weekday` names a weekday, the target day is the planned date shifted forward by `(7 + target weekday − planned weekday) modulo 7` days, and the order's `date_planned` must fall inside that single day. | none |
| `RP-RULE-103` | When no purchase order matches the grouping key, an order is created only from the requests whose quantity is not negative. A group made only of negative requests creates no order at all. | none |
| `RP-RULE-104` | A new purchase order is created with elevated rights under the superuser account, so that the user who triggered the need does not become a follower of an order they may have no right to see. | none |
| `RP-RULE-105` | When a matching purchase order already exists, its `references` gain every reference carried by the requests, and its `origin` gains every request origin that is not already present, comma separated; when the order had no origin, the collected origins joined with ", " become the origin. | none |
| `RP-RULE-106` | Two requests may share one purchase order line only when every component of the line merge key matches: the product; the request unit; `values.propagate_cancel`; `values.product_description_variants`; `values.orderpoint` when `values.move_destinations` is empty and the empty value otherwise; and, when the Drop Shipping capability package is installed, `values.sales_order_line`. | none |
| `RP-RULE-107` | When merging requests into one, the quantities are summed, the downstream moves are unioned, the reordering rule is the first non-empty one found, and every other value is taken from an arbitrary member of the group. | none |
| `RP-RULE-108` | A candidate existing line is searched only among the lines of the order for the same product that are not section or note lines, and must satisfy all of: the line's `propagate_cancel` equals `values.propagate_cancel`; when the request carries a reordering rule, has no downstream moves, and that rule is not a temporary rule, the line's `orderpoint` is that rule or is empty; when `values.force_unit_of_measure` is true, the line's unit equals the request unit; when `values.product_description_variants` is set, the line name matches the product display name in the vendor's language followed by a new line and the description, or the line name is exactly the product display name while the description equals the product name. Among the survivors, the first when sorted by `orderpoint` (lines with no reordering rule first) is taken. | none |
| `RP-RULE-109` | A temporary reordering rule (one created by the superuser with `trigger` equal to `manual`) never restricts the candidate line search, so that successive presses of Order on the replenishment report reuse the same line. | none |
| `RP-RULE-110` | A new purchase order line is never created with a quantity that is zero or negative. A merged request whose quantity is not strictly positive and that finds no candidate line is dropped. | none |
| `RP-RULE-111` | When an existing line is updated, its new quantity is the existing quantity plus the request quantity converted into the line's unit with half-up rounding, and vendor selection is run again for that new total, which may select a different price bracket. | none |
| `RP-RULE-112` | The unit price written on an updated line is the price of the newly selected Vendor Price, corrected for tax inclusion against the product's vendor taxes and the line's taxes in the order company, and converted from the Vendor Price currency into the order currency at today's rate when the two currencies differ. When no Vendor Price was found, the line's current unit price is kept unchanged. | none |
| `RP-RULE-113` | When the selected Vendor Price uses a different unit than the line and `values.force_unit_of_measure` is not true, the whole new line quantity is converted into the vendor's unit with half-up rounding and both the converted quantity and the vendor unit are written on the line. | none |
| `RP-RULE-114` | When `values.force_unit_of_measure` is not true and the vendor's unit differs from the request unit, a newly created line is expressed in the vendor's unit. When `values.force_unit_of_measure` is true, the request unit is kept. | none |
| `RP-RULE-115` | When `values.product_description_variants` is set and differs from the product name, the created line's name gains a new line and that description. | none |
| `RP-RULE-116` | The order date of a newly created purchase order is the minimum, over its positive requests, of `values.date_order` when set, otherwise `values.date_planned` minus the chosen vendor's lead time in calendar days. | none |
| `RP-RULE-117` | When a new line is added to an existing order and the order date computed from the line (the order's planned date, or the minimum planned date of the new lines, minus the vendor lead time) has an earlier date part than the order's current `date_order`, the order's `date_order` is moved back to that earlier value. The order date is never moved forward by this rule. | none |
| `RP-RULE-118` | When the vendor's grouping mode is `week` and `grouping_weekday` names a weekday, a newly created line's `date_planned` is shifted forward by `(7 + target weekday − planned weekday) modulo 7` days; and when the order has no `date_planned` yet, or its `date_planned` is not earlier than the shifted line date, the order's `date_order` is shifted forward by the same number of days, so that the interval between the order deadline and the expected arrival is preserved. | none |

---

## 8. The manufacture action

| Rule | Statement | Message on violation |
|---|---|---|
| `RP-RULE-120` | A request whose quantity is zero or negative at the rounding of the request unit creates no manufacturing order and reports nothing. | none |
| `RP-RULE-121` | The bill of materials is chosen in this order: `values.bill_of_materials`; the `bill_of_materials` of `values.orderpoint`; the best matching bill of type `normal` for the product, the rule's operation type and the company; the best matching bill of type `normal` for the product and the company, ignoring the operation type. | none |
| `RP-RULE-122` | An existing manufacturing order is extended instead of a new one being created only when the request origin is not the master production schedule marker, the bill does not enable batch sizes, and an order exists that matches exactly on: the bill of materials, the product, `state` in (`draft`, `confirmed`), not planned, the rule's operation type, the request company, no responsible user, and `references` equal to `values.references`. | none |
| `RP-RULE-123` | When `values.production_group` is set, a candidate manufacturing order must additionally have that production group among the ancestors of its own production group. | none |
| `RP-RULE-124` | When `values.orderpoint` is set, a candidate manufacturing order must additionally be either a draft whose deadline is not later than the procurement date, or a confirmed order whose start date is not later than the procurement date, where the procurement date is the end of the day `values.date_planned` minus the bill's manufacturing lead time. | none |
| `RP-RULE-125` | When the bill enables batch sizes, one manufacturing order is created per batch until the remaining quantity is not greater than zero, whatever candidate orders exist. | none |
| `RP-RULE-126` | The start date of a created manufacturing order is `values.date_planned` minus the bill's manufacturing lead time in calendar days; when that subtraction leaves the date unchanged because the lead time is zero, one hour is subtracted instead, so that production always starts strictly before the need. | none |
| `RP-RULE-127` | A created manufacturing order is confirmed immediately when it has no component moves and no work orders and (it came from a reordering rule or its downstream moves take from stock), or when it has component moves and did not come from a reordering rule. Otherwise it stays a draft until the whole reordering-rule batch has run. | none |

---

## 9. Push application

| Rule | Statement | Message on violation |
|---|---|---|
| `RP-RULE-130` | A move is never pushed further when it is an inventory adjustment move, or when it already has a downstream move whose source location equals the move's destination location or is an ancestor or a descendant of it. The second condition prevents creating a duplicate step when a pull rule has already built the chain. | none |
| `RP-RULE-131` | When the found push rule has a non-empty `push_condition` that the move does not satisfy, that rule is added to an excluded set and the search runs again excluding it. The loop stops at the first rule with no condition, at the first rule whose condition the move satisfies, or when no rule at all is found. | none |
| `RP-RULE-132` | A move that is a return (it has an origin returned move) is not pushed when the found rule's destination location equals the destination location of that origin returned move. Pushing would send the returned goods straight back where they came from. | none |
| `RP-RULE-133` | A transparent push rule ("Automatic No Step Added") rewrites the move: its scheduled date gains the rule lead time in calendar days and its destination location becomes the rule's `destination_location`; the move lines, when there are any, follow to the putaway location computed for the new destination and product, or to the new destination when there is no putaway rule. | none |
| `RP-RULE-134` | A transparent push rule re-runs push application on the same move only when the destination location actually changed. This is what stops an endless loop on a badly configured rule. | none |
| `RP-RULE-135` | A manual push rule ("Manual Operation") creates a copy of the move for the quantity actually completed on the source move, except that when the source move's demand quantity is negative, the source move's demand quantity is used instead. | none |
| `RP-RULE-136` | The destination location of the move created by a manual push rule is the rule's `destination_location`, replaced by the source move's final location when that final location is a descendant of the rule's destination location, and replaced by the customer location of the source move's partner when the source move has a partner and the rule's destination location is a customer location. | none |
| `RP-RULE-137` | The move created by a manual push rule carries the source move's final location only when the source move's destination location is not a descendant of that final location; otherwise its final location is left empty. | none |
| `RP-RULE-138` | The move created by a manual push rule gets supply method `make_to_order`, and is switched to `make_to_stock` when its source location bypasses reservation (a vendor, customer, production, inventory-loss or transit location). It is linked as a downstream move of the source move only when its source location does not bypass reservation. | none |
| `RP-RULE-139` | After a push, every downstream move of the original move that is not the newly created move is rewired: when a new move was created, the original move has a final location and the downstream move's source location equals that final location, the downstream move is detached from the original move and attached to the new move; otherwise, when the downstream move's source location is neither the destination location of the original move nor a descendant of it, the make-to-order link is broken. | none |

---

## 10. Confirming a move and creating the supply need

| Rule | Statement | Message on violation |
|---|---|---|
| `RP-RULE-140` | A draft move that already has origin moves becomes *waiting* and creates no need. A draft move with supply method `make_to_order` becomes *waiting* and creates a need. A draft move whose rule's supply method is `mts_else_mto` becomes *confirmed* and creates a need. Every other draft move becomes *confirmed* and creates no need. | none |
| `RP-RULE-141` | For a move whose rule's supply method is not `mts_else_mto`, the quantity to procure is the move's full demand quantity. | none |
| `RP-RULE-142` | For a move whose rule's supply method is `mts_else_mto`: when the move's real quantity is zero or negative, or when the move's source location bypasses reservation, the whole demand quantity is procured. Otherwise the free quantity of the product at the move's source location is read once per (location, product) pair, the quantity already consumed by earlier moves of the same batch for that pair is subtracted, the result is floored at zero to give the available quantity, and the quantity to procure is `max(move real quantity − available quantity, 0)` converted into the move's unit with half-up rounding. The consumed amount for that pair is then increased by `min(move real quantity, available quantity)`. | none |
| `RP-RULE-143` | The downstream-move link `move_destinations` is carried by the created need only when the move's own supply method is `make_to_order`. A move whose rule is `mts_else_mto` creates an unlinked need, because the part taken from stock and the part ordered are independent. | none |
| `RP-RULE-144` | The partner carried by the created need is set only when the rule's supply method is `make_to_order` or `mts_else_mto`. It is the partner of the destination location's warehouse when the move's source location is the company's internal transit location, otherwise the move's own partner. | none |
| `RP-RULE-145` | The warehouse carried by the created need is the move's `warehouse`, else the warehouse of the move's operation type; when the move's source location belongs to no warehouse, it is the supplier warehouse of the route of the move's rule instead. | none |
| `RP-RULE-146` | The routes carried by the created need are the move's own routes; when the move has none, the routes of the package types of the packages holding the move's result packages are used. | none |
| `RP-RULE-147` | The name of the created need is the rule's name, or `/` when the move has no rule. Its origin is the name of the move's first reference, else the move's origin, else the display name of its transfer. | none |
| `RP-RULE-148` | When the move's source location belongs to a warehouse and that warehouse's stock location is an ancestor of the source location, the need's `date_planned` is the move's own scheduled date and its `date_order` is that date minus the purchase delay accumulated along the rule chain walked from the source location with the move's routes. In every other case the need's `date_planned` is the move's scheduled date and no `date_order` is carried. | none |
| `RP-RULE-149` | Every move just moved to `confirmed` or `waiting` whose operation type reserves at confirmation gets `reservation_date` equal to today. | none |
| `RP-RULE-150` | A merged move whose demand quantity is negative is turned around: it is pushed first when it has a final location different from its destination location; then its source and destination locations are swapped, its final location becomes its new destination location, its origin and destination links are rebuilt by re-orienting each linked move according to the sign of its own quantity, the sign of its demand quantity is flipped, its operation type becomes the return operation type when one is configured, and its supply method becomes `make_to_stock`. | none |

---

## 11. Adjusting the supply method of an existing move

| Rule | Statement | Message on violation |
|---|---|---|
| `RP-RULE-151` | For a move that already exists, the search walks up from the move's source location looking for a rule whose `location_source` is the candidate location, whose `destination_location` is the move's destination location, whose `action` is not `push`, and, when the caller supplied one, whose operation type has the requested code. | none |
| `RP-RULE-152` | When no such rule is found at any level, the move's supply method becomes `make_to_stock` and no rule is written on the move. | none |
| `RP-RULE-153` | When a rule is found, it is written on the move, and the move's supply method becomes the rule's supply method when that is `make_to_stock` or `make_to_order`, or `make_to_stock` when the rule's supply method is `mts_else_mto`. | none |

---

## 12. Reordering rule evaluation and the scheduler

| Rule | Statement | Message on violation |
|---|---|---|
| `RP-RULE-160` | The rule chain of a reordering rule is built by the following loop. Set the current location to the rule's `source_location`. Repeat: select the rule for the product at the current location with the rule's `route` and warehouse; when no rule is found, stop; when the found rule takes from stock (its supply method is `make_to_stock`) or its action is neither `pull` nor `pull_push`, add it to the chain and stop; otherwise add it to the chain, set the current location to the found rule's `location_source` and repeat. A rule that appears twice in the chain is a configuration error. | `Invalid rule's configuration, the following rule causes an endless loop: <rule display name>` |
| `RP-RULE-161` | The forecast that drives a reordering rule is read at the end of the day `lead_horizon_date`, in the rule's `source_location`, and is then increased by the quantity already in progress that the forecast cannot see. | none |
| `RP-RULE-162` | Nothing is ordered when the forecast quantity is not lower than `product_minimum_quantity` at the product unit's rounding. `quantity_to_order_computed` is then zero. | none |
| `RP-RULE-163` | The scheduler evaluates only the rules whose `trigger` is `auto` and whose product is active. Manual rules are shown on the replenishment report but are executed only when a user presses Order. | none |
| `RP-RULE-164` | A snoozed rule is hidden from the replenishment report until the snooze date has passed. The scheduler is never affected by snoozing, because only automatic rules are scheduled and automatic rules can never be snoozed (`RP-RULE-043`, `RP-RULE-044`). | none |
| `RP-RULE-165` | The scheduler processes reordering rules in batches of one thousand identifiers. In batch mode each batch uses its own database cursor and is committed independently. | none |
| `RP-RULE-166` | A request is built only for the rules whose `quantity_to_order` is strictly positive at the product unit's rounding. | none |
| `RP-RULE-167` | When the run of a batch raises a procurement exception, the failing reordering rules are removed from the batch and the reduced batch is run again, so that the rules that can be processed still are. When no failing rule can be identified, the batch is abandoned and the event `Unable to process orderpoints` is logged. | none |
| `RP-RULE-168` | When the run of a batch raises a database serialization error and the scheduler is running in batch mode, the batch cursor is rolled back and the batch is retried. Outside batch mode the error is re-raised. | none |
| `RP-RULE-169` | For every reordering rule that failed, a warning activity is scheduled on the product template of the rule's product, with the failure message as its note, assigned to the product's responsible person or, when the product has none, to the superuser. The activity is not created when an activity whose note already contains that message exists on the same product template. | the failure message, as the activity note |
| `RP-RULE-170` | The reordering-rule run always carries the reordering-rule marker in its context. That marker is what turns a missing vendor into a hard failure (`RP-RULE-094`) rather than a silent fallback (`RP-RULE-095`), and what makes a nested move confirmation tolerate failures. | none |
| `RP-RULE-171` | The event-driven trigger matches, for each confirmed move, at most one reordering rule whose product is the move's product, whose `trigger` is `auto`, whose `source_location` is the move's source location or an ancestor of it, whose company is the move's company, and whose `source_location` is neither the move's destination location nor an ancestor of it. An internal transfer inside the watched location therefore creates no need. | none |
| `RP-RULE-172` | The event-driven trigger is disabled entirely while the stored parameter `inventory.disable_automatic_scheduler` is set. | none |
| `RP-RULE-173` | The scheduler reserves the moves matching: the named company when one was given; `state` in (`confirmed`, `partially_available`); `demand_quantity` not zero; and (`reservation_date` not later than today or the operation type reserves at confirmation). When the Manufacturing capability package is installed, finished-product moves of manufacturing orders are excluded. The moves are ordered by `reservation_date` ascending, then `priority` descending, then scheduled date ascending, then surrogate key ascending, and are reserved in chunks of one thousand. | none |
| `RP-RULE-174` | Any exception raised during the three scheduler tasks is logged with its stack trace and re-raised, which aborts that scheduled run. The next run starts from scratch. | none |
| `RP-RULE-175` | Manufacturing orders created as drafts by a reordering-rule batch are confirmed only after every reordering rule of that batch has run, so that the component needs of one manufacturing order cannot interfere with the rules still to be processed. | none |

---

## 13. Manual replenishment and the replenishment report

| Rule | Statement | Message on violation |
|---|---|---|
| `RP-RULE-180` | Pressing Order runs the reordering-rule procurement pass for the selected rules in the active company with user-facing errors enabled. | the failure message |
| `RP-RULE-181` | When exactly one rule was selected and the run fails, the error is presented as a redirect warning whose button is labelled `Edit Product` and which opens the product form of the rule's product. When more than one rule was selected, the error is shown unchanged. | the failure message plus the button `Edit Product` |
| `RP-RULE-182` | After a successful Order, the manual quantity override of every selected rule is cleared and `quantity_to_order` is recomputed. | none |
| `RP-RULE-183` | After a successful Order, every selected rule that was created by the superuser, has `trigger` equal to `manual` and now has a `quantity_to_order` that is zero or below is deleted. | none |
| `RP-RULE-184` | The notification shown after a successful Order on exactly one rule is: the purchase order created or modified since the start of the operation and linked to that rule, under the title `The following replenishment order has been generated`; otherwise the transfer of the stock move created or modified since the start of the operation and linked to that rule, under the title `The inter-warehouse transfers have been generated`, but only when that move's source location belongs to a different warehouse than the rule's or is a transit location and the move has a transfer; otherwise no notification at all. | none |
| `RP-RULE-185` | Order to Max first forces `quantity_to_order` to the multiple-rounded value of `product_maximum_quantity` minus `quantity_forecast`, then behaves exactly as Order. Because writing `quantity_to_order` runs the inverse rule, the forced value is stored as a manual override on a manual rule and is discarded on an automatic rule (`RP-RULE-056`). | none |
| `RP-RULE-186` | "Order and set to automatic" writes `trigger` equal to `auto` on the selected rules and then behaves exactly as Order. | none |
| `RP-RULE-187` | Opening the replenishment report first deletes every satisfied temporary rule (`RP-RULE-059`), then, when the opening context asks for it, recomputes `quantity_to_order_computed` and `deadline_date` on the remaining rules. | none |
| `RP-RULE-188` | The replenishment report considers only locations whose `replenish_location` flag is true, and only goods products that have at least one stock move. | none |
| `RP-RULE-189` | A location may not have `replenish_location` true while one of its ancestors or descendants also has it true. | `Another parent/sub replenish location <name> exists, if you wish to change it, uncheck it first` |
| `RP-RULE-190` | `replenish_location` is forced to false for any location whose usage is not `internal`. | none |
| `RP-RULE-191` | A temporary reordering rule is created only when no rule, active or archived, exists for that product and location; when one exists, its forecast quantity is increased by the shortage instead, so that the report shows the extra need on the existing rule. | see `RP-RULE-040` |
| `RP-RULE-192` | A temporary reordering rule is created under the superuser account with `product_minimum_quantity` 0.0, `product_maximum_quantity` 0.0, `trigger` `manual`, `name` `Replenishment Report`, the warehouse of the location or, when the location has none, the first warehouse of the location's company, and the company of the location. | none |
| `RP-RULE-193` | Snoozing writes `snoozed_until` on every selected rule. The shortcut `1 Day` means tomorrow, `1 Week` means today plus one week, `1 Month` means today plus one month, and `Custom` leaves the date for the user to type. | see `RP-RULE-044` |

---

## 14. The Replenishment Information wizard and the Product Replenish wizard

| Rule | Statement | Message on violation |
|---|---|---|
| `RP-RULE-200` | Editing the minimum or maximum quantity inside the Replenishment Information wizard writes straight back onto the reordering rule with the current user's rights, not with elevated rights. A user who may not write reordering rules is therefore refused. | the shared access-rights message of the platform |
| `RP-RULE-201` | Choosing a supplying warehouse whose free-to-use quantity is lower than the quantity to order opens a confirmation form titled `Quantity available too low` before anything is written. | `<warehouse name> can only provide <free quantity> <unit>, while the quantity to order is <quantity to order> <unit>.` |
| `RP-RULE-202` | "Order the available quantity" writes the chosen route on the reordering rule and sets the rule's `quantity_to_order` to the option's free quantity. "Order everything" writes only the route. | none |
| `RP-RULE-203` | When the Replenishment Information wizard was opened from the Product Replenish wizard, choosing a supplying warehouse writes the route on that wizard instead of on the reordering rule, and reopens that wizard. | none |
| `RP-RULE-204` | Setting a Vendor Price as the supplier of a reordering rule also sets the rule's route to a `buy` route when the rule's route contains no `buy` rule, and raises the rule's `quantity_to_order` to the vendor's minimum quantity converted into the product unit when the current quantity to order is lower. | none |
| `RP-RULE-205` | A route is offered by the Product Replenish wizard when it is selectable on products, or is one of the warehouse routes that is a valid resupply route for the product (`RP-RULE-013`); and none of its rules has the shared inter-company transit location as source or destination; and every one of its rules has a destination location that belongs to a warehouse. | none |
| `RP-RULE-206` | When the Purchase Inventory capability package is installed, the Product Replenish wizard additionally offers the routes of `buy` rules of the company whose operation type has code `incoming`, but only when the product has at least one Vendor Price. | none |
| `RP-RULE-207` | The global Dropship route is never offered by the Product Replenish wizard. | none |
| `RP-RULE-208` | The Product Replenish wizard always runs its request with `force_unit_of_measure` true, so that the typed unit is kept and is never converted into the vendor's purchase unit (`RP-RULE-114`). | none |
| `RP-RULE-209` | When a warehouse cannot be found for the company, opening the Stock Rules Report wizard is refused with the shared "no warehouse configured" redirect warning. | see `RP-RULE-046` |

---

## 15. Dates, deadlines and lateness

| Rule | Statement | Message on violation |
|---|---|---|
| `RP-RULE-220` | A pull rule sets the created move's scheduled date to `values.date_planned` minus the rule's `lead_time_days` in calendar days, and its deadline to `values.date_deadline` minus the same number of days, or empty when the request carries no deadline. | none |
| `RP-RULE-221` | A push rule sets the created move's scheduled date to the source move's scheduled date plus the rule's `lead_time_days` in calendar days, and copies the source move's deadline unchanged. | none |
| `RP-RULE-222` | A purchase order line sets both the scheduled date and the deadline of its receipt moves to the line's planned date, or to the order's planned date when the line has none. | none |
| `RP-RULE-223` | Writing a new deadline on a move shifts the deadline of every origin and destination move that is neither completed nor cancelled by the same number of days, recursively, keeping a set of already visited moves so that a cycle cannot loop forever. The shift is `current deadline − new deadline`; when the move had no deadline the shift is zero and nothing propagates. | none |
| `RP-RULE-224` | `delay_alert_date` of a move that is neither completed nor cancelled is the greatest scheduled date among its not-yet-completed origin moves, but only when that date is later than the move's own scheduled date; otherwise it is empty. It is always empty for completed and cancelled moves. A move with a non-empty `delay_alert_date` is shown as late. | none |
| `RP-RULE-225` | When a deadline change propagates from one document to another, a note is posted on each affected document with subject `Deadline updated due to delay on <origin document name>` and body `The deadline has been automatically updated due to a delay on <link to the origin document>.`, authored by the system account. The note is skipped when the most recent message on the document already carries the same subject. | none |
| `RP-RULE-226` | Writing `date_planned` on a purchase order line writes the same value as the deadline of the line's moves that are not yet completed; when the line has no such moves, the deadline is written on the line's downstream moves instead. | none |
| `RP-RULE-227` | The shared date-update operation only changes a purchase order line's `date_planned` when the line has no moves at all, or has at least one move that is neither completed nor cancelled. A line whose moves are all completed or cancelled keeps its planned date. | none |
| `RP-RULE-228` | All lead times in this domain are counted in calendar days. Weekends, public holidays and work centre capacity are never taken into account. | none |
| `RP-RULE-229` | The date at which a reordering rule's procurement is placed is `lead_horizon_date` at 12:00 in the time zone of the company's partner, falling back to coordinated universal time when the partner declares no time zone, converted to coordinated universal time; and then, when the company's replenishment horizon is not zero, reduced by that number of days. | none |

---

## 16. Cancellation

| Rule | Statement | Message on violation |
|---|---|---|
| `RP-RULE-240` | A completed move may not be cancelled unless its destination is an inventory-loss location. | `You cannot cancel a stock move that has been set to 'Done'. Create a return in order to reverse the moves which took place.` |
| `RP-RULE-241` | When a cancelled move has `propagate_cancel` true and every sibling move (the other origin moves of its downstream moves) is already cancelled, the downstream moves whose state is not `done` and whose source location equals the cancelled move's destination location are cancelled; every other downstream move has its supply method set to `make_to_stock` and is unlinked from the cancelled move. | none |
| `RP-RULE-242` | When a cancelled move has `propagate_cancel` false and every sibling move is completed or cancelled, the downstream moves have their supply method set to `make_to_stock` and are unlinked from the cancelled move; none of them is cancelled. | none |
| `RP-RULE-243` | When the stored parameter `inventory.cancel_originating_moves` is set, cancelling a move with `propagate_cancel` true also cancels its origin moves that are not completed. | none |
| `RP-RULE-244` | Breaking a make-to-order link removes the origin move from the downstream move's origin list, sets the downstream move's supply method to `make_to_stock`, and recomputes the downstream move's state from its remaining links and its reservation. | none |
| `RP-RULE-245` | Cancelling a purchase order cancels every not-yet-completed move of its lines. For the lines' downstream moves that are not completed and do not go to an inventory-loss location: moves whose rule's route is not the reception route of their destination warehouse fall back to `make_to_stock` (they belong to the customer's chain and must survive); moves also fed by another created purchase order line are only unlinked from this line; every remaining move is cancelled when the line's `propagate_cancel` is true and falls back to `make_to_stock` otherwise. | none |
| `RP-RULE-246` | For every already completed transfer of a cancelled purchase order, a note is posted. | `The purchase order <link to the order> this receipt is linked to was cancelled.` |
| `RP-RULE-247` | Deleting a purchase order line cancels its moves; downstream moves fed by more than one created purchase order line are only unlinked from this line; when `propagate_cancel` is true the remaining downstream moves are cancelled, otherwise they fall back to `make_to_stock` and their states are recomputed. | none |

---

## 17. Drop shipping

| Rule | Statement | Message on violation |
|---|---|---|
| `RP-RULE-250` | A transfer is a drop shipment when its source location is a vendor location, or a transit location with no company, **and** its destination location is a customer location, or a transit location with no company. | none |
| `RP-RULE-251` | An operation type whose code is `dropship` always has the shared vendor location as default source location, the shared customer location as default destination location, no warehouse, and is always shown in the operations overview. | none |
| `RP-RULE-252` | A drop shipping purchase order always carries a destination address; the customer's shipping address is written on it. | none |
| `RP-RULE-253` | The delivered quantity of a sales order line whose purchase order lines are drop shipped is the sum of the ordered quantities of those purchase order lines, excluding cancelled ones, converted into the sales line unit with half-up rounding. This replaces the quantity computed from stock moves, and applies only when the purchase order line product equals the sales order line product, which excludes kits with drop-shipped components. | none |
| `RP-RULE-254` | A sales order line whose purchase order line count is greater than zero may not have its product changed by a user of the purchase user group. | the shared read-only message of the platform |
| `RP-RULE-255` | A sales order line is flagged make to order, in addition to the ordinary tests owned by `../sales/`, when any rule of the line's routes has an operation type whose default source location is a vendor location and whose default destination location is a customer location. | none |
| `RP-RULE-256` | When a confirmed drop shipping purchase order covers lines belonging to more than one sales order, has no transfer that is neither completed nor cancelled, and holds at least one goods product, one transfer is created per sales order instead of one transfer for the whole order. Each transfer receives only the lines of that sales order, its moves are confirmed, numbered by ascending scheduled date in steps of five, and reserved, and a note linking back to the purchase order is posted on it. | none |
| `RP-RULE-257` | A purchase order counts drop shipments separately from receipts: the drop shipment count counts the transfers whose drop-shipment flag is true, and the incoming shipment count excludes exactly those. The same split is applied on the sales order between deliveries and drop shipments. | none |
| `RP-RULE-258` | The stock reference created for a drop shipping purchase order additionally links the sales order when every line of the order belongs to the same single sales order. | none |
| `RP-RULE-259` | The description written on a drop-shipped move is the outgoing picking description of the product rather than the incoming one. | none |
| `RP-RULE-260` | When the destination of a `buy` rule is the company's subcontracting location or any subcontracting location, the values carry no partner, and the first downstream move belongs to a manufacturing order that has a subcontractor, that subcontractor becomes the destination address of the created purchase order. | none |
| `RP-RULE-261` | When a `buy` rule's source is a vendor location, its destination is a subcontracting location and the values carry a partner, the purchase order grouping key gains the destination address as an extra component, so that components for different subcontractors are never merged into one order. | none |
| `RP-RULE-262` | When a reordering rule's location is a subcontracting location with exactly one subcontractor, the procurement values built from that rule carry that subcontractor as partner. | none |
| `RP-RULE-263` | The company's drop-ship-to-subcontractor operation type is active exactly when that company still has at least one active pull rule in the global Dropship route; the global Dropship route is active exactly when at least one active pull rule remains in it across every company. | none |

---

## 18. Inter-warehouse resupply

| Rule | Statement | Message on violation |
|---|---|---|
| `RP-RULE-270` | The transit location used between two warehouses is the company's internal transit location when both warehouses belong to the same company, and the shared inter-company transit location otherwise. When neither exists, the pair is skipped and no resupply route is created. The chosen transit location is activated. | none |
| `RP-RULE-271` | The output location of the supplying warehouse is its stock location when it delivers in one step, and its Output location otherwise. | none |
| `RP-RULE-272` | Inside a resupply route, a pull rule takes from stock when its source location is the supplying warehouse's stock location, and triggers another rule otherwise. | none |
| `RP-RULE-273` | When a supplying warehouse delivers in one step, an extra make-to-order rule from its output location to the transit location is added to the global "Replenish on Order" route, because such a warehouse has no delivery chain of its own to reuse. | none |
| `RP-RULE-274` | Removing a warehouse from `resupply_warehouses` archives the resupply routes that linked the two warehouses; adding it back unarchives the archived routes when they exist, and creates new ones only for the warehouses that had none. | none |
| `RP-RULE-275` | When the delivery steps of a warehouse change and the change crosses the boundary between one step and several steps, every rule of every route that this warehouse supplies, that is not a push rule and whose destination is a transit location, has its source location rewritten to the new output location and its supply method set to `make_to_order` when moving to several steps, or `make_to_stock` when moving back to one step. | none |
| `RP-RULE-276` | Moving a supplying warehouse back to one delivery step archives the extra "stock to Output" rules of the supplied routes and creates new make-to-order rules from the stock location to each transit destination with the outgoing operation type. | none |
| `RP-RULE-277` | Moving a supplying warehouse to several delivery steps unarchives those extra rules, creates a "stock to new output location" rule with the picking operation type for every supplied route that had none, and archives every rule of the global "Replenish on Order" route whose destination is a transit location and whose source is this warehouse's stock location. | none |
| `RP-RULE-278` | The company of a generated resupply route is the intersection of the two warehouses' companies, which is empty when they differ. | none |
| `RP-RULE-279` | Renaming a warehouse rewrites, in each of its route names, in each rule name of those routes, in the make-to-order rule name and in the `buy` rule name, the first occurrence of the old warehouse name by the new one. | none |

---

## 19. Company, currency and access

| Rule | Statement | Message on violation |
|---|---|---|
| `RP-RULE-290` | A Reordering Rule is visible only to users whose active company set contains its company. A Route and a Stock Rule are visible when their company is in the active company set or is empty. | none |
| `RP-RULE-291` | An inventory user may read Routes, Stock Rules and Reordering Rules but may not create, change or delete them. | the shared access-rights message of the platform |
| `RP-RULE-292` | An inventory administrator may create, read, change and delete Routes, Stock Rules and Reordering Rules. | none |
| `RP-RULE-293` | Any internal user may read Routes and Stock Rules, because rule selection runs on behalf of users who have no inventory rights at all (for example a salesperson confirming a sales order). | none |
| `RP-RULE-294` | An inventory user may create, read and change the Stock Rules Report wizard, the Product Replenish wizard and the Replenishment Option wizard, and may create, read, change and delete the Reordering Rule Snooze wizard. | none |
| `RP-RULE-295` | Only an inventory administrator may create, read or change the Replenishment Information wizard. An inventory user who is not an administrator cannot open the replenishment information dialog. | the shared access-rights message of the platform |
| `RP-RULE-296` | A purchase user and a purchase manager may read Locations, Warehouses and Reordering Rules; may create, read, change and delete Transfers; may create, read and change Stock Moves; and, for a purchase manager only, may also delete Stock Moves. | none |
| `RP-RULE-297` | An inventory user may read Purchase Orders and Purchase Order Lines but may not create, change or delete them. This is what lets an inventory user update the received quantity on a receipt that feeds a purchase order line. | none |
| `RP-RULE-298` | A purchase user and a purchase manager may read the Vendor Delay Report. | none |
| `RP-RULE-299` | The currency of a purchase order created by a `buy` rule is the currency of the chosen Vendor Price; when that is empty, the vendor's purchase currency for the order company; when that is empty too, the currency of the order company. The currency is part of the grouping key, so needs that resolve to different currencies never share an order. | none |
| `RP-RULE-300` | A Vendor Price expressed in a currency other than the order currency is converted at the rate of today, for both a newly created line and an updated line. | none |
| `RP-RULE-301` | Every document created by this domain carries the company determined by the rule, never the company of the user who triggered the need. | none |
| `RP-RULE-302` | Every quantity comparison and every quantity rounding in this domain uses the rounding of the relevant unit of measure, as specified in `../units-of-measure-and-packaging/calculations.md`. Comparisons are never exact equality comparisons on stored decimals. | none |
| `RP-RULE-303` | **Industry-standard completion.** A run that creates or extends documents must be atomic per batch: either every document of the batch is written, or none is. The scheduler achieves this with a save point per batch; a replacement must offer the same isolation, otherwise a partially processed batch can create duplicate purchase order lines on the next run. | none |
| `RP-RULE-304` | **Industry-standard completion.** Two concurrent runs for the same reordering rule must not both create a document. A replacement must serialize the evaluation of one reordering rule, for example by taking a row lock on the rule for the duration of its request, and must retry the batch when the database reports a serialization failure (see `RP-RULE-168`). | none |

---

## 20. Rounding rules

| Rule | Statement | Message on violation |
|---|---|---|
| `RP-RULE-310` | Every quantity stored by this domain carries the precision of the shared decimal precision setting named "Product Unit", whose default is two decimal places. | none |
| `RP-RULE-311` | The quantity to order is rounded **up** to a whole number of the replenishment multiple, then converted back into the product unit. When no multiple applies, no rounding to a multiple is performed and only the stored precision applies. | none |
| `RP-RULE-312` | Conversions between a request unit and a purchase line unit use half-up rounding. | none |
| `RP-RULE-313` | Conversions of a quantity in progress into the reordering rule unit are performed without rounding, so that a partial pack in progress is not lost. | none |
| `RP-RULE-314` | The daily demand and the average stock of the demand graph are rounded to the rounding of the product's unit; the ordering period is rounded to a whole number of days. | none |
| `RP-RULE-315` | The on-time delivery rate is expressed as a percentage with the full stored precision of the underlying quantities; when no quantity was ordered at all, the rate is the sentinel value −1, which the screens display as "no data" rather than as zero. | none |

---

## 21. Returning goods to a vendor

| Rule | Statement | Message on violation |
|---|---|---|
| `RP-RULE-330` | When the source location of the return has usage `supplier`, every move the wizard creates is stamped with the purchase order line and with the counterparty found by the chain walk of `calculations.md`, section 31, applied to the move being returned. When the walk finds nothing, both are left empty and no error is raised. | none |
| `RP-RULE-331` | After a return transfer has been built, when its moves resolve to exactly one counterparty and that counterparty differs from the transfer's own, the transfer's counterparty is rewritten to it. When the moves resolve to none or to more than one, the transfer keeps the counterparty it was created with. | none |
| `RP-RULE-332` | A move counts as a **purchase return** when its destination location usage is `supplier`, or when it has an originating returned move and either its destination location is the shared inter-company transit location or the originating returned move's source location usage is `supplier`. | none |
| `RP-RULE-333` | The received quantity of a purchase order line is the sum of the quantities of its incoming moves minus the sum of the quantities of its outgoing moves, both converted into the line unit with half-up rounding. Only moves that are not cancelled, whose destination location usage is not `inventory` and whose product is the line's product take part. | none |
| `RP-RULE-334` | Such a move is outgoing when it is a purchase return and (its `to_refund` is true or it has no originating returned move). It is incoming when it is not outgoing, its destination location usage is not `supplier`, and (it has no originating returned move or its `to_refund` is true). Every other move is ignored by the received quantity. | none |
| `RP-RULE-335` | The quantity read from a counted move is its completed quantity when the move is completed, and its demand quantity otherwise. A return that has been created but not validated therefore does not move the received quantity. | none |
| `RP-RULE-336` | Unticking "Update Quantities on Purchase Order" on a return line excludes the created move from both sides of `RP-RULE-334`, so the goods leave or enter the company without changing the received quantity. | none |
| `RP-RULE-337` | Changing the operation type of a return before it is validated changes the received quantity only when it changes the destination location of the return's moves, because `RP-RULE-332` and `RP-RULE-334` read the destination location and the originating returned move, never the operation type. | none |
| `RP-RULE-338` | The Return button is offered on a receipt only when the receipt belongs to a purchase order. Returns of transfers that do not come from a purchase order are reached through the operations of `../inventory-operations/`. | none |
| `RP-RULE-339` | An exchange created from a receipt detaches the exchange moves from their originating returned moves, so an exchange move is never a purchase return and never reduces the received quantity. A purchase order received once, returned once and exchanged once shows three transfers. | none |
| `RP-RULE-340` | An exchange may be created from a return transfer that has no originating transfer. The new transfer is then built from the return's own operation type, source location and destination location, no purchase order line is attached to its moves, and no received quantity changes. | none |
| `RP-RULE-341` | **Industry-standard completion.** A return may not send back more than has been received net of earlier returns. The "Return all" operation enforces this by proposing, per move, the completed quantity minus what earlier returns already took back; a replacement must apply the same ceiling when a quantity is typed by hand, otherwise the received quantity of the purchase order line can become negative. | `You cannot return more than what has been received.` |

---

## 22. The purchase analysis view

| Rule | Statement | Message on violation |
|---|---|---|
| `RP-RULE-350` | The `effective_date` of a purchase order is the earliest completion date among the order's completed transfers whose destination location usage is not `supplier`. A transfer with no completion date does not take part. A return to the vendor therefore never moves the effective date. | none |
| `RP-RULE-351` | The `days_to_arrival` column of the purchase analysis view is computed by the formula of `calculations.md`, section 30, and is aggregated as a plain average, never weighted by quantity or by amount. | none |
| `RP-RULE-352` | The warehouse column of the purchase analysis view is the warehouse of the purchase order's operation type. It is empty for every order whose operation type has no warehouse, which includes every drop shipping order. | none |
| `RP-RULE-353` | The grouping of the purchase analysis view includes the warehouse, the effective date and the earliest completion date, so two lines of one order that arrived on different days are not merged into one row. | none |
| `RP-RULE-354` | The purchase analysis view is read-only. No operation of this domain writes it; it is recomputed from the purchase order lines and their transfers on every read. | none |
