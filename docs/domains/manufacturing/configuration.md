# Manufacturing — Configuration

Settings, system parameters, sequences and numbering formats, shipped default records,
security groups, the complete access-rights matrix, record rules and scheduled jobs.

---

## 1. Settings

The settings of the domain are flags that grant a technical group or install a companion
capability. A flag that grants a group takes effect immediately for every user who holds
the implying group.

| Setting (storage name) | Kind | Default | Effect |
|---|---|---|---|
| By-Products (`group_mrp_byproducts`) | Group flag, implies *Produce residual products* | off | Shows the by-product lines on recipes and on orders, and includes the by-products' product documents in a recipe's attachments. |
| Work Orders (`group_mrp_routings`) | Group flag, implies *Manage Work Order Operations* | off | Shows operations on recipes and Work Orders on orders, and reveals the Work Orders menus, the work centre menus and the effectiveness reports. See §1.1 for the side effect of turning it off. |
| Work Order Dependencies (`group_mrp_workorder_dependencies`) | Group flag, implies *Use Operation Dependencies* | off | Shows the predecessor and successor fields on operations and on Work Orders. See §1.2 for the side effect of turning it off. |
| Unlock Manufacturing Orders (`group_unlocked_by_default`) | Group flag, implies *Unlocked by default* | off | Newly created orders are unlocked. See §1.3 for the side effect. |
| Allocation Report for Manufacturing Orders (`group_mrp_reception_report`) | Group flag, implies *Use Reception Report with Manufacturing Orders* | off | Enables the allocation control on orders, the automatic allocation report and its labels. |
| Master Production Schedule (`module_mrp_mps`) | Capability | off | Installs the master production schedule. |
| Product Lifecycle Management (`module_mrp_plm`) | Capability | off | Installs engineering change orders and recipe versioning. |
| Quality (`module_quality_control`) | Capability | off | Installs quality checks on operations and transfers. |
| Quality Worksheet (`module_quality_control_worksheet`) | Capability | off | Installs worksheets for quality checks. |
| Subcontracting (`module_mrp_subcontracting`) | Capability | off | Installs subcontracting (see §8). |

### 1.1 Turning Work Orders off and on

- **Turning it off** archives **every** operation record in the database.
- **Turning it on** unarchives only the operations whose last write timestamp equals the
  most recent write timestamp among all archived operations — that is, the batch that the
  previous "off" archived.

### 1.2 Turning Work Order Dependencies off

Turning the flag off clears the operation-dependency flag on **every** recipe that has it,
with elevated rights. Orders already planned keep their planned windows; only the dependency
declarations disappear.

### 1.3 Turning Unlock Manufacturing Orders on or off

The change is applied immediately to existing data:

- turning it **on** clears the locked flag of every order that is neither `done` nor
  `cancel` and is currently locked;
- turning it **off** sets the locked flag on every order that is neither `done` nor `cancel`
  and is currently unlocked.

---

## 2. Sequences and numbering formats

### 2.1 Manufacturing Order references

A Manufacturing Order's reference is drawn from the sequence of its **operation type**, so
each warehouse and each operation type numbers independently.

| Property | Value |
|---|---|
| Sequence name | "*the warehouse name* Sequence production" |
| Prefix | *the warehouse code* + "/" + *the operation type's sequence code, defaulting to* `MO` + "/" |
| Padding | 5 |
| Company | The warehouse's company |

**Example.** A warehouse coded `WH` with the default sequence code produces references
`WH/MO/00001`, `WH/MO/00002`, and so on.

The reference must be unique per company; a duplicate is refused with **"Reference must be
unique per Company!"**

Changing the operation type of a running order draws a **new** reference from the new type's
sequence and renames the matching Stock Reference.

### 2.2 Backorder numbering

A backorder's reference is derived from its parent's, not from a sequence:

```formula
suffix( sequence ) = "-" + ( 3 − 1 − floor( log10( sequence ) ) ) zeros + sequence
```

- Sequence 0 leaves the name unchanged.
- Sequence 1 gives `-001`, 12 gives `-012`, 123 gives `-123`, 1234 gives `-1234`.
- A name that already ends in a hyphen followed by digits has that suffix **replaced**
  instead of appended, but only when the maximum backorder sequence in the production group
  is above 1 or the sequence being applied is above 1.

**Example.** `WH/MO/00021` → first split → `WH/MO/00021-001` (the original) and
`WH/MO/00021-002` (the backorder). A second split of the backorder gives `WH/MO/00021-003`.

### 2.3 Unbuild Order references

| Property | Value |
|---|---|
| Sequence name | Unbuild |
| Sequence code | `mrp.unbuild` |
| Prefix | `UB/` |
| Padding | 5 |
| Next number | 1 |
| Increment | 1 |
| Scope | One sequence per company |

**Example.** `UB/00001`.

The sequence is created when a company is created and by a maintenance routine that finds
companies without one. When the sequence yields nothing, the reference stays the literal
text "New".

### 2.4 Auxiliary operation-type sequences

| Operation type | Sequence name | Prefix | Padding |
|---|---|---|---|
| Pick Components | "*the warehouse name* Sequence picking before manufacturing" | *code* + "/" + *sequence code, default* `PC` + "/" | 5 |
| Store Finished Product | "*the warehouse name* Sequence stock after manufacturing" | *code* + "/" + *sequence code, default* `SFP` + "/" | 5 |
| Subcontracting | "*the warehouse name* Sequence subcontracting" | *code* + "/" + *sequence code, default* `SBC` (suffixed with a counter when another such sequence already exists) + "/" | 5 |
| Resupply Subcontractor | "*the warehouse name* Sequence Resupply Subcontractor" | *code* + "/" + *sequence code, default* `RES` (same counter rule) + "/" | 5 |

### 2.5 Lot and serial numbers of produced goods

A produced lot or serial number is drawn, in order of preference:

1. from the product's own lot sequence;
2. otherwise from the shared sequence coded `stock.lot.serial`;
3. and, when the resulting name already exists for that product in that company, from the
   product's next-serial derivation.

When none of these yields a name the operation fails with **"Please set the first Serial
Number or a default sequence"**.

### 2.6 Work Order barcodes

```formula
work_order_barcode = order_reference + "/" + work_order_identifier
```

---

## 3. Security groups

### 3.1 The privilege

All manufacturing groups belong to a privilege named **Manufacturing**, sequence 5, in the
supply-chain category.

### 3.2 The groups

| Group | Sequence | Implies | Granted to |
|---|---|---|---|
| Manufacturing / User | 10 | Inventory / User | — |
| Manufacturing / Administrator | 20 | Manufacturing / User | The system user and the default administrator |
| Manage Work Order Operations | — | — | Through the Work Orders setting |
| Produce residual products | — | — | Through the By-Products setting |
| Unlocked by default | — | — | Through the Unlock Manufacturing Orders setting |
| Use Reception Report with Manufacturing Orders | — | — | Through the Allocation Report setting |
| Use Operation Dependencies | — | — | Through the Work Order Dependencies setting |

The administrator group carries the description "Manage the manufacturing processes and
generate reports on those processes."

---

## 4. Access rights matrix

Read, write, create and delete are shown as R, W, C and D. A dash means not granted by that
row; a user may still reach the record through another row.

### 4.1 Entities of this domain

| Entity | Manufacturing / User | Manufacturing / Administrator | Inventory / User |
|---|---|---|---|
| Bill of Materials (`mrp.bom`) | R | R W C D | R |
| Bill of Materials Line (`mrp.bom.line`) | R | R W C D | R |
| Bill of Materials By-Product (`mrp.bom.byproduct`) | R | R W C D | — |
| Operation (`mrp.routing.workcenter`) | R | R W C D | — |
| Work Centre (`mrp.workcenter`) | R | R W C D | — |
| Work Centre Tag (`mrp.workcenter.tag`) | R | R W C D | — |
| Work Centre Capacity (`mrp.workcenter.capacity`) | R | R W C D | — |
| Productivity Loss Category (`mrp.workcenter.productivity.loss.type`) | R | — | — |
| Productivity Loss Reason (`mrp.workcenter.productivity.loss`) | R | R W C D | — |
| Productivity Log (`mrp.workcenter.productivity`) | R W C D | — | — |
| Manufacturing Order (`mrp.production`) | R W C D | R | R |
| Production Group (`mrp.production.group`) | R W C | — | — |
| Work Order (`mrp.workorder`) | R W C D | R W C D | — |
| Unbuild Order (`mrp.unbuild`) | R W C D | R W C D | — |

Note the deliberate asymmetries:

- The administrator's row on Manufacturing Order grants **read only**; the administrator
  reaches write, create and delete through the user group it implies.
- Productivity Loss Category is readable by users and is not writable by anyone through
  these rows.
- Production Group grants no delete: groups are removed only by the merge logic, acting with
  elevated rights.

### 4.2 Assistant entities

| Entity | Manufacturing / User |
|---|---|
| Change Production Quantity (`change.production.qty`) | R W C |
| Insufficient Unbuild Quantity (`stock.warn.insufficient.qty.unbuild`) | R W C |
| Backorder Confirmation (`mrp.production.backorder`) | R W C |
| Backorder Confirmation Line (`mrp.production.backorder.line`) | R W C |
| Consumption Warning (`mrp.consumption.warning`) | R W C |
| Consumption Warning Line (`mrp.consumption.warning.line`) | R W C |
| Split Multiple Productions (`mrp.production.split.multi`) | R W C |
| Split Production (`mrp.production.split`) | R W C |
| Split Production Detail (`mrp.production.split.line`) | R W C D |
| Serial Number Assignment (`mrp.production.serials`) | R W C |

None of them grants delete except the split detail line, which the assistant rebuilds.

### 4.3 Entities of other domains granted to manufacturing roles

| Entity | Manufacturing / User | Manufacturing / Administrator |
|---|---|---|
| Product Variant | R | — |
| Product Template | R | — |
| Unit of Measure | R | — |
| Contact | R | R W C |
| Stock Move | R W C D | — |
| Working Schedule | R | — |
| Working Schedule Attendance Line | R W C D | R W C D |
| Working Schedule Leave | R W C D | R |
| Resource | R | R W C D |
| Vendor Pricelist Line | — | R |
| Pricelist Rule | — | R W C D |
| Product Document | — | R W C D |

Two asymmetries again: the administrator's row on the working-schedule leave grants read
only (creation comes from the user group), and contacts may not be deleted by the
administrator.

---

## 5. Record rules

All rules are non-updatable, so a reinstall does not overwrite a customised rule.

### 5.1 Multi-company rules

| Entity | Domain | Empty company allowed |
|---|---|---|
| Manufacturing Order | The company is among the user's allowed companies | no |
| Unbuild Order | The company is among the user's allowed companies | no |
| Work Order | The company is among the user's allowed companies | no |
| Productivity Log | The company is among the user's allowed companies | no |
| Work Centre | The company is among the user's allowed companies **or empty** | yes |
| Bill of Materials | The company is among the user's allowed companies **or empty** | yes |
| Bill of Materials Line | The company is among the user's allowed companies **or empty** | yes |
| Bill of Materials By-Product | The company is among the user's allowed companies **or empty** | yes |
| Operation | The company is among the user's allowed companies **or empty** | yes |

A recipe, a work centre or an operation with no company is shared by every company: this is
how a group-wide catalogue of recipes and shop-floor resources is expressed.

### 5.2 Subcontractor portal rules

Installed with the subcontracting capability, granted to the portal group:

| Entity | Domain |
|---|---|
| Manufacturing Order | The subcontractor is the user's commercial partner. |
| Bill of Materials | The recipe is among the recipes of the user's commercial partner. |
| Bill of Materials Line | The line belongs to one of those recipes. |
| Consumption Warning | The assistant covers one of the partner's orders. |
| Consumption Warning Line | The line's order is one of the partner's orders. |
| Stock Move | The move's finished order, or the production behind its origin moves, or its component order, has the user's commercial partner as subcontractor. |
| Stock Move Line | The same, through the line's move. |
| Transfer | The transfer's partner's commercial partner is the user's commercial partner. |
| Operation Type | The type is used by one of the partner's transfers or by one of the partner's orders. |
| Location | Restricted to the locations the partner may see. |

---

## 6. Shipped default records

### 6.1 Productivity loss categories

Four records, one per category value: Availability, Performance, Quality, Productive.

### 6.2 Productivity loss reasons

| Name | Category | Is a blocking reason | Sequence |
|---|---|---|---|
| Fully Productive Time | Productive | no | 0 |
| Material Availability | Availability | yes | 1 |
| Equipment Failure | Availability | yes | 2 |
| Setup and Adjustments | Availability | yes | 3 |
| Reduced Speed | Performance | no | 5 |
| Process Defect | Quality | yes | 6 |
| Reduced Yield | Quality | yes | 7 |

The two non-blocking reasons are the ones the system assigns automatically: *Fully
Productive Time* to a timer within the expected duration and *Reduced Speed* to the part
that exceeds it.

### 6.3 The manufacture route

| Property | Value |
|---|---|
| Name | Manufacture |
| Company | empty (global) |
| Sequence | 5 |
| Selectable on a product | no |
| Selectable on a warehouse | yes |
| Warehouses | the first warehouse, initially |

The first warehouse is additionally set to resupply by manufacturing.

### 6.4 Message subtypes

Five subtypes on the Manufacturing Order, none of them subscribed to by default:

| Name and description | Sequence |
|---|---|
| MO Confirmed | 101 |
| MO Progress | 102 |
| MO To Close | 103 |
| MO Done | 104 |
| MO Cancelled | 105 |

### 6.5 Digest tip

One tip aimed at the manufacturing user group, sequence 600, titled "Tip: Use tablets in
the shop to control manufacturing".

### 6.6 Installation hooks

| Hook | When | Effect |
|---|---|---|
| Pre-installation | Before the tables are created | Prepares the schema so that the installation on an existing database does not recompute every stored field one row at a time. |
| Post-installation | After the data is loaded | Creates the manufacturing warehouse data: the manufacturing operation type, the pre-production and post-production locations, the routes and the rules, for every existing warehouse. |
| Uninstallation | On removal | Removes the manufacturing operation types, routes and rules, and the values that reference them. |
| Company creation | Whenever a company is created | Creates the production location (named "Production", usage `production`) and sets it as the company's default production-location property; creates the unbuild sequence. |
| Accounting configuration | When the manufacturing-accounting capability is installed | For every company that already has a chart of accounts, loads the chart's production-account mapping into the product-category production account. |

---

## 7. Warehouse configuration

### 7.1 The manufacturing step configuration

| Value | Label | Locations activated | Operation types activated |
|---|---|---|---|
| `mrp_one_step` | Manufacture (1 step) | none | Manufacturing |
| `pbm` | Pick components then manufacture (2 steps) | Pre-Production | Manufacturing, Pick Components |
| `pbm_sam` | Pick components, manufacture, then store products (3 steps) | Pre-Production, Post-Production | Manufacturing, Pick Components, Store Finished Product |

### 7.2 The locations created per warehouse

| Field | Name | Usage | Barcode | Active when |
|---|---|---|---|---|
| Pre-Production (`pbm_loc_id`) | Pre-Production | internal | *the warehouse code without spaces, in capitals* + `PREPRODUCTION` | two or three steps |
| Post-Production (`sam_loc_id`) | Post-Production | internal | *the warehouse code without spaces, in capitals* + `POSTPRODUCTION` | three steps |

### 7.3 The operation types created per warehouse

| Field | Name | Code | Sequence code | Barcode | Default source | Default destination | Active when |
|---|---|---|---|---|---|---|---|
| `manu_type_id` | Manufacturing | `mrp_operation` | `MO` | *code* + `MANUF` | Pre-Production in two or three steps, otherwise Stock | Post-Production in three steps, otherwise Stock | resupply by manufacturing is on and the warehouse is active |
| `pbm_type_id` | Pick Components | `internal` | `PC` | *code* + `PC` | Stock | Pre-Production | resupply is on, two or three steps, warehouse active |
| `sam_type_id` | Store Finished Product | `internal` | `SFP` | *code* + `SFP` | Post-Production | Stock | resupply is on, three steps, warehouse active |

All three are created with new-lot creation and existing-lot use allowed. Their sequence
numbers are the warehouse's next sequence plus 1, plus 2 and plus 3 respectively, and the
warehouse's sequence counter advances by 4.

An operation type of code `mrp_operation` always has new-lot creation and existing-lot use
forced on, whatever the stored values say.

An operation type of code `mrp_operation` may not have a destination location of usage
`inventory`: **"You cannot set a scrap location as the destination location for a
manufacturing type operation."**

### 7.4 The routes created per warehouse

| Route field | Routing key | Name | Selectable on | Sequence | Active when |
|---|---|---|---|---|---|
| `pbm_route_id` | the manufacturing step configuration | "Manufacture (1 step)", "Pick components and then manufacture" or "Pick components, manufacture and then store products (3 steps)" | product category, warehouse | 10 | the configuration is not one step |

The route contains, per configuration:

| Configuration | Rules |
|---|---|
| One step | none |
| Two steps | pull: Stock → Pre-Production through *Pick Components*; pull: Pre-Production → Production through *Manufacturing* |
| Three steps | pull: Stock → Pre-Production through *Pick Components*; pull: Pre-Production → Production through *Manufacturing*; **push**: Post-Production → Stock through *Store Finished Product* |

### 7.5 The global rules created per warehouse

| Rule field | Action | Procurement method | Route | Source | Destination | Operation type | Name | Active when |
|---|---|---|---|---|---|---|---|---|
| `manufacture_pull_id` | `manufacture` | make to order | Manufacture (global) | — | Stock | Manufacturing | *the warehouse-and-location rule name* with the suffix "Production" | resupply by manufacturing is on |
| `manufacture_mto_pull_id` | `pull` | make to order | Replenish on Order (global) | Stock | Production | Manufacturing | *the rule name* with the suffix "MTO" | resupply by manufacturing is on |
| `pbm_mto_pull_id` | `pull` | make to order | Replenish on Order (global) | Stock | Pre-Production | Pick Components | *the rule name* with the suffix "MTO" | resupply is on **and** the configuration is not one step |

The manufacture rule additionally propagates cancellation exactly when the configuration is
three steps. Both make-to-order rules are created with automatic execution set to manual,
so they run only when a demand explicitly selects them.

### 7.6 Warehouse resupply flags

| Field | Meaning |
|---|---|
| Manufacture to Resupply (`manufacture_to_resupply`) | Computed as "this warehouse is among the warehouses of the manufacture rule's route". Writing it adds or removes the warehouse from that route. When no manufacture rule exists yet, the route is looked up by searching for a manufacture rule of that warehouse. |

A route that contains a manufacture rule is a valid resupply route for a product only when
that product has at least one recipe of kind `normal`.

---

## 8. Subcontracting configuration

### 8.1 Per company

A location named **Subcontracting**, of internal usage, is created per company and recorded
as the company's subcontracting location. It is also set as the company-scoped default of
the partner's subcontractor-location property. A maintenance routine creates it for any
company that lacks one, including archived companies.

The company's own subcontracting location is protected: changing its company is refused with
**"You cannot alter the company's subcontracting location"**. Any subcontractor location
must be internal and linked to the right company: **"In order to manage stock accurately,
subcontracting locations must be type Internal, linked to the appropriate company."**

### 8.2 Per partner

| Field (storage name) | Meaning |
|---|---|
| Subcontractor Location (`property_stock_subcontractor`) | Company-dependent. The location used as source and destination when goods are sent to this contact during subcontracting. Defaults to the company's subcontracting location. |
| Subcontractor (`is_subcontractor`) | Not stored, searchable. True when the partner has a portal user **and** at least one subcontracting recipe names the partner or its commercial partner. |

### 8.3 Per warehouse

| Field | Meaning |
|---|---|
| Resupply Subcontractors (`subcontracting_to_resupply`) | Default true. Activates the resupply route, rules and operation type. |
| Subcontracting Operation Type (`subcontracting_type_id`) | Name "Subcontracting", code `mrp_operation`, sequence code `SBC`, component-lot creation allowed, **inactive**, source the subcontracting location, destination the production location. |
| Subcontracting Resupply Operation Type (`subcontracting_resupply_type_id`) | Name "Resupply Subcontractor", code `internal`, sequence code `RES`, existing lots allowed and new lots forbidden, label printing on, barcode *code* + `RESUP`, source Stock, destination the subcontracting location, active when resupply is on and the warehouse is active. |
| Resupply Subcontractor route (`subcontracting_route_id`) | Routing key `subcontract`; one pull rule Stock → Subcontracting through the resupply operation type; selectable on a warehouse only; sequence 10. |
| Subcontracting make-to-order rule (`subcontracting_mto_pull_id`) | Pull, make to order, on the global replenish-on-order route, Stock → Subcontracting through the resupply operation type. |
| Subcontracting rule (`subcontracting_pull_id`) | Pull, make to order, on the global *Resupply Subcontractor on Order* route, Subcontracting → Production through the resupply operation type. |

### 8.4 The global resupply route

| Property | Value |
|---|---|
| Name | Resupply Subcontractor on Order |
| Company | empty (global) |
| Sequence | 15 |
| Selectable on a product | no |
| Selectable on a warehouse | yes |

The route is deactivated automatically when no warehouse has an active rule on it, and
reactivated (and linked to the warehouse) as soon as one has. On installation, every
existing warehouse is switched to resupplying subcontractors.

Archiving or unarchiving the resupply flag on a warehouse archives or unarchives the rules
whose operation type is that warehouse's resupply type and whose source or destination is
one of the subcontracting locations.

---

## 9. Accounting configuration

| Setting | Level | Meaning |
|---|---|---|
| Production Account (`property_stock_account_production_cost_id`) | Product category, company-dependent, deletion restricted, company-checked | The valuation counterpart for both components and finished products of a Manufacturing Order. Whatever remains on it after a production is the work-centre or employee cost. |
| Expense Account (`expense_account_id`) | Work centre, company-checked | Where the labour of that work centre is relieved from when an order is closed. When empty, the finished product's expense account is used. |
| Production Work In Progress Account (`account_production_wip_account_id`) | Company | The asset account the work-in-progress entry debits. |
| Production Work In Progress Overhead Account (`account_production_wip_overhead_account_id`) | Company | The counterpart for the overhead half of the work-in-progress entry. When empty, the company-dependent fallback of the category's production account is used. |
| Analytic Distribution (`analytic_distribution`) | Work centre | Distributes the work centre's time cost over analytic accounts. |
| Extra Unit Cost (`extra_cost`) | Manufacturing Order, not copied | An additional cost per produced unit, added to the production cost. It is carried over to backorders. |

The generic chart of accounts maps the production work-in-progress account to the account
tagged *work in progress* and the overhead account to the account tagged *cost of
production*. Several country charts override both with their own accounts.

---

## 10. System parameters

| Parameter | Default | Meaning |
|---|---|---|
| `mrp.workcenter_max_planning_iterations` | 50 | The number of fourteen-day windows the slot search examines before giving up. With the default, the planning horizon is 700 days. The value is floored at 1. |

---

## 11. Scheduled jobs

The domain adds **no scheduled job of its own**. It participates in two shared jobs:

| Job | Contribution |
|---|---|
| The replenishment scheduler | Runs the reordering rules, which may create Manufacturing Orders through the manufacture rule. Kit products are excluded from the products the scheduler considers, in batches of 2000. After every reordering rule has run, the draft orders created by those rules and having component moves are confirmed, deliberately last, so that the procurements they spawn cannot conflict with a rule that has not yet run. |
| The procurement exception job | Logs the exception activities prepared when an order or a move is cancelled or reduced. |

The scheduler can also be triggered manually from the *Planning* menu; that menu entry is
visible only to a technical user.

---

## 12. Operation type settings specific to manufacturing

An operation type of code `mrp_operation` carries the following manufacturing-specific
switches, in addition to the generic ones.

| Field (storage name) | Type | Default | Meaning |
|---|---|---|---|
| Create New Lots/Serial Numbers for Components (`use_create_components_lots`) | boolean | false | Allows creating new lots or serial numbers for the components of an order of this type. When false, creating one is refused. |
| Auto Print Done Production Order (`auto_print_done_production_order`) | boolean | false | Prints the production order document when an order is marked done. |
| Auto Print Produced Product Labels (`auto_print_done_mrp_product_labels`) | boolean | false | Prints the finished-product labels when an order is marked done. |
| Product Label to Print (`mrp_product_label_to_print`) | selection `pdf` / `zpl` | `pdf` | The format of those labels: the portable-document format, or the label-printer format. |
| Auto Print Produced Lot Label (`auto_print_done_mrp_lot`) | boolean | false | Prints the lot labels of the finished move lines when an order is marked done; requires the lot group. |
| Lot/Serial Label to Print (`done_mrp_lot_label_to_print`) | selection `pdf` / `zpl` | `pdf` | The format of those labels. |
| Auto Print Allocation Report (`auto_print_mrp_reception_report`) | boolean | false | Prints the allocation report when an order with assigned downstream moves is marked done; requires the allocation group. |
| Auto Print Allocation Report Labels (`auto_print_mrp_reception_report_labels`) | boolean | false | Prints the transfer labels of the downstream moves; requires the allocation group. |
| Auto Print Generated Lot/Serial Label (`auto_print_generated_mrp_lot`) | boolean | false | Prints the label as soon as a lot or serial number is generated. |
| Generated Lot/Serial Label to Print (`generated_mrp_lot_label_to_print`) | selection `pdf` / `zpl` | `pdf` | The format of that label. |

The operation type also exposes five manufacturing counters, computed over the orders of
that type that are neither `done` nor `cancel`:

| Counter (storage name) | Domain |
|---|---|
| Orders to Process (`count_mo_todo`) | state `confirmed` |
| Orders Waiting (`count_mo_waiting`) | readiness `waiting` |
| Orders Late (`count_mo_late`) | state `confirmed` and start date before today |
| Orders In Progress (`count_mo_in_progress`) | state `progress` |
| Orders To Close (`count_mo_to_close`) | state `to_close` |

Its date-aggregated board series, for a manufacturing operation type, is the start dates of
its `confirmed` orders, labelled "Confirmed".

---

## 13. Reordering rule settings specific to manufacturing

| Field (storage name) | Meaning |
|---|---|
| Bill of Materials (`bom_id`) | Company-checked, restricted to `normal` recipes of the rule's product and company (or no company). Setting it without a route sets the route to the first manufacture rule's route; clearing the route clears the recipe. |
| Recipe placeholder (`bom_id_placeholder`) | The display name of the recipe that would be used if none is chosen. |
| Effective Bill of Materials (`effective_bom_id`) | Not stored, searchable. The chosen recipe, or the default one. |
| Show recipe column (`show_bom`) | True when the rule's effective route is one of the manufacture routes. |

Behaviour:

- The allowed replenishment units of a rule whose route contains a manufacture rule include
  the recipe's unit.
- The supply warning is raised when the route contains a manufacture rule and the product has
  no recipe at all.
- The days to order of such a rule default to the recipe's days-to-prepare value.
- The replenishment multiple of such a rule is the recipe's unit.
- The procurement values carry the chosen recipe.
- The quantity in progress of a **kit** product is derived from its components:

  ```formula
  quantity_in_progress = min over components of ( available + in_progress ) ÷ qty_per_kit
                         − min over components of available ÷ qty_per_kit
  ```

  converted, unrounded, into the rule's unit.
- The quantity in progress of a manufactured product additionally counts the quantities of
  **draft** orders created by that rule, and of **confirmed** orders created by that rule
  whose start is at or before the rule's lead horizon and whose finish is after it.
- A replenishment that created an order shows a clickable notice: "The following
  replenishment order has been generated", linking to the order.

---

## 14. Company-level records created automatically

| Record | When | Properties |
|---|---|---|
| Production location | Company creation | Name "Production", usage `production`, set as the company default of the product production-location property. |
| Unbuild sequence | Company creation, and by a maintenance routine | See §2.3. |
| Subcontracting location | Company creation, with the subcontracting capability, and by a maintenance routine | Name "Subcontracting", usage `internal`, set as the company default of the partner subcontractor-location property. |
