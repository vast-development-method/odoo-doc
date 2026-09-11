# Inventory Operations — Configuration

Everything an installation can set, everything the installation ships with, and everything that runs on a schedule.

---

# 1. Application settings

These are the switches a person sees on the inventory settings screen. Each one is one of three kinds: a **group switch** that grants or withdraws a security group to every internal user, a **company field** stored on the company, or a **system parameter** stored as a key and a text value.

## 1.1 Group switches

| Setting label | Security group granted | Effect |
|---|---|---|
| Storage Locations | multi-location group (`stock.group_stock_multi_locations`) | Exposes Locations everywhere, exposes the internal-transfer Operation Types, exposes put-away rules and the source and destination Location fields on documents. |
| Multi-Step Routes | advanced-routing group (`stock.group_adv_location`) | Exposes Routes and Stock Rules. Turning it on forces the multi-location group on as well; turning the multi-location group off forces this one off. |
| Lots & Serial Numbers | lot group (`stock.group_production_lot`) | Exposes Lots and the lot fields on detail lines. Also granted to portal users. |
| Print Global Standards One Barcodes for Lots & Serial Numbers | structured-barcode group (`stock.group_stock_lot_print_gs1`) | Adds the structured barcode variants of the lot labels. |
| Display Lots & Serial Numbers on Delivery Slips | lot-on-slip group (`stock.group_lot_on_delivery_slip`) | Prints the lot and serial numbers on the delivery document. Also granted to portal users. Forced off when the lot group is switched off. |
| Packages | container group (`stock.group_tracking_lot`) | Exposes containers, the put-in-pack action and the container fields. |
| Consignment | owner group (`stock.group_tracking_owner`) | Exposes the owner fields on quantity records and detail lines. |
| Warnings for Stock | stock-warning group (`stock.group_warning_stock`) | Exposes the contact-level transfer warning text. |
| Signature | signature group (`stock.group_stock_sign_delivery`) | Exposes the signature field on Transfers. |
| Reception Report | reception-report group (`stock.group_reception_report`) | Exposes the Reception Report and its automatic-print switches. |

Two more groups are never exposed as a switch and are managed by the system: the **multi-warehouse group** (`stock.group_stock_multi_warehouses`), granted automatically as soon as one company has more than one active Warehouse and withdrawn when none has, and the two functional roles of section 4.

**Refusals.** Withdrawing the multi-location group while both it and the multi-warehouse group are implied for internal users is refused: "You can't deactivate the multi-location if you have more than once warehouse by company". Withdrawing the lot group while at least one product is tracked is refused: "You have product(s) in stock that have lot/serial number tracking enabled. \nSwitch off tracking on all the products before switching off this setting."

**Side effects.** Turning the multi-location group **on** activates the internal-transfer Operation Type of every Warehouse and deactivates the two simplified Location screens. Turning it **off** deactivates the internal-transfer Operation Type of the Warehouses that both receive in one step and deliver in one step, and re-activates those screens.

## 1.2 Company fields

| Setting label | Company field | Default | Meaning |
|---|---|---|---|
| Annual Inventory Month | `annual_inventory_month` | `12` (December) | The month of the yearly count for the Locations that have no counting frequency of their own. Leaving it empty switches the yearly count off. |
| Day of the month | `annual_inventory_day` | 31 | The day of that month. A value of zero or less is treated as the first day; a value beyond the month's length is treated as its last day. |
| Email Confirmation | `stock_move_email_validation` | false | Post the delivery confirmation message at validation. |
| Email Template | `stock_mail_confirmation_template_id` | the shipped delivery confirmation template | Which template to render. Restricted to templates written for Transfers. |
| Text Message Confirmation | `stock_text_confirmation` | false | Send a delivery confirmation text message at validation. |
| Text Message Type | `stock_confirmation_type` | `sms` | The channel used; the only shipped value is the text-message channel. |
| Text Message Template | `stock_sms_confirmation_template_id` | the shipped delivery text template | Which text template to render. Restricted to templates written for Transfers. |
| (hidden) | `has_received_warning_stock_sms` | false | Latch: the one-time warning about sending text messages has been shown. |
| Internal Transit Location | `internal_transit_location_id` | created with the company | The transit Location used between the company's Warehouses. |
| Replenishment Horizon | `horizon_days` | 365 | Owned by `../replenishment-and-procurement/`; listed here because it lives on the same screen. |

## 1.3 System parameters

Each is stored as a key with a text value. An absent key means the default in the table.

| Key | Default | Meaning |
|---|---|---|
| `stock.barcode_separator` | `,` (shipped as a record) | The character or characters that separate the individual encodings inside an aggregate barcode. Without it, no aggregate barcode can be produced at all. |
| `stock.agg_barcode_max_length` | 400 | The maximum length of one aggregate barcode; beyond it, a new aggregate barcode is started. |
| `stock.skip_quant_tasks` | absent (housekeeping runs) | When set, opening the quantity screens does **not** run the merge, clean-reservations and delete-empties pass. |
| `stock.merge_only_same_date` | absent | When set, two moves may only merge when their scheduled dates are equal. |
| `stock.merge_ignore_date_deadline` | absent | When set, the deadline is **removed** from the merge key, so moves with different deadlines may merge. |
| `stock.cancel_moves_origin` | absent | When set, cancelling a move that propagates cancellation also cancels its originating moves. |
| `stock.picking_no_auto_reserve` | absent | When set, completing a receipt or an internal transfer does **not** trigger the search for open moves it could satisfy. |
| `stock.no_auto_scheduler` | absent | When set, creating moves does not trigger the automatic reordering rules. |
| `stock.intercompany_auto_unpack` | absent | When set, validating a Transfer whose contact belongs to another company unpacks the destination containers of the moves that land in a transit Location. |
| `stock.propagate_uom` | absent | When set, a supply request keeps the unit of the requesting document instead of converting to the product unit. |
| `stock.report_stock_quantity_period` | absent | The horizon, in days, of the read-only daily quantity series. |
| `stock.show_expected_quantity_count` | absent | Exposes the expected-quantity column in the counting screens. |

## 1.4 Optional capabilities

The settings screen also offers to install companion capabilities. Each is a boolean that installs a package rather than a setting: expiration dates, batch/wave/cluster transfers, barcode scanner, barcode database lookup, text-message confirmation, delivery methods and each carrier connector, quality control and its worksheets, drop-shipping, and the dispatch management system.

One switch is neither a group nor a parameter: **Replenish on Order**. It reads and writes the active flag of the shipped replenish-on-order Route.

---

# 2. Numbering and sequences

## 2.1 Shipped sequences

| Purpose | Code | Prefix | Padding | Company |
|---|---|---|---|---|
| Reordering rules | `stock.orderpoint` | `OP/` | 5 | shared |
| Fallback transfer numbering | `stock.picking` | `INT/` | 5 | shared |
| Serial numbers | `stock.lot.serial` | none | 7 | shared |
| Containers | `stock.package` | `PACK` | 7 | shared |
| Scrap (one per company, created with the company) | `stock.scrap` | `SP/` | 5 | per company |
| Batch transfers | `picking.batch` | `BATCH/` | 5 | shared |
| Wave transfers | `picking.wave` | `WAVE/` | 5 | shared |

## 2.2 Operation Type sequences

Every Operation Type owns one sequence.

- **Created with a Warehouse**: name "*warehouse name* Sequence *the words below*", prefix "*short name*/*sequence prefix*/", padding 5, the Warehouse's company. The wording is: "in" for receipts, "out" for deliveries, "packing" for the pack type, "picking" for the pick type, "quality control" for the quality control type, "storage" for the storage type, "internal" for the internal transfer type, "cross dock" for the cross dock type. When a sequence of that exact name already exists in that company, the new one is renamed "*the name* (copy)(*the new sequence's identifier*)".
- **Created by hand with a Warehouse**: name "*warehouse name* Sequence *sequence prefix*", prefix "*short name*/*sequence prefix*/", padding 5, the Warehouse's company.
- **Created by hand without a Warehouse**: name "Sequence *sequence prefix*", prefix equal to the sequence prefix, padding 5, the active company.

Changing the sequence prefix of an Operation Type rewrites the name, the prefix and the padding of its sequence. Renaming a Warehouse rewrites the names of all eight sequences; changing its short name rewrites all eight prefixes.

The resulting transfer references therefore read `WH/IN/00001`, `WH/OUT/00001`, `WH/PICK/00001`, `WH/PACK/00001`, `WH/QC/00001`, `WH/STOR/00001`, `WH/INT/00001` and `WH/XD/00001` for a Warehouse whose short name is `WH`.

## 2.3 Batch and wave names

The stored name is **not** the raw sequence value. The raw value is split at its last slash; the name becomes *prefix part* + "/" + *the Operation Type's sequence prefix* + "/" + *number part*. With the shipped sequences and a delivery Operation Type whose prefix is `OUT`, the first batch is therefore named `BATCH/OUT/00001` and the first wave `WAVE/OUT/00001`. When the raw value has no slash, the name falls back to *the Operation Type's sequence prefix* + "/" + *the raw value* and a note is posted on the batch: "The sequence '*the sequence code*' is misconfigured. Its prefix should end with a '/' separator."

## 2.4 Container names

A container created without a name draws from its Package Type's own sequence when it has one (name "Package Type Sequence *the type's sequence prefix*", prefix equal to that sequence prefix, padding 7, the type's company), and from the shared container sequence otherwise. Clearing a container's name redraws it the same way.

## 2.5 Lot names

A Lot created without a name draws from the product's own lot numbering sequence when the product defines one; otherwise the name must be supplied. The shipped serial-number sequence has no prefix and a padding of 7.

## 2.6 Scrap names

A Scrap is created with the literal name "New" and draws its real reference from the company's scrap sequence at validation. When no sequence exists, the name stays "New".

---

# 3. Shipped records

## 3.1 Removal strategies

| Name | Method key |
|---|---|
| First In First Out (first in first out) | `fifo` |
| Last In First Out (last in first out) | `lifo` |
| Closest Location | `closest` |
| Least Packages | `least_packages` |

## 3.2 Shared locations

| Name | Usage | Company | Active |
|---|---|---|---|
| Vendors | vendor | shared | yes |
| Customers | customer | shared | yes |
| Inter-company transit | transit | shared | no (activated when a cross-company resupply Route is created) |

Two defaults are set so that every contact points at the shared Locations unless it overrides them: the contact-level vendor stock Location defaults to "Vendors" and the contact-level customer stock Location defaults to "Customers".

## 3.3 Shared route

| Name | Company | Active | Sequence | Selectable on |
|---|---|---|---|---|
| Replenish on Order (make to order) | shared | no | 5 | product categories |

Its sequence is deliberately lower than the resupply Routes' so that it is examined first. Each Warehouse owns exactly one Stock Rule inside it.

## 3.4 Per-company records created with the company

| Record | Values |
|---|---|
| Internal transit Location | name "Inter-warehouse transit", transit usage, the company, inactive. The company's own contact has its customer and vendor stock Locations pointed at it. |
| Inventory adjustment Location | name "Inventory adjustment", inventory-loss usage, the company. It becomes the company-level default of the product-level inventory-loss Location. |
| Production Location | name "Production", production usage, the company. It becomes the company-level default of the product-level production Location. |
| Scrap Location | name "Scrap", inventory-loss usage, the company. |
| Scrap sequence | name "*company name* Sequence scrap", code `stock.scrap`, prefix `SP/`, padding 5, starting at 1 with an increment of 1. |

## 3.5 The first warehouse

One Warehouse is created for the first company, with the short name `WH` and the company's main contact as address. Creating it creates the six Locations, the eight sequences, the eight Operation Types, the two Routes and the supply-on-order rule as described in `workflows.md`, section 1. A Warehouse is also created for any company that has none.

## 3.6 Message subtypes

The batch capability ships one discussion subtype, "Stage Changed", for Batch Transfers, not subscribed by default; the status field of a Batch Transfer is tracked under it.

---

# 4. Security groups

| Group | Purpose | Implies |
|---|---|---|
| User (`stock.group_stock_user`) | The operational role: create, confirm, reserve, validate, count, scrap. | Internal user |
| Administrator (`stock.group_stock_manager`) | The configuration role: everything the user role does, plus Warehouses, Locations, Operation Types, Routes, Rules, Storage Categories, Package Types, and deleting quantity records. | User |
| Manage Multiple Stock Locations | Exposes Locations. | — |
| Manage Multiple Warehouses | Exposes Warehouses. | — |
| Manage Lots / Serial Numbers | Exposes Lots. | — |
| Print Global Standards One Barcodes for Lot & Serial Numbers | Exposes the structured lot label formats. | — |
| Display Serial & Lot Number in Delivery Slips | Prints lots on the delivery document. | — |
| Manage Packages | Exposes containers. | — |
| Manage Push and Pull inventory flows | Exposes Routes and Rules. | — |
| Manage Different Stock Owners | Exposes owners. | — |
| A warning can be set on a partner (Stock) | Exposes the contact warning. | — |
| Require a signature on your delivery orders | Exposes the signature. | — |
| Use Reception Report | Exposes the Reception Report. | — |

The two functional roles belong to one privilege named "Inventory" in the supply-chain category; the User role has sequence 10 and the Administrator role sequence 20. The system user and the administrator user are members of the Administrator role out of the box.

---

# 5. Access rights matrix

Read / Write / Create / Delete, per entity and per group. A blank cell means the group has no access at all through that line; a user may still be covered by another line.

| Entity | Group | R | W | C | D |
|---|---|---|---|---|---|
| Warehouse | Administrator | yes | yes | yes | yes |
| Warehouse | Internal user | yes | — | — | — |
| Location | Administrator | yes | yes | yes | yes |
| Location | Internal user | yes | — | — | — |
| Location | Contact manager | yes | — | — | — |
| Transfer | User | yes | yes | yes | yes |
| Transfer | Administrator | yes | yes | yes | yes |
| Operation Type | Internal user | yes | — | — | — |
| Operation Type | User | yes | — | — | — |
| Operation Type | Administrator | yes | yes | yes | yes |
| Lot | User | yes | yes | yes | yes |
| Stock Move | User | yes | yes | yes | — |
| Stock Move | Administrator | yes | yes | yes | yes |
| Stock Move Line | Internal user | yes | yes | yes | yes |
| Stock Move Line | User | yes | yes | yes | yes |
| Stock Move Line | Administrator | yes | yes | yes | yes |
| Stock Quantity | Internal user | yes | — | — | — |
| Stock Quantity | User | yes | yes | yes | — |
| Package | Internal user | yes | — | — | — |
| Package | User | yes | yes | yes | yes |
| Package | Administrator | yes | yes | yes | yes |
| Package Type | User | yes | — | — | — |
| Package Type | Administrator | yes | yes | yes | yes |
| Package History | User | yes | yes | yes | — |
| Route | Internal user | yes | — | — | — |
| Route | Administrator | yes | yes | yes | yes |
| Stock Rule | Internal user | yes | — | — | — |
| Stock Rule | User | yes | — | — | — |
| Stock Rule | Administrator | yes | yes | yes | yes |
| Put-away Rule | Internal user | yes | — | — | — |
| Put-away Rule | Administrator | yes | yes | yes | yes |
| Removal Strategy | Internal user | yes | — | — | — |
| Storage Category | Internal user | yes | — | — | — |
| Storage Category | Administrator | yes | yes | yes | yes |
| Storage Category Capacity | Internal user | yes | — | — | — |
| Storage Category Capacity | Administrator | yes | yes | yes | yes |
| Scrap | User | yes | yes | yes | — |
| Scrap | Administrator | yes | yes | yes | yes |
| Scrap Reason Tag | User | yes | yes | yes | — |
| Scrap Reason Tag | Administrator | yes | yes | yes | yes |
| Document Reference | Internal user | yes | yes | yes | — |
| Daily quantity series | Internal user | yes | — | — | — |
| Reordering Rule | User | yes | — | — | — |
| Reordering Rule | Administrator | yes | yes | yes | yes |

Transient entities, all for the User role unless noted: Traceability Report (read, write, create), Return Transfer (read, write, create) and its line (read, write, create, delete), Backorder Confirmation and its line (read, write, create), Quantity History (read, write, create), Routes Report (read, write, create), Insufficient Quantity for Scrap (read, write, create), Replenish (read, write, create), Package Destination (read, write, create), Reordering Snooze (read, write, create, delete), Label Type Chooser (read, write, create), Lot Label Layout (read, write, create), Replenishment Option (read, write, create), Put in Pack (read, write, create). For the Administrator role only: Inventory Conflict, Inventory Warning, Request a Count, Replenishment Information and Quantity Relocation (read, write, create). The Inventory Adjustment Reference wizard is available to both roles.

Entities of other domains this domain opens up: products and product templates become readable to the User role; price lists, contacts (read, write, create), product attributes and their values become writable to the Administrator role; barcode nomenclatures and barcode rules become readable to the User role and writable to the Administrator role.

---

# 6. Record rules

All of them are multi-company rules that apply to every operation (read, write, create and delete) and to every group. `company_ids` denotes the set of companies the reader currently has enabled.

| Entity | Rule |
|---|---|
| Transfer | company must be one of the enabled companies |
| Operation Type | company must be one of the enabled companies |
| Put-away Rule | company must be one of the enabled companies |
| Lot | company must be one of the enabled companies **or empty** |
| Warehouse | company must be one of the enabled companies |
| Location | company must be one of the enabled companies **or empty** |
| Stock Move | company must be one of the enabled companies |
| Stock Move Line | company must be one of the enabled companies **or empty** |
| Stock Quantity | company must be one of the enabled companies **or empty** |
| Reordering Rule | company must be one of the enabled companies |
| Stock Rule | company must be one of the enabled companies **or empty** |
| Route | company must be one of the enabled companies **or empty** |
| Package | company must be one of the enabled companies **or empty** |
| Scrap | company must be one of the enabled companies |
| Daily quantity series | company must be one of the enabled companies |
| Storage Category | company must be one of the enabled companies **or empty** |

The batch capability adds the same kind of rule for Batch Transfers.

The distinction matters: an entity whose rule accepts an empty company can be shared between companies (the shared vendor and customer Locations, the shared replenish-on-order Route, company-less Lots), while an entity whose rule does not cannot.

---

# 7. Scheduled jobs

## 7.1 The inventory scheduler

| Property | Value |
|---|---|
| Name | "Procurement: run scheduler" |
| Interval | every 1 day |
| Runs as | the system user |
| Active | yes |

It performs three tasks, in this order, and reports progress after each:

1. **Reordering.** Select the reordering rules whose trigger is automatic and whose product is active (restricted to one company when the call names one), recompute their quantity to order and their deadline, and run them. Owned by `../replenishment-and-procurement/`; listed here because the same job performs the next two tasks.
2. **Reservation.** Select the Stock Moves whose company matches (when one is named), whose status is confirmed or partially available, whose demand is non-zero, and whose reservation date is on or before today **or** whose Operation Type reserves at confirmation. Order them by reservation date, then priority descending, then date ascending, then identifier ascending. Reserve them in chunks of one thousand, committing after each chunk.
3. **Housekeeping.** Run the merge, clean-reservations and delete-empties pass over all quantity records (`calculations.md`, section 12).

Any failure is logged and re-raised.

## 7.2 Jobs triggered by events, not by a clock

These are not scheduled but are listed here because they behave like background work:

| Trigger | Work |
|---|---|
| Opening the quantity screens | The housekeeping pass, unless the skip parameter is set. |
| Completing a receipt or an internal transfer | The search for open moves that the arrival could now satisfy, unless the no-auto-reserve parameter is set. |
| Creating moves | The automatic reordering rules for the affected products and Locations, unless the no-auto-scheduler parameter is set. |
| Confirming a Transfer | Automatic batching, when the Operation Type asks for it. |
| Validating a Transfer | Automatic batching and automatic waving of the backorders, when the Operation Type asks for it. |

---

# 8. Configuring an operation type

The choices an inventory manager makes on an Operation Type, and what each one changes.

| Choice | Consequence |
|---|---|
| Type of Operation | Decides the default source and destination Locations, whether lots may be created or only used, whether shipping labels are generated, whether the reservation method is meaningful, and which title the printed document carries. |
| Source and Destination Location | Become the defaults of every Transfer of the type and, through the Transfer, of every move. |
| Operation Type for Returns | The type used when a Transfer of this type is returned. Generated Warehouses point the receipt type and the delivery type at each other. |
| Sequence Prefix | Rewrites the numbering sequence. A duplicate prefix is warned about but accepted. |
| Reservation Method | At confirmation: reserve immediately and stamp today as the reservation date. Manually: never reserve automatically, and clear the reservation dates of the open moves. Before scheduled date: compute a reservation date per move and let the daily job reserve. Switching **to** the by-date method recomputes the reservation date of every open move of the type; switching **away from** it clears the reservation date of every move that is not assigned, done or cancelled. |
| Days / Days when starred | The number of days used by the by-date method, separately for normal and urgent moves. |
| Create Backorder | Ask, always or never. A return ignores this setting. |
| Shipping Policy | Copied onto each new Transfer; decides whether the Transfer is ready as soon as one line is reserved or only when all are. |
| Create New Lots / Use Existing Lots | Decide whether a person may type a new number, pick an existing one, both, or neither. Turning **both** off is the supported way to move tracked goods without recording numbers. |
| Move Entire Packages | Present containers rather than their contents in the scanning screens. |
| Set Package Type | Ask which container or container type to use before packing. |
| Show Detailed Operations | Show detail lines instead of moves on the Transfer screen. |
| The automatic printing switches | Queue the corresponding documents at validation. |
| Show Reception Report at Validation | Open the Reception Report after a successful validation when something remains to allocate. |
| Picking Properties | Define the schema of the free properties on Transfers of this type. |
| Automatic Batches and the grouping options | Enable automatic batching and waving; at least one grouping option must be chosen. |
| Maximum lines / Maximum transfers | Cap the size of automatically built batches and waves. Zero means no cap. |
| Auto-confirm | Confirm an automatically created batch or wave immediately. |
| Batch Properties | Define the schema of the free properties on batches of this type. |
| Dispatch Management and Docks | Expose the vehicle and dock fields on batches of this type and restrict the dock choice. |

---

# 9. Configuring a location

| Choice | Consequence |
|---|---|
| Location Type | Decides whether the Location holds counted stock (internal, transit) or is a counterpart (vendor, customer, inventory loss, production) or a grouping node (virtual). |
| Parent Location | Places the Location in the tree; the materialised path and the full name follow. |
| Company | Empty means shared. Cannot be changed afterwards. |
| Barcode | Must be unique per company. Defaults to the full name when the screen already computed one. |
| Replenishments | Marks the Location as the target of replenishment suggestions. Only one per branch. |
| Removal Strategy | Overrides the inherited strategy for goods leaving this Location and its descendants. The product category still wins over it. |
| Storage Category | Applies capacity and mixing limits to the Location. |
| Inventory Frequency | Schedules a cyclic count; zero switches it off. |

---

# 10. Configuring counting

| Choice | Where | Consequence |
|---|---|---|
| Counting frequency | on a Location | Cyclic counting for every quantity record in it. |
| Annual inventory month and day | on the company | A yearly count date for every quantity record whose Location has no cyclic frequency, and a ceiling for those that have one. |
| Scheduled date | on a quantity record | Overrides both, for that record. |
| Assigned to | on a quantity record | Restricts the record to that person's counting list; a non-manager sees only their own. |

---

# 11. Configuring put-away

1. Create Storage Categories with a maximum weight, a mixing policy and the capacities per product and per container type.
2. Attach a Storage Category to the Locations that share those limits.
3. Create Put-away Rules on the arrival Locations. Each rule names the product or the product category it applies to (or neither, meaning all), optionally the container types it applies to, the sublocation to store to, a priority, and a sublocation mode:
   - **No**: use the named sublocation itself.
   - **Last Used**: use the Location where this product was last put away under the named sublocation.
   - **Closest Location**: search the descendants of the named sublocation that carry the rule's Storage Category, preferring one that already holds the same product or the same container type.
4. Order the rules by priority; the specificity sort of `calculations.md`, section 9.1, then decides which one is tried first for a given arrival.
