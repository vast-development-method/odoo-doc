# Acceptance criteria of the Timesheets domain

Numbered Given / When / Then scenarios with concrete numbers. A re-implementation is behaviourally
equivalent to this specification when every one of them passes. Each scenario names the rule,
formula or state machine it exercises, so that a failure points at the paragraph that defines the
expected behaviour.

**Common fixture.** Unless a scenario says otherwise:

- **Company** *Northwind Studio*, currency euro with two decimal places, project time unit **Hours**,
  encoding unit **Hours** (encoding method "Hours / Minutes"), internal project *Internal* with the
  absence task *Time Off*.
- A **second company** *Meridian Works*, currency United States dollar with two decimal places,
  project time unit **Hours**, encoding unit **Days**, its own internal project and absence task.
- The shipped units: **Hours** (relative factor 1), **Days** (8 hours to the day), **Units** (an
  abstract unit sharing no reference unit with Hours). The unit rounding precision is `0.01`.
- **Working schedule** *Standard 40 hours*: Monday to Friday, 08:00–12:00 and 13:00–17:00, eight
  declared hours a day, forty a week, time zone of the company.
- **Employees of Northwind Studio**: *Iris Wolf* (hourly cost 42.35 euro, linked user *Iris Wolf*,
  privilege Timesheet User), *Tom Baros* (hourly cost 60.00 euro, linked user *Tom Baros*, privilege
  Timesheet Approver), *Lena Hoss* (no hourly cost, linked user *Lena Hoss*, privilege Timesheet
  User), *Paul Reiss* (hourly cost 75.00 euro, linked user *Paul Reiss*, privileges Timesheets
  administrator and Project Administrator). All four follow *Standard 40 hours*.
- **Employee of Meridian Works**: *Nils Berg* (hourly cost 55.00 United States dollars).
- **Project** *Redesign*: time tracking on, analytic account *Redesign* in the project plan (active),
  company *Northwind Studio*, customer *Deco Addict*, visibility "All internal users and invited
  portal users", allocated time 100 hours. Task *Homepage* (allocated time 20 hours) and its
  sub-task *Header* (allocated time 5 hours).
- **Products**: *Consulting Hours* (service, unit Hours, service policy "Based on Timesheets", sales
  price 120.00), *Support Pack* (service, unit Hours, service policy "Prepaid/Fixed Price", sales
  price 100.00, upselling threshold 1), *Milestone Build* (service, unit Hours, service policy "Based
  on Milestones"), *Day Rate Consulting* (service, unit Days, service policy "Based on Timesheets",
  sales price 900.00), *Manual Service* (service, unit Hours, service policy "Based on Delivered
  Quantity (Manual)").
- **External person** *Joel Willis*, a contact of *Deco Addict* with no internal privilege.
- Dates are in 2026. **2 March 2026 is a Monday**; 6 March 2026 is the Friday of that week.
- Where a scenario says "the sales capability", "the absence bridge", "the attendance capability" or
  "the margin capability", that capability is installed; where it does not, the base capability alone
  is assumed.

---

## A. The company, its units and its shipped records

### A1 — Creating a company ships its internal project

**Given** the absence bridge is installed and no company named *Aurora Ltd* exists.
**When** *Aurora Ltd* is created.
**Then** a project named "Internal" is created, belonging to *Aurora Ltd*, and is stored as that
company's internal project; the project is marked as an internal project so that it does not appear
in the ordinary project lists; and the company's project time unit and encoding unit are both the
Hours unit.
**Rule** [business-rules.md](business-rules.md) TS-201, TS-203.

### A2 — The internal project must belong to its company

**Given** *Aurora Ltd* and *Northwind Studio* both exist.
**When** the internal project of *Aurora Ltd* is written to be *Northwind Studio*'s *Internal*
project.
**Then** the write is refused with: "The Internal Project of a company should be in that company."
**Rule** TS-200.

### A3 — The two units default to Hours and the hour unit becomes protected

**Given** a fresh installation.
**When** the time-recording capability is installed.
**Then** the Hours unit exists with a relative factor of one; the Hours unit is given the
hours-and-minutes presentation behaviour `float_time` and the Days unit the half-day cycling
behaviour `float_toggle`; and every company's project time unit and encoding unit are the Hours unit.
**And** the Hours unit can no longer be archived or deleted.
**Rule** TS-203, TS-204, [configuration.md](configuration.md) §6.1.

### A4 — Choosing the day encoding method sets the encoding unit

**Given** *Northwind Studio* with the encoding method "Hours / Minutes".
**When** an administrator selects "Days / Half-days".
**Then** the company's encoding unit becomes the Days unit; the project time unit is unchanged and
stays Hours; no recorded line's stored quantity changes.
**And** the session payload handed to every client of that company now carries an encoding factor of
`1 ÷ 8 = 0.125`.
**Rule** TS-205, [calculations.md](calculations.md) §3.1.

### A5 — Only a timesheets administrator may change the settings

**Given** *Tom Baros* is a Timesheet Approver but not a timesheets administrator.
**When** he opens the settings screen.
**Then** the application block "Timesheets" is not shown to him, and the "Configuration" menu entry
is absent because it requires the settings-administrator privilege.
**Rule** [configuration.md](configuration.md) §1, §3.

### A6 — The shipped service product is protected

**Given** the sales capability and the shipped product *Service on Timesheets*.
**When** anybody tries to archive it, delete it, or set a company on it.
**Then** the operation is refused with: "The Service on Timesheets product is required by the
Timesheets app and cannot be archived, deleted nor linked to a company."
**Rule** TS-185.

---

## B. Recording time — the ordinary path

### B1 — A line records with every default applied

**Given** *Iris Wolf* is signed in, holds Timesheet User, and *Redesign* is her favourite project.
**When** she creates a recorded line supplying only the task *Homepage* and the quantity 3.5.
**Then** exactly one line exists with: project *Redesign* (from the task), task *Homepage*, employee
*Iris Wolf* (the employee linked to her user), user *Iris Wolf*, date the current date in her time
zone, unit Hours (the company's project time unit), quantity 3.5, description `/`, company *Northwind
Studio*, contact *Deco Addict* (from the project), analytic account *Redesign*, and monetary amount
−148.23.
**Arithmetic** `− ( 3.5 × 42.35 ) = −148.225`, rounded half up to two decimals = **−148.23**.
**Rule** TS-001, TS-002, TS-005, TS-006, TS-007, TS-012; [calculations.md](calculations.md) §1.4.

### B2 — An empty description becomes a single slash

**Given** B1.
**When** the description is written as an empty text.
**Then** the stored description is `/`, not an empty text.
**Rule** TS-002.

### B3 — A zero quantity is legal

**Given** B1's fixture.
**When** *Iris Wolf* records 0 hours on *Homepage*.
**Then** the line is created, its quantity is 0 and its monetary amount is 0.00; the conversion step
is skipped because the raw amount is exactly zero.
**Rule** TS-004; [calculations.md](calculations.md) §1.4.

### B4 — A negative quantity produces a positive amount

**Given** B1's fixture.
**When** *Iris Wolf* records −2 hours on *Homepage* to correct an earlier over-recording.
**Then** the line is created with quantity −2 and monetary amount **+84.70**.
**Arithmetic** `− ( −2 × 42.35 ) = +84.70`.
**Rule** TS-004; [calculations.md](calculations.md) §1.8.

### B5 — Clearing the project clears the task

**Given** B1's line.
**When** the project is written as empty.
**Then** the task becomes empty as well, and the record is no longer a recorded line: it becomes an
ordinary analytic line and leaves every timesheet list.
**Rule** TS-009.

### B6 — Writing a foreign project clears the task

**Given** B1's line, on project *Redesign* and task *Homepage*, and a second time-tracked project
*Atlas*.
**When** the project is written as *Atlas*.
**Then** the task *Homepage* is cleared, because it does not belong to *Atlas*; the project becomes
*Atlas*; the company, the analytic account and the amount are recomputed from *Atlas*.
**Rule** TS-010.

### B7 — Writing a task pulls its project

**Given** a recorded line on project *Atlas* with no task.
**When** the task *Homepage*, which belongs to *Redesign*, is written.
**Then** the project becomes *Redesign*.
**Rule** TS-011.

### B8 — The contact follows the task, then the project

**Given** task *Homepage* has its own contact *Gemini Furniture* and project *Redesign* has the
contact *Deco Addict*.
**When** a line is recorded on *Homepage*.
**Then** the line's contact is *Gemini Furniture*.
**When** the task's own contact is removed and the line's project is rewritten so that the contact is
recomputed.
**Then** the line's contact becomes *Deco Addict*.
**Rule** TS-012.

### B9 — Recording from the calendar over a week

**Given** *Iris Wolf* opens her calendar for the week of 2 March 2026 and drags across Monday 2 March
to Sunday 8 March.
**When** she confirms the multi-creation dialogue with project *Redesign* and 2 hours.
**Then** five lines are created, one for each of 2, 3, 4, 5 and 6 March; no line is created for
Saturday 7 or Sunday 8 March; and a success notification reads "Timesheets successfully created"
together with a warning notification reading "Some timesheets were not created: employees aren’t
working on the selected days".
**Rule** TS-027; [configuration.md](configuration.md) §9.1.

### B10 — A multi-creation that hits only non-working days

**Given** the same calendar.
**When** *Iris Wolf* drags across Saturday 7 and Sunday 8 March only and confirms.
**Then** no line is created and the single notification reads "No timesheets created: employees
aren’t working on the selected days".
**Rule** TS-027.

### B11 — The calendar block label, hour encoding

**Given** a line of 2.5 hours on project *Redesign*, whose display label is `Redesign`, in a company
encoding in hours.
**Then** the calendar block reads `Redesign (2h30)`.
**And** for 3 hours it reads `Redesign (3h)`; for −1.25 hours `Redesign (-1h15)`; for 0.5 hours
`Redesign (0h30)`.
**Rule** [calculations.md](calculations.md) §2.6.

### B12 — The calendar block label, day encoding

**Given** the same line of 8 hours, the company now encoding in days.
**Then** the block reads `Redesign (1d)`; a line of 4 hours reads `Redesign (0.5d)`.
**Rule** [calculations.md](calculations.md) §2.6.

### B13 — The task selector shows the remaining time

**Given** task *Homepage* with 20 hours allocated and 14.5 hours of total time spent, the company
encoding in hours.
**When** a person opens the task selector on a recorded line.
**Then** the entry reads `Homepage (05:30 remaining)`.
**And** with 22.25 hours spent it reads `Homepage (-02:15 remaining)`; with the company encoding in
days and 5.5 hours remaining it reads `Homepage (0.69 days remaining)`.
**Rule** [calculations.md](calculations.md) §2.7.

---

## C. Cost valuation

### C1 — The employee's own hourly cost is used by default

**Given** *Redesign* prices at the task rate.
**When** *Iris Wolf* records 7.5 hours.
**Then** the amount is **−317.63**.
**Arithmetic** `− ( 7.5 × 42.35 ) = −317.625` → half up → −317.63.
**Rule** [calculations.md](calculations.md) §1.5.

### C2 — An employee rate mapping overrides the cost

**Given** *Redesign* is billable, prices at the employee rate, and carries a mapping row for *Iris
Wolf* with a cost of 60.00.
**When** she records 7.5 hours.
**Then** the amount is **−450.00**; her own hourly cost of 42.35 is not consulted.
**Rule** [calculations.md](calculations.md) §1.6.

### C3 — A missing hourly cost yields a worthless line

**Given** *Lena Hoss* has no hourly cost.
**When** she records 6 hours on *Redesign*.
**Then** the line is created with quantity 6 and amount **0.00**.
**Rule** [calculations.md](calculations.md) §1.3.

### C4 — Currency conversion at the line's date

**Given** *Nils Berg* of *Meridian Works* has an hourly cost of 55.00 United States dollars, and the
analytic account of his project carries the euro, whose conversion rate from dollars for the acting
company on 4 March 2026 is 1.0837.
**When** he records 6 hours dated 4 March 2026.
**Then** the amount is **−357.62** euro.
**Arithmetic** `− ( 6 × 55.00 ) = −330.00`; `−330.00 × 1.0837 = −357.621`; half up → −357.62.
**Rule** [calculations.md](calculations.md) §1.7.

### C5 — Only three written values trigger a recomputation

**Given** a recorded line of 7.5 hours with amount −317.63.
**When** only the description is written.
**Then** the amount stays −317.63 and no recomputation runs.
**When** the quantity is written as 8.
**Then** the amount becomes **−338.80**.
**Rule** [calculations.md](calculations.md) §1.1.

### C6 — Splitting a line splits the quantity, not the amount

**Given** a line of 10 hours with an amount of −400.00 (an hourly cost of 40.00).
**When** its analytic distribution is written as 70 % to account *Alpha* and 30 % to account *Beta*
in the same plan.
**Then** the original line keeps 7 hours and its amount is recomputed to **−280.00**, one new line of
3 hours is created with an amount of **−120.00**, and the notification reads "1 analytic lines
created".
**And** the two lines together carry 10 hours and −400.00, not 20 hours.
**Rule** [calculations.md](calculations.md) §1.9.

### C7 — An archived analytic account stops valuation and creation

**Given** the analytic account *Redesign* is archived.
**When** *Iris Wolf* tries to record 2 hours on *Redesign*.
**Then** the operation is refused with: "Timesheets must be created with at least an active analytic
account defined in the plan 'Projects'." — the placeholder being the name of the plan the project's
account belongs to.
**Rule** TS-024.

---

## D. Validation failures on creation

### D1 — A private task refuses time

**Given** task *Secret Review* has no project (it is private).
**When** anybody tries to record time on it.
**Then** the operation is refused with: "Timesheets cannot be created on a private task."
**Rule** TS-020.

### D2 — An archived or foreign employee refuses time

**Given** employee *Ex Staffer* is archived.
**When** a line is created naming that employee.
**Then** the operation is refused with: "Timesheets must be created with an active employee in the
selected companies."
**And** the same message appears when the named employee belongs to a company that is not in the
active session.
**Rule** TS-021.

### D3 — A mandatory analytic plan not satisfied by the project

**Given** the plan *Departments* is compulsory for the business domain `timesheet`, and *Redesign*
carries no account in it.
**When** *Iris Wolf* records 2 hours on *Redesign*.
**Then** the operation is refused with: "'Departments' analytic plan(s) required on the project
'Redesign' linked to the timesheet." — the first placeholder being the comma-separated names of the
missing plans and the second the project's name.
**Rule** TS-022.

### D4 — A mandatory analytic plan not satisfied by the sales order item

**Given** the sales capability; the plan *Departments* is compulsory for the business domain
`timesheet`; the line is being bound to the item *Consulting Hours — 40 h* whose own analytic
distribution names accounts that do not cover *Departments*.
**When** the line is created.
**Then** the operation is refused with: "'Departments' analytic plan(s) required on the analytic
distribution of the sale order item 'Consulting Hours' linked to the timesheet."
**Rule** TS-023.

### D5 — One company for the line, its accounts, its task and its project

**Given** task *Homepage* belongs to *Northwind Studio* while the analytic account being written
belongs to *Meridian Works*.
**When** the line is created.
**Then** the operation is refused with: "The project, the task and the analytic accounts of the
timesheet must belong to the same company."
**Rule** TS-025; [calculations.md](calculations.md) §1.2.

### D6 — The absence task refuses hand-recorded time

**Given** the absence bridge and the absence task *Time Off* of *Northwind Studio*.
**When** *Iris Wolf* tries to record 4 hours on *Time Off*.
**Then** the operation is refused with: "You cannot create timesheets for a task that is linked to a
time off type. Please use the Time Off application to request new time off instead."
**Rule** TS-026.

### D7 — A person who is not an approver may only record for themselves

**Given** *Iris Wolf* holds Timesheet User only.
**When** she opens the employee selector on a recorded line.
**Then** the only employee offered is *Iris Wolf*.
**When** she nevertheless submits a line naming *Tom Baros*.
**Then** the operation is refused as an access problem with: "You cannot access timesheets that are
not yours."
**Rule** TS-030, TS-043.

### D8 — The project selector excludes what may not be timesheeted

**Given** projects *Redesign* (time-tracked), *Archive 2024* (time tracking off), *Template Kit* (a
template) and *Internal* (the company's internal project).
**When** *Iris Wolf* opens the project selector on a recorded line.
**Then** only *Redesign* is offered: the second is not time-tracked, the third is a template and the
fourth is an internal project.
**Rule** TS-028, TS-108, TS-123.

---

## E. The freeze chain on writing

### E1 — A public-holiday line refuses every write

**Given** the absence bridge and a line generated by the public holiday of 6 April 2026.
**When** *Paul Reiss*, a timesheets administrator, writes its description.
**Then** the write is refused with: "Timesheets linked to public holidays cannot be modified."
**Rule** TS-040.

### E2 — An absence-generated line refuses every write

**Given** the absence bridge and a line carrying an approved absence request.
**When** anybody writes its quantity.
**Then** the write is refused with: "You cannot modify timesheets that are linked to time off
requests. Please use the Time Off application to modify your time off requests instead."
**Rule** TS-041.

### E3 — An invoiced line refuses the six protected fields

**Given** the sales capability; a line of 4 hours bound to an item whose product is invoiced on the
delivered quantity, stamped with the posted invoice *INV/2026/0007*.
**When** its quantity is written as 5.
**Then** the write is refused with: "You cannot modify timesheets that are already invoiced."
**And** the same refusal follows a write of the employee, the project, the task, the bound item or
the date.
**When** instead only the description is written.
**Then** the write succeeds.
**Rule** TS-042.

### E4 — A mixed write refuses as a whole

**Given** E3's invoiced line and an uninvoiced line of 2 hours, both bound to items whose product is
invoiced on delivered quantity.
**When** both are written together with a new date.
**Then** the whole write is refused with "You cannot modify timesheets that are already invoiced.";
**neither** line changes.
**Rule** TS-042, note.

### E5 — A line stamped on a cancelled invoice is writable again

**Given** E3's line, and the invoice *INV/2026/0007* has been cancelled and does not carry the
legacy-invoicing payment status.
**When** its quantity is written as 5.
**Then** the write succeeds, because condition two of the invoiced freeze requires an invoice whose
status is not cancelled.
**Rule** TS-042; [calculations.md](calculations.md) §4.4.

### E6 — Another person's line refuses a write

**Given** *Iris Wolf* holds Timesheet User only.
**When** she writes the description of a line belonging to *Tom Baros*.
**Then** the write is refused as an access problem with: "You cannot access timesheets that are not
yours."
**Rule** TS-043.

### E7 — An archived employee cannot be set on an existing line

**Given** an existing line and the archived employee *Ex Staffer*.
**When** the employee is written as *Ex Staffer*.
**Then** the write is refused with: "You cannot set an archived employee on existing timesheets."
**Rule** TS-044.

### E8 — Writing a non-billable project clears the manual binding

**Given** the sales capability; a line bound by hand to the item *Consulting Hours — 40 h*, with the
manual-binding flag set.
**When** the line is written into project *Atlas*, whose billable switch is off.
**Then** the manual-binding flag becomes false, the automatic resolution runs and yields nothing, and
the line becomes unbound.
**Rule** TS-045.

### E9 — The presented editability flag

**Given** the line of E3, still stamped with a posted invoice.
**Then** the flag `readonly_timesheet` is true, the row is shown greyed, and every editable column is
locked.
**And** for an external collaborator the flag is true on every line, invoiced or not.
**Rule** TS-046.

---

## F. Deletion

### F1 — A public-holiday line cannot be deleted

**Given** the absence bridge and a public-holiday line.
**When** *Paul Reiss* deletes it.
**Then** the deletion is refused with: "You cannot delete timesheets that are linked to global time
off."
**Rule** TS-060.

### F2 — An absence line cannot be deleted, and offers a redirection

**Given** the absence bridge, a line carrying the approved absence request of *Iris Wolf*, and *Iris
Wolf* acting.
**When** she deletes the line.
**Then** the deletion is refused with: "You cannot delete timesheets that are linked to time off
requests. Please cancel your time off request from the Time Off application instead." and the refusal
carries the action "View Time Off" opening that request's own form.
**When** instead *Lena Hoss*, who neither owns the request nor administers absences, deletes it.
**Then** the same message is raised with **no** redirection.
**Rule** TS-061.

### F3 — A line on a posted invoice cannot be deleted

**Given** the sales capability and a line stamped with the posted invoice *INV/2026/0007*.
**When** it is deleted.
**Then** the deletion is refused with: "You cannot remove a timesheet that has already been
invoiced."
**Rule** TS-062.

### F4 — A line on a draft invoice may be deleted

**Given** the same line, with *INV/2026/0007* still a draft.
**When** it is deleted.
**Then** the deletion succeeds, and the invoice line's delivered quantity is recomputed without it.
**Rule** TS-062, note.

### F5 — The bridge deletes what a person cannot

**Given** the absence bridge and an approved absence with five generated lines.
**When** the absence is refused.
**Then** all five lines are deleted, because the bridge clears the absence reference first and
deletes afterwards.
**Rule** TS-063.

### F6 — A project with recorded lines cannot be deleted

**Given** *Redesign* holds 12 recorded lines.
**When** *Paul Reiss* deletes it.
**Then** the deletion is refused with the message beginning "This project has some timesheet entries
referencing it…", carrying the action "See timesheet entries" which opens
`timesheet_action_project` over that project.
**And** when two such projects are deleted at once the message begins "These projects have some
timesheet entries referencing them…".
**Rule** TS-104.

### F7 — A task with visible recorded lines cannot be deleted

**Given** task *Homepage* holds recorded lines that *Paul Reiss* may see.
**When** he deletes it.
**Then** the deletion is refused with the message beginning "This task can’t be deleted because it’s
linked to timesheets…", carrying the action "See timesheet entries".
**When** the lines exist but he may not see them.
**Then** the refusal is instead: "Some timesheet entries are weighing down these tasks! Remove them
first, then you’ll be able to delete the tasks!"
**Rule** TS-121, TS-122.

### F8 — A task with recorded lines cannot become private

**Given** task *Homepage* holds recorded lines.
**When** its project is cleared to make it private.
**Then** the write is refused with: "This task cannot be private because there are some timesheets
linked to it."
**Rule** TS-120.

---

## G. Visibility

### G1 — A Timesheet User sees only their own lines

**Given** *Iris Wolf* holds Timesheet User; *Tom Baros* has recorded 8 hours on *Redesign*.
**When** she opens "All Timesheets".
**Then** her own lines are listed and *Tom Baros*'s 8 hours are not, whatever the action's own
selection says.
**Rule** TS-081.

### G2 — An approver sees the lines the rule admits

**Given** *Tom Baros* holds Timesheet Approver.
**When** he opens "All Timesheets".
**Then** he sees his own lines and *Iris Wolf*'s lines.
**And** the menu entry `timesheet_menu_activity_user` is absent from his menu, because approvers
reach their own lines through the folder instead.
**Rule** TS-082, TS-092.

### G3 — An administrator sees every line of the active companies

**Given** *Paul Reiss* holds the timesheets administrator privilege and both companies are in his
active session.
**When** he opens "All Timesheets".
**Then** he sees the lines of *Northwind Studio* and of *Meridian Works*, and no line of a company
outside the session.
**Rule** TS-083.

### G4 — The attendance comparison is hidden from a non-user

**Given** the attendance capability, and *Nina Okafor* is an internal user with no timesheet
privilege.
**When** she opens the application menu.
**Then** the entry "Timesheets / Attendance Analysis" is absent.
**Rule** TS-093.

### G5 — An external collaborator sees only the lines of their own documents

**Given** project sharing is on and *Joel Willis* is a contact of *Deco Addict*, follower of task
*Homepage*.
**When** he opens `/my/timesheets`.
**Then** he sees the recorded lines of *Homepage* and of any other project or task he follows, and no
other line; and every one of them is presented as non-editable.
**Rule** TS-085, TS-087, TS-046.

### G6 — The generic analytic rules are narrowed

**Given** a person with generic analytic read rights but no timesheet privilege.
**When** they read analytic lines.
**Then** they see analytic lines whose project reference is empty, and none whose project reference
is set.
**Rule** TS-084.

---

## H. Aggregates on tasks and projects

### H1 — Time spent on a task excludes sub-tasks

**Given** task *Homepage* carries lines of 3, 4 and 2.5 hours, and its sub-task *Header* carries a
line of 6 hours.
**Then** the time spent on *Homepage* is **9.5** hours, its sub-task time spent is **6** hours and
its total time spent is **15.5** hours.
**Rule** [calculations.md](calculations.md) §5.1, §5.2, §5.3.

### H2 — Progress, time remaining and overtime

**Given** H1's figures and an allocation of 20 hours on *Homepage*.
**Then** time remaining is `20 − 15.5 = 4.5` hours, progress is `15.5 ÷ 20 = 0.775`, and overtime is
0.
**When** a further line of 8 hours is added to *Homepage*.
**Then** time spent is 17.5, total time spent 23.5, time remaining `20 − 23.5 = −4` hours, progress
`23.5 ÷ 20 = 1.175` and overtime **3.5** hours.
**Rule** [calculations.md](calculations.md) §5.3.

### H3 — A task with no allocation has no progress

**Given** task *Ad hoc* with no allocation and 6 hours recorded.
**Then** progress is 0, time remaining is `0 − 6 = −6` hours and overtime is 0 — the task is not in
overtime because there is nothing to exceed.
**Rule** [calculations.md](calculations.md) §5.3.

### H4 — Archived sub-tasks still count

**Given** *Header* is archived and still carries its 6 hours.
**Then** the sub-task time spent of *Homepage* is still 6 hours.
**Rule** TS-125.

### H5 — The project total, hour encoding

**Given** *Redesign* carries lines totalling 12.5 hours in one group and 3.25 hours in another, all
stored in Hours; the company encodes in hours.
**Then** the project's total recorded time is **15.75**.
**Rule** [calculations.md](calculations.md) §2.4, example A.

### H6 — The project total, day encoding

**Given** the same project with groups of 12 hours and 4 hours, the acting company now encoding in
days.
**Then** the total recorded time is **2.00** days.
**Arithmetic** contributions 12 and 4 are summed to 16 and divided by the encoding factor 8.
**Rule** [calculations.md](calculations.md) §2.4, example B.

### H7 — The project total with a mixed unit, as observed

**Given** *Redesign* carries one group of 12 hours and one group of 1 day, the company encoding in
days.
**Then** the total recorded time is **1.63** days, because the day group's own factor is suppressed
by the encode-in-days flag.
**And** the corrected behaviour would give 2.50 days; a rebuild that produces 2.50 fails this
scenario and satisfies the corrected behaviour.
**Rule** [calculations.md](calculations.md) §2.4, compatibility finding.

### H8 — The project stat button below the allocation

**Given** *Redesign* with 100 hours allocated, 62 hours recorded, the company encoding in hours.
**Then** the "Timesheets" stat button reads "62 / 100 Hours (62%)" in green.
**Rule** [calculations.md](calculations.md) §5.6; [interfaces.md](interfaces.md) §5.6.

### H9 — The project stat button at four fifths and above the allocation

**Given** the same project with 85 hours recorded.
**Then** the button reads "85 / 100 Hours (85%)" in amber.
**When** 118 hours are recorded.
**Then** the button reads "118 / 100 Hours" in red — the percentage is dropped — and a second button
"Extra Time" appears reading "18 Hours (+18%)".
**Rule** [calculations.md](calculations.md) §5.6.

### H10 — A project without an allocation

**Given** *Ad hoc Project* with no allocation and 12 hours recorded, the company encoding in hours.
**Then** the button reads "12 Hours" and no "Extra Time" button appears.
**Rule** [calculations.md](calculations.md) §5.6.

### H11 — The project overtime filter

**Given** *Redesign* with 118 hours recorded against 100 allocated.
**Then** its overtime flag is true and it appears under the project filter "Timesheets >100%".
**Rule** [calculations.md](calculations.md) §5.4; [interfaces.md](interfaces.md) §5.1.

---

## I. Unit conversion and display

### I1 — Hours into days

**Then** 8 hours is 1.0 days; 7.5 hours is 0.94 days; 4 hours is 0.5 days; 1 hour is 0.13 days;
20 hours is 2.5 days; −3 hours is −0.38 days. Each is rounded to two decimals, half away from zero.
**Rule** [calculations.md](calculations.md) §2.3.

### I2 — The total recorded duration on an order, day encoding

**Given** the sales capability; three lines of 7.5, 6 and 4.25 hours bound to items of one order;
project time unit Hours, encoding unit Days.
**Then** the order's "Recorded" stat button shows **2**.
**Arithmetic** `7.5 + 6 + 4.25 = 17.75`; `round(17.75 ÷ 8, 0.01, half up) = 2.22`;
`round(2.22) = 2`.
**Rule** [calculations.md](calculations.md) §2.5.

### I3 — The same order with hour encoding

**Given** I2 with the encoding unit Hours.
**Then** the button shows **18**, because `round(17.75) = 18`.
**Rule** [calculations.md](calculations.md) §2.5.

### I4 — The total recorded duration is hidden from a non-user

**Given** I2 and a person without the Timesheet User privilege.
**Then** the total recorded duration reads 0 for them.
**Rule** [calculations.md](calculations.md) §2.5.

### I5 — The remaining-time suffix on a sales order item, day encoding

**Given** the sales capability; an item with 13 hours of remaining time, the company encoding in days
with a project time unit of hours.
**Then** the selector entry ends with " (1.63 days remaining)".
**Arithmetic** `13 ÷ 8 = 1.625` unrounded at conversion, printed with two decimals as 1.63.
**Rule** [calculations.md](calculations.md) §2.8.

### I6 — The hourly cost shown on a mapping row

**Given** the sales capability; a mapping row whose stored cost is 60.00, the employee following
*Standard 40 hours* with eight hours a day.
**Then** with the company encoding in hours the presented hourly cost is **60.00**; with the company
encoding in days it is **480.00**, and typing 480.00 back stores 60.00.
**Rule** [calculations.md](calculations.md) §3.2.

---

## J. Binding a recorded line to a sales order item

### J1 — Task rate: the task's own item wins

**Given** the sales capability; *Redesign* is billable with no project item and no mapping row (the
pricing mode is therefore the task rate); task *Homepage* carries the item *Consulting Hours — 40 h*.
**When** *Iris Wolf* records 3 hours on *Homepage*.
**Then** the line is bound to *Consulting Hours — 40 h* and its manual-binding flag is false.
**Rule** [calculations.md](calculations.md) §4.2 step 2.1; [state-machines.md](state-machines.md) §1.

### J2 — Project rate: the project's item wins for a line with no task

**Given** *Redesign* carries the project item *Consulting Hours — 40 h* and no mapping row (the
pricing mode is therefore the project rate).
**When** *Iris Wolf* records 3 hours on *Redesign* with no task.
**Then** the line is bound to *Consulting Hours — 40 h*.
**Rule** [calculations.md](calculations.md) §4.2 step 1.2.

### J3 — Employee rate: the mapping wins

**Given** *Redesign* carries a mapping row binding *Iris Wolf* to *Senior Consulting — 20 h* and a
row binding *Tom Baros* to *Junior Consulting — 20 h* (the pricing mode is therefore the employee
rate); task *Homepage* carries *Consulting Hours — 40 h*.
**When** *Iris Wolf* records 3 hours on *Homepage* and *Tom Baros* records 2 hours on the same task.
**Then** her line is bound to *Senior Consulting — 20 h* and his to *Junior Consulting — 20 h*;
neither is bound to the task's own item.
**Rule** [calculations.md](calculations.md) §4.2 step 2.2.

### J4 — Employee rate with no matching row falls back to the task's item

**Given** J3, and *Lena Hoss* has no mapping row.
**When** she records 2 hours on *Homepage*.
**Then** her line is bound to the task's item *Consulting Hours — 40 h*.
**Rule** [calculations.md](calculations.md) §4.2 step 2.2.

### J5 — A manual binding is never overwritten

**Given** J1's line, bound automatically to *Consulting Hours — 40 h*.
**When** *Tom Baros* changes the bound item by hand to *Support Pack — 10 h*.
**Then** the manual-binding flag becomes true.
**When** the task's own item is afterwards changed to *Milestone Build — 1 h*.
**Then** the line stays bound to *Support Pack — 10 h*.
**Rule** TS-045 read in reverse; [calculations.md](calculations.md) §4.1.

### J6 — Adding a mapping row re-bills time already recorded

**Given** *Redesign* is billable and time-tracked; *Iris Wolf* has already recorded 3 lines totalling
9 hours, none manually bound, none invoiced.
**When** a mapping row is created binding *Iris Wolf* to *Senior Consulting — 20 h*.
**Then** all three lines are rewritten to that item, with elevated rights, bypassing the resolution
algorithm.
**And** a line of hers that carries a posted invoice is **not** rewritten.
**Rule** TS-146; [calculations.md](calculations.md) §4.3.

### J7 — Switching billing off unbinds every line

**Given** *Redesign* is billable with 20 bound lines.
**When** the billable switch is turned off.
**Then** every one of the 20 lines becomes unbound and every manual-binding flag becomes false.
**Rule** TS-105.

### J8 — One mapping row per project and employee

**Given** *Redesign* already carries a mapping row for *Iris Wolf*.
**When** a second row for *Iris Wolf* on the same project is created.
**Then** the creation is refused with: "An employee cannot be selected more than once in the mapping.
Please remove duplicate(s) and try again."
**And** the employee selector on a new row does not offer *Iris Wolf* at all.
**Rule** TS-140, TS-142.

### J9 — A billable project's item must be a service and not a re-invoiced cost

**Given** *Redesign* is billable.
**When** its project item is set to an item whose product is a storable good.
**Then** the write is refused with: "You cannot link a billable project to a sales order item that is
not a service."
**When** it is set to an item that is a re-invoiced cost.
**Then** the write is refused with: "You cannot link a billable project to a sales order item that
comes from an expense or a vendor bill."
**Rule** TS-102, TS-103.

### J10 — A typed cost on a mapping row sticks

**Given** a mapping row for *Iris Wolf* whose cost was left to follow her hourly cost of 42.35.
**When** a person types 60.00 into it.
**Then** the manual-cost flag becomes true.
**When** her hourly cost is afterwards changed to 45.00.
**Then** the row still carries 60.00.
**Rule** TS-145.

### J11 — Changing the project's customer clears a mapped item

**Given** a mapping row bound to an item of an order for *Deco Addict*.
**When** the project's customer is changed to *Gemini Furniture*.
**Then** the row's bound item is cleared, because the item no longer belongs to the project's
customer.
**Rule** TS-144.

---

## K. Delivered quantity and invoicing

### K1 — The delivered quantity of a time-tracked item

**Given** the sales capability; the item *Consulting Hours — 40 h* is sold in hours and its product
is invoiced on delivered quantity with time tracking; three lines of 2.5, 3 and 1.25 hours are bound
to it.
**Then** its delivered quantity is **6.75** hours.
**Rule** [calculations.md](calculations.md) §6.2.

### K2 — The same lines on an item sold in days

**Given** the item *Day Rate Consulting — 5 d*, sold in days, with the same three lines recorded in
hours.
**Then** its delivered quantity is **0.84** days.
**Arithmetic** `round(6.75 ÷ 8, 0.01, half up) = round(0.84375) = 0.84`.
**Rule** [calculations.md](calculations.md) §6.2.

### K3 — The same lines on an item sold in the abstract unit

**Given** an item sold in **Units**, which shares no reference unit with Hours, with the same three
lines.
**Then** its delivered quantity is **6.75** Units: the tolerant conversion returns the quantity
unchanged rather than failing.
**Rule** [calculations.md](calculations.md) §6.2.

### K4 — Two groups of units

**Given** an item sold in hours with four bound lines: three of 2 hours stored in Hours and one of
0.5 stored in Days.
**Then** the delivered quantity is **10.00** hours.
**Arithmetic** `6 × 1 ÷ 1 = 6.00` plus `0.5 × 8 ÷ 1 = 4.00`.
**Rule** [calculations.md](calculations.md) §6.2.

### K5 — Invoicing stamps every consumed line

**Given** K1's item with 6.75 hours delivered and nothing invoiced.
**When** a customer invoice is created from the order with the delivered-quantity method and no
period.
**Then** the invoice line carries a quantity of 6.75, and each of the three recorded lines is stamped
with that invoice.
**And** each stamped line becomes frozen for the six protected fields.
**Rule** [workflows.md](workflows.md) §8; TS-163; [state-machines.md](state-machines.md) §2.

### K6 — Which lines an invoice may consume

**Given** the sales capability; the item has four bound lines: one already stamped with a posted
invoice, one stamped with a cancelled invoice that is not legacy-invoiced, and two unstamped.
**When** a new invoice is created.
**Then** the three eligible lines — the cancelled-invoice one and the two unstamped ones — are
consumed and stamped; the posted-invoice one is not.
**Rule** TS-163; [calculations.md](calculations.md) §4.4.

### K7 — Only delivered-quantity service items consume time

**Given** an order carrying *Support Pack — 10 h* (prepaid, invoiced on the ordered quantity) and
*Consulting Hours — 40 h* (invoiced on delivered quantity with time tracking), each with bound lines.
**When** an invoice is created.
**Then** only the lines bound to *Consulting Hours — 40 h* are stamped; the prepaid item's lines are
left unstamped, because a prepaid item's quantity is invoiced from the order and not from recorded
time.
**Rule** [entities.md](entities.md) §11; [workflows.md](workflows.md) §8.

### K8 — The delivered quantity is not disturbed by invoicing

**Given** K5.
**Then** after the invoice is posted the item's delivered quantity is still 6.75; only its invoiced
quantity has changed.
**Rule** TS-167.

---

## L. Period invoicing, credit notes and re-invoicing

### L1 — A period restricts what is billed

**Given** the sales capability; the item *Consulting Hours — 40 h* has unstamped lines of 4 hours on
3 March, 6 hours on 10 March and 5 hours on 20 March 2026.
**When** an invoice is created with the delivered-quantity method and the period 1 March to 15 March
2026.
**Then** the invoice line carries **10.00** hours, the 3 March and 10 March lines are stamped, and
the 20 March line is left for a later invoice.
**And** the item's delivered quantity remains 15 hours.
**Rule** [calculations.md](calculations.md) §6.6, example A; TS-167.

### L2 — An item with nothing in the period is skipped without disturbance

**Given** L1, and a second item *Support Pack — 10 h* whose lines all fall outside the period.
**When** the same invoice is created.
**Then** the second item's quantity to invoice is written as zero and its invoice status is restored
to the value it had before the write, so that it is skipped rather than marked as invoiced.
**Rule** [calculations.md](calculations.md) §6.6 step 6.

### L3 — Posting a credit note releases the lines

**Given** the sales capability; *INV/2026/0007* consumed three lines totalling 6.75 hours and is
posted.
**When** a credit note reversing it is posted, carrying an invoice line for the same item.
**Then** the three lines have their consuming invoice cleared and become eligible for invoicing
again; their bound item is unchanged.
**Rule** TS-164; [state-machines.md](state-machines.md) §2.

### L4 — A credit note caps what a period may re-bill

**Given** the sales capability; an item with 15 hours delivered and 10 hours invoiced on an invoice
that has since been fully credited, so that all 15 hours are again unstamped; the period covers
everything.
**When** a new invoice is created.
**Then** the quantity to invoice is **5.00**, not 15.
**Arithmetic** `max(0, min(15, 15 − 10)) = 5.00`.
**Rule** [calculations.md](calculations.md) §6.6, example B; TS-168.

### L5 — An over-invoiced item is left out

**Given** an item with 8 hours delivered, 12 hours invoiced by hand and a credit note on the item; the
period quantity is 8.
**Then** the quantity to invoice is **0** and the item is left out of the invoice entirely.
**Arithmetic** `max(0, min(8, 8 − 12)) = max(0, −4) = 0`.
**Rule** [calculations.md](calculations.md) §6.6, example C.

### L6 — Deleting a draft invoice line releases its lines

**Given** the sales capability; a draft invoice consuming three lines.
**When** its invoice line is deleted.
**Then** the three recorded lines have their consuming invoice cleared, and their bound item is left
untouched — they are **not** re-bound.
**Rule** TS-165.

### L7 — Reversing with the modify method re-stamps onto the replacement

**Given** the sales capability; a posted invoice consuming three lines.
**When** it is reversed with the modify method, which posts a credit note and creates a replacement
draft invoice.
**Then** the credit note releases the three lines, and the replacement invoice re-stamps the same
three lines onto itself.
**Rule** TS-166.

---

## M. Upselling

### M1 — Crossing the shipped threshold raises one activity

**Given** the sales capability; the item *Support Pack — 40 h* orders 40 hours of a prepaid service
with the shipped threshold of 1; the order is confirmed and has the salesperson *Tom Baros*; the
item's upsell-warning flag is false.
**When** recorded time brings its delivered quantity to 40.5 hours.
**Then** exactly one to-do activity is scheduled on the order, assigned to *Tom Baros*, with the note
"Upsell <the order's link> for customer <the customer's link>"; every outstanding to-do activity on
the order is removed first; and the item's upsell-warning flag becomes true.
**Arithmetic** `40.5 > 40 × 1` compared to two decimals.
**Rule** [calculations.md](calculations.md) §6.7; TS-170.

### M2 — The activity is raised only once

**Given** M1.
**When** a further 2 hours are recorded, bringing the delivered quantity to 42.5.
**Then** no new activity is raised, because the flag is already true.
**Rule** TS-170.

### M3 — A lower threshold raises the opportunity earlier

**Given** the same item with an upselling threshold of 0.8.
**When** the delivered quantity reaches 33 hours.
**Then** the opportunity is raised, at 82.5 % of the ordered quantity.
**Arithmetic** `33.00 > 40 × 0.8 = 32.00`.
**Rule** [calculations.md](calculations.md) §6.7.

### M4 — The rounding edge

**Given** an item ordering 3 hours with a threshold of 1.
**When** the delivered quantity reaches 3.004 hours.
**Then** **no** opportunity is raised, because the comparison is made to two decimals and
`3.00 > 3.00` is false.
**Rule** [calculations.md](calculations.md) §6.7.

### M5 — No salesperson, no activity

**Given** M1's order with no salesperson and a customer with no salesperson.
**When** the delivered quantity exceeds the threshold.
**Then** no activity is raised and no flag is set.
**Rule** TS-171.

### M6 — The flag resets when the overrun disappears

**Given** M1, with the flag true and the delivered quantity at 40.5 hours.
**When** an invoice is created from the order and the delivered quantity afterwards equals exactly
the ordered 40 hours.
**Then** the flag is reset to false, so a later overrun raises a fresh opportunity.
**Rule** [calculations.md](calculations.md) §6.7.

---

## N. Classification and profitability

### N1 — The five recorded-time classifications

**Given** the sales capability and five recorded lines of 2 hours each on *Redesign*, each bound
differently.
**Then** the stored classification is:

| Bound to | Product policy | Classification |
|---|---|---|
| nothing, project billing type "billed manually" | — | `billable_manual` |
| nothing, project billing type non-billable | — | `non_billable` |
| *Consulting Hours — 40 h* | delivered quantity, time tracking | `billable_time` |
| *Milestone Build — 1 h* | delivered quantity, milestones | `billable_milestones` |
| *Support Pack — 10 h* | ordered quantity | `billable_fixed` |

**Rule** [calculations.md](calculations.md) §7.2 branch one.

### N2 — A positive amount on a time-tracked item becomes revenue

**Given** the sales capability; a line of +1.5 hours with an amount of **+45.00** — reachable when a
mapping cost is negative — bound to *Consulting Hours — 40 h*.
**Then** the classification is `timesheet_revenues`, because both the amount and the quantity are
strictly positive.
**When** instead the line carries −2 hours and +84.70 (the correction line of B4).
**Then** the classification is `billable_time`, because the quantity is not strictly positive.
**Rule** [calculations.md](calculations.md) §7.2 branch one step 4.

### N3 — A non-service product leaves the classification empty

**Given** a recorded line bound to an item whose product is a storable good.
**Then** the classification is empty, neither billable nor non-billable.
**Rule** [calculations.md](calculations.md) §7.2 branch one step 3.

### N4 — An ordinary analytic line is classified in the other branch

**Given** an analytic line with **no** project, an amount of 0 and a quantity of 0.
**Then** the classification is `other_revenues`, because branch two tests non-negative.
**And** a recorded line with the same figures is `billable_time`, because branch one tests strictly
positive.
**Rule** [calculations.md](calculations.md) §7.2, note.

### N5 — Profitability sections and their order

**Given** the sales capability; *Redesign* carries lines classified `billable_time` totalling
−1 240.00 and lines classified `timesheet_revenues` totalling +310.00, all in euro.
**Then** the revenues-and-costs panel shows the section "Timesheets (Billed on Timesheets)" with a
cost of −1 240.00 and the section "Timesheets revenues" with a revenue of +310.00; the sections
appear in the order fixed price, billed on timesheets, billed on milestones, billed manually,
non-billable, timesheets revenues; and the first four are foldable.
**Rule** [calculations.md](calculations.md) §7.3, §7.4.

### N6 — A project that is not time-tracked drops the four sections

**Given** project *Atlas* with time tracking off.
**Then** the sections `billable_fixed`, `billable_time`, `billable_milestones` and `billable_manual`
are removed from the revenue side and the remaining revenue rows are re-totalled.
**Rule** [calculations.md](calculations.md) §7.4 step 1.

### N7 — Vendor-bill lines are skipped

**Given** the project's analytic lines include a group whose analytic category is `vendor_bill`.
**Then** that group contributes nothing to the timesheet sections, so its cost is not counted twice.
**Rule** [calculations.md](calculations.md) §7.4 step 4.

### N8 — A section with two zero figures disappears

**Given** a section whose cost and revenue are both zero.
**Then** it is dropped before presentation.
**Rule** [calculations.md](calculations.md) §7.4.

---

## O. The analysis rows

### O1 — One row per recorded line, with revenue

**Given** the sales capability; a line of 3.5 hours bound to *Consulting Hours — 40 h* sold in hours
at 120.00 per hour, with a cost amount of −157.50.
**Then** the analysis row shows revenue **420.00**, billable time **3.5**, non-billable time **0** and
margin **262.50**.
**Arithmetic** `3.5 × 120.00 ÷ 1 × 1 = 420.00`; `420.00 + (−157.50) = 262.50`.
**Rule** [calculations.md](calculations.md) §8.1, §8.2.

### O2 — An item sold in days, a line recorded in hours

**Given** an item sold in days at 900.00 per day and a line of 6 hours.
**Then** the revenue is **675.00**.
**Arithmetic** `6 × 900.00 ÷ 8 × 1 = 675.00`.
**Rule** [calculations.md](calculations.md) §8.1.

### O3 — A fixed-price item invoiced on the ordered quantity

**Given** an item ordering 10 days with a subtotal before tax of 9 000.00, and a line of 6 hours.
**Then** the revenue is **675.00**.
**Arithmetic** unit revenue `9 000.00 ÷ 10 = 900.00`; `6 × 900.00 ÷ 8 × 1 = 675.00`.
**Rule** [calculations.md](calculations.md) §8.1.

### O4 — A milestone service produces no timesheet revenue

**Given** a line of 6 hours bound to *Milestone Build — 1 h*.
**Then** the revenue is **0**, whatever the quantity.
**Rule** [calculations.md](calculations.md) §8.1.

### O5 — An unbound line

**Given** a line of 2 hours with no bound item and a cost amount of −90.00.
**Then** billable time is **0**, non-billable time is **2**, revenue is **0** and margin is
**−90.00**.
**Rule** [calculations.md](calculations.md) §8.2.

### O6 — A line with no unit produces no analysis row

**Given** an analytic line with a project but no unit.
**Then** no analysis row exists for it; the row source joins the unit inclusively.
**Rule** TS-261.

### O7 — The analysis rows are read-only

**When** anybody tries to create, modify or delete an analysis row, through a screen or through the
generic transport.
**Then** the operation is refused; the rows are derived.
**Rule** TS-260.

---

## P. The attendance comparison

### P1 — A day with both attendance and recorded time

**Given** the attendance capability; one employee with an hourly cost of 40.00; on 4 March 2026 two
attendances of 4.25 and 3.5 hours and two recorded lines of 3 and 4 hours.
**Then** the comparison row shows attendance total **7.75**, timesheet total **7.00**, time difference
**0.75**, timesheet cost **280.00**, attendance cost **310.00** and cost difference **30.00**.
**Rule** [calculations.md](calculations.md) §9.

### P2 — A day with attendance but no recorded time

**Given** the same employee, one attendance of 8 hours on 5 March 2026 and no recorded line.
**Then** attendance total is **8.00**, timesheet total is **0**, time difference is **8.00**,
timesheet cost is reported **empty** (not zero), attendance cost is **320.00** and cost difference is
**320.00**.
**Rule** [calculations.md](calculations.md) §9.

### P3 — An employee with no hourly cost

**Given** *Lena Hoss*, 8 hours attended and 8 hours recorded on 6 March 2026.
**Then** the time difference is **0** and all three cost columns are reported **empty**.
**Rule** [calculations.md](calculations.md) §9; TS-265.

### P4 — Future days are excluded

**Given** an attendance and a recorded line dated tomorrow.
**Then** neither appears in the comparison, which stops at today.
**Rule** TS-266.

### P5 — The rows cannot be drilled into

**When** a person clicks a cell of the comparison pivot.
**Then** nothing opens: drilling is switched off, because a comparison row's identifier is not
stable.
**Rule** TS-264; [interfaces.md](interfaces.md) §4.2.

### P6 — The default ordering of a grouped read

**Given** a grouped read of the comparison rows with no explicit ordering, grouped by a part of the
date and by employee.
**Then** the date key is ordered **descending** and the employee key ascending.
**Rule** TS-267; [calculations.md](calculations.md) §9.

---

## Q. The cost per unit used in the margin

### Q1 — The cost comes from recorded time

**Given** the margin capability and the sales capability; the item *Consulting Hours — 40 h* is sold
in hours, its product carries no standard cost, and the lines bound to it total 20 hours and −850.00
of cost; the company's project time unit is Hours; one currency.
**Then** the item's cost per unit is **42.50** per hour.
**Arithmetic** `− (−850.00) ÷ 20 = 42.50`.
**Rule** [calculations.md](calculations.md) §10.

### Q2 — An item with no recorded line keeps the product's cost

**Given** the same item with no bound line and a product standard cost of 30.00.
**Then** the item's cost per unit is **30.00**.
**Rule** [calculations.md](calculations.md) §10 step 2.

### Q3 — An item sold in days, as observed

**Given** Q1's figures but the item sold in **days**, the company's project time unit being Hours.
**Then** the stored cost per unit is **340.00**.
**Arithmetic** `42.50 × 8 ÷ 1 = 340.00`.
**And** the corrected behaviour would give 5.3125 per hour; a rebuild that produces 5.3125 fails this
scenario and satisfies the corrected behaviour.
**Rule** [calculations.md](calculations.md) §10, compatibility finding.

### Q4 — Items excluded from the override

**Given** the margin capability.
**Then** the override does not apply to a re-invoiced cost item, to a confirmed service item whose
policy is prepaid, manual or milestones and whose cost per unit is already non-zero, or to an item
whose product carries a standard cost.
**Rule** TS-173; [calculations.md](calculations.md) §10.

---

## R. The absence bridge

### R1 — Approving a five-day absence generates five lines

**Given** the absence bridge; *Iris Wolf* follows a schedule of four eight-hour days and one
four-hour Friday; she requests absence from Monday 2 March to Friday 6 March 2026; the company has an
internal project and an absence task; the absence type's time classification is not "other".
**When** the request is approved.
**Then** five recorded lines are created on *Internal* / *Time Off*, dated 2, 3, 4, 5 and 6 March,
with quantities 8, 8, 8, 8 and 4, descriptions "Time Off (1/5)" through "Time Off (5/5)", employee
*Iris Wolf*, user *Iris Wolf*, company *Northwind Studio*, and each carrying the absence request.
**And** their total is **36 hours**.
**Rule** [calculations.md](calculations.md) §11.1.

### R2 — A flexible half day

**Given** the absence bridge; a flexible schedule declaring 7.6 hours a day; a half-day request whose
start and end fall on the same date with the same period.
**When** it is approved.
**Then** exactly one line is created, "Time Off (1/1)", of **3.8** hours.
**Rule** [calculations.md](calculations.md) §11.1 step 2.

### R3 — Regeneration never doubles the lines

**Given** R1's five lines.
**When** the request is re-approved, or its schedule is rewritten so that generation runs again.
**Then** the five old lines have their absence reference cleared and are deleted before the new ones
are created; exactly five lines exist afterwards, not ten.
**Rule** TS-221; [calculations.md](calculations.md) §11.1 step 5.

### R4 — Refusing an absence deletes its lines

**Given** R1's five lines.
**When** the request is refused.
**Then** all five lines are deleted.
**Rule** TS-220; [workflows.md](workflows.md) §11.2.

### R5 — A public holiday generates one line per employee

**Given** the absence bridge; a company-wide schedule exception covering the whole of Wednesday
4 March 2026 on *Standard 40 hours*, whose working intervals that day are 08:00–12:00 and
13:00–17:00; four employees follow that schedule and none has an approved absence that day.
**Then** four lines are created, one per employee, each "Time Off (1/1)" of **8** hours dated 4 March,
each carrying the exception.
**Arithmetic** `(12:00 − 08:00) + (17:00 − 13:00) = 8`.
**Rule** [calculations.md](calculations.md) §11.2; TS-225.

### R6 — A half-day public holiday

**Given** the same schedule and an exception from 13:00 to 17:00 on 4 March 2026.
**Then** each line carries **4** hours: the morning interval does not overlap the window.
**Rule** [calculations.md](calculations.md) §11.2.

### R7 — An absence suppresses the public-holiday line for the same day

**Given** R5, and *Iris Wolf* already holds an approved absence covering 4 March 2026.
**Then** only three lines are created; *Iris Wolf* gets none from the holiday, because her approved
absence covers the date.
**Rule** TS-223.

### R8 — Withdrawing the absence restores the suppressed holiday line

**Given** R7.
**When** *Iris Wolf*'s absence is cancelled.
**Then** her absence-generated lines are deleted, and one line of 8 hours dated 4 March carrying the
exception is created for her; no duplicate appears for the employees who already hold one.
**Rule** TS-224; [calculations.md](calculations.md) §11.3.

### R9 — An exception naming one employee generates nothing

**Given** a schedule exception that names the resource of *Tom Baros*.
**Then** no recorded line is generated by it at all.
**Rule** TS-225.

### R10 — Moving a public holiday rebuilds its lines

**Given** R5's four lines on 4 March.
**When** the exception's window is rewritten to Thursday 5 March 2026.
**Then** the four lines of 4 March are removed and four new lines dated 5 March are created.
**Rule** TS-226.

### R11 — Deleting a public holiday takes its lines with it

**Given** R5's four lines.
**When** the exception is deleted.
**Then** the four lines are deleted in cascade, and any absence that had been shortened by the
holiday is regenerated.
**Rule** TS-227; [configuration.md](configuration.md) §11.

### R12 — Creating an employee generates their future holiday lines

**Given** the absence bridge and a company-wide exception dated 6 April 2026, which is in the future.
**When** a new employee following *Standard 40 hours* is created.
**Then** lines for 6 April are generated for that employee.
**And** archiving that employee deletes every exception-carrying line of theirs dated today or later.
**Rule** TS-245; [calculations.md](calculations.md) §11.4.

### R13 — Changing an employee's working schedule re-sizes the future lines

**Given** an employee with generated holiday lines of 8 hours on 6 April 2026.
**When** their working schedule is changed to a half-time schedule of 4 hours a day.
**Then** the future lines are deleted and regenerated at **4** hours.
**Rule** [calculations.md](calculations.md) §11.4.

### R14 — Absence lines never become a favourite project

**Given** *Iris Wolf*'s most recent recorded line is an absence-generated line on *Internal*.
**When** she opens a new recorded line.
**Then** the defaulted project is her most recent **non-absence** project, not *Internal*.
**Rule** TS-228.

---

## S. The employee life cycle

### S1 — Deleting an employee with no recorded line

**Given** *Nina Okafor* is an employee with no recorded line.
**When** *Paul Reiss* uses the "Delete" entry.
**Then** the removal dialogue opens showing "Are you sure you want to delete these employees?" with
the buttons "Ok" and "Discard"; pressing "Ok" deletes the employee and returns the employee list.
**Rule** TS-243; [interfaces.md](interfaces.md) §5.8, §6.5.

### S2 — Deleting an active employee who holds recorded lines

**Given** *Iris Wolf* is active and holds 40 recorded lines.
**When** *Paul Reiss* uses the "Delete" entry.
**Then** the dialogue shows "Uh-oh! The employee you’re trying to delete still has some timesheets
hanging around and can’t be deleted." followed by "So here are your two options: either delete those
timesheets or consider archiving the employee instead.", with the buttons "Archive Employees" and
"See Timesheets"; "Archive Employees" opens the employee-termination dialogue and does not itself
archive.
**Rule** TS-243.

### S3 — Deleting an archived employee who holds recorded lines

**Given** *Ex Staffer* is archived and holds recorded lines; *Paul Reiss* is an approver.
**Then** the dialogue shows the first sentence followed by "Please first delete all of their
timesheets.", and offers "See Timesheets" as the primary button; there is no archive button.
**Rule** TS-243.

### S4 — A person who is not an approver is refused before the dialogue opens

**Given** *Nina Okafor* is not a Timesheet Approver; the selected employees hold recorded lines and
none of them is active.
**When** she uses the "Delete" entry.
**Then** the operation is refused with: "You cannot delete employees who have timesheets." and no
dialogue opens.
**Rule** TS-242.

### S5 — The recorded-time button on an employee

**Given** *Iris Wolf* holds recorded lines.
**Then** her employee form shows a "Timesheets" stat button, and pressing it opens her lines with her
employee defaulted.
**When** she is archived.
**Then** the same screen opens but creation is not offered.
**Rule** TS-240, TS-246; [interfaces.md](interfaces.md) §6.4.

### S6 — Employee display in a session spanning several companies

**Given** one user is linked to an employee in *Northwind Studio* and to another in *Meridian Works*,
and both companies are in the active session.
**Then** each employee's display label is suffixed with its company's name, so that the two can be
told apart on a recorded line.
**Rule** TS-244.

---

## T. External pages

### T1 — The external home page card

**Given** project sharing is on and *Joel Willis* may see 17 recorded lines.
**When** he opens the external home page.
**Then** a card titled "Timesheets" appears with the text "Review all timesheets related to your
projects", a counter reading 17, and a link to `/my/timesheets`.
**Rule** [interfaces.md](interfaces.md) §7.2.

### T2 — The recorded-time page pages at one hundred rows

**Given** *Joel Willis* may see 250 lines.
**When** he opens `/my/timesheets`.
**Then** the first hundred are shown, ordered newest first, and the pager offers three pages;
`/my/timesheets/page/3` shows the last fifty.
**Rule** [interfaces.md](interfaces.md) §7.3.

### T3 — Group totals ignore the page

**Given** T2, grouped by project, with one project holding 180 of the 250 lines totalling 640 hours.
**Then** the group header for that project reads "Total: " followed by 640 hours on **every** page,
not by the sum of the rows visible on the page.
**Rule** [interfaces.md](interfaces.md) §7.3.

### T4 — The default grouping depends on the capability

**Given** the base capability alone.
**Then** `/my/timesheets` opens with the grouping "None".
**Given** the sales capability.
**Then** it opens with the grouping "Sales Order Item".
**Rule** [interfaces.md](interfaces.md) §7.3.

### T5 — An unknown search target selects nothing

**When** `/my/timesheets` is called with a search target that is not one of the offered keys.
**Then** the page shows no row at all, rather than every row.
**Rule** [interfaces.md](interfaces.md) §7.3.

### T6 — The bound-item group header carries the ordered and remaining time

**Given** the sales capability; the grouping is "Sales Order Item"; the group's item orders 40 hours
of a prepaid service with 26.5 hours delivered; the company encodes in hours.
**Then** the header reads the item's label followed by "(40.00 Hours Ordered, 13.50 Hours
Remaining)".
**Arithmetic** remaining time `(40 − 26.5) × 1 ÷ 1 = 13.5`.
**Rule** [interfaces.md](interfaces.md) §7.3; [calculations.md](calculations.md) §6.3.

### T7 — The external task page totals

**Given** *Joel Willis* opens task *Homepage*, which has 20 hours allocated, 9.5 hours of its own
time, 6 hours on sub-tasks and 15.5 hours in total; the company encodes in hours.
**Then** the page shows "Total Time Spent: " 9.5, "Time recorded on sub-tasks: " 6, "Total Hours: "
15.5 and "Time Remaining: " 4.5; and "Progress:" reads 78 %.
**Rule** [interfaces.md](interfaces.md) §7.5; [calculations.md](calculations.md) §5.8.

### T8 — Switching the external card off hides everything

**Given** the external hiding capability and project sharing on.
**When** the customization `portal_my_home_timesheet` is deactivated.
**Then** the external home page card disappears, the "Timesheets" section and the "View Details"
button disappear from every task page, the "Time Spent" column disappears from the task list, the
bound-item and invoice columns disappear from the recorded-time page, and the "View Timesheets"
buttons disappear from the order and invoice pages.
**Rule** TS-088; [interfaces.md](interfaces.md) §7.2.

### T9 — The "View Timesheets" button on an order

**Given** the sales capability; a confirmed order with recorded time and external display on.
**Then** the external order page shows a "View Timesheets" button opening `/my/timesheets` with the
search target set to the order and the search text set to the order's reference.
**Rule** [interfaces.md](interfaces.md) §7.7.

### T10 — The task invoice route

**Given** the sales capability; task *Homepage* whose order carries three invoices.
**When** *Joel Willis* opens `/my/tasks/<the task identifier>/orders/invoices`.
**Then** the three invoices are listed, and the first hundred of them become his invoice browsing
history.
**When** he calls the same route with a task identifier that matches nothing.
**Then** a not-found response is returned.
**Rule** [interfaces.md](interfaces.md) §7.1.

### T11 — An external collaborator cannot edit

**Given** *Joel Willis* opens a recorded line from a shared task.
**Then** the external form shows the employee, the project and the task as read-only, and the
editability flag is true for him on every line.
**Rule** TS-046; [interfaces.md](interfaces.md) §3.6.

---

## U. Import, export and printing

### U1 — The import workbook is offered only on a timesheet list

**Given** *Iris Wolf* opens "All Timesheets", whose context carries the timesheet marker.
**Then** the import offer includes "Import Template for Timesheets".
**When** she opens an ordinary analytic line list, whose context does not carry the marker.
**Then** the offer is empty.
**Rule** [interfaces.md](interfaces.md) §6.1, §10.3.

### U2 — An import obeys every creation rule

**Given** an import file whose row names project *Redesign*, employee *Iris Wolf*, 3 hours and an
empty description, and a second row naming the archived employee *Ex Staffer*.
**When** the file is imported.
**Then** the first row creates a line whose description is `/` and whose amount is −127.05
(`− (3 × 42.35)`), and the second row is refused with "Timesheets must be created with an active
employee in the selected companies."
**Rule** [interfaces.md](interfaces.md) §10.3; TS-002, TS-021.

### U3 — The export carries stored values

**Given** the company encodes in days and a line of 4 hours.
**When** the layout "Timesheets" is exported.
**Then** the exported quantity is **4** — the stored value in the project time unit — not 0.5.
**Rule** [interfaces.md](interfaces.md) §10.2.

### U4 — The printed table and its total, hour encoding

**Given** three lines of 2.5, 3 and 1.25 hours on one task, the company encoding in hours.
**When** the task's "Timesheets" document is printed.
**Then** the table lists the three lines with their dates, employees, descriptions and times rendered
as hours and minutes, and the final row reads "Total (Hours)" followed by 6 hours 45 minutes.
**Rule** [interfaces.md](interfaces.md) §8.2.

### U5 — The printed table, day encoding

**Given** the same three lines, the company encoding in days.
**Then** the final row reads "Total (Days)" followed by **0.84** days.
**Arithmetic** `round(6.75 ÷ 8, 2 decimals) = 0.84`.
**Rule** [interfaces.md](interfaces.md) §8.2; [calculations.md](calculations.md) §2.3.

### U6 — The document file name

**Given** a set of recorded lines all belonging to task *Homepage*.
**Then** the document is named "Timesheets - Homepage".
**When** the set spans two tasks.
**Then** it is named "Timesheets".
**Rule** [interfaces.md](interfaces.md) §8.4.

### U7 — The sub-task sections of a task document

**Given** task *Homepage* with lines of its own, sub-task *Header* with lines, and *Header*'s own
sub-task *Logo* with lines.
**When** the task document is printed.
**Then** it shows *Homepage*'s table, then a heading "Sub-Task of 'Homepage': Header" with *Header*'s
table, then a heading "Sub-Task of 'Header': Logo" with *Logo*'s table, walking the tree in sequence
order to any depth.
**Rule** [interfaces.md](interfaces.md) §8.3.

### U8 — A document over an invoice

**Given** the sales capability; the posted invoice *INV/2026/0007* consumed six lines.
**Then** the invoice's "Timesheets" document heading reads "Timesheets for the INV/2026/0007 Invoice"
and lists the six lines.
**When** the invoice is still a draft and has no reference.
**Then** the heading reads "Timesheets for the Draft Invoice".
**Rule** [interfaces.md](interfaces.md) §8.3.

---

## V. Several companies and several currencies

### V1 — The company is forced from the task, then the project

**Given** *Redesign* belongs to *Northwind Studio* and task *Homepage* belongs to it as well.
**When** a line is created supplying a company of *Meridian Works*.
**Then** the stored company is *Northwind Studio*, forced from the task.
**When** a write attempts to clear the company.
**Then** the write is silently dropped and the company stays *Northwind Studio*.
**Rule** TS-006.

### V2 — Each company keeps its own units

**Given** *Northwind Studio* encodes in hours and *Meridian Works* in days, both storing in hours.
**When** *Paul Reiss* has both companies in his session and looks at a line of 4 hours on a
*Meridian Works* project.
**Then** the session payload carries an encoding factor of 1 for *Northwind Studio* and 0.125 for
*Meridian Works*, and the line is displayed according to the company it belongs to.
**Rule** [calculations.md](calculations.md) §3.1; [configuration.md](configuration.md) §7.

### V3 — The internal project is suffixed in a session spanning companies

**Given** both companies have an internal project named "Internal" and both are in the session.
**Then** each internal project's display label is suffixed with its company's name.
**Rule** TS-107.

### V4 — A cost in a foreign currency

**Given** V2 and *Nils Berg*, whose hourly cost is 55.00 United States dollars, recording 6 hours on
a project whose analytic account carries the euro; the rate on the line's date is 1.0837.
**Then** the stored amount is **−357.62** euro.
**Rule** [calculations.md](calculations.md) §1.7.

### V5 — Profitability converts at today's rate, not the line's

**Given** V4's line and a project whose currency is the euro.
**Then** the profitability panel converts the group's amount from the group's currency into the
project's currency using **today's** rate, while the line's own stored amount used the rate of the
line's date; the two figures may therefore differ.
**Rule** [calculations.md](calculations.md) §7.4 step 5, §1.4.

### V6 — A mapping row spanning companies

**Given** the sales capability; a mapping row is created for an employee who has no company set.
**Then** the employee is given the project's company as a default, so that the line the mapping later
produces passes the single-company check.
**Rule** TS-247.

---

## W. Rounding edges and boundary cases

### W1 — The half-up rounding of a cost

**Given** an hourly cost of 42.35 and a quantity of 7.5.
**Then** the raw amount is −317.625 and the stored amount is **−317.63**: the magnitude is rounded
away from zero at the half.
**Rule** [calculations.md](calculations.md) §1.5.

### W2 — Converting exactly one hour into days

**Then** 1 hour is **0.13** days, not 0.125: the day conversion rounds to two decimals.
**Rule** [calculations.md](calculations.md) §2.3.

### W3 — A conversion between unrelated units passes through

**Given** an item sold in **Units** and a line recorded in Hours.
**Then** the delivered quantity takes the hours figure unchanged; nothing fails and nothing is scaled.
**Rule** [calculations.md](calculations.md) §2.2, §6.2.

### W4 — Reading the abstract unit as hours

**Given** the sales capability; a confirmed item ordering 12 Units, whose generated project's company
stores in Hours.
**Then** the allocation is **12 hours**: the abstract unit is rewritten to the Hours unit before the
conversion, and the two are then identical.
**Rule** [calculations.md](calculations.md) §6.4.

### W5 — The allocation summed from several items

**Given** the sales capability; one order carries three service items sharing one project template:
10 hours, 2 days and 5 Units; the company stores in Hours.
**Then** the generated project's allocation is **31 hours**, its time tracking is on and its billable
switch is on.
**Arithmetic** `10 × (1 ÷ 1) + 2 × (8 ÷ 1) + 5 × (1 ÷ 1) = 31`.
**Rule** [calculations.md](calculations.md) §6.5; TS-172.

### W6 — The same order for a company storing in days

**Given** W5 with the company's project time unit set to Days.
**Then** the allocation is **3.875 days**, unrounded.
**Arithmetic** `10 × (1 ÷ 8) + 2 × (8 ÷ 8) + 5 × (1 ÷ 8) = 1.25 + 2 + 0.625`.
**Rule** [calculations.md](calculations.md) §6.5.

### W7 — A template with its own allocation wins

**Given** W5, but the product's project template already carries an allocation of 50 hours.
**Then** the generated project's allocation is **50 hours** and no sum is computed.
**Rule** [calculations.md](calculations.md) §6.5 step 1.

### W8 — The remaining time on an item sold in a pack unit

**Given** the sales capability; an item sold in a "pack" unit worth 10 hours, ordering 3 packs with
1.25 delivered, whose product's service policy is prepaid.
**Then** the remaining time is **17.5** hours, unrounded.
**Arithmetic** `(3 − 1.25) × 10 ÷ 1 = 17.5`.
**Rule** [calculations.md](calculations.md) §6.3.

### W9 — The remaining time is empty when it is not meaningful

**Given** an item whose product's service policy is "Based on Timesheets" rather than prepaid.
**Then** the "remaining time is meaningful" flag is false and the remaining time is empty; the
"Time Remaining on SO" figure is not shown anywhere for that item.
**Rule** TS-162; [calculations.md](calculations.md) §6.3.

### W10 — A quantity above twenty-four is accepted but flagged

**Given** *Iris Wolf* records 30 hours on a single day.
**Then** the line is created — no rule forbids it — and every list and form shows the quantity in red.
**Rule** TS-004; [interfaces.md](interfaces.md) §3.1.

### W11 — A future date is accepted

**Given** *Iris Wolf* records 4 hours dated 31 December 2027.
**Then** the line is created; the date is not constrained to the past.
**Rule** TS-003.

### W12 — Merging is offered only for unposted lines

**Given** the sales capability; a selection of four lines, two of which carry posted invoices.
**When** the generic merge operation is offered.
**Then** only the two lines that carry no posted invoice are candidates.
**Rule** TS-064.

### W13 — Aggregates are recomputed, never adjusted

**Given** a task with 15.5 hours of total time spent.
**When** a line of 2 hours is deleted.
**Then** every aggregate is recomputed from the surviving lines and reads 13.5; no figure is adjusted
by subtracting the deleted quantity from a stored total.
**Rule** TS-272.

### W14 — There is no document-level lock

**Given** two people editing two different lines of the same task at the same time.
**Then** neither is blocked; the domain has no document lock, and the aggregates settle to the same
value whichever write commits last.
**Rule** TS-270, TS-271.

### W15 — Elevated writes bypass the freezes deliberately

**Given** an invoiced and therefore frozen line.
**When** the invoicing step itself stamps or un-stamps it, with elevated rights.
**Then** the write succeeds; the freezes are checks on ordinary writes, not database constraints.
**Rule** TS-273.

---

## X. Conformance summary

A rebuild passes this suite when, for every scenario above, it produces the stated records, the
stated amounts to the stated number of decimals, the stated states and the stated messages
character for character. Two scenarios — H7 and Q3 — assert an **observed** behaviour that the
specification records as a compatibility finding; a rebuild that deliberately implements the
corrected behaviour named in each of them fails those two and passes the rest, and must say so in
its own conformance statement.
