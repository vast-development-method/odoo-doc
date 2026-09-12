# Workflows

This file specifies the end-to-end operational procedures of the fleet domain. Each procedure is a numbered sequence of steps. Every step states what it reads, what it creates or changes, which named operation it invokes, and the conditions under which it fails. Failures name the rule of [business-rules.md](business-rules.md) that produces the refusal, and state transitions name the machine of [state-machines.md](state-machines.md) that governs them.

| Procedure | Title |
|---|---|
| [W-01](#w-01-prepare-the-fleet-configuration) | Prepare the fleet configuration |
| [W-02](#w-02-register-a-vehicle) | Register a vehicle |
| [W-03](#w-03-assign-a-driver-to-a-vehicle) | Assign a driver to a vehicle |
| [W-04](#w-04-queue-a-future-driver) | Queue a future driver |
| [W-05](#w-05-apply-the-queued-driver-change) | Apply the queued driver change |
| [W-06](#w-06-record-a-distance) | Record a distance |
| [W-07](#w-07-record-a-service-by-hand) | Record a service by hand |
| [W-08](#w-08-create-services-from-a-posted-vendor-bill) | Create services from a posted vendor bill |
| [W-09](#w-09-change-or-remove-the-vehicle-on-a-bill-line) | Change or remove the vehicle on a bill line |
| [W-10](#w-10-record-a-coverage-contract) | Record a coverage contract |
| [W-11](#w-11-run-the-daily-contract-job) | Run the daily contract job |
| [W-12](#w-12-renew-a-contract) | Renew a contract |
| [W-13](#w-13-archive-a-vehicle) | Archive a vehicle |
| [W-14](#w-14-send-messages-to-the-drivers-of-selected-vehicles) | Send messages to the drivers of selected vehicles |
| [W-15](#w-15-save-a-composed-message-as-a-template) | Save a composed message as a template |
| [W-16](#w-16-release-the-vehicles-of-a-leaving-employee) | Release the vehicles of a leaving employee |
| [W-17](#w-17-follow-an-employees-change-of-work-contact) | Follow an employee's change of work contact |
| [W-18](#w-18-schedule-an-activity-plan-that-routes-to-the-fleet-manager) | Schedule an activity plan that routes to the fleet manager |
| [W-19](#w-19-analyse-fleet-costs) | Analyse fleet costs |
| [W-20](#w-20-analyse-distance-travelled) | Analyse distance travelled |
| [W-21](#w-21-dispatch-a-batch-transfer-with-a-vehicle) | Dispatch a batch transfer with a vehicle |

---

## W-01: Prepare the fleet configuration

**Actor.** A user holding the fleet administrator group. The configuration menu is visible only to that group.

**Preconditions.** The fleet capability package is installed. Sixty-seven manufacturers and four vehicle statuses are already present, having been loaded with the package; see [configuration.md](configuration.md).

**Steps.**

1. **Review the manufacturers.** Open the manufacturer board. Each card shows the logo, the name and the number of active models. Add a manufacturer by giving a name and, optionally, a logo, which will become the picture of every vehicle of every model of that manufacturer. *Failure:* a manufacturer without a name is refused by rule FLT-021.
2. **Create the models.** For each machine the company buys or leases, create a Vehicle Model with a name and a manufacturer, choose the vehicle kind, and fill the physical attributes the company cares about — seating capacity, number of doors, colour, trailer hitch, transmission, drive kind, fuel kind, power or horsepower, taxable horsepower, range and range unit, emissions figure and emission standard, model year, electric assistance. *Failure:* a model without a name or without a manufacturer is refused by rule FLT-020.
3. **Define the property set, if any.** On the model, declare the user-defined properties every vehicle of that model will carry. The definition lives on the model and the values live on each vehicle.
4. **Create the categories.** Give each load or usage class a name and a position. With the transport dispatch bridge installed, also give the maximum weight and the maximum volume. *Failure:* a duplicate category name is refused by rule FLT-009.
5. **Review the vehicle statuses.** Rename, reorder, fold or add statuses so that the board columns match the company's own process. *Failure:* a duplicate status name is refused by rule FLT-010.
6. **Create the tags.** Give each tag a name and a colour from the palette. *Failure:* a duplicate tag name is refused by rule FLT-011.
7. **Review the service types.** Two contract-category types and one service-category type are shipped. Add the kinds of work and the kinds of coverage the company uses, each with its category. The service-type screens are visible only to a user holding the technical-features flag; see [interfaces.md](interfaces.md). *Failure:* a type without a name or without a category is refused by rule FLT-022.
8. **Set the contract expiry alert delay.** Open the settings screen and set the number of days before expiry at which the system should start warning. The default is thirty. The value is stored in the system parameter `hr_fleet.delay_alert_contract`.
9. **Check that the daily job is enabled.** The job named "Fleet: Generate contracts costs based on costs frequency" runs once a day as the automation user. It is what moves contracts between states and raises renewal reminders.

**Result.** The fleet is ready to receive vehicles.

---

## W-02: Register a vehicle

**Actor.** A user holding the fleet officer group or the fleet administrator group. The officer group is enough: officers may create, read, update and delete vehicles.

**Preconditions.** At least one Vehicle Model exists.

**Steps.**

1. **Open a new vehicle.** From the fleet list or board, start a new record. The board offers a shortened form asking only for the model, the licence plate and the tags; the full form asks for everything.
2. **Choose the model.** *Failure:* leaving the model empty refuses the save under rule FLT-001. Choosing the model immediately runs the fifteen effective copy rules of [entities.md](entities.md) section 1.6: the transmission, model year, electric assistance, colour, seating capacity, number of doors, trailer hitch, emissions figure, emission standard, fuel kind, power, horsepower, taxable horsepower, category and range unit are each taken from the model when the model's value is not empty. The manufacturer is mirrored from the model. The emission unit is derived from the range unit. The picture is mirrored from the model, which mirrors it from the manufacturer. The vehicle kind is mirrored from the model and decides which groups of fields the form shows.
3. **Type the licence plate.** The display name is recomputed at once as manufacturer, solidus, model name, solidus, plate. With no plate the last part reads "No Plate".
4. **Fill the identity block.** Chassis number, tags, category if it should differ from the model's, and the vehicle description on the note page.
5. **Fill the driver block.** Driver, future driver, assignment date and — in a multi-company database — the company. The driver choice is restricted to Contacts with no company or with the vehicle's company. Leaving the company empty makes the vehicle visible to all companies.
6. **Fill the vehicle block.** Order date, registration date, cancellation date, chassis number, the current distance and its unit, the fleet manager, the location, and the plan-to-change marker for the vehicle's kind.
7. **Fill the tax block.** Taxable horsepower, first contract date, catalogue value including value added tax, purchase value and residual value.
8. **Fill the model block.** Every physical attribute copied in step 2 may be overwritten here; overwriting does not change the model.
9. **Fill the properties.** Values for the property set the model defines.
10. **Save.** The following happen in order:
    a. The synchronisation rule of the people bridge, if installed, resolves the driver Employee from the driver Contact and the future driver Employee from the future driver Contact, each requiring exactly one matching Employee; see [business-rules.md](business-rules.md), rule FLT-046.
    b. The plan-to-change marking of transition T4.6 is evaluated. On an ordinary creation it does nothing, because the vehicle kind is not part of the supplied values; see [state-machines.md](state-machines.md) section M4.3.
    c. The record is written. The status defaults to "New Request", the registration date and the first contract date to today, and the company to the current company.
    d. When the supplied values named a driver, one Driver Assignment Log is created with this vehicle, that driver and a start date of today. No end date is set and no reminder activity is scheduled, because the vehicle had no previous driver.
    e. When the current distance was supplied and is not zero, one Odometer Reading is created with that value, today's date, this vehicle and this vehicle's driver.

**Result.** One Vehicle, optionally one Driver Assignment Log and optionally one Odometer Reading.

---

## W-03: Assign a driver to a vehicle

**Actor.** A fleet officer or administrator. A human resources officer cannot do this: their access to vehicles is read-only.

**Preconditions.** The vehicle exists. The person to be made driver exists as a Contact.

**Steps.**

1. Open the vehicle form and set the driver field to the new Contact, or edit the driver column from the vehicle list.
2. On save the following happen in order, all inside the same write:
   a. The people bridge, if installed, resolves the driver Employee: it searches for Employees whose work contact is the new driver, limited to two results, and sets the driver Employee only when exactly one is found. When none or two or more are found, the driver Employee is emptied.
   b. If the vehicle already had a **different** driver Employee, the previous driver Contact and the previous driver Employee's user contact are removed from the vehicle's follower list, so the outgoing driver stops receiving the vehicle's messages.
   c. Because the written driver differs from the current one, one Driver Assignment Log is created with this vehicle, the new driver and a start date of today.
   d. If the vehicle had a previous driver, one activity of the generic to-do kind is scheduled on the vehicle. Its responsible user is the vehicle's fleet manager when there is one and the acting user otherwise. Its note is "Specify the End date of %s", with the placeholder replaced by the name of the previous driver.
   e. The vehicle's driver field is written. Because it is a tracked field, a message is posted to the vehicle's thread under the "Changed Driver" subtype rather than under the default tracking subtype.
   f. Every Vehicle Service of the vehicle re-derives its driver, and every Odometer Reading of the vehicle that has no driver yet takes the new one. Readings that already name a driver keep it.
3. **Close the previous assignment.** Open the assignment history from the vehicle's "Drivers History" counter button and set the end date of the previous driver's entry. Nothing does this automatically except a registered departure; the activity of step 2d is the reminder.

**Failure conditions.**

- Writing a driver at the same time as a lower distance is refused before anything else happens, by rule FLT-002.
- Nothing in this procedure refuses on the driver itself.

**Result.** The vehicle has a new driver, one new assignment entry is open, and — when there was a previous driver — one activity asks somebody to close the previous entry.

---

## W-04: Queue a future driver

**Actor.** A fleet officer or administrator.

**Preconditions.** The vehicle exists. The incoming person exists as a Contact belonging to the vehicle's company or to no company.

**Steps.**

1. Open the vehicle and set the future driver field.
2. On save the following happen in order:
   a. The people bridge resolves the future driver Employee by the same one-match rule as the driver.
   b. The guard of transition T4.5 is evaluated. The vehicle qualifies unless the "Waiting List" status exists **and** the vehicle's status — the one being written when the status is part of the same write, the stored one otherwise — is that status or the "New Request" status.
   c. The vehicle kinds of the qualifying vehicles are collected.
   d. Every **other** vehicle whose driver is the incoming person and whose kind is one of those collected is found, grouped by kind. Cars among them have `plan_to_change_car` set to true; bicycles have `plan_to_change_bike` set to true. Both writes are tracked, so each of those vehicles receives a message in its own thread.
   e. The future driver is written on the vehicle. Because the field is tracked, a message is posted under the "Changed Driver" subtype.
3. The "Apply New Driver" button appears on the vehicle form, because a future driver is now set.

**Failure conditions.** A future driver that belongs to a different company than the vehicle is refused by the company consistency check, rule FLT-019.

**Result.** The vehicle is queued for a change of driver, and every vehicle of the same kind that the incoming person is giving up is marked as becoming free.

---

## W-05: Apply the queued driver change

**Actor.** A fleet officer or administrator.

**Preconditions.** The vehicle has a future driver.

**Steps.**

1. Press "Apply New Driver" on the vehicle form. The operation runs on the single vehicle being displayed.
2. Every vehicle whose driver is the future driver of this vehicle and whose kind equals this vehicle's kind is found. This includes vehicles the incoming person drives today and excludes vehicles of the other kind.
3. On all of those vehicles, in one write: the driver is emptied, `plan_to_change_car` is cleared and `plan_to_change_bike` is cleared. Because the driver is being *emptied*, no assignment entry is created and no reminder activity is scheduled; the assignment entries of those vehicles stay open until somebody closes them.
4. On the vehicle itself, both plan-to-change markers are cleared.
5. On the vehicle itself, the driver is set to the future driver. This is a non-empty driver write, so W-03 step 2 runs in full: the driver Employee is resolved, the previous driver is unsubscribed, one assignment entry is created dated today, and — when the vehicle already had a driver — the end-date reminder activity is scheduled.
6. On the vehicle itself, the future driver is emptied. The "Apply New Driver" button disappears.

**Failure conditions.** None. The operation refuses nothing.

**Result.** The incoming person now drives this vehicle; the vehicles they were driving of the same kind are free; one assignment entry is open on this vehicle.

---

## W-06: Record a distance

There are three routes to an Odometer Reading. All three produce the same kind of record; they differ in what they fill and in what they refuse.

### W-06a: Type the reading directly

1. Open the odometer list from the fleet menu and add a row, or open the reading form.
2. Choose the vehicle. A change-reaction rule immediately refreshes the unit shown beside the value from the vehicle's odometer unit, before the record is saved.
3. Type the value and the date. The date defaults to today.
4. Save. The rule that derives the driver runs: because the reading has no driver yet, it takes the vehicle's current driver. The display name is derived as the vehicle's name, a space, a solidus, a space and the date.

**Failure conditions.** A reading without a vehicle is refused by rule FLT-024. Nothing refuses a value lower than an earlier one on this route: the never-decrease guard belongs to the vehicle's distance field, not to the reading.

### W-06b: Write the distance on the vehicle

1. Open the vehicle form and type a new value into the "Last Odometer" field beside the unit.
2. On save, before anything else, the guard of rule FLT-002 compares the typed value against the vehicle's current derived distance — that is, against the greatest value among its existing readings. *Failure:* a lower value is refused with the message "The odometer value cannot be lower than the previous one." and nothing at all is written, not even the other fields of the same save.
3. When the guard passes and the typed value is not zero, one Odometer Reading is created with that value, today's date in the acting user's time zone, the vehicle and the vehicle's current driver.
4. When the typed value is zero, no reading is created. The field is derived, so it reverts to the greatest existing value on the next read.

### W-06c: Write the distance on a service

1. Open a Vehicle Service and type a value into its distance field.
2. *Failure:* an empty or zero value is refused with the message "Emptying the odometer value of a vehicle is not allowed.", under rule FLT-003. The one exception is a service being **created** with a zero distance: the creation routine removes the zero from the supplied values before it reaches the rule, so a blank distance on a new service is silently ignored rather than refused.
3. One Odometer Reading is created with the typed value, the service's date — or today when the service has no date — and the service's vehicle. The reading's driver is derived from the vehicle as usual.
4. The service's odometer link is set to the new reading.

**Compatibility finding.** Step 3 of route W-06c writes the link onto the whole selection being saved rather than onto the individual service. Saving several services at once through a multiple-record edit therefore leaves every one of them pointing at the reading created for the last of them. This is recorded as FLT-C14 in [business-rules.md](business-rules.md). A corrected behaviour would write the link onto each service individually.

**Result.** One Odometer Reading. The vehicle's derived distance rises to the new value when the new value is the greatest.

---

## W-07: Record a service by hand

**Actor.** A user holding the fleet administrator group. A fleet officer may **read** services but not create, change or delete them; see [configuration.md](configuration.md).

**Steps.**

1. Open the service list from the fleet menu, or press the service counter button on a vehicle form, which opens the list already filtered to that vehicle and defaulting the vehicle on new records.
2. Choose the service kind. It defaults to the shipped type identified as `fleet.type_service_service_7` when that record is present; on an ordinary installation it is not, so the field starts empty. *Failure:* leaving it empty refuses the save under rule FLT-004.
3. Choose the vehicle. *Failure:* leaving it empty refuses the save under rule FLT-005. Choosing it fills the model, the manufacturer and the fleet manager as stored mirrors, and derives the driver — from the vehicle's driver Employee's work contact when the people bridge is installed and that Employee exists, and from the vehicle's driver otherwise.
4. Type the description, the date, the cost, the vendor and the vendor reference. The date defaults to today; it decides which month the cost falls into in the Fleet Analysis Report.
5. Optionally type the distance at which the work was done; route W-06c then runs.
6. Type any notes.
7. Set the progress stage from the status bar: New, Running, Done or Cancelled. It defaults to New and may be changed at any time in any direction.
8. Save.

**Failure conditions.** Rules FLT-004 and FLT-005 as above. If the service is bound to a journal item — which cannot happen on this route, because a hand-created service has no such link — typing a cost would be refused by rule FLT-012.

**Result.** One Vehicle Service, optionally one Odometer Reading, and a discussion thread on which the fleet team can follow up.

---

## W-08: Create services from a posted vendor bill

**Actor.** Any user who posts a vendor bill. No fleet permission is required: the services are created by the system, not by the user.

**Preconditions.** The accounting bridge is installed. The shipped service type identified as `account_fleet.data_fleet_service_type_vendor_bill`, named "Vendor Bill" and of the service category, exists. When it does not, this whole procedure is skipped and posting behaves exactly as it does without the bridge.

**Steps.**

1. A user enters a vendor bill and, on one or more of its product lines, chooses a vehicle in the vehicle column. The column appears only on a vendor bill, a vendor credit note or a vendor receipt, and it is hidden by default; a user shows it from the optional-column menu. When the vehicle-required flag of a line is true — which the shipped rule never makes it — the column becomes mandatory on a bill or a credit note.
2. The user posts the bill. The entry is posted first, so that its number exists and so that the set of entries posted for the first time is known.
3. For every journal item of every entry posted in this call, the item is **skipped** when any of the following holds. All four conditions are checked, in this order:
   a. The item names no vehicle.
   b. The item already carries a Vehicle Service.
   c. The entry is not a vendor bill. A vendor credit note and a vendor receipt are therefore skipped even though they may carry a vehicle.
   d. The item's display kind is not a product line. Tax lines, payment-term lines, section lines and note lines are therefore skipped.
4. For each item that survives, one Vehicle Service is prepared with: the "Vendor Bill" service type; the item's vehicle; the bill's partner as the vendor; the item's label as the description; and the item itself as the journal item link.
5. All prepared services are created in one operation. Creating them runs the ordinary service rules: the vehicle's model, manufacturer and fleet manager are mirrored onto each; the driver is derived; the date defaults to today, **not** to the bill date; the progress stage defaults to New; the cost is derived from the journal item as the item's debit in the company currency.
6. On each new service, one message is posted to its thread whose body is "Service Vendor Bill: %s", with the placeholder replaced by a link to the journal entry.

**Failure conditions.** None. The procedure never refuses; it only skips.

**Consequences worth stating.**

- The service's **date** is today, the day of posting, not the invoice date. A bill dated last month posted today produces a service dated today, and the cost therefore lands in this month's column of the Fleet Analysis Report.
- The service's **cost** is the debit of the item, in the company currency. On a bill in another currency the cost differs from the line's untaxed subtotal; see [calculations.md](calculations.md) and [accounting-effects.md](accounting-effects.md).
- Posting a bill a second time — after resetting it to draft and posting again — does not create a second service, because the item already carries one.
- The service's cost cannot then be typed; rule FLT-012 refuses it.
- The service cannot be deleted; rule FLT-013 refuses it.

**Result.** One Vehicle Service per qualifying bill line, each bound to its journal item and each carrying a message that links back to the bill.

---

## W-09: Change or remove the vehicle on a bill line

**Actor.** A user editing a vendor bill.

**Preconditions.** The bill is in draft, or has been reset to draft.

**Steps.**

1. **To move the cost to another vehicle:** reset the bill to draft, change the vehicle on the line, and post again.
   a. Writing a non-empty vehicle does not delete anything. The existing service, which is bound to the line, re-derives its own vehicle from the line and follows it. From that moment the service belongs to the new vehicle and disappears from the old vehicle's list.
   b. Posting again skips the line, because it already carries a service.
2. **To remove the cost from every vehicle:** reset the bill to draft and clear the vehicle on the line.
   a. Before the write is applied, every Vehicle Service bound to that line is deleted. The deletion carries the bypass marker `ignore_linked_bill_constraint`, so the guard of rule FLT-013 does not fire.
   b. The line's vehicle becomes empty and the line no longer carries a service.
   c. Posting again skips the line, because it names no vehicle.
3. **To put the cost back on a vehicle:** reset the bill to draft, set the vehicle again, and post again. Because the line now carries no service, step 3 of W-08 lets it through and a **new** service is created. It is a different record from the one deleted in step 2; its date is the new posting day and its thread starts empty.
4. **To delete the line entirely:** deleting a journal item first deletes every service bound to it, again with the bypass marker, and then removes the item.

**Failure conditions.** None on this route. The bypass marker exists precisely so that these four paths never hit the deletion guard.

**Result.** The set of Vehicle Services stays consistent with the set of bill lines that name a vehicle.

---

## W-10: Record a coverage contract

**Actor.** A user holding the fleet officer group or the fleet administrator group. Officers may create, read, update and delete contracts.

**Steps.**

1. Open the contract list from the fleet menu — which opens filtered to running contracts — or press the contract counter button on a vehicle form, which opens the list filtered to that vehicle and defaulting the vehicle on new records.
2. Choose the vehicle. *Failure:* leaving it empty refuses the save under rule FLT-006. A vehicle of another company is refused by rule FLT-019. Choosing it fills the driver as a read-only mirror, and — when the contract is created from a vehicle form — the responsible user defaults to that vehicle's fleet manager.
3. Choose the coverage kind. The choice is restricted to Fleet Service Types whose category is `contract`.
4. Type the reference, choose the insurer, and pick the service types the agreement includes.
5. Set the start date, which defaults to today, and the expiration date, which defaults to exactly one year after today. *Failure:* the expiration date is mandatory on the form whenever the recurring frequency is not `no`, under rule FLT-008.
6. Choose the responsible user. This is the user the renewal reminder will be assigned to; a contract with no responsible user never receives one.
7. Type the one-off activation cost and its date, and the recurring cost and its frequency. The frequency defaults to Monthly and is mandatory, under rule FLT-007.
8. Type the terms and conditions.
9. Save. The contract name is derived as the coverage kind's name, a space and the vehicle's name — or as the vehicle's name alone when there is no kind — and remains editable. The state defaults to `open`.
10. **Because the dates were part of this write, the state is immediately re-evaluated** by transitions T1.5 to T1.7 of [state-machines.md](state-machines.md): a start date after today yields `futur`; a start date on or before today with no expiration date, or with an expiration date on or after today, yields `open`; anything else yields `expired`.
11. The vehicle re-derives `contract_count`, `contract_renewal_due_soon`, `contract_renewal_overdue` and `contract_state`.

**Result.** One Vehicle Contract in the state its dates imply, and a vehicle whose renewal flags have been refreshed.

---

## W-11: Run the daily contract job

**Actor.** The system. The job runs once a day as the automation user, which bypasses record rules and access rights.

**Steps.** The job performs four passes, in this order.

1. **Raise renewal reminders.**
   a. Read the alert delay from the system parameter `hr_fleet.delay_alert_contract`, defaulting to thirty when it is unset.
   b. Compute the cut-off as today plus that many days.
   c. Find every contract whose state is exactly `open`, whose expiration date is strictly earlier than the cut-off, and which has a responsible user.
   d. From those, keep only the contracts that do **not** already carry an activity of the renewal type.
   e. On each remaining contract, schedule one activity of the renewal type, with its deadline set to that contract's expiration date and its responsible user set to that contract's responsible user.
   Note that a contract whose expiration date is already in the past also satisfies the cut-off test, so an overdue running contract receives a reminder with a deadline in the past. That is intentional: the activity then shows as late.
2. **Expire what has lapsed.** Find every contract whose state is neither `expired` nor `closed` and whose expiration date is strictly earlier than today, and move them all to `expired` through transition T1.8.
3. **Push back what has not started.** Find every contract whose state is neither `futur` nor `closed` and whose start date is strictly later than today, and move them all to `futur` through transition T1.9. A contract just expired in pass 2 whose start date is in the future is caught here and pushed back; see the compatibility finding in [state-machines.md](state-machines.md) section M1.2.
4. **Start what is due.** Find every contract whose state is exactly `futur` and whose start date is earlier than or equal to today, and move them all to `open` through transition T1.10.

**Failure conditions.** None. Each pass is a search followed by a write; nothing refuses.

**Side effects on vehicles.** Every vehicle whose contracts changed state re-derives its renewal flags, its derived contract state and its contract counter the next time they are read.

**Result.** Every contract's state agrees with today's date, and every running contract that is about to lapse carries exactly one renewal reminder.

---

## W-12: Renew a contract

**Actor.** The responsible user named on the contract, who receives the renewal activity.

**Steps.**

1. The user opens their activity list, or the contract activity view, and finds the "Contract to Renew" activity. Its deadline is the contract's expiration date.
2. **Either** the user extends the existing agreement: they open the contract and set a later expiration date.
   a. The write re-evaluates the state through T1.5 to T1.7. A later expiration date with a start date on or before today yields `open`.
   b. Because a non-empty expiration date was written, the existing renewal activity is rescheduled to the new deadline rather than duplicated. If the responsible user was written in the same operation, the activity is reassigned at the same time.
   c. The user marks the activity as done.
3. **Or** the user records a replacement agreement: they create a new contract on the same vehicle with a start date on or after the old contract's expiration date, and then cancel the old one through `action_close`.
   a. The vehicle's overdue flag, computed from the contract with the latest expiry among the non-cancelled ones, now reads that new contract and turns off.
   b. The search-based overdue rule behaves the same way, because it excludes a vehicle that has any contract in state `open` or `futur` whose expiry is today or later.
4. **Or** the user lets the agreement lapse: they do nothing. The daily job moves the contract to `expired` on the day after its expiration date, and the vehicle's overdue flag turns on.

**Result.** Either a contract with a later expiry, or a replacement contract and a cancelled predecessor, or an expired contract and a flagged vehicle.

---

## W-13: Archive a vehicle

**Actor.** A fleet officer or administrator.

**Steps.**

1. Open the vehicle and choose Archive from the record's action menu.
2. The desktop client shows a confirmation dialogue whose body is exactly "Every service and contract of this vehicle will be considered as archived. Are you sure that you want to archive this record?" with a confirm choice and a cancel choice. Cancelling does nothing at all.
3. On confirm, the archive flag of the vehicle is cleared. Before the write reaches the database, two cascades run:
   a. Every Vehicle Contract of the vehicle is found — whatever its state and whatever its own archive flag — and its archive flag is cleared.
   b. Every Vehicle Service of the vehicle is found and its archive flag is cleared.
4. The vehicle disappears from the default vehicle list and is reachable through the "Archived" filter.
5. The vehicle's counters are re-derived. Because they compare archive flags, the archived vehicle now reports the number of its archived contracts that are not cancelled and the number of its archived services. The counter buttons pass a filter that shows archived records to the list they open, so pressing them from an archived vehicle finds the archived children.

**Restoring.** Choosing Restore clears nothing and raises no dialogue: the vehicle's archive flag is set, and its contracts and services stay archived. Each has to be restored on its own, from the contract list or the service list with the "Archived" filter applied.

**Contracts keep moving.** The daily job does not filter on the archive flag, so a running contract on an archived vehicle still expires on schedule.

**Result.** One archived vehicle, all of its contracts archived and all of its services archived.

---

## W-14: Send messages to the drivers of selected vehicles

**Actor.** A user holding the fleet administrator group. The assistant's access rights grant read, write and create to that group only, and no delete to anybody.

**Steps.**

1. Select one or more vehicles in the vehicle list or on the vehicle board and choose the bound operation "Mail to Driver" from the action menu. The operation is bound to the list layout and the board layout only; it does not appear on a single vehicle's form.
2. A dialogue opens with the selected vehicles already filled in and the author set to the acting user's Contact.
3. **Either** choose a stored message template. The choice is restricted to templates bound to the Vehicle. Choosing one copies the template's subject and body into the dialogue and **replaces** the dialogue's attachments with the template's attachments.
4. **Or** type a subject and a body directly. The subject is mandatory on the form.
5. Optionally add attachments.
6. Press Send. The following happen in order:
   a. Every driver of every selected vehicle is collected, and those without an electronic mail address are separated out.
   b. *Failure:* when that separated set is not empty, nothing is sent at all. A notification of the danger kind is shown whose message is "The following vehicle drivers are missing an email address: %s.", with the placeholder replaced by the names of those drivers joined by a comma and a space. The dialogue stays open so the reader can correct the Contacts and try again. This is rule FLT-018.
   c. When a template was chosen, the subject and the body are rendered once **per vehicle**, so placeholders such as the licence plate resolve differently in each copy. When no template was chosen, the typed subject and the typed body are used unchanged for every vehicle.
   d. For each vehicle in turn, one message is posted to that vehicle's discussion thread, with: the chosen author as the author; that vehicle's rendered body; the light notification layout; the message kind "comment"; that vehicle's driver as the only recipient; and that vehicle's rendered subject.
7. Each recipient receives one electronic mail per vehicle they drive. A person driving three of the selected vehicles receives three messages.

**Result.** One message per selected vehicle on that vehicle's own thread, delivered to that vehicle's driver.

---

## W-15: Save a composed message as a template

**Actor.** As W-14.

**Steps.**

1. In the send dialogue, compose a subject and a body, and optionally attach files.
2. Press "Save as new template".
3. One message template is created with: the name "Vehicle: Mass mail drivers"; the composed subject, or no subject when it is empty; the composed body, or no body when it is empty; the Vehicle as its target entity; and the use-default-recipient flag set, so that the template addresses the record's own recipients.
4. Every attachment of the dialogue that the **acting user** created is moved to belong to the new template. Attachments created by another user are left where they are.
5. All of the dialogue's attachments, moved or not, are added to the template's attachment list.
6. The dialogue's template field is set to the new template.
7. The new template's form opens in a further dialogue so the reader can rename it or refine it.

**Failure conditions.** None; the operation refuses nothing. A template saved with an empty subject is permitted.

**Result.** One reusable message template bound to the Vehicle, carrying the composed text and the attachments.

---

## W-16: Release the vehicles of a leaving employee

**Actor.** A user with permission to register a departure in the people register.

**Preconditions.** The people bridge is installed. The employees being processed drive one or more vehicles.

**Steps.**

1. Open the departure assistant on one or more employees. The assistant asks for a departure reason, a departure date, and — added by this domain — whether to release the company vehicle. That option defaults to set when the acting user is a member of the fleet officer group and clear otherwise.
2. Register the departure. The people register performs its own departure work first; see [human resources core](../human-resources-core/).
3. When the release option is set, the release procedure runs:
   a. Collect the drivers to release: the user contact of each departing employee, together with the work contact of each departing employee. The work contacts are read with elevated permissions, so the procedure works for a user who cannot normally read them.
   b. Find every Driver Assignment Log whose driver is one of those contacts **and** whose end date is either empty or later than the departure date.
   c. Write the departure date as the end date on all of them in one operation. An entry that already ended before the departure date is left alone, so historic assignments are not rewritten.
   d. Find every Vehicle whose driver is one of those contacts.
   e. Write an empty driver and an empty driver Employee on all of them in one operation. Because the driver being written is **empty**, no new assignment entry is created and no reminder activity is scheduled.
4. The released vehicles appear in the "Available" filter of the vehicle list, which looks for vehicles with no future driver and either no driver or a set plan-to-change marker of the matching kind.

**Failure conditions.** None. The release refuses nothing, and it silently does nothing when the employees drive no vehicles.

**Related.** The offboarding activity plan shipped by this domain contains a line summarised "Take Back Fleet" whose responsible user is resolved by the fleet-manager rule of W-18.

**Result.** Every assignment entry of the leaver is closed on the departure date, and every vehicle they drove is free.

---

## W-17: Follow an employee's change of work contact

**Actor.** A user editing an Employee in the people register.

**Preconditions.** The people bridge is installed.

**Steps.**

1. Before the change is applied, the old work contact of each employee being edited is remembered.
2. The people register applies the change, including its own synchronisation between the employee and the linked user.
3. Afterwards, for each employee whose work contact actually changed:
   a. Every vehicle naming that employee as driver Employee **or** as future driver Employee is found, with elevated permissions.
   b. Among them, those naming the employee as **driver Employee** have their driver Contact rewritten to the new work contact. That is a non-empty driver write, so W-03 step 2 runs: a new assignment entry opens dated today, and — because the vehicle already had a driver — the end-date reminder activity is scheduled.
   c. Among them, those naming the employee as **future driver Employee** have their future driver Contact rewritten to the new work contact. That is a non-empty future driver write, so W-04 step 2 runs and the plan-to-change marking is re-evaluated against the new contact.
4. When the employee's mobility card was changed in the same operation, every vehicle naming that employee as driver Employee re-derives its own mobility card.

**Failure conditions.** *Failure:* emptying the work contact of an employee that has vehicles is refused by rule FLT-014 with the message "Cannot remove address from employees with linked cars.". The check runs as a constraint on the work contact, so it fires whether the field was emptied directly or emptied as a consequence of unlinking a user.

**Result.** The fleet follows the people register: a person whose contact record is replaced keeps their vehicles.

---

## W-18: Schedule an activity plan that routes to the fleet manager

**Actor.** A user scheduling an activity plan on one or more Employees.

**Preconditions.** The people bridge is installed. An activity plan that runs on Employees contains at least one line whose responsible kind is `fleet_manager`.

**Steps.**

1. Open the activity scheduling assistant on one or more employees and choose the plan.
2. For each line whose responsible kind is `fleet_manager`, the resolution rule runs against each selected employee:
   a. The employee's **first** vehicle is taken from their vehicle collection. "First" means first in the collection's own order, which for the vehicle entity is licence plate ascending then registration date ascending.
   b. *Error:* when the employee has no vehicle, the line yields the error "Employee %s is not linked to a vehicle.", with the placeholder replaced by the employee's name. The assistant cannot be saved while an error is present; see rule FLT-016.
   c. *Warning:* when the vehicle has no fleet manager, the line yields the warning "The vehicle of employee %(employee)s is not linked to a fleet manager, assigning to you.", with the placeholder replaced by the employee's name, and the responsible user falls back to the acting user. A warning does not block the save; see rule FLT-017.
   d. Otherwise the responsible user is the vehicle's fleet manager.
3. When several employees are scheduled at once and they resolve to different responsible users, the assistant shows no single responsible user on the line; each generated activity still receives its own.
4. Save and schedule. One activity per employee per line is created, each with the responsible user resolved for that employee.

**Failure conditions.**

- Saving the plan template itself with the fleet-manager responsible kind on a plan that does not run on Employees is refused by rule FLT-015 with the message "Fleet Manager is limited to Employee plans.".
- Scheduling against an employee with no vehicle is refused as described above.

**Result.** One activity per employee, each assigned to the manager of the vehicle that employee drives.

---

## W-19: Analyse fleet costs

**Actor.** A user holding the fleet administrator group. The reporting menu and the analysis access rights are restricted to that group.

**Steps.**

1. Open Reporting, then Costs. The screen opens as a chart, with a period filter on the cost date already applied and set to the current year.
2. The derived set is produced as specified in [calculations.md](calculations.md). In outline: for every active vehicle, one row per calendar month per cost kind, where the service row carries the sum of the costs of the vehicle's active, non-cancelled services dated in that month, and the contract row carries the sum of the one-off activation costs dated in that month plus the daily, monthly and yearly recurring costs attributable to that month.
3. Switch between the chart and the cross-table layout. The cross-table puts the year of the cost date and the cost kind across the top and the vehicle down the side, and measures the cost.
4. Filter to services only or to contracts only, search by vehicle name or driver name, and group by vehicle or by driver.
5. Open a single row in a read-only form to see its vehicle, driver, fuel kind, month, cost and cost kind.

**Failure conditions.** None; the set is read-only and refuses nothing. A user of another company sees only the rows of their own companies, through the record rule on the set's company column.

**Result.** A monthly cost picture per vehicle, split between coverage and work.

---

## W-20: Analyse distance travelled

**Actor.** A user holding the fleet administrator group, for the reporting menu; any user who can read a vehicle, for the per-vehicle route.

**Steps.**

1. **From the menu:** open Reporting, then Odometers. The screen opens as a line chart of distance travelled, grouped by month and by vehicle category, restricted to active vehicles.
2. **From a vehicle:** press the chart button beside the distance field on the vehicle form. The same screen opens, restricted to that one vehicle and grouped by month only.
3. The derived set is produced by the fifteen-step interpolation specified in [calculations.md](calculations.md). In outline: readings are reduced to one per vehicle per day by keeping the greatest; a zero reading is inserted on the registration date when that date precedes every reading; every month between the first and the last is generated; months without a reading are filled by interpolating between the surrounding readings in proportion to elapsed days; and a distance that straddles a month boundary is split between the two months in proportion to the days on each side.
4. Change the grouping to vehicle, category, fuel kind or model.

**Failure conditions.** None.

**Result.** A timeline of distance travelled per month, complete even for months in which nobody wrote a reading down.

---

## W-21: Dispatch a batch transfer with a vehicle

**Actor.** A warehouse user planning deliveries. Requires the transport dispatch bridge.

**Preconditions.** The operation type has dispatch management switched on, which the warehouse does automatically for its outgoing and incoming types, for its packing type when it ships in three steps and for its picking type when it ships in two.

**Steps.**

1. Create a batch transfer and add the transfers it should carry. On creation the batch orders its transfers by the customer postal code of each, ascending, with transfers whose Contact has no postal code sorting first; the resulting position is written as each transfer's position in the batch.
2. The dispatch group appears on the batch form because the operation type publishes dispatch management. Fill it:
   a. **Dock.** The choice is restricted to locations beneath the operation type's dock set. The dock is proposed automatically when every transfer of the batch shares one source location and that location is in the allowed dock set.
   b. **Vehicle.** Choose an owned vehicle, or leave it empty to mean a third-party carrier.
   c. **Vehicle category.** It follows the chosen vehicle's category and may be set on its own when no owned vehicle is chosen.
   d. **Driver.** It follows the chosen vehicle's driver and may be overridden.
3. The batch shows the estimated shipping weight beside the category's maximum weight as a progress bar, and the same for volume. The two percentages are computed as specified in [calculations.md](calculations.md).
4. Setting the dock redirects the goods:
   a. When the batch's operation type is an internal or an incoming one, every move of every transfer of the batch has its destination location set to the dock.
   b. Otherwise — an outgoing operation type — every move has its **source** location set to the dock.
   c. Clearing the dock resets each transfer's moves: every move whose destination is not beneath the transfer's own destination location has its destination set back to that location.
5. Assigning a further transfer to the batch applies the same redirection to that transfer alone, or the same reset when the batch has no dock.
6. The end date of the dispatch window is set to one hour after the scheduled date whenever it is empty or earlier than the scheduled date.
7. Print the batch document. It carries, above the transfer table, the dock, the vehicle and the vehicle category when each is set, and it adds a position column to the table showing each transfer's place in the ordered batch.
8. Merging two batches carries the vehicle and the dock of the batch being merged into the result.

**Failure conditions.** None are added by this domain. The refusals of the batch itself belong to [inventory operations](../inventory-operations/).

**Result.** A batch transfer that names the vehicle, the driver, the class of vehicle and the bay, with its goods routed through that bay and its document printed in delivery order.
