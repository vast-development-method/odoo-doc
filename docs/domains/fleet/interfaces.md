# Interfaces

This file specifies everything through which a person or another system reaches the fleet domain: the menu tree, every screen definition, every named operation that a button or a menu invokes, every analysis screen, the printable output, the message templates, the bound operations offered from a selection, and the import and export surface. Screen definitions are described by what they show and what they let a reader do, never by their markup; the general model of screens, layouts and window actions is specified in [../../overview/views-and-actions.md](../../overview/views-and-actions.md).

---

## 1. The menu tree

The root menu is titled "Fleet" and is visible to a member of the fleet officer group. It carries a vehicle icon and sits at position 220 in the application bar.

| Level | Menu | Position | Visible to | Opens |
|---|---|---|---|---|
| 1 | Fleet, the root | 220 | Fleet officer | Nothing itself |
| 2 | Fleet | 2 | Fleet officer | Nothing itself |
| 3 | Fleet | 0 | Fleet officer | The vehicles screen |
| 3 | Contracts | 2 | Fleet officer | The contracts screen |
| 3 | Services | 3 | Fleet officer | The services screen |
| 3 | Odometers | 10 | Fleet officer | The odometer readings screen |
| 2 | Reporting | 99 | Fleet administrator | Nothing itself |
| 3 | Costs | 1 | Fleet administrator | The cost analysis screen |
| 3 | Odometers | 2 | Fleet administrator | The distance analysis screen |
| 2 | Configuration | 100 | Fleet administrator | Nothing itself |
| 3 | Settings | 0 | The system administration group | The general settings screen, opened on the fleet panel |
| 3 | Models | 10 | Fleet administrator | Nothing itself |
| 4 | Manufacturers | 1 | Inherited from its parent | The manufacturers screen |
| 4 | Models | 5 | Inherited from its parent | The models screen |
| 4 | Categories | 10 | Inherited from its parent | The categories screen |
| 3 | Services | 20 | The technical-features flag | Nothing itself |
| 4 | Types | 1 | The technical-features flag | The service kinds screen |
| 3 | Vehicle | 30 | The technical-features flag | Nothing itself |
| 4 | Status | 10 | The technical-features flag | The vehicle statuses screen |
| 4 | Tags | 20 | The technical-features flag | The vehicle tags screen |
| 3 | Activity Types | 99 | The technical-features flag | The activity types screen restricted to contracts |

**Four consequences worth stating.**

1. **A fleet officer sees only the second-level "Fleet" branch.** Reporting and Configuration are both restricted to the fleet administrator group.
2. **Three configuration branches are hidden behind the technical-features flag**: service kinds, vehicle statuses and vehicle tags. An administrator without that flag cannot reach them from a menu at all, even though the access rights permit the operations. A rebuild must reproduce the restriction; a fleet administrator who needs to add a status turns the flag on for themselves.
3. **The vehicles screen carries a direct path.** The window action that opens it declares the path fragment `fleet`, so the screen has a stable address that can be bookmarked and linked to. Its resolution is specified in [../../interfaces/endpoint-catalog.md](../../interfaces/endpoint-catalog.md).
4. **The second-level menu and its first child are both titled "Fleet"**, which is the shipped naming.

---

## 2. Window actions

Fourteen window actions are declared. Each names the entity it opens, the layouts it offers in order, the filter it applies by default and the empty-state guidance it shows.

| External identifier | Title | Entity | Layouts, in order | Applied by default |
|---|---|---|---|---|
| `fleet_vehicle_action` | "Vehicles" | Vehicle | board, list, form, cross-table, activity | Nothing |
| `fleet_vehicle_log_contract_action` | "Contracts" | Vehicle Contract | list, board, form, chart, cross-table, activity | The "In Progress" filter |
| `fleet_vehicle_log_services_action` | "Services" | Vehicle Service | list, board, form, chart, cross-table, activity | Grouping by service kind |
| `fleet_vehicle_odometer_action` | "Odometers" | Odometer Reading | list, form, chart | Nothing |
| `fleet_vehicle_model_action` | "Models" | Vehicle Model | list, form | Grouping by manufacturer |
| `fleet_vehicle_model_brand_action` | "Manufacturers" | Vehicle Manufacturer | board, list, form | The "With Models" filter |
| `fleet_vehicle_model_category_action` | "Categories" | Vehicle Category | list | Nothing |
| `fleet_vehicle_state_action` | "Status" | Vehicle Status | list, form | Nothing |
| `fleet_vehicle_tag_action` | "Tags" | Vehicle Tag | the entity's default layouts | Nothing |
| `fleet_vehicle_service_types_action` | "Types" | Fleet Service Type | list, form | Grouping by category |
| `fleet_costs_reporting_action` | "Costs Analysis" | Fleet Analysis Report | chart, cross-table | The period filter on the cost date, set to the current year |
| `fleet_vehicle_odometer_reporting_action` | "Odometer Analysis" | Fleet Odometer Analysis Report | chart | Grouping by month and by category; restricted to active vehicles |
| `fleet_config_settings_action` | "Settings" | Configuration Settings | form | Opened on the fleet panel |
| `mail_activity_type_action_config_fleet` | "Activity Types" | Activity Type | list, board, form | Restricted to types bound to nothing or to the Vehicle Contract; new types default to the Vehicle Contract |

**Empty-state guidance.** Ten of the fourteen carry a short encouragement shown when the screen has no records: the vehicles screen invites the reader to create their first vehicle; the contracts screen explains that a contract may be a lease or an insurance, that it groups the services it covers, and that renewals are warned about automatically; the services screen explains that services may be occasional repairs or fixed maintenance; the odometer screen explains that any number of readings may be added for every vehicle; the models screen explains that several models may be defined for one manufacturer; the manufacturers screen invites the reader to create one; the categories screen explains that categories help arrange the fleet; the statuses screen explains that statuses may be customised to track each vehicle's evolution and gives active, being repaired and sold as examples; the tags screen invites the reader to add one; the service-kinds screen explains that each kind may be used in contracts, as a standalone service, or both; and the two analysis screens say that there is no data to analyse. The wording is guidance and carries no behaviour.

---

## 3. Screens, entity by entity

### 3.1 Vehicle

**Form.** A specialised form controller intercepts the archive entry of the action menu and shows the confirmation dialogue of rule FLT-034 in [business-rules.md](business-rules.md). The form is laid out as:

- **Header.** A button labelled "Apply New Driver", shown only when the vehicle has a future driver, which invokes `action_accept_driver_change`. Beside it the vehicle status as a clickable progress bar, so a reader moves a vehicle along the pipeline by clicking a segment.
- **Counter buttons.** Bills, shown only when the count is greater than zero, invoking `action_view_bills`, added by the accounting bridge. Employee, shown only when the vehicle has a driver Employee and only to the human resources officer group, invoking `action_open_employee`, added by the people bridge. Drivers History, invoking `open_assignation_logs`. Contracts, invoking `return_action_to_open` with the contracts screen named in its context and — when the vehicle is archived — with the archived filter applied. Services, in three mutually exclusive variants selected by the service activity indicator of [calculations.md](calculations.md), C-19: a plain one when the indicator is `none`, a red one when it is `overdue` and an amber one when it is `today`, all three invoking `return_action_to_open` with the services screen named. Odometer, shown only for a car, invoking `return_action_to_open` with the odometer screen named.
- **Archived ribbon.** Shown across the corner when the vehicle is archived.
- **Title block.** The manufacturer logo as the record picture; the model, with a placeholder suggesting a model name; the licence plate, with a placeholder suggesting a plate; the tags, coloured from the tag palette and offered without an inline create-and-edit path.
- **Driver group.** Driver, shown with an avatar and restricted to Contacts of the vehicle's company or of no company; driver Employee, hidden, added by the people bridge; mobility card, read-only, added by the people bridge; future driver with an avatar; future driver Employee, hidden; the assignment date; and the company, shown only in a multi-company database with a placeholder saying that leaving it empty makes the vehicle visible to all.
- **Vehicle group.** Category; order date; registration date and cancellation date, both shown only for a car; chassis number; the distance beside its unit with a chart button that invokes `action_open_odometer_report`, the whole row shown only for a car; the fleet manager with a user avatar; the location; and a "Make Vehicle Available" marker which is the plan-to-change flag for cars or for bicycles according to the vehicle kind, restricted to the fleet officer group, with a note saying that it lets the vehicle be offered to somebody else.
- **Properties.** The user-defined values, in two columns.
- **Tax page.** The taxable horsepower, shown as a monetary amount; the first contract date; the catalogue value including value added tax; the purchase value; the residual value.
- **Model page.** A model group holding the model year, the seating capacity and the number of doors for a car, the colour, the trailer hitch for a car, the frame kind and frame size in centimetres and the electric assistance for a bicycle. An engine group, shown only for a car, holding the fuel kind, the transmission, the power beside its unit or the horsepower beside its unit according to the power unit, the range beside its unit, the emissions figure beside its unit, and the emission standard with a placeholder naming three example standards.
- **Note page.** The vehicle description, with a placeholder inviting any other information.
- **Discussion thread** at the foot.

**Shortened form.** Used by the board's quick-create panel. It asks only for the model, the licence plate and the tags.

**List.** Rows are highlighted amber when the vehicle has a renewal due soon and not overdue, and red when it has a renewal overdue. Multiple-record editing is enabled and sample data is shown when the list is empty. Columns: licence plate; model with an avatar; category; fleet manager, hidden by default, with a user avatar; driver with an avatar; future driver with an avatar; chassis number, hidden by default; the emissions figure under the column heading "CO2 Emissions", hidden by default; registration date; tags; status as a badge, hidden by default; properties; the derived contract state as a badge, hidden by default, shown in the information colour when it is running and in the danger colour when it is expired; and the activity exception indicator. The people bridge adds the driver Employee and the future driver Employee, both hidden by default.

**Board.** Grouped by status by default, with a progress bar coloured by activity state — green for planned, amber for today, red for overdue. Each card shows the manufacturer logo, the licence plate and model in bold, the tags, the driver with an avatar, the future driver when there is one, the location with a map-pin glyph when there is one, the properties, and a footer carrying a link to the vehicle's contracts with the contract count and a warning triangle — amber when a renewal is due soon and red when one is overdue — together with the activity indicator.

**Search.** Typed text matches the vehicle name or the licence plate. A drivers field matches, without the people bridge, the driver of any assignment entry, the current driver or the future driver; with the people bridge it additionally matches the driver Employee of any assignment entry, the driver Employee and the future driver Employee. Further searchable fields are the model, the licence plate, the tags, the status, the properties and — with the people bridge — the mobility card. Filters: "Available", matching vehicles with no future driver and either no driver or a set plan-to-change marker of the matching kind; "Bikes"; "Cars"; "Trailer Hook"; "Planned for Change"; "Need Action", matching vehicles whose renewal is due soon or overdue; "Archived"; and the four hidden activity filters the platform supplies. Groupings: model, manufacturer, status, fuel kind and properties.

**Activity layout.** Shows the licence plate and the model beside the vehicle picture.

**Cross-table.** Status across the top; manufacturer, model and licence plate down the side.

### 3.2 Vehicle Contract

**Form.** The state as a clickable progress bar in the header. An archived ribbon. The contract name as the title. An information group holding the reference, the coverage kind, the insurer with an avatar, the included services as coloured chips, and the company in a multi-company database; beside it the start date, the expiration date — required whenever the recurring frequency is not `no` — and the responsible user with a user avatar. A vehicle group holding the vehicle and the driver with an avatar. A cost group holding the activation cost, described as the amount paid once at the creation of the contract; the recurring cost beside its frequency, the amount hidden when the frequency is `no`; and the activation cost date. A terms-and-conditions section with a placeholder inviting any other information. A discussion thread at the foot. The people bridge adds a counter button labelled "Employee" showing the driver Employee, visible to the human resources officer group and only when there is one, which invokes `action_open_employee`.

**List.** Ordered by expiration date. Rows are amber when the contract expires today, red when its days left is zero, it does not expire today and its vehicle has no other running contract, and greyed out when it is cancelled. Columns: name in bold; start date; expiration date shown as a remaining-days indicator; vehicle; insurer; driver with an avatar; recurring cost; frequency; state as a badge, in the information colour when running and in the danger colour when expired and the vehicle has no other running contract; and the activity exception indicator. The people bridge adds the driver Employee, hidden by default.

**Board.** One card per contract showing the vehicle in bold, the state as a chip, the start and expiration dates — the pair shown in red and bold when the expiration is in the past — and the insurer. A progress bar coloured by activity state.

**Chart.** Titled "Contract Costs Per Month", plotting the activation cost against the activation cost date and the vehicle.

**Cross-table.** Expiration date across the top; coverage kind and vehicle down the side.

**Search.** Typed text matches the vehicle name, the driver including their children, or the insurer including their children. Filters: "In Progress"; "Expired"; "Archived"; and the four hidden activity filters. The activity performer and the activity kind are searchable. Grouping: vehicle. The people bridge adds an employee field matching the driver Employee including their reports.

**Activity layout.** Shows the responsible user with an avatar and the vehicle.

### 3.3 Vehicle Service

**Form.** The stage as a clickable progress bar in the header. An archived ribbon. A left group holding the description, the service kind, the date, the cost and the vendor with an avatar; a right group holding the vehicle, the driver with an avatar, and the distance beside its unit. A notes section with a placeholder inviting any other information about the service completed. A discussion thread at the foot. The accounting bridge adds a counter button that invokes `action_open_account_move`, shown only when the service carries a journal item, labelled "Service's Bill" in green when the entry is posted and in amber otherwise; it also makes the cost field read-only whenever the journal item link is set. The people bridge adds the driver Employee as a hidden field.

**List.** Multiple-record editing is enabled and every group is expanded. Columns: date, read-only; description; service kind; vehicle, read-only, with an avatar; driver, read-only, with an avatar; vendor; the vendor reference, hidden as a column; notes; the cost with a column total labelled "Total"; and the stage, read-only, as a badge shown in the success colour when done, the warning colour when new and the information colour when running. The people bridge adds the driver Employee, read-only and hidden by default.

**Board.** Grouped by stage by default. Each card shows the vehicle picture, the vehicle name in bold, the stage as a chip whose colour depends on the stage, the service kind in italics, the driver, the date, the vendor, the cost and the activity indicator. A progress bar coloured by activity state.

**Chart.** Titled "Services Costs Per Month", plotting the cost against the date and the vehicle.

**Cross-table.** Service kind across the top; vendor and vehicle down the side; the cost as the measure.

**Search.** Typed text matches the vehicle, the service kind or the description. Filters: "Archived". Groupings: service kind, fleet manager, model and manufacturer.

**Activity layout.** Shows the vehicle picture, the vehicle and, when present, the description.

### 3.4 Odometer Reading

**Form.** One group holding the vehicle, the value beside its unit, and the date.

**List.** Editable in place, with new rows added at the top. Columns: date, vehicle with an avatar, driver with an avatar, value and unit. The people bridge adds the driver Employee, hidden by default.

**Chart.** Titled "Odometer Values Per Vehicle", plotting the value against the vehicle. Because the value aggregates by maximum rather than by sum, the chart shows the greatest reading per vehicle.

**Search.** Searchable fields: vehicle, driver, value and date. Groupings: vehicle and date.

### 3.5 Driver Assignment Log

**List.** Editable in place, with new rows added at the bottom, ordered by start date descending and then by the surrogate identifier descending. Columns: vehicle; driver with an avatar under the heading "Current Driver"; start date; end date.

The people bridge supplies two further list definitions, both derived from that one:

- **The vehicle history list**, used by the vehicle's Drivers History button: the vehicle column becomes hidden by default; the driver Employee is added with a user avatar after the driver; and after the end date, the attachment count and a paperclip button labelled "Attachments" that invokes `action_get_attachment_view`.
- **The employee history list**, used by the employee's Cars History button: the driver column is removed from its original position and re-added after the end date, hidden by default and headed "Current Driver"; the attachment count and the attachments button are added.

### 3.6 Vehicle Model

**Form.** An archived ribbon. A counter button that invokes `action_model_vehicle`, showing the vehicle count when it is greater than zero and, when it is zero, the words "New Vehicle" instead. The manufacturer logo as the record picture. A title block holding the model name with a placeholder suggesting a model name and the manufacturer with a placeholder suggesting a manufacturer. A group holding the vehicle kind and the category, the latter offered without an inline create-and-edit path. An information page holding: a model group shown only for a car with the model year, seating capacity, number of doors, colour and trailer hitch; a vehicle-information group shown only for a bicycle with the electric assistance; and an engine group shown only for a car with the fuel kind, required here, the transmission, the drive kind, the power beside its unit when the power unit is `power`, the range beside its unit, the emissions figure beside its unit, the emission standard with a placeholder naming three example standards, and the horsepower beside its unit and the taxable horsepower when the power unit is `horsepower`. A vendors page listing the suppliers as cards showing the name, the telephone number and the electronic mail address. A discussion thread at the foot. The transport dispatch bridge adds nothing to this form.

**List.** Multiple-record editing is enabled. Columns: manufacturer, name, vehicle count under the heading "Vehicles", category, vehicle kind and the emissions figure hidden by default.

**Board.** One card per model showing the name in bold and the manufacturer.

**Search.** Typed text matches the name or the manufacturer. Filters: "Contains Vehicle", matching models whose vehicle count is not zero; "Archived". Groupings: manufacturer, category and vehicle kind.

### 3.7 Vehicle Manufacturer

**Form.** A counter button that invokes `action_brand_model`, hidden when the model count is zero. A group holding the name and the logo.

**List.** Two columns: name and model count under the heading "Models".

**Board.** Ordered by name. Clicking a card body invokes `action_brand_model`, so the card opens the manufacturer's models rather than the manufacturer. Clicking the logo invokes `action_open_brand_form`, which is the only path to the manufacturer's own form. The card menu offers Configuration, which opens the form; Archive when the manufacturer is active; Restore when it is archived; and Delete when the reader may delete it. The card body shows the logo, the name in bold and the model count followed by the word "MODELS" in capitals.

**Search.** Typed text matches the name. Filters: "With Models", matching manufacturers whose model count is greater than zero; "Archived".

### 3.8 Vehicle Category

**List.** Editable in place, with new rows added at the bottom. Columns: the sequence as a drag handle and the name. The transport dispatch bridge adds the maximum weight and the maximum volume, both shown by default.

**Form.** A group holding the name, and a second group holding the sequence, shown only to the technical-features flag. The transport dispatch bridge adds, after the name, the maximum weight beside its unit label and the maximum volume beside its unit label.

### 3.9 Vehicle Status

**List.** Editable in place, with new rows added at the bottom. Columns: the sequence as a drag handle, the name and the folded marker.

**Form.** A group holding the name, the sequence and the folded marker.

### 3.10 Vehicle Tag

**List.** Editable in place, with new rows added at the bottom. Columns: the name and the colour, chosen through a colour picker.

**Form.** A group holding the name alone. The colour is not offered on the form.

### 3.11 Fleet Service Type

**List.** Editable in place, with new rows added at the bottom. Columns: the name and the category.

**Search.** Searchable fields: the name and the category. Grouping: category.

### 3.12 Fleet Analysis Report

**Chart.** Titled "Fleet Costs Analysis", plotting the cost against the month of the cost date and the cost kind. Sample data is shown when there is none.

**Cross-table.** The year of the cost date and the cost kind across the top; the vehicle down the side; the cost as the measure.

**List.** Creation is disabled. Columns: the vehicle name; the driver; the fuel kind, hidden by default; the cost date; the cost with a column total labelled "Sum of Cost"; the cost kind; and the company in a multi-company database.

**Form.** Creation and editing are both disabled. Two groups: the vehicle, the driver, the fuel kind and the company; and the cost date, the cost and the cost kind.

**Search.** Typed text matches the vehicle name or the driver. Filters: "Service"; "Contract"; a period filter on the cost date whose default period is the year. Groupings: vehicle and driver.

### 3.13 Fleet Odometer Analysis Report

**Chart.** Titled "Vehicle Odometer Timeline", drawn as an unstacked line with drill-through disabled, plotting the distance travelled. Sample data is shown when there is none.

**Search.** Five groupings and no filters: the month of the recorded date, the vehicle, the category, the fuel kind and the model.

### 3.14 Send Mails to Drivers

**Form**, shown as a dialogue. The subject, required, with a placeholder suggesting a communication about the reader's vehicle. The body, in a bordered editor with a placeholder inviting the message and with the edited value always preserved. The attachments, as a file-drop area. The template, labelled "Load template". Three footer buttons: "Send", the primary one, invoking `action_send` and bound to a keyboard shortcut; "Cancel", which closes the dialogue and discards; and "Save as new template", invoking `action_save_as_template`.

### 3.15 Screens this domain changes on other entities

| Entity | Screen | Change |
|---|---|---|
| Journal Entry | Form | The vehicle column, and the hidden vehicle-required flag, are added after the account column on both the item lines and the invoice lines; the column is hidden unless the entry is a vendor bill, a vendor credit note or a vendor receipt |
| Journal Entry | List, in a derived definition used only by the vehicle's Bills button | The date column is retitled "Creation Date" and the invoice date is added after it, editable only while the entry is a draft |
| Journal Item | List | The vehicle is added after the label, hidden by default |
| Employee | Form | A counter button labelled "Cars History", visible to the fleet administrator group and only when the count is greater than zero, invoking `action_open_employee_cars`; and the mobility card added to the application group, which this bridge also makes always visible, under the label "Fleet Mobility Card" |
| Employee | Search | The private vehicle plate field is replaced by the aggregated licence plate, so a search by plate finds both company and private vehicles |
| Departure Wizard | Form | The activity block, normally hidden, is made always visible, and the release-the-company-vehicle marker is added inside it under the label "Company Car" |
| Attachment | Board, in a derived definition used only by the assignment attachment browser | Clicking a card invokes `action_preview_attachment` and opens the stored file in a new window; the card menu offers only Delete |
| Configuration Settings | Form | A panel headed "Fleet" holding one block titled "Fleet Management" with the setting "End Date Contract Alert" |
| Batch Transfer | Form | A dispatch group, shown only when the operation type has dispatch management, holding the dock — shown only when the operation type publishes docks and only to the multiple-locations group — the vehicle with a placeholder saying "Third Party Provider", the vehicle category with a placeholder suggesting a heavy vehicle class, the estimated shipping weight beside its unit with the weight share as a progress bar, and the estimated shipping volume beside its unit with the volume share as a progress bar |
| Batch Transfer | List | The responsible user column becomes shown by default, and the vehicle category, vehicle, dock, volume share and weight share are added, all hidden by default |
| Batch Transfer | Board | The footer is replaced so that it shows the dock, the state as a state selector, the scheduled date and the responsible user |
| Batch Transfer | Search | The vehicle, dock and driver become searchable; five groupings are added — vehicle, vehicle category, scheduled date, operation type and dock; two filters are added — "Own Fleet", matching batches that name a vehicle, and "Third Party Carrier", matching batches that name a category but no vehicle; and four date filters are added — a period filter on the scheduled date, "Today", "Tomorrow" and "Next 7 Days" |
| Batch Transfer | Cross-table and chart | Two new definitions: a cross-table of the scheduled date against the vehicle, and a chart of the scheduled date by day against the vehicle category |
| Operation Type | Form | The dispatch-management marker is added before the automatic-batching option, together with the dock set, shown only when dispatch management is on and only to the multiple-locations group, headed "From" for an outgoing type and "To" otherwise |
| Operation Type | Board | A "Transport Management" section is added to the card menu of every operation type that has dispatch management, offering five entries that open the batch screen in five layouts: "Manage Batches" in list and form, "Dock Dispatching" in the timeline layout, "Batches by Route" in the board layout, "Calendar", and "Statistics" in the cross-table layout |
| Location | Form | The usage field becomes read-only when the location is being created from the dock picker, so that a location created as a dock cannot be given another usage in the same step |
| Transfer | List | The postal code, the shipping weight with a column total and the shipping volume with a column total are added after the operation type, all hidden by default; and on the list used inside a batch, the postal code becomes shown by default |

**Two filters of the batch search are known to be wrong.** The filter named "Tomorrow" matches every batch scheduled from tomorrow onwards rather than only tomorrow, and the filter named "Next 7 Days" matches every batch scheduled from seven days onwards rather than within the next seven days. Both are marked as wrong in the shipped definitions themselves. A rebuild should implement "Tomorrow" as a scheduled date on or after tomorrow and before the day after tomorrow, and "Next 7 Days" as a scheduled date on or after today and before today plus seven days, and should record the change; the observed behaviour is reproduced here so that a reader knows what an existing installation does.

---

## 4. Named operations

Every operation a button, a menu or another package invokes is listed here with its target, what it needs, what it does and what it returns. The complete procedures are in [workflows.md](workflows.md).

### 4.1 On the Vehicle

| Operation | Needs | Does | Returns |
|---|---|---|---|
| `action_accept_driver_change` | One vehicle with a future driver | Promotes the future driver, clears the plan-to-change markers, and frees the vehicles of the same kind the incoming driver is giving up. Procedure W-05 | Nothing |
| `return_action_to_open` | One vehicle, and the external identifier of a screen supplied in the calling context | Opens that screen restricted to this vehicle, with this vehicle as the default on new records and with grouping switched off | A window action |
| `open_assignation_logs` | One vehicle | Opens the driver assignment entries of the vehicle as a list, with the vehicle and its current driver as the defaults on new rows. The people bridge replaces the list definition with its own | A window action |
| `action_send_email` | One or more vehicles | Opens the Send Mails to Drivers assistant in a dialogue with those vehicles filled in | A window action |
| `action_open_odometer_report` | One vehicle | Opens the distance analysis restricted to this vehicle and grouped by month | A window action |
| `action_view_bills` | One vehicle. Added by the accounting bridge | Opens the purchase entries that carry an item naming this vehicle, in the derived list definition and the standard entry form | A window action |
| `action_open_employee` | One vehicle. Added by the people bridge | Opens the form of the vehicle's driver Employee | A window action |
| `act_show_log_cost` | One vehicle | Intended to open the cost log of the vehicle. It resolves a screen that does not exist and nothing invokes it; see compatibility finding FLT-C15 in [business-rules.md](business-rules.md) | A window action, in principle |
| `_get_analytic_name` | One vehicle | Yields the licence plate, or the text "No plate" when the plate is empty. An extension point for localizations; see [accounting-effects.md](accounting-effects.md) | Text |

### 4.2 On the Vehicle Model and the Vehicle Manufacturer

| Operation | Needs | Does | Returns |
|---|---|---|---|
| `action_model_vehicle` | One model | Opens the vehicles of the model. With at least one vehicle: board, list and form, titled "Vehicles", filtered and defaulted to the model. With none: a single form titled "Vehicle", defaulted to the model | A window action |
| `action_brand_model` | One manufacturer | Opens the models of the manufacturer as a list and form, titled "Models", filtered and defaulted to it | A window action |
| `action_open_brand_form` | One manufacturer | Opens the manufacturer's own form, titled "Manufacturer" | A window action |

### 4.3 On the Vehicle Contract

| Operation | Needs | Does | Returns |
|---|---|---|---|
| `action_open` | Any number of contracts | Sets the state to `open`. Transition T1.3 | Nothing |
| `action_draft` | Any number of contracts | Sets the state to `futur`. Transition T1.2 | Nothing |
| `action_expire` | Any number of contracts | Sets the state to `expired`. Transition T1.4 | Nothing |
| `action_close` | Any number of contracts | Sets the state to `closed`. Transition T1.1 | Nothing |
| `run_scheduler` | Invoked on the entity, not on records | Runs the four passes of the daily job. Procedure W-11 | Nothing |
| `scheduler_manage_contract_expiration` | Invoked on the entity | The four passes themselves; `run_scheduler` is the name the scheduled job calls | Nothing |
| `compute_next_year_date` | A date | Yields that date plus one calendar year. Formula C-13 | A date |
| `action_open_employee` | One contract. Added by the people bridge | Opens the form of the driver Employee of the contract's vehicle | A window action |

### 4.4 On the Vehicle Service

| Operation | Needs | Does | Returns |
|---|---|---|---|
| `action_open_account_move` | One service that carries a journal item. Added by the accounting bridge | Opens the journal entry of that item in a form, titled "Bill" | A window action |

### 4.5 On the Driver Assignment Log

| Operation | Needs | Does | Returns |
|---|---|---|---|
| `action_get_attachment_view` | One entry. Added by the people bridge | Opens the attachment browser restricted to this entry's attachments, in the derived card definition, with this entry as the default owner of new attachments | A window action |

### 4.6 On the Send Mails to Drivers assistant

| Operation | Needs | Does | Returns |
|---|---|---|---|
| `action_send` | One assistant record with vehicles and an author | Posts one message per vehicle to that vehicle's thread, addressed to its driver, or refuses when a driver has no address. Procedure W-14 | Nothing on success; a danger notification on refusal |
| `action_save_as_template` | One assistant record | Creates a message template named "Vehicle: Mass mail drivers" bound to the Vehicle, moves the acting user's attachments onto it, links them all, sets the assistant's template to it and opens it. Procedure W-15 | A window action opening the new template in a dialogue |

### 4.7 On entities of other domains

| Operation | Entity | Does |
|---|---|---|
| `action_open_employee_cars` | Employee | Opens the employee's driver assignment entries, restricted to entries naming both the employee and the employee's work contact, in the employee history list, titled "Cars History", with the employee's user contact and the employee as the defaults on new rows |
| `action_preview_attachment` | Attachment | Opens the stored file itself in a new window |
| `order_on_zip` | Batch Transfer | Sorts the batch's transfers by customer postal code and writes each transfer's position. Formula C-33 |

---

## 5. Bound operations

One operation is offered from a selection rather than from a record.

| External identifier | Label | Offered on | Layouts | What it does |
|---|---|---|---|---|
| `action_fleet_vehicle_send_mail` | "Mail to Driver" | Vehicle | The list and the board | Invokes `action_send_email` on the selected vehicles, which opens the Send Mails to Drivers assistant |

It does **not** appear on the vehicle form. A reader who wants to write to the driver of one vehicle selects that vehicle in the list.

---

## 6. Printable documents

**This domain declares no printable document of its own.** No vehicle sheet, no contract document and no service order is produced. Printing a vehicle, a contract or a service produces only the platform's generic record printout, specified in [../../interfaces/report-and-export-documents.md](../../interfaces/report-and-export-documents.md).

The transport dispatch bridge changes one document that belongs to another domain:

| Document | Owner | Change |
|---|---|---|
| The batch transfer document | [inventory operations](../inventory-operations/) | Three lines are added above the transfer table, each shown only when its value is present: the dock, labelled "Dock:"; the vehicle, labelled "Vehicle:"; and the vehicle category, labelled "Vehicle Category:". A leading column headed "Sequence" is added to the transfer table, showing each transfer's position within the batch as computed by formula C-33 |

---

## 7. Message templates and outbound messages

| Path | What is sent | To whom | Layout |
|---|---|---|---|
| The Send Mails to Drivers assistant | One message per selected vehicle, rendered per vehicle when a template is used and identical for all when it is not | The driver of each vehicle | The light notification layout |
| The driver-change tracking | An automatic tracking message on the vehicle's thread whenever the driver or the future driver changes, under the "Changed Driver" subtype | Every follower subscribed to that subtype | The platform's tracking layout |
| The service creation from a vendor bill | One message on the new service's thread, body "Service Vendor Bill: %s" with a link to the journal entry | Every follower of the service, which at creation is nobody | The platform's note layout |
| The renewal reminder | An activity, not a message; it appears in the responsible user's activity list and in their digest | The contract's responsible user | Not applicable |
| The end-date reminder | An activity of the generic to-do kind on the vehicle, note "Specify the End date of %s" with the previous driver's name | The vehicle's fleet manager, or the acting user when there is none | Not applicable |

No template is shipped; see [configuration.md](configuration.md), section 8.5. The whole outbound mechanism belongs to [../../runtime/mail-gateway.md](../../runtime/mail-gateway.md) and [../messaging-and-activities/](../messaging-and-activities/).

---

## 8. Inbound integration

**This domain publishes no route of its own, no incoming-mail alias and no webhook.** A vehicle cannot be created by sending a message to an address, and no external system can reach the fleet other than through the general transport contract of [../../interfaces/remote-transport-contracts.md](../../interfaces/remote-transport-contracts.md) and [../../interfaces/service-layer.md](../../interfaces/service-layer.md).

One address of another domain is reached by this one: the people bridge's attachment preview opens the platform's file-content path, `/web/content/`, followed by the attachment's identifier and its file name. That path is the platform's own and is specified in [../../runtime/attachments-and-file-store.md](../../runtime/attachments-and-file-store.md).

---

## 9. Import and export

Every entity of this domain is importable and exportable through the platform's generic mechanism, specified in [../../data/data-loading-and-exchange.md](../../data/data-loading-and-exchange.md). The rules below are the ones a reader importing fleet data must know.

### 9.1 Importing vehicles

1. The model is required. Supply it by its external identifier or by its display name, which is the manufacturer, a solidus and the model name.
2. Every one of the fifteen copied attributes is filled from the model **unless** the import supplies a value for it, because each is computed with an editable result and an explicitly supplied value wins.
3. Supplying a driver creates one Driver Assignment Log dated today for each imported vehicle. Importing a historic fleet therefore produces a set of assignment entries all dated on the import day; a reader who wants a truthful history imports the vehicles without drivers, imports the assignment entries with their real dates, and then sets the drivers.
4. Supplying a distance creates one Odometer Reading dated today, and a distance lower than an existing one refuses the whole import row under rule FLT-002.
5. Supplying a company is optional; an empty company makes the vehicle visible to every company.
6. The name, the derived contract state, the renewal flags, the counters, the service activity indicator, the bill count and the mobility card cannot be imported; all are derived.

### 9.2 Importing contracts

1. The vehicle is required and the recurring frequency is required.
2. Supplying the start date or the expiration date triggers the state re-evaluation, so the state supplied by the import may be overwritten in the same operation. A reader who wants a specific state imports the dates first and writes the state afterwards.
3. The name is derived unless supplied.

### 9.3 Importing services

1. The vehicle and the service kind are required.
2. A distance of zero is silently dropped; a distance greater than zero creates an Odometer Reading.
3. The journal item link should never be supplied by an import. A service imported with one becomes undeletable and its cost becomes unwritable, and no bill exists to correct it from.

### 9.4 Exporting

Both analysis result sets are exportable like any other list. Their rows carry the sequential identifier described in [entities.md](entities.md); that identifier is not stable across rebuilds of the set and must never be used as a key in an external system.

### 9.5 What the sample data set does to an import

An installation carrying the sample data has seventy-five Fleet Service Types and seven Vehicle Statuses rather than three and four. An import that matches service kinds or statuses by name therefore behaves differently on the two kinds of installation. A reader importing into an unknown database should match by external identifier, which is stable.
