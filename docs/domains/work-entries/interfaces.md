# Work Entries — Interfaces

Every menu, window action, view, named operation, printable document, client component, integration
point and import or export path of the domain.

---

## 1. Where the domain appears

The domain contributes **no menu item of its own**. That is deliberate and it is the first thing a
rebuild has to notice: the day book is reached from four places, and a rebuild that adds a menu
changes the navigation of the whole human resources application.

| Way in | Who sees it | What it opens |
|---|---|---|
| The "Work Entries" button on the employee form | Human Resources Administrator, and only when the employee has at least one work entry in any state | The work entries of that employee, calendar first |
| The navigation path `work-entries` | Anybody with read access who has the address | The work entry window action |
| A payroll capability's own menu | Payroll users | The same window actions, bound into that capability's menu tree |
| The conflict window action, from a dashboard tile or an automation | Human Resources Officer | The work entries filtered to the conflict state |

The button on the employee form is hidden entirely when the employee has no work entry at all,
which is decided by an existence indicator computed with one query for a whole list of employees.

---

## 2. Window actions

| Name, reproduced | Entity | Views offered, in order | Default filter | Navigation path |
|---|---|---|---|---|
| "Work Entry" | Work Entry | list, form, pivot | active employees only | `work-entries` |
| "Work Entry" (the conflict variant) | Work Entry | list, form, pivot | the conflict state | none |
| "Work Entry Types" | Work Entry Type | list, kanban, form | none | none |
| "Work Entry Regeneration" | Work Entry Regeneration Wizard | form, opened as a dialogue | none | none |
| "(employee display name) work entries" | Work Entry | calendar, list, form | the employee of the button | `work-entries` |

The first action carries an empty-state text of two paragraphs: "No data to display" and "Try to add
some records, or make sure that there is no active filter in the search bar." The third carries the
empty-state text "Create a new work entry type".

The last action is built by an operation on the employee rather than stored, so that it can carry the
employee as the default of new records and, optionally, an initial date on which the calendar should
open.

---

## 3. Views of the Work Entry

### 3.1 The list

| Column | Shown by default |
|---|---|
| Date | yes |
| Employee, with avatar | yes |
| Work entry kind, with its colour and display code | yes |
| Duration, rendered as hours and minutes | yes |
| State, as a coloured badge: informational for `draft`, success for `validated`, warning for `conflict` | yes |
| Description | no, available as an optional column |
| Payroll code | no, available as an optional column |
| External code | no, available as an optional column |

The list supports editing several selected rows at once, and shows sample data when empty.

### 3.2 The form

A header carrying the state as a status bar. Two status bars are declared and exactly one is shown:
the ordinary one, offering `draft` and `validated` and hidden while the state is `conflict`; and a
read-only one, offering `draft`, `validated` and `conflict` and shown only while the state is
`conflict`.

Two notices sit above the sheet:

- While the state is `validated`: "Note: Validated work entries cannot be modified."
- While the state is `conflict`, one of two texts: "This work entry cannot be validated. The work
  entry type is undefined." when the entry has no kind, and "The amount of work on the day should not
  exceed 24 hours." when it has one.

The sheet carries, on the left, the description with the placeholder "Additional Description...", the
work entry kind rendered by the domain's own kind component, and the employee with an avatar; on the
right, the date and the duration rendered as hours and minutes with the suffix "Hours". Every one of
those becomes read-only while the state is `validated`.

A second, derived form is used inside dialogues opened from the calendar. It additionally locks the
employee, the date and the company, and hides the state entirely.

### 3.3 The pivot report

Rows by employee, columns by work entry kind, one measure: the duration, rendered as hours and
minutes. It shows sample data when empty. This is the report a payroll administrator uses to check a
month before closing it.

### 3.4 The search view

| Searchable field | Note |
|---|---|
| Employee | |
| Department | the stored mirror, so no join is needed |
| Work entry kind | rendered by the kind component |
| Description | |

| Filter | Condition |
|---|---|
| "Draft" | the state is `draft` |
| "Validated" | the state is `validated` |
| "Conflicting" | the state is `conflict` |
| "Date" | a date filter on the date |
| "Current Month" | the date is not earlier than the first day of the current month and earlier than the first day of the following month |
| "Active" | the employee is active |
| "Archived" | the employee is archived |

| Grouping | By |
|---|---|
| "Employee" | employee |
| "Department" | department |
| "Type" | work entry kind |
| "Date" | date |

The "Current Month" filter is expressed against the **user's** reckoning of today, so in a zone ahead
of the universal scale the last day of the month is included and the last day of the previous month is
not. That behaviour is deliberate and is covered by a test.

### 3.5 The contextual operation on the list and the form

One contextual operation is offered on both, named "Set to Draft", at order fifty. Its intent is to
put the selection back into the `draft` state.

In the specified system the operation names an implementation that no package provides, so invoking it
fails. This is recorded as a **compatibility finding**; a corrected behaviour would write the state
`draft` onto the selection, which by the coupling of
[state-machines.md, section 1.2](state-machines.md#12-the-coupling-with-the-archived-flag) also
unarchives it, and would leave validated entries alone.

---

## 4. The work entry calendar and its data contract

The calendar is the domain's principal interface and it is not an ordinary calendar. Its contract is
set out here in full.

### 4.1 The declaration

| Property | Value | Consequence |
|---|---|---|
| Start field and stop field | both the **date** | Every event is a whole-day event; a work entry has no clock times |
| Scale | month only, opening in month | There is no week or day scale, because there is nothing to place within a day |
| Month overflow | off | The leading and trailing days of neighbouring months are not drawn |
| Quick create | off | A new entry must go through the multiple-creation form, so that it carries a kind and a duration |
| Colour source | the kind's colour, mirrored onto the entry | Each kind is visually distinct |
| Events per cell | at most nine before the cell collapses | |
| Date picker | hidden | |
| Unusual days | shown | See [4.4](#44-unusual-days) |
| Multiple-creation form | a dedicated one, described in [5.2](#52-the-multiple-creation-form) | |

The fields the calendar reads for each event are the state, the description — shown only when it is
not empty — the duration, the display code, the employee and the kind.

### 4.2 The generation call the calendar makes

Every time the calendar's range changes, and before the events are read, the client calls the
employee-level generation entry point with:

1. the employee of the view, taken from the view's own default employee;
2. the first date of the visible range;
3. the last date of the visible range;
4. no force flag.

The call is made whether or not the range has been generated before; the markers make it cheap and
idempotent. A rebuild that omits the call will show an empty calendar for any month the daily job has
not yet reached.

After that call the client reads two things in parallel: the events themselves, and the acting user's
six most recently used kinds — obtained by grouping the work entries the user created in the last
three months by kind and by day, newest first, limited to six — which become the shortcut buttons of
the multiple-selection toolbar. The result is sorted by display code where there is one and by name
otherwise.

### 4.3 Per-event behaviour

| Situation | Behaviour |
|---|---|
| The entry is validated | The event is not draggable, not resizable, not openable and not deletable; a padlock is drawn in its top right corner; its popover carries the text "You cannot edit a validated work entry" |
| The entry's duration is at least one hour | The popover offers "Split", which opens the split dialogue of [workflows.md, chapter 10](workflows.md#10-splitting-a-work-entry) |
| The entry's generation source differs from the source now set on its version | The event is drawn with a mismatch decoration, driven by the unstored mirror of the version's source |
| Any entry | The popover's delete action is styled as a destructive action and pushed to the right |

### 4.4 Unusual days

The calendar greys out the days on which work is not expected. The set is obtained by a named
operation on the Work Entry, which takes a first and a last date, turns them into instants at the
start and the end of the day on the universal time scale, and asks the **acting company's own default
working schedule** for its unusual days over that window, passing the empty company filter.

Two consequences, both observed:

- The greying reflects the **company's** schedule, not the employee's. An employee working Mondays and
  Wednesdays on a company whose schedule is Monday to Friday sees Tuesday and Thursday drawn as
  ordinary working days, with no entry in them.
- Because the company filter passed is empty, exclusions of every company are considered.

Both are recorded as a **compatibility finding**; a corrected behaviour would resolve the schedule of
the employee the calendar is showing and pass that employee's company.

### 4.5 The control-bar button

One button is added to the calendar's control bar, labelled "Reset". It opens the regeneration wizard
with the employees currently visible in the calendar and the visible range pre-filled.

---

## 5. Multiple creation, multiple selection and quick replacement

### 5.1 Selecting a block of days

Dragging across day cells selects them. The domain extends the platform's multiple-selection toolbar
rather than replacing it.

### 5.2 The multiple-creation form

A dedicated small form, used both by the "Set" button and by the "Replace" button. It carries the work
entry kind rendered by the kind component, the duration in hours with the literal suffix "hours", and
the description with the placeholder "Additional Description...". The employee, the colour and the
display code are carried invisibly so that the created rows are complete.

### 5.3 The multiple-selection buttons

| Button | Label or icon | Effect |
|---|---|---|
| Reset | a circular arrow, tooltip "Reset Selected Work Entries" | Hands the selected days to the regeneration operation in its slot mode |
| Set | the word "Set" | Opens the multiple-creation form and creates one entry per selected day |
| Replace | the word "Replace", inside that form | Creates one entry per selected day and then deletes the entries that were there before |
| Delete | the platform's own delete button, tooltip "Delete" | Deletes the selected non-validated entries |
| One per recent kind | the kind's display code, or its name when it has none, coloured by the kind's colour, tooltip "Replace by " followed by the kind's name | Performs the quick replacement of [5.4](#54-the-quick-replacement) |

Days holding a validated entry are removed from the selection before any of these runs, and validated
entries are excluded from every deletion.

### 5.4 The quick replacement

A shortcut button carries a kind and a duration of minus one, which is the marker meaning *keep each
day's own total*. The procedure is:

1. For each selected day, total the durations of that day's selected entries.
2. If that total is greater than zero, queue one new entry for that day carrying the chosen kind and
   that total.
3. If the day holds nothing, run a forced generation of that single day for the employee and remember
   the first entry it produced.
4. Write the chosen kind onto every entry remembered in step 3.
5. Create every entry queued in step 2, in one operation.
6. If at least one entry was created **and** the selection was not empty, delete every selected entry
   — across the whole selection, not day by day.
7. Reload the calendar.

The two branches exist because a day with no entries has no duration to preserve, and generating it is
the only way to discover what the schedule prescribed. Note that step 6 deletes the whole selection
rather than only the days that were replaced; a day whose entries were all validated is never in the
selection, because validated days are filtered out before any of this runs.

### 5.5 The reset

The reset hands the calendar's selected days to the regeneration operation as a list of pairs of an
employee and a date, together with the identifiers of the non-validated entries in those days. The
operation collapses the days into runs of consecutive dates and issues one forced generation per run;
the three regeneration guards do not fire. See
[calculations.md, section 15.5](calculations.md#155-collapsing-selected-days-into-runs).

---

## 6. Views of the Work Entry Type

### 6.1 The list

Name, display code, payroll code, colour as a colour picker, and the country as an optional hidden
column.

### 6.2 The form

The name as the title, with the placeholder "Work Entry Type Name". A ribbon reading "Archived" when
the kind is archived. On the left: the payroll code, the display code, the external code, the order —
visible only to the technical-features group — and the colour as a colour picker. On the right: the country
with creation and opening suppressed, the pay rate rendered as a percentage, and the extra-hours flag
labelled "Added to Monthly Pay". Below, a group headed "Time Off Options" into which the bridging
packages insert their own fields.

### 6.3 The kanban view

One card per kind, highlighted in the kind's colour, showing the name in bold and, in muted text, the
payroll code followed by the display code in parentheses when there is one. The card menu offers the
colour picker.

### 6.4 The search view

One search field matching the name **or** the payroll code, and one filter "Archived" on the archived
flag.

---

## 7. The regeneration wizard form

A dialogue titled "Regenerate Employee Work Entries", containing:

1. The employees, as avatars, restricted to employees of the acting companies that have at least one
   version, and rendered by a component that draws in red every employee listed in the validated-entry
   set.
2. The period, as a single date range control labelled "Period" spanning the from-date and the
   to-date.
3. Two informational lines, each shown only when its message is not empty, carrying the earliest and
   the latest available date messages of
   [calculations.md, section 15.3](calculations.md#153-the-interactive-clamping).
4. A permanent warning: "Warning: The work entry regeneration will delete all manual changes on the
   selected period."
5. A conditional warning, shown when the criteria are complete and at least one employee is in the
   validated set: "Employees in red will be skipped because they have at least one validated work
   entry."
6. A footer with three buttons: "Regenerate Work Entries" as the confirming action, shown only while
   the wizard is valid; a disabled button of the same label shown while it is not; and "Cancel".

The confirming button carries the shortcut key `q` and the cancel button the shortcut key `x`.

---

## 8. Named operations

These are the operations other domains, automations and clients invoke. Each is listed with what it
takes and what it returns.

| Operation | On | Takes | Returns | Notes |
|---|---|---|---|---|
| Generate work entries for a period | Employee | a first date, a last date, a force flag | the entries created | Called on a set of employees, or on none at all, in which case every employee is considered |
| Generate work entries for a period | Employee Version | the same | the entries created | Groups by company and zone, converts the period into a window, runs elevated |
| Open work entries | Employee | optionally an initial date | a window action | Used by the employee form's button |
| Validate | Work Entry | nothing | true when everything validated, false when a conflict condition marked something | The payroll entry point |
| Split | Work Entry, exactly one | a duration, a kind and a description | the identifier of the new entry | Guards in rule [`WKE-065`](business-rules.md#13-everything-else) |
| Unusual days | Work Entry | a first date and, optionally, a last date | a map of date to whether the day is unusual | Called by the calendar |
| Regenerate work entries | Work Entry Regeneration Wizard | optionally a list of day slots and a list of entry identifiers | nothing | Two modes; see [calculations.md, section 15.5](calculations.md#155-collapsing-selected-days-into-runs) |
| Approve the linked absence | Work Entry, exactly one | nothing | nothing | Forwards to the absence request's own approval; not elevated |
| Refuse the linked absence | Work Entry, exactly one | nothing | nothing | Forwards to the absence request's own refusal; elevated |
| Absence hours per absence kind | Work Entry | an employee, a first instant, a last instant | a map of absence kind to total hours | Widens the instants to whole days; counts only non-cancelled entries linked to a validated request |
| Has static work entries | Employee Version, exactly one | nothing | true when the generation source is `calendar` | Used by the engine and by the daily job |
| Remove work entries outside the contract period | Employee Version | nothing | nothing | Deletes, and retracts the markers |
| Cancel work entries | Employee Version | nothing | nothing | Deletes the non-validated entries of the version's window |
| Recompute work entries over a range | Employee Version, exactly one | a first date and a last date | nothing | Builds and runs a regeneration wizard with the guards skipped |

The list-and-form contextual operation "Set to Draft" is described in [3.5](#35-the-contextual-operation-on-the-list-and-the-form).

---

## 9. Client-side components

Three components are contributed and each changes what a rebuild's client must render.

| Component | Where | What it does |
|---|---|---|
| The work entry kind selector | Every place a kind is chosen — the form, the list, the search view, the multiple-creation form | Renders the kind as a coloured badge carrying its display code, rather than as plain text. It reads the display code and the colour alongside the name |
| The generation source selector | The employee form and the version form | Renders the source as a set of radio choices and shows a warning tooltip when the version is fully flexible: "Invalid option: For fully flexible calendars, the work entry source cannot be 'Working Hours'." |
| The employee tag list with an error state | The regeneration wizard | Renders each employee avatar and marks in red those listed in a second field, here the employees holding a validated entry in the range |

A fourth, smaller contribution is a reusable button template that disables itself and shows the
tooltip "Solve conflicts first" — the shape a payroll capability uses for its own "compute" buttons
when the period still holds conflicts.

---

## 10. Contributions to the views of other domains

| View | Domain | What this domain adds |
|---|---|---|
| Employee form | Human resources core | The "Work Entries" button, shown only when the employee has entries and only to a human resources administrator, and the generation source carried invisibly next to the schedule |
| Contract template form | Human resources core | The generation source carried invisibly, so that it can be copied from a template |
| Working schedule line list and form | Attendances and working time | The work entry kind, after the week type on the list and after the day period on the form |
| Working time exclusion list, form and search | Attendances and working time | The work entry kind after the resource on the form, after the last date on the list, and in the search view as both a field and a grouping, all restricted to human resources officers |
| Time Off Type form | Time off | A group headed "Payroll" carrying the work entry kind |
| Time Off Type list | Time off | The work entry kind as an optional column, shown by default |

---

## 11. Printable documents, exports and integrations

**Printable documents: none.** The domain produces no report and no printable document of its own. The
pivot report of [3.3](#33-the-pivot-report) is a screen report and is exported through the platform's
generic export.

**Message templates: none.** The domain sends no message. The only notifications a person receives
about work entries come from the change tracking of the three version fields, which posts into the
version's own message thread; that mechanism belongs to
[Messaging and Activities](../messaging-and-activities/README.md).

**The external code.** The one deliberate integration affordance is the external code on a work entry
kind, whose help text reads "Use this code to export your data to a third party". Nothing inside the
platform reads it. It is mirrored, read-only, onto every work entry and offered as an optional column
on the list so that a generic export can carry a third party's own vocabulary instead of the internal
payroll code.

**Import.** Work entries can be imported through the platform's generic import. Three consequences
follow from the rules and a rebuild must reproduce them:

1. Every imported row goes through the ordinary creation path, so the version is resolved from the
   employee and the date, the company is taken from the employee, the pay rate is copied from the
   kind, and the four conflict conditions run.
2. An import that suppresses the conflict check with the marker of rule
   [`WKE-027`](business-rules.md#5-the-four-conflict-conditions) is far faster and leaves the day book
   unchecked; the checks must then be run afterwards.
3. The payroll-code uniqueness of rule [`WKE-006`](business-rules.md#2-the-work-entry-type-catalogue)
   is a write-time validation and is **not** re-checked by a loader that bypasses the write path, so
   loading kinds that way can leave duplicate codes behind.

**Export.** The list, the pivot and the generic export all read the same fields. The fields a payroll
integration needs are: the employee, the date, the payroll code of the kind, the duration in hours, the
pay rate stored on the entry, and the extra-hours flag of the kind. The external code is available for
integrations that key on a third party's vocabulary.

---

## 12. Remote operation contracts

Everything in [chapter 8](#8-named-operations) is callable remotely under the platform's own
transport, described in [Remote transport contracts](../../interfaces/remote-transport-contracts.md).
Four contracts are worth stating because a client depends on their exact shape:

1. **Generation returns records, not a count.** The calendar's quick replacement reads the first
   returned record's identifier in order to write a kind onto it.
2. **Validation returns a true-or-false value, not an error.** A caller must test the value; a
   successful call does not imply that anything was validated.
3. **The regeneration operation accepts being called on an empty selection** when it is given a list
   of day slots, because the calendar calls it that way.
4. **The unusual-days operation is called on the entity, not on a record.** It takes two date strings
   and returns a map keyed by date string.
