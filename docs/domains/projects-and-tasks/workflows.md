# Operational workflows of the Projects and Tasks domain

Every workflow below is given as numbered steps with the role that performs it, the
preconditions, the records created or updated at each step, and the failure conditions. Roles are
named as follows:

| Role | Meaning |
|---|---|
| **Project administrator** | An internal user holding the "Administrator" privilege of the Project privilege set. May create, modify and delete projects; sees every project and every task attached to a project. |
| **Project user** | An internal user holding the "User" privilege of the Project privilege set. May create, modify and delete tasks; may read projects but not write them. |
| **Internal user** | Any signed-in employee, with or without the Project privileges. |
| **Collaborator** | An external (portal) person registered on a shared project at the "edit" or "edit with limited access" level. |
| **Portal reader** | An external person who follows a project or a task but is not a collaborator. |
| **Public visitor** | An unauthenticated visitor holding a signed access token. |
| **The system** | An automatic action: a scheduled job, an incoming message, a recomputation. |

---

## 1. Creating and configuring a project

**Role:** project administrator.
**Preconditions:** none.

1. The administrator opens the project board and creates a project, giving at minimum a name.
2. The system suppresses automatic follower subscription for the creation.
3. If the task label was supplied empty, the system replaces it by "Tasks".
4. If the administrator holds the "Use stages on project" privilege, the system assigns a project
   stage:
   - when the board was opened on a particular stage column and that stage belongs to a company,
     the project also adopts that company;
   - otherwise the project stage with the lowest sequence whose company is empty or equals the
     project's company is assigned.
5. The manager defaults to the acting user. The visibility defaults to "All internal users and
   invited portal users". The company is derived from the analytic account's company, then from
   the customer's company.
6. The administrator sets the customer, the start and expiration dates, the tags, and switches on
   the features required: task dependencies, milestones, recurring tasks.
7. Each feature switch runs the global group synchronisation of
   [entities.md](entities.md) §1.6, which makes the corresponding fields visible to every internal
   user and shows or hides the "Task Waiting" notification subtypes.
8. The administrator defines the task stages. Creating a stage from inside the project attaches it
   to the project automatically because the default project collection is taken from the calling
   context.
9. The administrator may enable a satisfaction survey on one or more stages: tick "Send a customer
   rating request", pick a rating template, and choose "when reaching this stage" or "on a
   periodic basis" with a frequency.
10. The administrator may open the incoming-address configuration and give the project an
    electronic mail address; the alias is created with the task as its target model and with the
    project pre-set as a default value on every task it creates.

**Records created:** one Project; zero or more Task Stages linked to it; one Alias when an address
is configured; possibly one Analytic Account when the billing surface requires one.

**Failure conditions:**

- the expiration date is before the start date → "The project's start date must be before its end
  date.";
- the chosen project stage belongs to a different company than the project → see
  [state-machines.md](state-machines.md) §4.3;
- the customer belongs to a different company than the project → "The project and the associated
  partner must be linked to the same company."

### 1.1 Creating a project "on the fly"

When a project is created by typing a name into a project picker, the system additionally creates
one task stage named "New" and attaches it to the new project, so that the project's board is not
empty.

---

## 2. Creating a task

### 2.1 By a project user, inside a project

**Role:** project user or project administrator.
**Preconditions:** the user can read the project.

1. The user opens the project's task board and creates a card, giving a title.
2. The system derives the stage: the first unfolded stage of the project, falling back to the
   first folded one when every stage is folded. The derivation runs once per project for a batch.
3. The system derives the company from the project, the customer from the parent task if any and
   otherwise from the project, and the tags from the parent task when a parent is given.
4. If assignees were given, the assignment date is stamped with the current moment.
5. The state is In Progress. A requested default of Waiting is downgraded.
6. Because the task has a stage, the ending date is set when the stage is folded and cleared
   otherwise, and the last-stage-update timestamp is stamped.
7. The system fills in the missing personal stages of every assignee
   ([entities.md](entities.md) §4.4).
8. The system sends a "you have been assigned to …" notification to every assignee other than the
   creator.
9. When the stage used is not yet on the project's board, it is attached to it.
10. When the project's visibility is "Invited internal and portal users" or "All internal users and
    invited portal users", a security token is generated on the task.
11. Every follower of the parent task is subscribed to the new task with the same subtypes; the
    creator is subscribed; carbon-copy addresses matching contacts that have an internal user are
    notified with the "you have been invited to follow" message and subscribed.
12. The "Task Created" subtype is posted with the body "A new task has been created in the
    *project name* project."

### 2.2 By a project user, as a private to-do

**Role:** any internal user.
**Preconditions:** none — the "User" privilege is required to create a task at all.

1. The user creates a task with no project.
2. Because there is no project, no stage is assigned; any stage supplied directly or as a context
   default is forced to empty.
3. Because there is neither a project nor a parent, the acting user is added to the assignees if
   they are not already there.
4. The customer is cleared: a task with no project and no parent may not keep a customer.
5. The creation message reads "A new task has been created and is not part of any project."
6. When the personal to-do package is installed and no title was given, the title is derived from
   the first line of the description, or set to "Untitled to-do".

### 2.3 Using the title shortcuts

Typing a title such as `Improve the configuration screen #feature #v16 @Mitchell !` in the
quick-creation field causes the parser of [entities.md](entities.md) §5.12 to run: the two tag
words create or link the tags "feature" and "v16", the user word links the single matching user,
the single exclamation mark sets the priority to "Medium priority", and the title is reduced to
"Improve the configuration screen".

If `@Mitchell` matches zero users or more than one, the word is left in the title and no assignee
is linked.

---

## 3. Moving a task through the stages

**Role:** project user, project administrator, or a collaborator at "edit" level.
**Preconditions:** the user can write the task.

1. The user drags the card into another column, or picks the stage in the form.
2. The system refuses the write when the task has no project and the write does not also set one:
   "You can only set a personal stage on a private task."
3. The system stamps the last-stage-update timestamp.
4. The system sets the ending date to the current moment when the new stage is folded, and clears
   it otherwise.
5. The generic field-tracking mechanism records a tracked-value entry with the previous and the new
   stage; that entry is the raw material of both the per-stage duration map and the burndown
   series.
6. The "Stage Changed" subtype is posted on the task and its project-level parent subtype
   "Task Stage Changed" reaches the project's followers.
7. When the new stage carries an electronic mail template and the task is not a template, that
   template is rendered and sent to the customer as an internal note, using the light notification
   layout, without keeping the log entry.
8. When the new stage has the rating switch on and its rating status is "when reaching this
   stage", a rating request is sent immediately (see §8).
9. The staleness computation restarts: the task becomes stale when the new stage's "days to rot"
   threshold is non-zero and the last-stage-update timestamp plus that many days is in the past.

### 3.1 Worked example — three stages, five tasks

**Given** a project *Website Revamp* whose task stages are, in sequence order, **Backlog**
(sequence 1, not folded, 0 days to rot), **In Development** (sequence 2, not folded, 7 days to rot)
and **Delivered** (sequence 3, **folded**, 0 days to rot); the company's working schedule is
Monday to Friday, 08:00–12:00 and 13:00–17:00, that is 8 working hours per day; there are no
leaves in the period.

**And** five tasks, all created on Monday 2 March at 09:00, none assigned at creation:

| Task | Title |
|---|---|
| T1 | Wireframes |
| T2 | Copywriting |
| T3 | Front-end build |
| T4 | Content migration |
| T5 | Launch checklist |

All five are created in **Backlog** (the first unfolded stage), with state In Progress, ending date
empty, last stage update Monday 2 March 09:00, assignment date empty.

**When** the following happens:

| Moment | Event |
|---|---|
| Mon 2 March 11:00 | T1 is assigned to one user. |
| Mon 2 March 14:00 | T1 moves to In Development. |
| Tue 3 March 09:00 | T2 and T3 are assigned. |
| Tue 3 March 15:00 | T2 and T3 move to In Development. |
| Wed 4 March 10:00 | T1 moves to Delivered. |
| Fri 6 March 16:00 | T2 moves to Delivered. |
| Mon 9 March 09:00 | T4 is assigned. |
| Mon 16 March 09:00 | (nothing; T3 is still in In Development.) |
| Tue 17 March 11:00 | T3 moves to Delivered. |
| — | T5 is never assigned and never moves. |

**Then** the date metrics are:

| Task | Assignment date | Ending date | Working hours to assign | Working days to assign | Working hours to close | Working days to close |
|---|---|---|---|---|---|---|
| T1 | Mon 2 Mar 11:00 | Wed 4 Mar 10:00 | 2.00 | 0.25 | 17.00 | 2.125 |
| T2 | Tue 3 Mar 09:00 | Fri 6 Mar 16:00 | 8.00 | 1.00 | 38.00 | 4.75 |
| T3 | Tue 3 Mar 09:00 | Tue 17 Mar 11:00 | 8.00 | 1.00 | 90.00 | 11.25 |
| T4 | Mon 9 Mar 09:00 | (empty) | 40.00 | 5.00 | 0.00 | 0.00 |
| T5 | (empty) | (empty) | 0.00 | 0.00 | 0.00 | 0.00 |

The arithmetic behind every one of these numbers is given in
[calculations.md](calculations.md) §2.5.

**And** the ending dates are set **only** by the move into Delivered, because Delivered is the only
folded stage. Moving from Backlog to In Development leaves the ending date empty.

**And** on Tuesday 10 March at 15:00, T3 becomes **stale**: it entered In Development on Tuesday
3 March at 15:00, the stage's threshold is 7 days, and 3 March 15:00 plus 7 days is 10 March 15:00,
which from that instant on is in the past. Its "days rotting" figure on 17 March at 11:00, just
before it moves, is **13** — the whole-day part of the elapsed calendar time from 3 March 15:00 to
17 March 11:00, which is 13 days and 20 hours. T1 and T2 never become stale because they left In
Development within 7 calendar days; T4 and T5 never become stale because Backlog's threshold is
zero. The staleness computation also requires the task not to be closed, which is the case here.

**And** the project counters at the end of the sequence are: task count 5, open task count 5,
closed task count 0 — because **none of the five tasks had its state changed**. Moving a card into
a folded stage does not close a task. If the users had additionally set T1, T2 and T3 to Done, the
counters would read 5, 2 and 3, and the project's task completion percentage would be
1 − 2 ÷ 5 = 0.6.

---

## 4. Blocking a task until another completes

**Role:** project user or project administrator.
**Preconditions:** the project has the task-dependency feature enabled.

1. The user opens task **B** and adds task **A** to its "Blocked by" list.
2. The system checks the dependency graph for cycles. A cycle raises "Two tasks cannot depend on
   each other."
3. The state recomputation runs on B: A is open, therefore B, being open, becomes **Waiting**.
4. The "Task Waiting" subtype is posted on B (it is hidden by default, so it only appears in the
   log when the dependency feature has been switched on at least once, which un-hides it).
5. B's "blocking task count" becomes 1 and its "closed blocking task count" 0. A's "dependent task
   count" becomes 1, because it counts only **open** dependents.
6. While B is Waiting, a user may still force it to Done or Cancelled; but if they then set it
   back to In Progress or Approved while A is still open, the write routine corrects the value
   back to Waiting.
7. When A is set to Done or Cancelled, the recursive recomputation runs on B: no open blocking task
   remains, therefore B returns to **In Progress** and the "Task In Progress" subtype is posted.
   A's dependent task count falls back to 0.
8. Removing the dependency link has the same effect as closing A.
9. Switching the project's dependency feature off moves every Waiting task of the project to
   In Progress and hides the "Task Waiting" subtypes when no project in the database has the
   feature any more.

**Worked example.** Given project *Website Revamp* with the dependency feature on, task
*Front-end build* (T3) in state In Progress and task *Launch checklist* (T5) in state In Progress.
When the user sets T5's "Blocked by" to T3, then T5's state becomes `04_waiting_normal`, T5's
blocking count is 1 and closed blocking count 0, and T3's dependent count is 1. When T3 is set to
`1_done`, then T5's state becomes `01_in_progress`, T5's closed blocking count becomes 1 and T3's
dependent count becomes 0.

**Failure condition.** Setting T3's "Blocked by" to T5 while T5 is already blocked by T3 raises
"Two tasks cannot depend on each other."

---

## 5. Sub-tasks

**Role:** project user or project administrator.

1. The user creates a child under a parent task, or sets the parent on an existing task.
2. The system refuses a parent that is the task itself: "Sorry. You can't set a task as its parent
   task." It refuses a cycle: "Error! You cannot create a recursive hierarchy of tasks."
3. A database constraint refuses a parent on a private task (one with no project): "A private task
   cannot have a parent." A database constraint refuses a parent on a recurring task: "You cannot
   convert this task into a sub-task because it is recurrent."
4. A constraint refuses clearing the project of a task that has sub-tasks: "This task has
   sub-tasks, so it can't be private."
5. The child's project is derived from the parent's project unless the child is explicitly
   displayed in the project in its own right.
6. "Displayed in project" becomes false when the child shares the parent's project — the child is
   then shown nested under its parent rather than as a top-level card.
7. The child's company is derived from the project, falling back to the parent's company; the
   child's customer from the parent's customer, falling back to the project's customer; the tags
   are copied from the parent at creation.
8. The parent's "sub-task allocated time" becomes the sum of its direct children's allocated time.
   The parent's sub-task count and closed sub-task count are recomputed, excluding template
   children of non-template parents.
9. Archiving the parent archives every child that is **not** displayed in the project in its own
   right. Deleting the parent deletes every descendant.
10. Setting a milestone on the parent propagates it down to the children that have no milestone,
    and to the children that shared the parent's previous milestone, in both cases only when the
    child is not closed and belongs to the milestone's project.

---

## 6. Recurring tasks

**Role:** project user or project administrator.
**Preconditions:** the project has the recurring-task feature enabled; the task has no parent (a
database constraint forbids a recurring sub-task).

### 6.1 Defining the recurrence

1. The user ticks "Recurrent" on a task and fills in: repeat every *N*, unit (days, weeks, months,
   years), until (forever, or until a date).
2. When the recurrence fields are supplied together with the switch at **creation**, a Task
   Recurrence record is created first and the task points at it.
3. When they are written on an existing task, the same happens: if the task already has a
   recurrence the values are written on it; otherwise a recurrence is created and linked.
4. Constraints: the interval must be strictly greater than zero — "The interval should be greater
   than 0"; when the type is "until", the end date must not be in the past — "The end date should
   be in the future".
5. Un-ticking "Recurrent" deletes the recurrence record and clears the switch on every task that
   belonged to it.

### 6.2 Producing the next occurrence

**Trigger:** a task whose state is written to a closed value.

1. The write-back routine of the state collects, for every recurrence represented in the write
   set, the occurrence with the **highest identifier**.
2. It keeps only the written tasks that are now closed **and** are that highest-identifier
   occurrence. A recurrence therefore never produces two next occurrences at once, and closing an
   older occurrence retroactively produces nothing.
3. For each kept task, the guard "should create an occurrence" is evaluated:
   - true when the recurrence type is not "until";
   - true when the task has no deadline;
   - otherwise true only when the recurrence has an end date and the task's deadline shifted by
     one recurrence delta, taken as a date, is **less than or equal to** that end date.
4. For each task passing the guard, the creation values are built:
   - start from the ordinary duplication values of the task, computed as if copying inside a
     project and with archived rows included;
   - force the priority to `0` (Low priority);
   - force the stage to the **first stage of the project's stage collection** (the first element of
     the project's ordered stage list), falling back to the task's own stage when the project has
     no stages;
   - recursively build the same values for every child of the task, attached as new children;
   - copy the recurrence link unchanged;
   - shift the deadline by one recurrence delta (an empty deadline stays empty).
5. The new occurrences are created with elevated rights.
6. The dependency graph of the originals is replayed onto the copies: a copy is blocked by, and
   blocks, the copies of the original's neighbours when those neighbours were themselves copied,
   and the originals otherwise.

### 6.3 Worked example — a weekly recurring task, four occurrences

**Given** project *Operations*, recurring tasks enabled, stages in order **To Do**, **Doing**,
**Done** (folded). A task *Weekly backup verification* is created on Monday 2 March with deadline
**Friday 6 March 17:00**, priority "High priority", one assignee, and the recurrence: repeat every
1 week, type "until", end date **Friday 3 April**.

Occurrence 1 is the task itself: identifier 101, deadline 6 March, priority 2, stage To Do.

**When** occurrence 1 is set to Done on 6 March:

1. It is the highest-identifier occurrence of the recurrence, and it is closed.
2. Guard: type is "until", the deadline exists, and 6 March + 1 week = 13 March ≤ 3 April → create.
3. Occurrence 2 is created: identifier 102, deadline **13 March 17:00**, priority **0**, stage
   **To Do** (the first stage of the project), same assignee, same recurrence, title unchanged
   (the copy runs in project-copy mode, so no " (copy)" suffix is appended).

**When** occurrence 2 is set to Done on 13 March:

- guard: 13 March + 1 week = 20 March ≤ 3 April → create occurrence 3, identifier 103, deadline
  **20 March 17:00**, priority 0, stage To Do.

**When** occurrence 3 is set to Done on 20 March:

- guard: 20 March + 1 week = 27 March ≤ 3 April → create occurrence 4, identifier 104, deadline
  **27 March 17:00**, priority 0, stage To Do.

**When** occurrence 4 is set to Done on 27 March:

- guard: 27 March + 1 week = 3 April ≤ 3 April → create occurrence 5, identifier 105, deadline
  **3 April 17:00**.

**When** occurrence 5 is set to Done on 3 April:

- guard: 3 April + 1 week = 10 April > 3 April → **no further occurrence**. The series stops with
  five tasks. Each of them reports "Tasks in Recurrence" = 5.

Had the recurrence type been "forever", every closure would have produced a further occurrence
indefinitely. Had the task had no deadline, every closure would likewise have produced a further
occurrence, since the guard passes immediately when there is no deadline — and the copy would
carry no deadline either.

**Deleting an occurrence.** Deleting occurrence 5 (the highest identifier) deletes the recurrence
record and clears the "Recurrent" switch on occurrences 1 to 4. Deleting occurrence 3 alone
deletes only that task.

---

## 7. Milestones

**Role:** project administrator to create, project user to tick.
**Preconditions:** the project has the milestone feature enabled.

### 7.1 Creating a milestone

1. The administrator opens the project's side panel or milestone list and creates a milestone with
   a name and, usually, a deadline.
2. The project must not be a template.
3. Tasks are attached to the milestone individually, or inherited from a parent task through the
   propagation of [entities.md](entities.md) §5.9.

### 7.2 Reaching a milestone

1. A user closes the last open task attached to the milestone.
2. The milestone's "can be marked as reached" flag becomes true, because the closed count is at
   least one and the open count is zero. The project's "can mark a milestone as reached" indicator
   becomes true.
3. The user ticks the milestone.
4. The reached flag becomes true and the reached date is stamped with today in the acting time
   zone.
5. The project's reached count rises; the progress percentage is recomputed as the integer
   division of reached × 100 by total.
6. The milestone leaves the unreached set, so it can no longer be the "next milestone" and can no
   longer make the project's "milestone deadline exceeded" indicator true.
7. With the sales-linked package installed and the milestone attached to a sales order item whose
   delivered quantity is tracked by milestones, the item's delivered quantity is recomputed as
   the sum of the quantity fractions of its **reached** milestones multiplied by the ordered
   quantity.

### 7.3 Worked example — a milestone reached

**Given** project *Website Revamp* with the milestone feature on, and a milestone **Design sign-off**
with deadline **Friday 13 March** and sequence 10. Three tasks are attached to it: T1 *Wireframes*,
T2 *Copywriting* and a third task T6 *Style guide*. No other milestone exists.

**Initial state on Wednesday 4 March**, with T1 Done and T2, T6 In Progress:

| Figure | Value | Why |
|---|---|---|
| Milestone task count | 3 | three attached tasks |
| Milestone done task count | 1 | T1 only |
| Can be marked as reached | false | one open task remains (two, in fact) |
| Deadline exceeded | false | 13 March is not before 4 March |
| Project milestone count | 1 | |
| Project milestones reached | 0 | |
| Project milestone progress | 0 | 1 × 0 ÷ 1 |
| Project next milestone | Design sign-off | the only unreached one |
| Project "can mark a milestone as reached" | false | |
| Project "milestone deadline exceeded" | false | |
| Project "milestone exceeded" | false | no unreached milestone with a deadline at or before today |

**When** T2 is set to Done on 10 March and T6 is set to Cancelled on 11 March:

| Figure | Value |
|---|---|
| Milestone done task count | 3 (Cancelled counts as closed) |
| Can be marked as reached | **true** |
| Project "can mark a milestone as reached" | **true** |

**When** the deadline passes without the milestone being ticked — on Monday 16 March:

| Figure | Value | Why |
|---|---|---|
| Deadline exceeded | **true** | 13 March < 16 March and not reached |
| Project "milestone exceeded" | **true** | an unreached milestone with a deadline ≤ today |
| Project "milestone deadline exceeded" | **false** | the walk requires the exceeded milestone to have an open task or no closed task; here all three tasks are closed |

**When** the user ticks the milestone on 16 March:

| Figure | Value |
|---|---|
| Reached | true |
| Reached date | 16 March |
| Deadline exceeded | false (the flag requires "not reached") |
| Project milestones reached | 1 |
| Project milestone progress | 100 (1 × 100 ÷ 1) |
| Project next milestone | empty |
| Project "milestone exceeded" | false |
| Project "can mark a milestone as reached" | false |

**And** the next project update generated after 16 March lists the milestone as checked, with the
suffix "(due 13 March 2026 - reached on **16 March 2026**)" where the reached date is rendered in
red because it is later than the deadline.

---

## 8. Collecting a customer rating

### 8.1 Configuring the request

**Role:** project administrator.

1. On a task stage, tick "Send a customer rating request".
2. Choose a rating electronic mail template whose model is the task.
3. Choose the rating status: "when reaching this stage" or "on a periodic basis"; in the latter
   case choose a frequency among daily, weekly, twice a month, once a month, quarterly, yearly.
4. Optionally tick "Automatic Kanban Status" so that the answer drives the task's state.
5. The switch's side effect shows the two "Task Rating" notification subtypes (and hides them again
   when the last stage with the switch on turns it off).

### 8.2 Sending on stage entry

**Trigger:** a task is written with a new stage whose rating switch is on and whose rating status
is "when reaching this stage".

1. For each such task, the rating template of the stage is taken and the task's customer is
   determined: the task's own customer, falling back to the project's customer.
2. The request is skipped when there is no template, when there is no customer, when the customer
   is the acting user's own contact, or when the task is a template.
3. Otherwise a rating request is sent, rendered in the customer's language, with immediate
   delivery forced.
4. Sending the request first obtains an access token: if the task already has an unconsumed rating
   for that customer, its token is reused; otherwise a new rating row is created with the customer,
   the rated operator (the task's single assignee's contact when there is exactly one assignee,
   otherwise empty), the task as the rated record, and a fresh token.
5. The message is posted on the task as an internal note with the light notification layout.

### 8.3 Sending periodically

**Trigger:** the daily scheduled job "Project Stage: Send rating".

1. Select every task stage where the rating switch is on, the rating status is "on a periodic
   basis", and the rating request deadline is at or before the current moment.
2. For each such stage, send the rating request on **every task of every project attached to the
   stage** — note that this is not restricted to the tasks currently sitting in that stage.
3. Recompute the stage's deadline as the current moment plus the number of days of the frequency
   map ([entities.md](entities.md) §3.4).
4. Commit after each stage, so that a failure on one stage does not lose the work done for the
   previous ones.

### 8.4 The customer answers

**Role:** external customer, signed in or not.

1. The message contains three addresses, one per face, of the form
   `/rate/<token>/<value>` with value 1 (unhappy), 5 (neutral) or 10 (happy). Those three
   **address** values are mapped to the three **rating** values 1, 3 and 5 respectively.
2. Opening the address looks the rating up by token. A missing token yields "not found".
3. When the visitor is signed in and their commercial parent differs from the rating's customer's
   commercial parent, an "invalid partner" page is rendered instead of the form, naming the model
   and the record.
4. Otherwise the submission form is rendered in the rating customer's language, pre-selecting the
   chosen face and offering a free-text comment.
5. Submitting posts to `/rate/<token>/submit_feedback` with the value and the comment. The value
   must be 1, 3 or 5; anything else raises "Incorrect rating: should be 1, 3 or 5 (received
   *the value*)".
6. The rating is applied: the value, the comment and the consumption flag are written, the rated-on
   timestamp is stamped, and a chatter message is posted on the task carrying an 18-by-18-pixel
   face image followed by the comment, authored by the rating's customer, with the "Task Rating"
   subtype. When the rating already had a message, that message is edited in place instead.
7. When the task's stage has "Automatic Kanban Status" on, the task's state is written: Approved
   when the value is at least 4, Changes Requested otherwise.
8. A thank-you page is rendered.

### 8.5 Worked example — an external customer rates a task

**Given** project *Website Revamp* with customer *Deco Addict*, task T1 *Wireframes* assigned to
one user *Abigail Peterson*, and the stage **Delivered** configured with "Send a customer rating
request", status "when reaching this stage", the shipped rating template, and "Automatic Kanban
Status" ticked.

**When** a project user moves T1 into Delivered on 4 March at 10:00:

1. T1's ending date is set to 4 March 10:00 and its last stage update to the same moment.
2. Because the stage's rating switch is on and its status is "stage", the system sends the rating
   request: it creates a rating row with customer *Deco Addict*, rated operator *Abigail Peterson*
   (there is exactly one assignee), rated record T1, parent record *Website Revamp*, value 0,
   consumed false, and a fresh token, say `a1b2…`.
3. The message "*Company name*: Satisfaction Survey" is sent to *Deco Addict* with three links:
   `/rate/a1b2…/1`, `/rate/a1b2…/5`, `/rate/a1b2…/10`.

**When** the customer clicks the happy face and submits the comment "Exactly what we wanted":

1. The address value 10 maps to the rating value 5.
2. The rating row becomes: value 5, comment "Exactly what we wanted", consumed true, rated-on
   the submission moment, text grade `top`, image threshold 5.
3. A chatter message is posted on T1 with the happy face and the comment, authored by
   *Deco Addict*.
4. Because the stage has automatic status and 5 ≥ 4, T1's state is written to `03_approved`, and
   the "Task Approved" subtype is posted.

**Then** the aggregates are:

| Figure | Value | Why |
|---|---|---|
| T1 last rating value | 5 | the most recent consumed rating |
| T1 last rating text | Happy | 5 ≥ 4 |
| T1 rating count | 1 | one consumed rating of value ≥ 1 |
| T1 average rating | 5.0 | |
| T1 average rating text | Happy | 5.0 ≥ 3.66 |
| T1 rating satisfaction | 100 | one "great" out of one |
| Project rating count | 1 | within the 30-day window |
| Project average rating | 5.0 | |
| Project rating satisfaction | 100 | |
| Project "show ratings" | true | at least one stage has the switch on |
| Project rating stat button | icon "smiling face", text "Average Rating", number "5 / 5" | 5.0 ≥ 3.66 |

**And** if a second task is later rated 2 by the same customer, the project figures become: count 2,
average (5 + 2) ÷ 2 = 3.5, satisfaction 1 × 100 ÷ 2 = 50, average text `ok` because
2.33 ≤ 3.5 < 3.66, and the stat button's icon becomes the neutral face with the number "3.5 / 5".

**And** 31 days after the first rating's last write, that rating leaves the project's 30-day
window: the project count drops to 1 and the project average becomes 2.0, while the **task**
aggregates are unaffected because the task-level aggregation has no time window.

---

## 9. Creating a task from an incoming message

**Role:** the system, triggered by an inbound electronic mail.
**Preconditions:** the project has an incoming address configured; the address resolves to the
project's alias, whose target model is the task and whose default values include the project.

1. The routing layer matches the recipient address to the project's alias and calls the task
   creation entry point with the message dictionary and the alias's default values (which contain
   the project).
2. The creation context is prepared with **no default assignee**, so that the gateway user does not
   become responsible.
3. If the message has no resolved author but has a sender address, a contact is found or **created**
   from that address, and that contact becomes the author.
4. The default values are assembled:
   - title: the message subject, or "No Subject" when there is none;
   - allocated time: 0;
   - **customer: the author contact**;
   - carbon copy: the sanitised carbon-copy addresses of the message, but only when the alias
     supplies a project (a task with no project keeps an empty carbon-copy list).
5. The alias's own default values are merged on top.
6. The recipients of the message are resolved:
   - every recipient address is sanitised and matched against existing contacts **without
     creating any**;
   - the project's own alias address is removed from the unresolved set, so that the project's
     address never becomes a contact;
   - the contacts that have at least one **internal** user become **assignees** of the task;
   - the contacts that have no user at all, plus the addresses that matched no contact, are
     appended to the carbon-copy list when the alias supplies a project.
7. The task is created through the ordinary creation routine, which stamps the assignment date
   because assignees are present, derives the stage from the project, generates a security token
   when the project's visibility permits external access, and sends the assignment notifications.
8. Every contact resolvable from the recipient and carbon-copy addresses — again without creating
   any — is subscribed as a follower of the task, but only when the task has a project.
9. The message is posted on the new task with the "Task Created" subtype.
10. After the message is posted, if the task still has no description, the message's subtype is the
    creation subtype, the task's customer is the message author, the message type is electronic
    mail and the message has a body, then the task's description is filled from the message body
    with the signature removed: every element matching an identifier of `Signature`, a
    smart-mail signature marker, or a span whose normalised text is exactly `--`, is stripped, and
    the remainder is sanitised.
11. The first image attachment of the message becomes the task's cover image when none is set.
12. Replies to the task's address go through the message-update path, which subscribes any newly
    resolvable recipients and carbon-copy contacts as followers.

The reply address of a task is the **project's** alias, not the task's own, so a conversation stays
on the project's address.

### 9.1 Worked example — an inbound message creating a task with its sender as customer

**Given** project *Support Intake* with the incoming address `support@example-domain`, visibility
"All internal users and invited portal users", three stages **New** (first, unfolded),
**Assigned**, **Closed** (folded). The contact *Joel Willis* exists with the address
`joel.willis@client-domain`; the contact *Abigail Peterson* exists and has an internal user; the
address `procurement@client-domain` matches no contact.

**When** a message arrives:

| Header | Value |
|---|---|
| From | `joel.willis@client-domain` |
| To | `support@example-domain`, `abigail.peterson@example-domain` |
| Carbon copy | `procurement@client-domain` |
| Subject | `Login page throws an error` |

**Then**:

1. The alias resolves to project *Support Intake*; the default values carry that project.
2. The author resolves to the existing contact *Joel Willis*; no contact is created.
3. The defaults become: title "Login page throws an error", allocated time 0, customer
   *Joel Willis*, carbon copy `procurement@client-domain` — the sanitised carbon-copy list.
4. The recipients are resolved: `support@example-domain` is the project's own address and is
   discarded; `abigail.peterson@example-domain` matches *Abigail Peterson*, who has an internal
   user, so she becomes an **assignee**. No address in the recipient list resolves to a
   contact-without-user, so nothing is appended to the carbon copy from that side.
5. The task is created in project *Support Intake*, stage **New**, state In Progress, customer
   *Joel Willis*, assignee *Abigail Peterson*, assignment date stamped with the reception moment,
   carbon copy `procurement@client-domain`, security token generated (the visibility allows portal
   access), last stage update stamped.
6. *Abigail Peterson* receives the "You have been assigned to Login page throws an error"
   notification.
7. *Joel Willis* is subscribed as a follower (he is resolvable from the recipient and carbon-copy
   sets — in fact through the author path he is already the customer; the subscription of
   resolvable recipients runs regardless).
8. Because `procurement@client-domain` resolved to no contact, no invitation is sent for it; had it
   resolved to a contact with an internal user, that person would have received the
   "You have been invited to follow Login page throws an error" message and become a follower.
9. The task's description is empty, the creation subtype was posted, the customer equals the
   message author and the message is of type electronic mail with a body — therefore the
   description is filled from the sanitised message body with the signature stripped.
10. Because the project's visibility is "All internal users and invited portal users" and
    *Joel Willis* is a follower, he can open the task at `/my/tasks/<identifier>`.

**And** when *Joel Willis* replies to `support@example-domain` quoting the task reference, the
message-update path runs instead: the reply is appended to the same task's thread and any newly
resolvable recipient contacts are subscribed.

---

## 10. Private tasks and sharing one with a second assignee

**Role:** any internal user.

1. A private to-do is a task with no project. Its creator is always among its assignees.
2. It has no stage; it is organised only through the creator's personal stages.
3. It may not have a parent and may not have sub-tasks.
4. It may not carry a customer: the customer computation clears it.
5. It is visible to a project user only through the private-task record rule, which additionally
   requires the user to be among its assignees — see [business-rules.md](business-rules.md) §3.4.

### 10.1 Worked example — a private task shared with a second assignee

**Given** internal user **Anita** (project "User" privilege) and internal user **Bram** (project
"User" privilege). Anita creates a private to-do *Prepare the quarterly board pack* with no
project.

**Then** at creation:

| Figure | Value |
|---|---|
| Project | empty |
| Stage | empty |
| State | `01_in_progress` |
| Assignees | Anita (added automatically because there is neither project nor parent) |
| Customer | empty |
| Assignment date | the creation moment |
| Personal stage assignment | one row (task, Anita, Anita's "Inbox") |
| Followers | Anita's contact |

**And** Bram cannot see the task: the private-task record rule requires him to be an assignee, and
he is not; the general task rule requires the task either to belong to a project readable by him —
it belongs to none — or for him to be a follower or an assignee, and he is neither.

**When** Anita adds Bram to the assignees:

1. The write stamps nothing new on the assignment date, because the task already had an assignee.
2. The system fills the missing personal stages: a second row is created in the shared table,
   (task, Bram, Bram's first personal stage). If Bram had no personal stage at all, the seven
   default ones would be created for him first, in his own language, and the first of them used.
3. Bram's contact is automatically subscribed as a follower of the task by the auto-subscription
   rule for assignees.
4. Bram receives the notification "You have been assigned to Prepare the quarterly board pack",
   rendered from the assignment template and sent with automatic deletion.
5. Bram can now read **and write** the task: the private-task rule matches because he is an
   assignee.

**And** each of them sees the task in their own personal column: Anita in her "Inbox", Bram in his
own first personal stage, independently. Anita moving the task to her "Today" column does not
affect Bram's column, because the personal stage is stored per (task, user) pair.

**When** Anita later removes herself from the assignees, leaving only Bram:

- Anita's personal stage assignment row disappears together with her assignee link (they are the
  same row);
- the assignment date is untouched because the task still has an assignee;
- Anita remains a **follower**, so she keeps read access through the general task rule, but the
  private-task rule no longer matches for her, and a project user needs both rules to be
  satisfiable — see [business-rules.md](business-rules.md) §3.4 for the exact consequence.

**When** the last assignee is removed:

- the assignment date is cleared to empty;
- no personal stage assignment row remains.

---

## 11. Publishing a project update

**Role:** project user or project administrator (write access on updates is granted to the project
"User" privilege).

1. The user opens the project's Updates view and creates an update, or picks a status directly on
   the project.
2. The default values are assembled:
   - the project is the active record;
   - the progress defaults to the previous update's progress;
   - the status defaults to the project's current last-update status, or "On Track" when that is
     "Set Status";
   - **the description defaults to the generated body**.
3. The generated body is assembled from four blocks, in this order:
   - a **Summary** block: a heading "Summary" followed by the prompt "How's this project going?";
   - an **Activities** block: shown only when the milestone section has content, consisting of the
     heading "Activities";
   - a **Profitability** block: shown only when the profitability values were produced and the
     project has an analytic account, consisting of a "Profitability" heading, a revenues table, a
     costs table and a total row — see [interfaces.md](interfaces.md) §"Project update body" and
     [calculations.md](calculations.md) §6 for every figure;
   - a **Milestones** block: shown only when there is at least one milestone to list, one recently
     re-dated, or one recently created — see [calculations.md](calculations.md)
     §"Milestone section of a project update".
4. The user edits the summary, saves, and the update is created.
5. On creation, the project's last-update link is set to this update with elevated rights, and the
   update's task count and closed task count are written from the project's counters **at that
   instant** (task count, and task count minus open task count).
6. The project's last-update status recomputes from the new update; the project's colour follows.
7. The update's own discussion thread notifies the project's followers through the "Update Created"
   subtype and its project-level parent.

### 11.1 Worked example — a project update with its generated summary

**Given** project *Website Revamp*, billable, with an analytic account, currency euro, task count
5, open task count 2, closed task count 3. Its milestone feature is on and it has two milestones:
**Design sign-off** (deadline 13 March, reached on 16 March) and **Go live** (deadline 30 April,
not reached, no closed task yet). A previous update exists, created on 1 March with progress 20
and status "On Track". Since that update, the deadline of **Go live** was moved from 20 April to
30 April, and a third milestone **Content freeze** (deadline 10 April) was created.

Profitability at the moment of writing, in euro:

| Section | Invoiced | To invoice |
|---|---|---|
| Other Services (`service_revenues`) | 12 000.00 | 3 000.00 |
| Customer Invoices (`other_invoice_revenues`) | 800.00 | 0.00 |
| **Revenues total** | **12 800.00** | **3 000.00** |

| Section | Billed | To bill |
|---|---|---|
| Vendor Bills (`other_purchase_costs`) | −4 500.00 | −500.00 |
| Other Costs (`other_costs_aal`) | −1 200.00 | 0.00 |
| **Costs total** | **−5 700.00** | **−500.00** |

**When** the user opens the update form on 20 March:

1. Progress defaults to 20, status defaults to "On Track", title is typed as "Sprint 3 review".
2. The generated description contains:
   - **Summary** — the heading and the prompt.
   - **Activities** — the heading, because the milestone section has content.
   - **Profitability** — the heading, then a revenues table with one row per section:
     | Revenues | Expected | To Invoice | Invoiced |
     |---|---|---|---|
     | Other Services | €15,000.00 | €3,000.00 | €12,000.00 |
     | Customer Invoices | €800.00 | €0.00 | €800.00 |
     | **Total Revenues** | **€15,800.00** | **€3,000.00** | **€12,800.00** |

     then a costs table:
     | Costs | Expected | To Bill | Billed |
     |---|---|---|---|
     | Vendor Bills | −€5,000.00 | −€500.00 | −€4,500.00 |
     | Other Costs | −€1,200.00 | €0.00 | −€1,200.00 |
     | **Total Costs** | **−€6,200.00** | **−€500.00** | **−€5,700.00** |

     then the total row:
     | Total | €9,600.00 / 61% | €2,500.00 / 83% | €7,100.00 / 55% |
     |---|---|---|---|

     The "Expected" column of a row is always *invoiced + to invoice* (or *billed + to bill*). The
     total row's three figures are the **margin** (revenues total + costs total =
     15 800 + 3 000 − 6 200 = … see below), the **to bill plus to invoice** figure and the
     **billed plus invoiced** figure, each followed by its percentage. The exact definitions and
     this example's arithmetic are in [calculations.md](calculations.md) §6.8.
   - **Milestones** — the heading, then:
     - a checklist of every milestone whose deadline is empty or earlier than today plus one year:
       "Design sign-off" **checked**, with the suffix "(due 13 March 2026 - reached on
       16 March 2026)" in red because the reached date is after the deadline; "Content freeze"
       unchecked with "(due 10 April 2026)" in grey; "Go live" unchecked with
       "(due 30 April 2026)" in grey;
     - the sentence "Since 1 March 2026 (last project update), the deadline for the following
       milestone has been updated:" followed by "Go live (20 April 2026 ⇒ 30 April 2026)";
     - the sentence "The following milestone has been added:" followed by "Content freeze
       (due 10 April 2026)".
3. The user saves. The update is created with task count **5** and closed task count **5 − 2 = 3**,
   captured at that instant. Its closed-task percentage is round(3 × 100 ÷ 5) = **60**.
4. The project's last-update link points at it; the project's last-update status becomes "On Track"
   and its colour 20.

---

## 12. Sharing a project with an external person

**Role:** project administrator.
**Preconditions:** the project's visibility is "Invited internal and portal users" or "All internal
users and invited portal users". When it is not, the sharing check returns false and the embedded
application is refused.

1. The administrator opens "Share" on the project. The dialogue is pre-filled: one line per
   existing collaborator at the level matching their limited flag, plus one line at level "Read"
   for every external follower who is not a collaborator, sorted by display name.
2. The administrator adds people, changes levels, or removes lines, and decides per line whether to
   send an invitation.
3. **Confirming creates the wizard record, and creating it applies the changes immediately**, in
   this order:
   1. Compute the collaborators to remove: every existing collaborator whose contact is not in the
      line list.
   2. Compute the followers to remove: every external follower of the project whose contact is not
      in the line list.
   3. For each line:
      - at level "edit" or "edit with limited access": when no collaborator row exists, queue the
        contact for creation at the matching limited flag; when one exists with a different
        limited flag, queue an update of that flag;
      - at level "read": when a collaborator row exists, queue it for removal;
      - in every case, when the contact does not yet follow the project, queue it for subscription.
   4. Contacts queued for creation at the **unlimited** level are filtered to those that are not
      already collaborators and that are flagged shareable; collaborator rows are created for them
      and **every task of the project subscribes them as followers**.
   5. Contacts queued for creation at the **limited** level are filtered the same way; collaborator
      rows are created with the limited flag true; tasks are **not** mass-subscribed.
   6. The queued removals and updates are applied.
   7. The queued subscriptions run through the project's follower-adding routine: the contacts are
      subscribed to the project, and, for every task whose customer is one of them or a child of
      one of them, subscribed to that task as well.
   8. The followers to remove are unsubscribed from the project, which also deletes any remaining
      collaborator rows those contacts had on the project and unsubscribes them from every task.
4. **Sending the invitations** then runs:
   - if no line requests an invitation for a person without a user account, or the sign-up scope is
     not "on invitation", the invitations are sent straight away;
   - otherwise a confirmation dialogue is shown first, because portal accounts will have to be
     created.
   - For the lines at "edit" or "edit with limited access" that request an invitation: the contacts
     that already have a user receive the public link; the others receive a sign-up link.
   - For the lines at "read" that request an invitation: the generic portal-sharing message is sent
     to them.
   - A success notification "Project shared with your collaborators." is returned.
5. The first collaborator row created anywhere in the database activates the two dormant portal
   security records; the last one deleted deactivates them.

### 12.1 What each level can actually do

| Capability | Portal reader (follower only) | Edit with limited access | Edit |
|---|---|---|---|
| Open `/my/projects/<identifier>` | yes | redirected to the embedded application | redirected to the embedded application |
| Open the embedded project application | no | yes | yes |
| See a task they do **not** follow | no | no | **yes** |
| Modify a task they follow | no | yes | yes |
| Create a task in the project | no | yes | yes |
| Delete a task | no | no | no |
| Toggle their own followership on a task | no | **no** (the follow button is hidden) | yes |
| Read the project's milestones | only through the portal page | yes | yes |

---

## 13. The customer portal

**Role:** portal reader, or public visitor holding a token.

1. `/my` shows a "Projects" counter and a "Tasks" counter. The project counter is the number of
   projects the person may read; the task counter is the number of tasks with a project that the
   person may read. Each is zero when the person has no read access at all to the entity.
2. `/my/projects` lists the projects the person may read, excluding templates, sortable by "Newest"
   (creation date descending) or "Name", with paging.
3. `/my/projects/<identifier>` shows one project. If the project has at least one collaborator and
   the visitor passes the sharing check, they are redirected to the embedded application instead.
   Otherwise the project's tasks are listed, grouped by stage by default.
4. `/my/tasks` lists every task with a project that the person may read, excluding tasks with a
   template ancestor, with a filter per project, a search, a sort and a grouping.
5. `/my/tasks/<identifier>` shows one task, with its description, its discussion thread and its
   attachments, each attachment given a generated access token so it can be downloaded.
6. `/my/projects/<identifier>/task/<identifier>` shows the same page in the context of the project,
   with previous/next navigation drawn from the visitor's browsing history stored in the session.
7. `/my/projects/<identifier>/task/<identifier>/subtasks` and `…/recurrent_tasks` list the
   descendants and the occurrences of the series respectively.

The listing routine applies, in order: the base filter of the page; the exclusion of tasks with a
template ancestor; the read record rule of the acting user, **unless** the page was reached with a
project access token by a public visitor, in which case the rule is skipped and the records are
read with elevated rights; the date range; the search term.

---

## 14. Creating a project from a template

**Role:** project administrator.

1. The administrator opens the template list and picks "Create a project from template *name*".
2. The dialogue collects the new project's name, and — when the template has both a start date and
   an expiration date — the new start and expiration dates, plus the incoming address to give the
   new project; and one line per role appearing on the template's tasks, each with the users who
   will take it.
3. On confirmation, exactly five values are passed to the instantiation: the name, the start date,
   the expiration date, the alias name and the alias domain.
4. The instantiation runs:
   1. When the template has both dates and the caller supplied neither, the start date defaults to
      **today** and the expiration date to today plus the template's own duration
      (template expiration − template start).
   2. Context defaults whose key is on the project template whitelist are merged in. The whitelist
      is: the milestone feature flag; with the sales-linked package, also the billable flag and the
      "opened from a sales order" marker; with the time-recording package, also the timesheet flag.
   3. The template is duplicated in "instantiate from template" mode. The duplication keeps the
      source's name unless one was given, keeps the source's project stage when the acting user
      holds the project-stages privilege, and drops every field on the project template blacklist
      — currently the customer.
   4. Every task is duplicated: children recursively, only the active ones, with the template flag
      forced off (because this is not a project-template-to-template copy), with the customer
      dropped, and without the " (copy)" suffix.
   5. The message "Project created from template *the template's name*." is posted on the new
      project.
5. Role dispatching then runs: for every new task and every mapping line that has at least one
   user, if the line's role is among the task's roles, those users are **added** to the task's
   assignees (existing assignees are kept).
6. Finally every task's role list is cleared, so the new project carries no roles.
7. The new project's task board is opened.

---

## 15. Converting a project into a template and back

1. **Project → template.** The project is duplicated with the template flag set, the customer
   cleared and the source's dates kept. The duplicate is put into template mode. The message
   "Template created from *the source project's name*." is posted on it. If the source project was
   active it is **archived**, and that fact is recorded so that an undo can reactivate it. A
   notification offering "undo" is returned; undoing deletes the template and, when recorded,
   reactivates the source.
2. **Template → project.** A confirmation dialogue is shown, carrying either the accumulated
   warnings plus "Are you sure you want to continue?" or the default question "This project is
   currently a template. Would you like to convert it back into a regular project?". On
   confirmation the template flag is cleared, every task's role list is cleared, and the message
   "Template converted back to regular project." is posted. Callbacks recorded with the
   confirmation run — with the sales-linked package, the pointer from every product to this
   template is cleared.

---

## 16. Archiving and deleting

| Operation | Cascade |
|---|---|
| Archive a project | Every task of the project, archived ones included, is written with the same flag. |
| Unarchive a project | Every task is reactivated. |
| Archive a task | Every child that is not displayed in the project in its own right is archived recursively. |
| Archive a task stage | Every task in the stage is archived. |
| Archive a project stage | Every project in the stage is archived. |
| Delete a project | The per-user embedded-action configuration rows are deleted; every task (archived included) is deleted; the project row is deleted; then every analytic account that was linked to a deleted project and has no analytic line is deleted. The three feature-group synchronisations run beforehand. |
| Delete a task | Every descendant is added to the deletion set; for each recurrence whose highest-identifier occurrence is being deleted, the recurrence record is deleted and the remaining occurrences lose the "Recurrent" flag; the rows are deleted, cascading to personal stage assignments and ratings. |
| Delete a task stage | Routed through the deletion dialogue; refused while a task still references it. |
| Delete a personal stage | Runs the reassignment of [entities.md](entities.md) §3.7; refused when it would leave an active internal user with no personal stage. |
| Delete a milestone | Deleted with its project (the link cascades). |
| Delete a project update | The project's last-update link falls back to the most recent remaining update. |
| Delete an analytic account | Refused while any task exists whose project points at it. |

---

## 17. Duplicating a project

See [entities.md](entities.md) §1.9 for the ordered algorithm. The user-visible result is:

- a new project named `<source name> (copy)`;
- the same followers, with the same subtype selections;
- the same milestones (only when the source had the milestone feature on), remapped;
- the same root tasks and their whole sub-tree, all reset to state In Progress, all keeping their
  original stage and their original name, all pointing at the new project;
- the dependency graph between the copied tasks rebuilt among the copies;
- every copied task's assignee list reduced to the source's **active** assignees;
- the recurrence records duplicated, so the copies form a new, independent series;
- the shared embedded actions and every user's arrangement of them, remapped.
