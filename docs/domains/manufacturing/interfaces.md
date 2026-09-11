# Manufacturing — Interfaces

Window actions and menus as user-visible navigation, views and what each shows, named
remote operations with their inputs and outputs, routes, reports and printable documents,
notification templates, external integrations, and import and export formats.

---

## 1. Navigation

### 1.1 The application menu

The root menu is **Manufacturing**, sequence 145, visible to the manufacturing user and
administrator groups. It contains five sections.

| Section | Sequence | Visible to |
|---|---|---|
| Operations | 10 | Manufacturing user |
| Planning | 15 | Manufacturing user |
| Products | 20 | Manufacturing user |
| Reporting | 25 | Manufacturing user |
| Configuration | 100 | Manufacturing administrator |

### 1.2 The complete menu tree

| Path | Opens | Additional visibility |
|---|---|---|
| Manufacturing → Operations → Manufacturing Orders | Manufacturing Orders | — |
| Manufacturing → Operations → Work Orders | Work Orders to do | *Manage Work Order Operations* |
| Manufacturing → Operations → Unbuild Orders | Unbuild Orders | — |
| Manufacturing → Operations → Scrap | Scrap records | — |
| Manufacturing → Planning → Planning by Production | Work Order planning grouped by order | *Manage Work Order Operations* |
| Manufacturing → Planning → Planning by Work Center | Work Order planning grouped by work centre | *Manage Work Order Operations* |
| Manufacturing → Planning → *Run scheduler* | The replenishment scheduler | Technical users only |
| Manufacturing → Products → Products | Product templates, filtered to goods, defaulting to storable | — |
| Manufacturing → Products → Product Variants | Product variants that are storable and are not kits | — |
| Manufacturing → Products → Bills of Materials | Bills of Materials | — |
| Manufacturing → Products → Lots/Serial Numbers | Lots and serial numbers | Lot group |
| Manufacturing → Reporting → Work Orders | Work Orders analysis | *Manage Work Order Operations* |
| Manufacturing → Reporting → Overall Equipment Effectiveness | Productivity logs analysis | *Manage Work Order Operations* |
| Manufacturing → Configuration → Settings | The settings form | System administrator |
| Manufacturing → Configuration → Operations | Operations | *Manage Work Order Operations* |
| Manufacturing → Configuration → Work Centers | Work centres | *Manage Work Order Operations* |

An additional entry **Manufacturings** is grafted into the Inventory transfers menu,
showing the Manufacturing Orders alongside the other operation types.

---

## 2. Window actions

| Action | Name | Entity | View modes | Domain | Default context |
|---|---|---|---|---|---|
| Manufacturing Orders | Manufacturing Orders | Manufacturing Order | list, kanban, form, calendar, pivot, graph, activity | the operation type is active | the *To Do* filter is preselected; the company defaults to the first allowed company |
| Manufacturing Orders (board) | Manufacturing Orders | Manufacturing Order | list, kanban, form | the operation type is the one clicked | the operation type is defaulted |
| Manufacturing Order form | Manufacturing Orders | Manufacturing Order | form | — | — |
| Manufacturings (transfers menu) | Manufacturings | Manufacturing Order | list, kanban, form, calendar, activity | — | the company defaults to the first allowed company |
| Manufacturings (board graph) | Manufacturings | Manufacturing Order | list, kanban, form, calendar, activity | — | the *Confirmed* filter is preselected |
| Bills of Materials | Bills of Materials | Bill of Materials | list, kanban, form | forced empty | the company defaults to the first allowed company |
| Recipes of a product template | Bill of Materials | Bill of Materials | — | the recipes of that template, or those listing it as a by-product | the template is defaulted |
| Recipes of a variant | Bill of Materials | Bill of Materials | — | forced empty | the variant is defaulted |
| Operations | Operations | Operation | list, kanban, form | — | — |
| Operations of a recipe | Operations | Operation | list, form | filtered on the recipe | the recipe is defaulted and the recipe field is hidden |
| Copy existing operations | Select Operations to Copy | Operation | list, form | the operation has no recipe, or its recipe is active | the target recipe is carried |
| Work Centers | Work Centers | Work Centre | list, kanban, form | — | — |
| Work Centers Overview | Work Centers Overview | Work Centre | kanban, form | — | — |
| Work Orders | Work Orders | Work Order | list, kanban, form, calendar, pivot, graph | — | the *To Do*, *In Progress* and *Blocked* filters are preselected |
| Work Orders of a work centre | Work Orders | Work Order | list, form, pivot, graph, calendar | the state is neither `done` nor `cancel` | grouped by the clicked work centre |
| Work Orders of an order | Work Orders | Work Order | list, form, calendar, pivot, graph | filtered on the order | — |
| Work Orders of an operation | Work Orders | Work Order | graph, pivot, list, form, calendar | the operation's recipe and state `done` | the *Done* filter is preselected |
| Work Orders Planning (by production) | Work Orders Planning | Work Order | list, form, calendar, pivot, graph | the production state is neither `done` nor `cancel` | grouped by order; the *To Do*, *Blocked* and *In Progress* filters are preselected |
| Work Orders Planning (by work centre) | Work Orders Planning | Work Order | list, form, calendar, pivot, graph | — | grouped by work centre; the same three filters; the work centre status is shown |
| Work Orders Analysis | Work Orders Analysis | Work Order | graph, pivot, list, form | — | grouped by work centre |
| Work Orders Performance | Work Orders Performance | Work Order | graph, pivot, list, form | the clicked work centre and state `done` | — |
| Work Center Loads | Work Center Loads | Work Order | graph, pivot | — | — |
| Overall Equipment Effectiveness (of a work centre) | Overall Equipment Effectiveness | Productivity Log | graph, pivot, list, form | the clicked work centre | the *This Month* filter is preselected |
| Overall Equipment Effectiveness (report) | Overall Equipment Effectiveness | Productivity Log | graph, pivot, list, form | — | grouped by work centre then loss reason; creation and editing disabled |
| Productivity Losses | Productivity Losses | Productivity Log | list, form, graph, pivot | — | the availability, performance and quality filters are preselected |
| Unbuild Orders | Unbuild Orders | Unbuild Order | list, kanban, form, activity | — | — |
| Stock Moves of an unbuild | Stock Moves | Stock Move Line | list, form | the line's move is the unbuild's produce move or its consume move | — |
| Inventory Moves of an order | Inventory Moves | Stock Move Line | list, form | the line's move is a component move or a finished move of the order | — |
| Change Quantity To Produce | Change Quantity To Produce | Change Production Quantity assistant | form | — | — |
| Consumption Warning | Consumption Warning | Consumption Warning assistant | form | — | the orders and the difference lines |
| Backorder question | You produced less than the initial demand | Backorder Confirmation assistant | form | — | the orders and one line per affected order |
| Assign Serial Numbers | Assign Serial Numbers | Serial Number Assignment assistant | form | — | the order, and the Work Order when opened from one |
| Split production | *(no title)* | Split Production assistant | form | — | the order |
| Split several productions | *(no title)* | Split Multiple Productions assistant | form | — | one line per order |
| Settings | Settings | Settings | form | — | the manufacturing section is opened |
| Products | Products | Product Template | kanban, list, form | — | goods filter preselected, storable defaulted |
| Product Variants | Product Variants | Product Variant | kanban, list, form | storable and not a kit | — |

---

## 3. Views

### 3.1 Manufacturing Order — form

**Header.** The state indicator runs draft → confirmed → progress → to close → done.

**Buttons in the header**, with their visibility:

| Button | Named operation | Shown when |
|---|---|---|
| Confirm | `action_confirm` | the order is `draft` |
| Plan | `button_plan` | the order has Work Orders and is not planned |
| Unplan | `button_unplan` | the order is planned |
| Check availability | `action_assign` | reservation is possible |
| Unreserve | `do_unreserve` | unreservation is possible |
| Start | `action_start` | the order is `confirmed` |
| Produce All | `button_mark_done` | the order is running and the producing quantity is zero or the full quantity |
| Produce | `button_mark_done` | the order is running and the producing quantity is a partial amount |
| Cancel | `action_cancel` | the order is not `done` |
| Update BoM | `action_update_bom` | the recipe is flagged outdated |
| Unbuild | `button_unbuild` | the order is `done` |
| Generate Serial / Generate Lot | `action_generate_serial` | the product is tracked and no number is set |
| Clear | `action_clear_lot_producing_ids` | producing lots are set |
| Allocation | `action_view_reception_report` | the allocation group holds and allocation is possible |
| Catalog (components) | `action_add_from_catalog_raw` | the component list is editable |
| Catalog (by-products) | `action_add_from_catalog_byproduct` | the by-product list is editable and the by-products group holds |
| Details (on a move line) | `action_show_details` | always |
| Lock / Unlock | `action_toggle_is_locked` | the lock control is visible |

**Statistic buttons.** Source orders, generated orders, backorders, transfers, unbuilds,
scraps, serial numbers, product forecast, and — with manufacturing accounting — the
valuation and the work-in-progress entries.

**Body.** Product, quantity to produce and unit, recipe, responsible, deadline, start and
finish, operation type, source and destination locations, the never-variant attribute
values, the components list, the by-products list, the Work Orders list, the finished
product's lots, a miscellaneous page with the origin, the company, the propagate flag and
the extra unit cost, and the discussion thread.

**Row indicators.** A component row shows the demanded quantity, the quantity to consume,
the consumed quantity, the unit, the reserved state and the manual-consumption flag. A
delayed order shows the delay popover; a Work Order row shows its own popover.

### 3.2 Manufacturing Order — list, kanban, calendar, pivot, graph, activity

- **List.** Reference, start date, product, quantity, unit, responsible, component status,
  readiness, state, and the delay indicator. Rows are coloured by lateness.
- **Kanban.** Grouped by state by default, showing the reference, the product, the quantity
  and the readiness.
- **Calendar.** Positioned on the start date.
- **Pivot and graph.** Measures over the quantity to produce, the produced quantity, the
  expected duration and the real duration.
- **Activity.** The scheduled activities per order.

### 3.3 Manufacturing Order — search

| Filter | Domain |
|---|---|
| To Do | state in draft, confirmed, progress, to close |
| Draft | state is draft |
| Confirmed | state is confirmed |
| Planned | the order is planned |
| In Progress | state is progress |
| To Close | state is to close |
| Done | state is done |
| Cancelled | state is cancel |
| Unbuilt | an unbuild of the order is done |
| Starred | priority is urgent |
| MO Pending | readiness is waiting |
| MO Ready | readiness is assigned |
| Components Available | the component availability state is available |
| Late Availability | the component availability state is late |
| Late | the start date is before now and the state is confirmed |
| Delayed Productions | a delay alert date exists, or the order is delayed |
| My MOs | the responsible is the current user |
| Before / Yesterday / Today / Tomorrow / The day after tomorrow / After | the date category of the start date |
| Date | a date range on the start date |
| Date: Last 365 Days | a hidden filter used by the "manufactured" statistic |
| My Activities / Late Activities / Today Activities / Future Activities / Warnings | the shared activity filters |

| Grouping | Field |
|---|---|
| Product | the product |
| Status | the state |
| Material Availability | the readiness |
| Date | the start date |

### 3.4 Bill of Materials — form

Product template, variant, reference, kind, quantity and unit, operation type, company,
flexible consumption, manufacturing readiness, manufacturing lead time, days to prepare
(with a **Compute** control), batch size and its switch, operation dependencies.

Three lists: components, by-products (with the by-products group) and operations (with the
Work Order Operations group). Each component row shows the component, the quantity, the
unit, the consuming operation, the variant restriction, the attachment count and the
sub-recipe indicator.

Controls: **Catalog** on the component and by-product lists, **Copy Existing Operations**
and **Add a line** on the operations list, and the recipe structure report.

### 3.5 Bill of Materials — search

| Filter | Domain |
|---|---|
| Manufacturing | kind is `normal` |
| Kit | kind is `phantom` |
| Archived | the recipe is archived |

| Grouping | Field |
|---|---|
| Product | the product template |
| *(unnamed)* | the recipe kind |
| *(unnamed)* | the default unit of measure |

### 3.6 Work Order — form, list, calendar, gantt-like planning, pivot, graph

**Form.** Name, order, product, work centre, operation, expected and real duration, the
duration deviation, the progress, the planned window, the produced quantity, the carried
quantity, the ready quantity, the predecessors and successors, the component move lines to
track, the time logs, and the working users. A **View WorkOrder** control opens the
shop-floor form.

**Search filters.**

| Filter | Domain |
|---|---|
| To Do | state is ready |
| Blocked | state is blocked |
| In Progress | state is progress |
| Finished | state is done |
| Cancelled | state is cancel |
| Late | the start is before now and the state is ready (planning view) or the start is at or before today (analysis view) |
| Start Date | a date range on the planned start |

| Grouping | Field |
|---|---|
| Work Center | the work centre |
| Manufacturing Order | the order |
| Product | the product |
| Status | the state |
| Date | the planned start |

The work centre grouping expands over every readable work centre, so an empty work centre
still shows its column.

### 3.7 Work Centre — form, list, kanban

**Form.** Name, code, tags, working schedule, time efficiency, setup and cleanup times,
hourly cost, expense account and analytic distribution (with manufacturing accounting),
alternative work centres, per-product capacities, the description, and the effectiveness
figures: blocked time, productive time, overall equipment effectiveness with its target,
and performance.

**Kanban.** One card per work centre showing the status colour, the load, the counts of
Work Orders to do, in progress, pending and late, and the weekly load graph. A blocked work
centre is marked with a red circle in grouped displays.

**Search.**

| Filter | Domain |
|---|---|
| Archived | the work centre is archived |
| *(grouping)* Company | the company |

### 3.8 Operation — form and list

Name, work centre, recipe, sequence, duration computation mode, the batch size for computed
mode, the manual duration, the computed cycle duration, the repetitions, the total
duration, the cost, the cost mode, the variant restriction, and the predecessors and
successors when dependencies are allowed.

**Search.**

| Filter | Domain |
|---|---|
| Archived | the operation is archived |
| *(grouping)* Bill of Material | the recipe |
| *(grouping)* Workcenter | the work centre |

A dedicated list view is used when copying operations into another recipe, offering
**Copy selected operations**.

### 3.9 Unbuild Order — form, list, kanban

Reference, product, quantity, unit, recipe, source order, lot, source and destination
locations, state, and the two read-only move lists. The **Unbuild** control validates.

**Search.**

| Filter | Domain |
|---|---|
| Draft | state is draft |
| Done | state is done |
| My Activities / Late Activities / Today Activities / Future Activities | the shared activity filters |

| Grouping | Field |
|---|---|
| *(unnamed)* | the product |
| Manufacturing Order | the source order |

### 3.10 Productivity Log — list, form, graph, pivot

Work centre, Work Order, order, user, loss reason, effectiveness category, start, end and
duration.

**Search.**

| Filter | Domain |
|---|---|
| Availability Losses | the category is availability |
| Performance Losses | the category is performance |
| Quality Losses | the category is quality |
| Fully Productive | the category is productive |
| Date | a date range on the start |
| *(grouping)* User | the user |
| *(grouping)* *(unnamed)* | the work centre |
| *(grouping)* Loss Reason | the loss reason |

### 3.11 Views added to other entities

| Entity | Addition |
|---|---|
| Product template and variant | Statistic buttons for the recipes, the recipes in which the product is used, and the quantity manufactured over the last 365 days; the manufacturing route; the **Compute price from recipe** control (with manufacturing accounting). |
| Operation type | The manufacturing counters, the component-lot switch and the automatic-printing switches. |
| Warehouse | The manufacturing step configuration, the resupply flag, and — with subcontracting — the subcontractor resupply flag. |
| Reordering rule | The recipe column and its placeholder, shown when the effective route is a manufacture route. |
| Transfer | The kit indicator, the count of generated orders, and the control that opens them; with subcontracting, the **Record components** control and the count of source purchase orders. |
| Stock Move | Dedicated detail forms for a component move (titled *Components*, source location hidden, manual consumption forced) and for a by-product move (titled *Move Byproduct*, destination location and reserved quantity hidden). |
| Scrap | The order and Work Order links and the kit selector. |
| Rule | The manufacture action and its explanatory sentence. |
| Analytic account | Statistic buttons for the Manufacturing Orders, the recipes and the Work Orders that reference it. |
| Journal entry | With manufacturing accounting, a statistic button for the work-in-progress orders the entry covers. |

---

## 4. Named remote operations

These are the operations a client or an integration invokes by name. Their inputs and
outputs are described in terms of entity references and plain values; none of them is
described as code.

### 4.1 On the Manufacturing Order

| Operation | Inputs | Output | Effect |
|---|---|---|---|
| `action_confirm` | the orders | true | Confirms them (see [workflows.md](workflows.md) §4.1). |
| `action_assign` | the orders | true | Reserves the component moves. |
| `do_unreserve` | the orders | nothing | Unreserves the component and finished moves that are not by-product moves. |
| `button_plan` | the orders | true | Confirms the drafts and plans the Work Orders. |
| `button_unplan` | the orders | nothing | Unplans, or refuses. |
| `action_plan_with_components_availability` | the orders | nothing | Moves the start to the latest component forecast date and plans. |
| `action_start` | one order | nothing | Moves a confirmed order to `progress`. |
| `set_qty_producing` | one order | nothing | Redistributes the producing quantity, picking manual-consumption moves. |
| `action_generate_serial` | one order, optionally a Work Order | nothing, a print action, or the serial-assignment action | Generates or asks for the producing lots. |
| `pre_button_mark_done` | the orders | true, or an assistant action | Runs the checks of [workflows.md](workflows.md) §5.3. |
| `button_mark_done` | the orders, optionally the list of orders to backorder | true, a print action, or a navigation action | Closes the orders. |
| `action_cancel` | the orders | true | Cancels them, or refuses. |
| `action_toggle_is_locked` | one order | true | Flips the locked flag. |
| `action_split` | the orders | an assistant action | Opens the split assistant. |
| `action_merge` | the orders | a navigation action to the merged order | Merges them, or refuses. |
| `action_generate_bom` | one order | a pre-filled recipe form action | Opens a new recipe built from the order. |
| `action_update_bom` | the orders | nothing | Applies the updated recipe and clears the outdated flag. |
| `button_unbuild` | one order | an unbuild form action | Opens a pre-filled Unbuild Order. |
| `button_scrap` | one order | a scrap form action | Opens a pre-filled scrap. |
| `action_view_reception_report` | the orders | the allocation report action | — |
| `action_view_mrp_production_childs` / `_sources` / `_backorders` | one order | a navigation action | Shows the related orders. |
| `action_view_mo_delivery` | one order | a navigation action | Shows the related transfers. |
| `action_see_move_scrap` | one order | a navigation action | Shows the scraps. |
| `action_view_mrp_production_unbuilds` | one order | a navigation action | Shows the unbuilds. |
| `action_view_serial_numbers` | the orders | a navigation action | Shows the producing lots, read only. |
| `action_clear_lot_producing_ids` | the orders | nothing | Clears the lots, zeroes the producing quantity and redistributes. |
| `action_product_forecast_report` | one order | the forecast report action | Shows the product forecast at the order's warehouse, highlighting this order's finished move. |
| `action_open_label_layout` / `action_open_label_type` | the orders | an assistant action | Chooses and prints labels. |
| `action_view_move_wip` | one order | a navigation action | With manufacturing accounting: the work-in-progress entries. |

### 4.2 On the Work Order

| Operation | Inputs | Output | Effect |
|---|---|---|---|
| `button_start` | the Work Orders, optionally a flag tolerating an invalid state | nothing | Starts them and opens a timer. |
| `button_pending` | the Work Orders | nothing | Closes the acting user's timer. |
| `button_finish` | the Work Orders | true | Finishes them (see [state-machines.md](state-machines.md) §3.3). |
| `action_mark_as_done` | the Work Orders | nothing | Finishes them and fills a zero real duration with the expected duration. |
| `action_cancel` | the Work Orders | the write result | Cancels them. |
| `set_state` | the Work Orders, a target state | nothing | Moves them to that state through the correct path. |
| `action_replan` | the Work Orders | true | Replans the ready and blocked Work Orders of their orders. |
| `button_unblock` | the Work Orders | true | Unblocks their work centres. |
| `end_previous` | the Work Orders, a flag | true | Closes the acting user's open timer, or every open timer. |
| `end_all` | the Work Orders | true | Closes every open timer. |
| `get_working_duration` | one Work Order | minutes | The additional duration of the still-open timers. |
| `get_duration` | one Work Order | minutes | The merged real duration. |
| `button_scrap` / `action_see_move_scrap` / `action_open_wizard` | one Work Order | a navigation action | — |

### 4.3 On the Bill of Materials

| Operation | Inputs | Output | Effect |
|---|---|---|---|
| `action_compute_bom_days` | the recipes | nothing, or a notice | Recomputes the days to prepare. |
| `action_copy_existing_operations` | one recipe | a navigation action | Opens the operations to copy. |
| `action_open_operation_form` | one recipe | a navigation action | Opens a new operation for that recipe. |
| `action_set_bom_on_orderpoint` | one recipe | the replenishment-information action | Sets the recipe (and, if needed, the manufacture route) on the reordering rule named by the caller, raising the quantity to order to the recipe quantity when it is lower. |
| `action_archive` / `action_unarchive` | the recipes | the result | Also archives or unarchives the operations. |
| `get_import_templates` | — | one entry: the label "Import Template for Bills of Materials" and the path of the spreadsheet template | Offers the import template. |

### 4.4 On the Unbuild Order

| Operation | Inputs | Output | Effect |
|---|---|---|---|
| `action_validate` | one Unbuild Order | nothing, or the insufficient-quantity assistant action | Checks the stock and unbuilds. |
| `action_unbuild` | one Unbuild Order | the write result | Performs the unbuild. |

### 4.5 On the Work Centre

| Operation | Inputs | Output | Effect |
|---|---|---|---|
| `unblock` | one work centre | true | Ends every open time log, or refuses. |
| `action_show_operations` | one work centre | a navigation action | The operations that use it. |
| `action_work_order` | the work centres | a navigation action | The Work Orders. |
| `action_work_order_alternatives` | one work centre | a navigation action | The Work Orders of its alternatives and of the work centres that name it as an alternative. |
| `_get_first_available_slot` | one work centre, a start instant, a duration in minutes, a direction, leaves to ignore, extra occupied intervals | a start and an end instant, or a failure and the text "No available slot 700 days after the planned start" | The scheduling primitive of [calculations.md](calculations.md) §12. |
| `_get_unavailability_intervals` | the work centres, a start and an end instant | per work centre, the list of unavailable intervals | Used by the planning displays. |

### 4.6 On the report entities

| Operation | Inputs | Output |
|---|---|---|
| Recipe structure report `get_html` | a recipe reference, a quantity (default 1), a variant reference | the recursive structure, plus the flag saying whether any level has attachments |
| Recipe structure report `get_warehouses` | — | the identifier, the name and the manufacturing operation type of every warehouse of the allowed companies |
| Order overview report `get_report_values` | an order reference | the report data and a display context stating whether units of measure are shown |

### 4.7 Assistant operations

| Assistant | Operation | Effect |
|---|---|---|
| Change Production Quantity | `change_prod_qty` | Rescales the order ([workflows.md](workflows.md) §6). |
| Backorder Confirmation | `action_close_mo` | Closes without backordering, except the forced orders. |
| Backorder Confirmation | `action_backorder` | Closes, backordering the ticked orders and the forced ones. |
| Consumption Warning | `action_confirm` | Closes with the consumption check skipped. |
| Consumption Warning | `action_set_qty` | Resets the consumed quantities to the expected ones and then closes. |
| Consumption Warning | `action_cancel` | Returns to the order's form when opened from a Work Order. |
| Split Production | `action_split` | Splits and applies the responsible and date of each detail line. |
| Split Production | `action_prepare_split` / `action_return_to_list` | Navigates within the multi-split assistant. |
| Serial Number Assignment | `action_generate_serial_numbers` | Fills the text area. |
| Serial Number Assignment | `action_apply` | Assigns the numbers to the order. |
| Serial Number Assignment | `action_split_and_assign_serials` | Splits the order one unit per number. |
| Insufficient Unbuild Quantity | `action_done` | Performs the unbuild anyway. |
| Label Type | `process` | Opens the product-label or lot-label layout. |
| Work-in-progress accounting | `confirm` | Posts the entry and its reversal. |

---

## 5. Routes

The core of the domain exposes **no route of its own**: it is a back-office domain. Two
route groups exist around it.

### 5.1 Product document upload

The shared product-document upload route is extended: when the caller marks the upload as
attached on a recipe, the created document records "attached on manufacturing" = `bom`, so
it appears in the recipe's attachments.

### 5.2 The subcontracting portal

Installed with the subcontracting capability. All routes require an authenticated user and
are rendered as website pages.

| Path | Method | Authentication | Purpose |
|---|---|---|---|
| `/my/productions` and `/my/productions/page/<page number>` | read | authenticated user | Lists the receipts whose partner's commercial partner is the visitor's commercial partner and that carry at least one subcontract move. Supports a creation-date range, the sortings *Newest* (creation date descending, then identifier descending) and *Name* (name ascending, then identifier ascending), and the filters *All*, *Done* (the transfer is done) and *Ready* (the transfer is assigned). Paged by the shared page size. |
| `/my/productions/<transfer identifier>` | read | authenticated user | Renders the subcontracting portal page for one receipt. Access is checked through the shared document-access rule; a failure returns *not found*, never an access error, so that the existence of the document is not leaked. |
| `/my/productions/<transfer identifier>/subcontracting_portal` | read | authenticated user | Renders the embedded back-office form for that receipt, with the session forced to the receipt's company as the only allowed company, and the action fixed to the subcontracting portal production action. |

The portal home page gains a **production count** counter: the number of receipts of the
visitor's commercial partner carrying a subcontract move.

---

## 6. Reports and printable documents

| Report | Entity | Format | File name | Available from |
|---|---|---|---|---|
| Production Order | Manufacturing Order | portable document format | "Production Order - *the order reference*" | The order's print menu |
| BoM Overview | Bill of Materials | portable document format | "Bom Overview - *the recipe display name*" | The recipe's print menu |
| MO Overview | Manufacturing Order | portable document format | "MO Overview - *the order display name*" | The order overview screen |
| Work Order | Work Order | portable document format | "Work Order - *the work order name*" | The Work Order's print menu |
| Finished Product Label (portable document format) | Manufacturing Order | portable document format | "Finished products - *the order reference*" | The order's print menu |
| Finished Product Label (label printer) | Manufacturing Order | plain text for a label printer | — | The order's print menu |

Two further printable documents are reached indirectly: the allocation report and the
transfer labels, both belonging to the inventory domain, and the lot labels, belonging to
the inventory domain as well.

### 6.1 The Production Order document

Sections, in order:

1. **Header** — the order reference, the state, the source document, the responsible, the
   product, the quantity to produce with its unit, the deadline and the planned dates, and
   a barcode of the reference.
2. **Products to consume** — one row per component: the product, the quantity demanded with
   its unit, the source location, and, for a tracked component, the lots. Components coming
   from a nested kit are grouped under the kit's name.
3. **Operations** — with the Work Order Operations group: one row per Work Order with its
   name, its work centre, its expected duration and its planned window.
4. **By-products** — with the by-products group: one row per by-product with its quantity
   and unit.
5. **Finished product** — the product, the produced quantity and the lots.

The label-printer variant of the finished-product label prints the product name, the
reference and a barcode per produced unit.

### 6.2 The recipe structure report

The interactive version is a foldable tree; the printable version flattens it to a list
whose visibility follows the chosen unfolded levels.

Each line carries: the name, the line kind (recipe, component, operation or by-product),
whether the product is storable, the quantity, the free quantity, the on-hand quantity, the
producible quantity, the unit, the cost, the resupply route name and detail, the route
alert flag, the lead time, the manufacturing lead time, the level, the recipe code, the
availability state and its text, and a status text such as "*n* To Manufacture" or
"*n* Ready To Produce".

Two aggregate lines are inserted per level when they apply: an **Operations** line carrying
the total minutes and the total operations cost, and a **Byproducts** line carrying the
total by-product quantity and the total by-product cost.

The report offers a quantity, a variant, a warehouse and a display mode (*overview* or
*forecast*).

### 6.3 The order overview report

Rows: the order itself, then its components (recursively, with a nested order shown as its
own sub-tree), then its operations, then its by-products, then a cost breakdown when the
order is done and has by-products.

Columns, each switchable in the printable version: the name, the quantity, the unit, the
replenishment document, the free-to-use and on-hand quantities, the reserved quantity, the
receipt date, the unit cost, the order cost, the recipe cost and the real cost. Each cost
may carry a comparison indicator: *danger* when the current value exceeds the expected one,
*success* when it is below.

The footer shows the unit order cost, the unit recipe cost and the unit real cost, each
being the corresponding total divided by the quantity (or by 1 when the quantity is zero).

---

## 7. Notifications and message templates

### 7.1 Message subtypes

Five subtypes on the Manufacturing Order, none subscribed to by default: MO Confirmed, MO
Progress, MO To Close, MO Done, MO Cancelled. Each state transition posts under its own
subtype, so a follower can subscribe to just the transitions of interest.

### 7.2 Tracked fields

The order's thread logs every change of the quantity to produce, the state and the
readiness. The tracked fields are sorted topologically before logging, so a computed field
is logged after the field it depends on.

### 7.3 The manufacturing exception template

Rendered as a warning panel. Its content:

> Exception(s) occurred on the manufacturing order(s): *a link to the order*. Manual actions
> may be needed.
>
> Exception(s):
> - *a link to the order*: *the new quantity* *the unit name* of *the product name*
>   **cancelled** *(when the exception is a cancellation)*
> - *a link to the order*: *the new quantity* *the unit name* of *the product name*
>   **ordered instead of** *the old quantity* *the unit name* *(otherwise)*
>
> Impacted Transfer(s): *(only when the exception is not a cancellation and impacted
> transfers exist)*
> - *a link to each impacted transfer*

It is posted as an activity on the upstream or downstream document, addressed to the
document's responsible, or to the product's responsible where the grouping uses that.

### 7.4 The move-change template

When the lot, source location or quantity of a **done** move line of an order is changed,
a message is posted on the order beginning with either

> **Consumed quantity has been updated.**

for a component move, or

> **Produced quantity has been updated.**

for a finished move, followed by the shared move-change body describing the old and new
values.

### 7.5 Origin-link notes

An automatically created order posts one of:

- "This production order has been created from Replenishment Report." — when the reordering
  rule was created by the system with a manual trigger;
- an origin-link note pointing at the reordering rule;
- an origin-link note pointing at the order whose component move caused this one.

### 7.6 Merge and unbuild notes

- Merging posts on each merged order: "This production has been merge in *the surviving
  order*".
- Unbuilding posts on the source order: "*the quantity* *the unit name* unbuilt in *a link
  to the unbuild order*", as an internal note.

### 7.7 Notices

| Notice | When |
|---|---|
| "Cannot compute days to prepare due to missing route info for at least 1 component or for the final product." | Computing the days to prepare when the structure is unavailable. |
| "Note that archived work center(s): '*names*' is/are still linked to active Bill of Materials, …" | Archiving a referenced work centre. Sticky. |
| "Note that product(s): '*names*' is/are still linked to active Bill of Materials, …" | Archiving a referenced component. Sticky. |
| "The following replenishment order has been generated" with a link to the order | A replenishment that created an order. |

### 7.8 Digest tip

One tip for the manufacturing user group: "Tip: Use tablets in the shop to control
manufacturing".

---

## 8. Import and export

### 8.1 The recipe import template

The recipe entity offers one import template, labelled **Import Template for Bills of
Materials**, served from the domain's static spreadsheet folder. It contains the columns a
recipe and its component lines need: the product, the reference, the quantity, the unit,
the recipe kind, and, per line, the component, the quantity and the unit.

### 8.2 Generic import and export

Every entity of the domain is importable and exportable through the shared mechanism.
Two characteristics matter for a reimplementation:

- A recipe cannot be created by name alone. An import that supplies only a name for the
  recipe relation fails with **"You cannot create a new Bill of Material from here."**
  unless the import context supplies a default product template, in which case the typed
  text becomes the recipe's reference.
- A Manufacturing Order's reference is readonly and is allocated from the operation type's
  sequence, so an import that supplies a reference has it honoured only when the value is
  neither empty nor the literal text "New".

### 8.3 The product catalogue data contract

Recipes and Manufacturing Orders both implement the shared product-catalogue contract, so a
user can add components from a catalogue panel.

| Contract element | Recipe | Manufacturing Order |
|---|---|---|
| Price shown per product | the product's standard price | the product's standard price |
| Currency | the active company's currency | the active company's currency |
| The list a product is added to | the component lines or the by-product lines, chosen by the caller | the component moves or the by-product moves, chosen by the caller |
| The quantity reported for a product | the sum of the quantities of the lines for that product, converted into the line unit | the sum of the demands of the moves for that product |
| Read-only when | more than one line exists for that product | more than one move exists for that product |
| Setting a quantity to zero | removes the lines | removes the moves |
| Product domain | the shared catalogue domain | the shared catalogue domain restricted to goods |
| Stock shown | no | yes |
| Search flags | a product already on the recipe is flagged | a product already on the order is flagged |

---

## 9. External integrations

The domain integrates with no external service. Its only outward contracts are:

| Contract | Direction | Description |
|---|---|---|
| The subcontracting portal | inbound | An external partner with portal access records the components consumed and the lots produced on the orders whose subcontractor is that partner. Restricted by record rules and by the writeable-field list. |
| The purchase side of subcontracting | inbound and outbound | A purchase order and its vendor bill supply the subcontracting service cost; the receipt drives the order. |
| The label printer format | outbound | The finished-product label and the lot label may be emitted as plain text for a label printer instead of as a portable document. |
| The barcode of a Work Order | outbound | The order reference, a slash and the Work Order identifier, so a shop-floor scanner can identify a Work Order. |
| The barcode of a manufacturing location or operation type | outbound | The warehouse code followed by `MANUF`, `PC`, `SFP`, `RESUP`, `PREPRODUCTION` or `POSTPRODUCTION`. |

---

## 10. Display and formatting rules

| Entity | Display rule |
|---|---|
| Bill of Materials | The reference followed by ": " when set, then the product template's display name; suffixed with " (*quantity* *unit name*)" in a quantity-aware context when the quantity is above one or the unit differs from the product's own. |
| Bill of Materials Line | The component's name. |
| Bill of Materials By-Product | The by-product's name. |
| Operation | The operation's name. |
| Work Centre | The resource's name, suffixed with two non-breaking spaces and a red circle when grouped with the status shown and the work centre is blocked. |
| Productivity Loss Category | The category value with its first letter capitalised. |
| Productivity Log | The loss reason's name. |
| Manufacturing Order | The reference. |
| Work Order | "*the order reference* - *the work order name*", prefixed with "*the product name* - " in a product-prefixed context. |
| Unbuild Order | The reference. |
| Operation line in the structure report | "*the operation name* - *the work centre name*", with the unit shown as "Minutes". |

Quantities are displayed at the shared "Product Unit" decimal precision. Durations are
displayed in minutes, with two decimals for the expected duration and the duration per unit.
Percentages are displayed as integers for the duration deviation and the performance, and
with two decimals for the overall equipment effectiveness and the progress.
