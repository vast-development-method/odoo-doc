# Acceptance criteria of the Projects and Tasks domain

Numbered Given / When / Then scenarios with concrete numbers. A re-implementation is behaviourally
equivalent when every one of them passes.

**Common fixture.** Unless a scenario says otherwise:

- one company, *Horizon Labs*, currency euro, whose working schedule declares, Monday to Friday,
  a morning block 08:00–12:00 (4.0 declared hours, 0.5 declared days) and an afternoon block
  13:00–17:00 (4.0 declared hours, 0.5 declared days); no leave in the period;
- the shipped project stages To Do (10), In Progress (15), Done (20, folded) and Cancelled
  (25, folded);
- a project **Website Revamp**, visibility "All internal users and invited portal users",
  customer *Deco Addict*, manager *Mona Ellis*, with the task stages **Backlog** (sequence 1, not
  folded, 0 days to rot), **In Development** (sequence 2, not folded, 7 days to rot) and
  **Delivered** (sequence 3, folded, 0 days to rot);
- users: *Mona Ellis* (Project Administrator), *Ravi Kapoor* and *Anita Sen* (Project User),
  *Bram De Vos* (Project User), *Nina Okafor* (internal user, no Project privilege),
  *Joel Willis* (portal person, contact of *Deco Addict*), and an unauthenticated visitor;
- dates are in 2026. 2 March 2026 is a Monday.

---

## A. Projects

### A1 — Creating a project applies every default

**Given** *Mona Ellis* is a project administrator and the project-stages privilege is **not**
granted.
**When** she creates a project named `Atlas Migration` with no other value.
**Then** the project has: visibility `portal`, manager *Mona Ellis*, task label "Tasks",
sequence 10, active true, no stage, no company (no analytic account and no customer supply one),
currency euro (the acting company's), last update status `to_define`, last update colour 0,
task count 0, open task count 0, closed task count 0, task completion percentage 0, and portal
address `/my/projects/<its identifier>`.
**And** the project has no task stage.

### A2 — Creating a project on the fly creates one stage

**Given** the same.
**When** *Mona Ellis* types `Atlas Migration` into a project picker and confirms the creation from
the name alone.
**Then** exactly one task stage named `New` is created and attached to the new project.

### A3 — The task label may not be empty

**Given** A1's project.
**When** the task label is written as an empty text.
**Then** the stored value is `Tasks`.

### A4 — The expiration date may not precede the start date

**Given** A1's project.
**When** the start date is written as 1 June and the expiration date as 31 May.
**Then** the write is refused with: "The project's start date must be before its end date."

### A5 — The two project dates behave as a pair

**Given** A1's project, which has neither date.
**When** only the start date is written as 1 June.
**Then** **nothing is stored**: the start-date key is dropped because no expiration date is stored
on the record and none is being written.
**When** instead both are written, 1 June and 31 August.
**Then** both are stored.
**When** afterwards the expiration date is written as empty.
**Then** **both** dates become empty.

### A6 — The project stage default follows the lowest sequence

**Given** the project-stages privilege is granted and the four shipped project stages exist.
**When** *Mona Ellis* creates a project with no stage.
**Then** its stage is **To Do** (sequence 10).

### A7 — A project stage of another company is refused

**Given** two companies *Horizon Labs* and *Northwind*, and a project stage **Review** whose
company is *Northwind*.
**When** a project whose company is *Horizon Labs* is saved with stage **Review**.
**Then** the save is refused with: "This project is associated with Horizon Labs, whereas the
selected stage belongs to Northwind. There are a couple of options to consider: either remove the
company designation from the project or from the stage. Alternatively, you can update the company
information for these records to align them under the same company."
**When** instead the project has **no** company.
**Then** the message is: "This project is not associated with any company, while the stage is
associated with Northwind. There are a couple of options to consider: either change the project's
company to align with the stage's company or remove the company designation from the stage".

### A8 — Changing a project stage's company is refused while projects of another company sit in it

**Given** the project stage **Review** with no company, containing one project of *Horizon Labs*.
**When** the stage's company is written as *Northwind*.
**Then** the write is refused with: "You are not able to switch the company of this stage to
Northwind since it currently includes projects associated with Horizon Labs. Please ensure that
this stage exclusively consists of projects linked to Northwind."

### A9 — Archiving a project archives its tasks

**Given** *Website Revamp* with five active tasks and two already archived.
**When** the project is archived.
**Then** all seven tasks are archived.
**When** the project is unarchived.
**Then** all seven tasks are active.

### A10 — Deleting a project

**Given** *Website Revamp* with an analytic account **WR-001** that has no analytic line, and five
tasks of which two have sub-tasks.
**When** the project is deleted.
**Then** all five tasks and all of their descendants are deleted, the project is deleted, and
**WR-001** is deleted.
**When** instead **WR-001** carries one analytic line.
**Then** the project and its tasks are deleted but **WR-001** survives.

### A11 — The analytic account is renamed with the project, but only when it is exclusive

**Given** projects *Website Revamp* and *Brand Refresh*, both pointing at the analytic account
**Shared**.
**When** *Website Revamp* is renamed to *Website Rebuild*.
**Then** **Shared** keeps its name.
**When** instead *Website Revamp* is the only project pointing at **Shared** and is renamed.
**Then** **Shared** is renamed to *Website Rebuild*.

### A12 — Changing the company of a project with analytic lines is refused

**Given** *Website Revamp* with analytic account **WR-001** carrying three analytic lines, company
*Horizon Labs*.
**When** the project's company is written as *Northwind*.
**Then** the write is refused with: "The project's company cannot be changed if its analytic
account has analytic lines or if more than one project is linked to it."

### A13 — Deleting an analytic account with tasks behind it is refused

**Given** **WR-001** linked to *Website Revamp*, which has one task.
**When** **WR-001** is deleted.
**Then** the deletion is refused with: "Before we can bid farewell to these accounts, you need to
tidy up the projects linked to them by removing their existing tasks!"

### A14 — Duplicating a project

**Given** *Website Revamp*, active, milestone feature **on**, dependency feature on, with two
followers, two milestones, and three root tasks T1, T2, T3, where T3 is blocked by T1 and T1 has
one sub-task; one task is archived; two assignees, one of whom is an inactive account.
**When** the project is duplicated.
**Then** the copy is named `Website Revamp (copy)`; it has the same two followers with the same
subtype selections; it has two new milestones; it has three root tasks plus the copied sub-task,
all in state `01_in_progress`, all keeping their original stage and their original title; the
archived task's copy is **active**; the copy of T3 is blocked by the **copy** of T1, not by T1;
the copied tasks' assignee lists contain only the active assignee; each copied task's milestone
points at the copy of the original's milestone.
**When** instead the source project has the milestone feature **off**.
**Then** the copy has **no** milestone.

### A15 — Feature flags grant a privilege group to everyone

**Given** no project in the database has the milestone feature on, and no internal user holds the
"Use Milestones" privilege.
**When** *Mona Ellis* switches the milestone feature on for *Website Revamp*.
**Then** the "Use Milestones" privilege becomes an implied privilege of the base internal-user
group, so *Nina Okafor* now holds it too.
**When** the feature is later switched off on *Website Revamp* and no other project has it.
**Then** the implication is removed and the privilege's explicit member list is cleared.

### A16 — Switching off the dependency feature releases waiting tasks

**Given** *Website Revamp* with the dependency feature on, task T5 in state `04_waiting_normal`
because T3 blocks it.
**When** the dependency feature is switched off on the project.
**Then** T5's state is `01_in_progress`, and the two "Task Waiting" subtypes become hidden if no
other project has the feature.

### A17 — Switching off the recurrence feature clears the recurrent switches

**Given** *Website Revamp* with the recurrence feature on and two recurring tasks.
**When** the feature is switched off.
**Then** both tasks have the "Recurrent" switch false. The recurrence records themselves are not
deleted by this operation.

---

## B. Tasks — creation and defaults

### B1 — A task created inside a project

**Given** *Website Revamp* and *Ravi Kapoor* (project user, who follows the project).
**When** he creates a task `Wireframes` on the project's board with no other value.
**Then** the task has: stage **Backlog** (the first unfolded stage), state `01_in_progress`,
company *Horizon Labs*, customer *Deco Addict* (inherited from the project), assignee
*Ravi Kapoor* (because no default project was carried in the acting context and no assignee was
given), assignment date the creation moment, ending date empty, last stage update the creation
moment, priority `0`, sequence 10, and a generated security token (the project's visibility is in
the portal range).
**And** the "Task Created" subtype is posted with the body "A new task has been created in the
"Website Revamp" project."

### B2 — A task created from the board of a project does not self-assign

**Given** the same, but the board carries the project as a default in the acting context.
**When** *Ravi Kapoor* creates a task with no assignee.
**Then** the task has **no** assignee and no assignment date.

### B3 — A private to-do always has its creator as an assignee

**Given** *Anita Sen* (project user).
**When** she creates a task `Prepare the quarterly board pack` with no project and no assignee.
**Then** the assignee list is exactly {*Anita Sen*}, the stage is empty, the customer is empty, the
state is `01_in_progress`, and one personal stage assignment exists: (task, *Anita Sen*, her first
personal stage, which is "Inbox").
**When** instead she creates it with the assignee *Ravi Kapoor* only.
**Then** the assignee list is {*Ravi Kapoor*, *Anita Sen*} — the creator is added.

### B4 — A task created with a project does not force the creator in

**Given** *Anita Sen*.
**When** she creates a task in *Website Revamp* with the assignee *Ravi Kapoor* only.
**Then** the assignee list is exactly {*Ravi Kapoor*}.

### B5 — A stage may not be set on a private to-do

**Given** B3's private to-do.
**When** a write sets the stage to **Backlog** without also setting a project.
**Then** the write is refused with: "You can only set a personal stage on a private task."

### B6 — Quick-creation shortcuts

**Given** *Website Revamp*; a tag `feature` exists; no tag `v16` exists; exactly one user matches
the name `Mitchell`.
**When** the display name `Improve the configuration screen #feature #v16 @Mitchell !` is written.
**Then** the title becomes `Improve the configuration screen`, the tag list is
{`feature`, a newly created `v16`}, the assignee list is {*Mitchell*}, and the priority is `1`.
**When** instead two users match `Mitchell`.
**Then** the assignee list is empty and the title becomes
`Improve the configuration screen @Mitchell`.
**When** the title ends in `!!!!` (four marks).
**Then** no priority is extracted and the pattern does not match at all, so nothing is parsed.

### B7 — A private task may not have a parent

**Given** B3's private to-do and any other task.
**When** a parent is written on it.
**Then** the write is refused with: "A private task cannot have a parent."

### B8 — A task with sub-tasks may not become private

**Given** a task T1 in *Website Revamp* with one sub-task.
**When** T1's project is cleared.
**Then** the write is refused with: "This task has sub-tasks, so it can't be private."

### B9 — A recurring task may not become a sub-task

**Given** a recurring task in *Website Revamp*.
**When** a parent is written on it.
**Then** the write is refused with: "You cannot convert this task into a sub-task because it is
recurrent."

### B10 — A task may not be its own parent, nor create a cycle

**Given** tasks A and B where A is the parent of B.
**When** A's parent is written as A.
**Then** the write is refused with: "Sorry. You can't set a task as its parent task."
**When** A's parent is written as B.
**Then** the write is refused with: "Error! You cannot create a recursive hierarchy of tasks."

### B11 — Customer and company must agree

**Given** a contact *Northwind Supplies* whose company is *Northwind*, and a task whose company is
*Horizon Labs*.
**When** that contact is written as the task's customer.
**Then** the write is refused with: "The task and the associated partner must be linked to the
same company."
**And** from the contact's side, writing *Northwind* as the company of a contact who is already
the customer of a *Horizon Labs* task is refused with: "Partner company cannot be different from
its assigned tasks' company"

### B12 — Sub-task display and roll-up

**Given** task A in *Website Revamp*, allocated time 0; children B (4.0 h) and C (2.5 h), both in
the same project; C has a child D (8.0 h).
**Then** A's sub-task allocated time is 6.5 and C's is 8.0.
**And** B and C are **not** displayed in the project in their own right (they share A's project);
D is not displayed either.
**When** B's project is changed to another project *Brand Refresh*.
**Then** B becomes displayed in the project, and A's sub-task count still counts B.
**When** A is archived.
**Then** C and D are archived (they are not displayed in the project) but B is **not**.

---

## C. Tasks — the state machine

### C1 — Blocking a task

**Given** *Website Revamp* with the dependency feature on; task T3 in `01_in_progress`; task T5 in
`01_in_progress`.
**When** T5's "Blocked by" is set to T3.
**Then** T5's state becomes `04_waiting_normal`; T5's blocking count is 1 and closed blocking
count 0; T3's dependent task count is 1.

### C2 — Unblocking

**Given** C1's result.
**When** T3's state is written to `1_done`.
**Then** T5's state becomes `01_in_progress`; T5's closed blocking count becomes 1; T3's dependent
task count becomes 0 (it counts only open dependents).

### C3 — Forcing a blocked task closed and reopening it

**Given** C1's result.
**When** T5's state is written to `1_done`.
**Then** it is stored as `1_done` — a blocked task may be forced closed.
**When** T5's state is then written to `01_in_progress` while T3 is still open.
**Then** the stored value is corrected to `04_waiting_normal`.
**When** T5's state is written to `1_canceled` instead.
**Then** it is stored as `1_canceled`.

### C4 — A cycle is refused

**Given** C1's result.
**When** T3's "Blocked by" is set to T5.
**Then** the write is refused with: "Two tasks cannot depend on each other."

### C5 — A task is never created waiting

**Given** an acting context whose default state is `04_waiting_normal`.
**When** a task is created.
**Then** its state is `01_in_progress`.

### C6 — Changing the project resets the state, unless it is waiting or closed

**Given** three tasks in *Website Revamp*: T1 in `03_approved`, T2 in `04_waiting_normal`, T3 in
`1_done`.
**When** all three are moved to *Brand Refresh* (by a write that does not set the state).
**Then** T1 becomes `01_in_progress`; T2 stays `04_waiting_normal`; T3 stays `1_done`.

### C7 — Reparenting does not reset the state

**Given** task T1 in `03_approved`.
**When** a parent is written on it.
**Then** its state remains `03_approved`.

### C8 — The stage and the state are independent

**Given** task T1 in stage **Backlog**, state `01_in_progress`.
**When** T1 is moved to **Delivered** (a folded stage).
**Then** its ending date becomes the current moment and its last stage update is stamped, but its
state remains `01_in_progress`.
**When** instead T1's state is written to `1_done` without moving it.
**Then** its stage remains **Backlog** and its ending date remains empty.

### C9 — Notification subtypes per state

**Given** T1 in `01_in_progress`.
**When** its state is written to `1_done`.
**Then** the "Task Done" subtype is posted, and the project-level "Task Done" subtype reaches the
project's followers.
**When** both the state and the stage change in the same write.
**Then** the **stage** subtype is posted, not the state subtype.

---

## D. Tasks — the date metrics

### D1 — The five-task scenario

**Given** the common fixture's working schedule, *Website Revamp*, and five tasks all created
Monday 2 March 09:00.

| Task | Assigned | Ended |
|---|---|---|
| T1 | Mon 2 Mar 11:00 | Wed 4 Mar 10:00 |
| T2 | Tue 3 Mar 09:00 | Fri 6 Mar 16:00 |
| T3 | Tue 3 Mar 09:00 | Tue 17 Mar 11:00 |
| T4 | Mon 9 Mar 09:00 | never |
| T5 | never | never |

**Then** the four stored figures are:

| Task | Working hours to assign | Working days to assign | Working hours to close | Working days to close |
|---|---|---|---|---|
| T1 | 2.00 | 0.250 | 17.00 | 2.125 |
| T2 | 8.00 | 1.000 | 38.00 | 4.750 |
| T3 | 8.00 | 1.000 | 90.00 | 11.250 |
| T4 | 40.00 | 5.000 | 0.00 | 0.000 |
| T5 | 0.00 | 0.000 | 0.00 | 0.000 |

### D2 — A public leave is subtracted

**Given** D1 and a company-wide leave covering the whole of Thursday 5 March.
**Then** T2's figures to close become 30.00 hours and 3.750 days; T3's become 82.00 hours and
10.250 days; T1's and T4's are unchanged.

### D3 — No working schedule means no metric

**Given** a project whose company has no working schedule and whose acting company has none
either.
**Then** all four figures of every task of that project are 0.00.

### D4 — Staleness

**Given** T3 of D1, which entered **In Development** (7 days to rot) on Tuesday 3 March at 15:00
and is not closed.
**Then** on Tuesday 10 March at 14:59 it is not stale; at 15:01 it is stale.
**And** on Tuesday 17 March at 11:00 its "days rotting" figure is **13**.
**When** T3's state is written to `1_done`.
**Then** it is no longer stale, because the staleness filter excludes closed tasks.

### D5 — Days to deadline

**Given** the current moment 10 March 12:00 in coordinated universal time and a task with deadline
13 March 18:00.
**Then** the analysis row reports 3.25.
**And** for a task with deadline 8 March 12:00 the figure is −2.00.

### D6 — Per-stage duration map

**Given** a task created Monday 2 March 09:00 in **Backlog**, moved to **In Development** on
Monday 2 March 14:00 and to **Delivered** on Wednesday 4 March 10:00, read on Wednesday 4 March
12:00.
**Then** the map is `{ Backlog: 18000, In Development: 158400, Delivered: 7200 }` seconds.

---

## E. Recurrence

### E1 — Four further occurrences of a weekly recurrence

**Given** a project *Operations* with the recurrence feature on and stages **To Do** (first),
**Doing**, **Done** (folded). A task `Weekly backup verification`, identifier 101, deadline
Friday 6 March 17:00, priority `2`, one assignee, stage **To Do**, recurrence: every 1 week, type
"until", end date Friday 3 April.
**When** occurrence 101 is set to `1_done` on 6 March.
**Then** occurrence 102 is created with deadline 13 March 17:00, priority **`0`**, stage
**To Do**, the same assignee, the same title (no " (copy)" suffix), the same recurrence.
**When** 102, 103 and 104 are closed in turn.
**Then** 103 (20 March), 104 (27 March) and 105 (3 April) are created.
**When** 105 is closed on 3 April.
**Then** **no** further occurrence is created, because 3 April + 1 week = 10 April is after the
end date.
**And** each of the five tasks reports "Tasks in Recurrence" = 5.

### E2 — Closing an older occurrence produces nothing

**Given** E1 after occurrence 103 exists.
**When** occurrence 102 is reopened and closed again.
**Then** no occurrence is created, because 102 is not the highest-identifier occurrence.

### E3 — A recurring task with no deadline

**Given** a recurring task with type "until", end date 3 April, and **no** deadline.
**When** it is closed.
**Then** a next occurrence **is** created, with no deadline, because the guard passes on the
absent deadline.

### E4 — Monthly clamping

**Given** a monthly recurrence, type "forever", occurrence 1 with deadline 31 January 12:00.
**When** it is closed.
**Then** occurrence 2 has deadline **28 February 12:00**.
**When** occurrence 2 is closed.
**Then** occurrence 3 has deadline 28 March 12:00 — the slip is not reversed.

### E5 — Recurrence constraints

**When** a recurrence is saved with interval 0.
**Then** it is refused with: "The interval should be greater than 0".
**When** a recurrence is saved with type "until" and an end date in the past.
**Then** it is refused with: "The end date should be in the future".

### E6 — Deleting occurrences

**Given** E1's five occurrences.
**When** occurrence 103 is deleted.
**Then** only 103 disappears; the recurrence survives; the remaining four keep the switch.
**When** occurrence 105 (the highest identifier) is deleted.
**Then** the recurrence record is deleted and 101, 102 and 104 have the "Recurrent" switch false.

### E7 — The next occurrence's stage when the project has no stage

**Given** a recurring task in a project with an **empty** stage collection.
**When** it is closed.
**Then** the next occurrence keeps the source task's own stage.

---

## F. Milestones

### F1 — A milestone reached

**Given** *Website Revamp* with the milestone feature on, and a single milestone **Design
sign-off**, sequence 10, deadline Friday 13 March, with three attached tasks T1, T2, T6. On
Wednesday 4 March, T1 is `1_done`, T2 and T6 are `01_in_progress`.
**Then**: milestone task count 3; done task count 1; can be marked as reached **false**; deadline
exceeded false; project milestone count 1; milestones reached 0; milestone progress 0; next
milestone = **Design sign-off**; project "can mark a milestone as reached" false; project
"milestone deadline exceeded" false; project "milestone exceeded" false.
**When** T2 is set `1_done` on 10 March and T6 is set `1_canceled` on 11 March.
**Then** done task count 3; can be marked as reached **true**; project "can mark a milestone as
reached" **true**.
**When** the clock reaches Monday 16 March without the milestone being ticked.
**Then** deadline exceeded **true**; project "milestone exceeded" **true**; project "milestone
deadline exceeded" **false** (the walk requires the exceeded milestone to have an open task or no
closed task).
**When** the milestone is ticked on 16 March.
**Then** reached true; reached date 16 March; deadline exceeded false; milestones reached 1;
milestone progress **100**; next milestone empty; project "milestone exceeded" false; project
"can mark a milestone as reached" false.

### F2 — Milestone progress truncates

**Given** a project with 3 milestones of which 2 are reached.
**Then** the progress is **66**, not 67.
**Given** 5 reached out of 8.
**Then** the progress is **62**.

### F3 — A saved milestone with no task

**Given** a saved milestone with no attached task, not reached.
**Then** "can be marked as reached" is **false**.
**Given** the same milestone before it is saved.
**Then** the value is **true**.

### F4 — Unticking re-stamps the reached date

**Given** F1's milestone, reached on 16 March.
**When** it is unticked on 20 March.
**Then** the reached date is empty.
**When** it is ticked again on 25 March.
**Then** the reached date is **25 March**, not 16 March.

### F5 — Milestone propagation to sub-tasks

**Given** *Website Revamp* with milestones **M1** and **M2**; task A with children B, C, D, all in
the project and not closed; B has no milestone; C has **M1** (the same as A); D has **M2**.
**When** A's milestone is written from **M1** to **M2**.
**Then** B receives **M2** (it had none); C receives **M2** (it shared A's previous milestone);
D keeps **M2** (it is unchanged); a closed child would have been skipped.

### F6 — A milestone of another project is rejected

**Given** task T1 in *Website Revamp* and a milestone **X** belonging to *Brand Refresh*.
**When** **X** is written on T1.
**Then** T1's milestone is stored as **empty**.

### F7 — Milestone delivered quantity

**Given** the sales-linked package, a sales order item for 10 consultancy days tracked by
milestones, with three milestones of quantities 3, 5 and 2 (fractions 0.3, 0.5, 0.2).
**When** the first and the third are reached.
**Then** the item's delivered quantity is `(0.3 + 0.2) × 10 = 5`.

---

## G. Ratings

### G1 — An external customer rates a task

**Given** *Website Revamp* with customer *Deco Addict*; task T1 assigned to exactly one user
*Abigail Peterson*; the stage **Delivered** configured with the rating switch on, status "when
reaching this stage", a rating template, and "Automatic Kanban Status" on.
**When** T1 is moved into **Delivered** on 4 March at 10:00.
**Then** a rating row is created with customer *Deco Addict*, rated operator *Abigail Peterson*,
rated record T1, parent record *Website Revamp*, value 0, consumed false, and a fresh
32-character hexadecimal token; and the request message is sent immediately in *Deco Addict*'s
language.
**When** *Deco Addict* opens `/rate/<token>/10` and submits the comment "Exactly what we wanted".
**Then** the address value 10 maps to the rating value **5**; the row becomes value 5, comment
"Exactly what we wanted", consumed true, rated-on stamped, text grade `top`; a chatter message is
posted on T1 with the happy face image and the comment, authored by *Deco Addict*; and T1's state
becomes **`03_approved`** because 5 ≥ 4 and the stage has automatic status.
**And** T1 reports: last rating value 5, last rating text Happy, rating count 1, average 5.0,
average text Happy, satisfaction 100.
**And** *Website Revamp* reports: rating count 1, average 5.0, satisfaction 100, "show ratings"
true; and its stat button shows the smiling face with the number `5 / 5`.

### G2 — A second, low rating

**Given** G1, plus a second task rated **2** by the same customer.
**Then** the project reports count 2, average 3.5, satisfaction 50, average text `ok`
(2.33 ≤ 3.5 < 3.66), and the stat button shows the neutral face with the number `3.5 / 5`.
**And** the second task's state becomes `02_changes_requested` if its stage has automatic status.

### G3 — The 30-day window applies to the project only

**Given** G2, and the clock advanced so that the first rating's last write is 31 days old.
**Then** the project reports count 1 and average 2.0; the **task** aggregates of the first task are
unchanged (count 1, average 5.0).

### G4 — No rating at all

**Given** a task and a project with no consumed rating.
**Then** both report count 0, average 0.0, average text "Not Rated yet", and satisfaction **−1**.

### G5 — An invalid rating value

**When** `/rate/<token>/7` is opened.
**Then** an error is raised reading: "Incorrect rating: should be 1, 3 or 5 (received 7)".
**When** the submission endpoint is posted with the value 4.
**Then** the same error is raised with "(received 4)".

### G6 — A signed-in visitor of another company

**Given** a rating whose customer is *Deco Addict*, and a signed-in portal person whose commercial
parent is *Northwind Supplies*.
**When** that person opens `/rate/<token>/10`.
**Then** the "invalid partner" page is rendered, naming the entity label and the record's display
name, and no rating is applied.

### G7 — The rating request is skipped

**Given** a stage with the rating switch on and a template.
**When** a task with **no** customer enters it.
**Then** no rating is sent.
**When** a task whose customer is the acting user's own contact enters it.
**Then** no rating is sent.
**When** a **template** task enters it.
**Then** no rating is sent.

### G8 — Re-using an unconsumed rating

**Given** a task with one unconsumed rating for *Deco Addict*.
**When** a second rating request is sent for the same customer on the same task.
**Then** **no** new rating row is created; the existing token is re-used.

### G9 — The periodic job

**Given** stage **In Development** with the rating switch on, status "on a periodic basis",
frequency "Once a Month", and a rating request deadline of 1 March 08:00; the stage is attached to
projects P and Q; P has 4 tasks and Q has 3, of which only 2 are currently in **In Development**.
**When** the daily job runs on 1 March at 09:00.
**Then** rating requests are attempted on **all 7** tasks of P and Q, not only the 2 in the
stage; and the stage's deadline is recomputed to 1 March 09:00 + 30 days = 31 March 09:00.

---

## H. The incoming message gateway

### H1 — A message creates a task with its sender as customer

**Given** project *Support Intake* with the incoming address `support@example-domain`, visibility
"All internal users and invited portal users", stages **New** (first, unfolded), **Assigned**,
**Closed** (folded). The contact *Joel Willis* exists with the address
`joel.willis@client-domain`. The contact *Abigail Peterson* exists with the address
`abigail.peterson@example-domain` and has an internal user. The address
`procurement@client-domain` matches no contact.
**When** a message arrives from `joel.willis@client-domain`, to `support@example-domain` and
`abigail.peterson@example-domain`, with carbon copy `procurement@client-domain` and subject
`Login page throws an error`.
**Then** a task is created in *Support Intake* with: title `Login page throws an error`, allocated
time 0.0, **customer *Joel Willis***, assignee *Abigail Peterson*, carbon copy
`procurement@client-domain`, stage **New**, state `01_in_progress`, assignment date stamped, a
generated security token, last stage update stamped.
**And** the project's own address is **not** turned into a contact.
**And** *Joel Willis* is subscribed as a follower.
**And** no invitation is sent for `procurement@client-domain`, because it resolves to no contact.
**And** the task's description is filled from the sanitised message body with the signature
removed, because the task had no description, the creation subtype was posted, the customer equals
the author, the message is of kind electronic mail and it has a body.

### H2 — No author

**Given** the same project.
**When** a message arrives from `newcomer@client-domain`, an address matching no contact.
**Then** a contact **is created** from that address, becomes the message's author and becomes the
task's customer.

### H3 — A recipient without a user account

**Given** the same project, plus a contact *Priya Raman* with the address
`priya@client-domain` and **no** user account.
**When** a message arrives addressed to `support@example-domain` and `priya@client-domain`.
**Then** *Priya Raman* is **not** an assignee; her formatted address is appended to the
carbon-copy list.

### H4 — A carbon-copy contact with an internal user

**Given** the same project, plus *Abigail Peterson* in carbon copy rather than in the recipient
list.
**Then** after creation she receives the notification "You have been invited to follow Login page
throws an error" and is subscribed as a follower.

### H5 — No subject

**When** a message with no subject arrives.
**Then** the task's title is `No Subject`.

### H6 — The reply address

**Given** H1's task.
**Then** replies are directed to the **project's** address, not to a task-specific one.

---

## I. Private tasks and sharing with a second assignee

### I1 — A private task shared with a second assignee

**Given** *Anita Sen* and *Bram De Vos*, both project users. *Anita Sen* creates a private to-do
`Prepare the quarterly board pack`.
**Then** at creation: no project, no stage, state `01_in_progress`, assignees {*Anita Sen*},
customer empty, assignment date the creation moment, one personal stage assignment (task,
*Anita Sen*, "Inbox"), followers {*Anita Sen*'s contact}.
**And** *Bram De Vos* can neither read nor write the task.
**When** *Anita Sen* adds *Bram De Vos* to the assignees.
**Then** the assignment date is **unchanged** (the task already had an assignee); a second personal
stage assignment (task, *Bram De Vos*, his first personal stage) is created — creating his seven
default personal stages first if he had none; *Bram De Vos*'s contact is auto-subscribed as a
follower; he receives "You have been assigned to Prepare the quarterly board pack"; and he can now
read and write the task.
**And** *Anita Sen* moving the task to her "Today" column does not change *Bram De Vos*'s column.
**When** *Anita Sen* removes herself from the assignees.
**Then** her personal stage assignment disappears with her assignee link; the assignment date is
unchanged; she remains a follower.
**When** the last assignee is removed.
**Then** the assignment date becomes **empty** and no personal stage assignment remains.

### I2 — A project administrator cannot read someone else's private to-do

**Given** I1's task before *Bram De Vos* is added, and *Mona Ellis* (project administrator) who is
neither an assignee nor a follower.
**Then** reading the task raises an access error.

### I3 — Personal stages are private

**Given** *Anita Sen*'s seven personal stages.
**Then** *Bram De Vos* cannot read, write or delete any of them, and neither can *Mona Ellis*,
because the global filter restricts the stage list to those with no owner or owned by the acting
user.

### I4 — Deleting the last personal stage is refused

**Given** *Anita Sen* with exactly one personal stage.
**When** that stage is deleted.
**Then** the deletion is refused with: "Each user should have at least one personal stage. Create
a new stage to which the tasks can be transferred after the selected ones are deleted."

### I5 — Personal stage reassignment on deletion

**Given** *Anita Sen* with personal stages of sequences 1 (Inbox), 2 (Today), 3 (This Week),
4 (This Month), 5 (Later), 6 (Done), 7 (Cancelled), and one task in "This Week".
**When** "This Week" is deleted.
**Then** the task moves to **"Today"** — the nearest remaining stage with a lower sequence.
**When** instead "Inbox" is deleted and a task sits in it.
**Then** that task moves to **"Today"** — the nearest remaining stage, since none has a lower
sequence.

### I6 — A personal stage may not be attached to a project

**When** a task stage is saved with both an owner and a project.
**Then** the save is refused with: "A personal stage cannot be linked to a project because it is
only visible to its corresponding user."

---

## J. Project updates

### J1 — A project update with its generated summary

**Given** *Website Revamp*, billable, analytic account present, currency euro, task count 5, open
task count 2. The milestone feature is on with **Design sign-off** (deadline 13 March, reached
16 March), **Go live** (deadline moved from 20 April to 30 April since the last update, not
reached) and **Content freeze** (created since the last update, deadline 10 April). A previous
update exists, created 1 March, progress 20, status "On Track". Profitability:
`service_revenues` invoiced 12 000.00 / to invoice 3 000.00; `other_invoice_revenues` invoiced
800.00 / to invoice 0.00; `other_purchase_costs` billed −4 500.00 / to bill −500.00;
`other_costs_aal` billed −1 200.00 / to bill 0.00.
**When** *Mona Ellis* opens the update form on 20 March.
**Then** the progress defaults to 20, the status defaults to "On Track", and the description is
pre-filled with:

1. the **Summary** heading and the prompt "How's this project going?";
2. the **Activities** heading (the milestone section has content);
3. the **Profitability** heading, the revenues table

   | Revenues | Expected | To Invoice | Invoiced |
   |---|---|---|---|
   | Other Services | €15,000.00 | €3,000.00 | €12,000.00 |
   | Customer Invoices | €800.00 | €0.00 | €800.00 |
   | **Total Revenues** | **€15,800.00** | **€3,000.00** | **€12,800.00** |

   the costs table

   | Costs | Expected | To Bill | Billed |
   |---|---|---|---|
   | Vendor Bills | −€5,000.00 | −€500.00 | −€4,500.00 |
   | Other Costs | −€1,200.00 | €0.00 | −€1,200.00 |
   | **Total Costs** | **−€6,200.00** | **−€500.00** | **−€5,700.00** |

   and the total row €9,600.00 / 61 %, €2,500.00 / 83 %, €7,100.00 / 55 %;
4. the **Milestones** heading, with the checklist ("Design sign-off" ticked, suffix
   "(due 13 March 2026 - reached on 16 March 2026)" with the reached date in red; "Content freeze"
   unticked with "(due 10 April 2026)" in grey; "Go live" unticked with "(due 30 April 2026)" in
   grey), the sentence "Since 1 March 2026 (last project update), the deadline for the following
   milestone has been updated:" with the entry "Go live (20 April 2026 ⇒ 30 April 2026)", and the
   sentence "The following milestone has been added:" with the entry "Content freeze
   (due 10 April 2026)".

**When** the update is saved with the title "Sprint 3 review".
**Then** it stores task count **5** and closed task count **3** (5 − 2); its closed-task percentage
is **60**; the project's last-update link points at it; the project's last update status is
"On Track" and its colour 20.

### J2 — Writing the project's status creates an update

**Given** *Website Revamp* with last update status "On Track".
**When** the project's last update status is written to `at_risk` on 20 March.
**Then** **no** direct write happens on the field; instead a Project Update is created with title
`Status Update - 20/03/2026` (rendered in the acting language's date format) and status
`at_risk`; the project's status then recomputes to `at_risk` and its colour to 22.
**When** the status is written to `to_define`.
**Then** the field is written directly and **no** update is created.

### J3 — Deleting updates

**Given** a project with updates U1 (1 March, "On Track") and U2 (20 March, "At Risk"); the
project's status is "At Risk".
**When** U2 is deleted.
**Then** the project's last-update link points at U1 and the status becomes "On Track".
**When** U1 is also deleted.
**Then** the link is empty and the status becomes "Set Status" with colour 0.

### J4 — Update defaults with no previous update

**Given** a project with no update.
**When** the update form is opened.
**Then** the progress defaults to 0 and the status defaults to **"On Track"** (not "Set Status",
which an update may never carry).

### J5 — The profitability block requires the administrator privilege

**Given** J1, but *Ravi Kapoor* (project user, not administrator) opens the form.
**Then** the profitability values are empty and the **Profitability** block is omitted from the
generated body.

---

## K. Profitability

### K1 — Analytic lines with no journal item behind them

**Given** *Website Revamp* in euro, company currency euro, with four analytic lines on its account
and no journal item behind them: −250.00 EUR, −100.00 EUR, +40.00 EUR, −90.00 USD at 0.90 euro per
dollar.
**Then** the contract contains
`{ other_revenues_aal, 14, invoiced 40.00, to_invoice 0.00 }` and
`{ other_costs_aal, 15, billed −431.00, to_bill 0.00 }`.

### K2 — Sales order items

**Given** *Website Revamp*, billable, euro, the time-recording package **not** installed, and
three confirmed sales order items: a prepaid service product with 9 000.00 invoiced and 3 000.00
to invoice; a consumable product with 2 000.00 invoiced and 0.00 to invoice; an advance-invoice
item with 1 500.00 invoiced.
**Then** the revenues list contains
`{ service_revenues, 6, invoiced 9 000.00, to_invoice 3 000.00 }`,
`{ materials, 7, invoiced 2 000.00, to_invoice 0.00 }` and
`{ downpayments, 20, invoiced 1 500.00, to_invoice −1 500.00 }`, with the totals
`invoiced 12 500.00` and `to_invoice 1 500.00`.
**When** the time-recording package is installed.
**Then** the first entry becomes `{ billable_fixed, 1, … }` with the same figures.

### K3 — Invoice lines and cost of goods sold

**Given** three customer invoice lines carrying the project's analytic account at 100 %: posted
ordinary income with balance −600.00; draft ordinary income with balance −200.00; posted
cost-of-goods-sold on an expense account with balance +350.00.
**Then** the contract contains
`{ other_invoice_revenues, 9, invoiced 600.00, to_invoice 200.00 }` and
`{ cost_of_goods_sold, 21, billed −350.00, to_bill 0.00 }`.
**When** the analytic share on the first line is 60 % instead of 100 %.
**Then** the invoiced revenue becomes 360.00.
**When** an invoice and a credit note cancel out exactly, so both figures are zero.
**Then** **no** entry is produced at all.
**When** a cost-of-goods-sold line's account is **not** an expense account.
**Then** the line is dropped entirely.

### K4 — Purchase orders

**Given** one confirmed purchase order line for 5 000.00 EUR at 100 % to the project, with one
posted bill line of 3 000.00 EUR and one draft bill line of 1 000.00 EUR, both 100 %, neither a
refund.
**Then** the contract contains `{ purchase_order, 10, billed −3 000.00, to_bill −2 000.00 }`.
**When** a posted refund line of 500.00 EUR is added against the same order line.
**Then** the entry becomes `{ purchase_order, 10, billed −2 500.00, to_bill −2 000.00 }`.
**When** instead no bill line exists at all.
**Then** the entry is `{ purchase_order, 10, billed 0.00, to_bill −5 000.00 }`.

### K5 — Vendor bills outside a purchase order

**Given** two vendor bill lines carrying the project's analytic account: posted with balance
+1 200.00 at 100 %, draft with balance +300.00 at 50 %.
**Then** the contract contains
`{ other_purchase_costs, 11, billed −1 200.00, to_bill −150.00 }`.
**And** every bill line already consumed by K4 is excluded from this section.

### K6 — Timesheet merging

**Given** a revenue entry `{ billable_time, 2, invoiced 0.00, to_invoice 4 000.00 }` already
present, and timesheet analytic lines producing, after conversion: `billable_time` −1 350.00,
`non_billable` −480.00, `timesheet_revenues` +900.00.
**Then** a **cost** entry `{ billable_time, 2, billed −1 350.00, to_bill 0.00 }` is added; the
existing revenue entry is unchanged; `{ non_billable, 5, billed −480.00, to_bill 0.00 }` and
`{ timesheet_revenues, 6, invoiced 900.00, to_invoice 0.00 }` are added.
**When** the project does not allow time recording.
**Then** the four `billable_*` **revenue** entries are removed and the revenue totals are
recomputed from the survivors.

### K7 — Derived totals and percentages

**Given** revenues total invoiced 12 800.00 and to invoice 3 000.00; costs total billed −5 700.00
and to bill −500.00.
**Then**: costs −6 200.00; revenues 15 800.00; margin 9 600.00; to bill plus to invoice 2 500.00;
billed plus invoiced 7 100.00; expected percentage 61; to-bill-to-invoice percentage 83;
billed-invoiced percentage 55; margin percentage 155.

### K8 — The panel is hidden for a non-billable project

**Given** *Website Revamp* with the billable flag off and the sales-linked package installed.
**Then** the panel document contains no profitability keys and the derived document is empty.

### K9 — The side-panel document is empty without the privilege

**Given** *Nina Okafor*, an internal user with no Project privilege.
**When** the panel document is requested for *Website Revamp*.
**Then** an empty document is returned.

---

## L. Visibility and access

### L1 — Visibility "Invited internal users"

**Given** *Website Revamp* with visibility `followers`, and task T1 in it.
**Then**:

| Actor | Read the project | Read T1 | Write T1 |
|---|---|---|---|
| *Nina Okafor* (internal, no Project privilege, not a follower) | refused | refused | refused |
| *Nina Okafor* after being subscribed to the project | allowed | allowed | refused (no access right) |
| *Ravi Kapoor* (project user, not a follower, not an assignee) | refused | refused | refused |
| *Ravi Kapoor* after being subscribed to **T1** only | refused | allowed | allowed |
| *Ravi Kapoor* as an assignee of T1 | refused | allowed | allowed |
| *Joel Willis* (portal) even after being subscribed to the project | refused | refused | refused |
| *Mona Ellis* (administrator, not a follower) | **allowed** | **allowed** | **allowed** |

### L2 — Visibility "All internal users"

**Given** *Website Revamp* with visibility `employees`.
**Then** *Nina Okafor* and *Ravi Kapoor* can read the project and every one of its tasks without
following anything; *Ravi Kapoor* can also write, create and delete tasks in it; *Joel Willis*
can read nothing, even after being subscribed to the project.

### L3 — Visibility "All internal users and invited portal users"

**Given** *Website Revamp* with visibility `portal`, task T1.
**Then** every internal user can read the project and its tasks.
**And** *Joel Willis*, subscribed to the **project** only, can read the project but **not** T1 —
the portal task filter requires him to follow the task or to hold an unlimited collaborator row.
**When** he is subscribed to **T1**.
**Then** he can read T1 but not write it.

### L4 — Visibility "Invited internal and portal users"

**Given** *Website Revamp* with visibility `invited_users`, task T1.
**Then** *Ravi Kapoor*, *Nina Okafor* and *Joel Willis* can read neither the project nor T1 until
they are made followers; *Mona Ellis* can read both without following.
**When** each is subscribed to the project.
**Then** each can read the project.
**When** each is subscribed to T1.
**Then** each can read T1.

### L5 — Changing the visibility away from the portal range

**Given** *Website Revamp* with visibility `portal`, *Joel Willis* subscribed to the project and
to T1, with a collaborator row and a security token on T1.
**When** the visibility is written to `employees`.
**Then** *Joel Willis* is unsubscribed from the project and from every task; every task's security
token is cleared; the project's security token is cleared. The collaborator row itself survives
but grants nothing, because every portal filter now fails.

### L6 — Changing the visibility into the portal range

**Given** *Website Revamp* with visibility `employees`, customer *Deco Addict*, and two tasks with
customers.
**When** the visibility is written to `portal`.
**Then** *Deco Addict* is subscribed as a follower of the project, and each task with a customer
subscribes that customer as a follower of the task.

### L7 — A project user cannot create a task in a project they do not follow

**Given** *Website Revamp* with visibility `followers` and *Ravi Kapoor* not following it.
**When** he creates a task in it.
**Then** the creation is refused with an access error.
**When** he is subscribed to the project and retries.
**Then** the creation succeeds.

### L8 — Nobody below administrator may write a project

**Given** *Website Revamp* with visibility `employees`.
**When** *Ravi Kapoor* writes the project's name.
**Then** the write is refused with an access error.
**When** he writes the "show on dashboard" flag.
**Then** the write succeeds, because it is applied through an elevated helper.

### L9 — A project user cannot rename a project stage

**Given** the task stage **Backlog** with no owner.
**When** *Ravi Kapoor* (project user, not administrator) renames it.
**Then** the write is refused: the only filter granting him write on stages requires the stage to
be owned by him.
**When** *Mona Ellis* renames it.
**Then** the write succeeds.

### L10 — Subscribing to a project does not subscribe to existing tasks

**Given** *Website Revamp* with three existing tasks.
**When** *Nina Okafor* is subscribed to the project.
**Then** she is **not** a follower of any of the three.
**When** a fourth task is created afterwards.
**Then** she **is** a follower of it, because she holds the "Task Created" project subtype.

### L11 — Unsubscribing from a project unsubscribes from its tasks

**Given** *Nina Okafor* following the project and, individually, task T1.
**When** she is unsubscribed from the project.
**Then** she is unsubscribed from T1 as well, and any collaborator row she had on the project is
deleted.

### L12 — A portal person cannot reach a project update

**Given** *Joel Willis*, a collaborator on *Website Revamp*.
**When** he attempts to read any project update.
**Then** the read is refused — the portal access-right row for updates grants nothing.

---

## M. Project sharing

### M1 — Granting "edit" to a new person

**Given** *Website Revamp* with visibility `portal`, five tasks, and *Joel Willis* who is a
shareable contact with an existing portal account and who does not yet follow the project.
**When** *Mona Ellis* opens the sharing dialogue, adds *Joel Willis* at level **Edit** with the
invitation box ticked, and confirms.
**Then**: a collaborator row is created with the limited flag **false**; **all five tasks**
subscribe *Joel Willis* as a follower; he is subscribed to the project; the two dormant portal
security records are **activated** (this is the first collaborator in the database); and he
receives the public link.
**And** he can now open the embedded application, see and edit **every** active task of the
project, create tasks in it, and toggle his own followership on any task. He cannot delete a task.

### M2 — Granting "edit with limited access"

**Given** the same, at level **Edit with limited access**.
**Then**: a collaborator row is created with the limited flag **true**; the tasks are **not**
mass-subscribed; he is subscribed to the project.
**And** he can open the embedded application but sees only the tasks he follows; the follow button
is hidden for him.

### M3 — Granting "read"

**Given** the same, at level **Read**.
**Then**: **no** collaborator row is created; he is subscribed to the project, and to every task
whose customer is him or one of his children; he receives the generic sharing message.
**And** he can open the project's customer-portal page and the pages of the tasks he follows, but
cannot edit anything and cannot open the embedded application.

### M4 — Lowering and removing

**Given** M1's result.
**When** the level is lowered to **Read**.
**Then** the collaborator row is deleted; he remains a follower.
**When** he is removed from the list entirely.
**Then** he is unsubscribed from the project and every remaining collaborator row of his on it is
deleted.
**When** he was the only collaborator in the database.
**Then** the two portal security records are **deactivated**.

### M5 — Duplicate collaborators

**When** the same contact is added twice as a collaborator on the same project.
**Then** the second row is refused with: "A collaborator cannot be selected more than once in the
project sharing access. Please remove duplicate(s) and try again."

### M6 — A person who is not shareable

**When** a contact that is not flagged as shareable is added at level **Edit**.
**Then** no collaborator row is created for them; they are silently skipped. The subscription to
the project still happens.

### M7 — Sign-up links

**Given** the sign-up scope is "on invitation" and a listed contact at level **Edit** has **no**
user account.
**When** the sharing is confirmed and the invitation requested.
**Then** a confirmation dialogue is shown first; on confirmation, contacts with a user receive the
public link and contacts without one receive a sign-up link.

### M8 — The embedded application refuses a non-collaborator

**Given** *Website Revamp* with visibility `portal` and no collaborator row for *Joel Willis*.
**When** he opens `/my/projects/<identifier>/project_sharing`.
**Then** the response is "not found".

### M9 — The embedded session is single-company

**Given** M1's result and *Website Revamp* in company *Horizon Labs*.
**When** *Joel Willis* opens the embedded application.
**Then** the session description lists exactly one permitted company, *Horizon Labs*, marks it
current, injects the project's identifier and name, injects the full currency table, and carries
the project's milestone and dependency flags.

### M10 — Field-level restriction

**Given** M1's result.
**When** *Joel Willis* writes the task's **title**.
**Then** the write succeeds — the title is on the writable list.
**When** he writes the task's **allocated time**.
**Then** the write is refused, because the field is on neither list.
**When** he reads the task's **assignee names**.
**Then** the read succeeds — the field is on the readable list, and it is computed with elevated
rights.
**When** he reads the task's **allocated time**.
**Then** the read is refused.
**When** he creates a sub-task and the sharing-creation marker is present.
**Then** writing the **project** link is allowed, exceptionally.

### M11 — Referenced records must be readable

**Given** M1's result and a milestone **X** in another project he cannot read.
**When** *Joel Willis* writes **X** as a task's milestone.
**Then** the write is refused for lack of read access on **X**.

---

## N. The customer portal

### N1 — The portal home counters

**Given** *Joel Willis* who can read two projects and seven tasks that have a project.
**When** he opens `/my`.
**Then** the project counter is 2 and the task counter is 7.
**Given** a person with no read access at all on projects.
**Then** the project counter is 0.

### N2 — A template project redirects

**When** a template project's portal page is opened.
**Then** the response redirects to `/my`.

### N3 — A project with collaborators redirects a qualifying visitor

**Given** *Website Revamp* with at least one collaborator, and *Joel Willis* holding a collaborator
row.
**When** he opens `/my/projects/<identifier>`.
**Then** he is redirected to `/my/projects/<identifier>/project_sharing`.

### N4 — A public visitor with a token

**Given** *Website Revamp* with a security token `abc…` and visibility `portal`.
**When** an unauthenticated visitor opens `/my/projects/<identifier>?access_token=abc…`.
**Then** the page renders; the task listing skips the acting user's read filter entirely and reads
the tasks with elevated rights.
**When** the token is wrong.
**Then** the response redirects to `/my`.

### N5 — Sorting a portal task list by status

**Given** four tasks in states `1_done`, `04_waiting_normal`, `01_in_progress`, `03_approved`.
**When** the listing is sorted by "Status".
**Then** the order is: In Progress, Approved, Done, Waiting — the declaration order of the
selection, applied in memory after the page is fetched.

### N6 — The milestone sort and grouping disappear when milestones do not apply

**Given** a listing in which no task has both the milestone feature on and a milestone set.
**When** the sort key "Milestone" or the grouping "Milestone" is requested.
**Then** the sort falls back to "Newest" and the grouping to "Project".

### N7 — Attachment upload restrictions

**Given** M1's result.
**When** *Joel Willis* uploads a file of a kind other than one of the four image kinds through the
sharing upload address.
**Then** the answer is a refusal carrying the message "Only jpeg, png, bmp and tiff images are
allowed as attachments."

### N8 — The task page content

**Given** H1's task, opened by *Joel Willis* who follows it.
**Then** the page shows: the title with the identifier in parentheses; the stage as a badge; the
state widget; the project as a link to its portal page (because he can read the project); the
milestone when set; the priority; the deadline; the allocated time, but only when it is above
zero; the description when not empty; one tile per attachment carrying a generated access token;
and the message thread opened with the task's token. The side bar lists the assignees with their
address and telephone number, and the customer.

---

## O. Reporting

### O1 — The analysis excludes private tasks

**Given** a private to-do and five project tasks.
**Then** the analysis contains exactly five rows.

### O2 — Zero is mapped to empty in the analysis

**Given** a task never assigned and never rated.
**Then** its analysis row reports empty (not 0) for the last rating value and for all four
working-time figures, so it does not pull the averages towards zero.

### O3 — The burndown grouping requirement

**When** the burndown series is read grouped by stage only.
**Then** it is refused with: "The view must be grouped by date and by Stage - Burndown chart or Is
Closed - Burnup chart".
**When** it is read grouped by day and by closing state.
**Then** it succeeds and produces the burnup series.

### O4 — A burndown series

**Given** one active task created Monday 2 March, moved to **In Development** on 4 March and to
**Delivered** on 17 March, allocated time 6.0 hours, read on 20 March, grouped by day and by
stage.
**Then** the summed counts are: **Backlog** 2 (2 and 3 March), **In Development** 13 (4 to
16 March), **Delivered** 4 (17 to 20 March), each point carrying allocated time 6.0.
**And** read grouped by day and by closing state, the "open" series is 1 on each of the 19 days
and the "closed" series is empty.

### O5 — The burndown filter split

**Given** a burndown read filtered on the project and on the bucket date.
**Then** the project condition is pushed into the task sub-query and the date condition is applied
to the series; only active tasks are considered.

### O6 — The digest indicator

**Given** *Website Revamp* with three tasks in unfolded stages and two in folded stages, plus one
private to-do.
**Then** the "Open Tasks" indicator reports **3** — folded stages and tasks with no project are
excluded.
**When** the indicator is computed for a user without the Project User privilege.
**Then** the computation raises "Do not have access, skip this data for user's digest email".

---

## P. Templates

### P1 — Converting a project into a template

**Given** *Website Revamp*, active, with a customer and dates 1 January to 12 March.
**When** *Mona Ellis* creates a template from it.
**Then** a new project is created with the template flag set, **no customer**, the same dates and
the same name; the message "Template created from Website Revamp." is posted on it; the source
*Website Revamp* is **archived**; a notification with an undo offer is returned.
**When** the undo is taken.
**Then** the template is deleted and *Website Revamp* is reactivated.

### P2 — Instantiating a template

**Given** a template with start 1 January and expiration 12 March (70 days), four tasks of which
two carry the role **Designer** and one carries **Reviewer**; today is 20 May.
**When** *Mona Ellis* instantiates it with the name `Atlas Launch`, no dates, and the mapping
Designer → {*Ravi Kapoor*}, Reviewer → {*Anita Sen*}.
**Then** a project `Atlas Launch` is created with start **20 May** and expiration **29 July**; it
has **no customer**; it has four tasks, none of them a template, each keeping its original title
and stage; the two Designer tasks gain *Ravi Kapoor* as an assignee in addition to any existing
one; the Reviewer task gains *Anita Sen*; every task's role list is then cleared; and the message
"Project created from template *the template's name*." is posted on the new project.

### P3 — Converting a template back

**Given** a template with no product pointing at it.
**When** the "convert back" operation is taken.
**Then** the confirmation message is "This project is currently a template. Would you like to
convert it back into a regular project?"; on confirmation the flag is cleared, every task's role
list is cleared, and the message "Template converted back to regular project." is posted.
**Given** instead a template with at least one product pointing at it (sales-linked package).
**Then** the confirmation message is "Converting this template to a regular project will unlink it
from its associated products.\nAre you sure you want to continue?"; on confirmation the pointer is
cleared on every such product.

### P4 — A private to-do cannot become a template

**When** the "convert to template" operation is taken on a task with no project.
**Then** a danger notification is returned reading "Private tasks cannot be converted into
templates" and nothing is written.

### P5 — Template tasks are excluded from the counters and the listings

**Given** *Website Revamp* with five ordinary tasks and two template tasks.
**Then** the project's task count is **5**; the portal task listings exclude both template tasks
and their descendants; the analysis excludes them.

---

## Q. Stage administration

### Q1 — Deleting a task stage that still holds tasks

**Given** the stage **In Development** with 4 tasks across 2 projects.
**When** *Mona Ellis* asks to delete it.
**Then** a dialogue opens listing both projects and reporting 4 tasks.
**When** she confirms the archive with two projects affected.
**Then** a second confirmation dialogue opens first.
**When** she confirms it.
**Then** the 4 tasks are archived and the stage is archived.
**When** instead she chooses to delete outright.
**Then** the stage rows are deleted; the deletion fails if a task still references the stage.

### Q2 — Unarchiving a stage with archived tasks

**Given** an archived stage with 3 archived tasks.
**When** it is unarchived.
**Then** an "Unarchive Tasks" dialogue is opened rather than returning silently; confirming
reactivates the 3 tasks.

### Q3 — Archiving a stage archives its tasks

**When** a task stage's active flag is written to false.
**Then** every task sitting in it is archived.
**When** a project stage's active flag is written to false.
**Then** every project sitting in it is archived.

### Q4 — A stage used by a task is attached to the project

**Given** task T1 created in *Website Revamp* with the stage **Triage**, which is not on the
project's board.
**Then** after the creation, **Triage** is attached to *Website Revamp*'s stage collection.

### Q5 — Stage entry side effects

**Given** stage **Delivered**, folded, with an electronic mail template.
**When** T1 enters it at 10:00.
**Then** the ending date becomes 10:00, the last stage update becomes 10:00, the "Stage Changed"
subtype is posted, and the template is sent to the customer as an internal note with the light
layout and without keeping the log.
**When** T1 leaves it for **In Development** (not folded).
**Then** the ending date becomes empty.

---

## R. Tags and roles

### R1 — Tag uniqueness

**When** a tag named `feature` is created while one already exists.
**Then** the creation is refused with: "A tag with the same name already exists."
**When** a tag is created **by name** with the text ` Feature `.
**Then** the existing `feature` is returned instead, because the lookup ignores case and
surrounding whitespace.

### R2 — Project-aware tag listing

**Given** *Website Revamp* whose 1000 most recent tasks use the tags `bug` and `ui`, and 40 other
tags exist in the database.
**When** a tag list is requested with the project in the acting context and a limit of 10.
**Then** `bug` and `ui` come **first**, followed by up to 8 further tags from an ordinary search
that excludes them; and the listing preserves that order.

### R3 — Transparent tags

**Given** a tag whose colour is 0.
**Then** it is not shown on the boards.

### R4 — Role dispatching clears the roles

**Given** P2's result.
**Then** no task of the new project carries any role.

---

## S. Notifications and messaging

### S1 — The assignment notification

**Given** task T1 with no assignee.
**When** *Mona Ellis* adds *Ravi Kapoor* and herself as assignees.
**Then** *Ravi Kapoor* receives "You have been assigned to *the task's display name*"; *Mona Ellis*
does **not** (the acting user is excluded); the assignment date is stamped; both are auto-subscribed
as followers.

### S2 — The notification is suppressed during duplication and recurrence

**When** a project or a task is duplicated, or a recurrence produces the next occurrence.
**Then** no assignment notification is sent.

### S3 — The project-transfer notification

**Given** *Brand Refresh* with two followers holding the "Task Created" project subtype, and task
T1 currently in *Website Revamp*.
**When** T1 is moved to *Brand Refresh*.
**Then** those two followers receive a notification whose body is "Task Transferred from Project
*Website Revamp link* to *Brand Refresh link*".
**When** instead T1 had no project.
**Then** the body is "Task Converted from To-Do".

### S4 — Posting requires only read access

**Given** *Nina Okafor*, who can read task T1 but not write it.
**When** she posts a message on T1.
**Then** the post succeeds.

### S5 — The cover image

**Given** task T1 with no cover image.
**When** a message with two attachments, the first of which is an image, is posted on it.
**Then** the first image attachment becomes the cover image.
**When** a second message with an image is posted.
**Then** the cover image is unchanged.

### S6 — The subtitle of a task notification

**Given** T1 in project *Website Revamp*, stage **Backlog**.
**Then** the notification body carries the subtitle "Project: Website Revamp, Stage: Backlog".
**Given** a task with no project but a stage — impossible in practice; **given** a private to-do
with neither.
**Then** no subtitle is added.

### S7 — Mention suggestions inside the shared application

**Given** M1's result, *Website Revamp* with followers A and B on the project and C on task T1.
**When** *Joel Willis* asks for mention suggestions on T1.
**Then** the answer contains only contacts drawn from {A, B, C} that match his search string.
**When** the project fails the sharing check.
**Then** the answer is empty.

---

## T. Multi-company

### T1 — A task inherits its company from the project

**Given** *Website Revamp* in *Horizon Labs*.
**When** a task is created in it.
**Then** its company is *Horizon Labs*.
**When** the task's project is changed to a project in *Northwind*.
**Then** its company becomes *Northwind*, unless the customer constraint refuses the write.

### T2 — Changing a task's company clears an incompatible project

**Given** task T1 in *Website Revamp* (*Horizon Labs*).
**When** the company is changed to *Northwind* in the form.
**Then** the project field is cleared by the form's reaction.

### T3 — The company filter

**Given** a user whose enabled companies are {*Horizon Labs*} only.
**Then** they cannot reach a project, a task, a project stage, a milestone, a project update or an
analysis row belonging to *Northwind*; they **can** reach those with no company.

### T4 — Switching a project's company reassigns its stage

**Given** the project-stages privilege granted, *Website Revamp* in *Horizon Labs* with stage
**Review** (company *Horizon Labs*), and a stage **Intake** (sequence 5, company *Northwind*).
**When** the project's company is written to *Northwind*.
**Then** the project's stage becomes **Intake**, the first in sequence order whose company is
*Northwind* or empty.
**And** the projects in the same write set that already had *Northwind* are written separately,
without the company key, and keep their stage.

---

## U. Users

### U1 — A new internal user receives seven personal stages

**When** an internal user account is created.
**Then** seven task stages owned by that user are created, in their own language, named Inbox
(1), Today (2), This Week (3), This Month (4), Later (5), Done (6, folded), Cancelled (7, folded).
**When** an external (portal) account is created.
**Then** **no** personal stage is created.

### U2 — The welcome to-do

**Given** the personal to-do package installed.
**When** an internal user *Ravi Kapoor* is created.
**Then** one task with no project is created, titled `Welcome Ravi Kapoor!`, assigned to him, with
a rendered body, created as the superuser and with the follower notification suppressed.

### U3 — The notification tray splits tasks and to-dos

**Given** the personal to-do package, *Ravi Kapoor* with: two activities on one project task (one
overdue, one planned) and one activity on a private to-do due today.
**Then** the tray shows two entries. The **Task** entry counts the project task **once**,
classified by its **earliest** activity deadline, which is overdue → overdue 1, total 1. The
**To-Do** entry counts the to-do once, today 1, total 1.

### U4 — The project manager's embedded arrangement is the default

**Given** *Website Revamp* managed by *Mona Ellis*, who has arranged its embedded actions.
**When** *Ravi Kapoor*, who has configured none of them, asks for his arrangement.
**Then** *Mona Ellis*'s configuration rows for the actions he has not configured are copied onto
him and merged into the answer.

---

## V. Edge cases and rounding

### V1 — Counters and the closed figure

**Given** a project with 5 tasks, 2 open, 3 closed.
**Then** task count 5, open 2, closed 3, task completion percentage 0.6, and the task button reads
`3 / 5 (60%)`.
**Given** a project with 0 tasks.
**Then** the completion percentage is 0 and the button reads `0 / 0`.

### V2 — The open counter excludes children of templates

**Given** a project with 3 ordinary open tasks and 1 open task whose parent is a template task.
**Then** the task count is 4 (the child of a template is not itself a template) but the open task
count is **3**.

### V3 — The closed-task percentage rounds half away from zero

**Given** an update with task count 12 and closed task count 7.
**Then** the percentage is round(58.333) = **58**.
**Given** task count 8 and closed task count 3.
**Then** the percentage is round(37.5) = **38**.

### V4 — The rating average display

**Given** an average of exactly 4.0.
**Then** the stat button reads `4 / 5` — no decimal part is shown for an integral average.
**Given** an average of 3.75.
**Then** it reads `3.8 / 5`.

### V5 — The cropped update title

**Given** a title of 60 characters.
**Then** the cropped title equals the title.
**Given** a title of 61 characters.
**Then** the cropped title is the first 57 characters followed by an ellipsis.

### V6 — The derived to-do title

**Given** a private to-do created with no title and a description whose plain text begins with
`**Draft the board pack** for Q2`.
**Then** the title becomes `Draft the board pack for Q2` — the first line, with asterisks removed
and whitespace trimmed.
**Given** a first line of 140 characters.
**Then** the title is its first 97 characters followed by an ellipsis.
**Given** no description at all.
**Then** the title is `Untitled to-do`.

### V7 — The rating request deadline

**Given** a stage with frequency "Twice a Month" and the current moment 1 March 09:00.
**Then** the deadline is 16 March 09:00 (15 days later).
**Given** frequency "Quarterly".
**Then** it is 30 May 09:00 (90 days later).

### V8 — A security token on a project outside the portal range

**Given** *Website Revamp* with visibility `employees`.
**When** a security token is written on it.
**Then** the stored token is **empty**.
**When** the same write targets two projects at once.
**Then** the write is refused by the single-record requirement.

### V9 — Analytic conversion rounding

**Given** an analytic cost bucket of −90.00 US dollars at a rate of 0.90 euro per dollar, and a
project in euro.
**Then** the converted figure is −81.00, rounded to the euro's two decimal places.
**Given** an advance-invoice amount of 1 234.567 in a foreign currency.
**Then** it is converted **without** rounding, unlike every other conversion in the contract.

### V10 — A section with cancelling figures is omitted

**Given** a customer invoice of 500.00 fully offset by a credit note of 500.00, both posted, both
100 % to the project.
**Then** the invoiced figure is 0.00 and the to-invoice figure is 0.00, so **no**
`other_invoice_revenues` entry is produced at all.

### V11 — Multiple analytic keys containing the same account

**Given** an invoice line whose analytic distribution has two keys both containing the project's
account, one at 30 % and one at 20 %.
**Then** the analytic share used is (30 + 20) ÷ 100 = **0.50**.

### V12 — Ordering of the profitability lists

**Given** a contract containing the revenue sections `downpayments` (20), `materials` (7) and
`service_revenues` (6).
**Then** the list is ordered `service_revenues`, `materials`, `downpayments`.
