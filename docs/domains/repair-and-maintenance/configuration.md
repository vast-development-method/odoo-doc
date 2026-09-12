# Configuration

Every capability package, master-data prerequisite, setting, default, shipped record, numbering sequence, property definition, security group, access right, record rule, menu entry and scheduled execution of the repair and maintenance domain, with its data type, its default value and its effect.

---

## 1. Capability packages

The domain is delivered as nine installable capability packages. A package is a deployment unit: installing it makes its entities, fields, screens and shipped records exist. A package marked as automatically installed appears as soon as both of the packages it bridges are present.

| Package | Depends on | Installed automatically | What it contributes |
|---|---|---|---|
| Maintenance | the messaging capability | no | Equipment, Equipment Category, Maintenance Request, Maintenance Stage, Maintenance Team, the Maintained Item definition, the equipment manager group, the shipped stages and team, the maintenance activity kind, the five message subtypes, the record rules and the whole maintenance menu. |
| Maintenance – People | the people capability and Maintenance | yes, when both are present | The assignment of equipment to an employee or a department, the employee-based owner derivation, the employee field on a Maintenance Request, the equipment counters on the employee record and on the public employee profile, the departure option, and the implication that makes a human resources officer an equipment manager. |
| Inventory – Maintenance | the inventory capability and Maintenance | yes, when both are present | The internal location of an equipment, the equipment counter on a location, and the matching of an equipment's serial number against registered lots and serial numbers. |
| Repairs | the sales-with-inventory capability and the sales management capability | no | Repair Order, Repair Tag, the Insufficient Repair Quantity Warning, the repair operation-type code and its four extra default locations, the repair fields on Stock Move, the warehouse repair operation type and its replenish-on-order rule, the repair service-tracking value, the whole repair menu, and the printed Repair Order. |
| Repair – Manufacturing | Repairs and the manufacturing capability | yes, when both are present | Kit explosion on the repair's parts, the Manufacturing Order counter and action on a repair, the repair counter and action on a Manufacturing Order, and the bill-of-materials filter in the parts catalog. |
| Repair – Purchasing | Repairs and the purchasing-with-inventory capability | yes, when both are present | The Purchase Order counter and action on a repair, and the repair counter and action on a Purchase Order. |
| Repair – Point of Sale | the point-of-sale capability and Repairs | yes, when both are present | The flag that marks a Sales Order Line as repair-backed, its loading into the point-of-sale screen data, the exclusion of those lines when a point-of-sale order builds its delivery, and the settlement quantity of such a line. |
| Repair – Subcontracting | the manufacturing-subcontracting capability and Repairs | yes, when both are present | A pure co-installation bridge. It contributes no entity, no field and no behaviour of its own; it exists so that a deployment holding both subcontracting and repairs is recognised as a supported combination. |
| Repair – commercial document layout | Repairs and the commercial document-layout localization | yes, when both are present | The printing date stamped on a printed Repair Order under that layout. See [../fiscal-localizations/](../fiscal-localizations/). |

---

## 2. Master-data prerequisites for repairs

A Repair Order cannot be created until all of the following exist. They are created automatically when a warehouse is created with the Repairs package installed; the table states what a rebuild must guarantee.

| Prerequisite | Requirement | Consequence when missing |
|---|---|---|
| A warehouse | At least one, in the repair's company. | No repair operation type can be resolved, so the required operation type cannot be defaulted and no repair can be created. |
| A stock location for that warehouse | The warehouse's own stock location. | The component source, the product source, the product destination and the recycle destination defaults are all empty, and four required fields cannot be filled. |
| A location of production usage in the company | Exactly one is needed; the lowest-numbered one is used. | Creating a warehouse fails with "Can't find any production location." One is created automatically when the company has none at the moment the warehouse is built. |
| A location of inventory-loss usage in the company, or in the shared set | Exactly one is needed; the lowest-numbered one is used. | Creating a warehouse fails with "No location of type Inventory Loss found". |
| A repair operation type | Code `repair_operation`, belonging to the warehouse. | No repair can be created; the operation type field is required. |
| A numbering sequence for that operation type | Named after the warehouse, prefixed with the warehouse short code and the sequence code. | Repairs are created with the literal reference `New` and cannot be told apart. |
| The replenish-on-order route | An installation-wide route; the warehouse's repair pull rule is attached to it. | A part that is not in stock cannot be pulled from manufacturing or from purchasing by the repair itself; only reordering rules would replenish it. |
| The *Product Unit* decimal precision | A decimal precision setting; two decimal places in the shipped configuration. | Quantity comparisons at confirmation and at completion have no declared rounding. |

### 2.1 Shipped repair operation type

One repair operation type is shipped for the main company's first warehouse, so that the required operation-type field can be defaulted from the very first installation.

| Setting | Shipped value |
|---|---|
| Name | "Repairs" |
| Code | `repair_operation` |
| Company | the main company |
| Warehouse | the first warehouse |
| Default source location, the components | the stock location of the first warehouse |
| Default destination location, the components | the production-usage location of the main company |
| Default remove destination location | the inventory-loss location of the main company |
| Default recycle destination location | the stock location of the first warehouse |
| Sequence code | `RO` |

The first warehouse is also updated to name this operation type as its repair operation type. The record is created before the warehouse update pass runs, precisely so that the default of the required operation-type field can already resolve while the rest of the shipped data is being loaded.

### 2.2 Settings of a repair operation type

| Setting | Type | Default | Effect |
|---|---|---|---|
| Name | text | "Repairs" | The label of the type and of its card on the inventory overview. |
| Code | selection | `repair_operation` | Marks the type as a repair type. Selecting it reveals the four repair location defaults and hides the backorder policy. |
| Warehouse | reference to one Warehouse | the warehouse being created | Determines the numbering prefix and which repairs the overview card counts. |
| Sequence code | text | `RO` | The middle segment of every repair reference of this type. |
| Default source location, labelled "Component Source Location" | reference to one Location | the warehouse stock location | Seeds the component source location on new repairs: where added parts are taken from. |
| Default destination location, labelled "Component Destination Location" | reference to one Location | the company production location | Mirrors permanently into the added-parts destination location: where added parts go and where removed and recycled parts come from. |
| Product Source Location | reference to one Location, required for repair types | the warehouse stock location | Seeds the product source location: where the product to repair sits. |
| Product Destination Location | reference to one Location, required for repair types | the warehouse stock location | Seeds the product destination location: where the repaired product is placed. |
| Remove Destination Location | reference to one Location, required for repair types | the company inventory-loss location | Mirrors permanently into the removed-parts destination location: where removed parts are written off. |
| Recycle Destination Location | reference to one Location, required for repair types | the warehouse stock location | Seeds the recycled-parts destination location: where recycled parts are returned. |
| Create New Lots/Serial Numbers | true or false | true | When false, a lot cannot be created from inside a Repair Order of this type; the serial-generation action and any inline lot creation are refused with the message of rule RM-050. |
| Use Existing Lots/Serial Numbers | true or false | true | Whether existing lots may be selected on the parts of a repair of this type. |
| Reservation method | selection | as for any operation type | Decides whether a part is reserved at confirmation or a set number of days before the scheduled date. It is read when a repair is renumbered and its parts are reserved again. |
| Repair Properties | property definition | empty | Defines the ad-hoc fields carried by every Repair Order of this type. |
| Barcode | text | the warehouse short code without spaces, upper-cased, followed by `RO` | Scanning identifier of the type. |

### 2.3 Numbering sequences

| Sequence | Created by | Name | Prefix | Padding | Company |
|---|---|---|---|---|---|
| Repair reference sequence, one per warehouse | The warehouse creation pass of the Repairs package | the warehouse name followed by " Sequence repair" | the warehouse short code, a forward slash, the operation type's sequence code (falling back to `RO`), a further forward slash | five digits | the warehouse's company |

The sequence is consumed at creation of a Repair Order and again whenever the operation type of an open repair changes; a number that has been drawn is never returned to the sequence.

### 2.4 Repair tags

| Setting | Type | Default | Effect |
|---|---|---|---|
| Tag Name | text, required, unique | none | The label. Duplicates are refused with "Tag name already exists!". |
| Color Index | whole number | a pseudo-random whole number from one to eleven | The colour of the tag chip. |

The Repair Orders Tags configuration entry is a developer-only menu entry; tags are normally created beforehand, because creating and editing a tag from the selector on the Repair Order form is disabled, so a tag must exist before it can be applied.

### 2.5 Product configuration that affects repairs

| Product setting | Values | Effect on this domain |
|---|---|---|
| Product kind | goods or service | Only goods may be the product to repair, and only goods appear in the parts catalog. |
| Track Inventory, that is, storable | true or false | When false, the availability check at confirmation is skipped entirely and the repair confirms directly. |
| Tracking | `none`, `lot`, `serial` | When not `none`, the lot field is shown on the repair and is required at completion. When `serial`, changing the product on a saved repair forces the quantity back to one. |
| Service tracking | gains the value `repair` | A sold service carrying it raises one Repair Order per confirmed order line. |
| Lot numbering sequence | a sequence on the product | Used first when generating a serial number for the product to repair. |
| Reference unit and permitted units | units of measure | Restrict the unit choices on the repair, and are protected by the unit-change guard of rule RM-090. |
| Routes: replenish on order, manufacture, buy | routes | Decide whether a missing part pulls a Manufacturing Order, a Purchase Order or an internal transfer. |
| Costing method and valuation method | standard, average cost or first in first out; manual or automated | Decide the cost layer a repair part consumes and whether the repair movement produces a journal entry. See [accounting-effects.md](accounting-effects.md). |

---

## 3. Master-data prerequisites for maintenance

| Prerequisite | Requirement | Consequence when missing |
|---|---|---|
| At least one Maintenance Stage | The pipeline columns. | A new request is created with no stage, which leaves it out of every stage-based filter and counter. |
| At least one stage carrying the closing flag | Closing a request requires it. | Requests can never close, close dates are never stamped, and the effectiveness measurements stay at zero for ever. |
| At least one Maintenance Team | The team field on a request is required. | No Maintenance Request can be created at all. |
| At least one Equipment Category | Not strictly required; a request may name no equipment and an equipment may have no category. | Equipment cannot be grouped, and category-level property definitions and responsible users are unavailable. |
| An alias domain for incoming mail | Required only for the mail intake. | A team cannot publish an alias address and requests cannot be raised by electronic mail. |
| The maintenance activity kind | Used by the one-activity-per-request rule. | A deployment that deletes it loses the automatic scheduling of technician work. |

### 3.1 Shipped maintenance stages

| Stage | Sequence | Folded in the pipeline | Closing flag |
|---|---|---|---|
| New Request | 1 | no | no |
| In Progress | 2 | no | no |
| Repaired | 3 | yes | yes |
| Scrap | 4 | yes | yes |

**Effects of the shipped set.** "New Request" is the stage with the lowest sequence, so it is the default stage of every new request, the target of the reopen operation, and the stage in which a recurrent successor is created. Both "Repaired" and "Scrap" carry the closing flag, which means a request dragged into "Scrap" also stamps a close date, also marks its activity done and also creates the successor of a recurrent preventive request. The business distinction between the two is purely a matter of reporting: "Repaired" records a success and "Scrap" records that the asset could not be saved.

### 3.2 Shipped maintenance team

| Team | Company | Members | Alias |
|---|---|---|---|
| Internal Maintenance | none, therefore visible from every company | none | no local part until one is configured |

### 3.3 Shipped activity kind

| Setting | Value |
|---|---|
| Name | "Maintenance Request" |
| Summary | "Maintenance Request" |
| Icon | a wrench |
| Applies to | Maintenance Request |

This is the activity kind used by the one-activity-per-request rule (RM-227).

### 3.4 Shipped message subtypes

| Subtype | Applies to | Subscribed by default | Hidden | Parent | Relation field |
|---|---|---|---|---|---|
| Request Created | Maintenance Request | no | yes | none | none |
| Status Changed | Maintenance Request | yes | no | none | none |
| Equipment Assigned | Equipment | no | no | none | none |
| Maintenance Request Created | Equipment Category | yes | no | Request Created | the request's category |
| Equipment Assigned | Equipment Category | yes | no | Equipment Assigned | the equipment's category |

**Effect of the two category-level subtypes.** A user who follows an Equipment Category is notified when a Maintenance Request is created for any equipment of that category, and when any equipment of that category is assigned, without following each equipment individually.

No message template is shipped by this domain: every notification it causes is a tracked-field message, a subtype message or an activity, never a rendered template.

### 3.5 Property definitions

Each Equipment Category carries a property definition. Every Equipment of that category then exposes exactly those ad-hoc fields. Changing the definition on the category changes what every equipment of the category shows. Moving an equipment to another category re-bases its properties on the new definition.

The same mechanism exists on the repair side: each repair operation type carries a repair property definition, and every Repair Order of that type exposes those fields.

### 3.6 Maintenance settings

| Setting | Type | Default | Effect |
|---|---|---|---|
| Custom Maintenance Worksheets (`module_maintenance_worksheet`) | true or false | false | Turning it on installs the capability that adds worksheet templates to Maintenance Requests, letting a technician fill a structured form while performing the work. Visible only to holders of the equipment manager group. The worksheet capability itself is outside this domain. |

This is the only configuration setting the domain contributes. It declares no system parameter of its own.

---

## 4. Access groups

### 4.1 Groups introduced by this domain

| Group | Privilege heading | Implies | Comment shown to the administrator |
|---|---|---|---|
| Equipment Manager | Maintenance, under the supply chain heading, at sequence 10 | Internal User | "The user will be able to manage equipment." |

The equipment manager group is granted to the system account and to the administrator account at installation.

### 4.2 Groups reused by this domain

| Group | Owning domain | Why this domain needs it |
|---|---|---|
| Internal User | [identity and access](../identity-and-access/) | The baseline for every maintenance permission: reading equipment, categories, stages and teams, and full access to Maintenance Requests subject to the record rules. |
| Inventory User | [inventory operations](../inventory-operations/) | The role that operates repairs. Every repair permission is attached to it. |
| Inventory Administrator | [inventory operations](../inventory-operations/) | Required for the repair reporting and configuration menus. |
| Lot and Serial Number Tracking | [inventory operations](../inventory-operations/) | Reveals the lot field on the repair form and on the printed document, and enables the equipment serial matching. |
| Unit of Measure | [units of measure and packaging](../units-of-measure-and-packaging/) | Reveals the unit columns on the repair form and on the printed document. |
| Multiple Companies | [identity and access](../identity-and-access/) | Reveals the company fields. |
| Product Variants | [products and catalog](../products-and-catalog/) | Reveals the Product Variants entry of the repair configuration menu. |
| Manufacturing User | [manufacturing](../manufacturing/) | Reveals the Manufacturing Order counter on a repair. |
| Purchase User | [purchasing](../purchasing/) | Reveals the Purchase Order counter on a repair. |
| Human Resources User | [human resources core](../human-resources-core/) | Reveals the equipment collection on an employee. With the people bridge installed it also implies the equipment manager group. |
| Settings Administration | [identity and access](../identity-and-access/) | Required for the maintenance Settings entry. |
| Developer visibility | [identity and access](../identity-and-access/) | Required for the Maintenance Stages entry, the Activity Types entry, the Repair Orders Tags entry, the repair quantity block on the repair form, the assigned date and the scrap date on the equipment form, the carbon-copy field on a request, and the request-date column of the request list. |

### 4.3 Model access matrix

| Entity | Group | Read | Write | Create | Delete |
|---|---|---|---|---|---|
| Repair Order | Inventory User | yes | yes | yes | yes |
| Repair Tag | Inventory User | yes | yes | yes | yes |
| Insufficient Repair Quantity Warning | Inventory User | yes | yes | yes | no |
| Equipment | Internal User | yes | no | no | no |
| Equipment | Equipment Manager | yes | yes | yes | yes |
| Equipment Category | Internal User | yes | no | no | no |
| Equipment Category | Equipment Manager | yes | yes | yes | yes |
| Maintenance Stage | Internal User | yes | no | no | no |
| Maintenance Stage | Equipment Manager | yes | yes | yes | yes |
| Maintenance Team | Internal User | yes | no | no | no |
| Maintenance Team | Equipment Manager | yes | yes | yes | yes |
| Maintenance Request | Internal User | yes | yes | yes | yes |
| Activity Kind | Equipment Manager | yes | yes | yes | yes |

### 4.4 Record rules

| Rule name | Entity | Applies to | Condition |
|---|---|---|---|
| Users are allowed to access their own maintenance requests | Maintenance Request | Internal User | the created-by user is the acting user, **or** the acting user's contact is among the followers, **or** the technician is the acting user |
| Users are allowed to access equipment they follow | Equipment | Internal User | the acting user's contact is among the followers |
| Administrator of maintenance requests | Maintenance Request | Equipment Manager | always true |
| Equipment administrator | Equipment | Equipment Manager | always true |
| Maintenance Request multi-company rule | Maintenance Request | everyone | the company is among the acting user's allowed companies, or is empty |
| Maintenance Equipment multi-company rule | Equipment | everyone | the company is among the acting user's allowed companies, or is empty |
| Maintenance Team multi-company rule | Maintenance Team | everyone | the company is among the acting user's allowed companies, or is empty |
| Maintenance Equipment Category multi-company rule | Equipment Category | everyone | the company is among the acting user's allowed companies, or is empty |
| Repair order multi-company | Repair Order | everyone | the company is among the acting user's allowed companies |

Maintenance Stage carries no company and no record rule; stages are global.

**Two consequences worth stating explicitly.**

1. The repair rule has no "or empty company" branch, because a Repair Order always has a company. The four maintenance rules do, because equipment, teams, categories and requests may be shared across companies by leaving the company empty.
2. An ordinary internal user may create a Maintenance Request for any equipment they follow, without holding the equipment manager group. Adding a person as a follower of a machine is therefore the intended lightweight way of letting them report faults on it.

### 4.5 Group implications contributed by this domain

| Implying group | Implied group | Contributed by |
|---|---|---|
| Equipment Manager | Internal User | the Maintenance package |
| Human Resources User | Equipment Manager | the Maintenance – People bridge |

---

## 5. Menus

### 5.1 Repairs

| Entry | Parent | Requires | Opens |
|---|---|---|---|
| Repairs | top level, at sequence 165 | Inventory User | nothing of its own |
| Orders | Repairs | Inventory User | The Repair Orders screen. |
| Reporting | Repairs | Inventory Administrator | nothing of its own |
| Repairs, under Reporting | Reporting | Inventory Administrator | The Repair Orders Analysis screen, grouped by product and by creation date. |
| Configuration | Repairs | Inventory Administrator | nothing of its own |
| Repair Orders Tags | Configuration | Developer visibility | The Repair Tags list. |
| Products | Configuration | Inventory Administrator | The product template screen. |
| Product Variants | Configuration | Product Variants | The product variant screen. |

### 5.2 Maintenance

| Entry | Parent | Requires | Opens |
|---|---|---|---|
| Maintenance | top level, at sequence 160 | no group of its own | nothing of its own |
| Dashboard | Maintenance | Equipment Manager or Internal User | The Maintenance Teams dashboard. |
| Maintenance, the section | Maintenance | Equipment Manager or Internal User | nothing of its own |
| Maintenance Requests | the Maintenance section | Equipment Manager or Internal User | The pipeline, with the Active filter applied and the technician defaulted to the acting user. |
| Maintenance Calendar | the Maintenance section | Equipment Manager or Internal User | The calendar, with the Active and To Do filters applied. |
| Equipment | Maintenance | Equipment Manager or Internal User | The Equipment screen, grouped by category. |
| Reporting, the first heading | Maintenance | Equipment Manager or Internal User | nothing of its own |
| Overall Equipment Effectiveness | the first Reporting heading | Equipment Manager or Internal User | Nothing on its own. It is a heading that the manufacturing capability fills. |
| Losses Analysis | the first Reporting heading | Equipment Manager or Internal User | Nothing on its own. It is a heading that the manufacturing capability fills. |
| Reporting, the second heading | Maintenance | no group of its own | nothing of its own |
| Maintenance Requests Analysis | the second Reporting heading | no group of its own | The analysis screen over Maintenance Requests, with the Active filter applied. |
| Configuration | Maintenance | Equipment Manager | nothing of its own |
| Settings | Configuration | Settings Administration | The Maintenance section of the settings screen. |
| Maintenance Teams | Configuration | Equipment Manager | The Teams screen. |
| Equipment Categories | Configuration | inherited from Configuration | The Equipment Categories screen. |
| Maintenance Stages | Configuration | Developer visibility | The Stages screen. |
| Activity Types | Configuration | Developer visibility | The activity kinds applicable to Maintenance Requests. |

---

## 6. Defaults summary

Every default applied when a record of this domain is created, in one place.

### 6.1 Repair Order

| Field | Default |
|---|---|
| `name`, the reference | the literal text `New`, replaced at creation by the operation type's sequence |
| `company_id` | the acting company |
| `state` | `draft`, unless the creator is the Sales-Order rule, which sets `confirmed` |
| `priority` | `0`, Normal |
| `user_id`, the responsible | the acting user |
| `schedule_date` | the current moment |
| `product_qty` | `1.0` |
| `under_warranty` | false |
| `is_parts_available`, `is_parts_late` | false |
| `picking_type_id` | the repair type of the responsible user's default warehouse in the company; otherwise the first repair type of a warehouse of that company; and in either case overridden by a value supplied through the screen context |
| the six location fields | the corresponding defaults of the operation type |
| `picking_id`, the return transfer | the value supplied by the screen context, when a repair is created from a transfer |
| `lot_id` | the value supplied by the screen context, when a repair is created from a lot |
| `reference_ids` | one new Stock Reference named after the repair |
| `l10n_din5008_printing_date` | the current date |

### 6.2 Repair part line

| Field | Default |
|---|---|
| Demanded quantity | `1.0` on a line added from the form |
| Company | the repair's company |
| Date | the repair's scheduled date |
| Part kind | `add` on a line added from the form or from the catalog |
| Source and destination locations | derived from the part kind |

### 6.3 Repair Tag

| Field | Default |
|---|---|
| `color` | a pseudo-random whole number from one to eleven |

### 6.4 Equipment

| Field | Default |
|---|---|
| `active` | true |
| `effective_date` | today in the acting user's time zone |
| `company_id` | the acting company |
| `equipment_assign_to`, with the people bridge | `employee` |
| `assign_date`, with the people bridge | today in the acting user's time zone, restamped on each assignment change |
| `owner_user_id`, with the people bridge | the acting user, then overwritten by the assignment derivation |
| `technician_user_id` | empty, then replaced by the chosen category's responsible user |
| `color` | `0` |
| `cost` | `0.0` |

### 6.5 Equipment Category

| Field | Default |
|---|---|
| `company_id` | the acting company |
| `technician_user_id` | the acting user |
| `color` | `0` |
| `fold` | true, because the category starts empty |

### 6.6 Maintenance Request

| Field | Default |
|---|---|
| `request_date` | today in the acting user's time zone |
| `owner_user_id` | the acting user |
| `employee_id`, with the people bridge | the employee record of the acting user |
| `company_id` | the acting company |
| `stage_id` | the stage with the lowest sequence |
| `maintenance_team_id` | the first team of the acting company, falling back to the first team of any company |
| `kanban_state` | `normal` |
| `archive` | false |
| `maintenance_type` | `corrective` |
| `instruction_type` | `text` |
| `repeat_interval` | `1` |
| `repeat_unit` | `week` |
| `repeat_type` | `forever` |
| `color` | `0` |
| `priority` | none |
| `user_id`, the technician | derived from the equipment when one is chosen |
| `duration` | `0.0` |

### 6.7 Maintenance Stage

| Field | Default |
|---|---|
| `sequence` | `20` |
| `fold` | false |
| `done` | false |

### 6.8 Maintenance Team

| Field | Default |
|---|---|
| `active` | true |
| `company_id` | the acting company |
| `color` | `0` |
| alias target entity | Maintenance Request |
| alias default values | the team field forced to this team |

---

## 7. Scheduled execution

This domain declares no scheduled job of its own. It relies on scheduled execution owned elsewhere in two places:

1. **Replenishment.** Confirming a Repair Order, and creating a part on a running Repair Order, trigger the replenishment scheduler for the part movements at once, synchronously. Whatever those movements then need — a Manufacturing Order, a Purchase Order or an internal transfer — is produced by the [replenishment and procurement](../replenishment-and-procurement/) domain, which also runs its own periodic pass over reordering rules.
2. **Activity reminders.** The activity deadlines created by Maintenance Requests are surfaced by the periodic activity digest owned by [messaging and activities](../messaging-and-activities/).

Neither the readiness fields of a repair nor the effectiveness measurements of an equipment are refreshed on a schedule. They are derived on read, from the current state of the movements and of the requests. A rebuild must therefore compute them at read time, or invalidate their caches on every change of their declared inputs, which are listed per field in [entities.md](entities.md).

---

## Reconciliation notes

1. **Number of capability packages.** One earlier text listed eight packages and described a ninth only in passing; the other listed nine. Nine are listed here, the ninth being the commercial document-layout bridge that contributes the printing date of the printed Repair Order.
2. **The identifier of the worksheet setting.** One earlier text named the setting after its effect rather than reproducing its stored name. The stored name is `module_maintenance_worksheet`, reproduced in section 3.6.
3. **The shipped repair operation type.** Only one earlier text recorded that a repair operation type is shipped for the main company's first warehouse in addition to the one that every warehouse creation produces. Both facts are true and both are in section 2.1 and section 2.2.
4. **Sequences.** One earlier text listed the numbering sequence only as a side effect of warehouse creation. It is a configuration record in its own right and is tabulated in section 2.3, together with the statement that a drawn number is never returned.
