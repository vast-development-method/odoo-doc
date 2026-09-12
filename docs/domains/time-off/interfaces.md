# Time Off — Interfaces

Everything that crosses the boundary of the domain: the named operations a client or another domain
invokes, with their inputs, their outputs, their side effects and their errors; the routes reached
from a notification; the screens, described by the fields they show, the buttons they offer and the
guard of each button; the analysis screens; the printable document; the notifications; the import
and export surface; and the contract another domain must honour.

Every operation takes an implicit actor, the authenticated user, and, where a selection is
meaningful, an implicit selection of records. Every error named below is the exact message of
[business-rules.md](business-rules.md).

---

## 1. Named operations on the Time Off Request

| Operation | Inputs | Output | Side effects | Errors |
|---|---|---|---|---|
| Create a request | the field values of one or more requests | the created requests | [workflows.md, chapter 2](workflows.md#2-submit-a-time-off-request), steps 7 to 13: followers, the approval activity, and the automatic approval under the "None needed" ladder | `TOF-020` to `TOF-026`, `TOF-040`, `TOF-046`, `TOF-050` to `TOF-053`, `TOF-071`, `TOF-090`, `TOF-110` |
| Update a request | the field values to change | nothing | Removes the Working Time Exclusion when the state leaves *Approved*; amends its instants when the dates change while the state stays *Approved*; rewrites a written absolute instant into the matching requested date; re-runs the coverage check; subscribes the new employee's login user | `TOF-024`, `TOF-027`, `TOF-028`, `TOF-040`, `TOF-046`, `TOF-050` to `TOF-053`, `TOF-110` |
| Approve a request | an optional flag suppressing the state check | true | [workflows.md, chapter 3](workflows.md#3-approve-a-time-off-request): routes to the first approval or to the validation, stamps the approver, materialises the absence, updates the activities | `TOF-040` to `TOF-043`, `TOF-046`, `TOF-110` |
| Refuse a request | none | true | [workflows.md, section 4.1](workflows.md#41-refuse) | `TOF-040`, `TOF-044` |
| Open the cancellation dialog | none | a dialog descriptor for the Cancel Time Off Wizard, medium size, with the request pre-filled | none | none |
| Cancel a request | a reason | nothing | [workflows.md, section 4.2](workflows.md#42-cancel) | `TOF-045` |
| Send back to approval | none | true | [workflows.md, section 4.3](workflows.md#43-reopen-an-approved-request), applied only to the records whose reopening permission is true | none; records that may not be reopened are silently skipped |
| Delete a request | none | nothing | Archives the Calendar Event, removes the Working Time Exclusion, deletes the timesheet lines and regenerates the public-holiday timesheet lines | `TOF-048` |
| Duplicate a request | optional overrides | the copy | none | `TOF-049` |
| Open the supporting documents | none | a screen descriptor listing the request's attachments as cards with creation disabled, titled "Supporting Documents" | none | none |
| Open the pending allocation requests | none | a screen descriptor listing allocations with the first-approval and second-approval filters pre-applied, titled "Allocation Requests", scoped to the contextual employee when that employee is not the actor's own | none | none |
| Ask for the unusual days | a start date and an optional end date | the set of days on which the contextual employee is not expected to work | none | none |

## 2. Named operations on the Time Off Allocation

| Operation | Inputs | Output | Side effects | Errors |
|---|---|---|---|---|
| Create an allocation | the field values of one or more allocations | the created allocations | Fills the department and the manager from the employee, initialises the accrual cursor, subscribes the followers, schedules the approval activity, and approves immediately under the "None needed" ladder | `TOF-060` to `TOF-062`, `TOF-111` |
| Update an allocation | the field values to change | nothing | Re-initialises the accrual cursor when the allocation kind, the plan, the validity dates or the employee change on a record that is not yet *Approved*; re-measures the approved excess when the amount or the state changes | `TOF-040`, `TOF-062` to `TOF-064`, `TOF-111`, `TOF-112` |
| Approve an allocation | none | true | [workflows.md, chapter 6](workflows.md#6-request-and-approve-a-time-off-allocation), step 5: routes to the first approval or to the validation, stamps the approvers, updates the activities | `TOF-040`, `TOF-041`, `TOF-064` |
| Refuse an allocation | none | true | Writes the state and the first approver, removes the approval activity | `TOF-040`, `TOF-044`, `TOF-064` |
| Delete an allocation | none | nothing | none | `TOF-067` |
| Duplicate an allocation | optional overrides | the copy, in state *To Approve* | none | none |
| Run the accrual | none, an operation on the entity rather than on a record | nothing | Advances every eligible accrual allocation to today | none |
| Simulate the future accrual | a date | the additional amount the allocation will have gained by that date | none; the simulation is discarded | none |

## 3. Named operations on the other entities

### 3.1 Time Off Type

| Operation | Inputs | Output |
|---|---|---|
| Ask for the balance data | a set of employees and an optional evaluation date | for each employee, the list of balance entries described in [calculations.md, chapter 7](calculations.md#7-aggregating-the-balance-for-a-date-and-the-dashboard-payload) |
| Ask for the balance data of a request | an optional evaluation date and a flag saying whether hidden types are included | the same list for the contextual employee, keeping only the types whose allocated amount is not zero |
| Ask whether an accrual allocation exists | none | true when the contextual employee holds at least one approved accrual allocation that has not expired |
| Open the allocations of this type | none | a screen descriptor listing every allocation of the type with the Approved and Currently Valid filters pre-applied |
| Open the requests of this type | none | a screen descriptor listing every request of the type |
| Open the accrual plans of this type | none | a screen descriptor listing the plans restricted to the type |

### 3.2 Employee and Public Employee

| Operation | Inputs | Output |
|---|---|---|
| Ask for the dashboard data | an optional evaluation date | a structure of three members: whether the contextual employee holds an accrual allocation, the balance entries of the visible types, and the number of the employee's allocations still in state *To Approve* |
| Ask for the number of allocation requests | none | the number of allocations of the contextual employee in state *To Approve* |
| Ask for the special days | a start date and an end date | a structure of two members, the Mandatory Days and the public holidays of the window, each as a list of whole-day calendar blocks; with the Indian localization package installed a third member carries the Optional Holidays |
| Ask for the public holidays | a start date and an end date | one block per public holiday: an identifier derived from the record and made negative, colour index zero, the whole day in the employee's time zone, the whole-day marker, and the holiday name as the title |
| Ask for the mandatory days as blocks | a start date and an end date | one block per Mandatory Day: a negative identifier, the record's colour index, the whole day, the whole-day marker, and the record name as the title |
| Ask for the mandatory days as a map | a start date and an end date | a map from each forbidden day, written as a date, to the colour index of the rule that forbids it |
| Open the time off dashboard | none | a screen descriptor opening the year calendar filtered on the selected employees |
| Open the personal absence calendar | none | a screen descriptor opening the calendar of one public employee profile, with the current-year filter pre-applied and employee names hidden |
| Ask for the overtime data | none | for each employee: the compensable extra hours, the non-compensable extra hours and the unspent compensable extra hours. Added by the attendance companion package |

### 3.3 Department, Working Schedule and Work Entry

| Operation | Inputs | Output |
|---|---|---|
| Open the department's requests | none | the request screen scoped to the department, with the first-approval filter, the active-employee filter and the active-absence filter pre-applied and employee names hidden |
| Open the department's allocations | none | the allocation screen scoped to the department, limited to state *To Approve*, with the second-approval filter pre-applied |
| Open the public holidays of a schedule | none | the Working Time Exclusions with no resource, filtered on that schedule |
| Approve the absence behind a work entry | none | approves the absence the work entry points at, when there is one |
| Refuse the absence behind a work entry | none | refuses that absence, with elevated rights |
| Ask for the absence durations between two dates | an employee, a start date and an end date | a map from Time Off Type to the total duration of the validated absence work entries of that employee in the window, from the start date at 00:00:00 to the end date at 23:59:59 |

### 3.4 The reporting projections

| Operation | Inputs | Output |
|---|---|---|
| Open the underlying record | none | the form of the request or of the allocation behind a Time Off Analysis row |
| Open the balance analysis | none | the Time Off Balance by Employee and Type pivot, scoped to the selected employees and excluding cancelled rows when launched from an employee selection, and scoped to the allowed companies otherwise; the empty state offers a link to the new-allocation screen |
| Approve from the calendar | none | approves the request behind a Time Off Calendar Report row. Error `TOF-097` |
| Refuse from the calendar | none | refuses the request behind a Time Off Calendar Report row. Error `TOF-097` |
| Print the sixty-day summary | a start date, a set of employees and a state filter | the printed document of [chapter 7](#7-reports-and-printable-documents) |

---

## 4. Routes

Five routes exist, all reached by following a link embedded in a notification message. All require
an authenticated session, all accept only the retrieval method, and all take two parameters: the
record identifier and a signed token.

| Route path | Purpose | Behaviour |
|---|---|---|
| `/leave/approve` | First approval of a request | Verifies the token against the record; on success runs the approve operation; on any error redirects to a generic fallback page; in every case the user ends on the record. |
| `/leave/validate` | Validation of a request | Identical to the previous one: it runs the same approve operation, which routes according to the ladder and the current state. |
| `/leave/refuse` | Refusal of a request | Verifies the token; on success runs the refuse operation; the same error and redirect behaviour. |
| `/allocation/validate` | Approval of an allocation | Verifies the token; on success runs the allocation approve operation; the same error and redirect behaviour. |
| `/allocation/refuse` | Refusal of an allocation | Verifies the token; on success runs the allocation refuse operation; the same error and redirect behaviour. |

The token is the per-record signature that the
[messaging and activities](../messaging-and-activities/README.md) domain generates for action
links. A mismatched token is treated exactly like a failed operation: the user lands on the fallback
page and the record is unchanged. No error is ever shown to the follower of such a link, so a
follower who is not allowed to approve simply sees the record unchanged.

---

## 5. Screens

### 5.1 The personal dashboard

A year calendar of the actor's own requests, opened by the Dashboard menu entry and reachable at the
application path `time-off`. Records shown: the actor's own requests, restricted to the employees of
the allowed companies. Default filter: the current year. Each block is coloured by the type's colour
index, hatched when the state is neither *Approved* nor *Refused*, struck through when *Refused*, and
hides the time of day; non-working days are shaded, and the public holidays and the Mandatory Days
are painted from the special-days operation.

Around the calendar the screen shows one tile per type that requires an allocation and is not hidden
from the dashboard. Each tile carries the type name, the cover image, the available amount, the
allocated amount, the amounts requested and approved, whether the type allows a negative balance and
by how much, the expiry notice when one exists, and a marker when the figures shown are not the
figures of today. A date picker moves the evaluation date, which recomputes every tile including the
future accrual. A counter shows how many allocation requests are still awaiting approval and opens
them.

Buttons: New Time Off, which opens the compact request dialog; New Allocation Request, which opens
the compact allocation dialog.

### 5.2 The request form

Header buttons, each shown only when its guard holds:

| Button | Guard | Operation |
|---|---|---|
| Approve | the can-approve flag is true and the record is stored | approve |
| Validate | the can-validate flag is true, the can-approve flag is false, and the record is stored | approve |
| Back to Approval | the reopening permission is true and the record is stored | send back to approval |
| Refuse | the can-refuse flag is true and the record is stored | refuse |
| Cancel | the cancellation permission is true and the record is stored | open the cancellation dialog |

Status bar: *To Approve* then *Second Approval* then *Approved* when the ladder is "By Employee's
Approver and Time Off Officer", and *To Approve* then *Approved* otherwise.

Body, in order:

1. the overlap warning, as a red alert, when it is not empty;
2. the rounding notice, as a blue alert, when it is not empty;
3. the Time Off Type, read-only once the state is *Cancelled*, *Refused*, *Approved* or *Second
   Approval*;
4. the requested period as a date range, labelled "Dates" for a day-based type and "Date" otherwise,
   read-only unless the state is *To Approve*, with the duration text printed beside it in
   parentheses;
5. for a half-day-based type, the start half and the end half, both required and read-only unless
   the state is *To Approve*;
6. for an hour-based type, the start hour and the end hour, both required, read-only once
   *Approved*;
7. for an hour-based type whose time zone differs from the reader's, a blue notice reading "The
   employee has a different timezone than yours! Here dates and times are displayed in the
   employee's work timezone" followed by the time zone name and "For your reference, the equivalent
   times in your timezone are displayed below.", then a read-only echo of the resolved instants
   labelled From and To;
8. the description, as free text with the placeholder "No description provided", read-only unless
   the state is *To Approve*;
9. the attachment widget, shown only while the state is *To Approve*;
10. on the right, the balance panel for the chosen employee and type;
11. a Cancelled ribbon when the state is *Cancelled*.

The approver variant additionally shows the employee as an avatar picker, restricted to the Time Off
Responsible group and read-only once the state is *Cancelled*, *Refused*, *Approved* or *Second
Approval*; the company in a multi-company setting; the read-only department; and the computed display
name as a title. The compact dialog variant hides the header, the right panel, the discussion thread
and the attachment preview, shows the employee read-only, and adds Approved and Refused ribbons.

### 5.3 The request list and cards

**List columns**: employee as an avatar picker, department hidden by default, Time Off Type in bold,
absolute start, absolute end, duration text labelled Duration, description hidden by default, and the
state as a coloured badge — amber for *To Approve* and *Second Approval*, green for *Approved*, red
for *Refused*. Header buttons: New Group Time Off, visible to the Time Off Responsible group, which
opens the multi-employee request dialog; Approve; Refuse. Row buttons: Approve when the can-approve
flag holds; Validate when the can-validate flag holds and the can-approve flag does not; Refuse when
the can-refuse flag holds. The employee, department, type, start and end columns become read-only as
soon as the state is *Cancelled*, *Refused*, *Approved* or *Second Approval*.

**Cards**: one card per request showing the employee avatar and name, the type, the requested period
rendered according to the request unit — a single date for a one-day request, a date range for a
multi-day request, the half labels for a half-day request, the hour range for an hour-based request
— the duration text labelled Amount, and, for a type that requires an allocation, the available
amount over the allocated amount labelled Current balance. A ribbon shows the state: To Approve, To
Validate, Cancelled, Refused or Approved. Card buttons: Approve, Validate and Refuse under the same
guards as the list, and a paperclip badge opening the supporting documents when at least one
attachment exists.

### 5.4 The request search panel

Free-text fields: employee, Time Off Type, description, the user whose activities are shown, and the
activity type.

| Filter | Condition |
|---|---|
| Waiting For Me, shown to the Time Off Responsible group but not to Officers | the state is `confirm` **and** ( the employee's login user is not the reader **or** ( it is the reader **and** the employee's Time Off Approver is the reader ) ) |
| Waiting For Me, shown to Officers | the state is `confirm` or `validate1` **and** ( the employee's login user is not the reader **or** ( the state is `confirm` **and** the request approval ladder is "By Time Off Officer" ) **or** the state is `validate1` ) |
| First Approval | the state is `confirm` |
| Second Approval | the state is `validate1` |
| To Approve | the state is `confirm` or `validate1` |
| Approved | the state is `validate` |
| Cancelled | the state is `cancel` |
| Refused | the state is `refuse` |
| My Time Off | the employee's login user is the reader |
| My Team | the employee's Time Off Approver is the reader **or** the employee's login user is the reader |
| My Department | the employee belongs to the reader's department |
| Start Date | a period filter on the absolute start, defaulting to the current year |
| My Activities, Late Activities, Today Activities, Future Activities | the ordinary activity filters, hidden from the menu |

Groupings: Employee, Type, Status and Date, the last by the absolute start. A side panel offers
Status and Department as facets.

### 5.5 The approval screens

"All Time Off", at the application path `time-off-approval`, opens the cards, the list, the form, the
calendar and the activity view of every request of the allowed companies, with both Waiting For Me
filters and the current year pre-applied and with employee names hidden.

"Allocations" opens the cards, the list, the form and the activity view of every allocation with My
Team and First Approval pre-applied.

The Overview screen, at the application path `time-off-overview`, opens the company calendar built on
the Time Off Calendar Report, restricted to active employees, with My Team, the current year,
Approved and First Approval pre-applied and with employee names hidden. Each block offers Approve and
Refuse to a reader who passes the test of rule `TOF-097`.

### 5.6 The allocation form, list and search panel

**Form.** Header buttons Approve, Validate and Refuse under the same guards as on a request. Status
bar: *To Approve* then *Second Approval* then *Approved* for a two-approval ladder, and *To Approve*
then *Approved* otherwise. Body: the description as a title while the state is *To Approve*,
read-only and regenerated from the type and the amount, and the description with its validity
afterwards; the Time Off Type, read-only once *Approved*; the Accrual Plan, shown and required only
for an accrual allocation and read-only unless the reader is an Officer and the state is not
*Approved*; the validity period, shown only to an Officer, labelled "Validity Period" for a regular
allocation in state *To Approve* and "Start Date" together with "Run until" for an accrual
allocation, with the placeholder "No Limit" and the read-only note "No limit" when there is no end
date; a warning reading "The allocated days cannot be used, because the allocation is set to finish
in the past." shown before saving when the chosen end date is already past; the amount, labelled
Allocation, shown in days or in hours according to the request unit and read-only outside *To
Approve* for a reader who is not an Officer; and the reasons, with the placeholder "Add a reason...",
read-only outside *To Approve*.

**List.** Columns: employee as an avatar picker, greyed when the employee is archived; department
hidden by default; Time Off Type in bold; title hidden by default; the amount as the duration text;
validity start and validity end, both hidden by default; the allocation kind; the Accrual Plan;
reasons hidden by default; and the state as a coloured badge. Row buttons: Approve, Validate, Refuse.
Header buttons: New Group Allocation, visible to the Time Off Responsible group; Approve; Refuse.

**Search panel.** Filters: First Approval, matching the states `confirm` and `validate1`; Second
Approval; Approved; Refused; Currently Valid, whose validity window contains today; Unread Messages;
My Team; My Department, offered to an Administrator only; My Allocations; Validity Start as a period
filter; and the ordinary activity filters. Groupings: Employee, Type, Allocation Type and Status.

### 5.7 The configuration screens

**Time Off Types.** A list ordered by a draggable sequence handle showing the display name, the
request unit, the request ladder, the notified officers, the allocation requirement, the allocation
ladder, the employee-request flag, the colour, the company and the country, the last two read-only
once the type is used. A card view shows the name, the maximum allowed and the amount already taken.
The form is laid out as: the name as a title; a first block with the request unit as radio buttons,
the "Count as" selector and the notified officers with the placeholder "Nobody to notify"; a second
block with the company, placeholder "Visible to all", and the country; then two side-by-side blocks,
"Time Off Requests" holding the request ladder as radio buttons and "Allocation Requests" holding the
allocation requirement, the employee-request flag and the allocation ladder; then a "Configuration"
block with the public-holiday flag, the dashboard visibility, the supporting-document requirement
labelled "Require Supporting Document", the accrual eligibility, the request-stacking flag offered
only on a Worked Time type, and the calendar entry flag; then a "Negative Cap" block, shown only when
an allocation is required, holding the negative flag and, when it is on, the words "up to", the
maximum excess amount and the unit word; and finally a "Display Option" block with the colour picker
and the cover image chooser. Three statistic buttons open the allocations, the requests and the plans
of the type. An Archived ribbon appears when the type is archived.

**Accrual Plans.** A list and a form. The form holds the title, the company in a multi-company
setting, the restricted Time Off Type, the accrual timing, the worked-time basis, the carry-over
anchor with its month and day for the custom anchor, the milestone transition mode shown only when
there is more than one milestone, and the milestone cards. A milestone card is edited in a dialog
carrying three sections: the accrual level options — the rate, the rate unit, the frequency and its
anchors, and the milestone start; the carry-over options — the action, the limit and the validity;
and the cap options — the yearly cap and the running cap. The dialog offers Save and Save and New.

**Public Holidays.** The Working Time Exclusions with no resource, in a list and a form, with the
date filter pre-applied. The same records are reachable from a working schedule.

**Mandatory Days.** A list, a form and a search panel filtering by date, department, job position and
company.

**Optional Holidays.** With the Indian localization package installed, a list and a form of the
Optional Holiday records, restricted to a company whose country is India.

### 5.8 The dialogs

| Dialog | Fields | Buttons |
|---|---|---|
| Multiple Requests | Time Off Type, allocation mode as radio buttons, the population field matching the mode, the start and end dates, the description | Generate Time Off, Discard |
| New Group Allocation | Time Off Type, the amount labelled Allocation, allocation mode, the population field, the allocation kind, the Accrual Plan when the kind is accrual, the validity dates, the reasons and the generated description | Allocate Time Off, Discard |
| Cancel Time Off | the reason | Cancel Time Off, Discard |
| Time Off Summary | the start date, the state selector | Print, Cancel |

---

## 6. Analysis screens

| Screen | Views | Default groupings and filters | Measures |
|---|---|---|---|
| Time Off by Employee | list, graph, pivot, calendar and form over the Time Off Request | grouped by employee and by type, filtered on the start date, on *To Approve* and on *Approved*, excluding *Cancelled* | duration in days |
| Time Off by Type | graph, list and pivot over the Time Off Analysis projection | grouped by type, filtered on *To Approve* and on *Approved* | number of days, number of hours |
| Time Off Analysis from a department | graph and pivot over the Time Off Analysis projection | scoped to the department, grouped by employee, by type and by month of the start instant | number of days |
| Balance | pivot over the Time Off Balance by Employee and Type projection | filtered on the current year, the company and the employee, with group expansion on | number of days and number of hours, split by Taken, Left and Planned |
| Time Off Ledger | list, pivot and form over the absence ledger, at the application path `absence-report` | grouped by date and by employee, filtered on a negative difference and on the last two months | expected hours, worked hours, approved absence hours, difference |

The default request graph plots the number of days measured by employee, by type and by absolute
start.

---

## 7. Reports and printable documents

### 7.1 There is exactly one printable document

The domain prints one document, the sixty-day summary. Everything else is read on screen or exported
through the platform's generic table export.

### 7.2 The sixty-day summary

**Trigger**: the Time Off Summary dialog, launched from a selection of employees.

**Inputs**: a start date, the selected employees, and a state filter whose values are Approved,
Confirmed and Both Approved and Confirmed. At print time the active records of the calling context
override the employee field of the dialog.

**Header**: the chosen start date, the date fifty-nine days later, and the state label rendered as
"Approved", "Confirmed" or "Confirmed and Approved".

**Content**: a month band naming each month inside the window together with the count of its days in
the window; a day band with the abbreviated weekday name and the day number of each of the sixty
days, shaded for Saturdays and Sundays; and one row per employee holding the employee name, sixty
cells and a row total.

**Cell colouring**: a weekend cell is shaded; a cell covered by an absence in the chosen states is
painted with the colour of that absence's type, taken from the palette of
[calculations.md, section 12.5](calculations.md#125-the-printed-sixty-day-summary). Later absences
overwrite earlier ones in the same cell.

**Grouping**: when the document is launched from a department selection the rows are grouped by
department and each group repeats the day band.

**Totals**: the row total is the sum of the day figures of the absences that intersect the window,
including the part of an absence that falls outside it.

**Legend**: one entry per Time Off Type present in the window, with its colour and its name.

**Page format**: the shipped "Time Off Summary" format of
[configuration.md, chapter 12](configuration.md#12-the-printed-page-format).

### 7.3 The four reporting projections

The four read-only projections — Time Off Analysis, Time Off Calendar Report, Time Off Balance by
Employee and Type, and the absence ledger — are described as entities in
[entities.md, chapter 10](entities.md#10-reporting-projections) and their row-building rules in
[calculations.md, chapter 12](calculations.md#12-report-row-building). They carry no write
operations.

---

## 8. Notifications, activities and messages

The complete catalogue of messages produced by this domain, with the trigger and the recipients of
each, is the table of
[workflows.md, section 5.4](workflows.md#54-discussion-thread-notifications). In addition:

- every state change of a Time Off Request is tracked in its discussion thread, together with every
  change of the Time Off Type, the employee, the two absolute instants and the two duration figures;
- every state change of a Time Off Allocation is tracked, together with the employee, the validity
  dates, the day amount and the Accrual Plan;
- reaching *Approved* posts under the notification subtype configured on the type, falling back to
  the shipped subtype of the entity;
- the notification sent to an approver embeds the action links of [chapter 4](#4-routes);
- the four approval activity types and the four notification subtypes are the shipped records of
  [configuration.md, chapter 8](configuration.md#8-shipped-notification-subtypes-and-activity-types).

---

## 9. Import and export

The domain adds no import or export operation of its own. Requests, allocations, types, plans,
milestones and mandatory days are imported and exported through the platform's generic record import
and table export, with three behaviours that a rebuild must reproduce:

1. A creation that arrives through the import path suppresses the scheduling of the approval
   activity, both on a request and on an allocation, so that a bulk load does not flood the approvers
   with to-do items.
2. A write of the allocation requirement of a Time Off Type whose value equals the value already
   stored is dropped before the validation of rule `TOF-034`, so that reloading a shipped catalogue
   never trips it.
3. The description of a request is exported through the masked field, so that an export made by a
   reader who is not an Officer carries the five characters `*****` rather than the private text.

---

## 10. The contract other domains depend on

- The balance operations of [section 3.1](#31-time-off-type) are the only sanctioned way for another
  domain to ask how much entitlement an employee has left. They must be implemented exactly as
  [calculations.md, chapter 6](calculations.md#6-the-balance-consumption-algorithm) and
  [chapter 7](calculations.md#7-aggregating-the-balance-for-a-date-and-the-dashboard-payload)
  describe, provisional figures included, because the request form, the type selector, the coverage
  check and the daily cancellation job all read them.
- The Working Time Exclusion is the only sanctioned way for another domain to learn that an employee
  is absent. A rebuild must write it at validation and remove it on refusal, cancellation, reopening
  and deletion; otherwise availability, payroll and planning silently diverge.
- The five routes of [chapter 4](#4-routes) are the only way an approver can act without opening the
  application. A rebuild that drops them must keep the notification usable by another means.
- The work entry link and the absence link on a timesheet line are the two back-references that let
  the payroll and timesheet domains find the absence behind a generated record; both are protected
  by the guards of
  [business-rules.md, chapter 10](business-rules.md#10-timesheet-lines-and-payroll-work-entries).

---

## 11. Reconciliation notes

1. **The operation catalogue.** Only one of the two source drafts carried it. It is kept in full,
   with the invented operation names replaced by descriptions in words, because an operation name is
   not a contractual identifier: what a rebuild must reproduce is the input, the output, the side
   effect and the error, all of which are listed.
2. **The routes.** Both drafts agreed on the five paths. The paths themselves are reproduced in code
   font because they are part of the integration contract; everything else is prose.
3. **Where the notification catalogue lives.** One draft carried the full table here, the other in
   its workflow document. It lives in [workflows.md](workflows.md#54-discussion-thread-notifications)
   and is referenced from here, so that a message is specified once.
4. **Import and export.** Neither draft stated the import behaviour explicitly, although both
   recorded the two suppressions it relies on. [Chapter 9](#9-import-and-export) gathers them.
