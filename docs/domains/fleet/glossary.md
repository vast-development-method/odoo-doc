# Glossary

Every term used in the fleet domain, defined. Terms are in alphabetical order. A term whose definition is owned by another domain is defined here only to the depth this domain needs, with a link to the owner.

---

**Activation cost.** The one-off amount a Vehicle Contract charges at its start, stored on the contract as `amount` and labelled "Activation Cost" on the form. It is attributed to the calendar month of the contract's activation cost date in the Fleet Analysis Report. Distinct from the recurring cost.

**Activation cost date.** The date field of a Vehicle Contract, stored as `date`. It decides which month the activation cost falls into, and — for a contract with a yearly recurring frequency — which calendar month of each year carries the recurring cost. It is not the start date and not the expiration date.

**Active.** The state of a record whose archive flag is set, meaning it appears in every default list, board and picker. See *archived*.

**Activity.** A planned piece of work assigned to a user with a deadline, a kind and a summary. Two are raised by this domain: the renewal reminder on a Vehicle Contract, and the end-date reminder on a Vehicle. The mechanism is owned by [messaging and activities](../messaging-and-activities/).

**Activity plan.** A named set of activity lines scheduled together against a record. This domain adds one responsible kind to those lines, `fleet_manager`, and ships one line summarised "Take Back Fleet" in the offboarding plan. See [configuration.md](configuration.md).

**Aggregated licence plate.** A derived text on an Employee holding the plates of every company vehicle the employee drives, joined by single spaces, with the employee's private plate appended when both exist. Formula C-24 in [calculations.md](calculations.md).

**Alert delay.** The number of days before a Vehicle Contract's expiration date at which the system begins calling the renewal due soon and begins raising the renewal reminder. Configured on the settings screen and stored in the system parameter `hr_fleet.delay_alert_contract`. Default 30. Formula C-27.

**Analysis result set.** A read-only derived collection of rows rebuilt from stored records rather than written by a user. This domain owns two: the Fleet Analysis Report and the Fleet Odometer Analysis Report.

**Archived.** The state of a record whose archive flag is cleared. It is hidden from every default search and reachable through the "Archived" filter. Five entities of this domain carry an archive flag: Vehicle, Vehicle Contract, Vehicle Service, Vehicle Model and Vehicle Manufacturer. Archiving a Vehicle archives all of its contracts and services; restoring it restores neither. See [state-machines.md](state-machines.md), machine M5.

**Assignment date.** The date on a Vehicle from which the vehicle becomes available to its future driver, stored as `next_assignation_date`. An empty value means available immediately. It is informational: nothing acts on it.

**Bill count.** The number of distinct purchase journal entries, not cancelled, that carry at least one item naming a Vehicle. Derived, and zero for a user who cannot read accounting. Formula C-23.

**Board.** A card layout in which records are grouped into columns. The vehicle board groups by Vehicle Status by default; the contract board and the service board group by state and stage respectively.

**Capacity share.** The percentage of a Vehicle Category's maximum weight or maximum volume that a planned Batch Transfer load takes. Formula C-31. Added by the transport dispatch bridge.

**Carbon dioxide emissions.** The emissions figure of a Vehicle or a Vehicle Model, stored as `co2` on the vehicle and `default_co2` on the model, expressed in the emission unit. Labelled "CO₂ Emissions". No aggregation is offered in grouped views, because summing emissions figures is meaningless.

**Catalogue value.** The list price of a Vehicle including value added tax, stored as `car_value`. A record of fact; it creates no ledger entry.

**Chassis number.** The unique number stamped on a Vehicle, stored as `vin_sn`. It is the vehicle identification number for a car and the serial number for another machine. Optional, tracked, and not carried into a duplicate. No uniqueness is enforced; see compatibility finding FLT-C21.

**Company currency.** The currency of the company that owns a record, reached through the record's company. Every monetary amount of this domain is expressed in it, and this domain performs no conversion.

**Compatibility finding.** An observed behaviour that looks like a defect, recorded as observed and accompanied by a statement of what a corrected behaviour would be. Twenty-five are recorded, `FLT-C01` to `FLT-C25`, in [business-rules.md](business-rules.md).

**Contract.** See *Vehicle Contract*.

**Contract counter.** The number of Vehicle Contracts of a Vehicle that are not cancelled and whose archive flag matches the vehicle's own. Formula C-12.

**Cost kind.** The classification of a row of the Fleet Analysis Report as either `contract` or `service`.

**Coverage kind.** The Fleet Service Type chosen as the kind of a Vehicle Contract, stored as `cost_subtype_id`. Restricted to types of the `contract` category.

**Derived contract state.** The state of the Vehicle Contract with the greatest expiration date among a Vehicle's contracts that have an expiration date and are not cancelled, mirrored onto the vehicle as `contract_state`. Its labels differ from the contract's own: "Incoming", "In Progress", "Expired", "Closed". When no contract qualifies it holds the empty text rather than an empty value. Formula C-16.

**Derived distance.** The greatest value among a Vehicle's Odometer Readings, or zero when it has none, offered on the vehicle as `odometer` and labelled "Last Odometer". Reading it takes the greatest, not the latest; writing it creates a new reading. Formula C-09 and compatibility finding FLT-C17.

**Dispatch management.** A marker on an Operation Type, added by the transport dispatch bridge, that makes the dispatch fields appear on batch transfers of that type and adds a transport section to the type's card menu. Switched on automatically by a warehouse for its outgoing, incoming and intermediate types.

**Dock.** An internal warehouse location that serves as a loading or unloading bay. A Batch Transfer may name one, which redirects the source or destination of its moves. Added by the transport dispatch bridge.

**Driver.** The Contact currently driving a Vehicle, stored as `driver_id`. Tracked under the "Changed Driver" subtype. Setting it opens a Driver Assignment Log and, when the vehicle already had a driver, schedules the end-date reminder.

**Driver Assignment Log.** The record of one period during which one Contact drove one Vehicle: a vehicle, a driver, a start date and an end date. Opened automatically on every non-empty driver write; closed by hand or by the departure procedure. Transport name `fleet.vehicle.assignation.log`.

**Driver Employee.** The Employee behind a Vehicle's driver Contact, resolved per company, stored as `driver_employee_id`. Added by the people bridge. Writing it rewrites the driver Contact and writing the driver Contact rewrites it, in both directions.

**Due soon.** The condition in which a Vehicle's latest contract expiry is in the future but closer than the alert delay. Reported by the flag `contract_renewal_due_soon` and searchable. The read form and the search form differ; see compatibility finding FLT-C16.

**Electric assistance.** A marker on a bicycle Vehicle or Vehicle Model saying that a motor assists the rider.

**Emission standard.** Free text on a Vehicle or Vehicle Model naming the regulatory test procedure under which the emissions figure was measured.

**Emission unit.** The unit of the emissions figure, either grammes per kilometre or grammes per mile, stored as `g/km` or `g/mi`. Derived from the range unit and not independently settable. Formula C-08.

**End-date reminder.** The activity of the generic to-do kind scheduled on a Vehicle when its driver is replaced, whose note is "Specify the End date of %s" with the previous driver's name, assigned to the fleet manager or to the acting user. It is the only prompt that asks a human to close the previous Driver Assignment Log.

**Expiration date.** The last day of coverage of a Vehicle Contract. Defaults to one year after the day the contract form was opened. Writing it re-evaluates the contract's state and reschedules the renewal reminder.

**Expires today.** A derived marker on a Vehicle Contract, true exactly on the expiry day of a running or expired contract. Used with days left to distinguish "expires today" from "already overdue". Formula C-14.

**Fleet administrator.** A member of the group named "Administrator" within the fleet privilege. Holds every fleet permission, implies the fleet officer group, and is the only group that may create and change Vehicle Services, configuration records and the reporting screens.

**Fleet Analysis Report.** The read-only monthly cost result set: one row per vehicle, per calendar month, per cost kind, carrying the total of that vehicle's service costs or contract costs in that month. Transport name `fleet.vehicle.cost.report`. Derivation C-28 and C-29.

**Fleet manager.** The user answerable for a Vehicle, stored as `manager_id`. Restricted to non-shared users of the vehicle's company who are members of the fleet officer group. Receives the end-date reminder and is the responsible user the fleet-manager activity plan kind resolves to.

**Fleet officer.** A member of the group named "Officer: Manage all vehicles" within the fleet privilege. May create, read, update and delete Vehicles, Vehicle Contracts, Odometer Readings and Driver Assignment Logs; may only read everything else.

**Fleet Odometer Analysis Report.** The read-only monthly distance result set: one row per vehicle per calendar month carrying the distance travelled and a running total, interpolated across months with no reading. Transport name `fleet.vehicle.odometer.report`. Derivation C-30.

**Fleet Service Type.** A named kind of work or coverage, carrying a category of either `contract` or `service`. One list serves both purposes. Transport name `fleet.service.type`.

**Folded.** A marker on a Vehicle Status saying that the board should show its column collapsed to a narrow strip.

**Frame kind.** The shape of a bicycle Vehicle's frame: `diamant`, `trapez` or `wave`.

**Frame size.** The size of a bicycle Vehicle's frame, in centimetres.

**Fuel kind.** The energy source of a Vehicle or Vehicle Model, one of nine stored values: `diesel`, `gasoline`, `full_hybrid`, `plug_in_hybrid_diesel`, `plug_in_hybrid_gasoline`, `cng` for compressed natural gas, `lpg` for liquefied petroleum gas, `hydrogen` and `electric`.

**Future driver.** The Contact queued to become a Vehicle's driver, stored as `future_driver_id`. Setting it marks the vehicles of the same kind that person currently drives as planned for change. Promoted to driver by the apply operation.

**Future driver Employee.** The Employee behind a Vehicle's future driver Contact, resolved by the same rule as the driver Employee. Added by the people bridge.

**Horsepower.** Engine power expressed in horsepower, stored as `horsepower`, shown only when the power unit is `horsepower`.

**Journal item.** One line of a journal entry. This domain adds a vehicle reference to it, and a posted vendor bill's product line carrying a vehicle produces a Vehicle Service. Owned by [general ledger](../general-ledger/).

**Licence plate.** The registration plate of a Vehicle, stored as `license_plate`. Part of the vehicle's display name, part of its default ordering, and the analytic name a localization uses. No uniqueness is enforced.

**Location.** Free text on a Vehicle saying where it is kept, for example a garage name. It is not a reference to a warehouse location.

**Mobility card.** A fuel or mobility card reference issued to an Employee, mirrored onto every Vehicle that employee drives. Added by the people bridge. Formula C-26.

**Model.** See *Vehicle Model*.

**Model year.** A year chosen from a list running from `1970` to the current calendar year inclusive, on both the Vehicle and the Vehicle Model. The list grows by one entry every first of January.

**Odometer Reading.** One dated distance measurement for one Vehicle, carrying a value, a date, a unit mirrored from the vehicle and a driver. Transport name `fleet.vehicle.odometer`. Created in three ways: typed directly, written through the vehicle's distance field, or written through a service's distance field.

**Odometer unit.** The unit of a Vehicle's distances, either `kilometers` labelled "km" or `miles` labelled "mi". Mirrored read-only onto every reading and every service of the vehicle.

**Overdue.** The condition in which a Vehicle's latest contract expiry is already past. Reported by the flag `contract_renewal_overdue` and searchable. The read form and the search form differ; see compatibility finding FLT-C16.

**Plan-to-change marker.** Either of the two markers on a Vehicle — `plan_to_change_car` for a car and `plan_to_change_bike` for a bicycle — saying that the vehicle's current driver has been queued to receive a different vehicle of the same kind, so that this one will shortly be free. Labelled "Make Vehicle Available" on the form.

**Power.** Engine power expressed in kilowatts, stored as `power`, shown only when the power unit is `power`.

**Power unit.** Which of the two power fields a Vehicle or Vehicle Model shows: `power` labelled "kW" or `horsepower`. Declared in the model-to-vehicle copy map but never actually copied; see compatibility finding FLT-C04.

**Purchase value.** The amount actually paid for a Vehicle, stored as `net_car_value`. A record of fact.

**Range.** The distance a Vehicle or Vehicle Model can travel on one full tank or charge, stored as `vehicle_range`, expressed in the range unit. Declared in the copy map but never actually copied; see compatibility finding FLT-C04.

**Range unit.** The unit of the range, either `km` or `mi`. On the Vehicle it is copied from the model; on both entities it drives the emission unit.

**Recurring cost.** The amount a Vehicle Contract charges once per period of its recurring frequency, stored as `cost_generated`. Never posted; it is spread across months by the Fleet Analysis Report.

**Recurring frequency.** How often a Vehicle Contract charges its recurring cost: `no`, `daily`, `weekly`, `monthly` or `yearly`. Required, default `monthly`. The weekly value is accounted for nowhere in the cost analysis; see compatibility finding FLT-C09.

**Registration date.** The day a Vehicle was registered, stored as `acquisition_date`. Defaults to today, is part of the default ordering, and is the starting month of both analysis result sets.

**Renewal reminder.** The activity of the "Contract to Renew" kind raised by the daily job on every running Vehicle Contract that has a responsible user and expires within the alert delay, with its deadline set to the contract's expiration date. Exactly one exists per contract at a time.

**Residual value.** The value a Vehicle is expected to hold at the end of the holding period, stored as `residual_value`. A record of fact.

**Responsible user.** The user answerable for renewing a Vehicle Contract, stored as `user_id`, and the user the renewal reminder is assigned to. Defaults to the fleet manager of the vehicle the contract is created from.

**Service.** See *Vehicle Service*.

**Service activity indicator.** A derived value on a Vehicle reading `none`, `overdue` or `today`, computed from the activity states of the vehicle's services and deciding which of three service counter buttons the form shows. Formula C-19.

**Service counter.** The number of Vehicle Services of a Vehicle whose archive flag matches the vehicle's own. Formula C-12.

**Service kind.** The Fleet Service Type chosen as the kind of a Vehicle Service, stored as `service_type_id`. Required, and not restricted to the `service` category; see compatibility finding FLT-C06.

**Stage.** The progress of a Vehicle Service: `new`, `running`, `done` or `cancelled`. Fully connected: any stage may follow any other, and no guard refuses. Machine M2.

**Start date.** The first day of coverage of a Vehicle Contract. Defaults to today. Writing it re-evaluates the contract's state.

**Status.** See *Vehicle Status*.

**System parameter.** A named value stored once per database and read by rules. This domain uses one: `hr_fleet.delay_alert_contract`, the alert delay. See [../../runtime/configuration-parameters.md](../../runtime/configuration-parameters.md).

**Tag.** See *Vehicle Tag*.

**Taxable horsepower.** A horsepower figure used by some jurisdictions to levy a road tax, stored as `horsepower_tax` and displayed as a monetary amount on the tax page of the vehicle form.

**Trailer hitch.** A marker on a Vehicle or Vehicle Model saying that a towing device is fitted. Because false counts as empty in the copy rule, the model can only ever turn it on, never off.

**Transport dispatch bridge.** The capability package that gives Vehicle Categories a maximum weight and volume, lets a Batch Transfer name a vehicle, a category, a driver and a dock, and adds dispatch management to Operation Types.

**Vehicle.** One physical machine — a car or a bicycle — that the company owns, leases, rents or lends. The centre of the domain. Transport name `fleet.vehicle`.

**Vehicle Category.** A load or usage class of Vehicle Models and Vehicles, carrying a name, a position and — with the transport dispatch bridge — a maximum weight and a maximum volume. Names are unique. Transport name `fleet.vehicle.model.category`.

**Vehicle Contract.** A coverage agreement on one Vehicle: a lease, an insurance, a maintenance agreement. Carries a coverage kind, dates, an activation cost, a recurring cost with a frequency, an insurer, a reference, the service kinds it includes and a four-state lifecycle. Transport name `fleet.vehicle.log.contract`.

**Vehicle kind.** Whether a Vehicle Model, and through it a Vehicle, is a `car` or a `bike`. Decides which groups of fields the form shows and which plan-to-change marker applies.

**Vehicle Manufacturer.** The maker of a Vehicle Model, carrying a name, a logo and a count of its active models. Sixty-seven are shipped. Transport name `fleet.vehicle.model.brand`.

**Vehicle Model.** The template describing a commercially named machine: a manufacturer, a model name and the physical attributes every unit of that model shares, fifteen of which are copied onto each vehicle. Also the home of the property definition a family of vehicles carries. Transport name `fleet.vehicle.model`.

**Vehicle Service.** One piece of work carried out on a Vehicle: its kind, its date, its cost, its vendor, its reference, the distance at the time, four progress stages and a discussion thread. Typed by hand or created automatically from a posted vendor bill line. Transport name `fleet.vehicle.log.services`.

**Vehicle Status.** One column of the vehicle pipeline board, an ordinary configuration record carrying a name, a position and a folded marker. Four are shipped and any number may be added. Names are unique. Transport name `fleet.vehicle.state`.

**Vehicle Tag.** A free label attached to any number of Vehicles, carrying a name and a colour index. Names are unique. Transport name `fleet.vehicle.tag`.

**Vendor.** On a Vehicle Service, the Contact that carried out the work. On a Vehicle Contract the equivalent field is the insurer. On a Vehicle Model, the collection of suppliers the model can be bought from.

**Vendor bill service kind.** The shipped Fleet Service Type named "Vendor Bill" of the `service` category, given to every Vehicle Service created from a posted vendor bill. When it is absent the whole creation routine is skipped.

**Vendor reference.** The reference a vendor put on their own document, recorded on a Vehicle Service as `inv_ref`, whose full name is invoice reference.

**Waiting list status.** A Vehicle Status named "Waiting List" that two guards of the driver assignment machine look up by name. It exists only in the sample data set, and its absence changes the behaviour of both guards; see compatibility finding FLT-C02.

**Work contact.** The Contact through which an Employee is reached. The link from a Vehicle to an Employee runs through it, and it may not be emptied while vehicles name the employee as driver.
