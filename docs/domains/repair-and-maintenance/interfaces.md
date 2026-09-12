# Interfaces

The named operations a client or an integration invokes, the screens described as views with their fields and buttons, the printed document, the incoming-mail channel, the notifications, the counters other screens read, the deep-link path pattern of every screen, and the scheduled execution this domain relies on.

A screen is a view over an entity with a stated shape: list, form, kanban board, calendar, pivot table, graph, activity board or search panel. A button is a control that invokes a named operation with the record, or with the current selection, as its subject. A screen is reached either from a menu entry or from a deep link built out of the screen's path pattern, which section 14 lists for every screen that carries one. Hint text is the greyed illustrative value a field shows while it is empty; the hint texts quoted below are written in full words, so a rebuild shows an equivalent hint rather than the same abbreviation.

---

## 1. Named operations on a Repair Order

Every operation takes the Repair Order or a set of Repair Orders as its subject. Operations marked "single subject" refuse a set.

| Operation | Inputs | Output | Side effects | Errors |
|---|---|---|---|---|
| Validate and confirm a repair | single subject | either nothing, or a request to open the insufficient-quantity dialogue with its pre-filled values | When the checks pass: adjusts the procurement method of every part, confirms every part, triggers the replenishment scheduler, writes the state to `confirmed`. | "You can not enter negative quantities." when a part has a negative demand; the platform's company-consistency error. |
| Confirm from the insufficient-quantity dialogue | single subject: the dialogue record | nothing | Clears the surrounding context of stray defaults and runs the confirmation proper on the dialogue's Repair Order. | The company-consistency error. |
| Start the repair | a set | nothing | Confirms any subject still new, then writes `under_repair` on every subject. | Those of the confirmation operation. |
| End the repair | a set | nothing | Runs the completion operation. | "Repair must be under repair in order to end reparation." when any subject is not under repair. |
| Complete the repair | a set | nothing | Cancels zero-quantity parts, marks parts picked, sets the originating service line's delivered quantity, creates the repaired-product movement, completes every movement without backorder, writes `done`. | "Serial number is required for product to repair : " followed by the product's display name, when a tracked product has no lot. |
| Cancel the repair | a set | nothing | Zeroes the originating Sales Order Line, cancels every part, writes `cancel`. | "You cannot cancel a Repair Order that's already been completed" when any subject is completed. |
| Set the repair back to new | a set | nothing | Cancels first when needed, restores the quantities of the part Sales Order Lines, writes `draft` on every part and on the repair. | Those of the cancellation operation. |
| Reserve the repair's parts | a set | the result of the reservation procedure | Reserves every part from its source location. | none of its own. |
| Unreserve the repair's parts | a set | nothing | Releases the reservations of every fully or partially available part. | none of its own. |
| Create a quotation from the repair | a set | a request to open the created Sales Order | Creates one Sales Order per subject, links it, and creates one Sales Order Line per `add` part. | "You cannot create a quotation for a repair order that is already linked to an existing sale order." and "You need to define a customer for a repair order in order to create an associated quotation.", each followed by "Concerned repair order(s):" and the offending references, one per line. |
| Open the repair's Sales Order | single subject | a request to open the Sales Order form | none | none |
| Generate a serial number for the product to repair | single subject | nothing | Creates a Lot or Serial Number for the product and writes it onto the repair. | "Please set the first Serial Number or a default sequence" when no name can be produced; the lot-creation refusal of rule RM-050 when the operation type forbids new lots. |
| Print the repair order | a set | a request to render the printed Repair Order | none | none |
| Open the product catalog | single subject | a request to open the catalog screen, restricted to goods, with the repair-specific search panel | none | none |
| Set a catalog quantity | single subject, a product and a quantity | the product's list price | Sets the demand of the existing `add` part, deletes it when the quantity is zero, or creates a new `add` part when none exists and the quantity is positive. | none |
| Explode the repair's kits | a set | nothing | Replaces every kit part by its components. Invoked automatically after creation and after every write. | none |
| Open the repair's Manufacturing Orders | single subject | a request to open the single production, or the list of them | none | none |
| Open the repair's Purchase Orders | single subject | a request to open the single Purchase Order, or the list of them | none | none |
| Open the repair's movement lines | single subject | a request to open the detail lines of the repair, in list then form shape | none | none |

## 2. Named operations invoked from other entities

| Operation | Subject | Output | Side effects | Errors |
|---|---|---|---|---|
| Create a repair from a transfer | single Transfer | a request to open a new Repair Order form whose context carries the transfer, the repair operation type of the transfer's warehouse and the transfer's contact | none until the form is saved | none |
| Open a transfer's repairs | a Transfer | a request to open the single linked repair, or the list of them | none | none |
| Open the repairs of a lot | single Lot or Serial Number | a request to open the Repair Orders of that lot, with a context that pre-fills a new repair with the lot's product, the lot and the lot's company | none | none |
| Open the repairs that used a lot | single Lot or Serial Number | a request to open the single Repair Order that consumed the lot, or the list titled "Repair orders of " followed by the lot name | none | none |
| Open a Sales Order's repairs | single Sales Order | a request to open the single bound repair, or the list of them | none | none |
| Open a Manufacturing Order's repairs | single Manufacturing Order | a request to open the single fed repair, or the list titled "Repair Source of " followed by the production reference | none | none |
| Open a Purchase Order's repairs | single Purchase Order | a request to open the single fed repair, or the list titled "Repair Source of " followed by the purchase reference | none | none |
| Open an operation type's repairs | an Operation Type | a request to open the Repair Orders of that type, titled with the type's display name | none | none |
| Open the confirmed repairs of an operation type | an Operation Type whose code is `repair_operation` | a request to open the Repair Orders screen filtered on confirmed repairs, replacing the ordinary transfer graph action | none | none |
| Archive a maintenance request | a set of Maintenance Requests | nothing | Writes the archive flag true and the recurrence flag false. | none |
| Reopen a maintenance request | a set of Maintenance Requests | nothing | Writes the archive flag false and the stage with the lowest sequence. | none |
| Open the matching serial number | single Equipment | a request to open the single matching lot, or the list of matching lots | none | returns success and does nothing when the lot screen is unavailable |
| Open a location's equipment | single Location | a request to open the Equipment of that location | none | none |
| Open an equipment's maintenance requests | single Equipment | a request to open the requests of that equipment, with the Active filter applied and the equipment, company and team pre-filled on a new request | none | none |
| Open a category's equipment | single Equipment Category | a request to open the equipment of that category, pre-filled and filtered on it | none | none |
| Open a category's maintenance requests | single Equipment Category | a request to open the requests of that category, with the Active filter applied | none | none |
| Open a team's requests | single Maintenance Team | a request to open the requests of that team, filtered by the maintenance kind carried in the screen context, defaulting to both kinds | none | none |
| Register a departure, extended | a set of Employees | the ordinary departure result | Additionally clears the equipment of each departing employee when the Free Equiments option is ticked. | none of its own |

---

## 3. Screens: Repair Order

### 3.1 Repair Order list

**Rows.** Repair Orders, ordered by priority descending then creation date descending. Rows of new repairs are rendered in an informational colour. Several rows can be edited at once. The screen shows sample data when it is empty.

| Column | Shown by default | Notes |
|---|---|---|
| Priority | yes | Rendered as a star. |
| Reference | yes | |
| Scheduled Date | yes | Rendered as a number of remaining days. |
| Product to Repair | yes | Read-only. |
| Component Status | yes | Hidden unless the repair is confirmed or under repair. Coloured green when available, amber when expected, red when late. |
| Product Quantity | no | Read-only unless the repair is new. |
| Unit | no | Requires the unit of measure group. Read-only. |
| Responsible | no | Rendered as an avatar. |
| Customer | yes | Read-only. |
| Transfer | no | |
| Sale Order | yes | |
| Component Source Location | no | |
| Company | yes | Requires the multiple companies group. Read-only. |
| Status | yes | Rendered as a badge: green for Repaired, informational for Confirmed, amber for Under Repair, red for Cancelled, muted for New. |
| Activity exception marker | yes | |

**Empty-state message.** "No repair order found. Let's create one!" followed by "In a repair order, you can detail the components you remove, add or replace and record the time you spent on the different operations."

### 3.2 Repair Order form

**Status bar.** New, Confirmed, Under Repair, Repaired. Cancelled is reachable but is not shown as a step.

**Buttons in the header, with their guards.**

| Button | Shown when | Invokes | Extra behaviour |
|---|---|---|---|
| Confirm Repair | the state is New | the confirmation operation | |
| Start Repair | the state is Confirmed | the start operation | |
| End Repair | the state is Under Repair and at least one part is short | the end operation | Asks first: "For some of the parts, there is a difference between the initial demand and the actual quantity that was used. Are you sure you want to confirm ?" |
| End Repair | the state is Under Repair and no part is short | the end operation | No prompt. |
| Check availability | at least one part can still be reserved | the reserve operation | |
| Unreserve | at least one part holds a reservation | the unreserve operation | |
| Create Quotation | a customer is set, the state is not Cancelled, and no Sales Order is linked | the quotation operation | |
| Cancel Repair | the state is neither Repaired nor Cancelled | the cancel operation | |
| Set to Draft | the state is Cancelled | the reset operation | |

**Buttons above the sheet.**

| Button | Shown when | Opens |
|---|---|---|
| Sale Order | a Sales Order is linked | That Sales Order. |
| Product Moves | the state is Repaired or Cancelled | The detail lines of the repair. |
| Manufacturing Orders, with the count | the manufacturing bridge is installed, the reader holds the manufacturing user group and the count is above zero | The productions feeding the repair. |
| Purchase Orders, with the count | the purchasing bridge is installed, the reader holds the purchase user group and the count is above zero | The purchases feeding the repair. |

**Title block.** The priority star and the reference.

**Left column.** Repair Request, shown only when the repair came from a Sales Order Line; Customer, read-only when a Sales Order is linked; Product to Repair, read-only when Repaired or Cancelled; Lot/Serial with a plus button that generates a serial number — the block is shown only when the product is tracked, the field is read-only when Repaired, required when Repaired and tracked, and requires the lot and serial number tracking group; Product Quantity and Unit in a developer-only block, the quantity read-only for serial-tracked products and when Repaired or Cancelled, the unit read-only unless the state is New; Transfer, shown only when one is linked and not creatable from here; Under Warranty, read-only when Repaired or Cancelled.

**Right column.** Scheduled Date, read-only when Repaired or Cancelled; Responsible, restricted to users who are not shared accounts; Company, requiring the multiple companies group; Tags, rendered as chips coloured by the tag colour and not creatable from here; Component Status, shown only when Confirmed or Under Repair and coloured as in the list; Operation Type, read-only when Repaired or Cancelled, hidden when the company owns only one repair operation type, and not creatable from here.

**Properties block.** The ad-hoc fields defined by the operation type, in two columns.

**Parts tab, a list edited in place.** The line control offers "Add a line" and a "Catalog" button that opens the catalog for this repair. New lines default to a demanded quantity of one, the repair's company, the repair's scheduled date and the part kind `add`.

| Column | Notes |
|---|---|
| Type | Required. The three values are Add, Remove and Recycle. |
| Product | Required. Read-only once the movement has left the new state, unless it was added after confirmation, and read-only once the movement has detail lines. |
| Forecast | A readiness marker per line. Hidden when the repair is Repaired. |
| Description | An optional column. |
| Date, Deadline | Optional columns. |
| Demand | Read-only when the movement is Done or Cancelled. |
| Quantity | The recorded quantity. Read-only until a product is chosen. |
| Unit | Read-only once the movement has left the new state, unless it was added after confirmation. |
| Picked | An optional column. |
| Lots/Serials | Chips. Requires the lot and serial number tracking group. Shown only for serial-tracked products whose details are available. Read-only until the line is saved. |
| Details | A button, shown when the movement has details to show. For a Recycle line it opens with the on-hand picker hidden and the destination location shown. |

The whole tab is read-only when the repair is Cancelled or Repaired.

**Repair Notes tab.** The internal notes, with the hint text "Add internal notes."

**Discussion thread.** Below the sheet, with followers, messages and activities.

### 3.3 Repair Order kanban board

Cards show the reference, the status as a coloured label, the product, the tags and the customer. A progress bar across the top groups the cards by activity state: planned in green, today in amber, overdue in red. Quick creation is disabled.

### 3.4 Repair Order search panel

**Searchable fields.** Repair Order, which matches the reference or the product name; Product to Repair; Customer, matching the customer and its children; Sale Order.

**Filters.** New, Confirmed, Under Repair, Repaired, Cancelled, Returned (a repair whose linked transfer is validated), Late (confirmed and scheduled before today), and a period filter on the creation date. Six hidden filters expose the scheduled-date categories Before, Yesterday, Today, Tomorrow, The day after tomorrow and After; a hidden Ready filter selects confirmed repairs with all parts available. Four hidden activity filters select My Activities, Late Activities, Today Activities and Future Activities.

**Groupings.** Customer, Product, Status, Company (requiring the multiple companies group), Properties.

### 3.5 Repair Order analysis

A graph screen over the creation date and the product, and a pivot table with the creation date in rows and the product in columns. The Repairs entry of the repair reporting menu opens the graph with the product grouping and the creation-date period already applied.

### 3.6 Repair Order activity board

Each row shows the responsible user as an avatar, the reference in bold, the product and the scheduled date.

---

## 4. Screens: repair-related additions to other entities

| Screen | Addition |
|---|---|
| Operation Type form | For a repair type: the Product Source Location and the Product Destination Location appear before the component source location; the component source and destination labels change to "Component Source Location" and "Component Destination Location"; the Remove Destination Location and the Recycle Destination Location appear after the component destination; the lot policy block becomes visible; the backorder policy is hidden. |
| Inventory overview card of a repair operation type | The card links to the Repair Orders of the type. Its primary button reads the ready count followed by "To Repair" when at least one repair is ready, and "Open" otherwise. Secondary links appear for Late, Confirmed and Under Repair when their counts are above zero. The card menu offers Orders (All, Ready), a New link that opens a blank repair for this type, and a Reporting link. A bar graph of confirmed repairs by scheduled date is drawn on the card, and clicking it opens the confirmed Repair Orders rather than transfers. |
| Transfer form | A Repair Orders button appears, with the count, when at least one repair is linked. A create-repair contextual action is available on the form. |
| Lot or Serial Number form | Two buttons: "Repair Parts: " followed by the count, opening the repairs that consumed the lot; and a button showing "To Do: " with the in-repair count and "Done: " with the repaired count, opening the repairs of the lot. |
| Sales Order form | A Repairs button with the count, requiring the inventory user group, shown only when at least one repair is linked. |
| Manufacturing Order form and Purchase Order form | A Repair Source button with the count, requiring the inventory user group. |
| Warehouse form | The repair operation type is shown, read-only, after the outgoing operation type. |
| Product form | The service tracking field is unconditionally visible, so that the Repair Order value can be chosen. |
| Product catalog search panel | Two filters are added: "In the Repair Order", shown only when the catalog was opened from a Repair Order, and "Bill of Materials Components", which selects the components of the repaired product's bill of materials. |
| Traceability report | A detail line whose movement belongs to a repair reports the Repair Order as its source document; the consumed and produced links of the repaired-product movement are followed when the ordinary rules find nothing. |
| Forecasted stock report | Repair part movements contribute no reservation data, and a movement bound both to a repair as an `add` part and to a Sales Order Line is counted once, as a repair. |

---

## 5. Screens: Equipment and categories

### 5.1 Equipment form

**Buttons above the sheet.**

| Button | Shown when | Opens |
|---|---|---|
| Serial Number | the serial number matches at least one registered lot | The matching lot, or the list of them. |
| Maintenance, with the open count | always | The Maintenance Requests of this equipment, with the Active filter, pre-filling the equipment, company and team on a new request. |

An "Archived" ribbon is drawn when the record is archived.

**Title.** The equipment name, with the hint text "for example, Light Emitting Diode Monitor".

**Left column.** Equipment Category, not openable from here; Company, requiring the multiple companies group, with the hint text "Visible to all"; and the assignment block. Without the people bridge the block is the Owner field. With it, the block is the Used By choice, then Employee, hidden when the choice is Department, and Department, hidden when the choice is Employee.

**Right column.** Maintenance Team; Technician, restricted to users who are not shared accounts; Assigned Date, a developer-only field; Scrap Date, a developer-only field; and, with the inventory bridge, the internal location labelled "Used in location".

**Properties block.** The ad-hoc fields defined by the category, in two columns.

**Description tab.** The note.

**Product Information tab.** Vendor, Vendor Reference, Model, Serial Number in one group; Effective Date, Cost (requiring the equipment manager group) and Warranty Expiration Date in the other.

**Maintenance tab.** Five statistics with their labels reproduced as "Expected MTBF" in days — the only editable one — "MTBF" in days, "Estimated Next Failure", "Latest Failure" and "MTTR" in days. Those five labels are reproduced as the system shows them; in the prose of this repository they are called the expected mean time between failures, the mean time between failures, the estimated next failure, the latest failure date and the mean time to repair.

**Discussion thread.** Below the sheet.

### 5.2 Equipment kanban board

Cards show the name in bold with the model designation in brackets, the serial number, the ad-hoc properties, a red badge reading the open count followed by "Request" when open requests exist, the activity marker and the owner's avatar. With the people bridge the card also shows the employee and the department, or the word "Unassigned" when no employee is set. A progress bar groups cards by activity state.

### 5.3 Equipment list

Columns: Name, Owner, Assigned Date (a developer-only column), Serial Number, Technician, Equipment Category, Company (requiring the multiple companies group), and the activity exception marker. With the people bridge, the Owner column is replaced by Employee and Department.

### 5.4 Equipment search panel

**Searchable fields.** Equipment, matching the name, the model designation, the serial number or the vendor reference; Category; Owner. With the people bridge, Employee and Department are searchable too.

**Filters.** My Equipment, Assigned, Unassigned, Under Maintenance (the open count above zero), Unread Messages, Archived, plus the four hidden activity filters. With the people bridge, Assigned means that an employee or a department is set and Unassigned means that neither is.

**Groupings.** Technician, Category, Owner, Vendor, Properties. With the people bridge, Employee and Department are added.

### 5.5 Equipment Category form

Two buttons above the sheet: Equipment with its count, and Maintenance with the open count. The title is the category name with the hint text "for example, Monitors". Below it: Responsible, restricted to users who are not shared accounts, and Company, requiring the multiple companies group, with the hint text "Visible to all"; then the comments.

### 5.6 Equipment Category list, kanban board and search panel

The list shows Name, Responsible and Company. The kanban card shows the name, "Equipment: " with the count, "Maintenance: " with the count and the responsible user's avatar. The search panel searches the category name and groups by Responsible.

---

## 6. Screens: Maintenance Requests

### 6.1 The pipeline, a kanban board

Columns are the Maintenance Stages, ordered by sequence, every stage shown even when empty, and collapsed when the stage's folding flag is set. A progress bar per column splits the cards by within-stage signal, green for Ready for next stage and red for Blocked.

Cards show the subject in bold, "Requested by: " followed by the created-by user — replaced by the employee with the people bridge — the equipment with its category in brackets, the scheduled date, the priority stars, the activity marker, an amber "Cancelled" badge when archived, the within-stage signal, and the technician's avatar. The card menu offers Edit, Delete and a colour picker.

### 6.2 Maintenance Request form

**Header buttons.**

| Button | Shown when | Invokes |
|---|---|---|
| Cancel | the request is not archived | the archive operation |
| Reopen Request | the request is archived | the reopen operation |

**Status bar.** The stages, clickable, hidden when the request is archived; an amber "Cancelled" badge is shown in its place.

**Within-stage signal.** Rendered at the top of the sheet as a three-state selector.

**Title.** The subject, labelled "Request", with the hint text "for example, Screen not working".

**Left column.** Created By — the created-by user, or the employee with the people bridge; Equipment, pre-filling the company and category on a new equipment; Category, requiring the equipment manager group and shown only when an equipment is chosen; Request Date, read-only; Close Date, read-only and shown only when the stage is a closing stage; Maintenance Type as a two-value choice.

**Right column.** Team, neither creatable nor openable from here; Responsible, meaning the technician; Scheduled Date; Scheduled End; the Recurrent switch, hidden for corrective requests; the recurrence block, hidden unless Recurrent is on, holding Repeat Every, the unit, the Until choice and the End Date, the last shown and required only for the `until` choice; Priority as stars; the carbon-copy addresses, a developer-only field; Company, requiring the multiple companies group.

**Notes tab.** The description, with the hint text "Internal Notes".

**Instructions tab.** The instruction medium as a three-value choice, then exactly one of: an uploaded document in the portable document format rendered in a document viewer, required for that choice, with the help text "Upload your file."; a slide-deck address rendered in a slide viewer, required for that choice, with the hint text "Google Slide Link"; or a rich text area, with the hint text "Your instructions".

**Discussion thread.** Below the sheet.

### 6.3 Maintenance Request list

Columns: Subject, Request Date (a developer-only column), Created By, Technician, Category (requiring the equipment manager group, read-only), Stage, Company (requiring the multiple companies group, read-only), and the activity exception marker. Several rows can be edited at once.

### 6.4 Maintenance Request calendar

The events run from the scheduled date to the scheduled end, are coloured by technician, and at most five are shown per day before collapsing. The technician is offered as a side filter. The Maintenance Calendar entry opens with the Active and To Do filters applied.

**Two kinds of event.** The calendar shows one **real event** for every Maintenance Request whose scheduled date falls on or before the end of the displayed range, and, on top of those, one **projected event** for every future occurrence a recurrent request would produce inside the displayed range. A projected event is a rendering of the same request at a later moment; it is not a record and nothing is written for it. The complete projection rule is calculation 32 in [calculations.md](calculations.md).

**Which requests project.** A request projects occurrences when its recurrence flag is true, its mirrored closing flag is false and its archive flag is false. A request that has reached a closing stage, or that has been cancelled, stops projecting at once, which is what makes a completed series disappear from the future of the calendar.

**Range rule.** The records are fetched with the single condition that the scheduled date is not later than the end of the displayed range, deliberately without a lower bound, so that a request whose own scheduled date lies far in the past still loads and can project its occurrences into the displayed range. Without that rule, a weekly series started three months ago would show nothing this month.

**Label.** A projected event carries the request's display name followed by a space, an opening parenthesis, a plus sign, the occurrence number counted from the request's own scheduled date, and a closing parenthesis.

**Length of an event.** A real event runs from the scheduled date to the scheduled end. When the scheduled end is empty, the event runs for the duration in hours from the start. A projected event runs for the duration in hours from the projected start, and for exactly one hour when the duration is zero.

**Editing by drag.** A real event can be dragged and resized; doing so writes the scheduled date, and the scheduled end follows it. A projected event is locked: dragging or resizing it has no effect at all and writes nothing, because the occurrence has no record of its own to carry the new moment. Moving a series therefore means moving its real event.

**Opening an event.** Double-clicking any event, real or projected, opens the form of the one real Maintenance Request behind it, never a copy and never a blank record. The same substitution applies to the Edit and Delete controls of the event's summary bubble, and to the click on an occurrence in the year shape. A user who opens the third projected occurrence of a weekly series and renames it is renaming the one request behind the whole series.

### 6.5 Maintenance Request analysis

A graph over the technician and the stage, measuring the duration; and a pivot table over the technician and the stage. The Maintenance Requests Analysis entry opens the graph with the Active filter applied.

### 6.6 Maintenance Request activity board

Each row shows the technician's avatar, the subject in bold and the equipment beneath it.

### 6.7 Maintenance Request search panel

**Searchable fields.** Request, meaning the subject; Category; Technician; Equipment; Created by User; Stage; Team. With the people bridge, Employee is searchable too.

**Filters.** My Maintenances (requests whose technician is the acting user; with the people bridge, requests whose employee is the acting user), To Do (the stage does not close), Done (the stage closes), Blocked (not closed and blocked), Ready (not closed and ready for the next stage), High-priority (not closed and priority High), Unscheduled (not closed and no scheduled date), period filters on the request date, the scheduled date and the close date, Unread Messages, Active (not archived), Cancelled (archived), and the four hidden activity filters.

**Groupings.** Assigned to, Category, Stage, Created By — by created-by user, or by employee with the people bridge.

---

## 7. Screens: Maintenance Teams

### 7.1 Team dashboard

A kanban board of teams, with creation disabled and cards that do not open the record directly.

Each card shows the team name, a primary button reading the open count followed by "To Do", and up to four secondary links, each shown only when its count is above zero:

| Link | Opens |
|---|---|
| the scheduled count followed by "Scheduled" | The calendar filtered on this team's open requests. |
| the high-priority count followed by "Top Priorities" | The team's open requests filtered on High priority. |
| the blocked count followed by "Blocked" | The team's open requests filtered on Blocked. |
| the unscheduled count followed by "Unscheduled" | The team's open, unscheduled requests. |

The card menu offers, under Requests: All, To Do, In Progress and Done; under Reporting: Maintenance Requests; plus a colour picker and a Configuration link that opens the team form.

### 7.2 Team form

Title: the team name, with the hint text "for example, Internal Maintenance". An "Archived" ribbon when archived. Then: Team Members as chips, restricted to users who are not shared accounts and not creatable inline; the Email Alias block, showing the alias address when reading and the local part, an at sign and the alias domain when editing; and the Company, requiring the multiple companies group, with the hint text "Visible to all".

### 7.3 Team list and search panel

The list, edited in place, shows Team Name, Team Members and Company. The search panel searches the team name and offers an Archived filter.

---

## 8. Screens: Maintenance Stages

A list edited from the top, with a drag handle on the sequence, then Name, "Folded in Maintenance Pipe" and "Request Done". A kanban board shows the stage names. A search panel searches the stage name. The screen is reachable only through the developer-only Maintenance Stages entry.

---

## 9. Printed document: Repair Order

**Trigger.** The print operation on a Repair Order, or the contextual print action on a Repair Order selection.

**One document per Repair Order**, rendered in the language of that repair's customer, inside the company's external document layout. The file is named "Repair Order - " followed by the reference.

**Content, in order.**

1. A heading reading "Repair Order #" followed by the reference.
2. An information block in two columns:
   - left: "Customer:" and the customer, shown only when one is set; "Product:" and the product to repair, shown only when one is set; "Lot/Serial:" and the lot, shown only when one is set and only to holders of the lot and serial number tracking group;
   - right: "Status:" and the state label; "Responsible:" and the responsible user, shown only when one is set.
3. A heading "Parts" over a two-column table:
   - "Description": for each part, the part kind in italic brackets, "(Add)", "(Remove)" or "(Recycle)", followed by the product name;
   - "Quantity", right aligned: the demanded quantity, followed by the unit name for holders of the unit of measure group.

   Every part of the repair is listed, whatever its kind and whatever its state, cancelled parts included.
4. A heading "Repair Notes" over the internal notes, shown only when notes exist.

**Totals.** None. The printed Repair Order shows no price, no cost and no total; it is a shop-floor document, not a commercial one. The commercial document for a repair is the quotation or the invoice produced through the Sales Order.

**Layout hooks.** Four editable regions are provided, before the heading, after the heading, before the parts table and after the notes, so that a deployment can insert its own content without replacing the document.

**Localization.** With the commercial document-layout bridge installed, the printing date of the Repair Order is stamped on the document. See [../fiscal-localizations/](../fiscal-localizations/).

---

## 10. Incoming-mail channel

**Address.** The alias local part and the alias domain configured on a Maintenance Team.

**Target entity.** Maintenance Request.

**Forced values.** The alias's default values force the team to the owning team; any other default values already stored on the alias are preserved and merged.

**Mapping of the incoming message.**

| Message part | Becomes |
|---|---|
| Subject | The request's subject. |
| Body | The first message of the request's discussion thread. |
| Sender | With the people bridge: the sender's address is normalised and matched against user login names; when a user matches, the employee record of the acting processing user is written onto the request. |
| Carbon-copy addresses | Retained on the request, so that later replies reach the same people. |
| Attachments | Attached to the first message of the thread. |

**After creation.** The ordinary creation rules run: the request lands in the stage with the lowest sequence, the close-date correction runs, the created-by user and the technician are subscribed, the "Request Created" message is posted — which reaches the followers of the equipment's category through the category-level subtype — and an activity is scheduled when the request carries a scheduled date.

---

## 11. Notifications

| Event | Channel | Recipients |
|---|---|---|
| A Maintenance Request is created | The "Request Created" subtype, hidden on the request itself and not subscribed by default there | Through the category-level "Maintenance Request Created" subtype, the followers of the equipment's category, who are subscribed by default. |
| A Maintenance Request changes stage | The "Status Changed" subtype, subscribed by default | The followers of the request: its created-by user, its technician and, with the people bridge, its employee's user. |
| An equipment is assigned | The "Equipment Assigned" subtype | The followers of the equipment, and, through the category-level counterpart subscribed by default, the followers of the equipment's category. |
| A Maintenance Request is scheduled or rescheduled | A scheduled activity of the maintenance activity kind | The request's technician, falling back to its created-by user, falling back to the acting user. The activity note reads "Request planned for " followed by a link to the equipment when one is named. |
| A Maintenance Request reaches a closing stage | The pending activity is marked done with feedback | The activity leaves the responsible person's to-do list; the completion is recorded in the thread. |
| A Repair Order changes state, or gains its product move | Tracked field changes | The followers of the Repair Order. A repair posts messages with the author-mention behaviour forced on, which means the author of a message is notified of their own mention, unlike the platform default. |

---

## 12. Scheduled execution

This domain declares no scheduled job of its own. It relies on scheduled execution owned elsewhere in two places: the replenishment scheduler, triggered synchronously when a Repair Order is confirmed or a part is created on a running repair, and the periodic activity digest that surfaces the deadlines created by Maintenance Requests. Both are set out in [configuration.md](configuration.md), section 7.

Neither the readiness fields of a repair nor the effectiveness measurements of an equipment are refreshed on a schedule; they are derived on read.

---

## 13. Counters exposed to other screens

| Counter | Entity | Consumer |
|---|---|---|
| The ready, confirmed, under-repair and late counters | Operation Type | The inventory overview card. |
| The repair count | Transfer | The Repair Orders button on the transfer form. |
| The repair count | Sales Order | The Repairs button on the Sales Order form. |
| The repair count | Manufacturing Order, Purchase Order | The Repair Source button on those forms. |
| The manufacturing and purchase counters | Repair Order | The buttons that open the feeding documents. |
| The in-repair count, the repaired count and the repair-part count | Lot or Serial Number | The two repair buttons on the lot form. |
| The equipment count | Equipment Category, Employee, Public Employee Profile, Location | The Equipment buttons on those forms. |
| The maintenance count and the open maintenance count | Equipment, Equipment Category | The Maintenance buttons on those forms. |
| The five dashboard counters | Maintenance Team | The team dashboard card. |
| The serial-match flag | Equipment | The Serial Number button on the equipment form. |

---

## 14. Deep-link path patterns of the screens

Every screen of this domain that is reachable from a menu entry also carries a stable path, so that a screen, or one record on it, can be addressed by a single web address and shared, bookmarked or sent in a notification. The address is built from three parts: the location of the deployment, one interface root segment that identifies the interactive interface — the [platform foundation](../platform-foundation/) domain owns that segment and states that a rebuild chooses its own single-segment root — and the screen path listed below. The patterns in this section are written relative to the interface root segment.

### 14.1 The three shapes of a screen address

| Path pattern | Method | Authentication | Purpose | Request | Response |
|---|---|---|---|---|---|
| the screen path | read | an authenticated internal user holding the groups of the corresponding menu entry | Open the screen in its first view shape, with the filters and the groupings carried by the screen's own context. | Optional search and grouping parameters carried in the query part of the address. | The interface document, which then loads the screen's records. |
| the screen path, a slash, and a record identifier | read | the same | Open the form of exactly that record of the screen's entity. The identifier is the surrogate whole-number primary key. | none | The interface document, positioned on that record's form. |
| the screen path, a slash, and the literal segment `new` | read | the same | Open a blank form of the screen's entity, with the screen's own default values already applied. The literal segment `new` is reserved and can never be a record identifier. | none | The interface document, positioned on an unsaved form. |

Three further rules govern these addresses.

1. **Unknown path.** A path segment that matches no screen path and is not a numeric identifier is treated as an action reference rather than as a path, which lets a screen with no path of its own still be addressed. Nothing in this domain relies on that fallback for a menu entry.
2. **Access.** The path grants nothing. The reader still passes the model access of rules RM-110 and RM-250, the record rules RM-111, RM-251, RM-252, RM-253 and RM-254, and the menu visibility of rules RM-113 and RM-256. A reader who follows the path to a record they may not read receives the ordinary access refusal, not an empty screen.
3. **Chaining.** Segments may be chained so that the address records the trail a user walked, for example a repair operation type followed by the repair screen it opened. The last segment of the chain is the screen that is displayed; the earlier ones form the breadcrumb.

### 14.2 The shipped screen paths of this domain

| Screen path | Screen | Entity | View shapes, in the order offered | Reached from | Groups required |
|---|---|---|---|---|---|
| `/repairs` | Repair Orders | Repair Order | list, kanban board, graph, pivot table, form, activity board | The Orders entry of the Repairs menu. | Inventory User |
| `/repair-orders-analysis` | Repair Orders Analysis | Repair Order | list, kanban board, graph, pivot table, form | The Repairs entry of the repair reporting menu. It opens on the graph, grouped as the reporting context dictates. | Inventory Administrator, through the reporting section |
| `/maintenance` | Maintenance Teams dashboard | Maintenance Team | kanban board, form | The Dashboard entry of the Maintenance menu. | Equipment Manager or Internal User |
| `/maintenance-requests` | Maintenance Requests | Maintenance Request | kanban board, list, form, pivot table, graph, calendar, activity board | The Maintenance Requests entry of the Maintenance section. It opens on the kanban pipeline grouped by stage, with the Active filter applied. | Equipment Manager or Internal User |
| `/maintenance-calendar` | Maintenance Calendar | Maintenance Request | calendar, kanban board, list, form, pivot table, graph, activity board | The Maintenance Calendar entry of the Maintenance section. It opens on the calendar with the Active and To Do filters applied. | Equipment Manager or Internal User |
| `/maintenance-requests-analysis` | Maintenance Requests Analysis | Maintenance Request | graph, pivot table, kanban board, list, form, calendar, activity board | The Maintenance Requests Analysis entry of the maintenance reporting section. It opens on the graph with the Active filter applied. | Equipment Manager or Internal User |
| `/equipments` | Equipment | Equipment | kanban board, list, form | The Equipment entry of the Maintenance menu. It opens on the kanban board grouped by category. | Equipment Manager or Internal User |
| `/equipement-categories` | Equipment Categories | Equipment Category | list, kanban board, form | The Equipment Categories entry of the maintenance configuration section. | the Configuration section itself, which requires Equipment Manager |
| `/maintenance-stages` | Stages | Maintenance Stage | list, kanban board, form | The Maintenance Stages entry of the maintenance configuration section, a developer-only entry. | Equipment Manager plus the developer visibility group |
| `/maintenance-teams` | Teams | Maintenance Team | list, kanban board, form | The Maintenance Teams entry of the maintenance configuration section. | Equipment Manager |
| `/maintenance-activity-types` | Activity Types | Activity Kind | list, kanban board, form | The Activity Types entry of the maintenance configuration section, a developer-only entry. It is narrowed to activity kinds that are either generic or aimed at Maintenance Request, and it creates new ones aimed at Maintenance Request. | Equipment Manager plus the developer visibility group |

The spelling of the equipment-categories path is reproduced exactly as it ships, misspelling included, because an address that a user may have bookmarked has to keep resolving; a rebuild that silently corrects the spelling breaks those bookmarks. This is recorded as a **compatibility finding**: a corrected behaviour would serve the corrected spelling as the canonical path and keep the shipped spelling as a permanent alias.

### 14.3 Screens of this domain that carry no path

These screens are opened only by a button, by a menu entry that passes its own context, or by another screen, and therefore carry no shareable path. Addressing them requires the action-reference fallback of rule 1 above.

| Screen | Entity | Opened from |
|---|---|---|
| Repair Orders of one operation type | Repair Order | The inventory overview card of a repair operation type. |
| Repair Orders, confirmed | Repair Order | The graph of the inventory overview card of a repair operation type. |
| Repair Order form, single record | Repair Order | The Repair Source buttons on a Manufacturing Order, a Purchase Order, a Sales Order, a Transfer and a Lot or Serial Number, when exactly one repair is linked. |
| Inventory moves of one repair | Stock Move Line | The developer-only Inventory Moves entry on the Repair Order form. |
| Repair Orders Tags | Repair Tag | The developer-only Repair Orders Tags entry of the repair configuration section. |
| Maintenance Settings | Configuration Settings | The Settings entry of the maintenance configuration section. |
| Maintenance Requests of one equipment | Maintenance Request | The Maintenance button on the Equipment form, and the action bound to that form. |
| Maintenance Requests of one category | Maintenance Request | The Maintenance button on the Equipment Category form. |
| Maintenance Requests of one team | Maintenance Request | The five counters of the team dashboard card, each with its own filter. |
| Equipment of one category | Equipment | The Equipment button on the Equipment Category form. |
| Equipment of one employee, of one public employee profile or of one location | Equipment | The Equipment button on those forms. |
| The insufficient-quantity dialogue | Insufficient Repair Quantity Warning | The confirmation of a Repair Order whose product to repair is short. |
| The product catalog for one repair | Product Variant | The Catalog button of the Parts tab on the Repair Order form. |

---

## 15. Import and export

This domain adds no import or export format of its own. Every entity it owns is readable and writable through the platform's ordinary record transport, described in [../../interfaces/remote-transport-contracts.md](../../interfaces/remote-transport-contracts.md), using the transport names listed in [README.md](README.md) and the field names listed in [entities.md](entities.md). Two consequences are worth stating for an integration:

1. Creating a Repair Order through the transport applies exactly the creation rules of [workflows.md](workflows.md), workflow 1, including the numbering, the Stock Reference and the location derivation. Supplying a reference other than the literal `New` suppresses the numbering.
2. Writing a repair-bound Stock Move through the transport applies the part-creation rules of workflow 5, including the immediate confirmation of a part added to a running repair.

---

## Reconciliation notes

1. **The manufacturing and purchase buttons on the Repair Order form.** One earlier text listed the named operations that open those documents but not the buttons that invoke them; the other listed the buttons. Both are here, in sections 1 and 3.2.
2. **The traceability and forecast report changes.** One earlier text described them as entity extensions only. They also change two reporting screens, so they are listed in section 4 as screen additions as well as in [entities.md](entities.md) as behaviour.
3. **The five labels of the Equipment maintenance tab.** One earlier text reproduced the shipped labels, which are abbreviations; the other wrote them out in full. Both forms are given in section 5.1: the shipped labels are reproduced in quotation marks because they are what a reader sees on the screen, and the full names are used in the prose of this repository.
