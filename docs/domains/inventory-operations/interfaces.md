# Inventory Operations — Interfaces

Everything this domain exposes: the navigation a person sees, the screens and what each one shows, the operations that can be called remotely with their inputs and outputs, the routes, the printable documents, the notifications, and the import and export formats.

> **Reproduced literals.** A few strings in this file are reproduced exactly as the system emits them — error messages, selection labels, generated record names — and therefore keep abbreviations that this specification would otherwise spell out. They are: `UoM` for unit of measure, `SN` for serial number, `ZPL` for the Zebra printer command language, `PDF` for Portable Document Format, `GS1` for Global Standards One, and the suffix `(MTO)` for make to order, that is the supply method this specification calls *advanced* or *trigger another rule*. Wherever such a string is quoted, the quotation is verbatim and must be reproduced character for character.

---

# 1. Navigation

The domain contributes one top-level application entry, "Inventory", placed at sequence 140.

| Level 1 | Level 2 | Level 3 | Opens | Visible to |
|---|---|---|---|---|
| Inventory | *(root)* | | The operation overview: one card per Operation Type the reader has pinned or that belongs to their companies | inventory user |
| Inventory | Operations | Transfers | Every Transfer | inventory user |
| Inventory | Operations | Transfers → Receipts | Transfers of kind receipt | inventory user |
| Inventory | Operations | Transfers → Deliveries | Transfers of kind delivery | inventory user |
| Inventory | Operations | Transfers → Internal | Transfers of kind internal transfer | inventory user |
| Inventory | Operations | Adjustments → Physical Inventory | The quantity records of internal and transit Locations, in counting mode | inventory user |
| Inventory | Operations | Procurement | The replenishment screens of `../replenishment-and-procurement/` | inventory user |
| Inventory | Products | Products | Product templates with their stock figures | inventory user |
| Inventory | Products | Product Variants | Product variants with their stock figures | inventory user |
| Inventory | Products | Lots / Serial Numbers | Lots | lot group |
| Inventory | Products | Packages | Containers | container group |
| Inventory | Products | Locations | Quantity records grouped by Location | multi-location group |
| Inventory | Reporting | Stock | The quantity records, extended with pivot and graph views | inventory manager |
| Inventory | Reporting | Moves Analysis | Stock Moves | inventory manager |
| Inventory | Reporting | Moves History | Stock Move Lines | inventory manager |
| Inventory | Configuration | Settings | The settings screen | inventory manager |
| Inventory | Configuration | Warehouse Management → Warehouses | Warehouses | inventory manager, multi-warehouse group |
| Inventory | Configuration | Warehouse Management → Operations Types | Operation Types | inventory manager |
| Inventory | Configuration | Warehouse Management → Locations | Locations | inventory manager, multi-location group |
| Inventory | Configuration | Warehouse Management → Routes | Routes | inventory manager, advanced-routing group |
| Inventory | Configuration | Warehouse Management → Rules | Stock Rules | inventory manager, advanced-routing group |
| Inventory | Configuration | Warehouse Management → Putaway Rules | Put-away Rules | inventory manager, multi-location group |
| Inventory | Configuration | Warehouse Management → Storage Categories | Storage Categories | inventory manager, multi-location group |
| Inventory | Configuration | Products → Product Categories | Product categories | inventory manager |
| Inventory | Configuration | Products → Attributes | Product attributes | inventory manager, variant group |
| Inventory | Configuration | Products → Units & Packagings | Units of measure | inventory manager |
| Inventory | Configuration | Products → Barcode Nomenclatures | Barcode nomenclatures | inventory manager |
| Inventory | Configuration | Delivery → Package Types | Package Types | inventory manager, container group |

The batch capability adds, under Operations: **Batch Transfers** and **Wave Transfers**, and contributes two counters to each Operation Type card.

---

# 2. Screens

## 2.1 The operation overview

One card per Operation Type. Each card shows the display name ("*warehouse name*: *type name*" when the type belongs to a Warehouse), a colour, and the counters of `entities.md`, section 5.4:

| Element | Meaning | Opens |
|---|---|---|
| Main figure | Transfers to process (ready, waiting or waiting-another-operation) | The Transfers of that type, filtered to those states |
| Ready | Transfers whose status is ready | Filtered list |
| Waiting | Transfers whose status is waiting or waiting-another-operation | Filtered list |
| Late | Transfers to process whose scheduled date is before today, or which are flagged late | Filtered list |
| Back Orders | Open Transfers that have a back-order link | Filtered list |
| Draft | Transfers in draft | Filtered list |
| Ready moves | Stock Moves of that type whose status is assigned | Move list |
| Bar chart | Six buckets — Before, Yesterday, Today, Tomorrow, The day after tomorrow, After — counting the Transfers to process by scheduled date | Clicking a bucket opens the Transfers of that date category |

A star toggles the card's presence in the reader's own overview. When every bucket is zero, the chart is shown as sample data and is not clickable.

## 2.2 Transfer screen

**Header buttons**, each shown under the stated condition:

| Button | Shown when |
|---|---|
| Mark as Todo (confirm) | the status is draft |
| Validate | the status is not draft, done or cancelled |
| Check Availability | the availability indicator is true |
| Unreserve | something is reserved |
| Return | the Transfer is done |
| Print | always |
| Cancel | the status is neither done nor cancelled |
| Unlock / Lock | always; the icon reflects the lock flag |
| Put in Pack | the container group is active and the status is neither done nor cancelled |
| Add entire packages | the container group is active, the kind is not receipt, and the status is neither done nor cancelled |
| Scrap | the status is neither draft nor cancelled |
| Allocation | the reception-report indicator is true |
| Split | the status is neither done nor cancelled |
| Next Transfers | the Transfer's moves have destination moves in other Transfers |
| Returns (counter) | the Transfer has returns |
| Packages (counter) | the Transfer has containers |

**Fields.** The reference as the title; the contact, the Operation Type, the source and destination Locations, the scheduled date, the deadline, the source document, the shipping policy, the responsible, the owner, the priority star, the properties, the availability text with its state colour, the delay popover, the instruction text from the contact, and the signature.

**Move list.** One row per Stock Move: product, description, demand, processed quantity, unit, packaging quantity, forecast indicator, the lot text box or lot selector (per the three indicators of `entities.md`, section 7.3), the picked tick, and the detail button when the detail indicator is true. When the Operation Type asks to show detailed operations, the detail lines are listed instead of the moves.

**Detail screen of one move.** Source Location, destination Location, lot, source container, destination container, owner, quantity, unit, picked tick, plus the quantity-record picker that fills all of those at once. Buttons to generate serial numbers and to paste a list of lot names.

**Discussion thread.** The status is tracked; the Operation Type and the responsible are tracked; the scheduled date is tracked. Notes are posted for backorder creation, for demand and detail-line changes, for signature, for batch validation and for deadline propagation.

## 2.3 Physical inventory screen

An editable list of quantity records restricted to internal and transit Locations, opened in counting mode. Columns: product, Location, lot, container, owner, on-hand quantity, unit, counted quantity, difference, scheduled date, assignee, last count date, the outdated indicator and the duplicated-serial-number indicator.

Filters offered: internal Locations, transit Locations, my counts, to count (scheduled on or before today), starred products, negative quantities, conflicts (outdated records), and by product, Location, lot, container, owner and product category.

Row actions: set current quantity, clear, apply, request a count, relocate, view history, view moves, view reordering rules. List actions: apply all (asks for a reference label first), request a count, relocate, import from a spreadsheet.

When the reader is not a manager, the screen is pre-filtered to their own assigned counts.

## 2.4 Quantity screens outside counting mode

The same records without the counted-quantity columns, with the available quantity shown, and with pivot and graph views when opened from the reporting menu. Opening them runs the housekeeping pass.

## 2.5 Location screen

Name, parent, type, company, barcode, replenishment flag, removal strategy, storage category, counting frequency, last count, next expected count, net and forecasted weight, and the emptiness indicator. Related lists: current stock, put-away rules, and — with the maintenance bridge — the equipment stored there.

## 2.6 Operation Type screen

Grouped as: identity (name, kind, Warehouse, sequence prefix, reference sequence, barcode, colour, company, return type); locations (default source, default destination); behavior (shipping policy, reservation method and its two day counts, backorder policy, create and use lots, show detailed operations, move entire packages, set package type); printing (the nine automatic-print switches with their two format choices); batching (automatic batches, the four batch grouping options, the three wave grouping options with their category and Location lists, the two limits, auto-confirm); dispatch (dispatch management, docks); and the property definitions.

## 2.7 Warehouse screen

Name, short name, address, company, then the two step configurations with an inline diagram of the resulting flow, the resupply list, and buttons to open the Warehouse's Routes and its Locations.

## 2.8 Batch and wave screens

Name, description, responsible, Operation Type, scheduled date, state, the Transfer list (for a batch) or the detail-line list (for a wave), the estimated weight and volume, and — with dispatch management — the vehicle, its category, the driver, the dock, the end date and the two load percentages. Buttons: confirm, validate, cancel, print, put in pack, add operations, detailed operations, packages, allocation, merge.

## 2.9 Reception report screen

Grouped by the source document of each demand. Each group shows the document, its scheduled date, its contact and its priority; each line shows the product, the quantity, the unit, and one of three states: assignable (with an Assign button), already assigned (with an Unassign button), or expected but not assignable (a draft incoming move). Buttons to print the report and to print one label per allocated move.

## 2.10 Traceability screen

A tree of completed detail lines, expandable upstream and downstream, showing reference, date, product, lot, quantity, unit, source Location, destination Location and document. A print button produces the portable-document-format version through the route of section 4.

---

# 3. Named remote operations

Each entry gives the entity, the operation name as it is reproduced, its inputs beyond the record set it is called on, and its output.

## 3.1 Transfer

| Operation | Inputs | Output and effect |
|---|---|---|
| `action_confirm` | — | Confirms the draft moves and triggers the replenishment scheduler. Returns true. |
| `action_assign` | — | Confirms draft Transfers, then reserves the open moves in priority and deadline order. Returns true. Fails with "Nothing to check the availability for." when there is nothing to reserve. |
| `do_unreserve` | — | Unreserves the moves. |
| `button_validate` | Context keys: `skip_sanity_check`, `skip_backorder`, `skip_sms`, `picking_ids_not_to_backorder`, `button_validate_picking_ids`, `cancel_backorder` | Runs the validation algorithm. Returns true, or a window action (the backorder screen, the text-message warning, the Reception Report), or a multi-print client action. |
| `action_cancel` | — | Cancels the moves and locks the Transfer. Returns true. |
| `action_split_transfer` | — | Splits the Transfer into a backorder without validating. |
| `action_toggle_is_locked` | — | Flips the lock flag. Returns true. |
| `action_put_in_pack` | `package_id`, `package_type_id`, `package_name` (all optional, keyword only) | Creates or reuses a container and assigns it. Returns the container, a wizard action or a label action. |
| `action_add_entire_packs` | `package_ids` | Adds those containers and their descendants to the Transfer. Returns true or false. |
| `do_print_picking` | — | Marks the Transfer printed and returns the transfer document action. |
| `action_open_label_layout` | — | Returns the product-label wizard action. |
| `action_open_label_type` | — | Returns the label-kind chooser when lots are present, otherwise the product-label wizard. |
| `action_view_reception_report` | — | Returns the Reception Report action. |
| `action_see_returns`, `action_see_packages`, `action_see_package_histories`, `action_see_move_scrap`, `action_next_transfer`, `action_detailed_operations`, `action_picking_move_tree` | — | Each returns a window action onto the related records. |
| `button_scrap` | — | Returns the scrap screen action pre-filled with the Transfer and its products. |
| `calculate_date_category` | a date and time | Returns one of `before`, `yesterday`, `today`, `day_1`, `day_2`, `after`, or the empty string. |
| `date_category_to_domain` | a field name and a category | Returns the filter conditions for that category. |
| `get_empty_list_help` | a help message | Returns the rendered empty-list guidance for the reader's Operation Type kind. |
| `get_action_picking_tree_incoming` / `_outgoing` / `_internal` / `get_action_click_graph` | — | Return the corresponding list actions. |

## 3.2 Operation Type

| Operation | Output |
|---|---|
| `get_stock_picking_action_picking_type` | The Transfer list for this type's kind. |
| `get_action_picking_tree_late`, `_backorder`, `_waiting`, `_ready`, `get_action_picking_type_ready_moves`, `get_action_picking_type_moves_analysis` | The corresponding filtered lists. |
| `action_redirect_to_barcode_installation` | The package installer filtered on the barcode capability. |
| `action_batch`, `action_wave` | The batch and wave lists (batch capability). |

## 3.3 Stock Move

| Operation | Inputs | Output |
|---|---|---|
| `action_show_details` | — | The detail screen of the move. |
| `action_add_packages` | context key `picking_id` | The container chooser. Fails with "You need a transfer to add these packages to." when no Transfer is in context. |
| `action_product_forecast_report` | — | The forecast screen for the move's product and Warehouse. |
| `action_open_reference` | — | The scrap, the Transfer, or the move itself. |
| `action_generate_lot_line_vals` | a context dictionary of default values, a mode (`generate` or `import`), a first lot name, a count, and a pasted text | A list of detail-line value dictionaries, with links rendered as identifier-and-name pairs, and the product's lot sequence advanced. |
| `split_lots` | a pasted text | A list of dictionaries carrying a typed lot name and a quantity. |

## 3.4 Stock Move Line

| Operation | Inputs | Output |
|---|---|---|
| `action_put_in_pack` | `package_id`, `package_type_id`, `package_name` | The container, a wizard action or a label action. |
| `action_revert_inventory` | — | A list action onto the reverting lines, or a danger notification "There are no inventory adjustments to revert." |
| `action_open_reference` | — | The move's reference document or the line itself. |
| `get_move_line_quant_match` | a move identifier, the identifiers of the lines being edited, the identifiers of the quantity records being edited | Two lists: per quantity record its recomputed available quantity and the lines attached to it; per line its quantity and its matching quantity record. |
| `action_open_add_to_wave` | context key `active_wave_id` | Adds the lines to that wave, or returns the wave chooser. |

## 3.5 Stock Quantity

| Operation | Inputs | Output |
|---|---|---|
| `action_view_quants` | — | The quantity list with pivot and graph, pre-filtered to internal Locations. Runs the housekeeping pass. |
| `action_view_inventory` | — | The physical inventory list. Runs the housekeeping pass. |
| `action_apply_inventory` | an optional date | Applies the counts, or returns the conflict screen. |
| `action_apply_all` | the active filter | Returns the reference-label screen for every record matching the filter. |
| `action_set_inventory_quantity` | — | Copies the on-hand quantity into the counted quantity and assigns the reader, or returns the already-set warning. |
| `action_set_inventory_quantity_zero` | — | Sets the counted quantity to zero (and applies it immediately in report mode). |
| `action_clear_inventory_quantity` | — | Clears the counted quantity, the difference, the flag and the assignee. |
| `action_reset` | — | Returns the reset warning screen. |
| `action_stock_quant_relocate` | — | Returns the relocation screen. Fails with the message of `business-rules.md`, rule 7.9. |
| `action_inventory_history`, `action_view_stock_moves`, `action_view_orderpoints` | — | Window actions onto the related records. |
| `get_aggregate_barcodes` | — | A list of aggregate barcode strings, built as in `workflows.md`, section 28. |
| `get_import_templates` | — | One entry: label "Import Template for Inventory Adjustments" and the path of the shipped spreadsheet. |

## 3.6 Package

| Operation | Inputs | Output |
|---|---|---|
| `unpack` | — | Detaches the child containers and relocates the contents out of the container. |
| `action_put_in_pack` | `package_id`, `package_type_id`, `package_name` | Nests the containers into a new or given one. |
| `action_remove_package` | context key `picking_ids` | Removes the containers from the Transfers. Returns true. |
| `action_add_to_picking` | context key `picking_id` | Adds the containers to that Transfer. |
| `action_view_picking` | — | The Transfers that touched the containers. |

## 3.7 Lot

| Operation | Output |
|---|---|
| `action_lot_open_quants` | The quantity records of the lot, in counting mode for a manager. |
| `action_lot_open_transfers` | The outgoing Transfers that carried the lot. |
| `generate_lot_names` | Given a first name and a count, the list of generated names. |
| `_get_next_serial` | Given a company and a product, the next serial number, or nothing. |

## 3.8 Warehouse

| Operation | Output |
|---|---|
| `action_view_all_routes` | Every Route of the Warehouse, including its resupply Routes and its supply-on-order Route. |
| `get_current_warehouses` | A list of the readable Warehouses with their identifier, name and short name. |

## 3.9 Reception report

| Operation | Inputs | Effect |
|---|---|---|
| `get_report_data` | the document identifiers and a data dictionary | The report content, with the documents, the per-source line lists, the per-source formatted dates and the unit-display flag rendered for a screen. |
| `action_assign` | the demand identifiers, the quantities, and per demand the incoming move identifiers | Links them, splitting the demands as needed, and re-reserves. |
| `action_unassign` | one demand identifier, a quantity, and the incoming move identifiers | Unlinks them, splitting the demand as needed, and unreserves. |

## 3.10 Scrap

| Operation | Output |
|---|---|
| `action_validate` | Performs the scrap, or returns the shortage screen. |
| `do_scrap` | Performs the scrap unconditionally. Returns true. |
| `do_replenish` | Raises a supply request for the scrapped quantity. |
| `action_get_stock_picking`, `action_get_stock_move_lines` | Window actions onto the related records. |

## 3.11 Batch Transfer

| Operation | Output |
|---|---|
| `action_confirm` | Confirms the batch and its Transfers. Returns true. |
| `action_assign` | Reserves every Transfer of the batch. |
| `action_done` | Validates the batch (detaching the empty Transfers first). Returns whatever the underlying validation returns. |
| `action_cancel` | Cancels the batch and detaches its Transfers. Returns true. |
| `action_print` | The batch document. |
| `action_put_in_pack` | Packs the batch's lines. |
| `action_merge` | Merges the selected batches and returns a notification naming the survivor with a link to it. |
| `action_batch_detailed_operations`, `action_see_packages`, `action_view_reception_report`, `action_open_label_layout` | Window actions. |
| `order_on_zip` | Re-sorts the Transfers by the contact's postal code and stamps the batch sequence (dispatch capability). |

---

# 4. Routes

| Path | Method | Authentication | Purpose |
|---|---|---|---|
| `/stock/<output_format>/<report_name>` | HyperText Transfer Protocol GET | signed-in user | Renders the traceability tree. The request carries `data` (the serialised tree the screen built), `active_id`, `active_model` and an optional `context`, all as text parameters. Only the `pdf` output format is implemented; it answers with the portable-document-format bytes, content type `application/pdf` and the attachment file name `stock_traceability.pdf`. Any failure is answered as a serialised error with code 0 and the message "Odoo Server Error", wrapped in an internal-server-error response. The traceability record used is the most recent one created by the calling user. |

The domain exposes no public (unauthenticated) route.

---

# 5. Printable documents

| Document | Applies to | Produced file name | Content |
|---|---|---|---|
| Picking Operations | Transfer | "Picking Operations - *the contact name* - *the reference*" | The Transfer as a picking list: header with the reference, the contact, the Operation Type, the scheduled date and the source document; one row per move with product, description, demand, processed quantity and unit; barcodes for the Transfer and for each product; lots and containers when the corresponding groups are active. Printing it sets the printed flag. |
| Delivery Slip | Transfer | "Delivery Slip - *the contact name* - *the reference*" | The Transfer as a delivery note: the delivery address when the Transfer goes to an external Location, the shipping and invoicing addresses, one row per aggregated product group (`calculations.md`, section 24.9) with the ordered and delivered quantities, the containers and their contents when the container group is active, the lots when the lot-on-slip group is active, and the signature when one was captured. |
| Packages | Transfer | "Packages - *the reference*" | One block per container of the Transfer, with its name, its barcode, its type, its dimensions and its contents. |
| Return slip | Transfer | — | A return label for the Transfer. |
| Count Sheet | Stock Quantity | "Count Sheet" | A blank counting list: product, Location, lot, container, owner, on-hand quantity and an empty column to write the count in. |
| Reception Report | Transfer (and Batch Transfer) | — | The report of section 2.9 as a document. |
| Reception Report Label | Stock Move | — | One label per allocated move; the number of copies is the demand rounded up to the next whole number. |
| Package Barcode with Contents | Package, Package History | — | The container's barcode and a list of its contents. |
| Package Barcode (portable document) | Package, Package History | — | The container's barcode alone. |
| Package Barcode (ZPL) | Package, Package History | — | The same, as printer-language output. |
| Location Barcode | Location | — | The Location's barcode and full name. |
| Lot/Serial Number (portable document) | Lot | — | The lot's barcode and name; the structured variant when the structured-barcode group is active. |
| Lot/Serial Number (ZPL) | Lot | — | The same, as printer-language output. |
| Operation type (portable document) and (ZPL) | Operation Type | — | The type's barcode. |
| Product Label (ZPL) | Product | — | The product's barcode, name and, in the price variants, its price. |
| Packaging Barcodes (ZPL) | packaging unit | — | The packaging unit's barcode. |
| Product Routes Report | Routes Report wizard | — | A diagram of the rules that apply to one product across the chosen Warehouses. |
| Traceability | Traceability Report | `stock_traceability.pdf` | The tree of section 2.10. |
| Batch Transfer | Batch Transfer | — | The batch as one picking list, with the Transfers in batch-sequence order (which the dispatch capability sets from the postal codes). |

## 5.1 Label layouts

The product-label wizard offers the formats `dymo`, `2x7xprice`, `4x7xprice`, `4x12`, `4x12xprice`, `zpl` and `zplxprice`, and a quantity choice of either the operation quantities or a custom number. The printer-language formats additionally offer four templates: Normal (2.25 by 1.25 inches), Small (1.25 by 1.00 inches), Alternative (2.00 by 1.00 inches) and Jewelry (2.20 by 0.50 inches), with a live preview.

The lot-label wizard offers a quantity choice of one label per lot or one label per unit, and the two formats `4x12` and `zpl`.

## 5.2 Automatic printing

At a successful validation the Operation Type's switches are read in this order and the matching actions are collected into one multi-print action:

1. Delivery Slip, for the Transfers whose type asks for it.
2. Return slip, likewise.
3. Reception Report, for the Transfers whose type asks for it, whose kind is not delivery and whose moves have destination moves. Requires the reception-report group.
4. Reception Report Labels, for the Transfers whose type asks for it and whose kind is not delivery; the label count per move is its demand rounded up. Requires the reception-report group.
5. Product labels, grouped by the format each Operation Type asks for.
6. Lot labels, grouped by the format each Operation Type asks for. Requires the lot group.
7. Container contents, for the Transfers whose type asks for it and which have destination containers. Requires the container group.

When the Reception Report should also be shown and print actions exist, the report action is returned as the follow-up of the multi-print action; when no print action exists, the report action is returned alone.

Separately, the container label is printed immediately when a container is created by the put-in-pack action and the Operation Type asks for it, in the chosen format.

---

# 6. Messages and notifications

| Occasion | Channel | Content |
|---|---|---|
| A delivery is validated and the company asks for email confirmation | a message in the Transfer's thread, sent immediately with the light notification layout and the comment subtype | The company's delivery template. Shipped template: name "Shipping: Send by Email", subject "*the company name* Delivery Order (Ref *the reference*)". |
| A delivery is validated, the company asks for text-message confirmation and the contact has a telephone number | a text message to the contact, not queued | The company's text template. Shipped template: name "Delivery: Send by SMS Text Message", body "*the company name*: We are glad to inform you that your order n° *the source document* has been shipped." — or the same sentence without the order number when there is no source document — followed, when a tracking reference exists, by " Your tracking reference is *the tracking reference*." |
| A backorder is created | a note in the parent Transfer's thread | "The backorder *a link to the backorder* has been created." |
| A demand or a detail-line characteristic changes on an open Transfer | a note in the Transfer's thread, under the internal-note subtype | The change-tracking template, rendering the move and the new values, naming the lot, the Locations, the containers and the owner by name rather than by identifier. |
| A signature is captured | a message in the Transfer's thread with the rendered delivery document attached | "Order signed by *the contact name*", or "Order signed" when there is no contact. |
| A chained move becomes late and shifts a deadline | a note on each affected document | Subject "Deadline updated due to delay on *the upstream document name*", body "The deadline has been automatically updated due to a delay on *a link to the upstream document*." Skipped when the last message already carries that subject. |
| An upstream document will deliver less than expected | a scheduled warning activity on each affected document, assigned to the product's responsible | The shortage template, naming the originating Transfer, the per-move old and new quantities, and the impacted Transfers found by walking the destination moves transitively. |
| A Transfer is validated as part of a batch | a note in the Transfer's thread | "**Transferred by:** Batch Transfer *a link to the batch*" |
| Transfers are detached from a batch at validation | a note in the batch's thread | "*the links to the detached transfers* was removed from the batch, no quantity processed" |
| A responsible is assigned or removed through a batch | a note in each Transfer's thread | "Assigned to *a link to the batch* Responsible" or "Unassigned responsible from *a link to the batch*" |
| A return or an exchange is created | a note in the new Transfer's thread | The origin-link template naming the Transfer it came from. |
| A batch changes state | tracked in the batch's thread under the shipped "Stage Changed" subtype | — |

---

# 7. Rule description sentences

Every Stock Rule renders a sentence describing what it does. Let *source* be the source Location display name or the words "Source Location" when empty; *destination* the destination Location display name or "Destination Location"; *operation* the Operation Type name or "Operation Type"; and *direct destination* the Operation Type's own default destination display name when it differs from the rule's destination Location.

- **Pull:** "When products are needed in **destination**, **operation** are created from **source** to fulfill the need." followed by the suffixes below.
- **Push:** "When products arrive in **source**, **operation** are created to send them to **destination**."
- **Pull and push:** the pull sentence, a blank line, then the push sentence.

Suffixes appended to the pull sentence:

- when a direct destination exists and the rule does not force its own destination: "The products will be moved towards **direct destination**, as specified from **operation** destination."
- when the supply method is advanced and a source Location is set: "A need is created in **source** and a rule will be triggered to fulfill it."
- when the supply method is take-from-stock-else-trigger and a source Location is set: "If the products are not available in **source**, a rule will be triggered to bring the missing quantity in this location."

---

# 8. Search, filter and grouping contracts

## 8.1 Transfers

Filters, by reproduced name and label: `to_do_transfers` "To Do", `my_transfers` "My Transfers", `draft` "Draft", `waiting` "Waiting", `available` "Ready", `late` "Late", `backorder` "Backorders", `reception` "Receipts", `delivery` "Deliveries", `internal` "Internal", and the six date-category filters `before` "Before", `yesterday` "Yesterday", `today` "Today", `day_1` "Tomorrow", `day_2` "The day after tomorrow", `after` "After".

Groupings, by reproduced name and label: `status` "Status" (on the status), `expected_date` "Scheduled Date" (on the scheduled date), `origin` "Source Document", `partner_country` "Destination Country" (on the contact's country), `picking_type` "Operation Type", `group_by_picking_properties` "Properties" (on the free properties).

Text search matches the reference, the source document, the contact and the product.

Two search fields are computed rather than stored and therefore need their own search contract:

- **Availability state** — accepts the values `available`, `expected`, `late` and the empty value. The empty value also matches Transfers whose status is done, cancelled or draft. For the other values the whole set of Transfers outside those three statuses is evaluated with the availability predicate of `calculations.md`, section 24.2, comparing each move's forecast expected date against its Transfer's scheduled date.
- **Date category** — accepts the six category values and turns each into the date boundaries of `calculations.md`, section 22.6.
- **Delay alert date** — only positive operators are supported; it delegates to the moves' own delay alert dates.

## 8.2 Quantity records

Filters, by reproduced name and label: `my_count` "My Counts" (the reader is the assignee), `to_count` "To Count" (the scheduled date is on or before today), `to_apply` "To Apply" (a counted quantity has been entered), `conflicts` "Conflicts" (the record is outdated), `negative` "Negative Stock" (the on-hand quantity is below zero), plus the usage filters offered by the Location panel.

Groupings: product, Location, lot, container, owner, product category, storage category, company.

Row actions, by reproduced name and label: `action_stock_quant_relocate` "Relocate", `action_view_stock_moves` "History", `action_view_orderpoints` "Replenishment". List action: `action_apply_all` "Apply All".

Columns whose labels differ from the field names: `quantity` is labelled "On Hand", `available_quantity` is labelled "Available", `inventory_diff_quantity` is labelled "Difference", `user_id` is labelled "User".

Two search fields need their own contract:

- **On hand** — resolves to the Location filter of the product-quantity computation of `../replenishment-and-procurement/`.
- **Outdated** — resolves to the identifiers of the records that carry a count and whose recorded difference no longer matches the on-hand quantity.

Filtering on a lot property is rewritten into a condition on the lot.

## 8.3 Containers

Filters: main packages (those with no parent container). Searching a container by its contained quantity records, by its open detail lines, by its Transfers, by its destination Location, by its owner, by its descendants or by its outermost destination container each has an explicit contract, because none of those fields is stored; each resolves by searching the underlying records first and then mapping to the whole destination chain.

## 8.4 Locations

Text search matches the full name or the barcode. The emptiness indicator is searchable and resolves to the Locations that are **not** among those whose internal or transit quantity records sum to a strictly positive figure.

## 8.5 Lots

Text search matches the name. The on-hand quantity is searchable with the six comparison operators and with membership; when the tested value would make zero match, the result also includes every lot with no quantity record at all. The contact list is searchable; asking for lots with no contact reverses the search and returns the lots never sent to any of the listed contacts.

## 8.6 Operation Types

Text search matches the name or the Warehouse name, and additionally understands the displayed form "*warehouse name*: *type name*" by splitting it on the colon and space. The favourite indicator is searchable and sortable, and the sort is performed by testing membership of the reader in the favourite-users list.

---

# 9. Import and export

## 9.1 Importing quantity records

A shipped spreadsheet template is offered, labelled "Import Template for Inventory Adjustments". Importing behaves specially:

1. The import runs in counting mode.
2. A row with no Location is given the stock Location of the company's first Warehouse.
3. The automatic merge with an existing record is **skipped** during import, so that one row produces exactly one record; the housekeeping pass collapses duplicates afterwards.
4. A row naming a lot that belongs to another product causes a lot with the same name to be searched, and created when missing, for the correct product.
5. Only the fields allowed in counting mode may be written.

## 9.2 Exporting

Every entity of the domain can be exported through the generic mechanism. Two computed columns behave specially on export: the available quantity of a grouped quantity read is produced as the summed on-hand quantity minus the summed reserved quantity, and the inventoried quantity is produced as the summed on-hand quantity. In report mode the counted quantity is deliberately exported as empty.

## 9.3 Barcode data

The aggregate barcode generator of `workflows.md`, section 28, is the export format used by handheld devices. It depends on two system parameters and on the barcode rules that map units of measure to structured application identifiers.

---

# 10. Integrations with other domains

| Domain | Contract |
|---|---|
| `../inventory-valuation-and-costing/` | Reads every completed Stock Move, its unit price field, its Locations, its quantity and its date; produces the journal entries. This domain guarantees that a completed move never changes its product, its unit or its quantity except through the replay of `calculations.md`, section 7.2, which the valuation domain observes. |
| `../replenishment-and-procurement/` | Owns the rule-matching algorithm, the supply request entry point, the forecast report, the lead-time arithmetic and the reordering rules. This domain calls it at three points: when confirming a move with the advanced supply method, when pushing after a completion, and when a scrap asks to be replenished. |
| `../purchasing/`, `../sales/`, `../manufacturing/` | Create Stock Moves through the rule engine, read back the received and delivered quantities from the completed moves, and are notified of shortages through the warning activity. |
| `../delivery-and-shipping/` | Adds carrier fields to Transfers and containers and consumes the shipping weight and volume computed here. |
| `../products-and-catalog/` | Supplies the product, its storable flag, its tracking mode, its weight and volume, its routes, its lot numbering sequence, its lot property definitions, and the barcode rules. |
| `../units-of-measure-and-packaging/` | Supplies every conversion, rounding and comparison used here, the packaging units carried on moves, and the container types those packaging units point at. |
| `../messaging-and-activities/` | Supplies the discussion threads, the activity scheduling, the message templates and the text-message channel. |
| `../repair-and-maintenance/` | Equipment records point at a Location and match a serial number by name. |
| `../fleet/` | Supplies vehicles, vehicle categories and drivers to dispatch management. |
| `../automation-and-integration/` | Supplies the numbering sequences, the per-company defaults, the system parameters and the scheduled job runner. |

## 8.7 Stock Moves (moves analysis)

Filters, by reproduced name and label: `ready` "Ready", `future` "To Do", `done` "Done", `incoming` "Incoming", `outgoing` "Outgoing", `inventory` "Inventory", `today` "Date".

Groupings, by reproduced name and label: `by_product` "Product" (on the product), `groupby_picking_type_id` "Operation Type", `groupby_picking_id` "Picking" (on the Transfer), `groupby_location_id` "Source Location", `groupby_dest_location_id` "Destination Location", `status` "Status", `groupby_create_date` "Creation Date", `groupby_date` "Scheduled Date".

## 8.8 Stock Move Lines (moves history)

Filters, by reproduced name and label: `todo` "To Do", `done` "Done", `incoming` "Incoming", `outgoing` "Outgoing", `internal` "Internal", `manufacturing` "Manufacturing", `inventory` "Inventory Adjustments", and three date windows: `filter_last_30_days` "Last 30 Days", `filter_last_3_months` "Last 3 Months", `filter_last_12_months` "Last 12 Months".

Groupings, by reproduced name and label: `groupby_product_id` "Product", `by_state` "Status", `by_date` "Date", `by_picking` "Transfers", `by_location` "Location", `by_category` "Category" (on the product category's full name).

## 8.9 Locations

Filters, by reproduced name and label: `in_location` "Internal", `customer` "Customer", `inventory` "Inventory Loss", `prod_inv_location` "Production", `supplier` "Vendor", `view` "Virtual", `empty_location` "Empty Locations".

Groupings: `warehouse_id` "Warehouse", `usage` "Location Type".

## 8.10 Lots

Filters and groupings, by reproduced name and label: `group_by_product` "Product", `group_by_location` "Location", `group_by_creation_date` "Creation date", `group_by_lot_properties` "Properties", plus a grouping by company.

When the list is grouped by Location, the empty groups shown are deliberately widened: the customer and vendor Locations, plus the stock Location of every Warehouse, are always offered as groups even when no lot currently sits there.

## 8.11 Containers

Filters, by reproduced name and label: `internal` "In internal locations", `main_packages` "Main Packages" (containers with no parent container).

## 8.12 Scraps

Groupings, by reproduced name and label: `product` "Product", `location` "Location", `scrap_location` "Scrap Location", `transfer` "Transfer".

---

# 11. Screen-level indicators

A number of fields exist only to drive a screen. They are listed here with the exact condition that turns them on, because a reader cannot infer them from the data model.

| Indicator | Entity | True when |
|---|---|---|
| Show check availability | Transfer | The status is waiting, waiting-another-operation or ready; not every move is picked or already fully processed; and at least one move is waiting-another-move, waiting or partially available with a non-zero demand. |
| Show allocation | Transfer, Batch Transfer | See `calculations.md`, section 24.4. |
| Show next transfers | Transfer | The destination moves of this Transfer's moves belong to at least one Transfer that is not one of this Transfer's returns. |
| Show lot text box | Transfer | The lot group is active, the Operation Type allows creating but not using existing lots, and the Transfer is not done. False outright when the Transfer has no detail line and the Operation Type does not allow creating lots. |
| Has tracking | Transfer | At least one move carries a tracked product. |
| Is signed | Transfer | A signature image is present. |
| Is scheduled date editable | Transfer | Always, except for a done or cancelled Transfer, where it follows the lock flag being off. |
| Has scrap move | Transfer | At least one move's destination Location has inventory-loss usage. |
| Details visible | Stock Move | See `calculations.md`, section 24.3. |
| Show quantity picker | Stock Move | The Operation Type kind is not receipt and the product is storable. |
| Show lot selector | Stock Move | The quantity picker is off, the lot text box is off, the product is tracked, and either existing lots may be used, the move is done, or the move is a return. |
| Show lot text box | Stock Move | The product is tracked, the Operation Type allows creating but not using existing lots, the move is not done, and the move is not a return. |
| Show assign serial / import lot | Stock Move | The product is tracked, a product is set, the Operation Type allows creating lots, the move is not a return, and the status is neither done nor cancelled. |
| Is initial demand editable | Stock Move | The Transfer is unlocked, or the move is in draft. |
| Lines without destination container | Stock Move | At least one line has a destination container and at least one has none. |
| Lot fields visible | Stock Move Line | With a Transfer that has an Operation Type and a tracked product: the type allows existing or new lots. Otherwise: the product is tracked. |
| Duplicated serial number | Stock Quantity | The same serial number appears on more than one record with a positive quantity in an internal or transit Location. |
| Outdated | Stock Quantity | A count has been entered and `counted − recorded difference` no longer equals the on-hand quantity. |
| Is empty | Location | The sum of on-hand quantities of the records directly in it (restricted to internal and transit) is at most zero. |
| Has contents | Package Type | At least one container of that type holds quantity records. |
| Valid serial shipping container code | Package | The container's name passes the encoding check. |
| Issues popover | Package | The container's open detail lines point at more than one destination Location. |
| Hide reservation method | Operation Type | The kind is receipt. |
| Show operation kind | Operation Type | The kind is receipt, delivery or internal transfer. |
| Show check availability | Batch Transfer | At least one move of the batch is not ready, cancelled or done. |
| Match serial | Equipment | At least one Lot exists whose name equals the equipment's serial number, and the reader may read Lots and is in the lot group. |

---

# 12. Onboarding and empty-state guidance

Every Transfer list renders its empty-state guidance from one shared template, parameterised by the Operation Type kind currently in force — taken from the calling context when it restricts the list to one kind, otherwise from the Transfers being listed. The three kinds therefore produce three different pieces of guidance (receive goods, deliver goods, move goods internally).

The quantity list produces its own: "Your stock is currently empty" followed by "Press the \"New\" button to define the quantity for a product in your stock or import quantities from a spreadsheet via the Actions menu".

---

# 13. Client-side actions returned by the domain

Three of the operations return a client-side action rather than a window action; an implementation must reproduce the shape, because the screens dispatch on it.

| Action | Returned by | Parameters |
|---|---|---|
| Multi-print | Validating a Transfer when the Operation Types ask for automatic printing | The list of report actions, and optionally one follow-up action (the Reception Report). |
| Display notification | Merging batches; reverting an adjustment with nothing to revert; adding lines to a wave | A title, a message with one substitution slot, an optional list of links each with a label and an address, whether the notification is sticky, and optionally a follow-up action that closes the current screen. |
| Soft reload | Adding lines to a wave from inside the wave screen | None. |

---

# 14. The printable documents, section by section

## 14.1 The delivery document

Rendered for a Transfer. The contact used for the address block is the Transfer's contact; the delivery address is shown only when the Transfer goes to an external Location, that is when its Operation Type kind is delivery and either the first move or the Transfer names a contact.

**Header.** The Transfer's reference as the title; the source document; the scheduled date; the contact; the shipping and invoicing addresses when they differ.

**Section 1 — Ordered quantities.** One row per Stock Move that has a demand, with:

| Column | Content |
|---|---|
| Product | The product's display name, with the move's transfer description underneath, that description stripped of a leading repetition of the product name |
| Ordered | The move's demand, formatted at the `Product Unit` precision, followed by the line unit's name when the multiple-units group is active, and underneath, in a muted style, the packaging quantity and the packaging unit's name |
| Delivered | The move's processed quantity, formatted the same way, and underneath the same quantity converted into the packaging unit |

**Section 2 — Delivered detail.** Present when the Transfer has detail lines. One block per outermost destination container, then one block for the loose lines. The rows come from the aggregation of `calculations.md`, section 24.9, so they are grouped by product, description, unit and packaging unit rather than by line.

| Column | Content | Shown when |
|---|---|---|
| Product | The aggregated product and description | always |
| Package | The container's name, or its path inside the outermost one for a nested container | the Transfer has more than one level of containers |
| Lot/Serial Number | The lot names of the lines in the group | the lot-on-slip group is active and at least one line carries a lot |
| Delivered | The aggregated processed quantity with its unit | always |

**Section 3 — Remaining quantities not yet delivered.** Present when the Transfer has backorders. One row per product still owed, from the same aggregation walked over the backorder chain: the product, and the quantity still owed.

**Footer.** The signature image when one was captured, above the contact's name.

## 14.2 The picking document

Rendered for a Transfer. Optimised for a person walking the warehouse rather than for a customer.

**Header.** The reference as a barcode and as text; the Operation Type; the source and destination Locations; the scheduled date; the source document; the contact.

**Body.** One row per Stock Move, or per detail line when the Operation Type asks to show detailed operations, with: the product as a barcode and as text, the description, the source Location, the destination Location, the lot, the source container, the destination container, the demand, the processed quantity and the unit. The Location, lot and container columns appear only when their respective groups are active.

Printing this document sets the Transfer's printed flag, which excludes it from further automatic grouping of new moves. The flag is also set when the document is rendered for a Transfer whose status is ready.

## 14.3 The container document

Rendered for a Transfer. One page per outermost container of the Transfer, each with: the container's name as a barcode and as text, its type, its dimensions, its weight, and a list of its contents — product, lot, quantity, unit — including the contents of every nested container, each nested container introduced by its own name.

For a done Transfer the content is read from the Package History snapshots rather than from the containers, so the document still prints correctly after the containers have been re-used.

## 14.4 The count sheet

Rendered for a set of Stock Quantity records. One row per record with: the product, the Location, the lot, the container, the owner, the on-hand quantity, the unit, and an empty column for the person to write the count in. Grouped by Location.

## 14.5 The reception report

Rendered for one or more Transfers. Grouped by the source document of each demand. Each group is introduced by that document's name, its scheduled date, its contact and its priority. Each row carries the product, the quantity, the unit and its state: assignable, already assigned, or expected but not assignable.

## 14.6 The label documents

| Document | Content |
|---|---|
| Product label | The product's barcode, its display name, and its price in the price variants. Four sheet layouts and one printer-language layout with four templates. |
| Lot label | The lot's barcode — the structured variant when the structured-barcode group is active — its name, and the product's name. |
| Container label | The container's barcode and name, with or without the content list. |
| Location label | The Location's barcode and full name. |
| Operation type label | The Operation Type's barcode and display name. |
| Packaging label | The packaging unit's barcode and name. |
| Reception report label | One label per allocated move, carrying the product, the quantity and the source document, repeated as many times as the demand rounded up. |

## 14.7 The routes diagram

Rendered from the Routes Report wizard. For each chosen Warehouse, a diagram whose nodes are Locations and whose edges are Stock Rules, each edge labelled with the Operation Type, the action, the supply method and the lead time. The rules the chosen product would actually take are highlighted.

## 14.8 The traceability document

Rendered through the route of section 4, in landscape. One row per unfolded node of the tree, with the seven columns of `calculations.md`, section 26.3, indented by level. The title is the display name of the record the report was opened from.

## 14.9 The batch document

Rendered for a Batch Transfer. One combined picking list covering every Transfer of the batch, the Transfers appearing in batch-sequence order — which the dispatch capability sets from the contacts' postal codes — each introduced by its reference and its contact, followed by its rows in the same shape as the picking document.
