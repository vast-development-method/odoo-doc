# Calculations

This file specifies every formula and every algorithm of the fleet domain. Each entry states the quantities involved with their units or currency, the order in which the operations are evaluated where order changes the result, what is rounded and to what precision by which method at which step, and at least one worked numeric example carried to the last decimal the rule produces.

Formulas are written in fenced blocks labelled `formula`, over quantities named in words, using the symbols × ÷ + − =.

## Conventions

- **Currency and rounding of money.** Every monetary amount in this domain is expressed in the currency of the company that owns the record, which the record mirrors from its company. No conversion between currencies takes place anywhere in this domain: the one place a foreign-currency figure enters — the cost of a service derived from a vendor bill — takes the already-converted company-currency amount from the ledger. Monetary amounts are rounded to the number of decimal places the company currency declares, using the currency's own rounding rule, at the moment the ledger produces them; this domain performs no further rounding of money.
- **Rounding of distances and shares.** Distances, capacity percentages and the interpolated figures of the distance analysis are plain decimals. No rounding is applied at any step of their derivation; the values are carried at full precision and rounded only for display, by the client, to the number of digits the field's display setting declares. Every worked example below is therefore carried to ten decimal places where the arithmetic does not terminate, and the reader is told where the value would be truncated for display.
- **Days.** Every day count in this domain is a count of whole calendar days between two dates, obtained by subtracting the earlier date from the later and taking the day component of the result. Time of day never enters, because every date field of this domain stores a calendar day and not an instant.
- **Today.** Unless an entry says otherwise, "today" is the current calendar day in the acting user's time zone. Two rules deliberately use the server's own calendar day instead, and each says so.

## Index

| Identifier | Title |
|---|---|
| [C-01](#c-01-the-display-name-of-a-vehicle) | The display name of a vehicle |
| [C-02](#c-02-the-display-name-of-a-vehicle-model) | The display name of a vehicle model |
| [C-03](#c-03-the-display-name-of-a-vehicle-category) | The display name of a vehicle category |
| [C-04](#c-04-the-display-name-of-a-driver-assignment-entry) | The display name of a driver assignment entry |
| [C-05](#c-05-the-display-name-of-an-odometer-reading) | The display name of an odometer reading |
| [C-06](#c-06-the-name-of-a-vehicle-contract) | The name of a vehicle contract |
| [C-07](#c-07-copying-attributes-from-the-model-to-the-vehicle) | Copying attributes from the model to the vehicle |
| [C-08](#c-08-the-emission-unit-from-the-range-unit) | The emission unit from the range unit |
| [C-09](#c-09-the-derived-distance-of-a-vehicle) | The derived distance of a vehicle |
| [C-10](#c-10-creating-a-reading-from-the-vehicles-distance-field) | Creating a reading from the vehicle's distance field |
| [C-11](#c-11-creating-a-reading-from-a-services-distance-field) | Creating a reading from a service's distance field |
| [C-12](#c-12-the-four-counters-of-a-vehicle) | The four counters of a vehicle |
| [C-13](#c-13-the-default-expiration-date-of-a-contract) | The default expiration date of a contract |
| [C-14](#c-14-days-left-and-expires-today) | Days left and expires today |
| [C-15](#c-15-whether-a-vehicle-already-has-a-running-contract) | Whether a vehicle already has a running contract |
| [C-16](#c-16-the-renewal-flags-and-the-derived-contract-state) | The renewal flags and the derived contract state |
| [C-17](#c-17-searching-for-vehicles-whose-renewal-is-due-soon) | Searching for vehicles whose renewal is due soon |
| [C-18](#c-18-searching-for-vehicles-whose-renewal-is-overdue) | Searching for vehicles whose renewal is overdue |
| [C-19](#c-19-the-service-activity-indicator-of-a-vehicle) | The service activity indicator of a vehicle |
| [C-20](#c-20-the-model-count-of-a-manufacturer) | The model count of a manufacturer |
| [C-21](#c-21-the-vehicle-count-of-a-model) | The vehicle count of a model |
| [C-22](#c-22-the-cost-of-a-service-derived-from-a-vendor-bill) | The cost of a service derived from a vendor bill |
| [C-23](#c-23-the-bill-count-of-a-vehicle) | The bill count of a vehicle |
| [C-24](#c-24-the-aggregated-licence-plate-of-an-employee) | The aggregated licence plate of an employee |
| [C-25](#c-25-the-vehicle-count-of-an-employee) | The vehicle count of an employee |
| [C-26](#c-26-the-mobility-card-of-a-vehicle) | The mobility card of a vehicle |
| [C-27](#c-27-the-alert-delay) | The alert delay |
| [C-28](#c-28-the-fleet-analysis-report-the-service-half) | The Fleet Analysis Report: the service half |
| [C-29](#c-29-the-fleet-analysis-report-the-contract-half) | The Fleet Analysis Report: the contract half |
| [C-30](#c-30-the-fleet-odometer-analysis-report) | The Fleet Odometer Analysis Report |
| [C-31](#c-31-the-capacity-shares-of-a-batch-transfer) | The capacity shares of a batch transfer |
| [C-32](#c-32-the-end-date-of-a-batch-transfer) | The end date of a batch transfer |
| [C-33](#c-33-the-order-of-the-transfers-inside-a-batch) | The order of the transfers inside a batch |

---

## C-01: the display name of a vehicle

**Quantities.** The manufacturer name of the vehicle's model, as text; the name of the vehicle's model, as text; the licence plate of the vehicle, as text.

**Rule.** The three parts are joined by the solidus character with no surrounding spaces. A missing manufacturer name contributes the empty text; a missing model name contributes the empty text; a missing licence plate contributes the translatable text "No Plate".

```formula
vehicle name = manufacturer name or empty text
             + "/"
             + model name or empty text
             + "/"
             + licence plate, or the text "No Plate" when the plate is empty
```

**Evaluation order.** The manufacturer name is read through the model, so a vehicle with no model contributes empty text for both the first and the second part. The result is then "//" followed by the plate or the fallback.

**Stored.** The result is stored, and re-derived whenever the manufacturer name, the model name or the licence plate changes.

**Rounding.** None; the quantities are text.

**Worked example 1.** A vehicle of the model "A3" of the manufacturer "Audi" with the plate "PAE 326" yields `Audi/A3/PAE 326`.

**Worked example 2.** The same vehicle with no plate yields `Audi/A3/No Plate`.

**Worked example 3.** A vehicle whose model has been given no manufacturer name — which cannot happen through the form, because the manufacturer name is required, but can happen if the name is later emptied through the transport layer — with the model "A3" and the plate "PAE 326" yields `/A3/PAE 326`.

---

## C-02: the display name of a vehicle model

**Quantities.** The manufacturer name, as text; the model name, as text.

**Rule.**

```formula
model display name = model name                                     when the manufacturer name is empty
model display name = manufacturer name + "/" + model name           when the manufacturer name is not empty
```

**Rounding.** None.

**Worked example.** The model "A3" of the manufacturer "Audi" displays as `Audi/A3`. A model "Crossover xv" whose manufacturer's name has been emptied displays as `Crossover xv`.

---

## C-03: the display name of a vehicle category

**Without the transport dispatch bridge** the display name is the category's name, unchanged.

**With the transport dispatch bridge** the capacities are appended when they are present.

**Quantities.** The category name, as text; the maximum weight, as a decimal in the platform's weight unit; the label of that weight unit, as text; the maximum volume, as a decimal in the platform's volume unit; the label of that volume unit, as text.

**Rule, as a numbered procedure.**

1. Start with the category name.
2. Build a list of capacity phrases. Add the phrase "the maximum weight, a space, the weight unit label" when the maximum weight is not zero. Add the phrase "the maximum volume, a space, the volume unit label" when the maximum volume is not zero. The weight phrase always comes first.
3. When the list is empty, the display name is the category name and the procedure ends.
4. Otherwise join the phrases with the locale's short list separator, which in English is a comma followed by a space.
5. The display name is the category name, a space, an opening parenthesis, the joined phrases, and a closing parenthesis.

```formula
display name = category name + " (" + capacity phrases joined by the short list separator + ")"
capacity phrase for weight = maximum weight + " " + weight unit label
capacity phrase for volume = maximum volume + " " + volume unit label
```

**Rounding.** The two capacities are rendered with the decimal representation the platform gives a decimal field; no rounding is applied by this rule.

**Worked example.** A category named "Transport Truck" with a maximum weight of 44 000 in a database whose weight unit is labelled `kg` and a maximum volume of 32 in a database whose volume unit is labelled `m³` displays as `Transport Truck (44000.0 kg, 32.0 m³)`. The same category with the volume left at zero displays as `Transport Truck (44000.0 kg)`.

---

## C-04: the display name of a driver assignment entry

**Quantities.** The display name of the entry's vehicle, as text; the name of the entry's driver, as text.

```formula
assignment display name = vehicle display name + " - " + driver name
```

The separator is a space, a hyphen-minus and a space.

**Rounding.** None.

**Worked example.** An entry on the vehicle `Audi/A3/PAE 326` for the driver "Marc Demo" displays as `Audi/A3/PAE 326 - Marc Demo`.

---

## C-05: the display name of an odometer reading

**Quantities.** The display name of the reading's vehicle, as text; the reading's date, rendered in the platform's canonical date form of four-digit year, hyphen, two-digit month, hyphen, two-digit day.

**Rule, as a numbered procedure.**

1. Take the vehicle's display name.
2. When that name is empty, the reading's name is the date rendered as text, and the procedure ends.
3. When the name is not empty and the reading has a date, the reading's name is the vehicle name, a space, a solidus, a space and the rendered date.
4. When the name is not empty and the reading has no date, the reading's name is the vehicle name alone.

```formula
reading name = vehicle display name + " / " + rendered date
```

**Stored.** The result is stored and re-derived whenever the vehicle or the date changes.

**Rounding.** None.

**Worked example.** A reading on 20 April 2026 for the vehicle `Audi/A3/PAE 326` is named `Audi/A3/PAE 326 / 2026-04-20`.

---

## C-06: the name of a vehicle contract

**Quantities.** The display name of the contract's vehicle, as text; the name of the contract's coverage kind, as text.

**Rule, as a numbered procedure.**

1. Take the vehicle's display name.
2. When that name is present **and** the coverage kind has a name, the contract's name is the coverage kind's name, a space, then the vehicle's display name.
3. Otherwise the contract's name is the vehicle's display name, which may be empty.

```formula
contract name = coverage kind name + " " + vehicle display name
```

**Stored and editable.** The result is stored and re-derived whenever the vehicle's name or the coverage kind changes, but a user may overwrite it; the next change of either dependency overwrites the user's text again.

**Rounding.** None.

**Worked example.** A leasing contract on the vehicle `Audi/A3/PAE 326`, whose coverage kind is the shipped type "Leasing", is named `Leasing Audi/A3/PAE 326`. The same contract with no coverage kind is named `Audi/A3/PAE 326`.

---

## C-07: copying attributes from the model to the vehicle

**Purpose.** Fifteen attributes of a vehicle are filled from its model when the model is set or changed, and stay editable afterwards.

**Quantities.** For each of the fifteen pairs listed in [entities.md](entities.md) section 1.6, the value of the model field and the value of the vehicle field.

**Rule, as a numbered procedure.** The procedure runs once per triggered rule, and each rule names exactly one vehicle field to fill.

1. Restrict the set of vehicles being processed to those that have a model. A vehicle with no model is left untouched, so its attribute keeps whatever value it had.
2. Keep a table of results per model, so that a set of vehicles sharing one model does the lookup once.
3. For each vehicle in turn, in the order of the set:
   a. When the vehicle's model is already in the table, take the prepared values from it and go to step 3d.
   b. Otherwise build the values: for each pair in the map, when the vehicle field named by the pair is one of the fields this rule was asked to fill **and** the model's value for that pair is not empty, include the model's value.
   c. Store the built values in the table under the model.
   d. Apply the values to the vehicle.
4. The vehicle's other fields are untouched.

**What "not empty" means.** The test is the ordinary truth test of the value's type. Zero is empty for a whole number and for a decimal; the empty text is empty for text; false is empty for a boolean; an unset reference is empty for a reference. A model whose seating capacity is zero therefore never overwrites a vehicle's seating capacity, and a model whose trailer hitch is false never overwrites a vehicle's trailer hitch.

**Consequence to reproduce.** Because false is treated as empty, the trailer hitch and the electric assistance can only ever be turned **on** by the model, never off. Changing a vehicle from a model with a hitch to a model without one leaves the hitch set on the vehicle.

**Rounding.** None; the values are copied unchanged.

**Worked example.** A model of the manufacturer "Volvo" named "FM" declares: transmission `automatic`, model year `2024`, colour "White", seating capacity 3, number of doors 2, trailer hitch true, emissions figure 0, emission standard empty, fuel kind `diesel`, power 0, horsepower 460, taxable horsepower 0, category "Transport Truck", range unit `km`. A vehicle is created on that model, having previously been created on no model, so every attribute starts empty or zero.

| Vehicle field | Model value | Copied? | Vehicle value after |
|---|---|---|---|
| `transmission` | `automatic` | yes | `automatic` |
| `model_year` | `2024` | yes | `2024` |
| `electric_assistance` | false | no, false is empty | false |
| `color` | "White" | yes | "White" |
| `seats` | 3 | yes | 3 |
| `doors` | 2 | yes | 2 |
| `trailer_hook` | true | yes | true |
| `co2` | 0 | no, zero is empty | 0 |
| `co2_standard` | empty | no | empty |
| `fuel_type` | `diesel` | yes | `diesel` |
| `power` | 0 | no | 0 |
| `horsepower` | 460 | yes | 460 |
| `horsepower_tax` | 0 | no | 0 |
| `category_id` | "Transport Truck" | yes | "Transport Truck" |
| `range_unit` | `km` | yes | `km` |

The vehicle's `vehicle_range` and `power_unit` are not filled at all, because their rules do not exist; see compatibility finding FLT-C04 in [business-rules.md](business-rules.md). The vehicle's `co2_emission_unit` is then derived from the copied range unit by C-08 and becomes `g/km`.

---

## C-08: the emission unit from the range unit

**Quantities.** The range unit, one of the two stored values `km` and `mi`.

```formula
emission unit = "g/km"   when the range unit is "km"
emission unit = "g/mi"   in every other case
```

**Where it applies.** On the Vehicle the result is stored; on the Vehicle Model it is not stored and is re-derived on every read. The rule is identical on both.

**Note on the "every other case" branch.** The rule tests only for `km`; anything else, including an empty range unit, yields `g/mi`. Since the range unit is required with a default of `km`, an empty value cannot occur through the form.

**Rounding.** None.

**Worked example.** A vehicle whose range unit is changed from `km` to `mi` has its emission unit changed in the same operation from `g/km` to `g/mi`. Its emissions figure is **not** converted: a vehicle recorded as 120 grammes per kilometre becomes a vehicle recorded as 120 grammes per mile. A rebuild must reproduce that, and a user changing the unit must retype the figure.

---

## C-09: the derived distance of a vehicle

**Quantities.** The values of the vehicle's odometer readings, as decimals in the vehicle's odometer unit.

**Rule.**

```formula
derived distance = the greatest value among the vehicle's odometer readings
derived distance = 0   when the vehicle has no odometer reading
```

**Evaluation.** The readings of the vehicle are ordered by value descending and the first is taken. The date plays no part; see compatibility finding FLT-C17 in [business-rules.md](business-rules.md).

**Not stored.** The value is re-derived on every read.

**Rounding.** None.

**Worked example.** A vehicle has three readings: 12 500 on 1 February 2026, 13 200 on 15 March 2026 and 12 900 on 2 April 2026 — the last being a correction typed against the wrong vehicle and later moved. The derived distance is **13 200**, not 12 900, because the greatest value wins. Writing 13 000 into the vehicle's distance field is then refused by rule FLT-002, because 13 200 is greater than 13 000.

---

## C-10: creating a reading from the vehicle's distance field

**Quantities.** The value typed into the vehicle's distance field, as a decimal; the vehicle's current driver.

**Rule, as a numbered procedure.**

1. **Guard.** Compare the typed value against the current derived distance of every vehicle in the selection, as C-09 gives it. When any vehicle's derived distance is strictly greater than the typed value, refuse the entire write with the message of rule FLT-002 and change nothing.
2. For each vehicle of the selection whose typed value is **not zero**, prepare one Odometer Reading with: that value; today's date in the acting user's time zone; that vehicle; and that vehicle's current driver at the moment of the write.
3. Create the prepared readings in one operation. Vehicles whose typed value is zero contribute nothing.

**Rounding.** None.

**Worked example.** A vehicle whose greatest reading is 12 500 receives 13 100 in its distance field on 12 September 2026, while its driver is "Marc Demo". The guard passes because 12 500 is not greater than 13 100. One reading is created: value 13 100, date 2026-09-12, driver "Marc Demo". The vehicle's derived distance becomes 13 100.

---

## C-11: creating a reading from a service's distance field

**Quantities.** The value typed into the service's distance field, as a decimal; the service's date; today.

**Rule, as a numbered procedure.**

1. For each service of the selection in turn:
   a. **Guard.** When the typed value is empty or zero, refuse with the message of rule FLT-003.
   b. Prepare one Odometer Reading with: that value; the service's date when it has one, and today in the acting user's time zone otherwise; and the service's vehicle. No driver is supplied, so the reading's own derivation fills it from the vehicle.
   c. Create the reading.
   d. Set the odometer link — **on the whole selection**, not on the individual service; see compatibility finding FLT-C14 in [business-rules.md](business-rules.md).
2. When the write is a creation and the supplied value is zero, step 1a is never reached, because the zero is removed from the supplied values first; see rule FLT-035.

**Rounding.** None.

**Worked example.** A service dated 20 April 2026 on the vehicle `Audi/A3/PAE 326` receives 13 400 in its distance field. One reading is created with value 13 400, date 2026-04-20 and that vehicle; its driver is filled from the vehicle. The service's odometer link points at it, and the service's distance field then reads 13 400.

---

## C-12: the four counters of a vehicle

All four counters are produced by one pass, and each is a count over a differently filtered set.

**Quantities.** The vehicle's own archive flag, as a boolean; the sets of related records.

```formula
odometer reading count = the number of odometer readings whose vehicle is this vehicle

driver history count   = the number of driver assignment entries whose vehicle is this vehicle

service count          = the number of vehicle services whose vehicle is this vehicle
                         and whose archive flag equals this vehicle's archive flag

contract count         = the number of vehicle contracts whose vehicle is this vehicle
                         and whose state is not "closed"
                         and whose archive flag equals this vehicle's archive flag
```

**Evaluation.** The service set and the contract set are read with the archive filter switched off, so archived records are visible to the count and the archive flag is compared explicitly. The odometer and history counts apply no archive filter at all, because neither entity has an archive flag.

**Not stored.** All four are re-derived on every read.

**Rounding.** None; the results are whole numbers.

**Worked example.** A vehicle is active. It has four odometer readings, three driver assignment entries, five services of which two are archived, and four contracts of which one is cancelled and one is archived and running.

| Counter | Value | Why |
|---|---|---|
| Odometer reading count | 4 | Every reading counts. |
| Driver history count | 3 | Every entry counts. |
| Service count | 3 | Five services less the two archived, because the vehicle is active. |
| Contract count | 2 | Four contracts, less the cancelled one, less the archived one. |

The same vehicle after being archived: its cascade archives all five services and all four contracts. The service count then reads 5 and the contract count reads 3 — four contracts less the cancelled one, all four now archived and therefore matching.

---

## C-13: the default expiration date of a contract

**Quantities.** The start date, as a calendar day.

```formula
default expiration date = start date + 1 year
```

**Evaluation.** The addition is calendar-aware: the year number is increased by one and the month and day are kept, except that a day that does not exist in the target month is clamped to the last day of that month.

**Rounding.** None.

**Worked example 1.** A contract started on 12 September 2026 defaults to an expiration of 12 September 2027.

**Worked example 2.** A contract started on 29 February 2024 defaults to an expiration of 28 February 2025, because 29 February 2025 does not exist.

**Where it applies.** The default is computed from today when the contract is created, not from whatever start date the user subsequently types. Changing the start date afterwards does not move the expiration date.

---

## C-14: days left and expires today

**Quantities.** The contract's expiration date, as a calendar day; the contract's state; today, taken here as the **server's** calendar day rather than the acting user's, because the rule reads the platform's own current date.

**Rule.**

```formula
difference in days = expiration date − today

days left      = difference in days   when the state is "open" or "expired", the expiration date is set and the difference is greater than 0
days left      = 0                    when the state is "open" or "expired", the expiration date is set and the difference is 0 or negative
days left      = −1                   in every other case

expires today  = true                 when the state is "open" or "expired", the expiration date is set and the difference is exactly 0
expires today  = false                in every other case
```

**Why days left never goes negative.** The value is used by the list decoration together with `expires today` and with C-15; the composition is what recovers the overdue case. See compatibility finding FLT-C20 in [business-rules.md](business-rules.md).

**Rounding.** None; the result is a whole number of days.

**Worked example.** Today is 12 September 2026.

| Contract | State | Expiration | Difference | Days left | Expires today |
|---|---|---|---|---|---|
| A | `open` | 2026-09-30 | 18 | 18 | false |
| B | `open` | 2026-09-12 | 0 | 0 | true |
| C | `open` | 2026-08-31 | −12 | 0 | false |
| D | `expired` | 2026-08-31 | −12 | 0 | false |
| E | `futur` | 2026-12-31 | 110 | −1 | false |
| F | `closed` | 2026-09-30 | 18 | −1 | false |
| G | `open` | none | — | −1 | false |

The list view colours row B amber, because it expires today. It colours rows C and D red, because their days left is zero, they do not expire today, and — assuming their vehicles have no other running contract — C-15 yields false for them. It leaves rows A, E, F and G uncoloured.

---

## C-15: whether a vehicle already has a running contract

**Quantities.** The contract's vehicle; the states and expiration dates of that vehicle's contracts; today, taken as the server's calendar day.

```formula
vehicle has a running contract = true   when at least one contract of this contract's vehicle
                                        has state "open" and an expiration date on or after today
vehicle has a running contract = false  otherwise
```

**Evaluation.** The test is made across the vehicles of the whole selection in one pass, and each contract of the selection then asks whether its own vehicle appeared among the results.

**Not stored.**

**Worked example.** Today is 12 September 2026. A vehicle has two contracts: one expired on 31 August 2026 in state `expired`, and one running from 1 September 2026 to 31 August 2027 in state `open`. Both contracts yield true for this field. The list therefore does not colour the expired one red, because the vehicle is already covered again.

---

## C-16: the renewal flags and the derived contract state

**Quantities.** The contracts of the vehicle that have an expiration date and whose state is not `closed`; the alert delay in days, from C-27; today, in the acting user's time zone.

**Rule, as a numbered procedure.**

1. Read the alert delay.
2. Group the qualifying contracts of every vehicle in the selection by the pair of vehicle and state, and take the greatest expiration date in each group.
3. For each vehicle, reduce the groups to one: keep the group with the greatest expiration date. When two groups tie on the expiration date, the first one encountered wins, and the order in which groups are encountered is not defined by this domain. **Industry-standard default**: a rebuild should break the tie deterministically, preferring the state that sorts earliest among `expired`, `futur`, `open`, so that the more urgent condition is reported.
4. When a vehicle has a surviving group, let the chosen expiration date and the chosen state be that group's.
   a. Compute the difference in days as the chosen expiration date minus today.
   b. The overdue flag is true when the difference is strictly less than zero.
   c. The due-soon flag is true when the overdue flag is false and the difference is strictly less than the alert delay.
   d. The derived contract state is the chosen state.
5. When a vehicle has no surviving group — it has no contract at all, or none with an expiration date, or all of them are cancelled — the overdue flag is false, the due-soon flag is false, and the derived contract state is the **empty text**, not an empty reference.

```formula
difference in days = chosen expiration date − today

overdue  = true   when difference in days < 0
due soon = true   when difference in days ≥ 0 and difference in days < alert delay
```

**Note on the boundary.** A difference of exactly zero — the contract expires today — yields overdue false and, with the standard delay of thirty, due soon true. A difference of exactly the alert delay yields both false, because the comparison is strict.

**Rounding.** None.

**Worked example 1.** Today is 12 September 2026 and the alert delay is 30. A vehicle has one contract in state `open` expiring on 30 September 2026. The difference is 18. Overdue is false; due soon is true because 18 is less than 30; the derived state is `open`, shown as "In Progress".

**Worked example 2.** Same day and delay. A vehicle has one contract in state `open` expiring on 12 October 2026. The difference is 30. Overdue is false; due soon is false, because 30 is not strictly less than 30; the derived state is `open`.

**Worked example 3.** Same day and delay. A vehicle has two contracts: one in state `expired` that lapsed on 7 September 2026, and one in state `expired` that expires on 22 September 2026. Step 2 groups both under the state `expired` and takes the greatest expiration, 22 September. The difference is 10. Overdue is false; due soon is true; the derived state is `expired`, shown as "Expired". The search of C-18, applied to the same vehicle, returns it as overdue. That divergence is compatibility finding FLT-C16.

**Worked example 4.** Same day and delay. A vehicle has one cancelled contract expiring on 30 September 2026 and nothing else. No group survives step 2, because cancelled contracts are excluded. Both flags are false and the derived state is the empty text.

---

## C-17: searching for vehicles whose renewal is due soon

**Quantities.** The alert delay in days, from C-27; today, in the acting user's time zone.

**Rule.** The search is only defined for the inclusion operator. For any other operator the rule declines and the platform falls back to its own handling of a non-stored field, which yields no result.

```formula
limit date = today + alert delay days
```

A vehicle matches when it has **at least one** contract satisfying all three of:

1. The expiration date is strictly later than today.
2. The expiration date is strictly earlier than the limit date.
3. The state is `open` or `expired`.

**Rounding.** None.

**Worked example.** Today is 12 September 2026 and the alert delay is 30, so the limit date is 12 October 2026. A vehicle has three contracts: one in state `open` expiring on 22 September 2026; one in state `closed` expiring on 30 September 2026; one in state `futur` expiring on 1 October 2026. Only the first satisfies all three conditions — the second is cancelled, the third is neither running nor expired — so the vehicle matches. A vehicle whose only contract expires exactly on 12 October 2026 does **not** match, because the second condition is strict.

---

## C-18: searching for vehicles whose renewal is overdue

**Quantities.** Today, in the acting user's time zone.

**Rule.** The search is only defined for the inclusion operator, as C-17.

A vehicle matches when **both** of the following hold:

1. It has at least one contract whose expiration date is set, is strictly earlier than today, and whose state is `open` or `expired`.
2. It has **no** contract whose expiration date is set, is on or after today, and whose state is `open` or `futur`.

The second condition is what makes the search treat a replacement contract as resolving the overdue condition.

**Rounding.** None.

**Worked example 1.** Today is 12 September 2026. A vehicle has one contract in state `open` that expired on 2 September 2026. Condition 1 holds. Condition 2 holds, because there is no other contract. The vehicle matches.

**Worked example 2.** Same day. The same vehicle also has a contract in state `open` running to 11 September 2027. Condition 1 still holds. Condition 2 fails, because that second contract is running and expires after today. The vehicle does not match.

**Worked example 3.** Same day. A vehicle has one contract in state `expired` that lapsed on 7 September 2026 and one contract also in state `expired` that expires on 22 September 2026. Condition 1 holds on the first. Condition 2 asks for a contract that is `open` or `futur` with an expiry on or after today; the second contract is `expired`, so it does not satisfy condition 2's inner test, and condition 2 holds. The vehicle matches. Compare worked example 3 of C-16, in which the record's own flag says the vehicle is not overdue: that is compatibility finding FLT-C16.

---

## C-19: the service activity indicator of a vehicle

**Quantities.** The aggregated activity states of the vehicle's services, each one of `overdue`, `today`, `planned` or empty.

**Rule, as a numbered procedure.**

1. Collect the aggregated activity state of every service of the vehicle.
2. Discard the empty ones and discard `planned`.
3. When nothing remains, the indicator is `none`.
4. Otherwise sort the remaining values as text, ascending, and take the first.

Because the two surviving values sort as `overdue` before `today`, the rule reduces to: overdue wins over today, and today wins over nothing.

```formula
service activity = "overdue"   when at least one service of the vehicle has an overdue activity
service activity = "today"     when no service has an overdue activity and at least one has an activity due today
service activity = "none"      otherwise
```

**Rounding.** None.

**Effect.** The vehicle form shows one of three service counter buttons: the plain one when the indicator is `none`, the red one when it is `overdue` and the amber one when it is `today`. All three show the same count.

**Worked example.** A vehicle has four services. Two carry no activity at all, one carries an activity due next week — aggregated state `planned` — and one carries an activity whose deadline was yesterday — aggregated state `overdue`. Step 2 discards two empties and the `planned`, leaving `overdue`. The indicator is `overdue`, and the red counter button is shown.

---

## C-20: the model count of a manufacturer

```formula
model count = the number of vehicle models whose manufacturer is this manufacturer
              and whose archive flag is set
```

**Stored.** The result is stored and re-derived whenever the archive flag of any model of the manufacturer changes, and whenever the set of models changes.

**Rounding.** None.

**Worked example.** A manufacturer has seven models, of which two are archived. Its model count is 5. Archiving a third reduces it to 4 at once, because the derivation depends on the archive flag of the models.

---

## C-21: the vehicle count of a model

```formula
vehicle count = the number of vehicles whose model is this model
```

**Not stored.** The count is re-derived on every read, and it applies the ordinary archive filter, so archived vehicles are not counted.

**Searchable.** Searching models by vehicle count is answered by deriving the count of **every** model and filtering the results, rather than by a database aggregate. A rebuild may implement the search as an aggregate, provided the same models are returned.

**Rounding.** None.

**Worked example.** A model has nine vehicles of which one is archived. Its vehicle count is 8. Searching for models with a vehicle count other than zero returns it.

---

## C-22: the cost of a service derived from a vendor bill

**Quantities.** The debit of the bound journal item, as a monetary amount in the company currency.

```formula
service cost = debit of the bound journal item, in the company currency
service cost = 0   when the service is bound to no journal item and no cost has been typed
```

**Evaluation.** The value is taken from the ledger after the ledger has performed its own currency conversion and its own rounding to the company currency's decimal places. This domain performs no conversion and no further rounding.

**Not the untaxed subtotal.** On a bill in the company currency the debit of a product line equals its untaxed subtotal, so the two coincide. On a bill in another currency they do not: the untaxed subtotal is expressed in the document currency and the debit in the company currency. See compatibility finding FLT-C08 in [business-rules.md](business-rules.md).

**Worked example 1, single currency.** A vendor bill in the company currency carries one product line naming a vehicle, with a quantity of 1 and a unit price of 50.00 and no tax. Its untaxed subtotal is 50.00 and its debit is 50.00. The generated service's cost is **50.00**.

**Worked example 2, two currencies.** The company currency is the one the ledger keeps its books in. A vendor bill is entered in a second currency at a rate of 2 units of the second currency to 1 unit of the company currency, with one product line naming a vehicle, quantity 1 and unit price 5 000.00 in the second currency. The line's untaxed subtotal is 5 000.00 in the second currency. Its debit is 5 000.00 ÷ 2 = 2 500.00 in the company currency, rounded to the company currency's two decimal places. The generated service's cost is **2 500.00**, and it differs from the untaxed subtotal.

**Worked example 3, a change of amount.** The bill of example 1 is reset to draft, the unit price is changed to 110.00, and the bill is posted again. No second service is created, because the line already carries one. The existing service's cost is re-derived and becomes **110.00**.

---

## C-23: the bill count of a vehicle

**Quantities.** The journal items naming this vehicle.

**Rule, as a numbered procedure.**

1. When the acting user does not hold the accounting read-only group, the vehicle's journal entry collection is empty and its bill count is zero. The procedure ends.
2. Otherwise collect every journal item that names this vehicle, whose parent entry's state is not cancelled, and whose parent entry's kind is one of the purchase kinds — a vendor bill, a vendor credit note or a vendor receipt.
3. Reduce those items to the set of distinct parent entries.
4. The vehicle's journal entry collection is that set, and the bill count is its size.

```formula
bill count = the number of distinct purchase journal entries, not cancelled,
             that carry at least one item naming this vehicle
```

**Rounding.** None.

**Worked example.** A vehicle is named on three items of one posted vendor bill and on one item of a cancelled vendor bill and on one item of a posted vendor credit note. The distinct set is the vendor bill and the credit note; the cancelled entry is excluded. The bill count is **2**.

---

## C-24: the aggregated licence plate of an employee

**Quantities.** The licence plates of the employee's vehicles, as texts; the employee's own private vehicle plate, as text.

**Rule, as a numbered procedure.**

1. Take the plates of the employee's vehicles, keeping only the vehicles that actually have a plate, in the order of the vehicle collection — which is licence plate ascending, then registration date ascending.
2. When the employee has a private plate **and** at least one of the employee's vehicles has a plate, the result is those plates followed by the private plate, all joined by single spaces.
3. Otherwise the result is those plates joined by single spaces, or — when that is empty — the private plate alone.

```formula
aggregated plate = company plates joined by " " + " " + private plate    when both exist
aggregated plate = company plates joined by " "                          when only company plates exist
aggregated plate = private plate                                         when only a private plate exists
aggregated plate = empty                                                 when neither exists
```

**Searchable.** Searching employees by plate matches either a company vehicle's plate or the private plate. Negative operators are not supported by that rule and fall back to the platform's default handling.

**Rounding.** None.

**Worked example 1.** An employee drives two vehicles with plates "PAE 326" and "ABC 001", and has a private plate "ZZZ 999". Step 1 yields, in plate order, "ABC 001" then "PAE 326". The result is `ABC 001 PAE 326 ZZZ 999`.

**Worked example 2.** The same employee with no private plate yields `ABC 001 PAE 326`.

**Worked example 3.** An employee who drives no company vehicle and has the private plate "ZZZ 999" yields `ZZZ 999`.

**Worked example 4.** An employee driving one vehicle with no plate and having no private plate yields the empty text, because step 1 keeps only vehicles that have a plate.

---

## C-25: the vehicle count of an employee

```formula
employee vehicle count = the number of driver assignment entries
                         whose driver Employee is this employee
                         and whose driver Contact is this employee's work contact
```

**What it is not.** It is not the number of vehicles the employee drives today; it is the length of the employee's vehicle history, and an employee who has held three vehicles in succession has a count of three even after giving them all back. The button it labels is titled "Cars History" for that reason.

**Why the second condition.** An entry whose driver Employee is the employee but whose driver Contact is no longer that employee's work contact is an entry that predates a change of contact. Excluding it keeps the count consistent with the list the button opens, which applies the same pair of conditions.

**Rounding.** None.

**Worked example.** An employee whose work contact is Contact P has four assignment entries: three naming Contact P and one naming Contact Q, which was the employee's work contact before it was replaced. The count is **3**, and the list the button opens shows those same three.

---

## C-26: the mobility card of a vehicle

**Quantities.** The vehicle's driver Contact; the mobility card of the Employee behind it.

**Rule, as a numbered procedure.**

1. When the vehicle has no driver, the mobility card is empty and the procedure ends.
2. Search for the first Employee whose **work contact** is the vehicle's driver.
3. When none is found, search for the first Employee whose **linked user's contact** is the vehicle's driver.
4. The mobility card is that Employee's mobility card, or empty when neither search found one.

**Stored.** The result is stored and re-derived whenever the driver changes, and also whenever an Employee's mobility card is written; that second trigger is explicit, because the derivation depends on a field of another entity that the platform does not track for it.

**Note on the two searches.** The two-step search is wider than the resolution of the driver Employee, which keys on the pair of work contact and company. A vehicle may therefore show the mobility card of an Employee that is not its driver Employee — for example when the driver is somebody's user contact rather than their work contact, or when the matching Employee belongs to another company.

**Rounding.** None.

**Worked example.** A vehicle's driver is Contact P. No Employee has Contact P as work contact, but one Employee's linked user has Contact P as its own contact, and that Employee's mobility card is "MC-4417". The vehicle's mobility card is `MC-4417`, while its driver Employee is empty, because the driver Employee resolution requires a work-contact match.

---

## C-27: the alert delay

**Quantities.** The system parameter `hr_fleet.delay_alert_contract`, read as a whole number.

```formula
alert delay in days = the value of the parameter hr_fleet.delay_alert_contract
alert delay in days = 30   when the parameter is not set
```

**Evaluation.** The parameter is read with elevated permissions, so a user who cannot read system parameters still gets the configured value rather than the fallback. The value is converted from text to a whole number; a parameter holding text that is not a whole number causes the conversion to fail, which is a configuration error rather than a business case.

**Where it is used.** In C-16 to decide whether a renewal is due soon; in C-17 to build the search limit date; and in the first pass of the daily job to decide which contracts get a reminder.

**Worked example.** The setting screen shows 30 by default. Changing it to 45 makes C-16 report "due soon" from 45 days before expiry and makes the daily job start raising reminders 45 days before expiry.

---

## C-28: the Fleet Analysis Report: the service half

**Purpose.** One row per vehicle per calendar month carrying the total of that vehicle's service costs in that month.

**Quantities.** The costs of the vehicle's services, as monetary amounts in the company currency; the dates of those services, as calendar days; the earliest service date across the whole database; today.

**Rule, as a numbered procedure.**

1. Build the sequence of months. The first is the calendar month of the **earliest service date in the whole database**, across every vehicle. The last is the month of today plus one month. The step is one month. Every month is represented by its first day.
2. Form every combination of an **active** vehicle with every month of that sequence.
3. To each combination, attach every service of that vehicle whose date falls in the same calendar month.
4. Discard every combination in which the attached service is missing, is archived, or has the state `cancelled`. Because the discard test reads properties of the attached service, a combination with no service at all is discarded too; see compatibility finding FLT-C18 in [business-rules.md](business-rules.md).
5. For each surviving combination, emit one row with: the vehicle; the vehicle's company; the vehicle's display name; the vehicle's **current** driver; the vehicle's fuel kind as text; the first day of the month; the vehicle kind taken from the model; the cost kind `service`; and the cost.

```formula
service cost of a vehicle in a month = the sum of the costs of that vehicle's services
                                       whose date falls in that month,
                                       which are active and are not cancelled
```

**Rounding.** The sum is a sum of stored monetary amounts, each already rounded to the company currency's decimal places. No further rounding is applied.

**Worked example.** Today is 12 September 2026. The earliest service in the database is dated 5 January 2026, so the month sequence runs from January 2026 to October 2026 inclusive. A vehicle has three services: 120.00 dated 5 January 2026 and active and not cancelled; 80.00 dated 22 January 2026 and active and not cancelled; 300.00 dated 3 March 2026 but cancelled.

| Month | Attached services | Surviving? | Row emitted |
|---|---|---|---|
| January 2026 | the two January services | yes | vehicle, 2026-01-01, `service`, cost 120.00 + 80.00 = **200.00** |
| February 2026 | none | no | none |
| March 2026 | the cancelled service | no, it is cancelled | none |
| April 2026 to October 2026 | none | no | none |

The vehicle therefore contributes exactly one service row for the whole period.

---

## C-29: the Fleet Analysis Report: the contract half

**Purpose.** One row per vehicle per calendar month carrying the total of that vehicle's contract costs attributable to that month.

**Quantities.** For each contract of the vehicle: the one-off activation cost and its date; the recurring cost, its frequency, its start date and its expiration date. All amounts are monetary in the company currency.

**Rule, as a numbered procedure.**

1. Build the sequence of months. The first is the calendar month of the **earliest vehicle registration date in the whole database**, across every vehicle. The last is the month of today plus one month. The step is one month.
2. Form every combination of an **active** vehicle with every month of that sequence. Unlike the service half, no further condition discards a combination, so every active vehicle produces a row for every month of the sequence, with a cost of zero where nothing applies.
3. To each combination, attach four independent sets of contracts of that vehicle:
   - **The one-off set**: contracts whose activation cost date falls in the same calendar month as the generated month.
   - **The daily set**: contracts whose recurring frequency is `daily`, whose start month is on or before the generated month, and whose expiration month is on or after the generated month.
   - **The monthly set**: contracts whose recurring frequency is `monthly`, whose start month is on or before the generated month, and whose expiration month is on or after the generated month.
   - **The yearly set**: contracts whose recurring frequency is `yearly`, whose generated month falls between the start date and the expiration date, and the number of whose activation cost date's calendar month equals the number of the generated month.
4. Compute the four contributions and add them.
5. Emit one row with the same columns as the service half, but with the cost kind `contract`.

```formula
one-off contribution  = the sum of the activation costs of the one-off set

daily contribution    = the sum, over the daily set, of
                        recurring cost × number of days in the coverage window of that month

  where the coverage window of that month runs
    from the later  of the first day of the generated month and the contract's start date
    to   the earlier of the first day of the following month and the contract's expiration date
  and the number of days is the whole-day component of the difference of those two

monthly contribution  = the sum of the recurring costs of the monthly set

yearly contribution   = the sum of the recurring costs of the yearly set

contract cost of a vehicle in a month = one-off contribution
                                      + daily contribution
                                      + monthly contribution
                                      + yearly contribution
```

**What is not covered.** The `weekly` frequency contributes nothing to any month; see compatibility finding FLT-C09 in [business-rules.md](business-rules.md). The `no` frequency contributes nothing beyond its one-off activation cost, which is correct.

**Rounding.** The daily contribution is a product of a monetary amount and a whole number of days; it is not rounded. The three other contributions are sums of stored monetary amounts. The total is not rounded further.

**Worked example 1, a daily contract.** A contract on one vehicle has a recurring cost of 3.50 per day, a start date of 10 February 2026 and an expiration date of 20 May 2026, and no activation cost.

| Month | Window start | Window end | Days | Contribution |
|---|---|---|---|---|
| January 2026 | — | — | — | 0.00, the contract's start month is later than January |
| February 2026 | 2026-02-10 | 2026-03-01 | 19 | 3.50 × 19 = **66.50** |
| March 2026 | 2026-03-01 | 2026-04-01 | 31 | 3.50 × 31 = **108.50** |
| April 2026 | 2026-04-01 | 2026-05-01 | 30 | 3.50 × 30 = **105.00** |
| May 2026 | 2026-05-01 | 2026-05-20 | 19 | 3.50 × 19 = **66.50** |
| June 2026 | — | — | — | 0.00, the contract's expiration month is earlier than June |

The total over the life of the contract is 66.50 + 108.50 + 105.00 + 66.50 = **346.50**, which is 99 days at 3.50, and 99 is indeed the number of days from 10 February to 20 May 2026.

**Worked example 2, a monthly contract with an activation cost.** A contract has an activation cost of 250.00 dated 15 February 2026, a recurring cost of 400.00 per month, a start date of 10 February 2026 and an expiration date of 9 February 2027.

| Month | One-off | Monthly | Total |
|---|---|---|---|
| January 2026 | 0.00 | 0.00 | **0.00** |
| February 2026 | 250.00 | 400.00 | **650.00** |
| March 2026 to December 2026 | 0.00 each | 400.00 each | **400.00** each |
| January 2027 | 0.00 | 400.00 | **400.00** |
| February 2027 | 0.00 | 400.00 | **400.00** |
| March 2027 | 0.00 | 0.00 | **0.00** |

The monthly cost is charged in full for the start month and in full for the expiration month, because the test is on the month and not on the day; the contract therefore contributes thirteen monthly charges for a twelve-month agreement. That is the observed arithmetic and must be reproduced.

**Worked example 3, a yearly contract.** A contract has a recurring cost of 1 200.00 per year, an activation cost date of 15 March 2026, a start date of 1 March 2026 and an expiration date of 28 February 2029.

The yearly set attaches the contract to a generated month when that month's first day lies between 1 March 2026 and 28 February 2029 and the month number of the generated month equals 3, the month number of the activation cost date. That is March 2026, March 2027 and March 2028. February 2029 does not qualify even though it is within the window, because its month number is 2. Each of those three months therefore receives **1 200.00**.

**Worked example 4, the multiplication defect.** A vehicle has three contracts: contract A with an activation cost of 500.00 dated 5 March 2026 and a frequency of `no`; contract B with a monthly recurring cost of 100.00 covering March 2026; and contract C with a monthly recurring cost of 150.00 also covering March 2026.

The four sets for March 2026 are: the one-off set holding A; the daily set empty; the monthly set holding B and C; the yearly set empty. The derivation forms the product of the four sets before summing, so it produces two combinations — one for B and one for C — and each of them carries A's activation cost. The sums are therefore:

```formula
one-off contribution  = 500.00 + 500.00 = 1 000.00
monthly contribution  = 100.00 + 150.00 =   250.00
contract cost         = 1 250.00
```

The correct figure is 500.00 + 100.00 + 150.00 = 750.00. The reported figure is **1 250.00**. This is recorded as compatibility finding FLT-C25 in [business-rules.md](business-rules.md). The multiplication occurs whenever more than one of the four sets is non-empty for the same vehicle and month, and the multiplier for each set is the product of the sizes of the other three, treating an empty set as size one.

---

## C-30: the Fleet Odometer Analysis Report

**Purpose.** One row per vehicle per calendar month carrying the distance travelled in that month, interpolated across months in which nobody wrote a reading down, and a running total.

**Quantities.** The values and dates of the vehicle's odometer readings, as decimals and calendar days; the vehicle's registration date.

### C-30.1 The fifteen steps

1. **Collect.** Take every odometer reading with its vehicle, its value, its date and its vehicle's registration date. Readings of archived vehicles are included; readings with no vehicle cannot exist.
2. **Reduce to one per day.** For each pair of vehicle and date, keep the reading with the greatest value. When two readings of the same vehicle and date share the greatest value, keep one of them arbitrarily; they are indistinguishable for every subsequent step.
3. **Seed the registration date.** For each vehicle whose registration date is set and is strictly earlier than every one of its reading dates, add a synthetic reading of value zero on the registration date.
4. **Bracket.** For each reading, in date order within its vehicle, record the date and value of the immediately preceding reading and the date and value of the immediately following reading. The first has no preceding pair and the last no following pair.
5. **Bound the range.** For each vehicle, the range starts at the first day of the month of the earliest registration date, or — when no registration date is present — at the first day of the month of the earliest reading date. It ends at the first day of the month of the latest reading date.
6. **Generate the months.** For each vehicle, generate one row per month from the range start to the range end inclusive, stepping by one month. Attach to each generated month the reading whose date falls in that month, when there is one. A generated month with a reading takes the reading's date, value and bracket pairs; a generated month without one takes the first day of the month as its date, a value of zero, and no bracket pairs.
7. **Bracket the generated months.** For each row of step 6, take the greatest preceding date and the least following date among the readings of step 4 that strictly bracket the row's date, preferring the row's own bracket dates where it has them.
8. **Attach the bracket values.** For each row, take the value of the row of step 6 whose date is the preceding date, and the value of the row whose date is the following date.
9. **Find the strictly preceding row and interpolate.**
   a. For each row, in date order within its vehicle, record the date of the immediately preceding row of this same set. Call it the strictly preceding date.
   b. Compute a raw distance for the row by the three-branch rule below.
10. **Fill the interpolated values.** For a row whose value is zero and whose raw distance exists, the interpolated value is the running sum of the raw distances of that vehicle up to and including this row, in date order. For every other row the interpolated value is the row's own value.
11. **Difference.** For each row, in date order within its vehicle, compute the raw distance as this row's interpolated value minus the preceding row's interpolated value, taking zero when there is no preceding row; and record the preceding row's date and the number of days between the two.
12. **Split across the month boundary.** Divide each row's raw distance between the month of the preceding date and the month of the row's date, by the two-branch rule below.
13. **Aggregate.** For each vehicle and month, add the two shares that fall in it. Then compute the running total of those sums within the vehicle, in month order.
14. **Find the earliest month.** For each vehicle, take the earliest month produced by step 13.
15. **Emit.** The result set is the union of: one row per vehicle carrying that earliest month, a distance of zero and a running total of zero; and every row of step 13 with its month **moved forward by one month**, carrying its running total and its distance. The rows are ordered by vehicle and then by month.

### C-30.2 The interpolation formula of step 9b

```formula
raw distance = (following value − preceding value)
             × ( days from the strictly preceding date to the row date
               ÷ days from the preceding date to the following date )
                                     when the row value is 0
                                     and both a preceding value and a following value exist

raw distance = row value             when there is no preceding value

raw distance = (row value − preceding value)
             × ( days from the strictly preceding date to the row date
               ÷ days from the preceding date to the row date )
                                     in every other case
```

A divisor of zero yields no value at all rather than an error, and step 10 then keeps the row's own value.

### C-30.3 The month-split formula of step 12

```formula
when the preceding date and the row date fall in the same calendar month:
    share of the row's month      = raw distance
    share of the preceding month  = 0

otherwise:
    share of the row's month      = raw distance
                                  × ( days from the first of the row's month to the row date
                                    ÷ number of days between the two dates )
    share of the preceding month  = raw distance
                                  × ( days from the preceding date to the first of the row's month
                                    ÷ number of days between the two dates )
```

When the number of days between the two dates is zero, the share of the row's month falls back to the raw distance in full and the share of the preceding month has no value. When the preceding date is absent, the comparison of the two calendar months has no answer, so the second branch is taken with a divisor that has no value, which again makes the share of the row's month fall back to the raw distance in full and leaves the preceding month with no share; a share with no value is discarded in step 13.

### C-30.4 Rounding

No rounding is applied at any step. The shares are ratios of whole day counts multiplied by decimals, and the running totals are sums of those. The client rounds for display only.

### C-30.5 Worked example

A vehicle is registered on 15 January 2026. It has exactly two odometer readings: 1 200 on 10 February 2026, and 4 200 on 20 April 2026.

**Step 1 and step 2.** Two readings: (2026-02-10, 1 200) and (2026-04-20, 4 200).

**Step 3.** The registration date, 15 January 2026, is earlier than both, so a synthetic reading (2026-01-15, 0) is added. Three readings now exist.

**Step 4.** The bracket pairs:

| Date | Value | Preceding date | Preceding value | Following date | Following value |
|---|---|---|---|---|---|
| 2026-01-15 | 0 | — | — | 2026-02-10 | 1 200 |
| 2026-02-10 | 1 200 | 2026-01-15 | 0 | 2026-04-20 | 4 200 |
| 2026-04-20 | 4 200 | 2026-02-10 | 1 200 | — | — |

**Step 5.** The range runs from 2026-01-01 to 2026-04-01.

**Step 6.** Four generated months:

| Generated month | Row date | Value |
|---|---|---|
| 2026-01-01 | 2026-01-15 | 0 |
| 2026-02-01 | 2026-02-10 | 1 200 |
| 2026-03-01 | 2026-03-01 | 0, no reading in March |
| 2026-04-01 | 2026-04-20 | 4 200 |

**Step 7 and step 8.** The brackets after widening:

| Row date | Preceding date | Preceding value | Following date | Following value |
|---|---|---|---|---|
| 2026-01-15 | — | — | 2026-02-10 | 1 200 |
| 2026-02-10 | 2026-01-15 | 0 | 2026-04-20 | 4 200 |
| 2026-03-01 | 2026-02-10 | 1 200 | 2026-04-20 | 4 200 |
| 2026-04-20 | 2026-02-10 | 1 200 | — | — |

**Step 9.** The strictly preceding dates are, in order: none, 2026-01-15, 2026-02-10, 2026-03-01. The raw distances:

- Row 2026-01-15: there is no preceding value, so the second branch applies and the raw distance is the row value, **0**.
- Row 2026-02-10: the row value is not zero, so the third branch applies. Days from 2026-01-15 to 2026-02-10 is 26; days from the preceding date 2026-01-15 to the row date is also 26.
  ```formula
  raw distance = (1 200 − 0) × (26 ÷ 26) = 1 200.0000000000
  ```
- Row 2026-03-01: the row value is zero and both bracket values exist, so the first branch applies. Days from the strictly preceding date 2026-02-10 to 2026-03-01 is 19; days from the preceding date 2026-02-10 to the following date 2026-04-20 is 69.
  ```formula
  raw distance = (4 200 − 1 200) × (19 ÷ 69) = 3 000 × 0.2753623188 = 826.0869565217
  ```
- Row 2026-04-20: the row value is not zero, so the third branch applies. Days from the strictly preceding date 2026-03-01 to 2026-04-20 is 50; days from the preceding date 2026-02-10 to the row date 2026-04-20 is 69.
  ```formula
  raw distance = (4 200 − 1 200) × (50 ÷ 69) = 3 000 × 0.7246376812 = 2 173.9130434783
  ```

**Step 10.** Only the March row has a zero value with a raw distance, so only it is replaced by the running sum. The running sums of the raw distances in date order are 0, 1 200, 2 026.0869565217, 4 200. The interpolated values are therefore:

| Row date | Interpolated value |
|---|---|
| 2026-01-15 | 0.0000000000 |
| 2026-02-10 | 1 200.0000000000 |
| 2026-03-01 | 2 026.0869565217 |
| 2026-04-20 | 4 200.0000000000 |

**Step 11.** The differences and day spans:

| Row date | Raw distance | Preceding date | Day span |
|---|---|---|---|
| 2026-01-15 | 0.0000000000 | — | — |
| 2026-02-10 | 1 200.0000000000 | 2026-01-15 | 26 |
| 2026-03-01 | 826.0869565217 | 2026-02-10 | 19 |
| 2026-04-20 | 2 173.9130434783 | 2026-03-01 | 50 |

**Step 12.** The splits:

- Row 2026-01-15: no preceding date, so the second branch with a divisor that has no value. The share of January is the raw distance in full, **0.0000000000**; there is no preceding-month share.
- Row 2026-02-10: the preceding date is in January and the row date in February, so the second branch. Days from 2026-02-01 to 2026-02-10 is 9; days from 2026-01-15 to 2026-02-01 is 17.
  ```formula
  share of February = 1 200 × (9 ÷ 26)  =   415.3846153846
  share of January  = 1 200 × (17 ÷ 26) =   784.6153846154
  ```
- Row 2026-03-01: the preceding date is in February and the row date in March, so the second branch. Days from 2026-03-01 to 2026-03-01 is 0; days from 2026-02-10 to 2026-03-01 is 19.
  ```formula
  share of March    = 826.0869565217 × (0 ÷ 19)  =     0.0000000000
  share of February = 826.0869565217 × (19 ÷ 19) =   826.0869565217
  ```
- Row 2026-04-20: the preceding date is in March and the row date in April, so the second branch. Days from 2026-04-01 to 2026-04-20 is 19; days from 2026-03-01 to 2026-04-01 is 31.
  ```formula
  share of April = 2 173.9130434783 × (19 ÷ 50) =   826.0869565217
  share of March = 2 173.9130434783 × (31 ÷ 50) = 1 347.8260869565
  ```

**Step 13.** Aggregating per month and computing the running total:

| Month | Shares added | Distance | Running total |
|---|---|---|---|
| 2026-01-01 | 0.0000000000 + 784.6153846154 | **784.6153846154** | 784.6153846154 |
| 2026-02-01 | 415.3846153846 + 826.0869565217 | **1 241.4715719063** | 2 026.0869565217 |
| 2026-03-01 | 0.0000000000 + 1 347.8260869565 | **1 347.8260869565** | 3 373.9130434782 |
| 2026-04-01 | 826.0869565217 | **826.0869565217** | 4 199.9999999999 |

The four distances add to 4 199.9999999999, which is 4 200 to the precision the arithmetic can carry; the residue is the accumulated repeating fraction of nineteen sixty-ninths and is not corrected anywhere.

**Step 14.** The earliest month is 2026-01-01.

**Step 15.** The emitted rows, with every computed month moved forward by one:

| Vehicle | Recorded date | Odometer value | Distance travelled |
|---|---|---|---|
| the vehicle | 2026-01-01 | 0.0000000000 | 0.0000000000 |
| the vehicle | 2026-02-01 | 784.6153846154 | 784.6153846154 |
| the vehicle | 2026-03-01 | 2 026.0869565217 | 1 241.4715719063 |
| the vehicle | 2026-04-01 | 3 373.9130434782 | 1 347.8260869565 |
| the vehicle | 2026-05-01 | 4 199.9999999999 | 826.0869565217 |

Note the displacement: the distance actually travelled between the January and February readings is reported against February, and the last row is dated May although the last reading is in April. That is compatibility finding FLT-C19 in [business-rules.md](business-rules.md).

---

## C-31: the capacity shares of a batch transfer

**Quantities.** The estimated shipping weight of the batch, as a decimal in the platform's weight unit; the maximum weight of the batch's vehicle category, in the same unit; the estimated shipping volume of the batch and the maximum volume of the category, in the platform's volume unit.

```formula
weight share used = 100 × ( estimated shipping weight ÷ maximum weight of the category )
volume share used = 100 × ( estimated shipping volume ÷ maximum volume of the category )
```

**Guards.** Each share is computed only when its capacity is not zero; when the capacity is zero or the category is not set, the share is zero and the progress bar is hidden.

**Rounding.** None is applied by the rule. The progress bar rounds for display.

**Worked example.** A batch carries an estimated shipping weight of 31 500 in a database whose weight unit is the kilogram, and its vehicle category declares a maximum weight of 44 000.

```formula
weight share used = 100 × (31 500 ÷ 44 000) = 100 × 0.7159090909 = 71.5909090909
```

The bar shows approximately 71.6 per cent. A batch of 48 000 against the same capacity yields 109.0909090909, and the bar shows more than one hundred per cent; nothing refuses an overloaded batch.

---

## C-32: the end date of a batch transfer

**Quantities.** The scheduled date of the batch, as an instant; the current end date, as an instant.

**Rule.**

```formula
end date = scheduled date + 1 hour   when the end date is empty,
                                     or the end date is earlier than the scheduled date
end date = unchanged                 otherwise
end date = empty                     when the scheduled date is empty and the end date is empty
```

**Evaluation.** The rule runs whenever the scheduled date changes. A user who has widened the window by hand keeps their value, because the rule only fires when the stored end date is empty or has fallen behind the scheduled date.

**Rounding.** None.

**Worked example.** A batch scheduled for 14 September 2026 at 08:00 with no end date receives an end date of 14 September 2026 at 09:00. The user widens it to 14 September 2026 at 12:00. The scheduled date is then moved to 09:00; because the end date of 12:00 is still later than the scheduled date, it is kept. The scheduled date is then moved to 13:00; because the end date of 12:00 is now earlier, it is replaced by 14:00.

---

## C-33: the order of the transfers inside a batch

**Quantities.** The postal code of the Contact of each transfer of the batch, as text.

**Rule, as a numbered procedure.**

1. Sort the transfers of the batch by their Contact's postal code, ascending, comparing the codes as text. A transfer whose Contact has no postal code is sorted as if its code were the empty text, which places it before every non-empty code.
2. Number the sorted transfers from zero upwards and write each number as that transfer's position within the batch.

**Rounding.** None; the positions are whole numbers.

**When it runs.** Once, when the batch is created. Adding a transfer to an existing batch does not re-sort it; the new transfer keeps whatever position it had.

**Worked example.** A batch is created with four transfers whose Contacts' postal codes are, in the order they were added, "1050", "1000", none and "1180". Sorted as text, the order becomes: the one with no code, then "1000", then "1050", then "1180". Their positions become 0, 1, 2 and 3 respectively. The printed batch document lists them in that order and prints the positions in a leading column.

**Note on text comparison.** Because the comparison is textual and not numeric, a code of "900" sorts before a code of "1000", and a code of "AB12" sorts after every code beginning with a digit. That is the observed behaviour and must be reproduced; postal codes are not numbers in every jurisdiction.
