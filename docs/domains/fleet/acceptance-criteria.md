# Acceptance criteria

Every scenario below is stated in Given, When and Then form with concrete records, concrete inputs and exact resulting records, amounts, dates and states. A rebuild that satisfies all of them behaves like the specified system on every path this domain owns.

## The fixture

Unless a scenario says otherwise, the following records exist and the following conditions hold. Amounts are in the currency of Company One unless stated. "Today" is **Saturday 12 September 2026** and the acting user's time zone is the same as the server's, so the two notions of today coincide except where a scenario separates them.

| Record | Value |
|---|---|
| Company One | The current company of the acting user. Its currency is the company currency. |
| Company Two | A second company the acting user is **not** allowed to operate in, unless a scenario says otherwise. |
| Manufacturer AUDI | A Vehicle Manufacturer named "Audi", shipped, with a logo. |
| Manufacturer NAKA | A Vehicle Manufacturer named "Nakamura", created for these scenarios, with no logo. |
| Model A3 | A Vehicle Model named "A3", manufacturer AUDI, vehicle kind `car`, transmission `automatic`, model year `2024`, colour "Grey", seating capacity 5, number of doors 5, trailer hitch false, fuel kind `diesel`, power 110, horsepower 150, taxable horsepower 11, emissions figure 120, emission standard "Euro 6", range 900, range unit `km`, category CAT-SEDAN. |
| Model XV | A Vehicle Model named "Crossover xv", manufacturer NAKA, vehicle kind `bike`, electric assistance true, and no other attribute filled. |
| CAT-SEDAN | A Vehicle Category named "Sedan", sequence 10. |
| CAT-TRUCK | A Vehicle Category named "Transport Truck", sequence 1, maximum weight 44 000, maximum volume 32. Present only when the transport dispatch bridge is installed. |
| Contact P | A Contact named "Marc Demo" with the electronic mail address recorded. |
| Contact Q | A Contact named "Joel Willis" with the electronic mail address recorded. |
| Contact R | A Contact named "Abigail Peterson" with **no** electronic mail address. |
| User FM | A user named "Fleet Manager One", a member of the fleet officer group, belonging to Company One. |
| User OFF | A user who is a member of the fleet officer group only. |
| User ADM | A user who is a member of the fleet administrator group. |
| User PERS | A user who is a member of the human resources officer group and of neither fleet group. |
| Employee E1 | An Employee of Company One whose work contact is Contact P, mobility card "MC-4417". |
| Alert delay | The system parameter holds 30. |

---

# Part one: the vehicle register

## Scenario 1: create a vehicle from a model and copy fifteen attributes

**Given** Model A3 as described in the fixture, and no vehicle exists.

**When** User OFF creates a Vehicle with the model set to A3 and the licence plate set to "PAE 326", and saves.

**Then** exactly one Vehicle exists, with:

| Field | Value | Why |
|---|---|---|
| `name` | `Audi/A3/PAE 326` | Formula C-01 |
| `brand_id` | AUDI | Mirrored from the model |
| `vehicle_type` | `car` | Mirrored from the model |
| `transmission` | `automatic` | Copied |
| `model_year` | `2024` | Copied |
| `color` | "Grey" | Copied |
| `seats` | 5 | Copied |
| `doors` | 5 | Copied |
| `trailer_hook` | false | **Not** copied, because false counts as empty |
| `fuel_type` | `diesel` | Copied |
| `power` | 110 | Copied |
| `horsepower` | 150 | Copied |
| `horsepower_tax` | 11 | Copied |
| `co2` | 120 | Copied |
| `co2_standard` | "Euro 6" | Copied |
| `category_id` | CAT-SEDAN | Copied |
| `range_unit` | `km` | Copied |
| `co2_emission_unit` | `g/km` | Derived from the copied range unit |
| `vehicle_range` | 0 | **Not** copied; the rule does not exist |
| `power_unit` | `power` | **Not** copied; it holds its own default |
| `electric_assistance` | false | Not copied; the model's value is false |
| `state_id` | The status named "New Request" | Default |
| `acquisition_date` | 2026-09-12 | Default |
| `contract_date_start` | 2026-09-12 | Default |
| `company_id` | Company One | Default |
| `active` | true | Default |
| `odometer_unit` | `kilometers` | Default |

**And** no Driver Assignment Log exists, because no driver was supplied.
**And** no Odometer Reading exists.

## Scenario 2: a vehicle with no licence plate

**Given** Model A3.

**When** User OFF creates a Vehicle with the model A3 and no plate.

**Then** the vehicle's name is `Audi/A3/No Plate`.

## Scenario 3: a vehicle may not be created without a model

**Given** nothing further.

**When** User OFF creates a Vehicle with the plate "XYZ 001" and no model.

**Then** the save is refused by rule FLT-001 with the platform's required-field message naming "Model", and no Vehicle is created.

## Scenario 4: changing the model re-copies the attributes but does not clear them

**Given** the vehicle of scenario 1, whose trailer hitch has since been set to true by hand and whose seating capacity has been set to 4 by hand.

**When** User OFF changes the model from A3 to XV, whose only filled attribute is electric assistance.

**Then** `electric_assistance` becomes true, `vehicle_type` becomes `bike`, `brand_id` becomes NAKA, and `name` becomes `Nakamura/Crossover xv/PAE 326`.
**And** `trailer_hook` stays true, `seats` stays 4, `transmission` stays `automatic`, `fuel_type` stays `diesel`, `co2` stays 120, and `category_id` stays CAT-SEDAN — every one of them because the new model's value for the corresponding field is empty and an empty model value never overwrites.

## Scenario 5: the emission unit follows the range unit and the figure is not converted

**Given** the vehicle of scenario 1, whose range unit is `km`, emission unit `g/km` and emissions figure 120.

**When** User OFF changes the range unit to `mi`.

**Then** the emission unit becomes `g/mi` and the emissions figure stays exactly 120. The vehicle now reads as 120 grammes per mile.

## Scenario 6: a category name must be unique

**Given** CAT-SEDAN named "Sedan".

**When** User ADM creates a second Vehicle Category named "Sedan".

**Then** the save is refused with the message "Category name must be unique" and no second category exists.

## Scenario 7: a status name must be unique

**Given** the shipped status named "New Request".

**When** User ADM, holding the technical-features flag, creates a Vehicle Status named "New Request".

**Then** the save is refused with the message "State name already exists".

## Scenario 8: a tag name must be unique

**Given** a Vehicle Tag named "Pool".

**When** User ADM creates a second Vehicle Tag named "Pool".

**Then** the save is refused with the message "Tag name already exists!".

## Scenario 9: deleting a status empties the vehicles that used it

**Given** a Vehicle Status named "Reserve" and two vehicles whose status is "Reserve".

**When** User ADM deletes that status.

**Then** the status no longer exists, the deletion is not refused, and both vehicles have an empty status.
**And** neither vehicle appears in any column of the vehicle board.

## Scenario 10: the manufacturer's model count follows the archive flag

**Given** manufacturer AUDI with three active models and one archived model, so its model count reads 3.

**When** User ADM archives one of the three active models.

**Then** the manufacturer's model count reads 2.

**And when** User ADM restores it, the count reads 3 again.

## Scenario 11: the model's vehicle count excludes archived vehicles

**Given** Model A3 with nine vehicles, of which one is archived.

**Then** the model's vehicle count reads 8.

**And** searching Vehicle Models for a vehicle count other than zero returns Model A3.

---

# Part two: driver assignment

## Scenario 12: assigning the first driver opens an assignment entry and schedules nothing

**Given** the vehicle of scenario 1 with no driver.

**When** User OFF sets the driver to Contact P and saves.

**Then** one Driver Assignment Log exists with vehicle = this vehicle, driver = Contact P, start date = 2026-09-12 and no end date.
**And** no activity is scheduled on the vehicle.
**And** a tracking message is posted to the vehicle's thread under the "Changed Driver" subtype.
**And** the vehicle's driver history count reads 1.

## Scenario 13: replacing a driver opens a second entry and schedules a reminder

**Given** the vehicle of scenario 12, whose driver is Contact P and whose fleet manager is User FM.

**When** User OFF sets the driver to Contact Q and saves.

**Then** a second Driver Assignment Log exists with driver = Contact Q, start date = 2026-09-12 and no end date.
**And** the first entry still has no end date.
**And** one activity of the generic to-do kind is scheduled on the vehicle, assigned to User FM, whose note is exactly "Specify the End date of Marc Demo".
**And** the vehicle's driver history count reads 2.

## Scenario 14: replacing a driver on a vehicle with no fleet manager

**Given** the vehicle of scenario 12 with no fleet manager, driven by Contact P, and User OFF acting.

**When** User OFF sets the driver to Contact Q.

**Then** the reminder activity is assigned to **User OFF**.

## Scenario 15: clearing the driver opens nothing and closes nothing

**Given** the vehicle of scenario 12, driven by Contact P, with one open assignment entry.

**When** User OFF clears the driver.

**Then** the vehicle has no driver.
**And** still exactly one Driver Assignment Log exists, still with no end date.
**And** no activity is scheduled.

## Scenario 16: queueing a future driver marks the other vehicles of the same kind

**Given** four vehicles: CAR1 of model A3 driven by Contact P; CAR2 of model A3 driven by Contact Q; BIKE1 of model XV driven by Contact P; BIKE2 of model XV driven by Contact Q. All four plan-to-change markers are clear. The "Waiting List" status does not exist.

**When** User OFF writes Contact Q as the future driver of CAR1 and of BIKE1 in one operation.

**Then** CAR1's future driver is Contact Q and BIKE1's future driver is Contact Q.
**And** CAR1's plan-to-change-car marker is still false, and BIKE1's plan-to-change-bicycle marker is still false — the marking is applied to the *other* vehicles, not to these.
**And** CAR2's plan-to-change-car marker is **true**, because Contact Q drives it and it is a car.
**And** BIKE2's plan-to-change-bicycle marker is **true**.
**And** CAR2's future driver and BIKE2's future driver are both still empty.

## Scenario 17: applying the queued driver change

**Given** the four vehicles of scenario 16 after that operation.

**When** User OFF presses "Apply New Driver" on CAR1.

**Then** CAR2's driver becomes empty, and both of its plan-to-change markers become false.
**And** CAR1's two plan-to-change markers become false.
**And** CAR1's driver becomes Contact Q and its future driver becomes empty.
**And** one new Driver Assignment Log exists on CAR1 with driver Contact Q, start date 2026-09-12 and no end date.
**And** one activity of the generic to-do kind is scheduled on CAR1 with the note "Specify the End date of Marc Demo", because CAR1 had a driver before the change.
**And** BIKE1 and BIKE2 are untouched, because the operation matches only vehicles of the same kind as the one it is invoked on.

## Scenario 18: the "Available" filter after a release

**Given** CAR2 from scenario 17, whose driver is empty, whose future driver is empty and whose plan-to-change markers are false.

**When** a reader applies the "Available" filter to the vehicle list.

**Then** CAR2 is returned, because it has no future driver and no driver.

**And given** CAR1 from scenario 17, whose driver is Contact Q and whose markers are false, CAR1 is **not** returned.

---

# Part three: distance

## Scenario 19: writing a distance on a vehicle creates a reading

**Given** the vehicle of scenario 12, driven by Contact P, with no odometer reading.

**When** User OFF types 12 500 into the vehicle's distance field and saves.

**Then** one Odometer Reading exists with value 12 500, date 2026-09-12, vehicle = this vehicle, driver = Contact P and unit `kilometers`.
**And** the vehicle's distance field reads 12 500.
**And** the reading's name is `Audi/A3/PAE 326 / 2026-09-12`.
**And** the vehicle's odometer reading count reads 1.

## Scenario 20: a lower distance is refused

**Given** the vehicle of scenario 19, whose derived distance is 12 500.

**When** User OFF types 12 000 into the distance field and, in the same save, also changes the location to "Garage B".

**Then** the save is refused with the message "The odometer value cannot be lower than the previous one."
**And** no Odometer Reading is created.
**And** the location is **not** changed, because the guard runs before any field is applied.

## Scenario 21: the derived distance is the greatest, not the latest

**Given** a vehicle with three readings: 12 500 dated 2026-02-01, 13 200 dated 2026-03-15 and 12 900 dated 2026-04-02.

**Then** the vehicle's distance field reads **13 200**.

**And when** User OFF types 13 000 into the distance field, the save is refused, because 13 200 is greater than 13 000.

## Scenario 22: a zero distance on a vehicle creates nothing

**Given** the vehicle of scenario 19, whose derived distance is 12 500.

**When** User OFF types 12 500 into the distance field — the same value — and saves.

**Then** the guard passes, one further Odometer Reading is created with value 12 500 and date 2026-09-12, and the vehicle now has two readings of the same value.

**And when** User OFF types 0 into the distance field and saves, the guard refuses, because 12 500 is greater than 0.

## Scenario 23: a reading may be created with any value directly

**Given** the vehicle of scenario 19, whose derived distance is 12 500.

**When** User OFF creates an Odometer Reading directly with vehicle = this vehicle, value 9 000 and date 2026-08-01.

**Then** the reading is created; nothing refuses it, because the never-decrease guard belongs only to the vehicle's distance field.
**And** the vehicle's distance field still reads 12 500.
**And** the reading's driver is Contact P, filled from the vehicle because the reading had none.

## Scenario 24: an explicitly chosen reading driver is never overwritten

**Given** an Odometer Reading whose driver has been set explicitly to Contact R and whose vehicle is CAR1, driven by Contact P.

**When** User OFF changes the reading's vehicle to CAR2, driven by Contact Q.

**Then** the reading's driver stays Contact R.

## Scenario 25: a reading must name a vehicle

**When** User OFF creates an Odometer Reading with value 1 000 and no vehicle.

**Then** the save is refused by rule FLT-024 with the platform's required-field message naming "Vehicle".

---

# Part four: services

## Scenario 26: recording a service by hand

**Given** the vehicle of scenario 12, driven by Contact P, model A3, manufacturer AUDI, fleet manager User FM; and a Fleet Service Type named "Oil Change" of the service category.

**When** User ADM creates a Vehicle Service with vehicle = the vehicle, kind = "Oil Change", description "Annual oil change", date 2026-09-10, cost 89.50, vendor = Contact Q, vendor reference "INV-3392".

**Then** the service exists with those values, and additionally: model = A3, manufacturer = AUDI, fleet manager = User FM, driver = Contact P, company = Company One, currency = the company currency, stage `new`, archive flag true.
**And** the service's display name is "Oil Change", because the naming field of the entity is the service kind.
**And** the vehicle's service count reads 1.

## Scenario 27: a fleet officer may not create a service

**Given** the same inputs as scenario 26.

**When** User OFF attempts to create the service.

**Then** the creation is refused by the access-rights matrix, rule FLT-038, with the platform's access-denied message. User OFF may open and read services but not create them.

## Scenario 28: a service must name a kind and a vehicle

**When** User ADM creates a Vehicle Service with a cost of 50.00, no kind and no vehicle.

**Then** the save is refused by rules FLT-004 and FLT-005, naming both "Service Type" and "Vehicle".

## Scenario 29: writing a distance on a service creates a linked reading

**Given** the service of scenario 26, dated 2026-09-10, on a vehicle whose derived distance is 12 500.

**When** User ADM types 12 800 into the service's distance field.

**Then** one Odometer Reading is created with value 12 800, date **2026-09-10** — the service's date, not today — and vehicle = the service's vehicle.
**And** the service's odometer link points at it, and the service's distance field reads 12 800.
**And** the vehicle's derived distance becomes 12 800.

## Scenario 30: emptying a service's distance is refused

**Given** the service of scenario 29, whose distance reads 12 800.

**When** User ADM clears the distance field and saves.

**Then** the save is refused with the message "Emptying the odometer value of a vehicle is not allowed."

## Scenario 31: a zero distance on a new service is dropped silently

**When** User ADM creates a Vehicle Service with vehicle, kind and cost, and a distance of 0.

**Then** the service is created, no Odometer Reading is created, the service's odometer link is empty and no message is shown.

## Scenario 32: a cancelled service leaves the cost analysis

**Given** the service of scenario 26, cost 89.50, dated 2026-09-10, in stage `new`, and the cost analysis showing 89.50 for that vehicle in September 2026.

**When** User ADM moves the service to stage `cancelled`.

**Then** the next derivation of the cost analysis reports no service row for that vehicle in September 2026, because a cancelled service is excluded and no other service exists in that month.

---

# Part five: the vendor-bill bridge

## Scenario 33: posting a vendor bill creates a service

**Given** the accounting bridge is installed and the "Vendor Bill" service kind exists. A vendor bill of Company One, in the company currency, with vendor Contact Q, invoice date 2026-09-01, accounting date 2026-09-01, in draft, carrying one product line labelled "Brake pads", quantity 1, unit price 50.00, no tax, and the vehicle column set to CAR1.

**When** the bill is posted, on 2026-09-12.

**Then** exactly one Vehicle Service is created with:

| Field | Value |
|---|---|
| Service kind | "Vendor Bill" |
| Vehicle | CAR1 |
| Vendor | Contact Q |
| Description | "Brake pads" |
| Journal item link | The product line of the bill |
| Cost | 50.00 |
| Date | **2026-09-12**, the posting day, not the bill date |
| Stage | `new` |

**And** one message is posted to the service's thread whose body is "Service Vendor Bill: " followed by a link to the journal entry.
**And** CAR1's bill count reads 1.
**And** no service is created for the payable line or for any tax line.

## Scenario 34: the cost of a billed service cannot be typed

**Given** the service of scenario 33, whose cost is 50.00.

**When** User ADM types 60.00 into the service's cost field through the transport layer or through a multiple-record edit.

**Then** the write is refused with the message "You cannot modify amount of services linked to an account move line. Do it on the related accounting entry instead."
**And** on the form the field is read-only, so the message is not normally reached.

## Scenario 35: changing the bill's amount changes the service's cost

**Given** the service of scenario 33.

**When** the bill is reset to draft, the line's unit price is changed to 110.00, and the bill is posted again.

**Then** no second service is created.
**And** the existing service's cost becomes 110.00.

## Scenario 36: a billed service cannot be deleted

**Given** the service of scenario 33.

**When** User ADM deletes it.

**Then** the deletion is refused with the message "You cannot delete log services records because one or more of them were bill created."

**And given** a second, hand-created service with no journal item, deleting that one succeeds.

**And given** a selection holding both, deleting the selection is refused entirely and neither is removed.

## Scenario 37: moving the vehicle on a bill line moves the service

**Given** the service of scenario 33 on CAR1, and a second vehicle CAR2.

**When** the bill is reset to draft, the line's vehicle is changed to CAR2, and the bill is posted again.

**Then** CAR1 has no service.
**And** CAR2 has the same service record, whose vehicle now reads CAR2 and whose cost is unchanged.
**And** no second service is created.

## Scenario 38: clearing the vehicle on a bill line deletes the service

**Given** the state after scenario 37.

**When** the bill is reset to draft and the line's vehicle is cleared.

**Then** the service is deleted, with the deletion guard bypassed.
**And** the line's service collection is empty.
**And** posting the bill again creates nothing.

## Scenario 39: putting the vehicle back creates a new service

**Given** the state after scenario 38, on 2026-09-20.

**When** the line's vehicle is set to CAR2 again and the bill is posted.

**Then** a **new** Vehicle Service is created, dated 2026-09-20, with an empty thread.

## Scenario 40: a vendor credit note creates nothing

**Given** the accounting bridge is installed.

**When** a vendor credit note carrying a product line that names CAR1, for 50.00, is posted.

**Then** no Vehicle Service is created.
**And** CAR1's bill count nevertheless rises by one, because the count includes every purchase entry that is not cancelled.

## Scenario 41: the cost of a billed service on a foreign-currency bill

**Given** a vendor bill of Company One entered in a second currency at a rate of two units of that currency to one unit of the company currency, with one product line naming CAR1, quantity 1, unit price 5 000.00 in the second currency, and no tax.

**When** the bill is posted.

**Then** the line's untaxed subtotal is 5 000.00 in the second currency and its debit is 2 500.00 in the company currency.
**And** the generated service's cost is **2 500.00**, which differs from the untaxed subtotal.

## Scenario 42: a cancelled bill leaves the service in place

**Given** the service of scenario 33 and its posted bill.

**When** the bill is cancelled.

**Then** the service still exists, still with cost 50.00, and still contributes to the cost analysis.
**And** CAR1's bill count drops to 0, because the count excludes cancelled entries.

## Scenario 43: the vehicle survives an accrual period change

**Given** a posted vendor bill of Company One dated 2026-09-01 with one expense item of 100.00 debit on the expense account, naming CAR1; and an expense accrual account.

**When** a user moves sixty per cent of that item to 2026-09-10 through the accrual assistant, using a miscellaneous journal.

**Then** four journal items exist across two entries:

| Entry date | Account | Debit | Credit | Vehicle |
|---|---|---|---|---|
| 2026-09-01 | Expense account | — | 60.00 | CAR1 |
| 2026-09-01 | Expense accrual account | 60.00 | — | none |
| 2026-09-10 | Expense account | 60.00 | — | CAR1 |
| 2026-09-10 | Expense accrual account | — | 60.00 | none |

**And** no Vehicle Service is created by any of the four.

---

# Part six: contracts and their lifecycle

## Scenario 44: creating a contract dated in the present

**Given** CAR1 whose fleet manager is User FM, and the shipped contract kind named "Leasing".

**When** User OFF creates a Vehicle Contract from CAR1's contract counter button with kind "Leasing", start date 2026-09-01, expiration date 2027-08-31, activation cost 250.00 dated 2026-09-01, recurring cost 400.00 monthly, insurer Contact Q, reference "POL-77".

**Then** the contract's name is `Leasing Audi/A3/PAE 326`.
**And** its responsible user defaults to User FM, because it was created from the vehicle.
**And** its state is `open`, arrived at by the re-evaluation of transition T1.6: the start date is on or before today and the expiration date is after today.
**And** its days left reads 353 — the number of days from 12 September 2026 to 31 August 2027 — and expires today is false.
**And** CAR1's contract count reads 1, its derived contract state reads `open`, its renewal-due-soon flag is false and its renewal-overdue flag is false.

## Scenario 45: creating a contract dated in the future

**When** User OFF creates a contract on CAR1 with start date 2026-10-01 and expiration date 2027-09-30.

**Then** the contract's state is `futur`, arrived at by transition T1.5, even though the default state is `open`.
**And** its days left reads −1 and expires today is false.

## Scenario 46: creating a contract already lapsed

**When** User OFF creates a contract on CAR1 with start date 2025-09-01 and expiration date 2026-08-31.

**Then** the contract's state is `expired`, arrived at by transition T1.7.
**And** its days left reads 0 and expires today is false.
**And** CAR1's renewal-overdue flag becomes true and its derived contract state reads `expired`.

## Scenario 47: the default expiration date

**When** User OFF opens a new contract form on 2026-09-12.

**Then** the start date defaults to 2026-09-12 and the expiration date defaults to 2027-09-12.

**And when** the form is opened on 2024-02-29, the expiration date defaults to 2025-02-28.

## Scenario 48: the four transition operations

**Given** the contract of scenario 44, in state `open`.

| When User OFF presses | Then the state becomes | And days left reads |
|---|---|---|
| Cancelled | `closed` | −1 |
| New | `futur` | −1 |
| Running | `open` | 353 |
| Expired | `expired` | 353 |

**And** after the state has been set to `closed`, writing a new expiration date does **not** re-evaluate it: the contract stays `closed`.

## Scenario 49: the recurring frequency is required and the expiry is conditionally required

**When** User OFF creates a contract with an empty recurring frequency.

**Then** the save is refused by rule FLT-007.

**And when** User OFF creates a contract from the form with a monthly frequency and no expiration date, the save is refused by rule FLT-008.

**And when** the same contract is created through the transport layer with a monthly frequency and no expiration date, it is accepted, its state is evaluated as `open` — because the "no expiration date" branch of transition T1.6 is satisfied — and it contributes nothing to the recurring half of the cost analysis.

## Scenario 50: the daily job expires a lapsed contract

**Given** a contract in state `open` with expiration date 2026-09-11, whose responsible user is User FM, and today is 2026-09-12.

**When** the daily job runs.

**Then** pass one raises one activity of the renewal kind on the contract, with a deadline of **2026-09-11**, which is already past, assigned to User FM.
**And** pass two moves the contract to `expired`.
**And** passes three and four leave it alone, because its start date is not later than today and its state is not `futur`.

## Scenario 51: the daily job starts a contract that is due

**Given** a contract in state `futur` with start date 2026-09-12 and expiration date 2027-09-11, and today is 2026-09-12.

**When** the daily job runs.

**Then** pass one skips it, because its state is not `open`.
**And** pass two skips it, because its expiration is not past.
**And** pass three skips it, because its start date is not later than today.
**And** pass four moves it to `open`.

## Scenario 52: the daily job raises exactly one reminder

**Given** a contract in state `open` with expiration date 2026-09-30 and responsible User FM, and the alert delay is 30, so the cut-off is 2026-10-12.

**When** the daily job runs on 2026-09-12.

**Then** one activity of the renewal kind is created with deadline 2026-09-30 and responsible User FM.

**And when** the job runs again on 2026-09-13, no second activity is created, because the contract already carries one.

**And when** User FM marks the activity done on 2026-09-14 without changing the contract, the job of 2026-09-15 raises a fresh one, again with deadline 2026-09-30.

## Scenario 53: rescheduling rather than duplicating

**Given** the contract of scenario 52 carrying one renewal activity with deadline 2026-09-30 and responsible User FM.

**When** User OFF sets the expiration date to 2027-03-31 and the responsible user to User ADM in the same save.

**Then** the state is re-evaluated and stays `open`.
**And** the existing activity's deadline becomes 2027-03-31 and its responsible user becomes User ADM.
**And** no second activity exists.

## Scenario 54: a contract with no responsible user never gets a reminder

**Given** a contract in state `open` with expiration date 2026-09-30 and **no** responsible user.

**When** the daily job runs any number of times.

**Then** no activity is ever created on it.

## Scenario 55: the oscillating contract

**Given** a contract in state `open` with start date 2026-12-01 and expiration date 2026-06-30 — an expiry before the start, which nothing refuses.

**When** the daily job runs on 2026-09-12.

**Then** pass two moves it to `expired`, because its expiration is past.
**And** pass three moves it back to `futur`, because its start date is later than today.
**And** two tracking messages are written to its thread in one run, and the same happens on every subsequent run. This is compatibility finding FLT-C12.

## Scenario 56: the contract reference is limited

**When** User OFF types a reference of sixty-five characters into a contract.

**Then** the value is bounded to sixty-four characters by rule FLT-026.

---

# Part seven: the renewal flags

## Scenario 57: due soon

**Given** CAR1 with one contract in state `open` expiring 2026-09-30, alert delay 30, today 2026-09-12. The difference is 18 days.

**Then** the renewal-overdue flag is false, the renewal-due-soon flag is **true**, and the derived contract state reads `open`, labelled "In Progress".
**And** the vehicle list highlights the row amber.
**And** the "Need Action" filter returns CAR1.
**And** the due-soon search returns CAR1, because the contract's expiry is after today and before 2026-10-12 and its state is `open`.

## Scenario 58: exactly at the delay boundary

**Given** CAR1 with one contract in state `open` expiring 2026-10-12. The difference is exactly 30.

**Then** the renewal-due-soon flag is **false**, because the comparison is strict.
**And** the due-soon search does not return CAR1, for the same reason.

## Scenario 59: expiring today

**Given** CAR1 with one contract in state `open` expiring 2026-09-12. The difference is 0.

**Then** the renewal-overdue flag is false and the renewal-due-soon flag is true.
**And** the contract's days left reads 0 and expires today reads **true**.
**And** the contract list highlights the row amber.
**And** the due-soon search does **not** return CAR1, because that search requires an expiry strictly after today.

## Scenario 60: overdue

**Given** CAR1 with one contract in state `open` expiring 2026-09-02. The difference is −10.

**Then** the renewal-overdue flag is **true**, the renewal-due-soon flag is false, and the derived contract state reads `open`.
**And** the vehicle list highlights the row red.
**And** the contract's days left reads 0 and expires today reads false, so the contract list highlights the contract row red as well, provided CAR1 has no other running contract with a future expiry.
**And** the overdue search returns CAR1.

## Scenario 61: a replacement contract clears the overdue search but not always the flag

**Given** CAR1 with two contracts: one in state `open` expiring 2026-09-02, and one in state `open` expiring 2027-09-01.

**Then** the read form takes the later expiry, computes a difference of 354, and reports overdue false and due soon false. The derived contract state reads `open`.
**And** the overdue search does **not** return CAR1, because it has a running contract expiring after today.
**And** the contract list does not highlight the lapsed contract red, because its vehicle has a running contract expiring on or after today.

## Scenario 62: the read form and the search form disagree

**Given** CAR1 with two contracts, **both in state `expired`**: one expiring 2026-09-07 and one expiring 2026-09-22.

**Then** the read form takes the later expiry, 2026-09-22, computes a difference of 10, and reports overdue **false** and due soon **true**.
**And** the overdue search **does** return CAR1, because it has an expired contract with a past expiry and no contract that is running or incoming with an expiry on or after today.
**And** the vehicle therefore appears in the overdue list while its own record says it is not overdue. This is compatibility finding FLT-C16.

## Scenario 63: cancelled contracts are ignored by both

**Given** CAR1 with one contract in state `closed` expiring 2026-09-02 and nothing else.

**Then** both flags are false and the derived contract state is the **empty text**, not `closed`.
**And** the overdue search does not return CAR1, because a cancelled contract does not satisfy its first condition.
**And** CAR1's contract count reads 0.

## Scenario 64: changing the alert delay changes both flags and the job

**Given** the state of scenario 58, in which the difference is exactly 30 and the due-soon flag is false.

**When** User ADM changes the setting to 45.

**Then** the due-soon flag becomes true, because 30 is strictly less than 45.
**And** the due-soon search returns CAR1, because the limit date is now 2026-10-27.
**And** the daily job's cut-off moves to 2026-10-27, so contracts expiring up to 45 days ahead now receive reminders.

---

# Part eight: archiving

## Scenario 65: archiving a vehicle cascades

**Given** CAR1, active, with two active contracts — one running and one cancelled — and three active services.

**When** User OFF archives CAR1 through the record's action menu.

**Then** the client shows a dialogue whose body is exactly "Every service and contract of this vehicle will be considered as archived. Are you sure that you want to archive this record?".

**And when** the reader confirms:

- CAR1's archive flag becomes false.
- Both contracts, including the cancelled one, have their archive flag cleared.
- All three services have their archive flag cleared.
- CAR1's service count reads 3 and its contract count reads 1 — three archived services and one archived, non-cancelled contract — because the counters compare archive flags.
- CAR1 no longer appears in the default vehicle list and is reachable through the "Archived" filter.

**And when** the reader cancels the dialogue, nothing at all changes.

## Scenario 66: restoring a vehicle restores nothing else

**Given** the state after scenario 65.

**When** User OFF restores CAR1.

**Then** CAR1's archive flag becomes true and no dialogue is shown.
**And** both contracts and all three services stay archived.
**And** CAR1's service count reads 0 and its contract count reads 0, because no active service or contract matches its now-active flag.

## Scenario 67: an archived vehicle's contracts still expire

**Given** the state after scenario 65, in which CAR1 is archived and its running contract expires 2026-09-11.

**When** the daily job runs on 2026-09-12.

**Then** the archived contract is moved to `expired`, because the job does not filter on the archive flag.

---

# Part nine: writing to drivers

## Scenario 68: sending a plain message to two drivers

**Given** CAR1 driven by Contact P and CAR2 driven by Contact Q, both with electronic mail addresses.

**When** User ADM selects both in the list, chooses "Mail to Driver", types the subject "Winter tyres" and the body "Please book your winter tyre change before 1 November.", and presses Send.

**Then** one message is posted to CAR1's thread with author = User ADM's Contact, subject "Winter tyres", the typed body, recipient Contact P and the light notification layout.
**And** one message is posted to CAR2's thread with the same subject and body and recipient Contact Q.
**And** the dialogue closes.

## Scenario 69: one driver has no address

**Given** CAR1 driven by Contact P and CAR3 driven by Contact R, who has no electronic mail address.

**When** User ADM selects both and presses Send.

**Then** a danger notification is shown with the message "The following vehicle drivers are missing an email address: Abigail Peterson."
**And** **no** message is posted, not even to CAR1.
**And** the dialogue stays open.

## Scenario 70: a vehicle with no driver is silently skipped

**Given** CAR1 driven by Contact P and CAR4 with no driver.

**When** User ADM selects both and presses Send.

**Then** no error is shown, one message is posted to CAR1 and nothing at all happens to CAR4.

## Scenario 71: a template renders once per vehicle

**Given** a message template bound to the Vehicle whose subject is "Service due for" followed by a placeholder for the licence plate; CAR1 with plate "PAE 326" driven by Contact P; CAR2 with plate "XYZ 001" driven by Contact Q.

**When** User ADM selects both, chooses the template and presses Send.

**Then** the message on CAR1 has the subject "Service due for PAE 326" and the message on CAR2 has the subject "Service due for XYZ 001".

**And when** no template is chosen and the subject is typed, both messages carry the identical typed subject with no substitution.

## Scenario 72: saving the composition as a template

**Given** the dialogue of scenario 68 with the subject "Winter tyres", the body as typed, and one attachment uploaded by User ADM.

**When** User ADM presses "Save as new template".

**Then** one message template exists named "Vehicle: Mass mail drivers", bound to the Vehicle, with that subject, that body and the default-recipient flag set.
**And** the attachment now belongs to the template and appears in its attachment list.
**And** the dialogue's template field points at the new template.
**And** the new template's form opens in a further dialogue.

---

# Part ten: the people bridge

## Scenario 73: the driver Employee is resolved per company

**Given** Employee E1 of Company One whose work contact is Contact P, and Employee E2 of Company Two whose work contact is also Contact P.

**When** User OFF creates a Vehicle of Company One with driver Contact P.

**Then** the vehicle's driver Employee is **E1**.

**And when** a user of Company Two creates a Vehicle of Company Two with driver Contact P, that vehicle's driver Employee is **E2**.
**And** the Driver Assignment Log created for the second vehicle carries driver Employee E2, because the entry resolves against its vehicle's company.

## Scenario 74: writing a driver Employee rewrites the driver Contact

**Given** a Vehicle of Company One with no driver, and Employee E1 whose work contact is Contact P.

**When** User OFF writes E1 into the vehicle's driver Employee field.

**Then** the vehicle's driver becomes Contact P, written by the synchronisation rule before the save.
**And** one Driver Assignment Log is created with driver Contact P, driver Employee E1 and start date 2026-09-12.

## Scenario 75: the outgoing driver is unsubscribed

**Given** a Vehicle whose driver Employee is E1, whose driver is Contact P, and E1 has a linked user whose own Contact is Contact P2. Both Contact P and Contact P2 follow the vehicle.

**When** User OFF writes a different Employee E3 into the driver Employee field.

**Then** Contact P and Contact P2 are both removed from the vehicle's follower list.
**And** the vehicle's driver becomes E3's work contact.

## Scenario 76: the mobility card reaches the vehicle

**Given** Employee E1 whose work contact is Contact P and whose mobility card is "MC-4417", and a Vehicle whose driver is Contact P.

**Then** the vehicle's mobility card reads `MC-4417`.

**And when** User PERS changes E1's mobility card to "MC-9001", the vehicle's mobility card becomes `MC-9001`.

## Scenario 77: an employee's work contact cannot be removed while vehicles are linked

**Given** Employee E1 whose work contact is Contact P, and a Vehicle whose driver Employee is E1.

**When** a user clears E1's work contact.

**Then** the save is refused with the message "Cannot remove address from employees with linked cars."

## Scenario 78: changing an employee's work contact moves the vehicle's driver

**Given** Employee E1 whose work contact is Contact P, and a Vehicle whose driver Employee is E1 and whose driver is Contact P.

**When** a user sets E1's work contact to Contact Q.

**Then** the vehicle's driver becomes Contact Q.
**And** one new Driver Assignment Log is created with driver Contact Q and start date 2026-09-12.
**And** one activity of the generic to-do kind is scheduled with the note "Specify the End date of Marc Demo".

## Scenario 79: the aggregated licence plate

**Given** Employee E1 driving two vehicles whose plates are "PAE 326" and "ABC 001", and whose own private vehicle plate is "ZZZ 999".

**Then** E1's aggregated licence plate reads `ABC 001 PAE 326 ZZZ 999` — the company plates in plate order, then the private one.

**And when** the private plate is cleared, it reads `ABC 001 PAE 326`.
**And when** both vehicles are released and only the private plate remains, it reads `ZZZ 999`.
**And** searching Employees by the text "PAE" returns E1 while both vehicles are held.

## Scenario 80: the employee's vehicle counter counts history

**Given** Employee E1 whose work contact is Contact P, with four Driver Assignment Logs: three naming both E1 and Contact P, and one naming E1 with the driver Contact P0, which was E1's work contact before it was replaced.

**Then** E1's vehicle counter reads **3**, and the "Cars History" button opens exactly those three entries.

## Scenario 81: releasing vehicles at a departure

**Given** Employee E1 whose work contact is Contact P and whose linked user's contact is Contact P2. Two vehicles are driven by Contact P. Three Driver Assignment Logs exist for Contact P: one with no end date, one with end date 2026-12-31 and one with end date 2026-06-30.

**When** a user registers E1's departure with a departure date of 2026-09-30 and the release option set.

**Then** the entry with no end date has its end date set to **2026-09-30**.
**And** the entry ending 2026-12-31 has its end date set to **2026-09-30**, because 2026-12-31 is later than the departure date.
**And** the entry ending 2026-06-30 is untouched, because it already ended before the departure date.
**And** both vehicles have their driver and their driver Employee emptied.
**And** no new Driver Assignment Log is created and no activity is scheduled, because the driver written is empty.

## Scenario 82: the departure option's default

**Given** User OFF, a member of the fleet officer group, and a user PLAIN who is not.

**Then** opening the departure assistant as User OFF shows the release option set by default.
**And** opening it as user PLAIN shows it clear by default.

## Scenario 83: the fleet-manager activity plan resolves to the vehicle's manager

**Given** an activity plan running on Employees with one line whose responsible kind is `fleet_manager`; Employee E1 driving one vehicle whose fleet manager is User FM.

**When** a user schedules that plan on E1.

**Then** the line's responsible user resolves to User FM, no error and no warning are raised, and one activity assigned to User FM is created.

## Scenario 84: the fleet-manager plan on an employee with no vehicle

**Given** the same plan and Employee E4 who drives nothing.

**When** a user schedules the plan on E4.

**Then** the assistant shows the error "Employee " followed by E4's name followed by " is not linked to a vehicle." and refuses to be saved.

## Scenario 85: the fleet-manager plan on a vehicle with no manager

**Given** the same plan and Employee E1 driving one vehicle whose fleet manager is empty, with User ADM acting.

**When** a user schedules the plan on E1.

**Then** the assistant shows the warning "The vehicle of employee " followed by E1's name followed by " is not linked to a fleet manager, assigning to you." and **may** still be saved.
**And** the created activity is assigned to User ADM.

## Scenario 86: the fleet-manager kind is refused on a non-employee plan

**When** User ADM creates an activity plan line with the responsible kind `fleet_manager` on a plan that runs on Vehicle Contracts.

**Then** the save is refused with the message "Fleet Manager is limited to Employee plans."

---

# Part eleven: the cost analysis

## Scenario 87: a service month and an empty month

**Given** today is 2026-09-12. The earliest service in the database is dated 2026-01-05. CAR1 is active and has three services: 120.00 dated 2026-01-05, active, stage `new`; 80.00 dated 2026-01-22, active, stage `done`; 300.00 dated 2026-03-03, active, stage `cancelled`. CAR1 has no contract.

**When** the cost analysis is derived.

**Then** exactly one service row exists for CAR1: month 2026-01-01, cost kind `service`, cost **200.00**.
**And** no service row exists for February 2026, because no service falls in it and the derivation discards months with none.
**And** no service row exists for March 2026, because the only service in it is cancelled.
**And** contract rows exist for CAR1 for every month from the month of the earliest registration date in the database to October 2026, each with a cost of 0.00, because the contract half keeps every month.

## Scenario 88: a daily recurring contract spread over four months

**Given** CAR1 with one contract: recurring cost 3.50 per day, frequency `daily`, start date 2026-02-10, expiration date 2026-05-20, no activation cost. CAR1 has no other contract and no service.

**When** the cost analysis is derived.

**Then** the contract rows for CAR1 are:

| Month | Cost |
|---|---|
| 2026-01-01 | 0.00 |
| 2026-02-01 | **66.50** |
| 2026-03-01 | **108.50** |
| 2026-04-01 | **105.00** |
| 2026-05-01 | **66.50** |
| 2026-06-01 and later | 0.00 |

**And** the four non-zero figures add to 346.50, which is ninety-nine days at 3.50.

## Scenario 89: a monthly contract with an activation cost

**Given** CAR1 with one contract: activation cost 250.00 dated 2026-02-15, recurring cost 400.00, frequency `monthly`, start date 2026-02-10, expiration date 2027-02-09. CAR1 has no other contract.

**Then** the contract row for 2026-02-01 carries **650.00**; each of the rows for 2026-03-01 through 2027-01-01 carries **400.00**; the row for 2027-02-01 carries **400.00**; and the row for 2027-03-01 carries 0.00.
**And** thirteen monthly charges are reported for a twelve-month agreement, because the coverage test is on the month and not on the day.

## Scenario 90: a yearly contract charges in the month of its cost date

**Given** CAR1 with one contract: recurring cost 1 200.00, frequency `yearly`, activation cost date 2026-03-15 with an activation cost of 0.00, start date 2026-03-01, expiration date 2029-02-28.

**Then** the contract rows carrying 1 200.00 are exactly 2026-03-01, 2027-03-01 and 2028-03-01.
**And** the row for 2029-02-01 carries 0.00, because its month number is 2 and the activation cost date's month number is 3.

## Scenario 91: a weekly contract contributes nothing

**Given** CAR1 with one contract: recurring cost 90.00, frequency `weekly`, start date 2026-02-01, expiration date 2026-12-31, activation cost 0.00.

**Then** every contract row of CAR1 carries 0.00. This is compatibility finding FLT-C09.

## Scenario 92: overlapping contract kinds multiply the amounts

**Given** CAR1 with three contracts, all covering March 2026: contract A with frequency `no` and an activation cost of 500.00 dated 2026-03-05; contract B with frequency `monthly` and a recurring cost of 100.00, start 2026-03-01, expiry 2026-12-31; contract C with frequency `monthly` and a recurring cost of 150.00, start 2026-03-01, expiry 2026-12-31.

**Then** the contract row for CAR1 in 2026-03-01 carries **1 250.00**, not 750.00: the activation cost is counted once per monthly contract. This is compatibility finding FLT-C25.
**And** the contract row for 2026-04-01 carries 250.00, because only the two monthly contracts apply and the one-off set is empty.

## Scenario 93: an archived vehicle produces no row at all

**Given** CAR1 archived, with the services and contracts of scenarios 87 and 88.

**Then** neither the service half nor the contract half produces any row for CAR1, because both restrict themselves to active vehicles.

## Scenario 94: the fuel kind of a cost row is the stored value

**Given** CAR1 whose fuel kind is `plug_in_hybrid_diesel`, with one service in January 2026.

**Then** the January service row shows `plug_in_hybrid_diesel` in the fuel column, not "Plug-in Hybrid Diesel". This is compatibility finding FLT-C10.
**And** the same vehicle in the distance analysis shows the label "Plug-in Hybrid Diesel", because that set mirrors the field properly.

---

# Part twelve: the distance analysis

## Scenario 95: two readings, four months, interpolated

**Given** CAR1 registered on 2026-01-15, with exactly two Odometer Readings: 1 200 dated 2026-02-10 and 4 200 dated 2026-04-20. CAR1 is active.

**When** the distance analysis is derived.

**Then** exactly five rows exist for CAR1:

| Recorded date | Odometer value | Distance travelled |
|---|---|---|
| 2026-01-01 | 0.0000000000 | 0.0000000000 |
| 2026-02-01 | 784.6153846154 | 784.6153846154 |
| 2026-03-01 | 2 026.0869565217 | 1 241.4715719063 |
| 2026-04-01 | 3 373.9130434782 | 1 347.8260869565 |
| 2026-05-01 | 4 199.9999999999 | 826.0869565217 |

**And** the four non-zero distances add to 4 199.9999999999, the residue being the repeating fraction the arithmetic cannot terminate.
**And** the last row is dated May although the last reading is in April, because every computed month is moved forward by one. This is compatibility finding FLT-C19.
**And** the step-by-step arithmetic is in [calculations.md](calculations.md), C-30.5.

## Scenario 96: two readings on the same day

**Given** CAR2 with two readings on 2026-03-01: one of 5 000 and one of 5 400.

**When** the distance analysis is derived.

**Then** only the reading of 5 400 is used, because the second step keeps the greatest value per vehicle and day.

## Scenario 97: no registration date

**Given** CAR3 with no registration date and two readings: 800 dated 2026-05-10 and 2 000 dated 2026-07-10.

**Then** no synthetic zero reading is inserted, because the third step requires a registration date.
**And** the range starts at 2026-05-01, the month of the earliest reading.

## Scenario 98: the distance analysis restricted to one vehicle

**Given** CAR1 as in scenario 95.

**When** a reader presses the chart button beside CAR1's distance field.

**Then** the distance analysis opens restricted to CAR1 and grouped by month only, without the category grouping the menu applies.

---

# Part thirteen: security and multiple companies

## Scenario 99: a fleet officer's permissions

**Given** User OFF, a member of the fleet officer group only.

**Then** User OFF may create, read, update and delete Vehicles, Vehicle Contracts, Odometer Readings and Driver Assignment Logs.
**And** User OFF may read but not change Vehicle Models, Vehicle Manufacturers, Vehicle Categories, Vehicle Statuses, Vehicle Tags, Fleet Service Types and Vehicle Services.
**And** User OFF sees no Configuration menu and no Reporting menu.
**And** attempting to open the cost analysis directly is refused, because the access rights grant it only to the fleet administrator group.

## Scenario 100: a human resources officer sees only vehicles with an employee driver

**Given** User PERS, a member of the human resources officer group and of neither fleet group; CAR1 whose driver Employee is E1; CAR5 whose driver is a Contact that matches no Employee.

**Then** User PERS may read CAR1 and may not read CAR5.
**And** User PERS may not change CAR1, because the rule grants read only and the access-rights row grants read only.
**And** User PERS sees no fleet menu, because the root menu is restricted to the fleet officer group; the vehicles reach them through the Employee form's counter buttons.

## Scenario 101: an administrator who is also a human resources officer sees everything

**Given** a user holding both the fleet administrator group and the human resources officer group; CAR1 with an employee driver and CAR5 without one.

**Then** the user reads both, because the four unrestricting administrator rules admit everything and group rules are combined by disjunction.

## Scenario 102: the multi-company rule on vehicles

**Given** CAR1 of Company One, CAR6 of Company Two, and CAR7 with no company; User OFF is allowed to operate only in Company One.

**Then** User OFF sees CAR1 and CAR7 and does not see CAR6.

## Scenario 103: the multi-company rule on odometer readings follows the vehicle

**Given** three Odometer Readings, one on each of CAR1, CAR6 and CAR7; User OFF allowed only in Company One.

**Then** User OFF sees the readings of CAR1 and CAR7 and does not see the reading of CAR6, even though the reading itself carries no company.

## Scenario 104: assignment entries are not filtered by company

**Given** one Driver Assignment Log on CAR6, which belongs to Company Two; User OFF allowed only in Company One.

**Then** User OFF **sees** that assignment entry through the assignment list, because no company rule exists on the entity. This is the gap recorded in rule FLT-036.

## Scenario 105: the fleet manager choice is restricted

**Given** CAR1 of Company One; User FM of Company One in the fleet officer group; User TWO of Company Two in the fleet officer group; User PORTAL, a shared user of Company One.

**Then** the fleet manager picker on CAR1 offers User FM and offers neither User TWO nor User PORTAL.

## Scenario 106: a future driver of another company is refused

**Given** CAR1 of Company One and a Contact belonging to Company Two.

**When** User OFF sets that Contact as CAR1's future driver.

**Then** the save is refused by rule FLT-019 with the platform's cross-company message.

## Scenario 107: a contract of the wrong company is refused

**Given** a Vehicle Contract of Company One.

**When** User OFF sets its vehicle to CAR6, which belongs to Company Two.

**Then** the save is refused by rule FLT-019.

---

# Part fourteen: the transport dispatch bridge

## Scenario 108: the capacity shares of a batch

**Given** the transport dispatch bridge is installed; CAT-TRUCK with a maximum weight of 44 000 and a maximum volume of 32; a Vehicle of a model whose category is CAT-TRUCK; a Batch Transfer whose vehicle is that Vehicle, whose estimated shipping weight is 31 500 and whose estimated shipping volume is 24.

**Then** the batch's vehicle category is CAT-TRUCK, filled from the vehicle.
**And** the weight share reads 71.5909090909 and the volume share reads 75.0000000000.
**And** both progress bars are shown.

**And when** the estimated shipping weight rises to 48 000, the weight share reads 109.0909090909 and nothing refuses the overload.

## Scenario 109: the capacity shares with no category

**Given** a Batch Transfer with no vehicle and no vehicle category.

**Then** both shares read 0 and both progress bars are hidden.

## Scenario 110: the batch takes the vehicle's driver

**Given** a Vehicle driven by Contact P and a Batch Transfer with no driver.

**When** a user sets that Vehicle on the batch.

**Then** the batch's driver becomes Contact P and its vehicle category becomes the vehicle's category.

**And when** the user then overwrites the driver with Contact Q, the value stays Contact Q; the derivation does not fight the user until the vehicle changes again.

## Scenario 111: the batch's end date

**Given** a Batch Transfer scheduled for 2026-09-14 at 08:00 with no end date.

**Then** the end date becomes 2026-09-14 at 09:00.

**And when** a user widens it to 12:00 and then moves the scheduled date to 09:00, the end date stays 12:00.
**And when** the scheduled date is then moved to 13:00, the end date becomes 14:00.

## Scenario 112: ordering the transfers by postal code

**Given** a Batch Transfer created with four transfers whose Contacts' postal codes are, in the order added, "1050", "1000", none and "1180".

**Then** their positions within the batch become: the one with no code 0, "1000" 1, "1050" 2, "1180" 3.
**And** the printed batch document lists them in that order with a leading Sequence column showing 0, 1, 2, 3.

## Scenario 113: setting a dock redirects the goods

**Given** a Batch Transfer whose operation type is an outgoing one with dispatch management and a dock set containing location DOCK-A; and two transfers in the batch.

**When** a user sets the batch's dock to DOCK-A.

**Then** every move of both transfers has its **source** location set to DOCK-A.

**And given** the same batch on an incoming operation type, setting the dock sets every move's **destination** location to DOCK-A instead.

**And when** the dock is cleared, every move whose destination is not beneath its transfer's own destination location has its destination set back to that location.

## Scenario 114: the batch document carries the dispatch details

**Given** a Batch Transfer with dock DOCK-A, vehicle CAR1 and category CAT-TRUCK.

**When** the batch document is printed.

**Then** the document shows, above the transfer table, three lines labelled "Dock:", "Vehicle:" and "Vehicle Category:" carrying DOCK-A, CAR1's display name and CAT-TRUCK's display name.
**And** the category's display name reads `Transport Truck (44000.0 kg, 32.0 m³)` in a database whose weight unit is the kilogram and whose volume unit is the cubic metre.

## Scenario 115: installing the bridge switches on dispatch management

**Given** two warehouses: WH1 shipping in one step, WH3 shipping in three steps.

**When** the transport dispatch bridge is installed.

**Then** WH1's outgoing and incoming operation types have dispatch management switched on, and no other type of WH1 does.
**And** WH3's outgoing, incoming and packing operation types have it switched on.
**And** WH3's outgoing type's dock set contains WH3's output location.

---

# Part fifteen: edge cases and rounding

## Scenario 116: a vehicle created with a driver and a distance in one call

**Given** Model A3 and Contact P.

**When** User OFF creates a Vehicle with model A3, plate "NEW 001", driver Contact P and a distance of 500, in one save.

**Then** one Vehicle exists.
**And** one Driver Assignment Log exists with driver Contact P and start date 2026-09-12.
**And** one Odometer Reading exists with value 500, date 2026-09-12 and driver Contact P.
**And** the vehicle's distance field reads 500.

## Scenario 117: a multiple-record distance edit refuses on any one record

**Given** CAR1 with a derived distance of 12 500 and CAR2 with a derived distance of 8 000.

**When** User OFF selects both in the list and types 10 000 into the distance column.

**Then** the whole edit is refused with the message "The odometer value cannot be lower than the previous one.", because CAR1's 12 500 is greater than 10 000.
**And** neither vehicle receives a reading.

## Scenario 118: a multiple-record service distance edit links them all to one reading

**Given** two Vehicle Services S1 and S2 on CAR1, dated 2026-09-01 and 2026-09-05 respectively, neither with a distance.

**When** User ADM selects both and types 14 000 into the distance column.

**Then** two Odometer Readings are created, one dated 2026-09-01 and one dated 2026-09-05, both of value 14 000.
**And** **both** S1 and S2 point at the reading created for the second of them, and the first reading is left with no service pointing at it. This is compatibility finding FLT-C14.

## Scenario 119: the service activity indicator prefers overdue

**Given** CAR1 with four services: two carrying no activity, one carrying an activity due 2026-09-19 — aggregated state `planned` — and one carrying an activity whose deadline was 2026-09-11 — aggregated state `overdue`.

**Then** CAR1's service activity indicator reads `overdue` and the form shows the red service counter.

**And when** the overdue activity is completed and only the planned one remains, the indicator reads `none` and the plain counter is shown, because `planned` is discarded.

**And when** an activity is due exactly on 2026-09-12, the indicator reads `today` and the amber counter is shown.

## Scenario 120: the contract list decoration composition

**Given** today is 2026-09-12 and four contracts, each on a different vehicle with no other contract:

| Contract | State | Expiry | Days left | Expires today | Vehicle has a running contract | Row colour |
|---|---|---|---|---|---|---|
| A | `open` | 2026-09-30 | 18 | false | true, itself | none |
| B | `open` | 2026-09-12 | 0 | true | true, itself | amber |
| C | `open` | 2026-08-31 | 0 | false | false, its expiry is past | red |
| D | `closed` | 2026-09-30 | −1 | false | false | greyed out |

**Then** the four rows are coloured exactly as the table states.

## Scenario 121: the year selection grows each January

**Given** the current calendar year is 2026.

**Then** the model-year list offered on a Vehicle and on a Vehicle Model runs from `1970` to `2026` inclusive, giving fifty-seven entries.

**And on** 1 January 2027 the list runs to `2027` and gives fifty-eight entries, without any migration.

## Scenario 122: the derived contract state holds the empty text

**Given** a Vehicle with no contract at all.

**Then** its derived contract state is the **empty text**, and a client that expects one of the four stored values must render it as blank rather than failing.

## Scenario 123: no journal entry is created by any fleet operation

**Given** the whole fixture, and a trial balance taken before the scenarios of parts one to twelve are run.

**When** every one of those scenarios is run except the vendor-bill scenarios of part five.

**Then** the trial balance is unchanged: no journal entry and no journal item was created by any fleet operation.

## Scenario 124: deleting a vehicle removes everything below it

**Given** CAR1 with two contracts, three services none of which carries a journal item, four odometer readings and three assignment entries.

**When** User OFF deletes CAR1.

**Then** CAR1, its two contracts, its three services, its four readings and its three assignment entries are all removed.
**And** nothing refuses the deletion.

## Scenario 125: the analysis identifiers are not stable

**Given** the cost analysis derived once, in which CAR1's January row carries the row identifier 17.

**When** the derived set is rebuilt — for instance because the package is updated.

**Then** CAR1's January row may carry any other identifier, and an external system that stored 17 as a key finds a different row. The identifiers must never be exported as keys.
