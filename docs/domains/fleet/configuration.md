# Configuration

This file specifies everything that is configured rather than transacted in the fleet domain: the settings a user can change, the system parameter behind them, the sequences the domain uses, the records shipped with the capability packages, the security groups and their privilege, the complete access-rights matrix, every record rule, the scheduled job, the activity type, the message subtype, the activity plan template, and the optional sample data set.

## 1. Capability packages

The domain is delivered by four packages. Each is named here by its business name; the technical name a package manager would use appears only in the machine-readable catalogues.

| Package | What it delivers | Depends on | Installed automatically |
|---|---|---|---|
| Fleet | Every entity of this domain, the two analysis result sets, the send-mail assistant, the security groups, the access rights, the record rules, the scheduled job, the activity type, the message subtype, the sixty-seven manufacturers, the four statuses and the two contract-category service kinds | The platform foundation and the messaging capability | No; it is an application a user installs |
| Accounting and Fleet bridge | The vehicle dimension on journal items, the creation of Vehicle Services from posted vendor bills, the vehicle carried through the accrual assistant, the vendor-bill service kind, the bill counter on the vehicle | Fleet and the accounting capability | Yes, as soon as both of its dependencies are present |
| Fleet History bridge to the people register | The driver Employee and future driver Employee on vehicles and assignment entries, the mobility card, the employee vehicle counters and aggregated plate, the departure release option, the fleet-manager responsible kind for activity plans, the attachment preview and the attachment counter | Fleet and the people register | Yes, as soon as both are present |
| Transport Dispatch bridge | The weight and volume capacities on Vehicle Categories, the vehicle, category, driver and dock on batch transfers, the dispatch-management flag and dock set on operation types, the postal code on transfers, and the batch document additions | Fleet and the batch-transfer capability | No; it is installed on demand. Installing it switches dispatch management on for every existing warehouse |

## 2. Settings

One setting is offered, on a dedicated panel of the general settings screen headed "Fleet". The panel is visible only to a member of the fleet administrator group, and the settings screen itself only to a member of the system administration group.

| Setting label | Presentation | Field | Default | Stored in | Meaning |
|---|---|---|---|---|---|
| "End Date Contract Alert" | The sentence "Send an alert &lt;number&gt; days before the end date", with the number typed in a narrow box in the middle | `delay_alert_contract`, the contract expiry alert delay | 30 | The system parameter `hr_fleet.delay_alert_contract` | The number of days before a contract's expiration date at which the system begins to call the renewal due soon and begins to raise the renewal activity |

**Where the value is consumed.** In three places, all specified in [calculations.md](calculations.md): the derived renewal flags of a vehicle, C-16; the search for vehicles whose renewal is due soon, C-17; and the first pass of the daily job. Each of the three reads the parameter with elevated permissions and falls back to 30 when it is unset.

**The parameter key carries the name of the people bridge.** The key is reproduced exactly as `hr_fleet.delay_alert_contract` even though the setting is declared by the Fleet package and works without the people bridge. A rebuild must use the same key, because an installation that has already stored a value stores it under that key.

## 3. Sequences

**The domain declares no numbering sequence.** No entity of this domain carries a generated reference. The identifying text of each is either typed by a user — the licence plate, the chassis number, the contract reference — or derived from other fields; the derivations are in [calculations.md](calculations.md), C-01 to C-06.

## 4. Records shipped with the ordinary installation

### 4.1 Vehicle statuses

Four Vehicle Statuses are shipped. They are ordinary records: a user holding the fleet administrator group may rename, reorder, fold, delete or add to them. They are loaded once and are not overwritten by a later update of the package, so local changes survive.

| External identifier | Name | Sequence | Folded | Referenced by a rule |
|---|---|---|---|---|
| `fleet_vehicle_state_new_request` | "New Request" | 4 | no | Yes, twice: it is the default status of a new vehicle, and it is one of the two statuses that suppress the plan-to-change marking when a future driver is set |
| `fleet_vehicle_state_to_order` | "To Order" | 5 | no | No |
| `fleet_vehicle_state_registered` | "Registered" | 7 | no | No |
| `fleet_vehicle_state_downgraded` | "Downgraded" | 8 | no | No |

The gaps at sequences 6, 9 and 10 are the positions of three further statuses that exist only in the sample data set; see section 9.

### 4.2 Fleet Service Types

Three Fleet Service Types are shipped with the ordinary installation, two by the Fleet package and one by the accounting bridge.

| External identifier | Name | Category | Delivered by | Referenced by a rule |
|---|---|---|---|---|
| `type_contract_omnium` | "Omnium" | `contract` | Fleet | No. The name denotes an all-risk motor insurance |
| `type_contract_leasing` | "Leasing" | `contract` | Fleet | No |
| `data_fleet_service_type_vendor_bill` | "Vendor Bill" | `service` | Accounting and Fleet bridge | **Yes.** It is the kind given to every Vehicle Service created from a posted vendor bill. When the record is absent, the whole creation routine is skipped and posting behaves as it does without the bridge |

Unlike the statuses, the vendor-bill service kind is loaded in updatable mode, so a later update of the bridge restores its name and category if they were changed.

### 4.3 Vehicle Manufacturers

Sixty-seven Vehicle Manufacturers are shipped by the Fleet package, each with a name and a logo. They are loaded in updatable mode. None of them is referenced by any rule; they exist so that a new installation can create models immediately.

| External identifier | Name | | External identifier | Name |
|---|---|---|---|---|
| `brand_abarth` | Abarth | | `brand_lexus` | Lexus |
| `brand_acura` | Acura | | `brand_lincoln` | Lincoln |
| `brand_alfa` | Alfa | | `brand_lotus` | Lotus |
| `brand_audi` | Audi | | `brand_maserati` | Maserati |
| `brand_austin` | Austin | | `brand_maybach` | Maybach |
| `brand_bentley` | Bentley | | `brand_mazda` | Mazda |
| `brand_bmw` | BMW | | `brand_mercedes` | Mercedes |
| `brand_bugatti` | Bugatti | | `brand_mg` | MG |
| `brand_buick` | Buick | | `brand_mini` | Mini |
| `brand_byd` | BYD | | `brand_mitsubishi` | Mitsubishi |
| `brand_cadillac` | Cadillac | | `brand_morgan` | Morgan |
| `brand_chevrolet` | Chevrolet | | `brand_nissan` | Nissan |
| `brand_chrysler` | Chrysler | | `brand_oldsmobile` | Oldsmobile |
| `brand_citroen` | Citroen | | `brand_opel` | Opel |
| `brand_corre_la_licorne` | Corre La Licorne | | `brand_peugeot` | Peugeot |
| `brand_daewoo` | Daewoo | | `brand_pontiac` | Pontiac |
| `brand_dodge` | Dodge | | `brand_porsche` | Porsche |
| `brand_ferrari` | Ferrari | | `brand_rambler` | Rambler |
| `brand_fiat` | Fiat | | `brand_renault` | Renault |
| `brand_ford` | Ford | | `brand_rolls-royce` | Rolls-Royce |
| `brand_gmc` | GMC | | `brand_saab` | Saab |
| `brand_holden` | Holden | | `brand_scion` | Scion |
| `brand_honda` | Honda | | `brand_skoda` | Skoda |
| `brand_hyundai` | Hyundai | | `brand_smart` | Smart |
| `brand_infiniti` | Infiniti | | `brand_steyr` | Steyr |
| `brand_isuzu` | Isuzu | | `brand_subaru` | Subaru |
| `brand_jaguar` | Jaguar | | `brand_suzuki` | Suzuki |
| `brand_jeep` | Jeep | | `brand_tesla_motors` | Tesla Motors |
| `brand_kia` | Kia | | `brand_toyota` | Toyota |
| `brand_koenigsegg` | Koenigsegg | | `brand_trabant` | Trabant |
| `brand_lagonda` | Lagonda | | `brand_volkswagen` | Volkswagen |
| `brand_lamborghini` | Lamborghini | | `brand_volvo` | Volvo |
| `brand_lancia` | Lancia | | `brand_willys` | Willys |
| `brand_land_rover` | Land Rover | | | |

The external identifier of the sixty-seventh entry contains a hyphen where the others use an underscore; it is reproduced exactly.

### 4.4 Nothing else is shipped

No Vehicle Model, no Vehicle Category, no Vehicle Tag and no Vehicle is shipped with the ordinary installation. A new installation therefore has manufacturers, statuses and three service kinds, and nothing else.

## 5. Security groups

### 5.1 The privilege

One group privilege is declared, so that the two groups appear together as one line of the user form.

| External identifier | Name | Sequence | Category |
|---|---|---|---|
| `res_groups_privilege_fleet` | "Fleet" | 17 | The people-management application category |

### 5.2 The groups

| External identifier | Name | Sequence | Implies | Granted at installation to |
|---|---|---|---|---|
| `fleet_group_user` | "Officer: Manage all vehicles" | 10 | The internal-user group | Nobody |
| `fleet_group_manager` | "Administrator" | 20 | `fleet_group_user`, and through it the internal-user group | The automation user and the administrator user |

**The implication matters.** Because the administrator group implies the officer group, every access right and every group record rule granted to the officer reaches the administrator as well. The odometer access right is granted to the officer group only, and administrators hold it through that implication.

**A user who holds neither group** sees no fleet menu at all — the root menu is restricted to the officer group — and has no access to any fleet entity except through the two human-resources rows of section 6.

## 6. Access rights

The matrix below is complete. R means read, W means write, C means create and D means delete. A blank cell means the group has no such permission through that row; it may still hold it through the implication of section 5.2.

| Entity | Group | R | W | C | D | Delivered by |
|---|---|---|---|---|---|---|
| Vehicle Model (`fleet.vehicle.model`) | Fleet officer | R | | | | Fleet |
| Vehicle Model | Fleet administrator | R | W | C | D | Fleet |
| Vehicle Manufacturer (`fleet.vehicle.model.brand`) | Fleet officer | R | | | | Fleet |
| Vehicle Manufacturer | Fleet administrator | R | W | C | D | Fleet |
| Vehicle Category (`fleet.vehicle.model.category`) | Fleet officer | R | | | | Fleet |
| Vehicle Category | Fleet administrator | R | W | C | D | Fleet |
| Vehicle Status (`fleet.vehicle.state`) | Fleet officer | R | | | | Fleet |
| Vehicle Status | Fleet administrator | R | W | C | D | Fleet |
| Vehicle Tag (`fleet.vehicle.tag`) | Fleet officer | R | | | | Fleet |
| Vehicle Tag | Fleet administrator | R | W | C | D | Fleet |
| Fleet Service Type (`fleet.service.type`) | Fleet officer | R | | | | Fleet |
| Fleet Service Type | Fleet administrator | R | W | C | D | Fleet |
| Vehicle (`fleet.vehicle`) | Fleet officer | R | W | C | D | Fleet |
| Vehicle | Fleet administrator | R | W | C | D | Fleet |
| Vehicle | Human resources officer | R | | | | Fleet History bridge |
| Vehicle Contract (`fleet.vehicle.log.contract`) | Fleet officer | R | W | C | D | Fleet |
| Vehicle Contract | Fleet administrator | R | W | C | D | Fleet |
| Vehicle Service (`fleet.vehicle.log.services`) | Fleet officer | R | | | | Fleet |
| Vehicle Service | Fleet administrator | R | W | C | D | Fleet |
| Odometer Reading (`fleet.vehicle.odometer`) | Fleet officer | R | W | C | D | Fleet |
| Driver Assignment Log (`fleet.vehicle.assignation.log`) | Fleet officer | R | W | C | D | Fleet |
| Fleet Analysis Report (`fleet.vehicle.cost.report`) | Fleet administrator | R | | | | Fleet |
| Fleet Odometer Analysis Report (`fleet.vehicle.odometer.report`) | Fleet administrator | R | W | C | D | Fleet |
| Send Mails to Drivers (`fleet.vehicle.send.mail`) | Fleet administrator | R | W | C | | Fleet |
| Activity Type (`mail.activity.type`) | Fleet administrator | R | W | C | D | Fleet |

**Four observations a rebuild must reproduce.**

1. **A fleet officer may not create a Vehicle Service.** Officers may read services but not write, create or delete them. Recording work done therefore requires the administrator group, while recording a contract does not. This is stated as rule FLT-038 in [business-rules.md](business-rules.md).
2. **Odometer readings and assignment entries are granted to the officer group only**, with no separate administrator row. Administrators reach them through the group implication.
3. **The Fleet Odometer Analysis Report is granted write, create and delete.** No screen offers any of them and the set is derived, so the permissions are unreachable; see rule FLT-039.
4. **The send assistant is granted no delete.** A transient record that is never deleted by a user is cleaned up by the platform's own housekeeping of transient records; see [../../runtime/transactions-and-concurrency.md](../../runtime/transactions-and-concurrency.md).

## 7. Record rules

Ten record rules are declared. All are loaded once and are not overwritten by a later update.

### 7.1 Multi-company rules

These five carry no group, which makes them global: they apply to every user, including administrators, and they are combined with any group rule by conjunction.

| External identifier | Entity | Permissions | Condition |
|---|---|---|---|
| `ir_rule_fleet_vehicle` | Vehicle | all four | The vehicle's company is one of the acting user's allowed companies, or is empty |
| `ir_rule_fleet_vehicle_log_contract` | Vehicle Contract | all four | The contract's company is one of the acting user's allowed companies, or is empty |
| `ir_rule_fleet_log_services` | Vehicle Service | all four | The service's company is one of the acting user's allowed companies, or is empty |
| `ir_rule_fleet_odometer` | Odometer Reading | all four | The **vehicle's** company is one of the acting user's allowed companies, or is empty |
| `ir_rule_fleet_report` | Fleet Analysis Report | all four | The row's company is one of the acting user's allowed companies, or is empty |

Two of the five — the odometer rule and the service rule — declare the global flag explicitly. The other three are global because they name no group. The effect is identical.

**Not covered by any company rule:** Driver Assignment Log, Vehicle Model, Vehicle Manufacturer, Vehicle Category, Vehicle Status, Vehicle Tag, Fleet Service Type and the Fleet Odometer Analysis Report. For the six configuration entities that is deliberate. For the assignment log and the distance analysis it is a gap; see rule FLT-036 in [business-rules.md](business-rules.md) for the industry-standard default a rebuild should apply.

### 7.2 The four unrestricting administrator rules

These four carry the fleet administrator group and declare **no condition**, which admits every record. Because group rules on one entity are combined by disjunction, their purpose is to neutralise a narrowing group rule for a user who is also in another group — in practice, the human-resources rule of section 7.3.

| External identifier | Entity | Group | Permissions | Condition |
|---|---|---|---|---|
| `fleet_rule_vehicle_visibility_manager` | Vehicle | Fleet administrator | all four | none, so everything is admitted |
| `fleet_rule_contract_visibility_manager` | Vehicle Contract | Fleet administrator | all four | none |
| `fleet_rule_service_visibility_manager` | Vehicle Service | Fleet administrator | all four | none |
| `fleet_rule_odometer_visibility_manager` | Odometer Reading | Fleet administrator | all four | none |

They do not weaken the multi-company rules of section 7.1, which are global and therefore always conjoined.

### 7.3 The human resources rule

| External identifier | Entity | Group | Permissions | Condition |
|---|---|---|---|---|
| `hr_fleet_rule_vehicle_visibility_hr_officier` | Vehicle | Human resources officer | **read only**; write, create and delete are explicitly withheld | The vehicle names a driver Employee, or names a future driver Employee |

Delivered by the Fleet History bridge. Its external identifier is reproduced exactly, including its spelling.

## 8. Scheduled job, activity type, message subtype and activity plan

### 8.1 The scheduled job

| External identifier | Name | Runs on | Interval | Runs as | What it does |
|---|---|---|---|---|---|
| `ir_cron_contract_costs_generator` | "Fleet: Generate contracts costs based on costs frequency" | Vehicle Contract | Every 1 day | The automation user, which bypasses access rights and record rules | Invokes the contract scheduler, whose four passes are specified in [workflows.md](workflows.md), procedure W-11 |

**The name is historical and the job no longer generates costs.** It moves contracts between the four states and raises renewal reminders; the costs are derived by the Fleet Analysis Report and are never stored. The name is reproduced exactly because an administrator looking for the job in the scheduled-jobs list must find it, and a rebuild that renames it must map the old name to the new one for that purpose.

The job is created in a mode that recreates it if it has been deleted, so an installation that removed it recovers it at the next update of the package.

### 8.2 The activity type

| External identifier | Name | Summary | Icon | Runs on |
|---|---|---|---|---|
| `mail_act_fleet_contract_to_renew` | "Contract to Renew" | "Contract to Renew" | A vehicle glyph | Vehicle Contract |

The Fleet package additionally registers this type with the platform's map of activity types raised by a scheduled job, declaring that the type runs on the Vehicle Contract and that its activities are **not** removed when the job runs again. That declaration is what makes the "already carries a reminder" test of the job's first pass stay true across runs, and therefore what gives each contract exactly one reminder.

An administration screen is offered for activity types bound to Vehicle Contracts, restricted to the technical-features flag; see [interfaces.md](interfaces.md).

### 8.3 The message subtype

| External identifier | Name | Description | Sequence | Runs on | Default |
|---|---|---|---|---|---|
| `mt_fleet_driver_updated` | "Changed Driver" | "Changed Driver" | 0 | Vehicle | Yes, so every follower of a vehicle is subscribed to it by default |

Every tracked change of the driver or of the future driver on a Vehicle is logged under this subtype instead of the platform's default tracking subtype. Every other tracked field of the Vehicle — the archive flag, the licence plate, the chassis number, the model, the registration date, the cancellation date, the first contract date, the status, the emissions figure, the catalogue value, the two plan-to-change markers, the driver Employee and the future driver Employee — is logged under the default subtype.

### 8.4 The activity plan template

| External identifier | Summary | Responsible kind | Belongs to |
|---|---|---|---|
| `offboarding_fleet` | "Take Back Fleet" | `fleet_manager`, labelled "Fleet Manager" | The offboarding activity plan of the people register |

Delivered by the Fleet History bridge. Scheduling the offboarding plan on a departing employee therefore creates one activity summarised "Take Back Fleet", assigned to the fleet manager of that employee's first vehicle, with the error and warning behaviour of rules FLT-016 and FLT-017.

### 8.5 Message templates

**No message template is shipped.** The send-mail assistant works without one, using a typed subject and body. A user may create a template with the assistant's own save operation, which produces a template named "Vehicle: Mass mail drivers" bound to the Vehicle; see [workflows.md](workflows.md), procedure W-15. Any number of further templates may be created by hand, and the assistant offers every template bound to the Vehicle.

### 8.6 Server action

| External identifier | Name | Runs on | Bound to | What it does |
|---|---|---|---|---|
| `action_fleet_vehicle_send_mail` | "Mail to Driver" | Vehicle | The Vehicle, on the list layout and the board layout | Invokes the vehicle's send-mail operation on the selected records, which opens the assistant |

## 9. The optional sample data set

The Fleet package and the Fleet History bridge each carry a sample data set that a user may choose to load when creating a database. It is not part of an ordinary installation. It is documented here for one reason: **two records that rules refer to by name exist only in it**, and the behaviour of an installation that has it therefore differs from one that does not.

### 9.1 Three further Vehicle Statuses

| External identifier | Name | Sequence | Referenced by a rule |
|---|---|---|---|
| `fleet_vehicle_state_ordered` | "Ordered" | 6 | No |
| `fleet_vehicle_state_reserve` | "Reserve" | 9 | No |
| `fleet_vehicle_state_waiting_list` | "Waiting List" | 10 | **Yes.** It is looked up by both guards of the driver assignment machine; see [state-machines.md](state-machines.md) section M4.3 and compatibility finding FLT-C02 |

### 9.2 Seventy-two further Fleet Service Types

Seventy-one of the service category and one of the contract category. One of them is referenced by a rule: `type_service_service_7`, "Repair and maintenance", is the default kind of a new Vehicle Service. On an installation without the sample data the default resolves to nothing and the field starts empty; see compatibility finding FLT-C01.

The seventeen general kinds of the fiscal and leasing family:

| External identifier | Name | Category |
|---|---|---|
| `type_service_service_1` | "Calculation Benefit In Kind" | `service` |
| `type_service_service_2` | "Depreciation and Interests" | `service` |
| `type_service_service_3` | "Tax roll" | `service` |
| `type_service_service_5` | "Summer tires" | `service` |
| `type_service_service_6` | "Snow tires" | `service` |
| `type_service_service_7` | "Repair and maintenance" | `service` |
| `type_service_service_8` | "Assistance" | `service` |
| `type_service_service_9` | "Replacement Vehicle" | `service` |
| `type_service_service_10` | "Management Fee" | `service` |
| `type_service_service_11` | "Rent (Excluding VAT)" | `service` |
| `type_service_service_12` | "Entry into service tax" | `service` |
| `type_service_service_13` | "Total expenses (Excluding VAT)" | `service` |
| `type_service_service_14` | "Residual value (Excluding VAT)" | `service` |
| `type_service_service_15` | "Options" | `service` |
| `type_service_service_16` | "Emissions" | `service` |
| `type_service_service_17` | "Touring Assistance" | `service` |
| `type_service_service_18` | "Residual value in %" | `service` |

The identifier `type_service_service_4` is absent from the sequence; nothing occupies it. Three of the names above contain the abbreviated form of value added tax and one contains the per-cent sign; all four are reproduced exactly as the labels a client displays.

The fifty-three workshop kinds:

| External identifier | Name | | External identifier | Name |
|---|---|---|---|---|
| `type_service_1` | "A/C Compressor Replacement" | | `type_service_28` | "Heater Hose Replacement" |
| `type_service_2` | "A/C Condenser Replacement" | | `type_service_29` | "Ignition Coil Replacement" |
| `type_service_3` | "A/C Diagnosis" | | `type_service_30` | "Intake Manifold Gasket Replacement" |
| `type_service_4` | "A/C Evaporator Replacement" | | `type_service_31` | "Oil Change" |
| `type_service_5` | "A/C Recharge" | | `type_service_32` | "Oil Pump Replacement" |
| `type_service_6` | "Air Filter Replacement" | | `type_service_33` | "Other Maintenance" |
| `type_service_7` | "Alternator Replacement", then "Ball Joint Replacement" | | `type_service_34` | "Oxygen Sensor Replacement" |
| `type_service_9` | "Battery Inspection" | | `type_service_35` | "Power Steering Hose Replacement" |
| `type_service_10` | "Battery Replacement" | | `type_service_36` | "Power Steering Pump Replacement" |
| `type_service_11` | "Brake Caliper Replacement" | | `type_service_37` | "Radiator Repair" |
| `type_service_12` | "Brake Inspection" | | `type_service_38` | "Resurface Rotors" |
| `type_service_13` | "Brake Pad(s) Replacement" | | `type_service_39` | "Rotate Tires" |
| `type_service_14` | "Car Wash" | | `type_service_40` | "Rotor Replacement" |
| `type_service_15` | "Catalytic Converter Replacement" | | `type_service_41` | "Spark Plug Replacement" |
| `type_service_16` | "Charging System Diagnosis" | | `type_service_42` | "Starter Replacement" |
| `type_service_17` | "Door Window Motor/Regulator Replacement" | | `type_service_43` | "Thermostat Replacement" |
| `type_service_18` | "Engine Belt Inspection" | | `type_service_44` | "Tie Rod End Replacement" |
| `type_service_19` | "Engine Coolant Replacement" | | `type_service_45` | "Tire Replacement" |
| `type_service_20` | "Engine/Drive Belt(s) Replacement" | | `type_service_46` | "Tire Service" |
| `type_service_21` | "Exhaust Manifold Replacement" | | `type_service_47` | "Transmission Filter Replacement" |
| `type_service_22` | "Fuel Injector Replacement" | | `type_service_48` | "Transmission Fluid Replacement" |
| `type_service_23` | "Fuel Pump Replacement" | | `type_service_49` | "Transmission Replacement" |
| `type_service_24` | "Head Gasket(s) Replacement" | | `type_service_50` | "Water Pump Replacement" |
| `type_service_25` | "Heater Blower Motor Replacement" | | `type_service_51` | "Wheel Alignment" |
| `type_service_26` | "Heater Control Valve Replacement" | | `type_service_52` | "Wheel Bearing Replacement" |
| `type_service_27` | "Heater Core Replacement" | | `type_service_53` | "Windshield Wiper(s) Replacement" |

Two further kinds complete the set: `type_contract_repairing`, named "Repairing", of the `contract` category; and `type_service_refueling`, named "Refueling", of the `service` category.

**A data defect in the sample set.** The external identifier `type_service_7` is declared twice, first for "Alternator Replacement" and then for "Ball Joint Replacement". Because the second declaration overwrites the first, an installation that loads the sample data ends up with a single record named "Ball Joint Replacement" and with no kind named "Alternator Replacement", and the sequence has a gap at `type_service_8`. The set therefore contains fifty-two workshop kinds rather than the fifty-three the numbering suggests. Recorded here as an **industry-standard default** for a rebuild: number the two separately.

### 9.3 Ten Vehicle Categories

| External identifier | Name | | External identifier | Name |
|---|---|---|---|---|
| `model_category_sedan` | "Sedan" | | `model_category_convertible` | "Convertible" |
| `model_category_estate` | "Estate" | | `model_category_mpv` | "MPU" |
| `model_category_compact` | "Compact" | | `model_category_bmx` | "BMX" |
| `model_category_suv` | "SUV" | | `model_category_vtt` | "VTT" |
| `model_category_coupe` | "Coupe" | | `model_category_city` | "City" |

The name of the seventh entry is reproduced exactly; the three letters do not spell the multi-purpose vehicle class its identifier names, and the discrepancy is in the shipped data.

### 9.4 What else the sample data contains

The Fleet package's sample data also contains Vehicle Models, Vehicles, Vehicle Contracts, Vehicle Services and Odometer Readings, and it changes two group memberships: it removes both fleet groups from the demonstration user, and it makes the default internal-user group imply the fleet administrator group so that every new user of a sample database can use the application. The Fleet History bridge's sample data links some of those vehicles to Employees. The Transport Dispatch bridge's sample data adds a "Transport" tag, two capacity-bearing categories — "Transport Truck" with a maximum weight of 44 000 and a maximum volume of 32, and "Pickup Van" with 7 000 and 15 — and the models and vehicles that use them.

None of that further sample content is referenced by any rule.

## 10. What an administrator configures, in order

1. Install the Fleet package. The accounting bridge and the people bridge install themselves if the accounting capability and the people register are already present.
2. Grant the fleet officer group to every user who must record vehicles, contracts, distances and assignments; grant the fleet administrator group to every user who must also configure models, categories, statuses, tags, service kinds and services, and who must see the reporting screens.
3. Set the contract expiry alert delay on the settings screen.
4. Review the four shipped statuses and add or rename to match the company's own vehicle process.
5. Add the Fleet Service Types the company uses, of both categories. The service-kind screens require the technical-features flag; a user without it configures them through the transport layer or has the flag granted temporarily.
6. Add Vehicle Categories, with capacities when the transport dispatch bridge is installed.
7. Add Vehicle Tags.
8. Add Vehicle Models under the shipped manufacturers, or add further manufacturers.
9. Declare the property definitions on the models that need them.
10. Confirm that the daily job is enabled.
