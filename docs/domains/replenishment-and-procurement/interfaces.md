# Interfaces

A client, an integration or a user reaches the Replenishment and Procurement domain through service operations, structured payloads, screens, printed documents, notifications and one scheduled job. All of them are specified below. No client technology is named; a screen is described by the fields it shows, the buttons it offers, the guard of each button, the filters and the groupings.

---

## 1. Service operations

Each operation states its inputs, its outputs, the records it writes and the errors it can raise. An operation marked "collection operation" acts on a set of records; an operation marked "model operation" needs no record at all.

### 1.1 `run_procurements` (model operation)

| Aspect | Specification |
|---|---|
| Inputs | a list of procurement requests (structure in `entities.md`, section 10); a boolean `raise_user_error`, default true |
| Output | true on success |
| Writes | Stock Moves (created and confirmed), Purchase Orders and Purchase Order Lines (created or extended), Manufacturing Orders (created or extended) |
| Errors | when `raise_user_error` is true, one user-facing error whose text is the failure messages joined with new lines; when it is false, a procurement exception carrying the list of (request, message) pairs |
| Failure messages | `No rule has been found to replenish "<product>" in "<location>".` plus a new line plus `Verify the routes configuration on the product.`; `No source location defined on stock rule: <rule name>!`; `There is no matching vendor price to generate the purchase order for product <product> (no vendor defined, minimum quantity not reached, dates not valid, ...). Go on the product form and complete the list of vendors.` |
| Notes | The whole batch fails before any document is created when at least one request finds no rule. |

### 1.2 `get_rule` (model operation)

| Aspect | Specification |
|---|---|
| Inputs | a product; a location; a values map that may carry `routes`, `packaging_unit_of_measure`, `warehouse` and `company` |
| Output | at most one Stock Rule |
| Writes | nothing |
| Errors | none; the absence of a rule is expressed by an empty result |

### 1.3 `get_push_rule` (model operation)

| Aspect | Specification |
|---|---|
| Inputs | a product; a destination location; a values map that may carry `routes`, `packaging_unit_of_measure`, `warehouse` and an extra condition |
| Output | at most one Stock Rule whose action is `push` or `pull_push` |
| Writes | nothing |
| Errors | none |

### 1.4 `get_rules_from_location` (collection operation on a product)

| Aspect | Specification |
|---|---|
| Inputs | a location; optional preferred routes |
| Output | the ordered set of Stock Rules that a need at that location would travel through |
| Writes | nothing |
| Errors | `Invalid rule's configuration, the following rule causes an endless loop: <rule display name>` |

### 1.5 `get_lead_days` (collection operation on a set of Stock Rules)

| Aspect | Specification |
|---|---|
| Inputs | a product; an optional value map carrying `forced_vendor_price`, `bill_of_materials` and `days_to_order`; two switches, "bypass the narrative description" and "bypass the replenishment horizon" |
| Output | the delay component map and the ordered narrative description (see `calculations.md`, section 2) |
| Writes | nothing |
| Errors | none |

### 1.6 `get_dates_information` (collection operation on a product)

| Aspect | Specification |
|---|---|
| Inputs | a date; a location; optional routes |
| Output | a pair: the planned date and the order date |
| Writes | nothing |
| Errors | the endless-loop error of 1.4 |

### 1.7 `run_scheduler` (model operation)

| Aspect | Specification |
|---|---|
| Inputs | a boolean "use a dedicated cursor" (batch mode), default false; an optional company |
| Output | an empty result |
| Writes | Stock Moves, Purchase Orders, Manufacturing Orders, reservations, merged stock quantity records, warning activities |
| Errors | any exception is logged with its stack trace and re-raised |

### 1.8 `procure_orderpoint_confirm` (collection operation on Reordering Rules)

| Aspect | Specification |
|---|---|
| Inputs | a boolean "use a dedicated cursor"; an optional company; a boolean `raise_user_error`, default true |
| Output | an empty result |
| Writes | the documents that the requests create; warning activities for the rules that failed |
| Errors | when `raise_user_error` is true, the user-facing error of 1.1 |

### 1.9 `action_replenish` (collection operation on Reordering Rules)

| Aspect | Specification |
|---|---|
| Inputs | a boolean `force_to_maximum`, default false |
| Output | a notification descriptor, or nothing |
| Writes | the documents; clears the manual quantity override; deletes satisfied temporary rules |
| Errors | on failure with exactly one selected rule, a redirect warning whose button `Edit Product` opens the product form; otherwise the plain error |

### 1.10 `action_replenish_auto` (collection operation on Reordering Rules)

Writes `trigger` equal to `auto` on the selected rules, then behaves as `action_replenish`.

### 1.11 `action_open_orderpoints` (model operation)

| Aspect | Specification |
|---|---|
| Inputs | an optional switch in the opening context, "force the recomputation"; an optional forced replenishment horizon |
| Output | the descriptor of the Replenishment screen |
| Writes | deletes satisfied temporary Reordering Rules; creates temporary Reordering Rules for the negative forecasts found; increases the forecast quantity of existing rules that cover a shortage |
| Errors | the endless-loop error of 1.4 |

### 1.12 `action_stock_replenishment_information` (collection operation on one Reordering Rule)

| Aspect | Specification |
|---|---|
| Inputs | none |
| Output | the descriptor of a dialog titled `Replenishment Information for <product display name> in <warehouse display name>` |
| Writes | one Replenishment Information record |
| Errors | the access-rights error when the caller is not an inventory administrator |

### 1.13 `action_snooze` (collection operation on the Reordering Rule Snooze wizard)

| Aspect | Specification |
|---|---|
| Inputs | the selected reordering rules and a date |
| Output | closes the dialog |
| Writes | `snoozed_until` on the selected rules |
| Errors | `You can only snooze manual orderpoints. You should rather archive 'auto-trigger' orderpoints if you do not want them to be triggered.` |

### 1.14 `action_remove_manual_quantity_to_order` (collection operation on Reordering Rules)

Sets `quantity_to_order_manual` to zero, which makes `quantity_to_order` fall back to the computed value.

### 1.15 `select_route`, `order_available`, `order_all` (collection operations on one Replenishment Option)

| Operation | Behavior |
|---|---|
| `select_route` | When the free quantity is lower than the quantity to order, opens the warning form titled `Quantity available too low`; otherwise behaves as `order_all`. When the opening context carries a Product Replenish wizard, writes the route on that wizard and reopens it instead. |
| `order_available` | Writes the route on the parent reordering rule, sets the rule's `quantity_to_order` to the option's free quantity, closes the dialog. |
| `order_all` | Writes the route on the parent reordering rule, closes the dialog. |

### 1.16 `action_set_vendor_price` (collection operation on one Vendor Price)

| Aspect | Specification |
|---|---|
| Inputs | the reordering rule named in the opening context; optionally a Product Replenish wizard named in the opening context |
| Output | the Replenishment Information dialog again, or the Product Replenish dialog when one was named |
| Writes | the rule's `vendor_price`; the rule's `route` when it held no `buy` rule; the rule's `quantity_to_order` raised to the vendor's minimum quantity converted into the product unit when it was lower |
| Errors | none; nothing happens when no reordering rule is named |

### 1.17 `print_report` (collection operation on the Stock Rules Report wizard)

Renders the routes diagram (section 5.1) for the chosen product and warehouses.

### 1.18 `get_report_values` (model operation, forecast report)

| Aspect | Specification |
|---|---|
| Inputs | a list of product variants, or a list of product templates |
| Output | the forecast report payload (section 4) |
| Writes | nothing |
| Errors | none |

### 1.19 `action_reserve_linked_transfers` and `action_unreserve_linked_transfers` (model operations)

| Aspect | Specification |
|---|---|
| Inputs | one stock move |
| Output | the set of moves that were acted upon |
| Writes | reservations. Reserve keeps the origin moves, recursively, whose state is not draft, cancelled, available or completed, and reserves them. Release keeps the origin moves whose state is not draft, cancelled or completed, and releases their reservations. |
| Errors | none |

### 1.20 `action_view_purchase` (collection operation on one Reordering Rule)

Opens the list of purchase orders that hold a line linked to this reordering rule.

### 1.21 `action_product_forecast_report` (collection operation on one Reordering Rule)

Opens the forecast report for the rule's product, carrying in the context the rule's warehouse, the formatted `lead_horizon_date` and the quantity to order.

### 1.22 `action_create_returns`, `action_create_returns_all` and `action_create_exchanges` (collection operations on one Return Wizard)

The three operations are owned by `../inventory-operations/`. This domain extends two of the steps they run, so their inputs, outputs and errors are restated here with the additions marked.

| Aspect | Specification |
|---|---|
| Inputs | the wizard, which carries the completed transfer and one line per returnable move, each with a quantity and the flag `to_refund` |
| Output | the descriptor of a form opened on the created return transfer |
| Writes | one transfer and one move per non-zero line; **added by this domain:** the purchase order line and the counterparty on each created move when the source location of the return has usage `supplier`, and the counterparty on the created transfer when its moves agree on exactly one |
| Side effects | the downstream moves of every returned move that are neither completed nor cancelled are unreserved; the new transfer is confirmed and reservation is attempted; the received quantity of every affected purchase order line is recomputed |
| Errors | `You may only return one picking at a time.`; `You may only return Done pickings.`; `No products to return (only lines in Done state and not fully returned yet can be returned).`; `Please specify at least one non-zero quantity.`; `You cannot return more than what has been received.` |
| Difference between the three | `action_create_returns` uses the typed quantities; `action_create_returns_all` first fills each line with the completed quantity of its move minus what earlier returns took back; `action_create_exchanges` runs the return and then, for a receipt, creates the return of that return |

---

## 2. Screens

### 2.1 Replenishment (the replenishment report)

**Reached by** Operations → Replenishment. **Opening the screen runs `action_open_orderpoints` first**, so the list already contains one temporary rule per shortage.

**List view, editable in place, multi-record editing allowed, sample data shown when empty.**

| Column | Notes |
|---|---|
| Product | read-only once saved |
| Location | shown only to users of the "Manage Multiple Stock Locations" group |
| Warehouse | shown only to users of the "Manage Multiple Warehouses" group; hidden by default |
| On Hand | read-only |
| Forecast | read-only |
| Route | hidden by default; shows the placeholder of the route that would be used when empty; shown greyed when empty |
| Trigger | hidden by default |
| Min | the minimum quantity |
| Max | the maximum quantity |
| Multiple | hidden by default; shows the placeholder of the fallback multiple when empty; shown greyed when empty |
| To Order | read-only when the trigger is `auto` |
| Unit | shown only when the unit of measure group is active |
| Deadline | hidden by default; shown in red when it is earlier than today |
| Company | hidden by default, read-only, shown only in a multi-company database |

**Buttons on each row.**

| Button | Guard | Effect |
|---|---|---|
| Forecast Report (chart icon) | the row is saved and `unwanted_replenish` is false | opens the forecast report |
| Forecast Report (warning icon), with the caption `Due to receipts scheduled in the future, you might end up with excessive stock . Check the Forecasted Report  before reordering` | the row is saved and `unwanted_replenish` is true | opens the forecast report |
| Replenishment Information (information icon) | the row is saved and `show_supply_warning` is false | opens the Replenishment Information dialog |
| Replenishment Information (warning icon), with the caption `Your product is missing a way to be replenished (Route, Vendor, Bill of Materials).` | the row is saved and `show_supply_warning` is true | opens the same dialog |
| Undo (circular arrow) | `quantity_to_order_manual` is not zero; otherwise the button is rendered disabled and invisible so that the column keeps its width | clears the manual override |
| Order | `quantity_to_order` is greater than zero | runs `action_replenish` |
| Automate | `quantity_to_order` is greater than zero and the trigger is not `auto` | runs `action_replenish_auto` |
| Snooze | the trigger is `manual` | opens the snooze dialog with this rule preselected |

**Search fields.** Product, Product Category, Route (searched on the effective route), Warehouse, Location.

**Filters.** Archived; Manual; Automatic; To Reorder (quantity to order greater than zero); Not Snoozed (no snooze date, or a snooze date not later than today).

**Groupings.** Warehouse, Location, Product, Category.

**Side panel.** Locations, Trigger, Category, each with record counts.

**Empty state.** `You are good, no replenishment to perform!` followed by `You'll find here smart replenishment propositions based on inventory forecasts. Choose the quantity to buy or manufacture and launch orders in a click. To save time in the future, set the rules as "automated".`

### 2.2 Reordering Rules

**Reached by** Configuration → Reordering Rules. The same list view as 2.1, with the Trigger column shown by default, and opened with the Automatic filter preselected.

**Search fields.** Product, Reordering Rule (the reference), Trigger, Warehouse, Location. **Filters.** Archived. **Groupings.** Warehouse, Location, Product.

**Empty state.** `No reordering rule found` followed by `Define a minimum stock rule so that the system automatically creates requests for quotations or confirmed manufacturing orders to resupply your stock.`

**Form view.** The reference as the title, with an "Archived" ribbon when the rule is archived. Left column: Product; Min Quantity with its unit and a button `Forecast Description` that opens the Replenishment Information dialog (guarded on the record being saved); Max Quantity with its unit; Multiple. Right column: Warehouse and Location (both shown only to users of the "Manage Multiple Stock Locations" group), Company (shown only in a multi-company database).

### 2.3 Routes

**Reached by** Configuration → Routes, visible only to users of the "Multi-Step Routes" group.

**List view.** Route name, Company.

**Form view.** Title: the route name, with the placeholder `e.g. Two-steps reception` and an "Archived" ribbon. Then Sequence and Supplied Warehouse (both for advanced users only) and Company (multi-company only, with the placeholder `Visible to all`). Then a section "Applicable On" introduced by `Select the places where this route can be selected`, with the switches Product Categories, Products, Package Type and Warehouses; ticking Warehouses reveals a warehouse selector. Then a section "Rules" holding the embedded rule list: a drag handle on the sequence, Action, Source Location, Destination Location.

**Search fields.** Name. **Filters.** Archived.

**Empty state.** `Add a new route` followed by an explanation that routes define the flows of products through the warehouses and can be assigned to a product, a product category, or fixed on a procurement or a sales order.

### 2.4 Stock Rules

**Reached by** Configuration → Rules, and from inside a route.

**List view.** Action, Source Location, Destination Location, Route, Company, Name (hidden by default).

**Form view.**

| Block | Content | Visibility |
|---|---|---|
| Title | Name | always |
| Left column | Action; Operation Type; Source Location (mandatory when the action is `pull`, `push` or `pull_push`); Destination Location; "Destination location origin from rule" (advanced users only, and only when the action is `pull` or `pull_push`); Automatic Move (only when the action is `push` or `pull_push`); Supply Method (only when the action is `pull` or `pull_push`) | as stated |
| Right column | the generated rule description sentence, read-only | always |
| Applicability | Route; Warehouse (advanced users only, hidden when the action is `push`); Company (multi-company only, mandatory when the action is `push`, placeholder `Visible to all`); Sequence (advanced users only) | as stated |
| Options | Partner Address (hidden when the action is `push`); Cancel Next Move (hidden when the action is `push`); Lead Time, followed by the word `days` | only when the action is `pull`, `push` or `pull_push` |
| Push Applicability | a condition editor over stock moves | only when the action is `push` or `pull_push` |

When the rule form is opened from inside a route, the Route field is removed and the Applicability block is shown only to advanced users and in a multi-company database.

**Search fields.** Name. **Filters.** Archived. **Groupings.** Route, Destination Location, Warehouse.

### 2.5 Replenishment Information dialog

**Reached by** the information button of the Replenishment screen, the `Forecast Description` button of the reordering rule form, and the Product Replenish wizard.

**Title.** `Replenishment Information for <product display name> in <warehouse display name>`.

**Header block.** Built from the lead-days payload (section 3): the trigger; today's date; the forecast date; the forecast quantity; the quantity to order; the minimum and maximum quantities, editable in place and written straight back onto the rule; the unit name; and the flag that says whether this is a temporary rule.

**Lead time block.** One row per description entry, walked from the last entry to the first. An entry whose value is a caption is shown as a labelled caption; an entry whose value is a number of days advances a running date, which starts at today, and the row then shows the label and the resulting date.

**Demand graph block.** The minimum line, the maximum line and the saw-tooth curve, with the selector "Based on" and the percentage factor; changing either recomputes the graph.

**Warehouses tab.** One row per inter-warehouse resupply route of the rule's warehouse, sorted by free-to-use quantity descending: the supplying warehouse, its free quantity, the unit, the quantity to order, the lead time as the text `<n> days`, and a `Select Route` button.

**Vendors tab.** Every Vendor Price of the product, shown when the rule has no preferred route or when at least one selected rule has action `buy`. Each row offers `Set as Supplier`, hidden on the row that is already the rule's vendor price. Each row also shows the last purchase date.

**Bills of Materials tab.** Every bill that could produce the product, shown under the equivalent condition for `manufacture`.

### 2.6 Quantity available too low (warning form)

**Title.** `Quantity available too low`. **Body.** `<warehouse name> can only provide <free quantity> <unit>, while the quantity to order is <quantity to order> <unit>.` **Buttons.** "order the available quantity" and "order everything".

### 2.7 Snooze dialog

**Fields.** Snooze for (`1 Day`, `1 Week`, `1 Month`, `Custom`, default `1 Day`); Snooze Date, filled automatically from the shortcut. **Button.** Confirm, which runs `action_snooze`.

### 2.8 Product Replenish dialog

**Fields.** Product; Quantity; Unit; Warehouse; Route; Vendor (shown only when the chosen route contains a `buy` rule); Scheduled Date; Company.

**Behavior.** Changing the route recomputes the scheduled date and proposes or clears the vendor. A button opens the Replenishment Information dialog for the product and warehouse. The confirm button runs one procurement request and then shows the notification of section 6.

### 2.9 Stock Rules Report dialog

**Fields.** Product (or Product Template with a variant selector when the template has several variants); Warehouses (mandatory); Apply specific routes (routes selectable on sales order lines, when the Sales Inventory capability package is installed). **Button.** Print, which renders the routes diagram.

### 2.10 Dropships

**Reached by** Operations → Dropships. A list, kanban, form and calendar of transfers restricted to the operation type code `dropship`, with the filter "Dropships" preselected and the contact shown as a delivery address.

### 2.11 On-time Delivery

**Reached by** a vendor's form and a purchase order. A graph of the Vendor Delay Report, with the filter "later than a year ago" preselected. Measures: Total Quantity, On-Time Quantity, On-Time Delivery Rate (aggregated as a weighted average, see `calculations.md`, section 22.2). Groupings: Vendor, Product, Product Category, Effective Date.

### 2.12 Purchase Analysis

**Reached by** the purchasing reporting menu, which is owned by `../purchasing/`. A pivot and a graph of the Purchase Analysis Report. This domain adds three columns to that screen.

| Item | Specification |
|---|---|
| Measure "Effective Days To Arrival" | The `days_to_arrival` column, shown with two decimal places and aggregated as a plain average. Formula in `calculations.md`, section 30. |
| Grouping "Warehouse" | The `operation_type` column, which carries the warehouse of the order's operation type. Rows whose order has no warehouse, that is every drop shipping order, are grouped under "None". |
| Grouping and filter "Effective Date" | The `effective_date` column, groupable by day, week, month, quarter and year. |

**Guards.** The screen is read-only. No button on it writes anything.

### 2.13 Return dialog

**Reached by** the Return button of a completed receipt. The button is shown only when the receipt belongs to a purchase order.

| Item | Specification |
|---|---|
| Fields shown per line | Product, Move Quantity (read-only, the quantity that was received), Quantity (editable, zero by default), Unit, and the flag "Update Quantities on Purchase Order" (`to_refund`, true by default) |
| Button "Return" | Creates the return transfer and opens it. Refuses with `Please specify at least one non-zero quantity.` when every quantity is still zero. |
| Button "Return all" | Fills every line with the completed quantity of its move minus what earlier returns already took back, then behaves as "Return". |
| Button "Exchange" | Creates the return and, for a receipt, immediately creates the return of that return, so the goods come back in. |
| Errors on opening | `You may only return one picking at a time.`; `You may only return Done pickings.`; `No products to return (only lines in Done state and not fully returned yet can be returned).` |
| Added by this domain | The created moves carry the purchase order line and the vendor when the return goes to a vendor location; the created transfer's counterparty is rewritten to the vendor when its moves agree on one. |

---

## 3. The Replenishment Information payload

A structured document with the following members.

| Member | Type | Content |
|---|---|---|
| `lead_horizon_date` | text | the rule's forecast date, formatted in the user's language |
| `lead_days_description` | list of triples | one triple (label, value, is_a_date) per description entry, produced as described below |
| `today` | text | today's date, formatted in the user's language |
| `trigger` | text | `auto` or `manual` |
| `quantity_forecast` | text | the forecast quantity rendered at the "Product Unit" precision |
| `quantity_to_order` | text | the quantity to order rendered at the "Product Unit" precision |
| `product_minimum_quantity` | text | the minimum quantity rendered at the "Product Unit" precision |
| `product_maximum_quantity` | text | the maximum quantity rendered at the "Product Unit" precision |
| `product_unit_of_measure_name` | text | the unit label |
| `virtual` | boolean | true when the trigger is `manual` **and** the rule was created by the superuser, that is when it is a temporary rule of the replenishment report |

**Building `lead_days_description`.** Start a running date at today and walk the description entries produced by the lead-days computation **in reverse order**. For an entry whose value is a caption, emit (label, caption, false). For an entry whose value is a number of days, add that number of days to the running date and emit (label, the running date formatted in the user's language, true).

**Worked example.** The lead-days description of a purchasing chain is, in order: (`Receipt Date`, 3), (`Vendor Lead Time`, `+ 3 day(s)`), (`Order Deadline`, 2), (`Days to Purchase`, `+ 2 day(s)`), (`Time Horizon`, `+ 365 day(s)`). Today is 1 March. Walking backwards: `Time Horizon` is a caption, emitted as is; `Days to Purchase` is a caption, emitted as is; `Order Deadline` is 2 days, so the running date becomes 3 March and the row reads "Order Deadline, 3 March"; `Vendor Lead Time` is a caption; `Receipt Date` is 3 days, so the running date becomes 6 March and the row reads "Receipt Date, 6 March". The user therefore reads: order by 3 March, receive on 6 March.

---

## 4. The forecast report payload

A structured document with a header and a list of lines.

### 4.1 Header

| Member | Content |
|---|---|
| `product_templates` | when opened on templates: the templates with their display names; false otherwise |
| `product_template_keys` | the template keys, when opened on templates |
| `product_variants` | when opened on templates: one entry per variant with its key and its attribute combination name; when opened on variants: the variants with their display names |
| `product_variant_keys` | the variant keys |
| `multiple_product` | true when more than one variant is covered |
| `product` | a map from variant key to the per-product block below |
| `lines` | the reconciliation lines (section 4.3) |
| `user_can_edit_transfers` | true when the reader belongs to the inventory user group |

### 4.2 The per-product block

| Member | Content |
|---|---|
| `unit_of_measure_name` | the display name of the product's unit |
| `quantity_on_hand` | the on-hand quantity |
| `forecast_quantity` | the forecast quantity |
| `free_to_use_quantity` | the unreserved quantity |
| `incoming_quantity` | the incoming quantity |
| `outgoing_quantity` | the outgoing quantity |
| `draft_transfer_quantity` | a pair: the quantity of draft incoming moves and the quantity of draft outgoing moves for this warehouse |
| `accumulated_quantities` | a pair: the accumulated incoming and outgoing quantities of every contribution block |
| `lead_time` | a pair: `total_delay`, the cumulative lead time of the rule chain at the warehouse stock location, and `details`, the narrative description of that computation |

### 4.3 A reconciliation line

| Member | Content |
|---|---|
| `document_in` | the source document of the incoming move (its model name, key and display name), or false |
| `document_out` | the source document of the outgoing move, or false |
| `receipt_date` | the incoming move's date formatted in the user's language, or false |
| `delivery_date` | the outgoing move's date formatted in the user's language, or false |
| `product` | the product key and display name |
| `replenishment_filled` | false when this line is unsatisfied demand |
| `is_late` | true when the line carries both moves and the outgoing date is earlier than the incoming date |
| `delivery_late` | true when the outgoing move is not completed and its date is in the past |
| `receipt_late` | true when the incoming move is not completed and its date is in the past |
| `quantity` | the quantity of the line, rounded at the product unit |
| `move_out` | the outgoing move's key and date, plus its transfer's key and priority when it has a transfer |
| `move_in` | the incoming move's key and date |
| `reservation` | the reserving transfer's model name, name and key, or false |
| `in_transit` | true when the quantity is held outside the warehouse stock location, or when the reserving move itself has origin moves |
| `is_matched` | true when either move is one of the moves the caller asked to highlight |
| `unit_of_measure` | the product's unit |

---

## 5. Printed documents

### 5.1 Routes diagram

**Produced by** the Stock Rules Report wizard.

**Content.** A grid. The columns are locations, ordered as follows: first the locations whose usage is vendor or production; then, warehouse by warehouse, the locations of that warehouse in flow order, starting from the locations that are reached by a rule whose source lies outside the warehouse; then the locations whose usage is customer; then every remaining location. The locations of the product's reordering rules are appended when they are not already present.

**Header lines.** For each location that has at least one putaway rule for the product or at least one reordering rule for the product, a header cell listing those putaway rules and those reordering rules.

**Route lines.** One line per rule of the routes to display. The routes to display are the product's own routes, the total routes of the product's category, and the routes of the selected warehouses; a rule is shown only when it has no warehouse or when its warehouse is one of the selected ones. Each line marks the rule's source column as "origin" and its destination column as "destination", in the colour assigned to its route. The destination column of a `pull` rule is the operation type's default destination location; for every other action it is the rule's own destination location.

**Colours.** Routes are coloured by rotating through the seven-colour cycle orange, purple, forest green, dark cyan, steel blue, red, lime green.

**Direction.** The grid is mirrored when the reader's language is written right to left.

### 5.2 Forecast report

**Produced by** the forecast report screen and printable. It renders the payload of section 4 as a header block with the quantities and the lead time breakdown, followed by one row per reconciliation line showing the outgoing document, the incoming document, the quantity, the dates, and the lateness and reservation indicators.

### 5.3 Purchase order documents

The purchase order and the request for quotation documents are owned by `../purchasing/`. This domain adds to them the incoterm named place, and adds to the exception block the notice that a purchase order created from a sales order was cancelled.

---

## 6. Notifications

| Trigger | Kind | Content |
|---|---|---|
| A reordering rule ordered and a purchase order was created or modified | transient notification | title `The following replenishment order has been generated`, with one link labelled with the purchase order display name |
| A reordering rule ordered and an inter-warehouse transfer was created or modified | transient notification | title `The inter-warehouse transfers have been generated`, with one link labelled with the transfer name |
| The Product Replenish wizard created a purchase order | transient notification | title `The following replenishment order have been generated`, with one link to that purchase order |
| The Product Replenish wizard created a stock move | transient notification | a link to the move's transfer, when it has one |
| A `buy` request found no vendor and did not come from a reordering rule | message on the originating document | the mentions of the users to notify, a line break, `No supplier has been found to replenish`, the product display name in bold, and `this product should be manually replenished.` |
| A reordering rule failed during the scheduler | warning activity on the product template | the failure message as the activity note, assigned to the product's responsible person or to the superuser |
| A deadline change propagated to another document | note on the affected document | subject `Deadline updated due to delay on <origin document name>`, body `The deadline has been automatically updated due to a delay on <link to the origin document>.` |
| A purchase order that fed a completed receipt was cancelled | note on the transfer | `The purchase order <link to the order> this receipt is linked to was cancelled.` |
| A drop shipping purchase order was split into one transfer per sales order | note on each created transfer | a link back to the purchase order |

---

## 7. Scheduled job

| Property | Value |
|---|---|
| Name | `Procurement: run scheduler` |
| Frequency | every 1 day |
| Runs as | the superuser account |
| Tasks | three, announced to the progress tracker: (1) recompute and run every automatic reordering rule; (2) reserve every waiting move whose reservation date has been reached; (3) merge duplicated stock quantity records |
| Batching | reordering rules in batches of one thousand, each with its own cursor and commit; move reservation in chunks of one thousand, each committed |
| Log lines | `A batch of <n> orderpoints is processed and committed`; `A batch of <n> moves are assigned and committed`; `Unable to process orderpoints` when a failing batch cannot be reduced |
| Failure | logged with its stack trace and re-raised, which aborts the run |

**Industry-standard completion.** A replacement must make the job re-entrant: two overlapping runs must not create two documents for the same reordering rule. Taking a row lock on each reordering rule for the duration of its request, and retrying a batch on a serialization failure, is sufficient (see `RP-RULE-303` and `RP-RULE-304`).

---

## 8. Request endpoints

This domain exposes no request endpoint of its own. Every operation of section 1 is invoked through the platform's generic record-and-operation transport, whose authentication level is "authenticated user" and whose access control is the access groups and record rules of `configuration.md`, sections 11 and 12. The screens of section 2 read and write records through the same transport.

---

## 9. Exported files

This domain exports no file of its own. The Replenishment screen deliberately disables spreadsheet export, because its rows are partly temporary records that are deleted as soon as the shortage they describe is covered. Every other list of this domain supports the platform's generic export.
