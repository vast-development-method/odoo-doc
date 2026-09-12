# Business rules

This file enumerates every validation, constraint, invariant, permission check, locking consideration and observed defect of the fleet domain. Each rule carries a stable identifier of the form `FLT-nnn` for a rule and `FLT-Cnn` for a compatibility finding. Identifiers are stable within this repository and are referenced from the other files of this folder.

Every rule states: what it protects, the exact condition that makes it fail, the exact message the system shows when it fails, where the placeholders in that message come from, and when the check runs. Messages are reproduced verbatim in quotation marks; placeholders inside a message are described in words so that a rebuild can produce the same text with its own values.

## Rule index

| Identifier | Short title | Kind | Entity |
|---|---|---|---|
| [FLT-001](#flt-001-a-vehicle-must-name-a-model) | A vehicle must name a model | Required field | Vehicle |
| [FLT-002](#flt-002-the-distance-of-a-vehicle-may-never-decrease) | The distance of a vehicle may never decrease | Guard with message | Vehicle |
| [FLT-003](#flt-003-the-distance-of-a-service-may-not-be-emptied) | The distance of a service may not be emptied | Guard with message | Vehicle Service |
| [FLT-004](#flt-004-a-service-must-name-a-service-kind) | A service must name a service kind | Required field | Vehicle Service |
| [FLT-005](#flt-005-a-service-must-name-a-vehicle) | A service must name a vehicle | Required field | Vehicle Service |
| [FLT-006](#flt-006-a-contract-must-name-a-vehicle) | A contract must name a vehicle | Required field | Vehicle Contract |
| [FLT-007](#flt-007-a-contract-must-name-a-recurring-frequency) | A contract must name a recurring frequency | Required field | Vehicle Contract |
| [FLT-008](#flt-008-a-recurring-contract-must-carry-an-expiration-date) | A recurring contract must carry an expiration date | Conditional required field | Vehicle Contract |
| [FLT-009](#flt-009-category-names-are-unique) | Category names are unique | Database constraint | Vehicle Category |
| [FLT-010](#flt-010-status-names-are-unique) | Status names are unique | Database constraint | Vehicle Status |
| [FLT-011](#flt-011-tag-names-are-unique) | Tag names are unique | Database constraint | Vehicle Tag |
| [FLT-012](#flt-012-the-cost-of-a-billed-service-may-not-be-typed) | The cost of a billed service may not be typed | Guard with message | Vehicle Service |
| [FLT-013](#flt-013-a-billed-service-may-not-be-deleted) | A billed service may not be deleted | Deletion guard with message | Vehicle Service |
| [FLT-014](#flt-014-an-employee-with-vehicles-must-keep-a-work-contact) | An employee with vehicles must keep a work contact | Constraint with message | Employee |
| [FLT-015](#flt-015-the-fleet-manager-responsible-kind-is-limited-to-employee-plans) | The fleet-manager responsible kind is limited to employee plans | Constraint with message | Activity Plan Template |
| [FLT-016](#flt-016-an-employee-scheduled-through-the-fleet-manager-kind-must-drive-a-vehicle) | An employee scheduled through the fleet-manager kind must drive a vehicle | Blocking error with message | Activity Plan Template |
| [FLT-017](#flt-017-a-vehicle-without-a-fleet-manager-falls-back-to-the-acting-user) | A vehicle without a fleet manager falls back to the acting user | Non-blocking warning with message | Activity Plan Template |
| [FLT-018](#flt-018-every-driver-written-to-must-have-an-electronic-mail-address) | Every driver written to must have an electronic mail address | Refusal with message | Send Mails to Drivers |
| [FLT-019](#flt-019-company-consistency) | Company consistency | Cross-record check | Vehicle, Vehicle Contract |
| [FLT-020](#flt-020-a-model-must-name-a-name-and-a-manufacturer) | A model must carry a name and a manufacturer | Required fields | Vehicle Model |
| [FLT-021](#flt-021-a-manufacturer-must-carry-a-name) | A manufacturer must carry a name | Required field | Vehicle Manufacturer |
| [FLT-022](#flt-022-a-service-kind-must-carry-a-name-and-a-category) | A service kind must carry a name and a category | Required fields | Fleet Service Type |
| [FLT-023](#flt-023-a-vehicle-status-must-carry-a-name) | A vehicle status must carry a name | Required field | Vehicle Status |
| [FLT-024](#flt-024-an-odometer-reading-must-name-a-vehicle) | An odometer reading must name a vehicle | Required field | Odometer Reading |
| [FLT-025](#flt-025-a-tag-must-carry-a-name) | A tag must carry a name | Required field | Vehicle Tag |
| [FLT-026](#flt-026-a-contract-reference-is-limited-to-sixty-four-characters) | A contract reference is limited to sixty-four characters | Length limit | Vehicle Contract |
| [FLT-027](#flt-027-an-assignment-entry-must-name-a-vehicle-and-a-driver) | An assignment entry must name a vehicle and a driver | Required fields | Driver Assignment Log |
| [FLT-028](#flt-028-the-send-assistant-must-name-vehicles-and-an-author) | The send assistant must name vehicles and an author | Required fields | Send Mails to Drivers |
| [FLT-029](#flt-029-the-send-assistant-must-carry-a-subject) | The send assistant must carry a subject | Form-level required field | Send Mails to Drivers |
| [FLT-030](#flt-030-a-contracts-kind-must-be-of-the-contract-category) | A contract's kind must be of the contract category | Choice restriction | Vehicle Contract |
| [FLT-031](#flt-031-the-fleet-manager-choice-is-restricted) | The fleet manager choice is restricted | Choice restriction | Vehicle |
| [FLT-032](#flt-032-the-driver-employee-choice-is-restricted-by-company) | The driver Employee choice is restricted by company | Choice restriction | Vehicle |
| [FLT-033](#flt-033-deleting-a-vehicle-status-empties-the-reference) | Deleting a vehicle status empties the reference | Deletion behaviour | Vehicle Status |
| [FLT-034](#flt-034-archiving-a-vehicle-archives-its-contracts-and-services) | Archiving a vehicle archives its contracts and services | Cascade | Vehicle |
| [FLT-035](#flt-035-a-zero-distance-supplied-when-creating-a-service-is-dropped) | A zero distance supplied when creating a service is dropped | Silent normalisation | Vehicle Service |
| [FLT-036](#flt-036-multi-company-visibility) | Multi-company visibility | Record rules | Five entities |
| [FLT-037](#flt-037-human-resources-officers-see-only-vehicles-with-an-employee-driver) | Human resources officers see only vehicles with an employee driver | Record rule | Vehicle |
| [FLT-038](#flt-038-access-rights) | Access rights | Permission matrix | Every entity |
| [FLT-039](#flt-039-the-analysis-sets-are-derived-and-must-not-be-written) | The analysis sets are derived and must not be written | Invariant | Both analysis sets |
| [FLT-040](#flt-040-the-deletion-guard-bypass-has-exactly-two-callers) | The deletion guard bypass has exactly two callers | Invariant | Vehicle Service |
| [FLT-041](#flt-041-which-bill-lines-produce-a-service) | Which bill lines produce a service | Skip conditions | Journal Entry |
| [FLT-042](#flt-042-the-template-choice-is-restricted-to-vehicle-templates) | The template choice is restricted to vehicle templates | Choice restriction | Send Mails to Drivers |
| [FLT-043](#flt-043-a-negative-model-search-falls-back-to-the-display-name) | A negative model search falls back to the display name | Search behaviour | Vehicle Model |
| [FLT-044](#flt-044-a-contracts-state-is-re-evaluated-only-when-a-date-is-written) | A contract's state is re-evaluated only when a date is written | Trigger condition | Vehicle Contract |
| [FLT-045](#flt-045-a-contract-carries-at-most-one-renewal-reminder) | A contract carries at most one renewal reminder | Invariant | Vehicle Contract |
| [FLT-046](#flt-046-a-driver-employee-is-resolved-only-on-an-unambiguous-match) | A driver Employee is resolved only on an unambiguous match | Resolution rule | Vehicle |
| [FLT-047](#flt-047-an-odometer-readings-driver-is-filled-only-when-empty) | An odometer reading's driver is filled only when empty | Resolution rule | Odometer Reading |
| [FLT-048](#flt-048-a-dock-must-lie-beneath-the-operation-types-dock-set) | A dock must lie beneath the operation type's dock set | Choice restriction | Batch Transfer |
| [FLT-049](#flt-049-locking-and-concurrency) | Locking and concurrency | Concurrency | Several |
| [FLT-050](#flt-050-a-vehicle-may-be-deleted-at-any-time) | A vehicle may be deleted at any time | Deletion behaviour | Vehicle |

### Compatibility findings

| Identifier | Short title |
|---|---|
| [FLT-C01](#flt-c01-the-default-service-kind-exists-only-in-demonstration-data) | The default service kind exists only in demonstration data |
| [FLT-C02](#flt-c02-the-waiting-list-status-exists-only-in-demonstration-data) | The waiting-list status exists only in demonstration data |
| [FLT-C03](#flt-c03-the-plan-to-change-marking-at-creation-time-is-unreachable) | The plan-to-change marking at creation time is unreachable |
| [FLT-C04](#flt-c04-the-range-and-the-power-unit-are-never-copied-from-the-model) | The range and the power unit are never copied from the model |
| [FLT-C05](#flt-c05-the-model-counter-compares-the-archive-flag-against-a-text) | The model counter compares the archive flag against a text |
| [FLT-C06](#flt-c06-a-services-kind-is-not-restricted-to-the-service-category) | A service's kind is not restricted to the service category |
| [FLT-C07](#flt-c07-a-readings-driver-employee-mirrors-the-vehicle-not-the-reading) | A reading's driver Employee mirrors the vehicle, not the reading |
| [FLT-C08](#flt-c08-the-billed-cost-rule-depends-on-one-amount-and-stores-another) | The billed cost rule depends on one amount and stores another |
| [FLT-C09](#flt-c09-the-weekly-recurring-frequency-is-ignored-by-the-cost-analysis) | The weekly recurring frequency is ignored by the cost analysis |
| [FLT-C10](#flt-c10-the-fuel-kind-of-the-cost-analysis-is-carried-as-plain-text) | The fuel kind of the cost analysis is carried as plain text |
| [FLT-C11](#flt-c11-the-vehicle-required-flag-is-always-false) | The vehicle-required flag is always false |
| [FLT-C12](#flt-c12-a-contract-whose-expiry-precedes-its-start-oscillates) | A contract whose expiry precedes its start oscillates |
| [FLT-C13](#flt-c13-deleting-a-vehicle-leaves-journal-items-pointing-at-nothing) | Deleting a vehicle leaves journal items pointing at nothing |
| [FLT-C14](#flt-c14-the-service-distance-link-is-written-to-the-whole-selection) | The service distance link is written to the whole selection |
| [FLT-C15](#flt-c15-the-cost-log-operation-names-a-screen-that-does-not-exist) | The cost-log operation names a screen that does not exist |
| [FLT-C16](#flt-c16-the-renewal-flags-and-their-search-rules-can-disagree) | The renewal flags and their search rules can disagree |
| [FLT-C17](#flt-c17-the-last-distance-of-a-vehicle-is-the-greatest-not-the-latest) | The last distance of a vehicle is the greatest, not the latest |
| [FLT-C18](#flt-c18-the-service-half-of-the-cost-analysis-drops-months-with-no-service) | The service half of the cost analysis drops months with no service |
| [FLT-C19](#flt-c19-the-distance-analysis-shifts-every-month-forward-by-one) | The distance analysis shifts every month forward by one |
| [FLT-C20](#flt-c20-days-left-conflates-expiring-today-with-already-overdue) | Days left conflates expiring today with already overdue |
| [FLT-C21](#flt-c21-no-uniqueness-is-enforced-on-the-plate-or-the-chassis-number) | No uniqueness is enforced on the plate or the chassis number |
| [FLT-C22](#flt-c22-the-departure-option-carries-a-misspelled-storage-name) | The departure option carries a misspelled storage name |
| [FLT-C23](#flt-c23-a-billed-service-is-dated-on-the-posting-day-not-the-bill-date) | A billed service is dated on the posting day, not the bill date |
| [FLT-C24](#flt-c24-assignment-entries-are-closed-only-at-departure) | Assignment entries are closed only at departure |

---

# Part one: required fields and length limits

## FLT-001: a vehicle must name a model

**What it protects.** Every physical attribute of a vehicle, its manufacturer, its kind and its property definition are reached through the model. A vehicle without a model would have none of them.

**Failing condition.** Saving a Vehicle whose `model_id` is empty.

**Message.** The platform's generic required-field message, naming the field by its label "Model". The text is not authored by this domain.

**When it runs.** On creation and on every write that empties the field.

## FLT-004: a service must name a service kind

**Failing condition.** Saving a Vehicle Service whose `service_type_id` is empty.

**Message.** The platform's generic required-field message, naming the field by its label "Service Type".

**Note.** A default is declared for this field, pointing at a Fleet Service Type that exists only in the demonstration data set; see FLT-C01. On an ordinary installation the field therefore starts empty and this rule fires unless the user chooses one.

## FLT-005: a service must name a vehicle

**Failing condition.** Saving a Vehicle Service whose `vehicle_id` is empty.

**Message.** The platform's generic required-field message, naming the field by its label "Vehicle".

**Interaction.** The rule that follows a bound journal item's vehicle deliberately skips items with no vehicle, precisely so that it can never empty this field. The bridge deletes the service instead; see FLT-040.

## FLT-006: a contract must name a vehicle

**Failing condition.** Saving a Vehicle Contract whose `vehicle_id` is empty.

**Message.** The platform's generic required-field message, naming the field by its label "Vehicle".

## FLT-007: a contract must name a recurring frequency

**Failing condition.** Saving a Vehicle Contract whose `cost_frequency` is empty.

**Message.** The platform's generic required-field message, naming the field by its label "Recurring Cost Frequency".

**Note.** The default is `monthly`, so a user who never touches the field satisfies the rule. Choosing `no` is how a contract with no recurring cost is recorded; an empty value is not.

## FLT-008: a recurring contract must carry an expiration date

**What it protects.** The recurring-cost arithmetic of the Fleet Analysis Report reads the expiration date as the end of the coverage window. Without it, a daily or monthly recurring cost would have no end.

**Failing condition.** Saving a Vehicle Contract from the contract form whose `cost_frequency` is anything other than `no` and whose `expiration_date` is empty.

**Message.** The platform's generic required-field message, naming the field by its label "Contract Expiration Date".

**When it runs.** This requirement is declared on the form, not on the entity. A contract created through the transport layer without passing through the form may therefore have a recurring frequency and no expiration date. In that state the contract counts as running under transition T1.6 of [state-machines.md](state-machines.md) — because the "no expiration date" branch of that guard is satisfied — and it contributes nothing to the recurring half of the Fleet Analysis Report, because every recurring clause of that derivation requires an expiration date.

## FLT-020: a model must carry a name and a manufacturer

**Failing condition.** Saving a Vehicle Model whose `name` is empty, or whose `brand_id` is empty.

**Message.** The platform's generic required-field message, naming the field by its label "Model name" or "Manufacturer".

**Additional form requirement.** When the model's vehicle kind is `car`, the fuel kind is also mandatory on the form. It is not mandatory on the entity, so a bicycle model or a model created through the transport layer may have none.

## FLT-021: a manufacturer must carry a name

**Failing condition.** Saving a Vehicle Manufacturer whose `name` is empty.

**Message.** The platform's generic required-field message, naming the field by its label "Name".

## FLT-022: a service kind must carry a name and a category

**Failing condition.** Saving a Fleet Service Type whose `name` is empty, or whose `category` is empty.

**Message.** The platform's generic required-field message, naming the field by its label "Name" or "Category".

## FLT-023: a vehicle status must carry a name

**Failing condition.** Saving a Vehicle Status whose `name` is empty.

**Message.** The platform's generic required-field message, naming the field by its label "Name".

## FLT-024: an odometer reading must name a vehicle

**Failing condition.** Saving an Odometer Reading whose `vehicle_id` is empty.

**Message.** The platform's generic required-field message, naming the field by its label "Vehicle".

**Note.** The value itself is **not** required. A reading of zero, or with no value at all, may be stored. The analysis set treats a missing value as zero.

## FLT-025: a tag must carry a name

**Failing condition.** Saving a Vehicle Tag whose `name` is empty.

**Message.** The platform's generic required-field message, naming the field by its label "Tag Name".

## FLT-026: a contract reference is limited to sixty-four characters

**What it protects.** The policy number column is declared with a fixed width so that it fits the layouts of the documents insurers exchange.

**Failing condition.** Writing more than sixty-four characters into `ins_ref` on a Vehicle Contract.

**Message.** The platform truncates or refuses according to its own storage rule for bounded text; this domain declares only the bound. A rebuild must enforce sixty-four characters.

## FLT-027: an assignment entry must name a vehicle and a driver

**Failing condition.** Saving a Driver Assignment Log whose `vehicle_id` is empty, or whose `driver_id` is empty.

**Message.** The platform's generic required-field message, naming the field by its label "Vehicle" or "Driver".

**Note.** Neither date is required. An entry may therefore be stored with no start date and no end date, which is what happens when a user adds a row by hand from the list without filling the dates. The rule that creates entries automatically always sets the start date to today.

## FLT-028: the send assistant must name vehicles and an author

**Failing condition.** Sending from the Send Mails to Drivers assistant whose `vehicle_ids` collection is empty, or whose `author_id` is empty.

**Message.** The platform's generic required-field message, naming the field by its label "Vehicles" or "Author".

**Note.** The author defaults to the acting user's Contact and the vehicle collection is filled from the selection the operation was launched on, so both are satisfied on the ordinary path.

## FLT-029: the send assistant must carry a subject

**Failing condition.** Sending with an empty subject.

**Message.** The platform's generic required-field message, naming the field by its label "Subject".

**When it runs.** This requirement is declared on the assistant's form, not on the entity, so a caller reaching the assistant through the transport layer may send with no subject.

---

# Part two: uniqueness

## FLT-009: category names are unique

**What it protects.** Categories are chosen by name on models, on vehicles and on batch transfers. Two categories with the same name would be indistinguishable in every one of those pickers.

**Failing condition.** Storing a Vehicle Category whose `name` equals the name of another category. The check is a database uniqueness constraint named `_name_uniq` over the `name` column, so it also catches concurrent inserts.

**Message.** "Category name must be unique"

**Note.** The comparison is the database's exact comparison. Two names differing only in case, or in leading or trailing spaces, are therefore accepted as different. **Industry-standard default**: a rebuild that wants case-insensitive uniqueness should normalise the name before comparing, and this specification records that the observed system does not.

## FLT-010: status names are unique

**Failing condition.** Storing a Vehicle Status whose `name` equals the name of another status. The check is a database uniqueness constraint named `_fleet_state_name_unique` over the `name` column.

**Message.** "State name already exists"

**Note.** The name is translatable. The constraint applies to the stored source value, not to the translations, so two statuses may still read identically in a secondary language.

## FLT-011: tag names are unique

**Failing condition.** Storing a Vehicle Tag whose `name` equals the name of another tag. The check is a database uniqueness constraint named `_name_uniq` over the `name` column.

**Message.** "Tag name already exists!"

The exclamation mark is part of the message and must be reproduced.

## FLT-C21: no uniqueness is enforced on the plate or the chassis number

**Observed behaviour.** Neither `license_plate` nor `vin_sn` on the Vehicle carries a uniqueness constraint. Two vehicles may be recorded with the same plate and the same chassis number, and nothing warns.

**Why it matters.** Both fields identify a physical machine to an outside party — the registration authority and the manufacturer respectively — so duplicates are always data-entry errors.

**Corrected behaviour.** Declare a uniqueness constraint on the chassis number over non-empty values only, and a uniqueness constraint on the pair of licence plate and company over non-empty plates only, the pairing being needed because two companies of the same database may legitimately have held the same plate at different times. Mark them as **compatibility finding** so that an installation migrating existing data can clean duplicates first.

---

# Part three: guards with messages

## FLT-002: the distance of a vehicle may never decrease

**What it protects.** The distance shown on a vehicle is derived from its readings, and the derivation takes the greatest value. Allowing a lower value to be written would create a reading that the derivation immediately ignores, so the user's change would appear to have no effect at all.

**Failing condition.** A write on one or more Vehicles that includes the field `odometer`, where **at least one** of the vehicles being written has a current derived distance strictly greater than the value being written.

**Message.** "The odometer value cannot be lower than the previous one."

**Placeholders.** None.

**When it runs.** At the very start of the write, before every other rule of that write and before any field is applied. A save that combines a lower distance with other changes therefore applies none of them.

**Scope.** The comparison is made against every vehicle in the selection, so a multiple-record edit that lowers the distance of any one of them refuses the whole edit.

**What it does not cover.** Writing an Odometer Reading directly with a lower value is not refused, on any route. The guard belongs to the vehicle's derived field alone.

## FLT-003: the distance of a service may not be emptied

**What it protects.** The distance field on a service is derived from a linked reading. Emptying it would leave the link pointing at a reading that no longer matches, so the system refuses instead of guessing.

**Failing condition.** A write on one or more Vehicle Services that includes the field `odometer` with an empty or zero value.

**Message.** "Emptying the odometer value of a vehicle is not allowed."

**Placeholders.** None.

**When it runs.** When the write reaches the field's write rule, which is after the ordinary fields of the same write have been applied.

**Exception.** See FLT-035: a zero supplied when *creating* a service is removed from the supplied values before it reaches this rule, so it is silently ignored rather than refused.

## FLT-012: the cost of a billed service may not be typed

**What it protects.** A service that came from a vendor bill must always agree with that bill. Letting a user type a different figure would produce a fleet cost report that disagrees with the ledger.

**Failing condition.** A write on one or more Vehicle Services that includes the field `amount`, where **at least one** of the services carries a journal item in `account_move_line_id`.

**Message.** "You cannot modify amount of services linked to an account move line. Do it on the related accounting entry instead."

**Placeholders.** None.

**When it runs.** When the write reaches the field's write rule.

**Scope.** The check tests the whole selection, so writing a cost onto a mixed selection of billed and unbilled services refuses the whole write, including the unbilled ones.

**Client behaviour.** The form makes the cost field read-only whenever the journal item link is set, so the message is normally reached only through the transport layer or through a multiple-record edit.

**Corrective route.** The user opens the bill, resets it to draft, changes the line's amount, and posts again. The service's cost then follows, because it is derived from the item.

## FLT-013: a billed service may not be deleted

**What it protects.** The one-to-one correspondence between qualifying bill lines and services. Deleting the service by hand would leave the line with no service and would make the posting routine create a second one the next time the bill is posted.

**Failing condition.** Deleting one or more Vehicle Services where **at least one** of them carries a journal item, and the deletion does not carry the bypass marker `ignore_linked_bill_constraint`.

**Message.** "You cannot delete log services records because one or more of them were bill created."

**Placeholders.** None.

**When it runs.** As a deletion guard, before any row is removed. The guard is not applied when the package is being uninstalled.

**Scope.** The check tests the whole selection, so deleting a mixed selection of billed and unbilled services refuses the whole deletion.

**Bypass.** See FLT-040.

## FLT-014: an employee with vehicles must keep a work contact

**What it protects.** The driver of a vehicle is a Contact, and the link from a vehicle to an Employee runs through that Contact. Removing the Contact would sever the link and leave the vehicle pointing at an Employee that can no longer be resolved.

**Failing condition.** Writing an empty `work_contact_id` on one or more Employees, where at least one Vehicle names one of those Employees in `driver_employee_id`. The search for those vehicles is made with elevated permissions, so the check is not weakened by a user who cannot read the fleet.

**Message.** "Cannot remove address from employees with linked cars."

**Placeholders.** None.

**When it runs.** As a constraint on the work contact, after the write has been applied and before the transaction is committed. It fires whether the field was emptied directly or emptied as a side effect of another change, such as unlinking the employee's user.

**Corrective route.** Release the vehicles first, either by clearing the driver on each or by registering the departure with the release option set.

## FLT-015: the fleet-manager responsible kind is limited to employee plans

**What it protects.** The resolution rule of the fleet-manager kind reads the record being scheduled as an Employee and asks for its vehicles. On a plan that runs on anything else there are no vehicles to ask for.

**Failing condition.** Saving an Activity Plan Template whose `responsible_type` is `fleet_manager` and whose plan does not run on Employees.

**Message.** "Fleet Manager is limited to Employee plans."

**Placeholders.** None.

**When it runs.** As a constraint on the pair of plan and responsible kind, so it fires both when the kind is changed on a template and when the template is moved to a different plan.

## FLT-016: an employee scheduled through the fleet-manager kind must drive a vehicle

**What it protects.** The same resolution rule. An employee with no vehicle has no fleet manager to route the activity to.

**Failing condition.** Resolving a fleet-manager line of an activity plan against an Employee whose vehicle collection is empty.

**Message.** "Employee %s is not linked to a vehicle."

**Placeholders.** The single placeholder is replaced by the name of the employee.

**When it runs.** While the scheduling assistant is being filled in, once per employee per fleet-manager line. The message is collected as a blocking error: the assistant shows it in its error list and refuses to be saved while it is present.

**Corrective route.** Assign a vehicle to the employee, or remove that employee from the selection, or use a plan whose lines resolve differently.

## FLT-017: a vehicle without a fleet manager falls back to the acting user

**What it protects.** Nothing is refused here; the rule exists so that a missing fleet manager does not silently produce an unassigned activity.

**Failing condition.** Resolving a fleet-manager line against an Employee whose first vehicle exists but has no `manager_id`.

**Message.** "The vehicle of employee %(employee)s is not linked to a fleet manager, assigning to you."

**Placeholders.** The named placeholder is replaced by the name of the employee. The word "you" refers to the acting user, who becomes the responsible user of the generated activity.

**When it runs.** With FLT-016, once per employee per fleet-manager line. The message is collected as a warning: the assistant shows it in its warning list and **may still be saved**.

## FLT-018: every driver written to must have an electronic mail address

**What it protects.** A message posted to a Contact with no address produces a delivery failure the sender never sees. The assistant checks first and refuses the whole send rather than sending some and failing on others.

**Failing condition.** Pressing Send in the Send Mails to Drivers assistant when at least one driver of at least one selected vehicle has no electronic mail address on their Contact.

**Message.** "The following vehicle drivers are missing an email address: %s."

**Placeholders.** The single placeholder is replaced by the names of the affected drivers, joined by a comma followed by a space, in the order the drivers were collected from the selected vehicles.

**Presentation.** The message is shown as a transient notification of the danger kind, not as a blocking dialogue. The assistant stays open.

**When it runs.** First, before any rendering and before any message is posted. **Nothing at all is sent** when the check fails, not even to the drivers that do have an address.

**Note.** A selected vehicle with no driver at all contributes nothing to the check and nothing to the send: the collection of drivers simply skips it. A user who selects a vehicle with no driver therefore sees no error and no message for that vehicle.

---

# Part four: choice restrictions and resolution rules

## FLT-030: a contract's kind must be of the contract category

**Restriction.** The `cost_subtype_id` field of a Vehicle Contract accepts only Fleet Service Types whose `category` is `contract`.

**Enforcement.** Declared on the field, so it is applied both by the picker in the client and by the transport layer when a value is supplied.

**Message on violation.** The platform's generic message for a value outside a field's permitted set.

## FLT-031: the fleet manager choice is restricted

**Restriction.** The `manager_id` field of a Vehicle accepts only users that satisfy all three of:

1. The user is not a shared or portal user.
2. The user's company equals the vehicle's company.
3. The user is a member of the fleet officer group.

**Enforcement.** Declared on the field as a dynamic restriction evaluated against the record being edited, so changing the vehicle's company changes the set of acceptable managers.

**Message on violation.** The platform's generic message for a value outside a field's permitted set.

**Consequence.** A vehicle whose company is empty accepts only users whose own company is empty, which in practice is none. A rebuild should note that setting the company to empty makes the manager field effectively unusable until a company is chosen.

## FLT-032: the driver Employee choice is restricted by company

**Restriction.** The `driver_employee_id` and `future_driver_employee_id` fields of a Vehicle accept only Employees whose company is empty **or** equal to the vehicle's company.

**Enforcement.** Declared on the fields.

**Related resolution.** The automatic resolution of FLT-046 applies a stricter test: it requires the Employee's company to **equal** the vehicle's company, so an Employee with no company is never resolved automatically even though it may be chosen by hand.

## FLT-042: the template choice is restricted to vehicle templates

**Restriction.** The `template_id` field of the Send Mails to Drivers assistant accepts only message templates whose target entity is the Vehicle.

**Enforcement.** Declared on the field as a dynamic restriction.

**Why.** The assistant renders the chosen template against a Vehicle. A template bound to another entity would resolve none of its placeholders.

## FLT-046: a driver Employee is resolved only on an unambiguous match

**Rule.** When a driver Contact is written onto a Vehicle and the people bridge is installed, the driver Employee is derived as follows:

1. Search Employees whose work contact is the written Contact, asking for at most two results.
2. When exactly one is found, set the driver Employee to it.
3. When none is found, or when two or more are found, set the driver Employee to empty.

The same three steps apply to the future driver Contact and the future driver Employee.

**A stricter rule applies to the stored derivation.** The stored `driver_employee_id` is also produced by a rule that runs whenever the driver changes for any reason. That rule groups Employees by the pair of work contact and company, and takes the Employee whose pair is (the vehicle's driver, the vehicle's company). It therefore never crosses companies, and it takes the **first** of the matching Employees when several share the pair rather than refusing.

**Consequence.** In a database where one person is employed by two companies through the same work contact, writing that Contact as the driver of a vehicle of company A resolves to the company-A Employee, and writing it on a vehicle of company B resolves to the company-B Employee. The two-result search of step 1 would find two Employees and yield empty, but the stored derivation immediately overwrites that with the company-matched one.

## FLT-047: an odometer reading's driver is filled only when empty

**Rule.** The rule that derives the driver of an Odometer Reading from its vehicle assigns a value **only when the reading's driver is still empty**. A driver chosen explicitly is never overwritten, not even when the reading is later moved to a different vehicle.

**Consequence.** Moving a reading from one vehicle to another leaves the old driver on it. That is the intended behaviour: the reading records who was driving when the measurement was taken, which does not change because the record was corrected.

## FLT-048: a dock must lie beneath the operation type's dock set

**Restriction.** The `dock_id` field of a Batch Transfer accepts only locations that are the operation type's dock locations or descendants of them.

**Related restriction.** The `dock_ids` field of an Operation Type accepts only internal locations of that operation type's warehouse.

**Enforcement.** Both are declared on the fields. Changing the warehouse of an operation type clears its dock set, so a stale dock cannot survive a warehouse change.

## FLT-043: a negative model search falls back to the display name

**Rule.** Searching Vehicle Models by display name normally matches either the model name or the manufacturer name. That widening is declared only for positive operators. When the operator is a negative one — "is not", "does not contain" and their relatives — the widening rule declines to answer and the platform falls back to matching the display name column alone.

**Consequence.** Searching for models whose display name does not contain a text behaves differently from the negation of searching for models whose display name contains it. A rebuild must reproduce the asymmetry, because reversing it would change which records a saved filter returns.

---

# Part five: deletion, archival and cascades

## FLT-033: deleting a vehicle status empties the reference

**Rule.** Deleting a Vehicle Status is always permitted. Every Vehicle that pointed at it has `state_id` emptied. The deletion is not blocked and no message is shown.

**Consequence.** A vehicle with no status disappears from every board column, because the board groups on the status. It is still visible in the list and the form.

## FLT-034: archiving a vehicle archives its contracts and services

**Rule.** Writing a false `active` on one or more Vehicles first writes a false `active` on every Vehicle Contract of those vehicles and on every Vehicle Service of those vehicles, whatever their own current archive flag and whatever their state.

**Not symmetric.** Writing a true `active` on a Vehicle does nothing to its contracts or services.

**Client confirmation.** The desktop client intercepts the archive entry of a vehicle's action menu and shows a dialogue whose body is exactly "Every service and contract of this vehicle will be considered as archived. Are you sure that you want to archive this record?". The archive proceeds only on confirm. The dialogue is a client behaviour: archiving through the transport layer, through a multiple-record operation on the list, or through an automated rule performs the cascade with no dialogue.

## FLT-050: a vehicle may be deleted at any time

**Rule.** No guard prevents the deletion of a Vehicle. A vehicle with contracts, services, readings, assignment entries and posted vendor bills naming it may be deleted.

**Consequences, in order.**

1. Contracts, services, readings and assignment entries are removed with the vehicle, because each holds a required reference to it.
2. Removing the services would ordinarily hit the deletion guard of FLT-013. It does not, because the guard is a check on an explicit deletion of services and the removal here is driven by the required reference; a rebuild must ensure the same, or a vehicle with billed services would become undeletable.
3. Journal items keep their vehicle column pointing at nothing; see FLT-C13.

## FLT-040: the deletion guard bypass has exactly two callers

**Rule.** The marker `ignore_linked_bill_constraint` suppresses the guard of FLT-013. Exactly two places set it, both in the accounting bridge:

1. Writing an empty vehicle onto a Journal Item, which first deletes the services bound to that item.
2. Deleting a Journal Item, which first deletes the services bound to it.

**Invariant.** No user-facing operation sets the marker. From any screen, the guard is absolute.

## FLT-035: a zero distance supplied when creating a service is dropped

**Rule.** When a Vehicle Service is created and the supplied values contain the key `odometer` with a value that is empty or zero, that key is removed from the supplied values before the record is created.

**Why.** Without the removal, a blank distance on a new service would reach the field's write rule, which would refuse under FLT-003 — or, on a version that did not refuse, would create an Odometer Reading of zero and pollute the distance analysis.

**Scope.** Creation only. A zero written onto an existing service is refused under FLT-003.

---

# Part six: permissions and visibility

## FLT-038: access rights

The complete matrix is reproduced in [configuration.md](configuration.md). The rules that follow from it, expressed as prohibitions, are:

1. A user who holds neither fleet group has no access at all to any fleet entity, except through the two rows granted to the human resources officer group.
2. A fleet officer may **create, read, update and delete** Vehicles, Vehicle Contracts, Odometer Readings and Driver Assignment Logs.
3. A fleet officer may only **read** Vehicle Models, Vehicle Manufacturers, Vehicle Categories, Vehicle Statuses, Vehicle Tags and Fleet Service Types. Every configuration screen is therefore read-only for an officer, and the configuration menu is hidden from them entirely.
4. A fleet officer may only **read** Vehicle Services. Creating, changing and deleting a service requires the fleet administrator group. This is the single most surprising line of the matrix, because a service is an operational record rather than a configuration one; it is recorded here so that a rebuild does not "correct" it.
5. A fleet administrator may create, read, update and delete everything above, plus Activity Types, plus the Fleet Odometer Analysis Report.
6. A fleet administrator may only **read** the Fleet Analysis Report.
7. A fleet administrator may create, read and update the Send Mails to Drivers assistant, but not delete it.
8. A human resources officer may only **read** Vehicles, and only those the record rule of FLT-037 admits.
9. Because the fleet administrator group implies the fleet officer group, an administrator holds every officer permission as well; the odometer row, which is granted to the officer group only, therefore reaches administrators through that implication.

## FLT-036: multi-company visibility

Five record rules restrict what a user sees across companies. All five are global rules, which means they apply to every user including administrators, and they are combined with any group rule by conjunction.

| Entity | Admitted when |
|---|---|
| Vehicle | The vehicle's company is one of the acting user's allowed companies, **or** is empty. |
| Vehicle Contract | The contract's company is one of the acting user's allowed companies, or is empty. |
| Vehicle Service | The service's company is one of the acting user's allowed companies, or is empty. |
| Odometer Reading | The **vehicle's** company is one of the acting user's allowed companies, or is empty. The reading itself has no company. |
| Fleet Analysis Report | The row's company is one of the acting user's allowed companies, or is empty. |

**Not covered.** Driver Assignment Logs, Vehicle Models, Vehicle Manufacturers, Vehicle Categories, Vehicle Statuses, Vehicle Tags, Fleet Service Types and the Fleet Odometer Analysis Report carry no company rule. They are visible to every user who holds the matching access right, in every company. For the six configuration entities that is deliberate: a fleet's manufacturers and statuses are shared. For the assignment log and the distance analysis it is a gap: an assignment entry names a vehicle of a company the reader may not see, and the distance analysis carries no company column at all. **Industry-standard default**: a rebuild should add a rule on the assignment log admitting entries whose vehicle's company is allowed or empty, mirroring the odometer rule, and should add a company column and matching rule to the distance analysis.

## FLT-037: human resources officers see only vehicles with an employee driver

**Rule.** A group rule on the Vehicle, granted to the human resources officer group, admits a vehicle only when it names a driver Employee **or** a future driver Employee. The rule grants read permission only: write, create and delete are explicitly withheld.

**Combination.** Group rules on one entity are combined by disjunction. Four further group rules, granted to the fleet administrator group, carry an empty condition and therefore admit everything. A user who holds both the fleet administrator group and the human resources officer group consequently sees every vehicle, because the administrator rule admits it whatever the human resources rule says.

**A fleet officer is unaffected.** The human resources rule names only the human resources officer group, so it does not apply to a user who is not in that group. A fleet officer who is not a human resources officer therefore sees every vehicle their companies allow.

**The four empty administrator rules.** They exist on the Vehicle, the Vehicle Contract, the Vehicle Service and the Odometer Reading. Their purpose is exactly the neutralisation described above: without them a fleet administrator who is also a human resources officer would be narrowed to vehicles with an employee driver. They do not weaken the multi-company rules, which are global and therefore always conjoined.

---

# Part seven: derivation and posting rules

## FLT-041: which bill lines produce a service

**Rule.** After a Journal Entry is posted for the first time, one Vehicle Service is created for each of its Journal Items that survives all four of the following skip tests, evaluated in this order:

1. Skip when the item names no vehicle.
2. Skip when the item already carries a Vehicle Service.
3. Skip when the entry's kind is not a **vendor bill**. Vendor credit notes and vendor receipts are skipped, even though the vehicle column is offered on them.
4. Skip when the item's display kind is not a product line. Tax lines, payment-term lines, section headings and notes are skipped.

**Precondition.** The shipped Fleet Service Type named "Vendor Bill" must exist. When it does not, the whole rule is skipped and posting behaves as it does without the bridge.

**The service created.** Its kind is "Vendor Bill"; its vehicle is the item's vehicle; its vendor is the entry's partner; its description is the item's label; its journal item link is the item; its date is today; its cost is the item's debit; its progress stage is New.

**The message logged.** One message is posted to the new service's thread whose body is "Service Vendor Bill: %s", with the single placeholder replaced by a link to the journal entry, rendered as the entry's name.

**Idempotence.** Because of skip test 2, resetting the entry to draft and posting again creates nothing further.

## FLT-044: a contract's state is re-evaluated only when a date is written

**Rule.** The date-driven re-evaluation of a contract's state fires only when the write includes `start_date` or `expiration_date`. Writing any other field, including the state itself, the amounts, the frequency or the vehicle, does not re-evaluate.

**Further conditions inside the re-evaluation.** A contract is considered only when it has a start date and is not cancelled. Contracts failing either test are left exactly as they are.

**Order.** The re-evaluation runs **after** the write has been applied, so it reads the new dates.

## FLT-045: a contract carries at most one renewal reminder

**Rule.** The daily job creates a renewal activity only on contracts that do not already carry an activity of the renewal type. Writing a non-empty expiration date or a non-empty responsible user reschedules the existing activity rather than creating a second one.

**Registration.** The renewal activity type is registered with the platform as an activity type raised by a scheduled job and declared **not** to be removed when the job runs again, which is what allows the "already carries one" test to stay true across runs.

**Consequence.** Completing the reminder without changing the contract's dates causes the job to raise a new one on its next run, because the contract then no longer carries an open activity of that type. That is intended: an unrenewed contract keeps nagging.

## FLT-039: the analysis sets are derived and must not be written

**Rule.** The Fleet Analysis Report and the Fleet Odometer Analysis Report hold no rows of their own. Every read re-derives them from the stored Vehicles, Vehicle Contracts, Vehicle Services and Odometer Readings.

**The access rights say otherwise, and they are wrong.** The matrix grants create, write and delete on the Fleet Odometer Analysis Report to the fleet administrator group, and grants read only on the Fleet Analysis Report. No screen offers any of those write operations, and attempting one against a derived set fails at the storage layer. A rebuild should grant read only on both sets. Recorded here rather than as a separate compatibility finding because the excess permission is unreachable.

## FLT-019: company consistency

**Rule.** Three references are checked against the company of the record that holds them:

1. The `future_driver_id` of a Vehicle must belong to the vehicle's company or to no company.
2. The `vehicle_id` of a Vehicle Contract must belong to the contract's company or to no company.
3. The `manager_id` of a Vehicle is restricted by FLT-031, which includes the same test.

**Message on violation.** The platform's generic cross-company message, which names the record, the field and the two companies involved.

**Not checked.** The `driver_id` of a Vehicle is **not** checked against the company, only restricted in the picker by the form's own condition. A driver of another company supplied through the transport layer is accepted. The `vehicle_id` of a Vehicle Service is not checked against the service's company either.

---

# Part eight: locking and concurrency

## FLT-049: locking and concurrency

The domain declares no explicit locking anywhere. Four places are read-then-write sequences that two concurrent transactions can interleave, and a rebuild has to decide how to handle each:

1. **The never-decrease guard (FLT-002).** The guard reads the vehicle's greatest existing reading and then creates a new one. Two concurrent writes of 10 000 and 10 500 against a stored 9 000 both pass the guard and both create a reading; the derived distance afterwards is 10 500, which is correct, but the 10 000 reading exists and is a legitimate historic record. No corruption results. **Industry-standard default**: no locking is needed.
2. **The daily job.** The job's four passes each read a set and then write it. A user editing a contract's dates at the same moment may have their change overwritten by the job's later pass, or the reverse. The window is one job run per day, and both outcomes leave the contract in a state consistent with its dates. **Industry-standard default**: take no lock; the last writer wins and the next run corrects any disagreement.
3. **The bill posting routine (FLT-041).** Two users posting two entries that both name the same vehicle create two services concurrently. Because the skip test is on the *item*, not on the vehicle, both are correct and no duplicate arises.
4. **The plan-to-change marking (transition T4.5).** Two users queueing the same future driver on two vehicles concurrently both mark the same third vehicle. The write is idempotent: both set the flag to true.

The one place a rebuild **must** take care is the derived distance of a vehicle, which is a search with a limit rather than an aggregate. On a very large reading table that search must be supported by an index on the pair of vehicle and value, or the vehicle list becomes slow. That is a performance consideration and not a correctness one.

---

# Part nine: compatibility findings

Each finding records an observed behaviour that looks like a defect, states what a corrected behaviour would be, and says whether reproducing the observed behaviour matters for compatibility.

## FLT-C01: the default service kind exists only in demonstration data

**Observed.** The `service_type_id` field of a Vehicle Service declares a default that resolves an external identifier belonging to the demonstration data set. On an ordinary installation the record is absent and the lookup, which is written not to fail when the record is missing, yields nothing. The field therefore starts empty and the user must choose a kind.

**Corrected behaviour.** Ship a service-category Fleet Service Type as part of the ordinary installation — a general "Repair and maintenance" kind would match the demonstration record's name — and default to it. Alternatively, remove the default and rely on the required-field rule.

**Compatibility.** Reproducing the observed behaviour matters only in that a rebuild must not fail when the default cannot be resolved.

## FLT-C02: the waiting-list status exists only in demonstration data

**Observed.** Two guards of the driver assignment machine look up a Vehicle Status named as the waiting list. That status is shipped only with the demonstration data. On an ordinary installation both guards take their "the record does not exist" branch, which in one case skips the exclusion of the new-request status as well. See [state-machines.md](state-machines.md) section M4.3.

**Corrected behaviour.** Either ship the waiting-list status as part of the ordinary installation, or restate both guards so that the new-request exclusion applies independently of whether the waiting-list status exists.

**Compatibility.** Reproducing the observed behaviour matters: an installation that has the demonstration data behaves differently from one that does not, and both must be reproducible.

## FLT-C03: the plan-to-change marking at creation time is unreachable

**Observed.** The creation routine of the Vehicle inspects the supplied values for a vehicle kind in order to decide which plan-to-change flag to set on other vehicles. The vehicle kind is a mirrored, read-only field taken from the model; an ordinary creation call never supplies it. The branch therefore never fires from a screen, and a vehicle created with a future driver marks nothing.

**Corrected behaviour.** Read the vehicle kind from the chosen model rather than from the supplied values, so that creating a vehicle with a future driver marks that person's other vehicles exactly as writing a future driver afterwards does.

**Compatibility.** Reproducing the observed behaviour matters, because a rebuild that "fixes" it would set flags an existing installation does not set.

## FLT-C04: the range and the power unit are never copied from the model

**Observed.** The map that drives the model-to-vehicle copy contains seventeen entries, but only fifteen of them are reachable. The vehicle's `vehicle_range` and `power_unit` are plain stored fields with no computed rule, so nothing ever invokes the copy for them. A rule named after the range does exist but is attached to no field.

**Corrected behaviour.** Declare both vehicle fields as computed from the model, stored and editable, exactly like the fifteen that work, so that choosing a model fills the range and the power unit.

**Compatibility.** Reproducing the observed behaviour matters: an existing installation has vehicles whose range is empty although their model declares one.

## FLT-C05: the model counter compares the archive flag against a text

**Observed.** The rule that counts a manufacturer's active models filters models on the archive flag using the literal text `true` instead of the boolean value. The platform coerces the text, so the observed result is the intended one.

**Corrected behaviour.** Compare against the boolean.

**Compatibility.** The observed and corrected behaviours are identical, so a rebuild may implement the corrected form freely.

## FLT-C06: a service's kind is not restricted to the service category

**Observed.** A Vehicle Contract's kind is restricted to Fleet Service Types of the `contract` category, but a Vehicle Service's kind is not restricted at all. A contract-category type such as a lease may be chosen as the kind of a piece of work.

**Corrected behaviour.** Restrict the service's kind to Fleet Service Types of the `service` category, symmetrically with the contract.

**Compatibility.** Reproducing the observed behaviour matters, because existing installations may hold services whose kind is a contract-category type, and a rebuild that restricted the field would make those records unsaveable.

## FLT-C07: a reading's driver Employee mirrors the vehicle, not the reading

**Observed.** The driver Employee shown on an Odometer Reading is mirrored from the reading's **vehicle**, not resolved from the reading's own driver Contact. A reading taken two years ago therefore shows today's driver Employee, while the reading's own driver Contact correctly shows the person who was driving then.

**Corrected behaviour.** Resolve the driver Employee from the reading's own driver Contact, mirroring the way the Driver Assignment Log does it.

**Compatibility.** Reproducing the observed behaviour matters for any report that groups readings by driver Employee.

## FLT-C08: the billed cost rule depends on one amount and stores another

**Observed.** The rule that derives the cost of a billed service declares its dependency on the journal item's **untaxed subtotal in the document currency** but stores the item's **debit in the company currency**. On a bill in the company currency the two are equal; on a bill in another currency they differ, and the stored value is the company-currency one.

**Corrected behaviour.** Declare the dependency on the field actually used, so that the derived value is refreshed by every change that can alter it. The value taken should stay the company-currency debit, because a mixed-currency fleet cost report would otherwise be meaningless.

**Compatibility.** The stored value is the correct one and must be reproduced. Only the dependency list is wrong, and a rebuild should declare it correctly.

## FLT-C09: the weekly recurring frequency is ignored by the cost analysis

**Observed.** A Vehicle Contract offers five recurring frequencies: none, daily, weekly, monthly and yearly. The contract half of the Fleet Analysis Report accounts for the daily, monthly and yearly frequencies. It accounts for the weekly frequency nowhere, so a weekly recurring cost contributes zero to every month.

**Corrected behaviour.** Add a weekly clause that attributes, to each month the contract covers, the recurring cost multiplied by the number of days of the month within the coverage window divided by seven.

**Compatibility.** Reproducing the observed behaviour matters, because a rebuild that added the missing clause would report higher costs than the existing system for the same data.

## FLT-C10: the fuel kind of the cost analysis is carried as plain text

**Observed.** The Fleet Analysis Report carries the vehicle's fuel kind as a text column rather than as a selection. Screens therefore show the stored value, such as `plug_in_hybrid_diesel`, rather than the label "Plug-in Hybrid Diesel". The Fleet Odometer Analysis Report, by contrast, mirrors the field properly and does show labels.

**Corrected behaviour.** Declare the column as a mirror of the vehicle's fuel kind, as the distance analysis does.

**Compatibility.** The change is presentational only; the underlying values are identical.

## FLT-C11: the vehicle-required flag is always false

**Observed.** The `need_vehicle` flag on a Journal Item is computed by a rule that always yields false. Nothing in this repository makes it true, so the vehicle column on a bill line is never mandatory.

**Corrected behaviour.** None is needed in this domain. The flag is deliberately an extension point: a localization whose road-tax or benefit-in-kind rules require a vehicle on certain expense accounts overrides the rule and makes the column mandatory. The finding is recorded so that a rebuild keeps the extension point rather than deleting an apparently dead field.

**Compatibility.** Keeping the field and its always-false default is required, because localizations depend on it.

## FLT-C12: a contract whose expiry precedes its start oscillates

**Observed.** Nothing refuses an expiration date earlier than the start date. Such a contract is moved to `expired` by the second pass of the daily job and back to `futur` by the third pass, in the same run and on every subsequent run, producing two tracking messages per day for ever.

**Corrected behaviour.** Refuse an expiration date strictly earlier than the start date at write time, with a message naming both dates.

**Compatibility.** Reproducing the observed behaviour matters only in that existing data may contain such contracts; a rebuild that adds the refusal must not make them unsaveable.

## FLT-C13: deleting a vehicle leaves journal items pointing at nothing

**Observed.** The vehicle reference on a Journal Item is an optional reference with no deletion rule declared, so deleting a Vehicle removes the row while items continue to reference it. The reference then resolves to nothing and the analysis of costs by vehicle silently loses those amounts.

**Corrected behaviour.** Either restrict the deletion of a Vehicle that is named on any journal item, or declare that deleting the vehicle empties the reference on the items. The first is preferable, because a vehicle whose costs are in the ledger is part of the accounting record.

**Compatibility.** Reproducing the observed behaviour is not desirable; a rebuild should choose the restriction and state the change.

## FLT-C14: the service distance link is written to the whole selection

**Observed.** The rule that creates an Odometer Reading from a service's distance field iterates over the selection, creating one reading per service, but writes the resulting link onto the **whole selection** rather than onto the individual service. Saving several services at once through a multiple-record edit leaves every one of them pointing at the reading created for the last of them, and the readings created for the earlier ones are orphaned.

**Corrected behaviour.** Write the link onto the individual service inside the loop.

**Compatibility.** The observed behaviour is a data defect; a rebuild should implement the corrected form and state the change.

## FLT-C15: the cost-log operation names a screen that does not exist

**Observed.** The Vehicle declares an operation `act_show_log_cost` whose purpose is to open the cost log of one vehicle. It resolves a window action by external identifier — `fleet.fleet_vehicle_costs_action` — that is defined nowhere in the system. No menu, button or view invokes the operation, so it is never reached; a caller reaching it through the transport layer would receive the platform's "record not found" failure.

**Corrected behaviour.** Remove the operation, or point it at the Fleet Analysis Report screen restricted to the vehicle, which is what a cost log of one vehicle now means.

**Compatibility.** Nothing depends on it; a rebuild may omit it.

## FLT-C16: the renewal flags and their search rules can disagree

**Observed.** The two renewal flags on a Vehicle are produced one way when they are read from a record and a different way when they are searched on.

- **Reading** takes the contract with the greatest expiration date among the vehicle's contracts that have an expiration date and are not cancelled, and derives both flags from that one contract's expiry.
- **Searching for overdue** asks whether the vehicle has *any* contract that is running or expired and whose expiry is past, **and** has no contract that is running or incoming and whose expiry is today or later.
- **Searching for due soon** asks whether the vehicle has *any* contract that is running or expired whose expiry is strictly after today and strictly before today plus the alert delay.

A worked divergence: a vehicle has one contract in state `expired` that lapsed five days ago and one contract also in state `expired` that expires in ten days. Reading takes the later one, computes ten days to run, and reports "due soon" and "not overdue". Searching for overdue finds the lapsed contract, finds no running or incoming contract with a future expiry, and returns the vehicle. The list therefore shows a vehicle in the overdue filter whose own record says it is not overdue.

**Corrected behaviour.** Express one of the two in terms of the other. The search form is the more useful of the two, because it treats a replacement contract as resolving the overdue condition, so the read form should be restated to match it.

**Compatibility.** Reproducing both forms exactly is required, because saved filters and list decorations depend on the search form while the record's own colouring depends on the read form.

## FLT-C17: the last distance of a vehicle is the greatest, not the latest

**Observed.** The field labelled "Last Odometer" on a Vehicle is derived by taking the reading with the **greatest value**, ordering by value descending and taking one. It is not the reading with the latest date.

**Consequence.** A mistyped reading of 999 999 permanently pins the vehicle's displayed distance at that value and, through FLT-002, makes every subsequent honest reading unwritable through the vehicle's field. The only remedy is to delete the mistyped reading.

**Corrected behaviour.** Either rename the field to "Highest Odometer", or derive it from the reading with the latest date and, among readings of the same date, the greatest value — which is what the distance analysis does in its second step.

**Compatibility.** Reproducing the observed behaviour matters, because the never-decrease guard is defined against it.

## FLT-C18: the service half of the cost analysis drops months with no service

**Observed.** The service half of the Fleet Analysis Report generates one candidate row per vehicle per month, joins the services of that month, and then filters on conditions that include properties of the joined service — that the service is active and that it is not cancelled. For a month with no service at all, the joined service is absent and both conditions fail, so the candidate row is discarded rather than kept with a cost of zero.

**Consequence.** The service half produces rows only for months in which a vehicle actually had a service. The contract half, whose filter mentions only the vehicle, does produce a row for every month, with a cost of zero where nothing applies.

**Corrected behaviour.** Move the two service conditions into the join so that a month with no service yields a row with a cost of zero, matching the contract half.

**Compatibility.** Reproducing the observed behaviour matters for any chart that counts rows rather than summing costs.

## FLT-C19: the distance analysis shifts every month forward by one

**Observed.** The last step of the distance analysis emits two things: one synthetic row per vehicle carrying the earliest computed month, a distance of zero and a running total of zero; and every computed row with its month shifted **forward by one month**. The synthetic row is not shifted.

**Consequence.** The timeline of a vehicle begins with a zero month, and every real month is labelled one month later than the month whose distance it carries. A distance travelled in March is reported against April.

**Corrected behaviour.** Emit the synthetic row one month **before** the earliest computed month and leave the computed rows unshifted, which produces the same leading zero without displacing the data.

**Compatibility.** Reproducing the observed behaviour matters, because a rebuild that corrected it would move every point of every existing chart by one month.

## FLT-C20: days left conflates expiring today with already overdue

**Observed.** The `days_left` field of a Vehicle Contract yields the number of days to expiry when that number is strictly positive, and **zero** otherwise — that is, zero both on the expiry day and on every day after it. A companion field `expires_today` distinguishes the two, being true only when the difference is exactly zero.

**Consequence.** The contract list colours a row red when `days_left` is zero, `expires_today` is false and the vehicle has no other running contract. That composition is what recovers the "already overdue" case, and a rebuild that simplified `days_left` to a signed difference would break the colouring rule that depends on it.

**Corrected behaviour.** Let `days_left` carry the signed difference — negative when overdue — and derive the colouring from the sign. Keep `expires_today` for the exact-zero case.

**Compatibility.** Reproducing the observed behaviour matters, because the list decoration is expressed in terms of it.

## FLT-C22: the departure option carries a misspelled storage name

**Observed.** The option added to the departure assistant is stored under the name `release_campany_car`, in which the word for the organisation is misspelled. The label shown to the reader is correct.

**Corrected behaviour.** Rename the stored field. The change is safe within this domain, because nothing outside the assistant reads it.

**Compatibility.** The storage name is reproduced here exactly, because an integration that drives the departure assistant through the transport layer must use it character for character.

## FLT-C23: a billed service is dated on the posting day, not the bill date

**Observed.** A Vehicle Service created from a posted vendor bill takes its date from the field's default, which is today. It does not take the bill's invoice date or the entry's accounting date.

**Consequence.** A bill dated in January and posted in March produces a service dated in March, so the cost appears in March in the Fleet Analysis Report while the ledger records it in January or February. The fleet report and the ledger therefore disagree on the period.

**Corrected behaviour.** Set the service's date from the entry's accounting date, which is the date the ledger uses.

**Compatibility.** Reproducing the observed behaviour matters, because correcting it would move existing costs between months in the analysis.

## FLT-C24: assignment entries are closed only at departure

**Observed.** A Driver Assignment Log is opened automatically on every non-empty driver write, but it is closed automatically only by the departure procedure. Replacing a driver leaves the previous entry open; the only prompt is the reminder activity whose note is "Specify the End date of %s".

**Consequence.** A vehicle whose driver changes several times without anybody acting on the reminders accumulates several open entries, and any report of "who was driving on a given date" returns more than one answer.

**Corrected behaviour.** When a driver is replaced, set the end date of the previous open entry of that vehicle to the day before the new start date, and keep the reminder activity only for the case where several entries are open.

**Compatibility.** Reproducing the observed behaviour matters, because a rebuild that closed the entries would change the answers the assignment history gives for existing data.
