# Entities

This file specifies every entity the fleet domain owns: its purpose, its lifecycle, its complete field table, its relations, its uniqueness rules, its defaults, its computed fields with the rules that produce them and the fields those rules depend on, its ordering, its display-name rule, its duplication behaviour, its archival behaviour and its multi-company behaviour. It then specifies every field and every changed behaviour that this domain adds to entities owned by other domains.

## Conventions used in every field table

- **Identifier** reproduces the storage name of the field exactly, in code font. For a multi-valued reference held in an association table, the identifier is the name of the relation used in the transport contract, and the association table and its two columns are named in the row.
- **Full name** gives the field's business name in words. Every reproduced identifier carries its full name the first time it appears in this document.
- **Type** gives the abstract data type. *reference to one* is a single-valued reference to another entity. *collection of* is a multi-valued reference. *selection* is a closed list of stored values each carrying a display label. *decimal* is a fixed-precision number, *whole number* an integer, *monetary* a decimal carrying a currency, *text* a single-line string, *long text* a multi-line string, *rich text* a formatted string, *image* an uploaded picture, *date* a calendar day, *date and time* an instant, and *properties* a set of user-defined values whose definition lives on another record.
- **Target** names the referenced entity for a reference field.
- **Required** says whether a value must be present when the record is stored.
- **Default** gives the value used when none is supplied.
- **Computed rule and dependencies** names the rule that produces the value and the fields whose change re-runs it, and says whether the result is stored in the table and whether a user may overwrite it.
- **Meaning** states what the field means and any behaviour attached to it.

Unless a row says otherwise: a plain stored field is carried into a duplicate of the record; a computed field, a mirrored field and a multi-valued reference are not. Each departure from that default is stated in the row that departs from it.

Every persistent entity carries the platform's common fields — the surrogate identifier, the creation timestamp and creating user, and the last-update timestamp and updating user. Those are specified once in [../../overview/entity-and-field-system.md](../../overview/entity-and-field-system.md) and are not repeated here. An entity that carries a discussion thread additionally carries the follower list, the message list, the unread and action-needed counters, the delivery-error counters and the attachment counter; an entity that carries scheduled activities additionally carries the activity list, the aggregated activity state, the next activity date, summary, kind and responsible user, and the activity exception decoration. Those two sets are specified in [../messaging-and-activities/](../messaging-and-activities/); each entity below states which of them it carries.

Throughout, *the current company* means the company the acting user is operating in, *the acting user* means the user performing the operation, and *today* means the calendar day in the acting user's time zone unless the text says otherwise.

---

# Part one: the vehicle register

## 1. Vehicle

**Vehicle** (`fleet.vehicle`, table `fleet_vehicle`). Generated reference page: [../../references/entities/fleet.vehicle.md](../../references/entities/fleet.vehicle.md).

### 1.1 Purpose

A Vehicle is one physical machine that the company owns, leases, rents or lends. It answers six questions:

1. **What is it?** A Vehicle Model, which carries the manufacturer; a licence plate; a chassis number; a category; free tags; and seventeen physical attributes copied from the model.
2. **Who is responsible for it?** A fleet manager, who is a user; a driver, who is a Contact and may also be an Employee; and optionally a future driver who is queued to replace the driver.
3. **Where is it in its life?** A configurable Vehicle Status forming the pipeline board, plus a registration date, an order date and a cancellation date.
4. **What is it worth?** A catalogue value including value added tax, a purchase value, a residual value and a taxable horsepower figure.
5. **What has happened to it?** Collections of driver assignment entries, odometer readings, services and contracts, each with a counter shown on the record.
6. **Does it need attention?** Two renewal flags derived from its contracts, the state of its latest contract, and a service-activity indicator derived from the activities scheduled on its services.

### 1.2 Inherited behaviour

The Vehicle participates in three shared behaviours specified elsewhere:

- **Discussion thread** — messages, followers, attachment counters and tracked-field logging. See [../messaging-and-activities/](../messaging-and-activities/). Changes to the driver or to the future driver are logged under the dedicated subtype described in [configuration.md](configuration.md) rather than the default tracking subtype.
- **Scheduled activities** — planned activities with deadlines, kinds, summaries, responsible users and an exception decoration. See [../messaging-and-activities/](../messaging-and-activities/).
- **Avatar and image set** — the Vehicle carries the platform's five image sizes (`image_1920`, `image_1024`, `image_512`, `image_256`, `image_128`, the full names being image at one thousand nine hundred and twenty pixels, at one thousand and twenty-four, at five hundred and twelve, at two hundred and fifty-six and at one hundred and twenty-eight pixels) and the five matching avatar sizes (`avatar_1920`, `avatar_1024`, `avatar_512`, `avatar_256`, `avatar_128`). The Vehicle redefines `image_128` as a mirror of its model's picture, so the picture shown for a vehicle is the manufacturer logo published through the model. When no picture is available the avatar falls back to a generated monogram built from the first letter of the display name, and when the display name is empty to a neutral grey placeholder. See [../../overview/entity-and-field-system.md](../../overview/entity-and-field-system.md).

### 1.3 Ordering, display name and identity

- **Default ordering**: by `license_plate` (licence plate) ascending, then by `acquisition_date` (registration date) ascending. Vehicles without a plate therefore group together at one end of the list.
- **Display name**: the value of `name`, which is itself computed; the rule is given in [calculations.md](calculations.md).
- **Name search**: a text typed into a vehicle reference field is matched against `name` and against `driver_id.name`, so a vehicle can be found by typing the name of the person who drives it.
- **Uniqueness**: no database uniqueness constraint is declared. Neither the licence plate nor the chassis number is enforced unique. **Industry-standard default**: a rebuild that wants to prevent duplicate registrations should add a partial uniqueness rule on the chassis number over non-empty values only, because the field is optional; this specification records that the observed system enforces nothing.
- **Indexed columns**: `driver_employee_id` is indexed with an index that skips empty values. No other index beyond the primary key and the foreign keys the platform creates by default is declared.
- **Archival**: the Vehicle carries `active`. Archiving a vehicle also archives all of its contracts and all of its services; see section 1.8 and [workflows.md](workflows.md).
- **Deletion**: deleting a vehicle is permitted for the fleet officer and the fleet administrator. Deleting the Vehicle Status a vehicle points at empties `state_id` rather than blocking the deletion.

### 1.4 Complete field table

#### 1.4.1 Identity and description

| Identifier | Full name | Type | Target | Required | Default | Computed rule and dependencies | Meaning |
|---|---|---|---|---|---|---|---|
| `name` | Name | text | — | no | — | `_compute_vehicle_name`, depending on `model_id.brand_id.name`, `model_id.name` and `license_plate`; stored; not editable | The display name of the vehicle, assembled from manufacturer, model and plate. The assembly rule is in [calculations.md](calculations.md). |
| `description` | Vehicle Description | rich text | — | no | — | — | Free notes about the vehicle, shown on a dedicated page of the form. |
| `active` | Active | boolean | — | no | true | — | Whether the vehicle is in the working set. Changes are recorded in the discussion thread. Clearing it cascades to contracts and services, see section 1.8. |
| `license_plate` | Licence Plate | text | — | no | — | — | The registration plate of the vehicle. Changes are recorded in the discussion thread. Used in the display name and in the default ordering. |
| `vin_sn` | Chassis Number | text | — | no | — | — | The unique number stamped on the vehicle, which is the vehicle identification number for a car or the serial number for another machine. Changes are recorded in the discussion thread. Not carried into a duplicate, because a duplicate is a different physical machine. |
| `model_id` | Model | reference to one | `fleet.vehicle.model` | yes | — | — | The Vehicle Model. Changes are recorded in the discussion thread. Setting or changing it re-runs the seventeen copy rules of section 1.6. |
| `brand_id` | Manufacturer | reference to one | `fleet.vehicle.model.brand` | no | — | mirrored from `model_id.brand_id`; stored; editable | The manufacturer. Stored so that it can be grouped and searched, and editable so that a vehicle may be recorded under a different manufacturer than its model declares. |
| `category_id` | Category | reference to one | `fleet.vehicle.model.category` | no | — | `_compute_category`, depending on `model_id`; stored; editable | The load or usage class of the vehicle. |
| `tag_ids` | Tags | collection of | `fleet.vehicle.tag` | no | — | — | Free labels. Held in the association table `fleet_vehicle_vehicle_tag_rel`, whose column `vehicle_tag_id` holds the vehicle and whose column `tag_id` holds the tag. Not carried into a duplicate. |
| `vehicle_type` | Vehicle Kind | selection | — | no | — | mirrored from `model_id.vehicle_type`; not stored | Either `car` labelled "Car" or `bike` labelled "Bike". Decides which groups of fields the form shows and which of the two plan-to-change flags applies. |
| `image_128` | Image at one hundred and twenty-eight pixels | image | — | no | — | mirrored from `model_id.image_128`; read-only | The picture shown for the vehicle, which is the manufacturer logo carried through the model. |
| `vehicle_properties` | Properties | properties | — | no | — | — | User-defined values whose definition is held in `model_id.vehicle_properties_definition`. Carried into a duplicate. |
| `description` shown above and `location` shown below are the only free text fields on the record. | | | | | | | |

#### 1.4.2 People

| Identifier | Full name | Type | Target | Required | Default | Computed rule and dependencies | Meaning |
|---|---|---|---|---|---|---|---|
| `manager_id` | Fleet Manager | reference to one | `res.users` | no | — | — | The user answerable for the vehicle. The choice is restricted to users that are not shared or portal users, that belong to the vehicle's company, and that are members of the fleet officer group. This user receives the end-date reminder activity when the driver changes, and is the responsible user proposed by the fleet-manager activity plan kind. |
| `driver_id` | Driver | reference to one | `res.partner` | no | — | — | The Contact currently driving the vehicle. Changes are recorded in the discussion thread under the driver subtype. Not carried into a duplicate. Setting it opens a driver assignment entry; see [workflows.md](workflows.md). |
| `future_driver_id` | Future Driver | reference to one | `res.partner` | no | — | — | The Contact queued to become the driver. Changes are recorded in the discussion thread under the driver subtype. Not carried into a duplicate. Validated against the vehicle's company. Setting it flags the vehicles of the same kind that this person currently drives as planned for change. |
| `next_assignation_date` | Assignment Date | date | — | no | — | — | The day from which the vehicle becomes available to the future driver. An empty value means available immediately. |
| `driver_employee_id` | Driver Employee | reference to one | `hr.employee` | no | — | `_compute_driver_employee_id`, depending on `driver_id`; stored; editable | Added by the people bridge. The Employee whose work contact is the driver and whose company is the vehicle's company. Changes are recorded in the discussion thread. Restricted to employees with no company or with the vehicle's company. Indexed, skipping empty values. |
| `driver_employee_name` | Driver Employee Name | text | — | no | — | mirrored from `driver_employee_id.name`; not stored | Added by the people bridge. The name of the driver Employee, offered for display and search. |
| `future_driver_employee_id` | Future Driver Employee | reference to one | `hr.employee` | no | — | `_compute_future_driver_employee_id`, depending on `future_driver_id`; stored; editable | Added by the people bridge. The Employee behind the future driver, resolved by the same rule. Changes are recorded in the discussion thread. |
| `mobility_card` | Mobility Card | text | — | no | — | `_compute_mobility_card`, depending on `driver_id`; stored; not editable | Added by the people bridge. The mobility card reference published by the Employee behind the driver. |

#### 1.4.3 Dates and status

| Identifier | Full name | Type | Target | Required | Default | Computed rule and dependencies | Meaning |
|---|---|---|---|---|---|---|---|
| `state_id` | Status | reference to one | `fleet.vehicle.state` | no | the shipped status named "New Request", when that record exists | — | The pipeline column of the vehicle. Changes are recorded in the discussion thread. Grouping by this field expands to every defined status, including those with no vehicle, so that the board shows empty columns. Deleting the referenced status empties this field rather than blocking the deletion. |
| `order_date` | Order Date | date | — | no | — | — | The day the vehicle was ordered from its supplier. |
| `acquisition_date` | Registration Date | date | — | no | today | — | The day the vehicle was registered. Changes are recorded in the discussion thread. Used in the default ordering and as the starting month of both analysis result sets. |
| `write_off_date` | Cancellation Date | date | — | no | — | — | The day the vehicle's plate was cancelled or withdrawn. Changes are recorded in the discussion thread. |
| `contract_date_start` | First Contract Date | date | — | no | today | — | The day the first coverage contract of the vehicle began. Changes are recorded in the discussion thread. Informational; it does not drive any contract state. |
| `location` | Location | text | — | no | — | — | Where the vehicle is kept, for example a garage name. Free text; it is not a reference to a warehouse location. |

#### 1.4.4 Physical attributes

Each attribute in this group except `odometer_unit`, `power_unit`, `frame_type`, `frame_size`, `vehicle_range` and `co2_emission_unit` is computed from the model by a rule that copies the model's value when that value is not empty, and each remains editable afterwards. The shared copy rule is specified in section 1.6 and in [calculations.md](calculations.md).

| Identifier | Full name | Type | Target | Required | Default | Computed rule and dependencies | Meaning |
|---|---|---|---|---|---|---|---|
| `seats` | Seating Capacity | whole number | — | no | — | `_compute_seats`, depending on `model_id`; stored; editable | Number of seats. |
| `doors` | Number of Doors | whole number | — | no | — | `_compute_doors`, depending on `model_id`; stored; editable | Number of doors, counting a boot or hatch door where the model counts it. |
| `color` | Colour | text | — | no | — | `_compute_color`, depending on `model_id`; stored; editable | The colour of the vehicle, as free text. |
| `model_year` | Model Year | selection | — | no | — | `_compute_model_year`, depending on `model_id`; stored; editable | A year. The list of stored values runs from `1970` to the current calendar year inclusive, each stored value being the four-digit year as text and each label the same year. The list therefore grows by one entry every first of January. |
| `trailer_hook` | Trailer Hitch | boolean | — | no | false | `_compute_trailer_hook`, depending on `model_id`; stored; editable | Whether a towing device is fitted. |
| `transmission` | Transmission | selection | — | no | — | `_compute_transmission`, depending on `model_id`; stored; editable | `manual` labelled "Manual"; `automatic` labelled "Automatic". |
| `fuel_type` | Fuel Kind | selection | — | no | — | `_compute_fuel_type`, depending on `model_id`; stored; editable | Nine stored values shared with the model; the full list is in section 2.4. |
| `power` | Power | decimal | — | no | — | `_compute_power`, depending on `model_id`; stored; editable | Engine power in kilowatts. Shown only when the power unit is `power`. |
| `horsepower` | Horsepower | decimal | — | no | — | `_compute_horsepower`, depending on `model_id`; stored; editable | Engine power in horsepower. Shown only when the power unit is `horsepower`. |
| `horsepower_tax` | Horsepower Taxation | decimal | — | no | — | `_compute_horsepower_tax`, depending on `model_id`; stored; editable | The taxable horsepower figure some jurisdictions levy a road tax on. Displayed as a monetary amount on the tax page of the form. |
| `power_unit` | Power Unit | selection | — | yes | `power` | — | `power` labelled "kW", meaning kilowatts; `horsepower` labelled "Horsepower". Decides which of the two power fields the form shows. Listed in the model-to-vehicle copy map but never copied, because the field is not computed — see the compatibility finding in [business-rules.md](business-rules.md). |
| `co2` | Carbon Dioxide Emissions | decimal | — | no | — | `_compute_co2`, depending on `model_id`; stored; editable | The emissions figure of the vehicle. Its label is reproduced as "CO₂ Emissions". No aggregation is offered for this field in grouped views, because summing emissions figures across vehicles is meaningless. Changes are recorded in the discussion thread. |
| `co2_emission_unit` | Emission Unit | selection | — | yes | `g/km` | `_compute_co2_emission_unit`, depending on `range_unit`; stored; not editable | `g/km` labelled "g/km", meaning grammes per kilometre; `g/mi` labelled "g/mi", meaning grammes per mile. Follows the range unit exactly. |
| `co2_standard` | Emission Standard | text | — | no | — | `_compute_co2_standard`, depending on `model_id`; stored; editable | The regulatory test procedure under which the emissions figure was measured, as free text. |
| `vehicle_range` | Range | whole number | — | no | — | — | The distance the vehicle can travel on one full tank or charge. Listed in the model-to-vehicle copy map but never copied, because the field is not computed — see the compatibility finding in [business-rules.md](business-rules.md). |
| `range_unit` | Range Unit | selection | — | yes | `km` | `_compute_range_unit`, depending on `model_id`; stored; editable | `km` labelled "km", meaning kilometres; `mi` labelled "mi", meaning miles. Drives the emission unit. |
| `frame_type` | Bicycle Frame Kind | selection | — | no | — | — | `diamant` labelled "Diamant"; `trapez` labelled "Trapez"; `wave` labelled "Wave". Shown only when the vehicle kind is `bike`. |
| `frame_size` | Frame Size | decimal | — | no | — | — | The frame size in centimetres. Shown only when the vehicle kind is `bike`. |
| `electric_assistance` | Electric Assistance | boolean | — | no | — | `_compute_electric_assistance`, depending on `model_id`; stored; editable | Whether a bicycle has a motor assisting the rider. Shown only when the vehicle kind is `bike`. |

#### 1.4.5 Distance

| Identifier | Full name | Type | Target | Required | Default | Computed rule and dependencies | Meaning |
|---|---|---|---|---|---|---|---|
| `odometer` | Last Odometer | decimal | — | no | — | `_get_odometer`, not stored; writing it runs `_set_odometer` | Reading it yields the greatest recorded value among the vehicle's odometer readings, or zero when there is none. Writing a non-zero value creates a new odometer reading dated today for the vehicle and its current driver. Writing a value lower than the current one is refused; see [business-rules.md](business-rules.md), rule FLT-002. |
| `odometer_unit` | Odometer Unit | selection | — | yes | `kilometers` | — | `kilometers` labelled "km"; `miles` labelled "mi". Mirrored read-only onto every odometer reading and every service of the vehicle. |
| `odometer_count` | Odometer Reading Count | whole number | — | no | — | `_compute_count_all`, not stored | The number of odometer readings recorded for the vehicle, whatever their date. |

#### 1.4.6 Value

| Identifier | Full name | Type | Target | Required | Default | Computed rule and dependencies | Meaning |
|---|---|---|---|---|---|---|---|
| `car_value` | Catalog Value Including Value Added Tax | decimal | — | no | — | — | The list price of the vehicle with tax included. Changes are recorded in the discussion thread. Displayed as a monetary amount using the company currency. |
| `net_car_value` | Purchase Value | decimal | — | no | — | — | The amount actually paid for the vehicle. Displayed as a monetary amount. |
| `residual_value` | Residual Value | decimal | — | no | — | — | The value expected at the end of the holding period. Displayed as a monetary amount. |
| `currency_id` | Currency | reference to one | `res.currency` | no | — | mirrored from `company_id.currency_id`; not stored | The currency in which the three value fields and every contract and service amount of the vehicle are expressed. |

#### 1.4.7 Related record counters and derived flags

| Identifier | Full name | Type | Target | Required | Default | Computed rule and dependencies | Meaning |
|---|---|---|---|---|---|---|---|
| `log_drivers` | Assignment Logs | collection of | `fleet.vehicle.assignation.log` | no | — | inverse field `vehicle_id` | Every driver assignment entry of the vehicle. |
| `log_services` | Services | collection of | `fleet.vehicle.log.services` | no | — | inverse field `vehicle_id` | Every service of the vehicle. |
| `log_contracts` | Contracts | collection of | `fleet.vehicle.log.contract` | no | — | inverse field `vehicle_id` | Every contract of the vehicle. |
| `history_count` | Driver History Count | whole number | — | no | — | `_compute_count_all`, not stored | The number of driver assignment entries. |
| `service_count` | Service Count | whole number | — | no | — | `_compute_count_all`, not stored | The number of services whose archive flag equals the vehicle's own archive flag. An active vehicle therefore counts only its active services, and an archived vehicle only its archived ones. |
| `contract_count` | Contract Count | whole number | — | no | — | `_compute_count_all`, not stored | The number of contracts that are not cancelled and whose archive flag equals the vehicle's own archive flag. |
| `contract_renewal_due_soon` | Has Contracts To Renew | boolean | — | no | — | `_compute_contract_reminder`, depending on `log_contracts`; not stored; searchable through `_search_contract_renewal_due_soon` | True when the latest contract expiry is in the future but closer than the configured alert delay. The computation rule and the search rule differ; both are specified in [calculations.md](calculations.md). |
| `contract_renewal_overdue` | Has Contracts Overdue | boolean | — | no | — | `_compute_contract_reminder`, depending on `log_contracts`; not stored; searchable through `_search_get_overdue_contract_reminder` | True when the latest contract expiry is already past. |
| `contract_state` | Last Contract State | selection | — | no | — | `_compute_contract_reminder`, depending on `log_contracts`; not stored | The state of the contract with the latest expiry. `futur` labelled "Incoming"; `open` labelled "In Progress"; `expired` labelled "Expired"; `closed` labelled "Closed". When the vehicle has no contract with an expiry date, the field holds the empty text rather than an empty reference. |
| `service_activity` | Service Activity | selection | — | no | — | `_compute_service_activity`, depending on `log_services`; not stored | `none` labelled "None"; `overdue` labelled "Overdue"; `today` labelled "Today". Derived from the activity states of the vehicle's services; the rule is in [calculations.md](calculations.md). Chooses which of three service counter buttons the form shows and in which colour. |
| `plan_to_change_car` | Planned To Change Car | boolean | — | no | false | — | Marks a car whose driver has been queued to receive a different car, so that the vehicle may be offered to somebody else. Changes are recorded in the discussion thread. |
| `plan_to_change_bike` | Planned To Change Bicycle | boolean | — | no | false | — | The same marker for a bicycle. Changes are recorded in the discussion thread. |
| `bill_count` | Bills Count | whole number | — | no | — | `_compute_move_ids`, not stored | Added by the accounting bridge. The number of purchase journal entries that carry at least one item naming this vehicle and are not cancelled. Zero for a user who cannot read accounting. |
| `account_move_ids` | Journal Entries | collection of | `account.move` | no | — | `_compute_move_ids`, not stored | Added by the accounting bridge. Those same journal entries. Its name in the transport contract is `account_moves`. |

#### 1.4.8 Company

| Identifier | Full name | Type | Target | Required | Default | Computed rule and dependencies | Meaning |
|---|---|---|---|---|---|---|---|
| `company_id` | Company | reference to one | `res.company` | no | the current company | — | The company that owns the vehicle. May be left empty, which makes the vehicle visible to every company; the record rule of [configuration.md](configuration.md) admits both the user's companies and the empty value. |
| `country_id` | Country | reference to one | `res.country` | no | — | mirrored from `company_id.country_id`; not stored | The country of the owning company, offered so that localizations can show country-specific groups on the form. |
| `country_code` | Country Code | text | — | no | — | mirrored from `country_id.code`; not stored | The two-letter code of that country, offered for the same purpose. |

### 1.5 Relations

| Relation | Cardinality | Other end | Deletion behaviour |
|---|---|---|---|
| Vehicle to Vehicle Model | many to one | `fleet.vehicle.model` | The model may not be deleted while a vehicle references it. |
| Vehicle to Vehicle Manufacturer | many to one, mirrored and stored | `fleet.vehicle.model.brand` | The manufacturer may not be deleted while a vehicle references it. |
| Vehicle to Vehicle Category | many to one | `fleet.vehicle.model.category` | The category may not be deleted while a vehicle references it. |
| Vehicle to Vehicle Status | many to one | `fleet.vehicle.state` | Deleting the status empties the field on every vehicle that pointed at it. |
| Vehicle to Vehicle Tag | many to many through `fleet_vehicle_vehicle_tag_rel` | `fleet.vehicle.tag` | Deleting a tag removes the association rows. |
| Vehicle to Driver Assignment Log | one to many through `vehicle_id` | `fleet.vehicle.assignation.log` | The vehicle reference on the entry is required, so deleting a vehicle deletes its entries. |
| Vehicle to Odometer Reading | one to many through `vehicle_id` | `fleet.vehicle.odometer` | The vehicle reference on the reading is required. |
| Vehicle to Vehicle Service | one to many through `vehicle_id` | `fleet.vehicle.log.services` | The vehicle reference on the service is required. |
| Vehicle to Vehicle Contract | one to many through `vehicle_id` | `fleet.vehicle.log.contract` | The vehicle reference on the contract is required. |
| Vehicle to Journal Item | one to many through `vehicle_id` on the item | `account.move.line` | Added by the accounting bridge. Deleting a vehicle is not blocked by journal items; the reference is a plain optional field. **Compatibility finding**, recorded in [business-rules.md](business-rules.md) as FLT-C13. |
| Vehicle to Batch Transfer | one to many through `vehicle_id` on the batch | `stock.picking.batch` | Added by the transport dispatch bridge. |

### 1.6 The seventeen attributes copied from the model

Seventeen attributes are declared as copied from the Vehicle Model. Each copy is a separate rule whose only dependency is `model_id`, and each writes one field. All seventeen share one procedure, specified as a numbered algorithm with a worked example in [calculations.md](calculations.md). The map of model field to vehicle field is:

| Model field | Vehicle field | Reached by rule |
|---|---|---|
| `transmission` | `transmission` | `_compute_transmission` |
| `model_year` | `model_year` | `_compute_model_year` |
| `electric_assistance` | `electric_assistance` | `_compute_electric_assistance` |
| `color` | `color` | `_compute_color` |
| `seats` | `seats` | `_compute_seats` |
| `doors` | `doors` | `_compute_doors` |
| `trailer_hook` | `trailer_hook` | `_compute_trailer_hook` |
| `default_co2` | `co2` | `_compute_co2` |
| `co2_standard` | `co2_standard` | `_compute_co2_standard` |
| `default_fuel_type` | `fuel_type` | `_compute_fuel_type` |
| `power` | `power` | `_compute_power` |
| `horsepower` | `horsepower` | `_compute_horsepower` |
| `horsepower_tax` | `horsepower_tax` | `_compute_horsepower_tax` |
| `category_id` | `category_id` | `_compute_category` |
| `range_unit` | `range_unit` | `_compute_range_unit` |
| `vehicle_range` | `vehicle_range` | declared in the map, never reached |
| `power_unit` | `power_unit` | declared in the map, never reached |

The last two rows are **compatibility findings**: the two vehicle fields are plain stored fields with no computed rule attached, so the model's range and the model's power unit are never propagated to a vehicle even though the map names them. A corrected behaviour would declare both fields computed from the model, stored and editable, exactly like the fifteen that work. They are recorded in [business-rules.md](business-rules.md) as FLT-C04.

### 1.7 Defaults applied at creation

1. `company_id` is set to the current company.
2. `acquisition_date` and `contract_date_start` are set to today.
3. `state_id` is set to the shipped status named "New Request" when that record is present; when it is absent, the field is left empty.
4. `active` is set to true, `trailer_hook` to false, `odometer_unit` to `kilometers`, `power_unit` to `power`, `range_unit` to `km` and `co2_emission_unit` to `g/km`.
5. The seventeen copy rules run as soon as `model_id` is known, overwriting the defaults of the fields they cover when the model carries a value.
6. When the values supplied at creation name a driver, one driver assignment entry is created; see [workflows.md](workflows.md), procedure W-02.

### 1.8 Archival behaviour

Clearing `active` on a vehicle also clears `active` on every contract of that vehicle and on every service of that vehicle, whatever their state. Restoring `active` on the vehicle does **not** restore them; each has to be restored on its own. The desktop client warns before archiving with the exact text "Every service and contract of this vehicle will be considered as archived. Are you sure that you want to archive this record?" and performs the archive only when the reader confirms.

Because `service_count` and `contract_count` count only records whose archive flag matches the vehicle's, an archived vehicle shows the counts of its archived contracts and services and an active vehicle shows the counts of its active ones. The counters therefore stay meaningful on both sides of the archive line.

### 1.9 Multi-company behaviour

- `company_id` may be empty, which means the vehicle is shared across companies.
- The record rule admits a vehicle whose company is one of the acting user's allowed companies or is empty.
- The fleet manager must belong to the vehicle's company.
- The future driver is validated against the vehicle's company.
- The driver Employee and the future driver Employee are resolved per company: an Employee qualifies only when its work contact is the driver Contact **and** its company is the vehicle's company. A person employed by two companies therefore resolves to a different Employee on a vehicle of each company.
- Contracts carry their own company, defaulting to the current company, and the contract's vehicle is validated against it.
- Services carry their own company, defaulting to the current company, and are filtered by a record rule on that company.
- Odometer readings carry no company of their own and are filtered by a record rule on the company of their vehicle.

### 1.10 Extension points contributed by other packages

| Package | What it adds to the Vehicle |
|---|---|
| Accounting and Fleet bridge | `bill_count` and `account_move_ids`, and the operation that opens the vendor bills of the vehicle |
| Fleet History bridge to the people register | `driver_employee_id`, `driver_employee_name`, `future_driver_employee_id`, `mobility_card`, the two-way contact and employee synchronisation at creation and at write time, the unsubscription of the outgoing driver, and the operation that opens the driver Employee |
| Transport Dispatch bridge | No field on the Vehicle itself; the Vehicle becomes selectable on a Batch Transfer and lends its category and its driver to that batch |
| Analytic naming extension point | The rule `_get_analytic_name` yields the licence plate of the vehicle, or the text "No plate" when the plate is empty. Nothing inside this repository calls it; it exists so that a localization can turn a vehicle into an analytic dimension. See [accounting-effects.md](accounting-effects.md). |

---

## 2. Vehicle Model

**Vehicle Model** (`fleet.vehicle.model`, table `fleet_vehicle_model`). Generated reference page: [../../references/entities/fleet.vehicle.model.md](../../references/entities/fleet.vehicle.model.md).

### 2.1 Purpose

A Vehicle Model is the template that describes a commercially named machine: a manufacturer plus a model name, together with the physical attributes every unit of that model shares. It exists so that those attributes are typed once and copied onto each vehicle, and so that a property definition can be attached to a family of vehicles.

### 2.2 Inherited behaviour

The Vehicle Model carries a discussion thread, scheduled activities and the avatar and image set, exactly as the Vehicle does. Its `image_128` is redefined as a mirror of its manufacturer's logo, which is how a manufacturer logo reaches a vehicle.

### 2.3 Ordering, display name and identity

- **Default ordering**: by `name` ascending.
- **Display name**: the manufacturer name, a solidus, then the model name, when the model has a manufacturer; otherwise the model name alone. The rule is in [calculations.md](calculations.md).
- **Name search**: a typed text matches either the model name or the manufacturer name. Negative operators are not supported by that rule, so an exclusion search falls back to the platform default of matching the display name only.
- **Uniqueness**: none is declared. Two models of the same manufacturer may carry the same name.
- **Archival**: the model carries `active`. Archiving a model does not archive its vehicles.

### 2.4 Complete field table

| Identifier | Full name | Type | Target | Required | Default | Computed rule and dependencies | Meaning |
|---|---|---|---|---|---|---|---|
| `name` | Model Name | text | — | yes | — | — | The commercial name of the model. Changes are recorded in the discussion thread. |
| `brand_id` | Manufacturer | reference to one | `fleet.vehicle.model.brand` | yes | — | — | The manufacturer. Changes are recorded in the discussion thread. Indexed, skipping empty values. |
| `category_id` | Category | reference to one | `fleet.vehicle.model.category` | no | — | — | The load or usage class, copied onto each vehicle of the model. Changes are recorded in the discussion thread. |
| `vendors` | Vendors | collection of | `res.partner` | no | — | — | The suppliers from whom this model can be bought or leased. Held in the association table `fleet_vehicle_model_vendors`, whose column `model_id` holds the model and whose column `partner_id` holds the Contact. |
| `active` | Active | boolean | — | no | true | — | Whether the model is offered when creating a vehicle. |
| `vehicle_type` | Vehicle Kind | selection | — | yes | `car` | — | `car` labelled "Car"; `bike` labelled "Bike". Changes are recorded in the discussion thread. Mirrored onto every vehicle of the model. |
| `vehicle_count` | Vehicle Count | whole number | — | no | — | `_compute_vehicle_count`, not stored; searchable through `_search_vehicle_count` | The number of vehicles of this model. |
| `image_128` | Image at one hundred and twenty-eight pixels | image | — | no | — | mirrored from `brand_id.image_128`; read-only | The manufacturer logo. |
| `vehicle_properties_definition` | Vehicle Properties Definition | properties definition | — | no | — | — | The definition of the user-defined values that every vehicle of this model carries in its `vehicle_properties`. |
| `model_year` | Model Year | selection | — | no | — | — | The same year list as on the vehicle, from `1970` to the current calendar year. Changes are recorded in the discussion thread. |
| `seats` | Seating Capacity | whole number | — | no | — | — | Number of seats. Changes are recorded in the discussion thread. |
| `doors` | Number of Doors | whole number | — | no | — | — | Number of doors, counting a boot or hatch door where applicable. Changes are recorded in the discussion thread. |
| `color` | Colour | text | — | no | — | — | Default colour. Changes are recorded in the discussion thread. |
| `trailer_hook` | Trailer Hitch | boolean | — | no | false | — | Whether a towing device is fitted as standard. Changes are recorded in the discussion thread. |
| `transmission` | Transmission | selection | — | no | — | — | `manual` labelled "Manual"; `automatic` labelled "Automatic". Changes are recorded in the discussion thread. |
| `drive_type` | Drive Kind | selection | — | no | — | — | `fwd` labelled "Front-Wheel Drive (FWD)"; `awd` labelled "All-Wheel Drive (AWD)"; `rwd` labelled "Rear-Wheel Drive (RWD)"; `4wd` labelled "Four-Wheel Drive (4WD)". Describes which wheels are driven. Not copied onto the vehicle. |
| `default_fuel_type` | Fuel Kind | selection | — | no | `electric` | — | Nine stored values: `diesel` labelled "Diesel"; `gasoline` labelled "Gasoline"; `full_hybrid` labelled "Full Hybrid"; `plug_in_hybrid_diesel` labelled "Plug-in Hybrid Diesel"; `plug_in_hybrid_gasoline` labelled "Plug-in Hybrid Gasoline"; `cng` labelled "CNG", meaning compressed natural gas; `lpg` labelled "LPG", meaning liquefied petroleum gas; `hydrogen` labelled "Hydrogen"; `electric` labelled "Electric". Required on the form when the vehicle kind is `car`. Changes are recorded in the discussion thread. |
| `power` | Power | decimal | — | no | — | — | Engine power in kilowatts. Changes are recorded in the discussion thread. |
| `horsepower` | Horsepower | decimal | — | no | — | — | Engine power in horsepower. Changes are recorded in the discussion thread. |
| `horsepower_tax` | Horsepower Taxation | decimal | — | no | — | — | The taxable horsepower figure. Changes are recorded in the discussion thread. |
| `power_unit` | Power Unit | selection | — | yes | `power` | — | `power` labelled "kW"; `horsepower` labelled "Horsepower (hp)". Decides which of the two power fields the model form shows. |
| `default_co2` | Carbon Dioxide Emissions | decimal | — | no | — | — | The emissions figure of the model, whose label is reproduced as "CO₂ Emissions". Changes are recorded in the discussion thread. |
| `co2_emission_unit` | Emission Unit | selection | — | yes | — | `_compute_co2_emission_unit`, depending on `range_unit`; not stored | `g/km` labelled "g/km"; `g/mi` labelled "g/mi". Follows the range unit. Because it is required but not stored, it is re-derived on every read and never written. |
| `co2_standard` | Emission Standard | text | — | no | — | — | The regulatory test procedure under which the emissions figure was measured. Changes are recorded in the discussion thread. |
| `electric_assistance` | Electric Assistance | boolean | — | no | false | — | Whether the bicycle has a motor assisting the rider. Changes are recorded in the discussion thread. |
| `vehicle_range` | Range | whole number | — | no | — | — | The distance the model can travel on one full tank or charge. |
| `range_unit` | Range Unit | selection | — | yes | `km` | — | `km` labelled "km"; `mi` labelled "mi". |

### 2.5 Relations

| Relation | Cardinality | Other end | Deletion behaviour |
|---|---|---|---|
| Vehicle Model to Vehicle Manufacturer | many to one | `fleet.vehicle.model.brand` | The manufacturer may not be deleted while a model references it. |
| Vehicle Model to Vehicle Category | many to one | `fleet.vehicle.model.category` | The category may not be deleted while a model references it. |
| Vehicle Model to Contact | many to many through `fleet_vehicle_model_vendors` | `res.partner` | Deleting a Contact removes the association rows. |
| Vehicle Model to Vehicle | one to many through `model_id` on the vehicle | `fleet.vehicle` | Deletion of the model is blocked while vehicles reference it. |
| Vehicle Model to Vehicle Service | one to many through the service's mirrored `model_id` | `fleet.vehicle.log.services` | The mirror is stored on the service for grouping; it is not a constraint. |

### 2.6 Operations on the model

`action_model_vehicle` opens the vehicles of this model. When the model already has at least one vehicle, the operation opens the board, list and form views under the title "Vehicles", filtered to this model and defaulting new records to it. When the model has none, it opens a single form under the title "Vehicle", with the model already filled in.

---

## 3. Vehicle Manufacturer

**Vehicle Manufacturer** (`fleet.vehicle.model.brand`, table `fleet_vehicle_model_brand`). Generated reference page: [../../references/entities/fleet.vehicle.model.brand.md](../../references/entities/fleet.vehicle.model.brand.md).

### 3.1 Purpose

A Vehicle Manufacturer is the maker of a Vehicle Model. It carries a name and a logo, and publishes the number of models it has. The logo is the picture that reaches a vehicle through its model.

### 3.2 Ordering, display name and identity

- **Default ordering**: by `name` ascending.
- **Display name**: the `name` field.
- **Uniqueness**: none is declared. Two manufacturers may carry the same name.
- **Archival**: the manufacturer carries `active`. The board view offers archive and restore from the card menu.
- **Discussion thread**: none. The manufacturer carries neither a thread nor activities.

### 3.3 Complete field table

| Identifier | Full name | Type | Target | Required | Default | Computed rule and dependencies | Meaning |
|---|---|---|---|---|---|---|---|
| `name` | Name | text | — | yes | — | — | The manufacturer's name. |
| `active` | Active | boolean | — | no | true | — | Whether the manufacturer is offered when creating a model. |
| `image_128` | Logo | image | — | no | — | — | The manufacturer logo, stored at a maximum of one hundred and twenty-eight pixels in each direction. Mirrored onto every model of the manufacturer and, through the model, onto every vehicle. |
| `model_count` | Model Count | whole number | — | no | — | `_compute_model_count`, depending on `model_ids.active`; stored | The number of active models of this manufacturer. |
| `model_ids` | Models | collection of | `fleet.vehicle.model` | no | — | inverse field `brand_id` | Every model of the manufacturer, archived ones included. |

The counting rule filters models on the archive flag using the literal text `true` rather than the boolean value. That is a **compatibility finding**, recorded in [business-rules.md](business-rules.md) as FLT-C05; the observed effect is the intended one, because the platform coerces the text to the boolean, but a rebuild should compare against the boolean.

### 3.4 Operations on the manufacturer

- `action_brand_model` opens the list and form of models, filtered to this manufacturer and defaulting new models to it, under the title "Models".
- `action_open_brand_form` opens the form of this manufacturer. It exists because the card view opens the model list when the card body is clicked, so a separate path is needed to reach the manufacturer itself; the logo on the card is that path.

---

## 4. Vehicle Category

**Vehicle Category** (`fleet.vehicle.model.category`, table `fleet_vehicle_model_category`). Generated reference page: [../../references/entities/fleet.vehicle.model.category.md](../../references/entities/fleet.vehicle.model.category.md).

### 4.1 Purpose

A Vehicle Category groups models and vehicles into load or usage classes, such as a transport truck or a pickup van. With the transport dispatch bridge installed, a category also publishes the maximum weight and the maximum volume a vehicle of that class can carry, which the dispatch screens use to report how full a planned load is.

### 4.2 Ordering, display name and identity

- **Default ordering**: by `sequence` ascending, then by the surrogate identifier ascending. The list view exposes the sequence as a drag handle.
- **Display name**: the `name` field. With the transport dispatch bridge installed the display name is extended with the capacities; the exact assembly is in [calculations.md](calculations.md).
- **Uniqueness**: the database constraint `_name_uniq` declares `name` unique. Its message is "Category name must be unique".
- **Archival**: the category carries no archive flag.
- **Discussion thread**: none.

### 4.3 Complete field table

| Identifier | Full name | Type | Target | Required | Default | Computed rule and dependencies | Meaning |
|---|---|---|---|---|---|---|---|
| `name` | Name | text | — | yes | — | — | The category name. Unique across all categories. |
| `sequence` | Sequence | whole number | — | no | — | — | The ordering weight. Shown on the form only to users holding the technical-features flag. |
| `weight_capacity` | Maximum Weight | decimal | — | no | — | — | Added by the transport dispatch bridge. The greatest weight a vehicle of this category can carry, expressed in the platform's weight unit. |
| `weight_capacity_uom_name` | Weight Unit Label | text | — | no | — | `_compute_weight_capacity_uom_name`, not stored | Added by the transport dispatch bridge. The label of the platform's weight unit, taken from the system parameter that names it. See [../units-of-measure-and-packaging/](../units-of-measure-and-packaging/). |
| `volume_capacity` | Maximum Volume | decimal | — | no | — | — | Added by the transport dispatch bridge. The greatest volume a vehicle of this category can carry, expressed in the platform's volume unit. |
| `volume_capacity_uom_name` | Volume Unit Label | text | — | no | — | `_compute_volume_capacity_uom_name`, not stored | Added by the transport dispatch bridge. The label of the platform's volume unit. |

### 4.4 Relations

| Relation | Cardinality | Other end | Deletion behaviour |
|---|---|---|---|
| Vehicle Category to Vehicle Model | one to many through `category_id` on the model | `fleet.vehicle.model` | Deletion is blocked while a model references the category. |
| Vehicle Category to Vehicle | one to many through `category_id` on the vehicle | `fleet.vehicle` | Deletion is blocked while a vehicle references the category. |
| Vehicle Category to Batch Transfer | one to many through `vehicle_category_id` on the batch | `stock.picking.batch` | Added by the transport dispatch bridge. |

---

## 5. Vehicle Status

**Vehicle Status** (`fleet.vehicle.state`, table `fleet_vehicle_state`). Generated reference page: [../../references/entities/fleet.vehicle.state.md](../../references/entities/fleet.vehicle.state.md).

### 5.1 Purpose

A Vehicle Status is one column of the vehicle pipeline board. Statuses are data, not code: an administrator creates, renames, reorders and folds them freely, and the vehicle's status field is an ordinary reference to one of them. That is why the vehicle's own progress is not a fixed state machine; see [state-machines.md](state-machines.md) for the consequences.

### 5.2 Ordering, display name and identity

- **Default ordering**: by `sequence` ascending. Records with equal sequence fall back to the platform's tie-break on the surrogate identifier.
- **Display name**: the `name` field, which is translatable.
- **Uniqueness**: the database constraint `_fleet_state_name_unique` declares `name` unique. Its message is "State name already exists".
- **Archival**: none. A status is deleted, not archived, and deleting it empties the field on the vehicles that pointed at it.

### 5.3 Complete field table

| Identifier | Full name | Type | Target | Required | Default | Computed rule and dependencies | Meaning |
|---|---|---|---|---|---|---|---|
| `name` | Name | text | — | yes | — | — | The status label. Translatable, so the same status reads differently in each installed language while remaining one record. See [../../runtime/translation.md](../../runtime/translation.md). |
| `sequence` | Sequence | whole number | — | no | — | — | The position of the column on the board and the position of the row in the list, where it is exposed as a drag handle. |
| `fold` | Folded In Board | boolean | — | no | false | — | Whether the board shows this column collapsed to a narrow strip. A folded column is used for statuses that hold many finished vehicles. |

---

## 6. Vehicle Tag

**Vehicle Tag** (`fleet.vehicle.tag`, table `fleet_vehicle_tag`). Generated reference page: [../../references/entities/fleet.vehicle.tag.md](../../references/entities/fleet.vehicle.tag.md).

### 6.1 Purpose

A Vehicle Tag is a free label attached to any number of vehicles, used to slice the fleet along axes the status pipeline does not cover, for example a project, a site or a fuel card scheme.

### 6.2 Ordering, display name and identity

- **Default ordering**: none is declared, so the platform default of the surrogate identifier ascending applies, which means creation order.
- **Display name**: the `name` field, which is translatable.
- **Uniqueness**: the database constraint `_name_uniq` declares `name` unique. Its message is "Tag name already exists!", including the exclamation mark.
- **Archival**: none.

### 6.3 Complete field table

| Identifier | Full name | Type | Target | Required | Default | Computed rule and dependencies | Meaning |
|---|---|---|---|---|---|---|---|
| `name` | Tag Name | text | — | yes | — | — | The label. Translatable and unique. |
| `color` | Colour Index | whole number | — | no | — | — | The index into the client's tag palette. Chosen through a colour picker in the list view and used by every screen that shows vehicle tags. |

---

## 7. Fleet Service Type

**Fleet Service Type** (`fleet.service.type`, table `fleet_service_type`). Generated reference page: [../../references/entities/fleet.service.type.md](../../references/entities/fleet.service.type.md).

### 7.1 Purpose

A Fleet Service Type names a kind of work or a kind of coverage. One list serves two purposes, separated by a category: a type of the `contract` category may be chosen as the kind of a Vehicle Contract, and a type of the `service` category is the kind of a Vehicle Service. A contract also lists the service types it includes, which is how a lease declares that it covers tyre changes and roadside assistance.

### 7.2 Ordering, display name and identity

- **Default ordering**: by `name` ascending.
- **Display name**: the `name` field, which is translatable.
- **Uniqueness**: none is declared.
- **Archival**: none.

### 7.3 Complete field table

| Identifier | Full name | Type | Target | Required | Default | Computed rule and dependencies | Meaning |
|---|---|---|---|---|---|---|---|
| `name` | Name | text | — | yes | — | — | The name of the kind of work or coverage. Translatable. |
| `category` | Category | selection | — | yes | — | — | `contract` labelled "Contract"; `service` labelled "Service". Decides where the type may be used: the kind field of a contract is restricted to `contract` types, while the kind field of a service is not restricted at all. The second half of that statement is a **compatibility finding** recorded as FLT-C06 in [business-rules.md](business-rules.md): a rebuild should restrict the service kind to `service` types for symmetry. |

### 7.4 Relations

| Relation | Cardinality | Other end | Deletion behaviour |
|---|---|---|---|
| Fleet Service Type to Vehicle Contract | one to many through `cost_subtype_id` on the contract | `fleet.vehicle.log.contract` | Deletion is blocked while a contract names the type. |
| Fleet Service Type to Vehicle Service | one to many through `service_type_id` on the service | `fleet.vehicle.log.services` | Deletion is blocked while a service names the type, because the reference is required. |
| Fleet Service Type to Vehicle Contract, as an included service | many to many through the contract's `service_ids` | `fleet.vehicle.log.contract` | Deleting a type removes the association rows. |

---

# Part two: the record streams

## 8. Driver Assignment Log

**Driver Assignment Log** (`fleet.vehicle.assignation.log`, table `fleet_vehicle_assignation_log`). Generated reference page: [../../references/entities/fleet.vehicle.assignation.log.md](../../references/entities/fleet.vehicle.assignation.log.md).

### 8.1 Purpose

A Driver Assignment Log is one line of the answer to the question "who drove this vehicle, and when". One entry is opened automatically each time a vehicle receives a driver. Its end date is left empty at that moment and is filled in later, either by hand in response to the reminder activity the system schedules, or automatically when the driver leaves the company.

### 8.2 Ordering, display name and identity

- **Default ordering**: by the creation timestamp descending, then by `date_start` descending. The most recently recorded assignment therefore comes first even when two assignments start on the same day.
- **Display name**: the vehicle's name, a space, a hyphen, a space, then the driver's name. The rule is in [calculations.md](calculations.md).
- **Uniqueness**: none. A vehicle may have any number of overlapping entries; nothing prevents two open entries at once.
- **Archival**: none.
- **Discussion thread**: none. The entry does, however, accept attachments, and the people bridge counts them.

### 8.3 Complete field table

| Identifier | Full name | Type | Target | Required | Default | Computed rule and dependencies | Meaning |
|---|---|---|---|---|---|---|---|
| `vehicle_id` | Vehicle | reference to one | `fleet.vehicle` | yes | — | — | The vehicle that was driven. Indexed. |
| `driver_id` | Driver | reference to one | `res.partner` | yes | — | — | The Contact who drove it. |
| `date_start` | Start Date | date | — | no | — | — | The first day of the assignment. Set to today by the rule that opens the entry. |
| `date_end` | End Date | date | — | no | — | — | The last day of the assignment. Empty while the assignment is current. Set by hand, or to the departure date by the departure procedure. |
| `driver_employee_id` | Driver Employee | reference to one | `hr.employee` | no | — | `_compute_driver_employee_id`, depending on `driver_id`; stored; editable | Added by the people bridge. The Employee whose work contact is the driver and whose company is the **vehicle's** company. |
| `attachment_number` | Attachment Count | whole number | — | no | — | `_compute_attachment_number`, not stored | Added by the people bridge. The number of attachments stored against this entry, so that an insurance certificate or a handover form can be filed with the assignment. |

### 8.4 Operations

`action_get_attachment_view` opens the attachment browser restricted to the attachments of this entry, in a card layout whose cards open the stored file in a new window rather than opening the attachment record. Newly added attachments default to this entry as their owner.

---

## 9. Odometer Reading

**Odometer Reading** (`fleet.vehicle.odometer`, table `fleet_vehicle_odometer`). Generated reference page: [../../references/entities/fleet.vehicle.odometer.md](../../references/entities/fleet.vehicle.odometer.md).

### 9.1 Purpose

An Odometer Reading is one dated distance measurement for one vehicle. Readings are the raw material of the odometer analysis result set and of the distance shown on a service record. They are created in three ways: typed directly, written through the vehicle's distance field, or written through a service record's distance field.

### 9.2 Ordering, display name and identity

- **Default ordering**: by `date` descending. Readings without a date sort together.
- **Display name**: derived from the vehicle's name and the reading date by the rule in [calculations.md](calculations.md), and stored in `name`.
- **Uniqueness**: none. Several readings may share a vehicle and a date; the analysis result set resolves that by keeping the greatest value per vehicle and date.
- **Archival**: none.
- **Discussion thread**: none.
- **Aggregation**: the value field aggregates by maximum rather than by sum in grouped views, because summing successive odometer readings is meaningless.

### 9.3 Complete field table

| Identifier | Full name | Type | Target | Required | Default | Computed rule and dependencies | Meaning |
|---|---|---|---|---|---|---|---|
| `name` | Name | text | — | no | — | `_compute_vehicle_log_name`, depending on `vehicle_id` and `date`; stored | The display name of the reading. |
| `date` | Date | date | — | no | today | — | The day the measurement was taken. |
| `value` | Odometer Value | decimal | — | no | — | — | The distance shown on the instrument, expressed in the vehicle's odometer unit. Aggregated by maximum. |
| `vehicle_id` | Vehicle | reference to one | `fleet.vehicle` | yes | — | — | The vehicle measured. |
| `unit` | Unit | selection | — | no | — | mirrored from `vehicle_id.odometer_unit`; read-only | `kilometers` labelled "km"; `miles` labelled "mi". A change-reaction rule refreshes it in the editing client as soon as the vehicle is chosen, before the record is stored. |
| `driver_id` | Driver | reference to one | `res.partner` | no | — | `_compute_driver_id`, depending on `vehicle_id`; stored; editable | The Contact driving the vehicle when the measurement was taken. The rule fills it from the vehicle's current driver **only when it is still empty**, so an explicitly chosen driver is never overwritten by a later change of vehicle. |
| `driver_employee_id` | Driver Employee | reference to one | `hr.employee` | no | — | mirrored from `vehicle_id.driver_employee_id`; read-only | Added by the people bridge. The Employee currently behind the vehicle's driver. Because it mirrors the vehicle rather than the reading's own driver, it shows today's employee and not the employee at the time of the reading. **Compatibility finding** FLT-C07 in [business-rules.md](business-rules.md). |

---

## 10. Vehicle Service

**Vehicle Service** (`fleet.vehicle.log.services`, table `fleet_vehicle_log_services`). Generated reference page: [../../references/entities/fleet.vehicle.log.services.md](../../references/entities/fleet.vehicle.log.services.md).

### 10.1 Purpose

A Vehicle Service records one piece of work carried out on a vehicle: what was done, when, by whom, for how much, at what distance, and how far the job has progressed. A service is either typed by a fleet officer or created automatically when a vendor bill line that names a vehicle is posted; in the second case the service is bound to that journal item and its cost becomes read-only.

### 10.2 Inherited behaviour

The Vehicle Service carries a discussion thread and scheduled activities. The activities scheduled on services are what drive the vehicle's service-activity indicator.

### 10.3 Ordering, display name and identity

- **Default ordering**: none is declared, so the platform default of the surrogate identifier ascending applies.
- **Display name**: the name of the service kind, because `service_type_id` is declared as the record's naming field. Two services of the same kind on different vehicles therefore display identically; screens that list services always show the vehicle beside the kind for that reason.
- **Uniqueness**: none. A journal item may, however, bind to at most one service in practice, because the bridge creates exactly one per qualifying item and deletes it when the item stops naming a vehicle.
- **Archival**: the service carries `active`. Archiving the vehicle archives the service.

### 10.4 Complete field table

#### 10.4.1 Identification and progress

| Identifier | Full name | Type | Target | Required | Default | Computed rule and dependencies | Meaning |
|---|---|---|---|---|---|---|---|
| `service_type_id` | Service Type | reference to one | `fleet.service.type` | yes | the shipped type identified as `fleet.type_service_service_7`, when that record exists | — | The kind of work. The default resolves to a record that is only present when demonstration data has been loaded; on an installation without it, the field simply starts empty and the user must choose one. **Compatibility finding** FLT-C01. |
| `description` | Description | text | — | no | — | — | A one-line description of the work. Filled from the label of the bill line when the service comes from a vendor bill. |
| `notes` | Notes | long text | — | no | — | — | Free notes about the work done. |
| `date` | Date | date | — | no | today | — | The day the work was carried out. Drives the month a service cost falls into in the analysis result set. |
| `state` | Stage | selection | — | no | `new` | — | The progress of the job. `new` labelled "New"; `running` labelled "Running"; `done` labelled "Done"; `cancelled` labelled "Cancelled". Changes are recorded in the discussion thread. Grouping by this field expands to all four values so that an empty column still appears on the board. |
| `active` | Active | boolean | — | no | true | — | Whether the service is in the working set. |

#### 10.4.2 Vehicle and people

| Identifier | Full name | Type | Target | Required | Default | Computed rule and dependencies | Meaning |
|---|---|---|---|---|---|---|---|
| `vehicle_id` | Vehicle | reference to one | `fleet.vehicle` | yes | — | with the accounting bridge: `_compute_vehicle_id`, depending on `account_move_line_id.vehicle_id`; stored; editable | The vehicle serviced. Indexed. When the service is bound to a journal item, the rule follows the vehicle named on that item, but it never empties the field, because the field is required: an item that loses its vehicle causes the service to be deleted instead, see section 10.7. |
| `model_id` | Model | reference to one | `fleet.vehicle.model` | no | — | mirrored from `vehicle_id.model_id`; stored | The vehicle's model, stored so that services can be grouped by model. |
| `brand_id` | Manufacturer | reference to one | `fleet.vehicle.model.brand` | no | — | mirrored from `vehicle_id.model_id.brand_id`; stored | The vehicle's manufacturer, stored so that services can be grouped by manufacturer. |
| `manager_id` | Fleet Manager | reference to one | `res.users` | no | — | mirrored from `vehicle_id.manager_id`; stored | The vehicle's fleet manager, stored so that services can be grouped by the person answerable for them. |
| `purchaser_id` | Driver | reference to one | `res.partner` | no | — | `_compute_purchaser_id`, depending on `vehicle_id` and, with the people bridge, on `purchaser_employee_id`; stored; editable | The Contact who had the work done. Without the people bridge the rule copies the vehicle's driver. With it, a service that has a driver Employee takes that Employee's work contact instead, and the rest fall back to the vehicle's driver. |
| `purchaser_employee_id` | Driver Employee | reference to one | `hr.employee` | no | — | `_compute_purchaser_employee_id`, depending on `vehicle_id`; stored; editable | Added by the people bridge. Copies the vehicle's driver Employee. |
| `vendor_id` | Vendor | reference to one | `res.partner` | no | — | — | The Contact that carried out the work. Filled from the partner of the bill when the service comes from a vendor bill. |
| `inv_ref` | Vendor Reference | text | — | no | — | — | The reference the vendor put on their own document. Its full name is invoice reference. |

#### 10.4.3 Money

| Identifier | Full name | Type | Target | Required | Default | Computed rule and dependencies | Meaning |
|---|---|---|---|---|---|---|---|
| `amount` | Cost | monetary | — | no | — | without the accounting bridge: a plain stored field. With it: `_compute_amount`, depending on `account_move_line_id.price_subtotal`; stored; editable, and writing it runs `_inverse_amount` | The cost of the work in the company currency. Changes are recorded in the discussion thread. When the service is bound to a journal item, the value taken is the **debit** of that item, expressed in the company currency; typing a value is refused, see [business-rules.md](business-rules.md), rule FLT-012. |
| `currency_id` | Currency | reference to one | `res.currency` | no | — | mirrored from `company_id.currency_id`; not stored | The currency of the cost. |
| `company_id` | Company | reference to one | `res.company` | no | the current company | — | The company that owns the service. Used by the record rule that filters services by company. |

#### 10.4.4 Distance

| Identifier | Full name | Type | Target | Required | Default | Computed rule and dependencies | Meaning |
|---|---|---|---|---|---|---|---|
| `odometer_id` | Odometer Reading | reference to one | `fleet.vehicle.odometer` | no | — | — | The reading taken when the work was done. |
| `odometer` | Odometer Value | decimal | — | no | — | `_get_odometer`, not stored; writing it runs `_set_odometer` | Reading it yields the value of the linked reading, or zero when there is none. Writing a non-zero value creates a new reading dated on the service's date, or on today when the service has no date, and links it. Writing an empty or zero value is refused; see [business-rules.md](business-rules.md), rule FLT-003. A zero supplied at creation time is silently dropped instead of refused, so that a blank field on a new service does not create a reading of zero. |
| `odometer_unit` | Unit | selection | — | no | — | mirrored from `vehicle_id.odometer_unit`; read-only | The unit of the value, taken from the vehicle. |

#### 10.4.5 Accounting link

| Identifier | Full name | Type | Target | Required | Default | Computed rule and dependencies | Meaning |
|---|---|---|---|---|---|---|---|
| `account_move_line_id` | Journal Item | reference to one | `account.move.line` | no | — | — | Added by the accounting bridge. The posted vendor bill line that produced this service. Used as a one-to-one link: the bridge creates exactly one service per qualifying item. Indexed, skipping empty values. |
| `account_move_state` | Journal Entry State | selection | — | no | — | mirrored from `account_move_line_id.parent_state`; not stored | Added by the accounting bridge. The state of the journal entry the item belongs to, used to colour the button that opens the bill: green when the entry is posted and amber otherwise. |

### 10.5 Relations

| Relation | Cardinality | Other end | Deletion behaviour |
|---|---|---|---|
| Vehicle Service to Vehicle | many to one | `fleet.vehicle` | Required. Deleting a vehicle deletes its services. |
| Vehicle Service to Fleet Service Type | many to one | `fleet.service.type` | Required. The type may not be deleted while a service names it. |
| Vehicle Service to Odometer Reading | many to one | `fleet.vehicle.odometer` | Optional. Deleting the reading empties the link. |
| Vehicle Service to Journal Item | many to one, used as one to one | `account.move.line` | Deleting the item deletes the service, through the explicit bypass described in section 10.7. |

### 10.6 The cost of a billed service

When a service carries a journal item, its cost is the **debit** of that item. The debit is expressed in the company currency, while the untaxed subtotal named in the rule's dependency list is expressed in the document currency. On a bill in the company currency the two coincide; on a bill in another currency they do not, and the value stored is the company-currency one. This is deliberate — a fleet cost report in mixed currencies would otherwise be unusable — but the mismatch between the declared dependency and the value taken is a **compatibility finding**, recorded as FLT-C08.

### 10.7 Deletion guard and its bypass

A service that carries a journal item may not be deleted; the attempt is refused with the message quoted in [business-rules.md](business-rules.md), rule FLT-013. The guard is skipped when the deletion carries the bypass marker `ignore_linked_bill_constraint`. Exactly two callers set it, both in the accounting bridge:

1. Writing an empty vehicle onto a journal item, which deletes the services bound to that item before the write proceeds.
2. Deleting a journal item, which deletes the services bound to it before the item is removed.

No user-facing path sets the marker, so from a screen the guard is absolute.

---

## 11. Vehicle Contract

**Vehicle Contract** (`fleet.vehicle.log.contract`, table `fleet_vehicle_log_contract`). Generated reference page: [../../references/entities/fleet.vehicle.log.contract.md](../../references/entities/fleet.vehicle.log.contract.md).

### 11.1 Purpose

A Vehicle Contract records one coverage agreement on one vehicle: a lease, an all-risk insurance, a maintenance agreement or any other arrangement with a start date, an expiration date and a price. It answers five questions:

1. **What kind of agreement is it, and with whom?** A contract-category Fleet Service Type, an insurer Contact and a free reference.
2. **For how long?** A start date and an expiration date, the latter defaulting to one year after the former.
3. **What does it cost?** A one-off activation cost and a recurring cost with one of five frequencies.
4. **What does it include?** A set of Fleet Service Types listed as included services.
5. **Where is it in its life?** One of four states, and — while it is running or expired — the number of days left before it expires.

### 11.2 Inherited behaviour

The Vehicle Contract carries a discussion thread and scheduled activities. The renewal reminder the daily job raises is an activity of a dedicated type; see [configuration.md](configuration.md).

### 11.3 Ordering, display name and identity

- **Default ordering**: by `state` descending, then by `expiration_date` ascending. Because the stored state values sort alphabetically, descending order puts running contracts first, then incoming ones, then expired ones, then cancelled ones; within one state the soonest expiry comes first.
- **Display name**: the value of `name`, which is computed from the service type and the vehicle name and remains editable.
- **Uniqueness**: none.
- **Archival**: the contract carries `active`. Archiving the vehicle archives the contract.

### 11.4 Complete field table

#### 11.4.1 Identification

| Identifier | Full name | Type | Target | Required | Default | Computed rule and dependencies | Meaning |
|---|---|---|---|---|---|---|---|
| `name` | Name | text | — | no | — | `_compute_contract_name`, depending on `vehicle_id.name` and `cost_subtype_id`; stored; editable | The title of the contract. The assembly rule is in [calculations.md](calculations.md). |
| `cost_subtype_id` | Type | reference to one | `fleet.service.type` | no | — | — | The kind of coverage. Restricted to Fleet Service Types whose category is `contract`. |
| `ins_ref` | Reference | text | — | no | — | — | The policy or agreement number given by the insurer. Its full name is insurance reference. Limited to sixty-four characters. Not carried into a duplicate, because a duplicate is a different agreement. |
| `insurer_id` | Vendor | reference to one | `res.partner` | no | — | — | The Contact providing the coverage. |
| `notes` | Terms and Conditions | rich text | — | no | — | — | The terms of the agreement. Not carried into a duplicate. |
| `service_ids` | Included Services | collection of | `fleet.service.type` | no | — | — | The kinds of work the agreement covers. |
| `active` | Active | boolean | — | no | true | — | Whether the contract is in the working set. |

#### 11.4.2 Vehicle and people

| Identifier | Full name | Type | Target | Required | Default | Computed rule and dependencies | Meaning |
|---|---|---|---|---|---|---|---|
| `vehicle_id` | Vehicle | reference to one | `fleet.vehicle` | yes | — | — | The covered vehicle. Changes are recorded in the discussion thread. Indexed. Validated against the contract's company. |
| `purchaser_id` | Driver | reference to one | `res.partner` | no | — | mirrored from `vehicle_id.driver_id`; not stored | The Contact currently driving the covered vehicle. |
| `purchaser_employee_id` | Driver Employee | reference to one | `hr.employee` | no | — | mirrored from `vehicle_id.driver_employee_id`; not stored | Added by the people bridge. The Employee behind that driver. |
| `user_id` | Responsible | reference to one | `res.users` | no | the fleet manager of the vehicle named in the acting context, when the contract is created from a vehicle | — | The user answerable for renewing the contract and the user the renewal activity is assigned to. Indexed. |

#### 11.4.3 Dates and progress

| Identifier | Full name | Type | Target | Required | Default | Computed rule and dependencies | Meaning |
|---|---|---|---|---|---|---|---|
| `start_date` | Contract Start Date | date | — | no | today | — | The first day of coverage. Changes are recorded in the discussion thread. Writing it re-evaluates the state; see [state-machines.md](state-machines.md). |
| `expiration_date` | Contract Expiration Date | date | — | no | one year after today | — | The last day of coverage. Changes are recorded in the discussion thread. Writing it re-evaluates the state and reschedules the renewal activity. Required on the form whenever the recurring frequency is not `no`. |
| `date` | Date | date | — | no | — | — | The day the one-off activation cost was incurred. Decides the month the activation cost falls into in the analysis result set, and — for a yearly recurring cost — decides which calendar month of each year carries the recurring amount. |
| `state` | Status | selection | — | no | `open` | — | The lifecycle state. `futur` labelled "New"; `open` labelled "Running"; `expired` labelled "Expired"; `closed` labelled "Cancelled". Changes are recorded in the discussion thread. Not carried into a duplicate, so a duplicate starts at the default `open`. The complete machine is in [state-machines.md](state-machines.md). |
| `days_left` | Warning Date | whole number | — | no | — | `_compute_days_left`, depending on `expiration_date` and `state`; not stored | The number of days before expiry while the contract is running or expired and has an expiry date; zero once that day has arrived or passed; minus one in every other case. The rule is in [calculations.md](calculations.md). |
| `expires_today` | Expires Today | boolean | — | no | — | `_compute_days_left`, depending on `expiration_date` and `state`; not stored | True exactly on the expiry day of a running or expired contract. |
| `has_open_contract` | Vehicle Has A Running Contract | boolean | — | no | — | `_compute_has_open_contract`, depending on `vehicle_id`; not stored | True when the contract's vehicle has at least one contract in state `open` whose expiry is today or later. Used by the list view to suppress the red highlight on an expired contract whose vehicle is already covered again. |

#### 11.4.4 Money

| Identifier | Full name | Type | Target | Required | Default | Computed rule and dependencies | Meaning |
|---|---|---|---|---|---|---|---|
| `amount` | Cost | monetary | — | no | — | — | The one-off activation cost, paid once at the start of the agreement. Labelled "Activation Cost" on the form. Changes are recorded in the discussion thread. Falls into the month of `date` in the analysis result set. |
| `cost_generated` | Recurring Cost | monetary | — | no | — | — | The amount charged once per period of the chosen frequency. Changes are recorded in the discussion thread. |
| `cost_frequency` | Recurring Cost Frequency | selection | — | yes | `monthly` | — | `no` labelled "No"; `daily` labelled "Daily"; `weekly` labelled "Weekly"; `monthly` labelled "Monthly"; `yearly` labelled "Yearly". Changes are recorded in the discussion thread. The analysis result set spreads the recurring cost differently for each; the weekly frequency is not handled at all, which is **compatibility finding** FLT-C09. |
| `currency_id` | Currency | reference to one | `res.currency` | no | — | mirrored from `company_id.currency_id`; not stored | The currency of both amounts. |
| `company_id` | Company | reference to one | `res.company` | no | the current company | — | The company that owns the contract. |

### 11.5 Relations

| Relation | Cardinality | Other end | Deletion behaviour |
|---|---|---|---|
| Vehicle Contract to Vehicle | many to one | `fleet.vehicle` | Required. Deleting a vehicle deletes its contracts. |
| Vehicle Contract to Fleet Service Type as its kind | many to one | `fleet.service.type` | Optional. The type may not be deleted while a contract names it. |
| Vehicle Contract to Fleet Service Type as an included service | many to many | `fleet.service.type` | Deleting a type removes the association rows. |
| Vehicle Contract to Contact | many to one, as the insurer | `res.partner` | Optional. |
| Vehicle Contract to User | many to one, as the responsible | `res.users` | Optional. |

### 11.6 The renewal activity

Every running contract that has a responsible user and expires within the configured alert delay receives one activity of the dedicated renewal type, whose deadline is the contract's expiry date and whose responsible user is the contract's responsible. The daily job that creates it skips any contract that already carries an activity of that type, so exactly one reminder exists per contract. Changing the expiry date or the responsible user of a contract reschedules the existing activity rather than creating a second one. The complete procedure is in [workflows.md](workflows.md), procedure W-09.

---

# Part three: the analysis result sets

Both entities in this part are read-only derived result sets. They hold no stored rows of their own: each is rebuilt from the stored records whenever its package is installed or updated, and every read of it re-derives its rows. Neither may be created, and the access rights of [configuration.md](configuration.md) record where the system nevertheless grants write permissions that no screen uses.

## 12. Fleet Analysis Report

**Fleet Analysis Report** (`fleet.vehicle.cost.report`, derived set `fleet_vehicle_cost_report`). Generated reference page: [../../references/entities/fleet.vehicle.cost.report.md](../../references/entities/fleet.vehicle.cost.report.md).

### 12.1 Purpose

The Fleet Analysis Report answers the question "what did each vehicle cost me, month by month, and was it services or coverage". It produces one row per vehicle, per calendar month, per cost kind. The derivation, with every arithmetic step and a worked example, is in [calculations.md](calculations.md).

### 12.2 Ordering and identity

- **Default ordering**: by `date_start` descending, so the most recent month comes first.
- **Row identity**: a sequential number assigned over the rows ordered by vehicle ascending. It is not stable across rebuilds and must not be stored anywhere.
- **Archival, discussion thread, activities**: none.

### 12.3 Complete field table

| Identifier | Full name | Type | Target | Meaning |
|---|---|---|---|---|
| `vehicle_id` | Vehicle | reference to one | `fleet.vehicle` | The vehicle the cost belongs to. Read-only. |
| `name` | Vehicle Name | text | — | The vehicle's display name at the time the set was derived. Read-only. |
| `company_id` | Company | reference to one | `res.company` | The vehicle's company. Read-only. Used by the record rule that filters the set by company. |
| `driver_id` | Driver | reference to one | `res.partner` | The vehicle's **current** driver, not the driver at the time of the cost. Read-only. |
| `fuel_type` | Fuel | text | — | The vehicle's fuel kind, carried as plain text rather than as a selection, so it shows the stored value and not the label. **Compatibility finding** FLT-C10. Read-only. |
| `vehicle_type` | Vehicle Kind | selection | — | `car` labelled "Car"; `bike` labelled "Bike", taken from the vehicle's model. Read-only. |
| `date_start` | Date | date | — | The first day of the calendar month the cost is attributed to. Read-only. |
| `cost` | Cost | decimal | — | The amount attributed to that vehicle, that month and that cost kind, in the company currency. Read-only. |
| `cost_type` | Cost Type | selection | — | `contract` labelled "Contract"; `service` labelled "Service". Read-only. |

---

## 13. Fleet Odometer Analysis Report

**Fleet Odometer Analysis Report** (`fleet.vehicle.odometer.report`, derived set `fleet_vehicle_odometer_report`). Generated reference page: [../../references/entities/fleet.vehicle.odometer.report.md](../../references/entities/fleet.vehicle.odometer.report.md).

### 13.1 Purpose

The Fleet Odometer Analysis Report answers the question "how far did each vehicle travel in each month", given that odometer readings are recorded irregularly and sometimes not at all for months on end. It fills the gaps by interpolating between the readings that exist, proportionally to elapsed days, and splits a reading that straddles a month boundary between the two months. The fifteen-step derivation is in [calculations.md](calculations.md).

### 13.2 Ordering and identity

- **Default ordering**: by `recorded_date` descending.
- **Row identity**: a sequential number assigned over the derived rows. Not stable across rebuilds.
- **Archival, discussion thread, activities**: none.

### 13.3 Complete field table

| Identifier | Full name | Type | Target | Meaning |
|---|---|---|---|---|
| `vehicle_id` | Vehicle | reference to one | `fleet.vehicle` | The vehicle the distance belongs to. Read-only. |
| `category_id` | Category | reference to one | `fleet.vehicle.model.category` | Mirrored from `vehicle_id.category_id`. Offered as a grouping axis and applied by default when the screen opens. |
| `model_id` | Model | reference to one | `fleet.vehicle.model` | Mirrored from `vehicle_id.model_id`. Offered as a grouping axis. |
| `fuel_type` | Fuel Kind | selection | — | Mirrored from `vehicle_id.fuel_type`, so here it does show labels. Offered as a grouping axis. |
| `recorded_date` | Date | date | — | The first day of the calendar month the distance is attributed to. Read-only. |
| `mileage_delta` | Distance Travelled | decimal | — | The distance attributed to that vehicle and that month, in the vehicle's odometer unit. Read-only. This is the measure the timeline plots. |
| `odometer_value` | Odometer Value | decimal | — | The running total of the distance travelled up to and including that month. Read-only. |

---

## 14. Send Mails to Drivers

**Send Mails to Drivers** (`fleet.vehicle.send.mail`, transient assistant `fleet_vehicle_send_mail`). Generated reference page: [../../references/entities/fleet.vehicle.send.mail.md](../../references/entities/fleet.vehicle.send.mail.md).

### 14.1 Purpose

The Send Mails to Drivers assistant composes one message and posts a rendered copy of it to each selected vehicle, addressed to that vehicle's driver. It exists so that a fleet officer can tell every driver about a tyre-change campaign or a parking change in one operation, while each message still lands on the thread of the vehicle it concerns and so stays traceable.

### 14.2 Inherited behaviour

The assistant carries the platform's message composition behaviour: a subject and a body that follow a chosen message template, the template reference itself, the rendering language, the flag that says whether the body still matches the template, and the two flags that decide whether the acting user may edit the raw body. Those fields are specified in [../messaging-and-activities/](../messaging-and-activities/). The assistant fixes the rendering target to the Vehicle, so every placeholder in a template is resolved against a vehicle record.

### 14.3 Complete field table

| Identifier | Full name | Type | Target | Required | Default | Computed rule and dependencies | Meaning |
|---|---|---|---|---|---|---|---|
| `vehicle_ids` | Vehicles | collection of | `fleet.vehicle` | yes | the records the operation was launched on | — | The vehicles whose drivers will be written to. |
| `author_id` | Author | reference to one | `res.partner` | yes | the Contact of the acting user | — | The Contact the messages are posted as. |
| `template_id` | Message Template | reference to one | `mail.template` | no | — | — | An optional stored template. Its choice is restricted to templates bound to the Vehicle. Choosing one copies its subject and its body into the assistant and replaces the assistant's attachments with the template's. |
| `attachment_ids` | Attachments | collection of | `ir.attachment` | no | — | — | Files to carry with the messages. Held in the association table `fleet_vehicle_mail_compose_message_ir_attachments_rel`, whose column `wizard_id` holds the assistant and whose column `attachment_id` holds the attachment. Reading the collection bypasses the attachment access filter, so a file attached by another user is still visible in the assistant. |
| `subject` | Subject | text | — | yes on the form | — | from the composition behaviour, depending on `template_id`; stored; editable | The subject line. |
| `body` | Contents | rich text | — | no | — | from the composition behaviour, depending on `template_id`; stored; editable | The message body. |
| `render_model` | Rendering Target | text | — | — | — | `_compute_render_model`, depending on `subject` | Fixed to the Vehicle. |

### 14.4 Operations

Both operations are specified step by step in [workflows.md](workflows.md), procedures W-11 and W-12:

- `action_send` posts one message per vehicle, or refuses when a driver has no electronic mail address.
- `action_save_as_template` turns the composed subject, body and attachments into a new stored template bound to the Vehicle and opens it.

---

# Part four: fields this domain adds to entities of other domains

## 15. Journal Item

**Journal Item** (`account.move.line`), owned by [general ledger](../general-ledger/). The accounting bridge adds:

| Identifier | Full name | Type | Target | Required | Default | Computed rule and dependencies | Meaning |
|---|---|---|---|---|---|---|---|
| `vehicle_id` | Vehicle | reference to one | `fleet.vehicle` | no | — | — | The vehicle the item's amount belongs to. Indexed, skipping empty values. Shown as an optional column on the item lines of a bill and on the general journal item list. Only shown at all when the entry is a vendor bill, a vendor credit note or a vendor receipt. |
| `need_vehicle` | Vehicle Required | boolean | — | no | — | `_compute_need_vehicle`, not stored | Whether the vehicle must be filled on this item. The bridge always yields false; it exists purely so that a localization whose road-tax or benefit-in-kind rules require a vehicle on certain accounts can override the rule and make the column mandatory. Its always-false result is **compatibility finding** FLT-C11. When true, the client makes the vehicle column required on vendor bills and vendor credit notes. |
| `vehicle_log_service_ids` | Vehicle Services | collection of | `fleet.vehicle.log.services` | no | — | inverse field `account_move_line_id` | The Vehicle Service generated from this item. Declared as a collection but used as a single link: at most one service exists per item. Excluded from exported string translations. |

Changed behaviour:

- **Writing** an empty vehicle onto an item first deletes the services bound to it, with the deletion guard bypassed.
- **Deleting** an item first deletes the services bound to it, with the deletion guard bypassed.

## 16. Journal Entry

**Journal Entry** (`account.move`), owned by [general ledger](../general-ledger/). The accounting bridge adds no field. It changes posting: after an entry is posted for the first time, every product item of it that names a vehicle and does not already carry a service produces one Vehicle Service. The complete rule, including every skip condition and the message logged on the new service, is in [workflows.md](workflows.md), procedure W-07, and its ledger consequences are in [accounting-effects.md](accounting-effects.md).

## 17. Create Automatic Entries

**Create Automatic Entries** (`account.automatic.entry.wizard`), owned by [general ledger](../general-ledger/). The accounting bridge adds no field. It changes the preparation of the counterpart items of a period change: when the source item names a vehicle, every generated item that uses the same account as the source item also names that vehicle. Items that use the accrual account do not, because they are not the continuation of the original expense line.

## 18. Employee

**Employee** (`hr.employee`), owned by [human resources core](../human-resources-core/). The people bridge adds:

| Identifier | Full name | Type | Target | Required | Default | Computed rule and dependencies | Meaning |
|---|---|---|---|---|---|---|---|
| `car_ids` | Vehicles | collection of | `fleet.vehicle` | no | — | inverse field `driver_employee_id` | The vehicles this employee currently drives. Readable only by members of the fleet administrator group or the human resources officer group. |
| `employee_cars_count` | Vehicle Count | whole number | — | no | — | `_compute_employee_cars_count`, not stored | The number of driver assignment entries naming this employee **and** naming the employee's work contact as driver — that is, the length of the employee's vehicle history, not the number of vehicles held now. Readable only by members of the fleet administrator group. |
| `license_plate` | Licence Plate | text | — | no | — | `_compute_license_plate`, depending on `private_car_plate` and `car_ids.license_plate`; not stored; searchable through `_search_license_plate` | The plates of the employee's company vehicles joined by single spaces, with the employee's private plate appended when both exist. Readable only by members of the human resources officer group. The assembly rule is in [calculations.md](calculations.md). |
| `mobility_card` | Mobility Card | text | — | no | — | — | The reference of a fuel or mobility card issued to the employee. Readable only by members of the fleet officer group. Mirrored onto every vehicle the employee drives. |

Changed behaviour:

- A constraint on the work contact refuses to empty it while vehicles name the employee as driver; see [business-rules.md](business-rules.md), rule FLT-014.
- Writing a new work contact onto an employee rewrites the driver of every vehicle that names the employee as driver Employee, and the future driver of every vehicle that names the employee as future driver Employee.
- Writing a new mobility card re-derives the mobility card on every vehicle the employee drives.
- The operation `action_open_employee_cars` opens the employee's driver assignment entries, restricted to entries that name both the employee and the employee's work contact, under the title "Cars History".

## 19. Public Employee Profile

**Public Employee Profile** (`hr.employee.public`), owned by [human resources core](../human-resources-core/). The people bridge adds one read-only field `mobility_card`, the mobility card, so that the reference is visible on the profile a colleague may read without holding human resources permissions.

## 20. Departure Wizard

**Departure Wizard** (`hr.departure.wizard`), owned by [human resources core](../human-resources-core/). The people bridge adds:

| Identifier | Full name | Type | Target | Required | Default | Computed rule and dependencies | Meaning |
|---|---|---|---|---|---|---|---|
| `release_campany_car` | Release Company Vehicle | boolean | — | no | true when the acting user is a member of the fleet officer group, false otherwise | — | Whether registering the departure should also free the vehicles the leaver drove. The storage name is reproduced exactly, including its spelling. Labelled "Company Car" beside the other departure options. |

Changed behaviour: registering a departure runs the release procedure of [workflows.md](workflows.md), procedure W-10, when the flag is set. The departure form is also changed so that the block of activity options, which is normally hidden, is always shown, because this option lives in it.

## 21. Activity Plan Template

**Activity Plan Template** (`mail.activity.plan.template`), owned by [messaging and activities](../messaging-and-activities/). The people bridge adds one more stored value to the responsible-kind selection: `fleet_manager`, labelled "Fleet Manager". Removing the bridge sets that value back to the selection's default on every template that used it.

Changed behaviour:

- A constraint refuses the new kind on a plan that does not run on Employees; see [business-rules.md](business-rules.md), rule FLT-015.
- The resolution rule takes the employee's first vehicle and yields its fleet manager, with an error when the employee has no vehicle and a warning when the vehicle has no manager; see [business-rules.md](business-rules.md), rules FLT-016 and FLT-017.

## 22. Activity Type

**Activity Type** (`mail.activity.type`), owned by [messaging and activities](../messaging-and-activities/). The fleet package adds one entry to the map that tells the platform which activity types are raised by a scheduled job: the renewal type is declared to run on Vehicle Contracts and to be **kept** rather than removed when the job runs again. That is what makes the renewal reminder appear exactly once per contract.

## 23. Attachment

**Attachment** (`ir.attachment`), platform foundation; see [../../runtime/attachments-and-file-store.md](../../runtime/attachments-and-file-store.md). The people bridge adds the operation `action_preview_attachment`, which opens the stored file itself in a new window instead of opening the attachment record. It is bound to the card view used by the driver assignment attachment browser.

## 24. Configuration Settings

**Configuration Settings** (`res.config.settings`), platform foundation; see [../../runtime/configuration-parameters.md](../../runtime/configuration-parameters.md). The fleet package adds:

| Identifier | Full name | Type | Required | Default | Meaning |
|---|---|---|---|---|---|
| `delay_alert_contract` | Contract Expiry Alert Delay | whole number | no | 30 | The number of days before a contract's expiry at which the system starts calling it due for renewal and starts raising the renewal activity. Stored in the system parameter `hr_fleet.delay_alert_contract`. |

## 25. Batch Transfer

**Batch Transfer** (`stock.picking.batch`), owned by [inventory operations](../inventory-operations/). The transport dispatch bridge adds:

| Identifier | Full name | Type | Target | Required | Default | Computed rule and dependencies | Meaning |
|---|---|---|---|---|---|---|---|
| `vehicle_id` | Vehicle | reference to one | `fleet.vehicle` | no | — | — | The vehicle that will carry the batch. Leaving it empty while filling the category means a third-party carrier of that class. |
| `vehicle_category_id` | Vehicle Category | reference to one | `fleet.vehicle.model.category` | no | — | `_compute_vehicle_category_id`, depending on `vehicle_id`; stored; editable | The class of vehicle needed. Follows the chosen vehicle's category, and may be set on its own when no owned vehicle is chosen. |
| `driver_id` | Driver | reference to one | `res.partner` | no | — | `_compute_driver_id`, depending on `vehicle_id`; stored; editable | The person who will drive. Follows the chosen vehicle's driver and may be overridden. |
| `dock_id` | Dock | reference to one | `stock.location` | no | — | `_compute_dock_id`, depending on `picking_ids`, `picking_ids.location_id`, `picking_ids.location_dest_id` and `picking_type_id`; stored; editable | The loading or unloading bay the batch uses. Restricted to locations under the operation type's dock set. |
| `allowed_dock_ids` | Allowed Docks | collection of | `stock.location` | no | — | mirrored from `picking_type_id.dock_ids`; not stored | The dock set the operation type publishes. |
| `vehicle_weight_capacity` | Vehicle Payload Capacity | decimal | — | no | — | mirrored from `vehicle_category_id.weight_capacity`; not stored | The greatest weight the chosen class can carry. |
| `vehicle_volume_capacity` | Vehicle Volume Capacity | decimal | — | no | — | mirrored from `vehicle_category_id.volume_capacity`; not stored | The greatest volume the chosen class can carry. |
| `weight_uom_name` | Weight Unit Label | text | — | no | — | `_compute_weight_uom_name`, not stored | The label of the platform's weight unit. |
| `volume_uom_name` | Volume Unit Label | text | — | no | — | `_compute_volume_uom_name`, not stored | The label of the platform's volume unit. |
| `used_weight_percentage` | Weight Share Used | decimal | — | no | — | `_compute_capacity_percentage`, depending on `estimated_shipping_weight`, `vehicle_category_id.weight_capacity`, `estimated_shipping_volume` and `vehicle_category_id.volume_capacity`; not stored | The share of the weight capacity the planned load takes, as a percentage. The formula is in [calculations.md](calculations.md). |
| `used_volume_percentage` | Volume Share Used | decimal | — | no | — | the same rule | The share of the volume capacity the planned load takes, as a percentage. |
| `end_date` | End Date | date and time | — | no | — | `_compute_end_date`, depending on `scheduled_date`; stored; editable | The planned end of the dispatch window. Set to one hour after the scheduled date whenever it is empty or earlier than the scheduled date, and left alone otherwise. |
| `has_dispatch_management` | Dispatch Management | boolean | — | no | — | mirrored from `picking_type_id.dispatch_management`; not stored | Whether the dispatch group is shown on the batch form at all. |

Changed behaviour:

- Creating a batch orders its transfers by customer postal code and, when a dock is already set, redirects the moves to that dock.
- Writing a dock onto a batch redirects the moves to it, or resets them when the dock is emptied. The redirection rule is in [workflows.md](workflows.md), procedure W-13.
- Merging batches carries the vehicle and the dock of the batch being merged into the merged result.

## 26. Operation Type

**Operation Type** (`stock.picking.type`), owned by [inventory operations](../inventory-operations/). The transport dispatch bridge adds:

| Identifier | Full name | Type | Target | Required | Default | Computed rule and dependencies | Meaning |
|---|---|---|---|---|---|---|---|
| `dispatch_management` | Dispatch Management | boolean | — | no | false | — | Whether the dispatch fields and the dispatch menu entries appear for this operation type. |
| `dock_ids` | Dock Locations | collection of | `stock.location` | no | — | `_compute_dock_ids`, depending on `warehouse_id`; stored; editable | The internal locations of the operation type's warehouse that serve as loading or unloading bays. Cleared when the warehouse changes. Held in the association table `dock_location_stock_picking_type_rel`. |

## 27. Transfer

**Transfer** (`stock.picking`), owned by [inventory operations](../inventory-operations/). The transport dispatch bridge adds one field `zip`, the customer postal code, mirrored from the transfer's Contact and searchable through a rule that forwards the search to the Contact. It is the key the batch ordering uses. The bridge also changes writing: assigning a transfer to a batch redirects or resets its move destinations according to the batch's dock.

## 28. Warehouse

**Warehouse** (`stock.warehouse`), owned by [inventory operations](../inventory-operations/). The transport dispatch bridge adds no field. It changes the values a warehouse writes onto its operation types: dispatch management is switched on for the outgoing type and the incoming type always, for the packing type when the warehouse ships in three steps, and for the picking type when it ships in two; and for a warehouse that ships in two or three steps the outgoing type's dock set is seeded with the warehouse's output location. Installing the bridge applies the same switch to every warehouse that already exists.

---

# Part five: cross-entity invariants

1. **One vehicle, one current driver.** The vehicle holds exactly one driver reference. History is kept in the assignment entries, not in the vehicle. Nothing enforces that at most one assignment entry is open at a time; a rebuild that wants that guarantee must add it, and this specification records that the observed system does not.
2. **A service's vehicle is never emptied.** The rule that follows a journal item's vehicle skips items with no vehicle, and the bridge deletes the service instead. There is therefore no state in which a service exists without a vehicle.
3. **A billed service's cost equals its item's debit.** Every path that could set the cost independently is closed: typing is refused, and the rule re-derives the value whenever the item changes.
4. **Archive follows the vehicle downward only.** Archiving a vehicle archives its contracts and services; restoring the vehicle restores nothing.
5. **Counters match the archive side.** The service and contract counters on a vehicle count records on the same side of the archive line as the vehicle itself.
6. **A contract's company owns its vehicle.** The company check on the contract's vehicle reference makes the two consistent; the same check applies to a vehicle's future driver.
7. **Emission unit follows range unit.** On both the model and the vehicle the emission unit is derived from the range unit and cannot be set independently.
8. **A driver Employee always matches the vehicle's company.** The resolution rule keys on the pair of work contact and company, so a vehicle of one company never resolves to an employee of another.
