# Workflows of the Human Resources Core domain

End-to-end operational procedures, step by step, with the role that performs each one, its
preconditions, the records it creates or updates, and its failure conditions.

---

## 0. Roles

| Role | Group held | What they can do in this domain |
|---|---|---|
| Internal user | the base internal group | Read the Public Employee directory; read and write a defined set of fields about **themselves** through their own preferences; read departments, job positions, work locations and employee tags; grant recognition badges. |
| Department manager | no extra group; recognised by being the Manager of a Department | Additionally sees the department card menu, and the department-scoped report filter grants them access to their own department's members and descendants. |
| Human Resources Officer | the officer group | Full create, read, write and delete on Employee, Employee Version, Department, Job Position, Employee Tag, Departure Reason, Contract Type, Resource, Working Schedule and Working Schedule Attendance; sees every private employee field **except** the twelve administrator-only ones. |
| Human Resources Administrator | the administrator group, which implies the officer group | Additionally: wage, contract dates, contract wage, salary structure type, contract type and the derived contract indicators; work locations; salary structure types; employee activity plans; unrestricted visibility of Employee Versions across companies; manual presence declarations. |
| System Administrator | the system group | Read-only access to the private Employee, for support purposes. |

---

## 1. Creating an employee by hand

**Performed by:** Human Resources Officer.

**Preconditions:** the acting company is set; a working schedule exists on the company.

**Steps.**

1. The officer opens the employee form and enters at minimum a name.
2. On saving, the platform partitions the entered values into employee values and version
   values, keeping only those version values the officer is allowed to write.
3. A Resource record is created carrying the name, the time zone (from the chosen working
   schedule when one was given), the company and the working schedule.
4. The employee row is created, pointing at the Resource. Its Active flag mirrors the
   Resource's.
5. An Employee Version is created with the version values. Its Effective Date defaults to
   today; its Human Resources Responsible defaults to the acting user; its Work Address
   defaults to the acting company's own default address; its Salary Structure Type defaults
   to the first structure type of the company's country, or failing that the first with no
   country; its Marital Status defaults to single; its Home-Work Distance Unit defaults to
   kilometres; its Employee Kind defaults to employee.
6. The version's Employee link is written to the new employee, and the remaining delegated
   values are written onto it.
7. A work contact is created for the employee — with elevated rights, so that an officer who
   lacks the contact-creation right still succeeds — carrying the work email, the work
   telephone, the employee name, the image and the company. The employee's Work Contact is
   set to it.
8. When the employee has no image and the acting user may write views, a placeholder avatar
   is generated from the name, stored on the employee and copied onto the work contact.
9. Every discussion channel whose department list includes the new employee's department has
   its automatic member set recomputed, adding the new person.
10. A message is logged on the employee's thread, in bold and with a link, reading
    "**Congratulations!** May I recommend you to setup an onboarding plan?" where "onboarding
    plan" links to the activity-plan scheduling wizard pre-targeted at this employee.
11. The record set is invalidated so that every computed value is recomputed from storage.

**Records created:** one Resource, one Employee, one Employee Version, optionally one
Contact, optionally one image attachment, one thread message.

**Failure conditions:** a duplicate badge identifier; a badge identifier with a disallowed
character or longer than eighteen characters; a personal identification number that is not
all digits; a user already linked to another employee in the same company; any of the
version constraints.

**Special case — a salary simulation.** When the creating context carries the salary
simulation marker, steps 8 to 11 are skipped: no avatar is generated, no channels are
recomputed and no onboarding message is logged.

---

## 2. Creating an employee linked to a user, and the fields that synchronise

This is the central integration of the domain and it can be entered from either side.

### 2.1 Entry point A — the officer sets the User field on an existing employee

**Performed by:** Human Resources Officer.

**Steps.**

1. The officer picks a user in the employee's User field. An on-change fires immediately,
   before saving:
   a. The employee's Work Contact is set to the user's contact.
   b. The employee's User is set.
   c. When the employee has **no** image, the employee's image is set to the user's image.
      When it already has one, the image is left alone.
   d. When the user has a time zone, the employee's time zone is set to it.
   e. When the employee has no name, the name is set to the user's name.
2. On saving, the same synchronisation is applied again server-side, and one further step
   runs: any **other** employee of the same company which has no user of its own and whose
   work contact is that user's contact has its Work Contact cleared. This prevents a stale
   second employee from silently sharing the contact.
3. The unique (User, Company) constraint is checked. A violation reports "A user cannot be
   linked to multiple employees in the same company."
4. If the write also changed the time zone, and the employee's user's company matches the
   employee's company, and the new time zone differs from the user's, the **user's** time
   zone is updated too.
5. The channels whose department list includes the employee's department are recomputed,
   because the employee now has a user and therefore a contact that can be a channel member.

### 2.2 Entry point B — the officer creates a user from the employee

**Performed by:** Human Resources Officer.

**Precondition:** the employee has no user; otherwise the operation refuses with "This
employee already has an user."

**Steps.**

1. A simplified user form opens as a dialogue, pre-filled with: a marker binding the new
   user to this employee, the employee's name, the employee's work telephone as the
   telephone, the employee's work mobile as the mobile number, the employee's work email as
   the login, and the employee's work contact as the contact.
2. On saving the user, the platform sees the binding marker and writes the new user onto the
   designated employee's User field, which runs the synchronisation of
   [section 2.1](#21-entry-point-a--the-officer-sets-the-user-field-on-an-existing-employee).

### 2.3 Entry point C — a user is created with the "create an employee" switch

**Performed by:** anyone allowed to create users.

**Steps.**

1. The user is created normally.
2. Because the creation values carried the create-employee switch, an employee is created
   with: the user's name, the acting company, and the synchronised values — work contact set
   to the user's contact, user set, image set from the user's image, time zone set from the
   user's time zone when it has one.
3. The employee creation then proceeds exactly as in
   [section 1](#1-creating-an-employee-by-hand), including the work contact creation (which
   is skipped, because a work contact was already supplied), the avatar generation and the
   onboarding message.

### 2.4 Entry point D — the user asks for an employee from their own profile

**Performed by:** the user, on themselves.

**Precondition:** the acting company must be one of the user's companies; otherwise the
operation refuses with "You are not allowed to create an employee because the user does not
have access rights for *the company name*".

**Steps.** An employee is created with the user's name, the acting company and the
synchronised values.

### 2.5 The complete synchronisation table

Once the link exists, values flow in both directions. The table below is the complete
contract.

| Value | Direction | When it flows | Notes |
|---|---|---|---|
| Work Contact | user → employee | Whenever the User field is written or changes in the form. | Set to the user's contact. |
| Image | user → employee | Whenever the User field is written or changes, **only if** the employee has no image of its own. Also whenever the user's image is written, applied in two separate passes so that employees with their own image are not overwritten. | |
| Time Zone | user → employee | Whenever the User field is written or changes, **only if** the user has a time zone. Also whenever the user's time zone is written. | |
| Time Zone | employee → user | Whenever the employee's time zone is written, **only if** the employee's company equals the user's company and the values differ. | |
| Name | user → employee | Whenever the user's name is written. Also at link time, when the employee has no name. | |
| Electronic Mail | user → employee | Whenever the user's electronic mail address is written; it lands on the employee's **Work Email**. | |
| Weekday home-working locations | user → employee | Whenever any of the seven are written on the user. | Present only with the home-working capability. |
| Telephone (`phone`) | employee reads the user | Always, as a read-only mirror. | |
| Electronic Mail (`email`) | employee reads the user | Always, as a read-only mirror. | |
| Instant Messaging Status | employee reads the user | Always, as a read-only mirror, possibly prefixed by today's work-location kind. | |
| Is a Portal or Public Account | employee reads the user | Always, as a read-only mirror. | |
| User is Active | employee reads the user | Always, as a read-only mirror. | |
| Avatar at each size | employee falls back to the user | On read, when the employee has no own image at that size. | |
| The twenty-eight delegated employee fields on the user | user reads and writes the employee | On every read and write of the user, resolved against the user's employee **in the acting company**. | Listed in [business rules, section 6](business-rules.md#6-what-a-person-may-change-about-themselves). |
| Work Telephone | employee ↔ work contact | On read, computed from the contact; on write, written back to the contact. | Only when the contact has at most one employee. |
| Work Email | employee ↔ work contact | Same. | Same condition. |
| Work Contact change | employee → bank accounts | On write of the Work Contact, each of the employee's bank accounts is moved to the new contact, and its outgoing-payment permission is revoked if it had one. | |

### 2.6 Worked scenario

An officer creates an employee named "Nadia Farrell" with work email
`nadia.farrell@example.com`, no image, and no user.

- A Resource "Nadia Farrell" is created; the employee is created; a version dated today is
  created; a contact "Nadia Farrell" is created with that electronic mail address and set as
  the work contact; a placeholder avatar is generated and stored on both.

The officer then opens the create-user operation.

- A user form opens with login `nadia.farrell@example.com`, name "Nadia Farrell", contact set
  to the work contact just created.
- On saving, the user is created and written onto the employee's User field.
- The synchronisation sets the employee's Work Contact to the user's contact — which is the
  same contact, since it was pre-filled — leaves the image alone if the user picked one, and
  copies the user's time zone onto the employee if the user has one.

Later the person changes their own time zone in their preferences.

- The user's time zone is written; the propagation set includes the time zone, so the
  employee of the acting company receives it.
- The employee's write then sees a time-zone change, compares the employee's company with the
  user's company, finds them equal and the values already equal, and writes nothing back —
  the loop terminates.

---

## 3. Creating users in bulk from selected employees

**Performed by:** Human Resources Officer.

**Steps.**

1. The officer selects employees and launches the bulk create-users operation. A confirmation
   is raised first, warning that the users will be created with the default template's rights
   and that the subscription cost may increase; the operator must press "Confirm".
2. The operation classifies each selected employee by the ordered rules of
   [business rules, section 14.2](business-rules.md#142-creating-users-in-bulk-from-employees).
3. The users for the accepted class are created in one call, each carrying the employee
   binding marker, the employee's name, work telephone, normalised work email as login, and
   work contact.
4. Because each carries the binding marker, the user creation writes each new user onto its
   employee.
5. A chain of notifications is returned: one per non-empty class, ending with a soft reload
   when at least one user was created.

---

## 4. Archiving and unarchiving an employee

**Performed by:** Human Resources Officer.

### 4.1 Archiving

1. The set of employees that are currently active is noted.
2. The generic archive runs, setting Active to false, which also archives each Resource.
3. If anything was actually archived:
   a. Every Employee whose Manager or Coach points at one of the archived employees has that
      link emptied. The same is done for a configurable list of *user*-valued links, which is
      empty in the core domain and populated by other capabilities.
   b. When exactly **one** employee was archived and the calling context did not carry the
      no-wizard marker, the Departure Registration Wizard opens as a dialogue, pre-targeted
      at that employee.

### 4.2 Unarchiving

1. The generic unarchive runs, setting Active to true and unarchiving the Resource.
2. The three departure fields on the employee — Departure Reason, Additional Information and
   Departure Date — are cleared.

The contract end date written at departure is deliberately **not** cleared; reopening the
contract is a separate, explicit decision.

---

## 5. Creating a new Employee Version

**Performed by:** Human Resources Officer (or Administrator, for the wage and contract
fields).

**Precondition:** a date must be supplied; otherwise the operation fails immediately because
the effective date is required.

**Steps.**

1. Normalise the supplied date to a calendar date.
2. Find the version to copy: the version in force on that date
   ([calculations, section 3.3](calculations.md#33-the-version-in-force-on-an-arbitrary-date)).
   If none is found, fall back to the first version of this employee by identifier order.
3. **Short circuit:** if the version to copy already has exactly that effective date, return
   it unchanged. Creating a version for a date that already has one is a no-operation, not an
   error.
4. Determine the contract period covering that date, giving a pair (period start, period
   end). Let the new version's contract start date be the supplied value or, failing that,
   the period start; and the contract end date be the supplied value or, failing that, the
   period end.
5. **Propagate an end-date-only change.** If the resulting contract start date equals the
   period start but the resulting contract end date differs from the period end, then every
   version of this employee whose contract start date equals the period start is written,
   with elevated rights and with the synchronisation marker set, with the new contract end
   date. This keeps the whole contract period consistent before the new version joins it.
6. Check that the acting user may write the employee, and may write the version being copied.
7. Assemble the copy values:
   a. Start from the fixed set: the effective date, the employee, the contract start date and
      the contract end date; plus the Active flag when one was supplied; plus the working
      schedule when one was supplied.
   b. Let the *new values* be every supplied value that is not already in that fixed set.
   c. Take a full copy of the version being copied, with elevated rights, and drop from it
      every collection-valued field that the new values also mention — so that a caller
      replacing a collection does not end up with the old members plus the new ones.
   d. Overlay the fixed set on top.
8. Create the new Employee Version with elevated rights from the assembled values, then drop
   back to the acting user's rights.
9. Protect, from recomputation, every version field that the new values do **not** mention and
   that is marked as copyable — so that applying the new values does not cause a computed
   field to overwrite a value that was deliberately copied from the predecessor.
10. Within that protection: first write any structured-document fields that were copied but
    not overridden, so that free extra fields are carried over correctly; then write the new
    values.
11. Return the new version.

**Records created:** one Employee Version. **Records updated:** possibly every other version
of the same contract period, when step 5 applied.

**Failure conditions:** the unique effective date index; the contract overlap check; the
start-before-end check; the access checks of step 6.

### 5.1 Worked scenario — a department change mid-contract

An employee has one version:

| Version | Effective Date | Department | Contract Start | Contract End | Wage |
|---|---|---|---|---|---|
| V1 | 1 January 2026 | Sales | 1 January 2026 | empty | 3 500 |

On 1 July 2026 the person moves to Marketing with a wage of 3 800. The officer creates a
version dated 1 July 2026 with Department = Marketing and Wage = 3 800.

- Step 2 finds V1 as the version in force on 1 July 2026.
- Step 3 does not short-circuit, because V1's effective date is 1 January.
- Step 4: the contract period covering 1 July 2026 is (1 January 2026, empty). No contract
  dates were supplied, so the new version takes (1 January 2026, empty).
- Step 5 does not apply: the end date is unchanged.
- Step 7 copies V1 wholesale, then overlays effective date 1 July 2026 and the contract
  dates, then applies Department = Marketing and Wage = 3 800.

Result:

| Version | Effective Date | Department | Contract Start | Contract End | Wage | Effective start | Effective end |
|---|---|---|---|---|---|---|---|
| V1 | 1 January 2026 | Sales | 1 January 2026 | empty | 3 500 | 1 January 2026 | 30 June 2026 |
| V2 | 1 July 2026 | Marketing | 1 January 2026 | empty | 3 800 | 1 July 2026 | empty |

Both versions belong to one contract period. The overlap check passes because the two periods
are identical, which is explicitly allowed. On 1 July the Current Version moves to V2 and the
employee's Resource working schedule is rewritten to V2's schedule — the same schedule here,
so nothing visible changes.

---

## 6. Starting a contract on a date

**Performed by:** Human Resources Administrator.

**Precondition:** the caller has already established that the employee is not under contract
on that date. The guard that establishes it produces, on failure, "The employee is already in
contract on *the date*. Please select a date outside existing contracts".

**Steps.**

1. Normalise the date.
2. Load the employee's contract periods. Among those whose start is **later** than the given
   date, take the earliest; the new contract's end date is the day before it. When there is
   no later period, the new contract is open-ended.
3. If a version of this employee already carries exactly that effective date, write the
   contract start date and the computed contract end date onto it and return it — no new
   version is created.
4. Otherwise create a version dated on that date with those contract dates, following
   [section 5](#5-creating-a-new-employee-version).

### 6.1 Worked scenario

An employee has a version dated 1 January 2027 opening a contract on 1 January 2027. An
administrator starts a contract on 1 June 2026.

- Later periods: (1 January 2027, …). The earliest is 1 January 2027, so the new contract
  ends **31 December 2026**.
- No version is dated 1 June 2026, so a new version is created dated 1 June 2026 with contract
  dates (1 June 2026, 31 December 2026).
- The overlap check passes: the new period ends the day before the existing one begins.

---

## 7. Changing a contract date

**Performed by:** Human Resources Administrator.

Contract dates are shared by every version of a contract period, so writing one is not a
simple field write. The procedure below applies whenever a write touches the contract start
date or the contract end date and the synchronisation marker is **not** already set in the
context.

**Steps.**

1. **Consistency guard.** Group the versions being written by contract start date. If more
   than one distinct contract start date appears, refuse with "Cannot modify multiple versions
   contract dates with different contracts at once."
2. **Single-version employees.** If a contract start date is being set, split off the versions
   whose employee has exactly one version. For those, write the supplied values **plus** an
   effective date equal to the new contract start date, with the synchronisation marker set so
   this procedure does not recurse. Rationale: for an employee with a single version there is
   no history to preserve, so moving the version to the contract start date keeps the two
   dates aligned.
3. If none of the remaining versions has a contract start date at all, apply the write
   directly and stop.
4. Build the *non-date values*: every supplied value except the contract end date, and except
   the contract start date when it has a non-empty value. (A contract start date being
   **cleared** is kept in this set, so that clearing propagates through the normal path.)
5. For each employee among the remaining versions:
   a. Let the *first version* be the first of that employee's versions in the write set.
   b. Determine the target pair: contract start date from the supplied values when present,
      otherwise the first version's; contract end date likewise.
   c. If the first version has a contract start date, find every version of that employee
      belonging to the contract period bounded by the first version's current contract dates,
      take one representative version per period, and write the target pair onto all of them
      with the synchronisation marker set.
   d. Otherwise write the target pair onto the versions being written, with the marker set.
6. Apply the non-date values to the remaining versions.

### 7.1 Worked scenario — closing a contract

An employee has three versions, all in one contract period (1 January 2026, empty):

| Version | Effective Date | Contract Start | Contract End |
|---|---|---|---|
| V1 | 1 January 2026 | 1 January 2026 | empty |
| V2 | 1 April 2026 | 1 January 2026 | empty |
| V3 | 1 August 2026 | 1 January 2026 | empty |

The administrator sets the contract end date to 31 October 2026 on V3.

- Step 1: only one contract start date appears; the guard passes.
- Step 2: the employee has three versions, so it is not a single-version employee.
- Step 4: the non-date values are empty.
- Step 5: the target pair is (1 January 2026, 31 October 2026). V3 has a contract start date,
  so the versions of the period (1 January 2026, empty) are found — V1, V2 and V3 — and all
  three are written with the target pair, with the marker set so no recursion occurs.
- The overlap check runs on all three; their periods are identical, which is allowed.

Result: all three versions carry (1 January 2026, 31 October 2026). The derived effective end
dates become 31 March 2026, 31 July 2026 and 31 October 2026 respectively.

---

## 8. Registering a departure

**Performed by:** Human Resources Officer, for one or several employees.

**Entry points:** archiving exactly one employee opens the wizard automatically; the wizard
can also be launched directly over a selection.

**Steps.**

1. The wizard opens, defaulted as described in
   [entities, section 24.1](entities.md#241-departure-registration-wizard-hrdeparturewizard).
   For a single employee the departure date is that employee's suggested date, which is its
   existing departure date when its current version's effective end date already lies in the
   past, and today otherwise.
2. The operator picks a departure reason (required), optionally writes additional
   information, confirms the departure date, and ticks or unticks the three switches: set the
   contract end date, remove the related user, free the equipment.
3. On confirming:
   a. **Date guard.** For each selected employee's current version, if it has a contract start
      date later than the departure date, refuse with "Departure date can't be earlier than
      the start date of current contract."
   b. **User archiving eligibility.** When "remove the related user" is ticked, group the
      selected employees by user; count, with elevated rights, how many employees each of
      those users has in total. A user is eligible for archiving only when the number of its
      selected employees equals its total number of employees. Eligible users go to one list,
      the rest to another.
   c. **Termination mode.** When the calling context carries the employee-termination marker,
      collect the still-active selected employees for archiving, and collect their eligible
      users for archiving too.
   d. Archive the collected employees with the no-wizard marker set, so that this wizard is
      not reopened by the archive. Archive the collected users.
   e. Write, on **every** selected employee — archived or not — the departure reason, the
      additional information and the departure date. Because those three fields are delegated,
      they land on each employee's current version. The additional information, when non-empty,
      is also posted on each employee's thread as a message reading "Additional Information:
      *the description*".
   f. When "set the contract end date" is ticked, take the current versions that have a
      contract start date and write the departure date as their contract end date — which
      propagates through the whole contract period by
      [section 7](#7-changing-a-contract-date).
   g. When "free the equipment" is ticked, clear the assigned-equipment collection of every
      selected employee, which unassigns each item.
   h. Build the notification chain: the success notice listing archived users, then the danger
      notice listing users that could not be archived because they still have other active
      employees.

**Everything a departure archives or cancels, in one list:**

| Effect | Condition |
|---|---|
| The employee is archived, and with it its Resource. | Termination mode; employee currently active. |
| The Manager and Coach links pointing at the departing employee are emptied on every other employee. | Always, as part of archiving. |
| The Departure Reason, Additional Information and Departure Date are written on the current version. | Always. |
| The additional information is posted on the employee's thread. | When it is non-empty. |
| The contract end date of the whole current contract period becomes the departure date. | When "set the contract end date" is ticked and a contract start date exists. |
| Every equipment item assigned to the employee is unassigned. | When "free the equipment" is ticked. |
| The linked user is archived. | When "remove the related user" is ticked, termination mode is on, and every employee of that user is in the selection. |
| Future working-schedule leaves are **not** touched by this workflow. | Always — moving or cancelling them belongs to the time-off domain. |

---

## 9. Changing a department manager

**Performed by:** Human Resources Officer.

**Steps.**

1. The officer writes a new Manager on the department.
2. **Before** the value is stored, the platform collects every employee that: belongs to this
   department, currently reports to the department's **outgoing** manager, and is not the
   incoming manager itself.
3. Those employees' Manager is set to the incoming manager.
4. The department's Manager is then written.

Employees of the department who report to somebody other than the outgoing department manager
keep their own reporting line.

---

## 10. Applying a contract template

**Performed by:** Human Resources Officer.

Two entry points exist and they behave slightly differently.

### 10.1 From the employee form, by picking a template

1. The officer sets the Contract Template field on the employee (which lands on the current
   version).
2. An on-change fires: for every field of the template that is in the **whitelist** and is not
   itself a derived mirror, the template's value is copied onto the employee.
3. The whitelist is: Job Position, Department, Contract Type, Salary Structure Type, Wage,
   Working Hours, Human Resources Responsible.

### 10.2 Through the Contract Template Wizard

1. The officer opens the wizard from the employee and picks a template.
2. The wizard takes a full copy of the template's values, keeps only the whitelisted,
   non-derived ones, and writes them onto the **employee** — which lands on the current
   version.
3. The wizard then writes the Contract Template field on the employee's version, recording
   which template was applied.

### 10.3 At version creation

When an Employee Version is created with a Contract Template already set, the template's
whitelisted values are merged into the creation values **with the explicitly supplied values
winning** — so an officer who both picks a template and overrides the wage gets their wage,
not the template's.

The company used to resolve the whitelist is the template's company, or the acting company
when the template has none.

---

## 11. Recording and progressing a skill

**Performed by:** Human Resources Officer, or the employee themselves where the interface
allows it.

### 11.1 Recording a new skill

1. The user adds a row to the employee's Current Skills collection, choosing a skill type.
2. The skill defaults to the first skill of that type; the level defaults to the type's default
   level, or its first level; the validity start defaults to today; the validity stop is empty.
3. On saving, the create instruction is rewritten by
   [calculations, section 15.3](calculations.md#153-transforming-creates): any still-valid
   assertion of the same employee and skill is expired first, then the new assertion is
   created.

### 11.2 Progressing a level

1. The user changes the level on an existing row of the Current Skills collection.
2. On saving, the write instruction is rewritten by
   [calculations, section 15.5](calculations.md#155-transforming-writes): because the level is
   one of the four identifying fields, the existing record is stopped at yesterday and a new
   record is created with the new level, a validity start of today and an empty validity stop.
3. The Current Skills collection now shows only the new record; the full Skills collection
   shows both. The history reports pick both up.

The worked example is in
[state machines, section 6.4](state-machines.md#64-worked-example-of-a-level-progression).

### 11.3 Removing a skill

1. The user deletes a row from the Current Skills collection.
2. On saving, the delete instruction is rewritten by
   [calculations, section 15.2](calculations.md#152-transforming-a-delete-into-an-expiry): the
   record is stopped at yesterday, unless it was recorded today or yesterday or is already
   stopped, in which case it is genuinely deleted; and unless stopping it would create a
   conflict, in which case it is likewise deleted.

### 11.4 Recording a certification

1. The user adds a row whose skill type is flagged as a certification type, choosing a level
   and — this time meaningfully — a validity start and a validity stop.
2. Several certifications of the same skill and level may coexist provided their windows
   differ. An exact duplicate is silently dropped rather than rejected.
3. When the validity stop is passed, the certification leaves the Current Skills collection —
   except that if no other assertion of that skill is still valid, the most recently expired
   one is shown, so the lapse is visible rather than the skill simply vanishing.

### 11.5 Certification renewal reminders

The scheduled job of
[calculations, section 23](calculations.md#23-certification-reminder-job) raises a file-upload
activity for every employee whose job position expects a certification that the employee
either lacks or holds with an expiry within three months, assigning it to the employee's own
user, or their manager's user, or the job position's recruiter, in that order.

---

## 12. Setting a home-working location for one weekday

**Performed by:** the person themselves, or a Human Resources Officer on their behalf.

**Entry point:** clicking a day in the home-working planner opens the Home-working Location
Wizard, pre-filled with that date and that employee.

**Steps.**

1. The user picks a location and decides between "only this date" (Recurring off) and "every
   week on this weekday" (Recurring on).
2. On confirming, with no date the operation does nothing at all.
3. Resolve the employee: the wizard's employee, or the context's default employee, or the
   acting user's own employee.
4. Look up an existing exception for that (employee, date).
5. Compute the weekday index of the date, with Monday as 0, and from it the matching weekday
   location field.
6. Branch:

| Branch | Condition | Effect |
|---|---|---|
| Recurring | Recurring is on | Delete the exception for that date if one exists, then write the chosen location onto the weekday field **through the employee's user**, with elevated rights on the user lookup. |
| Redundant | Recurring is off and the chosen location equals the weekday default | Delete the exception if one exists. Nothing else. |
| Update | Recurring is off, the chosen location differs from the weekday default, and an exception already exists | Rewrite that exception's date, employee and location. |
| Create | Recurring is off, the chosen location differs from the weekday default, and no exception exists | Create an exception with that date, employee and location. |

### 12.1 Worked scenario — a home-working exception on one weekday

Nadia's weekly plan is: Monday Office, Tuesday Office, Wednesday Home, Thursday Office,
Friday Home, Saturday and Sunday empty. Today is Monday 14 September 2026.

**Case one — she works from home on Thursday 17 September only.**

- Weekday index of 17 September 2026 (a Thursday) is 3, so the field is the Thursday location,
  whose current value is Office.
- Recurring is off; the chosen location (Home) differs from Office; no exception exists for
  that date; so an exception is created: (employee Nadia, date 17 September 2026, location
  Home).
- Her Thursday default stays Office. On 17 September her Work Location Name reads "Home", her
  Work Location Kind reads `home`, her presence icon becomes `presence_home` and her
  instant-messaging status is prefixed, becoming for example `home_online`.
- On 24 September, the following Thursday, she is back at Office, because the exception was
  bound to one date only.

**Case two — she then decides every Thursday should be at home.**

- She reopens the wizard on 17 September with Recurring on.
- The exception for 17 September is deleted **first**, then the Thursday location field is
  written to Home through her user.
- Her plan becomes: Monday Office, Tuesday Office, Wednesday Home, Thursday Home, Friday Home.
  Both 17 and 24 September now read Home, and no exception record exists.

**Case three — she comes to the office on Thursday 1 October.**

- The Thursday default is now Home; the chosen location is Office, which differs; no exception
  exists; so an exception (Nadia, 1 October 2026, Office) is created.

**Case four — she cancels that plan and sets 1 October back to Home.**

- The chosen location (Home) now equals the Thursday default, so the exception is **deleted**
  rather than rewritten. The database holds no redundant row.

---

## 13. Launching an onboarding or offboarding plan

**Performed by:** Human Resources Officer or Administrator.

**Steps.**

1. From an employee (or a selection of employees), the officer opens the activity scheduling
   wizard.
2. The wizard notices that the target entity is the Employee and therefore enables department
   filtering: the department shown is the employees' common department, or empty when they
   differ.
3. The list of available plans is narrowed: when no department is resolved, only plans with no
   department; otherwise, plans with no department **or** plans of that department.
4. The suggested plan date is derived from the employees' effective start dates by
   [calculations, section 14.3](calculations.md#143-the-suggested-plan-date).
5. On confirming, each template of the plan is turned into an activity whose responsible is
   resolved by [calculations, section 14](calculations.md#14-resolving-the-responsible-of-a-planned-activity).
6. Errors abort the launch; warnings are shown but the launch proceeds with the acting user as
   the substitute responsible.

Two plans ship with the platform and are described in
[configuration, section 6.4](configuration.md#64-shipped-activity-plans).

---

## 14. Configuring and running advanced presence detection

**Performed by:** Human Resources Administrator, in the settings.

**Steps.**

1. The administrator switches on one or both of "Based on number of emails sent" and "Based
   on IP Address", sets the message threshold and the comma-separated list of valid network
   addresses, and saves.
2. Saving the settings **immediately** runs the evidence-gathering job of
   [state machines, section 5.4](state-machines.md#54-the-evidence-gathering-job) for the
   acting company, so the presence board is populated at once rather than at the next
   scheduled run.
3. From then on, every read of an employee's presence state applies the advanced rule.
4. The administrator may override any individual employee with "declare present" or "declare
   absent", may log a presence note on the employee's thread reading "*the employee name* has
   been noted as *the stored presence state* today", may send a text message to the employee's
   work mobile using the shipped absence template, and may open a time-off request pre-filled
   as an unplanned absence for today.

---

## 15. Editing the salary distribution across bank accounts

**Performed by:** Human Resources Officer.

**Steps.**

1. The officer adds or removes bank accounts on the employee. Each change triggers the
   rebalancing of
   [calculations, section 10.2](calculations.md#102-rebalancing-when-the-set-of-accounts-changes).
2. To set explicit shares, the officer opens the Bank Account Allocation Wizard. Its creation
   reads the current distribution and builds one line per account; an account missing from the
   distribution aborts the opening.
3. The officer edits each line's amount, amount kind (percentage or fixed), order and trust
   flag.
4. On saving:
   a. Each line's amount is rounded **down** to two decimal places.
   b. The trust flag is written onto the bank account with elevated rights.
   c. If any line is a percentage line, the percentage total must equal 100 to within four
      decimal places; otherwise the save is refused.
   d. The assembled map is written onto the employee.

---

## 16. Printing a badge

**Performed by:** Human Resources Officer.

The badge report renders, per employee, a card of fixed size showing: the company logo when
one exists, the employee's avatar, the employee's name, the employee's job position name,
and — when a badge identifier exists — that identifier rendered as a bar code. Cards are laid
out side by side and are never split across a page break. The printed file is named
"Badge - *the employee name with any forward slash removed*".

---

## 17. Printing a curriculum vitae

**Performed by:** Human Resources Officer.

**Steps.**

1. The officer selects employees and opens the curriculum vitae wizard.
2. The wizard offers a primary and a secondary colour, each defaulting to the company's
   corresponding brand colour or, failing that, a mid grey; and three switches: show skills,
   show contact information, show other sections. The last two switches are only offered when
   they are applicable — "other sections" only when at least one selected employee has a
   resume line with no type, and "skills" only when at least one selected employee has a
   skill.
3. Confirming opens the printable document at a dedicated address carrying the employee
   identifiers, the two colours and the three switches as parameters.
4. The document groups each employee's resume lines by resume line type name, with lines that
   have no type gathered under the heading "Other" — and omitted entirely when the "show other
   sections" switch is off.
5. The internal career history derived from the versions
   ([calculations, section 19](calculations.md#19-internal-career-history-derived-from-versions))
   is fetched separately and interleaved.

---

## 18. Loading the demonstration scenario

**Performed by:** Human Resources Officer, from the employee list.

**Steps.**

1. If the research and development department of the demonstration data already exists,
   nothing is loaded and the client simply reloads.
2. Otherwise the employee demonstration scenario file is applied in initialisation mode with
   elevated rights.
3. If the skills capability is present — detected by the presence of the resume-line
   collection on the employee — the skills demonstration scenario file is applied as well.
4. The client reloads.

A second, simpler entry point exists that checks for the demonstration employee tag instead
and applies both scenario files unconditionally when it is absent.

---

## 19. Importing employees

**Performed by:** Human Resources Officer.

The Employee offers one import template, labelled "Import Template for Employees" and served
from a fixed static address. Imported rows create employees through the ordinary creation
path of [section 1](#1-creating-an-employee-by-hand), so every default, every constraint and
every side effect described there applies — including the creation of a Resource, a first
Employee Version and a work contact for each imported row.

---

## 20. Moving an employee's future absences when the schedule changes

**Performed by:** Human Resources Officer, as part of changing a working schedule.

1. The officer creates a new version with a different Working Schedule, or writes the schedule
   on the current version.
2. Writing the schedule on the current version also writes the employee's Resource schedule,
   through the version's inverse.
3. The officer then invokes the leave-transfer operation on the **old** schedule, naming the
   new schedule as the target and, optionally, the resources concerned and a cut-off date.
   Every leave of the old schedule, for those resources, starting on or after the cut-off
   (defaulting to the start of today), is moved to the new schedule.
4. Independently of that manual transfer, the automatic rule on Working Schedule Leaves
   assigns a leave to the schedule of the employee's **current version** whenever the leave's
   start falls inside that version's effective period.
