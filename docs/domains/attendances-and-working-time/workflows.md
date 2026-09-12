# Attendances and Working Time — Workflows

Each procedure below states its actors, its preconditions, its numbered steps with the
branches taken, the records written at each step with their field values, the messages
emitted and its postconditions. Rule references of the form `AWT-nnn` point at
[business-rules.md](business-rules.md); the arithmetic is in
[calculations.md](calculations.md) and
[working-schedule-algorithms.md](working-schedule-algorithms.md); the state changes are in
[state-machines.md](state-machines.md).

| Chapter | Procedure | Principal actor |
|---|---|---|
| [2](#2-creating-and-shaping-a-working-schedule) | Creating and shaping a Working Schedule | a configuration administrator |
| [3](#3-recording-a-closure-or-a-personal-absence) | Recording a closure or a personal absence | a configuration administrator, or any user for their own resource |
| [4](#4-preparing-a-shared-terminal-for-first-use) | Preparing a shared terminal for first use | an attendance officer for all employees |
| [5](#5-recording-an-attendance-by-hand) | Recording an attendance by hand | an attendance officer |
| [6](#6-checking-in-at-a-shared-terminal) | Checking in at a shared terminal | an employee at the device |
| [7](#7-checking-in-from-the-application-menu-bar) | Checking in from the application menu bar | any signed-in user with an employee record |
| [8](#8-correcting-or-deleting-an-attendance) | Correcting or deleting an attendance | an attendance officer |
| [9](#9-regenerating-extra-hours) | Regenerating extra hours | the system, on every relevant change |
| [10](#10-automatic-check-out) | Automatic check-out | the unattended job runner |
| [11](#11-absence-detection) | Absence detection | the unattended job runner |
| [12](#12-approving-refusing-and-partially-approving-extra-hours) | Approving, refusing and partially approving extra hours | an approver |
| [13](#13-converting-extra-hours-into-absence-entitlement) | Converting extra hours into absence entitlement | an employee and an absence approver |
| [14](#14-reviewing-attendance-and-acting-on-the-results) | Reviewing attendance and acting on the results | an attendance officer |
| [15](#15-comparing-recorded-presence-with-recorded-timesheet-time) | Comparing recorded presence with recorded timesheet time | an attendance officer who also records timesheets |

---

## 1. How to read these procedures

- A **step** is one thing the system does or one thing a person does. Steps are ordered and
  a failing step aborts the whole procedure unless the step says otherwise.
- A **branch** states the condition that selects it. Branches are exhaustive: where a
  condition can fail, the failing branch is written out.
- **Records written** names the entity and the fields set. Where a value is derived, the
  procedure names the chapter that derives it rather than repeating the arithmetic.
- Instants written as a time of day with the word *local* are wall-clock times in the zone
  the step names; every other instant is on the universal time scale.
- Every procedure that changes worked time ends by invoking
  [chapter 9](#9-regenerating-extra-hours). That invocation is never optional and never
  deferred.

---

## 2. Creating and shaping a Working Schedule

**Actors.** A configuration administrator, that is a user holding the platform settings
group. Every internal user may read schedules; only that group may create or change them
(`AWT-108`).

**Preconditions.** A company exists and has a default Working Schedule; a company created
without one receives one automatically (`AWT-033`).

**Steps.**

1. The administrator opens the working-schedule list and starts a new record. The form
   proposes: the name "Working Hours of " followed by the company name; the acting company;
   the time zone of the request, failing that the acting user's zone, failing that the zone
   of the built-in administrator user, failing that universal time; the full-time reference
   taken from the company's default schedule; and a copy of that schedule's periods together
   with its two-week flag. When the company has no default schedule, or when the default is
   a two-week schedule and this one is not, the built-in fifteen-line forty-hour pattern of
   [entities.md, chapter 3.9](entities.md#39-the-built-in-forty-hour-pattern) is proposed
   instead (`AWT-039`).
2. The administrator chooses the schedule type.
   - **Branch, flexible.** The period tab disappears. The administrator enters the average
     hours per day and the total hours per week directly; those two values are the daily and
     weekly budgets and are never recomputed (`AWT-028`). Nothing else in this chapter
     applies.
   - **Branch, fully fixed.** The periods are edited line by line, as in the following steps.
3. **Branch, duration-based entry.** The administrator presses the duration switch and
   confirms the text "If checked, the working schedule will be based on an amount of hours
   (defined for each day) and not a start & end time anymore.Do you confirm ?", reproduced
   exactly including the missing space. Every break period is deleted (`AWT-026`). Each
   remaining period is then edited by its length alone, and its two clock times are derived
   by centring the length on twelve o'clock, as specified in
   [calculations.md, chapter 2.3](calculations.md#23-deriving-clock-times-from-a-length).
4. **Branch, two-week layout.** The administrator presses the two-week switch and confirms
   "Are you sure you want to switch to a 2-week calendar? All work entries will be lost."
   Two section markers are inserted and every period is duplicated into both weeks
   (`AWT-030`). Two tabs appear, "Week 1 Working Hours" and "Week 2 Working Hours", each
   preceded by a sentence naming the first and last day of the current week and stating
   whether it is the even or the odd week.
5. For each period the administrator sets the name, the weekday, the period kind — morning,
   break, afternoon or full day — and either the two clock times or, on a duration-based
   schedule, the length. Both hours are clamped as the value is typed (`AWT-020`): the start
   to the range zero to `23.99`, the end to the range zero to `24`, and the end is finally
   raised to at least the start.
6. Each change recomputes the two averages, unless the schedule is flexible: the total hours
   per week, the number of working days per week and the average hours per day, by the
   formulas of
   [calculations.md, chapter 1](calculations.md#1-the-averages-of-a-working-schedule). Each
   period's length in hours and length in days follow
   [calculations.md, chapter 2](calculations.md#2-the-lengths-of-a-working-schedule-line).
7. On save the overlap validation runs (`AWT-021`), and, in two-week mode, the
   sections-first validation (`AWT-022`). A failure aborts the whole save with the quoted
   message.
8. The full-time reference is recomputed from the company's default schedule (`AWT-029`) and
   the work-time rate follows. A schedule with no company keeps the reference the
   administrator typed.
9. Optionally the administrator attaches closures. The "Closing Days" action opens the
   schedule's global exclusions in a calendar; the "Resources Time Off" action opens the
   exclusions that name a resource. Both lead to [chapter 3](#3-recording-a-closure-or-a-personal-absence).

**Records written.** One Working Schedule; one Working Schedule Line per period, including
the two section markers in two-week mode.

**Postconditions.** The schedule has a coherent set of periods, a time zone, an average per
day, a total per week, a full-time reference and a work-time rate. Every Resource pointing
at it immediately produces different intervals.

**Failure conditions.** Overlapping periods (`AWT-021`); a period before the first section
in two-week mode (`AWT-022`); a break period on a duration-based schedule (`AWT-026`); a
missing time zone (`AWT-019`); a missing name.

**Worked example — a two-week schedule averaging thirty-eight hours.** First week, Monday to
Friday, morning `08:00`–`12:00`, break `12:00`–`13:00`, afternoon `13:00`–`17:00`, that is
forty countable hours. Second week, the same except that Friday holds only the morning, that
is thirty-six. The raw weekly total is seventy-six, halved to `38.00`; the distinct working
weekdays are five plus five, halved to five; the average per day is round( 38 ÷ 5 , 2 ) =
`7.60`; against a forty-hour reference the work-time rate is `95.00` and the schedule is not
full time.

---

## 3. Recording a closure or a personal absence

**Actors.** A configuration administrator for a closure that names no resource; any internal
user for a record that names their own resource; the absence domain for validated absence
requests and for public holidays (`AWT-109`).

**Steps.**

1. The actor opens the exclusion list, or the calendar reached from a schedule, and starts a
   new record. When neither instant is supplied, both are defaulted to the whole of today in
   the relevant schedule's zone: the start becomes the first moment of today in that zone and
   the end its last representable moment, both converted to the universal scale
   ([entities.md, chapter 5.3](entities.md#53-defaults-on-an-empty-form)).
2. **Branch, a closure.** The resource is left empty. A schedule may be chosen or left empty:
   leaving it empty makes the closure apply to every schedule of the record's company;
   choosing one restricts it to that schedule. Only the platform settings group may create
   or change a record with no resource.
3. **Branch, a personal absence.** A resource is chosen, which immediately sets the schedule
   to that resource's schedule. An ordinary user may create, change and delete records of
   their own resource only.
4. The actor sets the reason, the two instants and the time type — `leave` for time that is
   removed from working time, `other` for time that stays inside it (`AWT-058`).
5. On save the end instant is recomputed when it is empty or not strictly later than the
   start (`AWT-054`), the ordering validation runs, and, for two closures of the same
   schedule, the overlap validation of `AWT-055` runs.
6. On save, and on every later change or deletion, every attendance of the affected
   employees on the affected local days is queued for the regeneration of
   [chapter 9](#9-regenerating-extra-hours). On a change the filter is computed twice, once
   before and once after the write, and their union is used, so that moving an exclusion
   recomputes both the day it left and the day it arrived at
   ([entities.md, chapter 5.7](entities.md#57-effect-on-attendance-overtime)).

**Records written.** One Working Time Exclusion.

**Postconditions.** The working intervals of the affected resources no longer contain the
excluded time. Day counting, hour counting, planning, unavailability and the extra-hours
computation all reflect it at once.

**Worked example.** A closure of the whole of Tuesday 11 November 2025 for the first company,
plus a personal absence for one employee from Thursday 13 November `08:00` to `12:00` local.
Over the week Monday 10 November to Friday 14 November the employee's work intervals total
twenty-eight hours instead of forty, and the duration measurement returns 3.5 days.

---

## 4. Preparing a shared terminal for first use

**Actors.** An attendance officer for all employees.

**Preconditions.** The actor holds the officer-for-all group; the company has a terminal key,
which every company receives on creation.

**Steps.**

1. The officer opens the terminal entry of the navigation, which calls the terminal-menu
   route with the company identifier. A reader who does not hold the officer-for-all group
   receives a not-found response (`AWT-110`).
2. When the signed-in session has a password, the session is signed out first — a deliberate
   precaution against leaving an authenticated session on a shared device — and the browser
   is redirected to the company's terminal address.
3. On the settings screen the officer may switch the terminal mode between badge only, badge
   and manual selection, and manual selection only. The client calls the settings route with
   the token and the chosen mode, and the mode is written on the company of the acting user.
4. The officer may create a missing employee: the client calls the create-employee route with
   a name and the token, and an employee of the token's company is created with that name.
5. The officer may attach a badge to an employee that has none: the client asks for the
   employees of that company without a badge identifier, optionally narrowed by a name
   fragment, and then calls the set-badge route with the employee identifier, the badge value
   and the token.
6. While the page stays open the client calls the keep-alive route periodically, which
   refreshes the session so that a terminal left open across a shift does not expire.

**Records written.** The company's terminal mode; zero or more Employees; zero or more badge
identifiers on existing employees.

**Postconditions.** The device shows the identification screen of
[chapter 6](#6-checking-in-at-a-shared-terminal) and the intended employees can identify
themselves.

**Failure conditions.** A token that matches no company (nothing happens, and an empty
structure is returned); a reader without the officer-for-all group (a not-found response for
the menu route, and the informational notification "You don't have the rights to execute
that action." for the trial action).

---

## 5. Recording an attendance by hand

**Actors.** An attendance officer, an officer for all employees, or an administrator.

**Preconditions.** The actor may create attendance records for the chosen employee under
`AWT-100` and `AWT-101`.

**Steps.**

1. The actor opens the attendance list and starts a new record.
2. The form proposes the actor's own employee when the actor holds the officer-for-all group,
   and proposes nothing otherwise. The choice of employee is limited to employees of the
   allowed companies, and further to employees who name the actor as attendance approver when
   the actor holds only the officer group.
3. The actor enters the check-in, which defaults to the current instant, and the check-out.
   Both fields are editable only when the derived manager flag is true (`AWT-103`).
4. On save the validations of `AWT-041` to `AWT-044` run. Any failure aborts the whole save
   with the quoted message.
5. The record is created with both capture channels at their default `manual` (`AWT-050`) and
   with no evidence block. The stored date is the check-in read in the employee's effective
   zone (`AWT-049`). The worked hours are computed by
   [calculations.md, chapter 3](calculations.md#3-the-worked-hours-of-one-attendance).
6. The regeneration of [chapter 9](#9-regenerating-extra-hours) runs over the affected window.

**Records written.** One Attendance; zero or more Attendance Overtime Lines.

**Postconditions.** A closed attendance exists; the employee's balance has moved when the
company approves extra hours automatically.

**Worked example.** The fixture schedule of
[acceptance-criteria.md](acceptance-criteria.md) expects eight hours on a Monday. An officer
records `08:00`–`12:00` and `13:00`–`18:00` local. Presence is nine hours, the balance is
plus one, and one line of `1` hour is created against the second record, covering
`17:00`–`18:00`.

---

## 6. Checking in at a shared terminal

**Actors.** An employee standing at the device; the terminal browser session, which is
unauthenticated and is identified only by the token in its address.

**Preconditions.** A device has been opened on the company's terminal address
([chapter 4](#4-preparing-a-shared-terminal-for-first-use)); the terminal mode allows the
identification method the employee will use; for badge identification the employee has a
badge identifier, and for manual identification with a number the employee has a personal
identification number.

### 6.1 Rendering the page

1. The device requests the terminal page with the token in the path. The company whose
   terminal key equals the token is looked up. **Branch, no company matches:** a not-found
   response is returned and nothing else happens (`AWT-110`).
2. The page payload is assembled: the token, the company identifier, the company name, the
   departments of that company with their employee counts, the terminal mode, the trial flag,
   the badge source, whether device and location tracking is enabled, the language and the
   platform version. The structure is specified in
   [interfaces.md, chapter 2.1](interfaces.md#21-data-structures).
3. **Security branch.** When the signed-in user of the requesting session has a password and
   the page was not opened from the trial action, the session is signed out before the page
   is rendered, which prevents an unattended terminal from exposing the rest of the platform.
   When the page was opened from the trial action, or when the session has no password and is
   not the public user, the page opens on its settings screen instead of the identification
   screen.

### 6.2 Identification by badge

1. The employee presents the badge. The device reads it with the configured source: a
   dedicated reader, the front camera or the rear camera.
2. The client locks the reader against a double read and blocks the interface.
3. **Branch, device and location tracking enabled.** The client asks the browser for the
   current coordinates and calls the badge route with the token, the badge value and the
   coordinates; on refusal or failure it calls the same route without them. **Branch,
   tracking disabled.** The route is called without coordinates.
4. The company is resolved from the token, then the single employee of that company whose
   badge identifier equals the scanned value.
   - **Branch, no employee matches.** The response is an empty structure and the client shows
     the notification "No employee corresponding to Badge Identifier '*scanned value*.'",
     echoing the scanned value exactly as read.
   - **Branch, an employee matches.** The evidence block of
     [interfaces.md, chapter 2.2](interfaces.md#22-the-evidence-block-built-on-the-server) is
     built with channel `kiosk`, and the change-state operation is invoked with elevated
     rights.
5. The change-state operation reads the employee's attendance state. When the employee is not
   checked in it creates an attendance with the current instant as check-in and the evidence
   applied to the check-in side. When the employee is checked in it finds the open record and
   writes the current instant as check-out with the evidence applied to the check-out side.
   When the state says checked in and no open record can be found, it refuses with the message
   of `AWT-048`.
6. The employee information block is returned: the name, the picture, the total approved extra
   hours, the confirmation delay in milliseconds, the check-in and check-out of the last
   record, the extra hours already recorded for today, whether a personal identification
   number is required, whether extra hours may be displayed, and the hour aggregates.
7. The client shows the confirmation screen, which greets or takes leave of the employee by
   name, shows the instant, shows the hours worked today and, when the company permits it,
   today's extra hours and the running balance. It closes after the configured number of
   seconds or when the acknowledgement action is pressed.

### 6.3 Identification by manual selection

1. The employee presses the manual-identification action. The client asks for a page of
   employees with the token, a page size, an offset and a filter.
2. The filter is validated: only conditions on the name and the department, with the
   operators *equals* and *contains*, are accepted; anything else is refused with the message
   of `AWT-111`.
3. The filter is restricted to the company of the token, and the employees are returned
   ordered by name and then by identifier, each with the identifier, the display name, the job
   position, the avatar, the attendance state and the capture channel of the last record,
   together with the total number of matches.
4. The employee finds the row — optionally narrowing the list by typing part of a name or by
   choosing a department in the side panel — and presses it.
5. The client asks for that employee's information block. The employee must belong to the
   company of the token; otherwise an empty structure is returned.
6. **Branch, the company requires a personal identification number.** The client shows a
   numeric keypad with the prompt naming the employee and the direction of the action. The
   employee enters the number and confirms.
7. The client calls the manual-selection route with the token, the employee identifier, the
   entered number and, when tracking is enabled, the coordinates.
   - **Branch, the employee does not belong to the company of the token, or the number does
     not match.** The response is an empty structure, nothing is written, and the client shows
     the notification "Wrong Personal Identification Number". No message distinguishes a wrong
     number from an unknown employee (`AWT-110`).
   - **Branch, the check succeeds.** The change-state operation is invoked with elevated
     rights and channel `kiosk`, and the employee information block is returned.
8. The confirmation screen behaves as in [chapter 6.2](#62-identification-by-badge).

**Records written.** One Attendance created or one Attendance closed, with both channels
recorded as `kiosk`; the extra-hours lines the regeneration produces.

**Postconditions.** The employee's attendance state has flipped; the terminal has returned to
its identification screen.

---

## 7. Checking in from the application menu bar

**Actors.** Any signed-in user who has an employee record.

**Preconditions.** The employee's company has the menu-bar widget enabled; the user is signed
in.

**Steps.**

1. The client renders the widget from the deferred part of the session payload, which already
   carries the employee identifier, the hours accumulated today, the hours accumulated earlier
   today, the hours of the current stretch, the last check-in, the attendance state, whether
   the widget is enabled and whether device tracking is enabled. When that payload is absent
   the client calls the widget route instead
   ([entities.md, chapter 14.7](entities.md#147-request-routing--owned-by-the-platform-foundation)).
2. The user presses the single action button, whose label is "Check in" when the state is
   checked out and "Check out" when it is checked in.
3. **Branch, device tracking enabled and the browser offers a location service and is
   online.** The client requests the current coordinates with high accuracy and a ten-second
   limit.
   - On success it calls the widget check-in route with the latitude and the longitude.
   - On failure it shows the confirmation "Unable to get a valid location. Do you want to
     proceed with your check-in/out anyway?" with the confirm label "Proceed Anyway"; on
     confirmation the route is called without coordinates, on cancellation nothing happens.
4. **Branch, device tracking disabled.** The route is called at once without coordinates.
5. The employee of the signed-in user is resolved **in the company selected in the client**,
   not in the user's default company. The evidence block is built with channel `systray` and,
   only when the employee's company enables device and location tracking, with the place name
   resolved from the coordinates — or the literal `Unknown` when the lookup fails or is
   refused — the coordinates supplied by the client or, failing them, those derived from the
   network address, the network address itself and the browser family.
6. The change-state operation of [chapter 6.2](#62-identification-by-badge), step 5, is
   invoked and writes the record.
7. The full employee information block is returned and the client repaints the widget.
8. **Error branch.** When the connection is lost, the client shows the notification
   "Connection lost. Check in/out could not be recorded." under the title "Attendance Error"
   and no record is written.

**Records written.** One Attendance created or one Attendance closed, with the channel of the
side just written recorded as `systray`.

**Postconditions.** The widget shows the new state, the hours of the current stretch and the
total for the day, each rounded to two decimal places for display only (`AWT-126`).

---

## 8. Correcting or deleting an attendance

**Actors.** An attendance officer who is the approver of the employee concerned, an officer
for all employees, or an administrator.

**Steps.**

1. The actor opens the record from the list, from the day board or from the management queue.
2. The two instants are editable only when the derived manager flag is true (`AWT-103`).
3. **Branch, the employee is reassigned.** The guard of `AWT-046` runs before the write; a
   failure raises the access message and nothing is written.
4. The window affected by the current values is computed **before** the write, and the window
   affected by the new values **after** it; the extra hours are regenerated over the **union**
   of the two ([entities.md, chapter 8.7](entities.md#87-the-recomputation-window)). This is
   what moves a quantity from one local day to another when an instant crosses midnight.
5. **Branch, the check-out is cleared.** The record becomes open again, subject to the
   one-open-record guard (`AWT-043`). Its extra-hours lines disappear, because an open record
   never carries one (`AWT-064`).
6. **Branch, deletion.** The affected window is computed first, the record is deleted, and the
   regeneration then runs over that window on the records that remain.

**Records written.** One Attendance changed or deleted; every Attendance Overtime Line of the
affected days deleted and rebuilt.

**Postconditions.** The record reflects the correction; the lines of every day touched, before
and after, are consistent with the rules and with the remaining records.

**Worked example.** An employee has an attendance from `08:00` to `12:00` on 3 January with a
shortfall line of minus four hours, absence management being on. The officer adds a second
record from `13:00` to `17:00`: the day now totals eight countable hours, the balance is zero
and no line remains. Adding a third from `18:00` to `19:00` produces one line of plus one on
the third record. Extending the third to `20:00` makes it plus two. Deleting the second record
turns the day into six countable hours and the third record then carries a line of minus two.

---

## 9. Regenerating extra hours

This is the central computation of the domain. It runs automatically after every change that
can move worked time, and on demand from the rule-set form.

**Triggers and the scope each passes.**

| Trigger | Scope |
|---|---|
| An attendance is created | The window of the created records. |
| An attendance is written and its employee, check-in or check-out changed | The union of the window before and the window after the write. |
| An attendance is deleted | The window the deleted records covered. |
| A Working Time Exclusion is created, moved or deleted | Every attendance of the affected employees on the affected local days, as computed in [entities.md, chapter 5.7](entities.md#57-effect-on-attendance-overtime). |
| Either company tolerance amount changes | Every attendance of every employee of that company (`AWT-119`). |
| The "Regenerate overtimes" action on an Overtime Ruleset | Every attendance of every employee whose Employee Version names that rule set, from the earliest of those versions' dates onwards. |

**Steps.**

1. **Determine the scope.** When no explicit scope is given, build it from the records at
   hand: group them by employee, keep only the closed ones, read the earliest check-in and
   the latest check-out in the employee's **own** zone, and then
   - when any rule of any applicable rule set uses the weekly period, widen the local range
     to the Monday on or before the first local date and the Sunday on or after the last
     (`AWT-062`);
   - otherwise use the local dates themselves.
   The condition per employee selects that employee's attendances whose check-in is at or
   before the last moment of the last local day and whose check-out is **strictly after** the
   first moment of the first local day, both converted to the universal scale (`AWT-063`).
   With no closed attendance in the set, the condition selects nothing and the procedure
   stops.
2. **Remember the days a person has touched.** Translate the attendance condition into a
   condition on the lines by renaming the check-in to the line's start instant and the
   check-out to its stop instant. Load the matching lines and record the pairs of employee and
   day of every line whose encoded amount differs from its computed amount, or whose status is
   still `to_approve` (`AWT-060`).
3. **Delete every matching line.**
4. **Collect the attendances to evaluate:** the union of the records at hand and every
   attendance matching the condition, keeping only the closed ones. Stop when none remain.
5. **Widen the evaluation span** by one day on each side, from the first moment of the day
   before the earliest check-in date to the last representable moment of the day after the
   latest check-out date, on the universal scale. The margin absorbs every zone shift.
6. **Resolve the employment periods:** for each employee, the stretches during which each
   Employee Version is in force inside the span, each converted using the zone of that
   version's schedule, or of the employee's resource when the version has no schedule.
7. **Assemble the schedule picture** per employee, in naive local time, with its four parts —
   working periods, break periods, absences and fully flexible stretches — as specified in
   [calculations.md, chapter 6.2](calculations.md#62-assembling-the-schedule-picture). Each
   family is intersected with the employment period it belongs to, so a change of schedule
   mid-span is honoured, and each family uses its own zone before the zone is stripped.
8. **Resolve the rule set of every attendance:** the version in force at the attendance's
   local check-in names it. Attendances are grouped by rule set; an attendance whose version
   names none produces nothing (`AWT-065`).
9. **For each rule set, generate the amounts** by
   [calculations.md, chapter 6](calculations.md#6-the-extra-hours-generation-algorithm),
   producing, per attendance, per local day and per exact combination of rules, one value set
   holding the employee, the day, the duration rounded to four decimal places, the start
   instant equal to the attendance's check-in, the stop instant equal to its check-out, the set
   of rules, the combined pay rate and the compensable-as-time-off flag.
10. **Force the pending status** on every value set whose pair of employee and day was
    recorded in step 2 (`AWT-060`).
11. **Create the lines** in one operation.
12. **Mark for recomputation** the extra hours, the regular hours, the validated extra hours
    and the extra-hours status of every attendance in scope.

**Records written.** Attendance Overtime Lines deleted and created. No attendance is written;
only derived fields are marked.

**Postconditions.** For every day in scope the lines are exactly those the current rules and
the current worked time imply; days a person had touched are pending again; every attendance
shows consistent worked, regular, extra and validated hours.

**Failure conditions.** None that a user sees: the procedure is idempotent and produces the
same lines from the same inputs. Its only observable hazard is that line identifiers change
(`AWT-061`).

---

## 10. Automatic check-out

**Actors.** The unattended job runner. The job is named "Attendance: Automatically check-out
employees" and runs every four hours (`AWT-098`), with elevated rights (`AWT-099`).

**Steps.**

1. **Select** every open attendance whose employee's company has automatic check-out enabled
   and whose employee's schedule is **not** flexible. Stop when there are none. The
   compatibility finding of `AWT-093` applies to employees with no schedule at all.
2. **Compute the hours already worked**, per employee and per local day, from the closed
   records whose check-in is later than the first moment of the day of the earliest
   candidate's check-in. The local day is read in the zone of the version in force at that
   record's date.
3. **Read each company's tolerance**, expressed in decimal hours and defaulting to two.
4. **For each candidate record**, with the zone of the version covering the record's date:
   - read the check-in and the current instant in that zone;
   - the elapsed duration is the difference of the two, in hours;
   - the hours already worked are the amount computed in step 2 for that employee and the
     local day of the check-in;
   - the expected hours are the total length of the employee's expected work intervals from
     the first moment of that local day to the same moment of the next day, which already
     excludes breaks and approved absences
     ([calculations.md, chapter 6.7](calculations.md#67-the-expected-attendances-of-an-employee));
   - **guard:** act only when *elapsed + already worked − tolerance* is greater than
     *expected* (`AWT-090`); otherwise leave the record open and move on;
   - set the check-out provisionally to `23:59:59` of the check-in's local day, which makes
     the record's worked hours computable for the whole day and subtracts that day's break;
   - the excess is *the provisional worked hours − ( expected + tolerance − already worked )*;
   - write the final check-out as the later of *the provisional check-out minus the excess*
     and *the check-in plus one second*, together with the channel `auto_check_out`
     (`AWT-091`);
   - post the note "This attendance was automatically checked out because the employee
     exceeded the allowed time for their scheduled work hours." in the record's discussion
     thread (`AWT-092`).
5. The ordinary write path then runs the regeneration of
   [chapter 9](#9-regenerating-extra-hours).

**Records written.** One Attendance closed per candidate; one message per closed record; the
extra-hours lines of the affected days.

**Postconditions.** Every forgotten record of a non-flexible employee is closed at the instant
that makes the day total exactly the expected time plus the tolerance.

**Worked example — twelve hours open, one-hour tolerance.** The schedule is `08:00`–`12:00`,
break, `13:00`–`17:00`, eight expected hours. The employee was present `08:00`–`12:00`
(closed, four worked hours) and checked in again at `13:00` without checking out. The job runs
at `22:00`. Elapsed nine, already worked four, expected eight: 9 + 4 − 1 = 12 > 8, so it
fires. The provisional check-out is `23:59:59`, giving 10.9997 worked hours; the excess is
10.9997 − ( 8 + 1 − 4 ) = 5.9997; the final check-out is `18:00:00`. The day totals 4 + 5 = 9
hours, the eight expected plus the one-hour tolerance.

**Worked example — more than a day late.** The employee checked in on 30 January at `08:00`
and the job runs on 1 February at `23:00`. Elapsed about sixty-three hours, so the guard
fires. The provisional check-out is 30 January `23:59:59` — the check-in's own day — the
excess is 14.9997 − 9 = 5.9997 and the final check-out is 30 January `18:00`.

**Worked example — a two-week schedule.** With a tolerance of zero and a check-in at `08:00`,
a run on a first-week Wednesday whose only period is the afternoon `13:00`–`17:00` closes the
record at `12:00` with four worked hours, while a run on a second-week Wednesday with a full
day closes it at `17:00` with eight worked hours.

**Worked example — an employee still inside the allotted time.** Tolerance one hour, expected
eight, check-in at `21:00` local, job at `23:00` local: 2 + 0 − 1 = 1 is not greater than 8,
so the record stays open.

---

## 11. Absence detection

**Actors.** The unattended job runner. The job is named "Attendance: Detect Absences for
employees" and runs every four hours (`AWT-098`), with elevated rights (`AWT-099`).

**Steps.**

1. Compute *yesterday* as today at `00:00:00` minus one day, on the unattended process's own
   clock and not in any employee's zone.
2. Select the companies that have absence management enabled. Stop when there are none.
3. Collect the employees that already have at least one extra-hours line dated *yesterday*;
   they are not absent.
4. Select the absent employees: not in that set, belonging to one of those companies, whose
   schedule is not flexible, and whose current version's contract started on or before
   *yesterday* (`AWT-094`).
5. For each of them create a **technical attendance**: check-in at the first moment of
   *yesterday* read in the employee's effective zone and converted to the universal scale,
   check-out one second later, both channels `technical` (`AWT-095`).
6. Because the record is closed, the ordinary creation path runs the regeneration of
   [chapter 9](#9-regenerating-extra-hours), which measures a presence of 0.0003 hours against
   the whole expected day and emits the corresponding shortfall — provided the company has
   absence management enabled and the applicable rule takes its expected quantity from the
   employee's schedule.
7. Delete again every technical attendance whose resulting extra hours are zero at three
   decimal places (`AWT-096`); an employee who was not in fact expected to work leaves no
   trace.
8. Post the note "This attendance was automatically created to cover an unjustified absence on
   that day." on every technical attendance that survives (`AWT-097`).

**Records written.** One Attendance per absent employee, some of them deleted again in step 7;
one Attendance Overtime Line per surviving record; one message per surviving record.

**Postconditions.** Every missed working day of an absence-managed company carries a
one-second technical attendance and a negative line equal to the whole expected day.

**Follow-up branch.** When a real attendance is entered afterwards for that day, the
regeneration recomputes the day from both records; the shortfall disappears and the line
attached to the technical attendance becomes zero.

**Worked example.** Absence management is on; the schedule expects eight hours on Wednesday
29 July; the employee recorded nothing. The job runs on 30 July, creates a record from 29 July
`00:00:00` local to `00:00:01` local, and the quantity rule computes a presence of one second
against an expectation of eight hours, emitting a line of approximately minus eight. Had the
employee been on a validated absence all day, the expectation would be zero, the balance would
round to zero, and the technical attendance would be deleted again.

---

## 12. Approving, refusing and partially approving extra hours

**Actors.** An attendance officer who is the approver of the employee, an officer for all
employees, or an administrator.

**Preconditions.** At least one extra-hours line is linked to the record; the actor's derived
manager flag on the record or on the line is true (`AWT-103`).

**Steps.**

1. The actor opens the management queue, which lists closed records with the pending filter
   and the active-employees filter pre-selected, showing the employee, the two instants, the
   worked time, the extra hours and the validated extra hours.
2. **Branch, approve everything on the record.** The actor presses the approve action on the
   row or in the list header. Every line linked to that record moves to `approved`
   (`AWT-089`); the record's derived status becomes `approved` and its validated extra hours
   become the sum of the encoded amounts.
3. **Branch, refuse everything on the record.** Every linked line moves to `refused` and the
   validated extra hours fall to zero.
4. **Branch, partial approval.** The actor opens the record, goes to the extra-hours detail
   tab, edits the encoded amount of one line to the amount actually granted, and presses the
   approve action on that line alone. That line becomes `approved`; the record's validated
   extra hours reflect the reduced amount while its computed extra hours are unchanged
   (`AWT-085`). The record's derived status stays `to_approve` while other lines remain
   pending and becomes `approved` once every line is approved.
5. **Branch, grant the time back as absence.** Before approving, the actor sets the
   compensable-as-time-off flag on the line. The approved amount then feeds the deductible
   balance of [chapter 13](#13-converting-extra-hours-into-absence-entitlement) as well as, or
   instead of, being paid.
6. Any later regeneration of that same local day recreates the line with its computed amount
   and forces the status back to `to_approve`, because step 2 of
   [chapter 9](#9-regenerating-extra-hours) recorded the day as touched.

**Records written.** Attendance Overtime Lines written; the linked Attendances' derived fields
recomputed.

**Postconditions.** The employee's balance contains exactly the sum of the encoded amounts of
the approved lines.

**Worked example.** An employee works `08:00`–`18:00` against a nine-hour expected day and the
company approves manually. One line of one hour is generated with status `to_approve`; the
record shows one extra hour and zero validated. The approver reduces the encoded amount to
`0.5` and approves: the record shows one extra hour computed and half an hour validated, and
the balance rises by half an hour. A second attendance is later added on a **different** day;
the first day is untouched, so its line keeps its approved status and its encoded half hour
while its computed amount is recalculated to one hour.

---

## 13. Converting extra hours into absence entitlement

**Actors.** An employee, and the absence approver. This procedure crosses into
[Time Off](../time-off/); only the part this domain owns is specified here.

**Preconditions.** An absence kind exists that deducts extra hours and requires no allocation;
the employee has approved extra-hours lines flagged compensable as time off.

**Steps.**

1. The **deductible balance** is computed by
   [calculations.md, chapter 14](calculations.md#14-the-extra-hours-ledger-used-by-the-absence-domain):
   the sum of the encoded amounts of the employee's approved lines flagged compensable as time
   off, minus the hours of every request of a deductible kind whose state is neither refused
   nor cancelled, minus the hours of every allocation of a deductible kind whose state is
   pending, first-level validated or validated.
2. The employee requests absence of that kind. On creation, on any change of duration, dates,
   state, employee or kind, and on approval, the balance is recomputed.
3. **Guard.** When the resulting balance would be negative, the request is refused with "You
   do not have enough extra hours to request this leave" when the requester is the employee,
   and with "The employee does not have enough extra hours to request this leave." when it is
   somebody else.
4. The absence kind's displayed name is decorated with the amount still available, rendered in
   hours and minutes.

**Records written.** None in this domain. The absence request and its allocation belong to
[Time Off](../time-off/); approving such a request writes nothing on the attendance records
and nothing on the extra-hours lines.

**Postconditions.** The banked extra hours and the absence taken against them stay in balance.

**Worked example.** The employee has three approved compensable lines of 2, 1.5 and 0.5 hours,
that is four hours. A pending request of two hours of the deductible kind reduces the balance
to two. A second request of three hours would bring it to 4 − 2 − 3 = −1 and is therefore
refused.

---

## 14. Reviewing attendance and acting on the results

**Actors.** An attendance officer or an administrator.

**Steps.**

1. The actor opens the attendance overview, which lists the records of the current period with
   the groupings by check-in month and by employee pre-selected. The employee grouping is
   widened by `AWT-113`, so employees with no record in the selection still appear.
2. The actor narrows the list with the filters: own records, own team, still at work, errors,
   automatically closed, a period on the check-in, active or archived employees.
3. **Branch, an error is found.** A record drawn in the error colour is either open with a
   check-in more than a day old, or closed with more than sixteen worked hours, or closed by
   the absence-detection job (`AWT-052`). The actor opens it and applies
   [chapter 8](#8-correcting-or-deleting-an-attendance).
4. **Branch, analysis.** The actor opens the reporting entry, which starts on a pivot of the
   last three months grouped by month and employee, with worked hours, regular hours, extra
   hours and validated extra hours as measures, and offers a graph that plots worked hours per
   employee per week.
5. **Branch, an approval backlog.** The actor opens the management entry, which is the queue of
   [chapter 12](#12-approving-refusing-and-partially-approving-extra-hours).
6. **Branch, the day board.** The actor opens the day-oriented board, one row per employee and
   one column per hour, and reads at a glance who is present, who is outside their schedule and
   which records are in error.

**Records written.** None by the review itself; whatever the branches write.

---

## 15. Comparing recorded presence with recorded timesheet time

**Actors.** An attendance officer who also holds the timesheet user group.

**Steps.**

1. The actor opens the comparison analysis. It is read-only and derived: one row per employee,
   per day and per company.
2. Each row reports the total timesheet time of that day, the total presence time of that day,
   their difference, the timesheet cost, the presence cost and the cost difference. Each cost
   is the employee's hourly cost multiplied by the corresponding number of hours, and a
   product of zero is reported as no value rather than as zero
   ([calculations.md, chapter 11](calculations.md#11-the-amounts-of-the-timesheet-comparison)).
3. The day of an attendance is its check-in read in the zone of the schedule of the employee's
   current version; the day of a timesheet line is the date stored on the line. Attendances
   with a check-in later than today and timesheet lines dated later than today are excluded.
4. Any grouping on a date is ordered descending by default.

**Records written.** None; the analysis writes nothing and stores nothing.

**Worked example.** An employee with an hourly cost of thirty records eight presence hours and
seven timesheet hours on one day. The row reports seven timesheet hours, eight presence hours,
a difference of one, a timesheet cost of two hundred and ten, a presence cost of two hundred
and forty and a cost difference of thirty.

---

## 16. Reconciliation notes

One source version carried a complete workflow file; the other described the same operations
inside its entity and calculation chapters, in more detail on the arithmetic and in less on
the actors. Both were merged procedure by procedure and the following points were resolved.

1. **The state tables that opened the workflow file of one version** — the openness of a
   record, the line status, the derived status of a record and the employee's attendance state
   — belong to [state-machines.md](state-machines.md) under rule seven of the documentation
   rules. They were moved there in full, with their stored values and diagrams added, and are
   linked from here rather than repeated.
2. **The order of the procedures.** One version began with the capture channels, the other
   with the configuration of a schedule. The configuration procedures come first here, because
   nothing can be captured before a schedule and a terminal exist.
3. **The company used by the menu-bar widget.** One version said "the user's employee". The
   employee is resolved in the company selected in the client, which matters for a user who has
   an employee in each of two companies; that is [chapter 7](#7-checking-in-from-the-application-menu-bar),
   step 5.
4. **The provisional check-out of the automatic job.** One version wrote the guard and the
   truncation as one step. They are two, and the provisional value exists only so that the
   day's break is subtracted before the excess is measured; that is
   [chapter 10](#10-automatic-check-out), step 4.
5. **The scope condition of a regeneration.** One version stated the strict inequality on the
   check-out, the other did not. It is strict, and the consequence — a record ending exactly at
   midnight is not rebuilt by the following day — is stated in step 1 of
   [chapter 9](#9-regenerating-extra-hours) and carried by `AWT-063`.
6. **The absence-detection clock.** One version read *yesterday* in the employee's zone. The
   day is taken from the unattended process's own clock; only the check-in instant of the
   technical attendance is localised. That is [chapter 11](#11-absence-detection), steps 1 and
   5.
7. **The deductible balance** was described in one version as a workflow of this domain. The
   requests and the allocations belong to [Time Off](../time-off/); this domain owns only the
   balance formula and the two refusal messages, which is what
   [chapter 13](#13-converting-extra-hours-into-absence-entitlement) specifies.
