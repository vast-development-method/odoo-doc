# Acceptance criteria for the Human Resources Core domain

Numbered Given / When / Then scenarios with concrete values. A reimplementation is
behaviourally equivalent when every scenario below passes.

Unless a scenario says otherwise:

- the company is **Northwind SA**, its currency is the euro with a rounding step of 0.01, its
  country is Belgium, and its default working schedule is **Standard 40 Hours** (Monday to
  Friday, 08:00–12:00 and 13:00–17:00, time zone Europe/Brussels);
- the acting user is **Olga Reyes**, a Human Resources Administrator;
- "today" is **Friday 11 September 2026** in the Europe/Brussels time zone.

---

## Part A — Creating an employee and linking a user

### A1. Creating a minimal employee creates a resource, a version and a work contact

**Given** no employee named "Nadia Farrell" exists.

**When** Olga creates an employee with Employee Name "Nadia Farrell" and Work Email
`nadia.farrell@northwind.example` and saves.

**Then**:

1. A Resource named "Nadia Farrell" exists, belonging to Northwind SA, with working schedule
   Standard 40 Hours and time zone Europe/Brussels.
2. The Employee exists, Active is true, Company is Northwind SA, and its Resource is the one
   above.
3. Exactly **one** Employee Version exists for this employee, with Effective Date
   11 September 2026, Human Resources Responsible Olga Reyes, Marital Status `single`,
   Employee Kind `employee`, Home-Work Distance Unit `kilometers`, Work Address the default
   address of Northwind SA's own contact, and Salary Structure Type the first Belgian salary
   structure type.
4. The employee's Current Version is that version, and the Version pointer resolves to it.
5. A Contact named "Nadia Farrell" exists with electronic mail address
   `nadia.farrell@northwind.example`, company Northwind SA, and the employee's Work Contact
   points at it.
6. The employee's image and the contact's image both carry a generated placeholder avatar.
7. The employee's thread carries one logged message reading "**Congratulations!** May I
   recommend you to setup an onboarding plan?".
8. Number of Versions reads 1.

### A2. Creating an employee linked to a user synchronises five values

**Given** a user **Nadia Farrell** exists with login `nadia.farrell@northwind.example`,
contact "Nadia Farrell (user)", an uploaded photograph, and time zone `Europe/Lisbon`;
**and** the employee of scenario A1 exists with no user and with the generated placeholder
image.

**When** Olga sets the employee's User to that user and saves.

**Then**:

1. The employee's Work Contact becomes "Nadia Farrell (user)".
2. The employee's image is **not** replaced, because the employee already had one (the
   generated placeholder counts as an image).
3. The employee's Time Zone becomes `Europe/Lisbon`.
4. The employee's User is the user.
5. The employee's Telephone, Electronic Mail, Instant Messaging Status, Is a Portal or Public
   Account and User is Active all read through to the user.
6. No second employee of Northwind SA is linked to that user.

### A3. The image is copied when the employee has none

**Given** the same user as A2, **and** an employee "Tom Bauer" created through an import with
no image at all and no work contact avatar.

**When** Olga sets the employee's User to a user holding a photograph.

**Then** the employee's Image becomes the user's photograph.

### A4. Writing the employee's time zone updates the user's

**Given** the employee of A2 with User Nadia Farrell, employee company Northwind SA, user
company Northwind SA, both time zones `Europe/Lisbon`.

**When** Olga writes the employee's Time Zone as `Europe/Brussels`.

**Then** the user's time zone also becomes `Europe/Brussels`.

**And when** the same write is performed on an employee whose company is **Southwind SA**
while the user's company is Northwind SA, **then** the user's time zone is left unchanged.

### A5. A user cannot be linked to two employees of the same company

**Given** the employee of A2 linked to user Nadia Farrell in Northwind SA, **and** a second
employee "N. Farrell" also in Northwind SA.

**When** Olga sets the second employee's User to Nadia Farrell.

**Then** the save is refused with "A user cannot be linked to multiple employees in the same
company."

**And given** instead that the second employee belongs to **Southwind SA**, **then** the save
succeeds and both employees exist, one per company.

### A6. Linking a user detaches a stale sharing employee

**Given** employee "Ghost Record" of Northwind SA with **no** user and Work Contact "Nadia
Farrell (user)"; **and** employee "Nadia Farrell" of Northwind SA with no user.

**When** Olga sets Nadia Farrell's User to the user whose contact is "Nadia Farrell (user)".

**Then** "Ghost Record"'s Work Contact is cleared, and Nadia Farrell's Work Contact becomes
"Nadia Farrell (user)".

### A7. Creating a user from the employee

**Given** the employee of A1 with no user and Work Email
`nadia.farrell@northwind.example`, and Olga also holds the application-administrator group.

**When** Olga presses "Create User".

**Then** a simplified user form opens, pre-filled with name "Nadia Farrell", login
`nadia.farrell@northwind.example`, telephone the employee's work telephone, mobile the
employee's work mobile, and contact the employee's work contact.

**And when** she saves it, **then** the new user is written onto that employee's User field
and the synchronisation of A2 runs.

**And given instead** that the employee already has a user, **then** pressing "Create User"
fails with "This employee already has an user."

### A8. Bulk user creation classifies every employee

**Given** six employees selected:

| Employee | Work Email | Other state |
|---|---|---|
| Ada | `ada@northwind.example` | already has a user |
| Ben | empty | |
| Cleo | `not-an-address` | |
| Dov | `olga@northwind.example` | a user already exists with that login |
| Eve | `eve@northwind.example` | |
| Fay | `eve@northwind.example` | duplicate of Eve |

**When** Olga runs the bulk user creation and confirms.

**Then**:

1. Exactly **one** user is created, for Eve, with login `eve@northwind.example`.
2. The notification chain carries, in order of construction: a success notice "Users Eve
   creation successful"; a warning "User already exists for Those Employees Ada"; a danger
   notice "You need to set the work email address for Ben"; a danger notice "You need to set
   a valid work email address for Cleo"; a warning "User already exists with the same email
   for Employees Dov"; a warning "The following employees have the same work email address:
   Eve, Fay".
3. Fay gets no user, because she shares a normalised address with Eve.
4. Before any of this, the confirmation reads "You're about to invite new users. 6 users will
   be created with the default user template's rights. Adding new users may increase your
   subscription cost. Do you wish to continue?" with a button labelled "Confirm".

### A9. A person editing their own data notifies the responsible

**Given** employee Nadia Farrell with Human Resources Responsible Olga Reyes, **and** Nadia
is logged in as herself.

**When** Nadia changes her Private Telephone and her Emergency Contact in her own
preferences.

**Then** a notification is posted on her employee record addressed to Olga's contact, reading
"Personal information update.", then "The following fields were modified by Nadia Farrell",
then a two-item list naming the labels of the two fields, then in italics "You are receiving
this message because you are the HR Responsible of this employee."

**And given instead** that the employee has **no** Human Resources Responsible, **then** no
notification is posted at all.

### A10. The self-service boundary

**Given** Nadia is logged in as herself and holds no human-resources group.

**Then**:

1. She can read and write her own Private Telephone, Private Email, Emergency Contact,
   Emergency Telephone, Badge Identifier, Personal Identification Number, Home-Work Distance
   in Kilometres, Visa Expiration Date, Work Location, the seven weekday home-working
   locations, and the rest of the self-writable list.
2. She can read but not write her own Company Employee link, Employee's Working Hours, Work
   Contact and Bank Accounts.
3. She cannot read her own Wage, Contract Start Date or Contract End Date through any path.
4. She cannot read **anyone else's** Private Telephone.

---

## Part B — Employee Versions and the version in force

### B1. An employee with two dated versions: the one chosen on a date

**Given** employee **Marek Dolny** with exactly two versions:

| Version | Effective Date | Department | Job Title | Working Schedule | Wage | Contract Start | Contract End |
|---|---|---|---|---|---|---|---|
| V1 | 1 January 2026 | Sales | Account Manager | Standard 40 Hours | 3 500.00 | 1 January 2026 | empty |
| V2 | 1 July 2026 | Marketing | Campaign Lead | Half-Time Mornings | 3 800.00 | 1 January 2026 | empty |

**Then** the version in force is:

| Requested date | Version returned | Department read through the employee |
|---|---|---|
| 15 December 2025 | V1 (fallback: no version starts on or before that date) | Sales |
| 1 January 2026 | V1 | Sales |
| 30 June 2026 | V1 | Sales |
| 1 July 2026 | V2 | Marketing |
| 11 September 2026 (today) | V2 | Marketing |
| 31 December 2030 | V2 | Marketing |

**And** the stored Current Version is V2.

**And** the derived dates are:

| Version | Effective Start Date | Effective End Date | Is Past | Is Current | Is Future | Is Under Contract |
|---|---|---|---|---|---|---|
| V1 | 1 January 2026 | 30 June 2026 | true | false | false | false |
| V2 | 1 July 2026 | empty | false | true | false | true |

**And** reading the employee with no context returns Department = Marketing, Job Title =
Campaign Lead, Wage = 3 800.00 and Working Schedule = Half-Time Mornings.

**And** reading the employee with the context key `version_id` set to V1's identifier returns
Department = Sales, Job Title = Account Manager, Wage = 3 500.00 and Working Schedule =
Standard 40 Hours.

**And** passing, in that same context key, the identifier of a version belonging to a
**different** employee makes the Version pointer fall back to the Current Version, V2.

### B2. Creating the second version copies the first

**Given** Marek with only V1 as in B1.

**When** Olga creates a version dated 1 July 2026 supplying Department = Marketing, Job Title
= Campaign Lead, Working Schedule = Half-Time Mornings, Wage = 3 800.00.

**Then**:

1. A new version V2 exists with exactly those values, plus the contract dates copied from the
   contract period covering 1 July 2026, that is (1 January 2026, empty).
2. Every other field of V2 — nationality, private address, marital status, work address,
   human resources responsible, salary structure type — equals V1's.
3. V1 is unchanged.
4. The overlap check passes, because V1's and V2's contract periods are identical.

### B3. Creating a version on a date that already has one is a no-operation

**Given** Marek with V1 and V2 as in B1.

**When** Olga creates a version dated **1 July 2026** with Wage = 4 000.00.

**Then** no new version is created and the existing V2 is returned **unchanged**, still
carrying Wage 3 800.00.

### B4. Two active versions cannot share an effective date

**Given** Marek with V1 and V2 as in B1.

**When** Olga creates a third version, directly on the version entity, with Effective Date
1 July 2026 and Active true.

**Then** the save is refused with "An employee cannot have multiple active versions sharing
the same effective date."

**And when** V2 is first archived and the same creation is retried, **then** it succeeds,
because the unique rule only covers active versions.

### B5. An employee must keep at least one version

**Given** Marek with V1 and V2.

**When** Olga deletes both V1 and V2 in one operation.

**Then** the deletion is refused with "Employee Marek Dolny must always have at least one
active version."

**And when** she deletes only V2, **then** the deletion succeeds, the Current Version becomes
V1, and reading the employee returns Department Sales again.

### B6. Every version cannot be archived or unassigned

**Given** Marek with V1 and V2.

**When** Olga archives both at once, **then** the write is refused with "Cannot archive all
the active versions of an employee."

**When** she instead clears the Employee link on both at once, **then** the write is refused
with "Cannot unassign all the active versions of an employee."

**When** she archives only V2, **then** it succeeds and the Current Version becomes V1.

### B7. An employee with only future versions

**Given** employee **Ines Kovac** with exactly one version dated **1 December 2026**.

**Then** the Current Version is that version even though it is in the future, because the
fallback returns the employee's first version; Is Future reads true; and reading the employee
returns that version's department and schedule.

### B8. Falling back to archived versions

**Given** employee Ines with two versions, one dated 1 January 2026 and one dated
1 June 2026, **and** both are archived.

**Then** the version in force on 11 September 2026 is the **1 June 2026** one, because with
no active version the selection falls back to the full set and takes the greatest effective
date not in the future.

### B9. Writing a delegated field lands on the pointed-at version

**Given** Marek with V1 and V2 as in B1, and the Version pointer resolving to V2.

**When** Olga writes Department = Operations on the **employee**.

**Then**:

1. V2's Department becomes Operations.
2. V1's Department is still Sales.
3. V2's Last Modified On becomes the moment of the write and Last Modified By becomes Olga.
4. The tracking entry on the employee's thread is prefixed with "Modified on the Version *the
   display name of V2*".

**And when** she instead opens the employee with the context key `version_id` set to V1 and
writes Department = Operations, **then** V1's Department becomes Operations and V2's stays
Marketing.

### B10. Changing the schedule on the current version moves the resource

**Given** Marek with Current Version V2 whose Working Schedule is Half-Time Mornings, and a
Resource whose schedule is Half-Time Mornings.

**When** Olga writes Working Schedule = Standard 40 Hours on the employee.

**Then** V2's Working Schedule becomes Standard 40 Hours **and** the employee's Resource
schedule becomes Standard 40 Hours.

**And when** she makes the same change through the context of V1 — which is not the current
version — **then** V1's schedule changes but the Resource's schedule does not.

### B11. The nightly refresh moves the current version

**Given** Marek with V1 dated 1 January 2026 and V2 dated **12 September 2026**, today being
11 September 2026, so the Current Version is V1.

**When** the clock passes midnight and the current-version refresh job runs on
12 September 2026.

**Then** the stored Current Version becomes V2, the Public Employee view now projects V2's
department and job position, and the employee's Resource schedule is rewritten to V2's
schedule.

**And** for every employee whose correct Current Version did not change, no write is
performed.

### B12. A version with no contract start date

**Given** employee **Paul Idris** with one version dated 1 January 2026, no Contract Start
Date and no Contract End Date.

**Then** Effective Start Date = 1 January 2026; Effective End Date = empty; Is Current = true;
Is Under Contract = **false**; and the contract period covering today is the empty pair.

### B13. The gap between a contract end and the next version

**Given** employee **Ruth Nagy** with three versions:

| Version | Effective Date | Contract Start | Contract End |
|---|---|---|---|
| V1 | 1 January 2026 | 1 January 2026 | empty |
| V2 | 1 July 2026 | 1 January 2026 | 31 October 2026 |
| V3 | 1 December 2026 | 1 December 2026 | empty |

**Then**:

| Version | Effective Start | Effective End |
|---|---|---|
| V1 | 1 January 2026 | 30 June 2026 |
| V2 | 1 July 2026 | **31 October 2026** |
| V3 | 1 December 2026 | empty |

**And** on 15 November 2026: the version in force is V2, but Is Under Contract for that date
is **false**, and the contract period covering that date is the empty pair.

---

## Part C — Contract periods

### C1. Two overlapping contracts are refused

**Given** employee **Sven Holm** with one version dated 1 January 2026 carrying contract dates
(1 January 2026, 30 June 2026).

**When** Olga creates a version dated 1 May 2026 with contract dates (1 May 2026,
31 December 2026).

**Then** the save is refused with:

> Sven Holm already has a contract running during the selected period.
>
> Please either:
>
> - Change the start date so that it doesn't overlap with the existing contract, or
> - Create a new employee if this employee should have multiple active contracts.

**And when** she instead uses contract dates (1 July 2026, 31 December 2026), **then** the
save succeeds: the two periods touch but do not overlap.

**And when** she uses (30 June 2026, 31 December 2026), **then** it is refused, because
30 June is inside the first period.

### C2. Several versions may share one contract period

**Given** employee Sven with one version dated 1 January 2026 carrying (1 January 2026,
30 June 2026).

**When** Olga creates a version dated 1 April 2026 carrying exactly (1 January 2026,
30 June 2026).

**Then** the save **succeeds**: two versions sharing the same period is the normal case and is
explicitly exempted from the overlap test.

### C3. Start must precede end

**When** Olga sets contract dates (1 July 2026, 30 June 2026) on a version.

**Then** the save is refused with "Start date (2026-07-01) must be earlier than contract end
date (2026-06-30)." — with the two dates rendered as the platform renders them.

### C4. An end date without a start date is impossible

**When** a contract end date is written on a version whose contract start date is empty,
bypassing the form.

**Then** the database refuses it with "The contract must have a start date."

**And** through the form, clearing the Contract Start Date immediately clears the Contract
End Date by an on-change rule, so the combination cannot be produced.

### C5. Closing a contract propagates across the whole period

**Given** employee **Hana Prokop** with three versions, all carrying (1 January 2026, empty):

| Version | Effective Date |
|---|---|
| V1 | 1 January 2026 |
| V2 | 1 April 2026 |
| V3 | 1 August 2026 |

**When** Olga sets Contract End Date = 31 October 2026 on V3 alone.

**Then**:

1. V1, V2 **and** V3 all carry (1 January 2026, 31 October 2026).
2. The derived Effective End Dates become 31 March 2026, 31 July 2026 and 31 October 2026.
3. No overlap error is raised, because the three periods remain identical.

### C6. Writing contract dates across two different contracts is refused

**Given** Hana with V1..V3 in period (1 January 2026, 31 October 2026), **and** a fourth
version V4 dated 1 December 2026 in period (1 December 2026, empty).

**When** Olga selects V3 and V4 together and writes Contract End Date = 31 December 2026.

**Then** the write is refused with "Cannot modify multiple versions contract dates with
different contracts at once."

### C7. Setting a contract start date on a single-version employee moves the effective date

**Given** employee **Juno Best** with exactly **one** version dated 11 September 2026 with no
contract dates.

**When** Olga sets its Contract Start Date to 1 October 2026.

**Then** that version's Effective Date **also** becomes 1 October 2026, so the two stay
aligned.

**And given instead** that Juno has two versions, **then** only the contract start date is
written and the effective dates are left alone.

### C8. Starting a contract before an existing future one

**Given** employee **Karl Vos** with one version dated 1 January 2027 opening the contract
period (1 January 2027, empty).

**When** Olga starts a contract on 1 June 2026.

**Then** a version dated 1 June 2026 is created carrying contract dates (1 June 2026,
**31 December 2026**), because the earliest later period starts on 1 January 2027 and the new
contract is closed the day before.

### C9. Starting a contract on a date that already has a version

**Given** Karl with a version dated 1 June 2026 and no contract dates.

**When** Olga starts a contract on 1 June 2026.

**Then** **no** new version is created; the existing version receives the contract dates.

### C10. Starting a contract on a covered date is refused

**Given** Karl under contract from 1 June 2026 to 31 December 2026.

**When** the guard is invoked for the date 1 August 2026.

**Then** it fails with "The employee is already in contract on 1 Aug 2026. Please select a
date outside existing contracts" — the date rendered in the reader's language using the
medium date pattern.

### C11. Closing before opening

**Given** a version with Contract Start Date 1 January 2026 and no Contract End Date.

**When** the finished-contract guard is invoked on it.

**Then** it fails with "Before creating a new contract, close the current one by setting an
end date."

---

## Part D — Departure

### D1. A departure and everything it archives or cancels

**Given**:

- Employee **Ruben Alba**, Active, of Northwind SA, linked to user Ruben Alba who has **no
  other** employee.
- His Current Version V2 is dated 1 March 2026 and carries contract dates (1 January 2026,
  empty). Version V1 dated 1 January 2026 carries the same contract dates.
- He is the Manager of employees Lin Chen and Omar Diaz, and the Coach of Lin Chen.
- Two equipment items, a laptop and a telephone, are assigned to him.
- He holds skills and resume lines.

**When** Olga archives Ruben, the Departure Registration Wizard opens, and she completes it
with Departure Reason = Resigned, Additional Information = "Leaving for a competitor",
Departure Date = **30 September 2026**, "Set Contract End Date" ticked, "Remove Related User"
ticked, "Free Equipment" ticked, in termination mode.

**Then**, in order:

1. **Date guard passes**: V2's Contract Start Date, 1 January 2026, is not later than
   30 September 2026.
2. **User eligibility**: Ruben's user has exactly one employee, Ruben, and Ruben is in the
   selection, so the user is eligible for archiving.
3. **Employee archived**: Ruben's Active becomes false; his Resource's Active becomes false.
4. **User archived**: the user Ruben Alba is archived.
5. **Reporting links cleared**: Lin Chen's Manager becomes empty, Omar Diaz's Manager becomes
   empty, Lin Chen's Coach becomes empty.
6. **Departure recorded**: V2's Departure Reason becomes Resigned, its Additional Information
   becomes "Leaving for a competitor", its Departure Date becomes 30 September 2026.
7. **Thread message**: a message is posted on Ruben's thread reading "Additional Information:
   \n Leaving for a competitor".
8. **Contract closed**: V2's Contract End Date becomes 30 September 2026 **and**, by
   propagation across the contract period, V1's Contract End Date becomes 30 September 2026
   too. V1's derived Effective End Date is 28 February 2026 and V2's is 30 September 2026.
9. **Equipment freed**: the laptop and the telephone are unassigned from Ruben.
10. **Notification**: a success notice reads "The following users have been archived: Ruben
    Alba".
11. **Not affected**: his skills, his resume lines, his badge identifier, his bank accounts,
    his home-working plan and any working-schedule leaves are all left exactly as they were.

### D2. The departure date may not precede the contract start

**Given** Ruben's Current Version carries Contract Start Date 1 January 2026.

**When** Olga registers a departure dated 31 December 2025.

**Then** the operation fails with "Departure date can't be earlier than the start date of
current contract." and **nothing** is written — the employee is not archived, no departure
field is set, no equipment is freed.

### D3. A user with another active employee is not archived

**Given** user **Sara Loft** linked to employee Sara Loft (Northwind SA) and employee Sara
Loft (Southwind SA), both active.

**When** Olga selects only the Northwind employee and registers a departure with "Remove
Related User" ticked, in termination mode.

**Then**:

1. The Northwind employee is archived.
2. The user is **not** archived.
3. A danger notice reads "The following users have not been archived as they are still linked
   to another active employees: Sara Loft".

**And when** she selects **both** employees and repeats, **then** both are archived and the
user is archived too, with the success notice.

### D4. Bulk archiving opens no wizard

**Given** three active employees selected.

**When** Olga archives all three at once.

**Then** all three are archived, the Manager and Coach links pointing at them are emptied
everywhere, and **no** Departure Registration Wizard opens — the wizard is only offered when
exactly one employee is archived.

### D5. Unarchiving clears the departure but not the contract end

**Given** Ruben after scenario D1.

**When** Olga unarchives him.

**Then**:

1. His Active becomes true and his Resource's Active becomes true.
2. His Departure Reason, Additional Information and Departure Date are all cleared.
3. His Contract End Date **remains** 30 September 2026 on both V1 and V2.
4. Lin Chen's and Omar Diaz's Manager links are **not** restored.

### D6. Registering a departure without archiving

**Given** the wizard is launched directly over a selection, **not** in termination mode.

**When** Olga confirms with a reason and a date.

**Then** the departure fields are written on each employee's current version and, when ticked,
the contract is closed and the equipment freed — but **no** employee and **no** user is
archived.

### D7. A departure reason cannot be deleted

**When** Olga deletes the shipped departure reason "Retired".

**Then** the deletion is refused with "Default departure reasons cannot be deleted."

**And when** she deletes a custom departure reason that a version still references, **then**
the deletion is refused by the restrict rule on the relation.

---

## Part E — The privacy boundary

### E1. An ordinary user reading the employee list gets public data

**Given** **Lin Chen**, an internal user holding no human-resources group.

**When** Lin opens the employee directory and reads the list.

**Then**:

1. She sees names, departments, job positions, job titles, work telephone numbers, work
   electronic mail addresses, work locations, managers, coaches, photographs and presence
   icons.
2. She does **not** see any date of birth, private telephone number, private address,
   national identification number, badge identifier, personal identification number, bank
   account, wage or contract date.
3. When the client explicitly asks the private Employee model for the list, the search runs
   against the Public Employee and the resulting identifiers are returned as private-model
   records, so the client sees a coherent recordset.

### E2. Asking for a private field by name fails cleanly

**Given** Lin Chen as above.

**When** the client asks the private Employee for the field names "name" and "birthday".

**Then** the call fails with:

> The fields “birthday”, which you are trying to read, are not available for employee public
> profiles.

### E3. Opening the private form redirects

**Given** Lin Chen.

**When** she follows a link that resolves to an employee record.

**Then** the form that opens is the **public** employee form, and the action's target entity
has been switched to the Public Employee.

**And when** the client requests several views of the private Employee at once, **then** the
request is refused with:

> You are not allowed to access "Employee" (hr.employee) records.
> We can redirect you to the public employee list.

offered together with a button labelled "Employees profile".

### E4. Display names never leak through an error

**Given** Lin Chen.

**When** a record she can read shows a link to an employee.

**Then** the link's label is the employee's name, taken from the Public Employee — not a raw
identifier and not an access error.

### E5. The consented date of birth is the only personal data that crosses

**Given** employee Nadia Farrell with Date of Birth **14 March 1990**.

**Case 1** — "Show to all employees" is **off**. **Then** Lin Chen reads Public Date of Birth
as the literal text `hidden`, and cannot read the Date of Birth at all.

**Case 2** — "Show to all employees" is **on**. **Then** Lin Chen reads Public Date of Birth
as "14 March", and still cannot read the Date of Birth itself, so the **year** never leaves
the private model.

### E6. An officer sees people but not pay

**Given** **Pia Lund**, a Human Resources Officer who is **not** an Administrator.

**Then**:

1. She can read and write every private employee field: dates of birth, private addresses,
   identification numbers, badge identifiers, bank accounts, tags, skills, departure data.
2. She **cannot** read Wage, Contract Wage, Contract Start Date, Contract End Date, End of
   Trial Period, Salary Structure Type, Contract Type, Effective Start Date, Effective End
   Date, Is Current, Is Past, Is Future or Is Under Contract.
3. The Payroll tab of the employee form is not shown to her, and the contract filters of the
   search view are not offered.

### E7. Bank account visibility and masking

**Given** a bank account `BE68539007547034` owned by a contact that is the work contact of an
employee, and a second bank account owned by an ordinary supplier contact.

**Then**:

1. Lin Chen, an ordinary internal user, can see the supplier's account and **cannot** see the
   employee's.
2. If the employee's account nevertheless reaches Lin through a relation she may traverse,
   its display name reads `BE**********7034` — two visible characters, ten asterisks, four
   visible characters.
3. Pia Lund, a Human Resources Officer, sees both accounts with their full numbers.

### E8. Editing a many-to-many of employees without read access

**Given** a record of some other entity that carries a link-to-many field targeting the
Employee, and Lin Chen who cannot read the Employee.

**When** Lin creates or writes that record, setting the employee link.

**Then** the write succeeds: the sentinel placed in the reading context by the shared
behaviour grants read — and only read — on the Employee for the duration of the operation.

**And when** Lin tries to read an employee outside such an operation, **then** the access
check refuses as usual.

### E9. Seeing across companies for reporting lines

**Given** Lin Chen of Northwind SA whose Manager is **Emil Roth**, an employee of Southwind
SA; and Lin's allowed companies contain only Northwind SA.

**Then** Lin can see Emil's Public Employee record, because the record rule's third clause
permits reading one's own manager across companies.

**And given** Emil's user is the reader instead, **then** Emil can see Lin, because the second
clause permits a manager to see their reports across companies.

---

## Part F — Skills

### F1. A skill progressing a level

**Given** employee Nadia Farrell with exactly one skill assertion:

| Record | Skill | Level | Validity Start | Validity Stop |
|---|---|---|---|---|
| S1 | English | A2 (progress 40) | 1 March 2026 | empty |

and today is 11 September 2026, and the "Languages" skill type is **not** a certification
type.

**When** Olga changes S1's level to **B1** (progress 60) and saves.

**Then**:

1. S1 still exists with Level A2 and Validity Stop **10 September 2026** (yesterday).
2. A new record S2 exists with Skill English, Level B1, Validity Start **11 September 2026**
   (today) and Validity Stop empty.
3. The employee's Current Skills contains only S2.
4. The employee's Skills contains both S1 and S2.
5. The employee's Distinct Skills contains English exactly once.
6. The Skills Inventory report shows one row for Nadia and English at level B1 with a progress
   value of **0.60**.
7. The Skill History report shows, at date 1 March 2026, English at 40; and at date
   11 September 2026, English at 60.

### F2. Progressing a level on the day the skill was recorded

**Given** the same employee, but S1 was recorded **today**, 11 September 2026, with Validity
Start 11 September 2026.

**When** Olga changes the level to B1.

**Then** S1 is **deleted** outright rather than stopped, because its Validity Start is not
earlier than yesterday; only S2 remains, with Validity Start 11 September 2026.

### F3. Adding a second level of the same skill expires the first

**Given** Nadia holding English at A2 since 1 March 2026, still valid.

**When** Olga adds a **new** assertion English at C1 with Validity Start today.

**Then** the A2 assertion is stopped at 10 September 2026 and the C1 assertion is created —
the same outcome as changing the level, reached through a create instruction instead of a
write.

### F4. Two valid levels of the same skill are refused

**When** an assertion English at B2 with Validity Start 1 June 2026 and Validity Stop empty is
created directly while an assertion English at A2 with Validity Start 1 March 2026 and
Validity Stop empty already exists and the create rewriting is bypassed.

**Then** the overlap constraint refuses with a message beginning "The following skills can't
be created as they overlap or exactly match existing skills:" and continuing with a bullet
naming the conflicting stored record, its display name "English: A2" and its window.

### F5. Certifications may coexist with different windows

**Given** a skill type "Platform Certifications" flagged as a certification type, with a level
"Certified".

**When** Olga records, for Nadia, "Platform: Certified" valid from 1 January 2024 to
31 December 2024, and then "Platform: Certified" valid from 1 June 2025 to 31 May 2026.

**Then** both assertions exist and neither conflicts, because the windows differ.

**And when** she records a third with exactly the same skill, level, start and stop as the
second, **then** the duplicate is silently dropped and no third record appears.

### F6. A lapsed certification stays visible

**Given** Nadia's only "Platform: Certified" assertion ran to **31 May 2026**, which is in the
past.

**Then** her Current Skills contains that expired assertion anyway, because it is a
certification skill with no still-valid assertion and the most recently expired one is kept.

**And given** she also holds a still-valid "Platform: Certified" from 1 June 2026 to
31 May 2027, **then** only the valid one appears.

### F7. The validity window must be ordered

**When** Olga records a skill with Validity Start 1 October 2026 and Validity Stop
1 September 2026.

**Then** the save is refused with a message beginning "The following skills have their valid
stop date prior to their valid start date:" followed by a bullet naming the assertion and the
two dates.

**And** while still editing the form, the Show Date Warning marker turns true so the interface
warns before the save is attempted.

### F8. Skill and level must match the skill type

**When** Olga records an assertion whose Skill Type is "Languages" but whose Skill is a soft
skill.

**Then** the save is refused with "The skill Leadership and skill type Languages don't match".

**And when** the Skill Type is "Languages" but the Level belongs to "Soft Skills", **then**
the save is refused with "The skill level Advanced is not valid for skill type: Languages".

### F9. Only one default level per skill type

**Given** the "Languages" skill type whose default level is A1.

**When** Olga sets B1 as the default level.

**Then** A1's default flag becomes false and B1's becomes true; no other level of the type
carries the flag.

### F10. A skill type must not be emptied

**When** Olga removes every level from the "Soft Skills" type.

**Then** the save is refused with "The following skills type must contain at least one skill
and one level: Soft Skills".

### F11. Level progress is bounded

**When** Olga sets a skill level's Progress to 120.

**Then** the database refuses with "Progress should be a number between 0 and 100."

### F12. A non-officer may edit only their own skills

**Given** Lin Chen, an internal user with no human-resources group.

**Then** she can read every employee's skills and resume lines, and can create, change and
remove **only** her own.

**And when** she tries to add a skill to Nadia Farrell, **then** the record rule refuses.

### F13. Certification reminders

**Given** job position "Platform Engineer" expecting "Platform: Certified"; employee **Tom
Bauer** holding that job, with a linked user; today 11 September 2026; three months from today
is 11 December 2026.

| Tom's certification state | Reminder raised? | Deadline |
|---|---|---|
| No "Platform: Certified" assertion at all | **yes** | today, 11 September 2026 |
| Valid to 1 November 2026 | **yes** | 1 November 2026 |
| Valid to 1 March 2027 | no — later than three months away | — |
| Valid with **no** stop date | no — treated as permanent | — |
| Any of the above, but an open file-upload activity with the summary "Platform: Certified" already exists on Tom | no — already covered | — |

The activity's summary is "Platform: Certified", its note is "Certification missing or
expiring soon", and it is assigned to Tom's user; failing that his manager's user; failing
that the job position's recruiter; and the employee is skipped when none exists.

---

## Part G — Presence

### G1. Presence derived from a sign-in, with the base rule only

**Given** company Northwind SA with "Presence Based on User Status" **on** and the two
advanced rules **off**; employee Nadia Farrell, Active, linked to user Nadia Farrell, working
schedule Standard 40 Hours, time zone Europe/Brussels; the moment is Friday 11 September 2026
at 10:15 Brussels time, which is inside her 08:00–12:00 attendance.

| Situation | User messaging status | Inside working hours now? | Presence State | Presence Icon | Icon shown? |
|---|---|---|---|---|---|
| She signs in and the client reports her as online | `online` | yes | `present` | `presence_present` | yes, she has a user |
| She signs in and the client reports her as online, but at 20:00 | `online` | no | `present` | `presence_present` | yes |
| She has not signed in | `offline` | yes | `absent` | `presence_absent` | yes |
| She has not signed in, at 20:00 | `offline` | no | `out_of_working_hour` | `presence_out_of_working_hour` | yes |
| She is signed in but idle | `away` | yes | `out_of_working_hour` | `presence_out_of_working_hour` | yes |
| Her record is archived | anything | — | `archive` | `presence_archive` | yes |
| She has no linked user at all | — | — | `out_of_working_hour` | `presence_out_of_working_hour` | **no** |

**And** the "inside working hours now" probe uses the window from **now** to **now plus one
hour**, evaluated in her time zone against her working schedule; it is run once per (time
zone, schedule) pair, not once per employee.

**And** the probe is only run at all for the employees whose messaging status is `offline`.

### G2. The sign-in also records the network address

**Given** Nadia is an internal user and connects from the address `192.0.2.44`, and no
connection log row exists for her, that address and today.

**When** her messaging presence is refreshed.

**Then** exactly one connection log row is created carrying that address, in its own
transaction; and a second refresh from the same address on the same day creates **no** second
row.

### G3. Last activity

**Given** Nadia's last recorded presence is 11 September 2026 at 09:02 in the reference frame,
and her time zone is Europe/Brussels.

**Then** Last Activity Date reads 11 September 2026 and Last Activity Time reads the short
clock rendering of that moment in her time zone.

**And given** her last recorded presence is 9 September 2026, **then** Last Activity Date reads
9 September 2026 and Last Activity Time is **empty**, because it is only filled when the last
presence falls on today.

### G4. Advanced presence with the message-count rule

**Given** Northwind SA with "Presence Based on Messages Sent" on and a threshold of **5**, and
the presence job last ran today.

| Messages Nadia's contact authored since midnight | Message Sent Today after the job | Inside working hours? | Presence State |
|---|---|---|---|
| 7 | true | yes | `present` |
| 7 | true | no | `out_of_working_hour` |
| 2 | false | yes, and the time-off layer says she is absent | `absent` |
| 2 | false | yes, and the time-off layer does not say she is absent | `out_of_working_hour` |

**And given** the company's Last Presence Computation stamp falls on **yesterday**, **then**
even with Message Sent Today true and inside working hours, the state is not `present` — the
stale evidence is ignored and the state falls to `out_of_working_hour` or `absent` by the
second and third branches.

### G5. Advanced presence with the network-address rule

**Given** "Presence Based on Network Address" on, and the company's valid address list is
`192.0.2.44,192.0.2.45`.

**When** the presence job runs and Nadia's connection log for today contains `192.0.2.44`.

**Then** her Network Address Seen Today becomes true and she is removed from the message-count
pass — the two rules are evaluated in order, network address first, and each employee is
attributed to at most one of them.

### G6. A manual declaration overrides everything

**Given** advanced presence on, and Nadia currently computing as `absent`.

**When** Olga, a Human Resources Administrator, declares her **present**.

**Then** Manually Set Present becomes true, Manual Presence Override Active becomes true, the
Stored Presence State becomes `present`, and every subsequent read of her Presence State
returns `present` regardless of the evidence.

**And when** Olga declares her **absent** instead, **then** Manually Set Present becomes
false, Manual Presence Override Active stays true, the Stored Presence State becomes `absent`,
and reads return `absent`.

**And when** Pia Lund, an officer who is not an administrator, attempts either, **then** the
operation fails with "You don't have the right to do this. Please contact an Administrator."

### G7. Writing the stored presence state to present sets the manual flag

**When** any write sets an employee's Stored Presence State to `present`.

**Then** Manually Set Present is also set to true by the same write.

### G8. The presence job resets before it evaluates

**When** the presence job runs for Northwind SA.

**Then**, in order: all four indicator flags are reset to false on every employee of the
company; the network-address rule is applied if switched on; the message-count rule is applied
to the remaining employees if switched on; the company's Last Presence Computation is stamped
with the current moment; and the freshly computed presence state is copied into the Stored
Presence State of **every** employee of the company.

---

## Part H — Home working

### H1. A home-working exception on one weekday

**Given** employee Nadia Farrell with the weekly plan Monday Office, Tuesday Office, Wednesday
Home, Thursday **Office**, Friday Home, Saturday empty, Sunday empty; and no exceptions.

**When** she opens the planner on **Thursday 17 September 2026**, chooses **Home** and leaves
"Recurring" off, and confirms.

**Then**:

1. An Employee Home-Working Location is created with Employee = Nadia, Date =
   17 September 2026, Location = Home.
2. Her Thursday weekday field is still Office.
3. On 17 September 2026: Work Location Name reads "Home", Work Location Kind reads `home`,
   Presence Icon reads `presence_home`, Show Presence Icon is true, and her instant-messaging
   status becomes `home_` followed by her messaging status — for example `home_online`.
4. On **24 September 2026**, the next Thursday, all four read Office / `office` /
   `presence_office` / `office_online`, because the exception was bound to one date.

### H2. Making the exception recurring removes it

**Given** the state after H1.

**When** Nadia reopens the planner on 17 September 2026, chooses Home and switches "Recurring"
**on**.

**Then**:

1. The exception for 17 September 2026 is **deleted first**.
2. Her Thursday weekday field is written to Home, through her user, so the write passes the
   self-service writable list.
3. Both 17 and 24 September now read Home, and no exception record exists.

### H3. Choosing the weekday default deletes a redundant exception

**Given** the state after H2 — Thursday default is Home — and an exception (Nadia,
1 October 2026, Office).

**When** Nadia sets 1 October 2026 to **Home**, with "Recurring" off.

**Then** the exception is **deleted**; no record with a location equal to the weekday default
is ever stored.

### H4. Updating an existing exception

**Given** Thursday default Home, and an exception (Nadia, 1 October 2026, Office).

**When** Nadia sets 1 October 2026 to **Other**, with "Recurring" off.

**Then** the existing exception is rewritten to Other; no second exception is created.

### H5. One exception per employee per day

**When** two exceptions for Nadia on 1 October 2026 are created, bypassing the wizard.

**Then** the second is refused with "Only one default work location and one exceptional work
location per day per employee."

### H6. Deleting a work location

**Given** work location "Satellite Office" used as Nadia's Tuesday default.

**When** Olga deletes it.

**Then** the deletion is refused with "You cannot delete locations that are being used by your
employees".

**And given** instead that it is used only by an exception on 3 November 2026 and by no
weekday default, **then** the deletion succeeds and that exception is deleted, so
3 November 2026 returns to the Tuesday default.

### H7. The home-working payload

**Given** Nadia with the plan of H1 and one exception (17 September 2026, Home).

**When** the work-location payload is requested for the window 14 to 20 September 2026.

**Then** the payload for Nadia carries her user identifier, her employee identifier, her
contact identifier, her name, seven weekday entries each with a location kind, a location name
and a work location identifier, and one exception entry keyed `2026-09-17` carrying the
exception record's identifier, the kind `home`, the name "Home" and the work location
identifier.

**And given** she has no exception in the window, **then** the payload carries **no**
exceptions member at all.

---

## Part I — Departments, jobs and the organisation chart

### I1. Department manager change re-points the right employees

**Given** department **Research** with Manager **Alice**; members Bob (Manager Alice), Carol
(Manager Alice), Dan (Manager Bob).

**When** Olga sets Research's Manager to **Erin**.

**Then** Bob's Manager becomes Erin; Carol's Manager becomes Erin; **Dan's Manager stays Bob**;
and Research's Manager becomes Erin. If Erin herself were a member reporting to Alice, she
would be excluded from the re-pointing and keep her own manager link.

### I2. Department cycles are refused

**When** Olga sets department A's Parent Department to department B while B's Parent Department
is already A.

**Then** the save is refused with "You cannot create recursive departments."

### I3. Complete name and master department

**Given** departments Group → Europe → Belgium.

**Then** Belgium's Complete Name reads "Group / Europe / Belgium"; its display name is the
complete name, unless the reading context asks for non-hierarchical naming, in which case it is
"Belgium"; its Master Department is Group; and its hierarchy path lists the three identifiers
in order.

### I4. Department counters

**Given** Research with 4 employees of Northwind SA and 2 of Southwind SA, and the reader's
allowed companies are Northwind SA only.

**Then** Total Employees reads **4**.

**And given** three activity plans exist — two scoped to Research and one with no department —
**then** Activity Plans Count reads **3**.

### I5. Job headcount forecast

**Given** job "Senior Developer" currently held by 4 active employees, with Target 2.

**Then** Current Number of Employees reads 4 and Total Forecasted Employees reads **6**.

**And when** Target is set to −1, **then** the save is refused with "The expected number of new
employees must be positive."

### I6. Job name uniqueness

**Given** job "Developer" in department Research of Northwind SA.

**When** Olga creates another "Developer" in department Research of Northwind SA, **then** the
save is refused with "The name of the job position must be unique per department in company!"

**When** she creates "Developer" in department **Sales** of Northwind SA, **then** it succeeds.

### I7. The organisation chart around an employee

**Given** the chain Fred → Erin → Dana → Cody → Bree → Alma (each reporting to the next), and
Fred has two direct subordinates, Gwen and Hugo.

**When** the chart is requested for Fred with no requested depth.

**Then**:

1. The centred node is Fred, carrying his identifier, name, a link of the form
   `/mail/view?model=hr.employee.public&res_id=` followed by his identifier, his job identifier
   and name, a direct subordinate count of 2, his indirect subordinate count, and his last
   write moment in whole milliseconds.
2. The manager list contains at most five ancestors, ordered most senior first.
3. The "more managers" flag is true when more than five ancestors exist.
4. The children list contains Gwen and Hugo.

**And given** Fred cannot be read by the caller, **then** the response carries an empty manager
list and an empty children list and nothing else.

### I8. A reporting cycle terminates

**Given** Alma reports to Fred and Fred reports to Alma.

**Then**:

1. The subordinate walk from Alma terminates, and Alma is **not** counted among her own
   subordinates.
2. The chart's ancestor walk stops as soon as a person already collected is reached again.
3. An activity plan whose responsible kind is Manager, launched on someone inside such a cycle
   where nobody has a user, reports "Oops! It seems there is a problem with your team
   structure. We found a circular reporting loop and no one in that loop is linked to a user.
   Please double-check that everyone reports to the correct manager."

### I9. Subordinate listing kinds

**Given** Fred with direct subordinates Gwen and Hugo, and Gwen with direct subordinate Ivy.

| Requested kind | Result |
|---|---|
| `direct` | Gwen, Hugo |
| `indirect` | Ivy |
| absent | Gwen, Hugo, Ivy |

---

## Part J — Activity plans and reminders

### J1. Responsible resolution, the happy path

**Given** employee Nadia with Manager Olga (who has a user) and Coach Pia (who has a user).

**Then** a template with responsible kind Manager resolves to Olga's user; Coach resolves to
Pia's user; Employee resolves to Nadia's own user.

### J2. Walking up the chain when the designated person has no user

**Given** Nadia's Manager is **Quinn**, who has no user; Quinn's Manager is **Rita**, who also
has no user; Rita's Manager is Olga, who has a user.

**When** a template with responsible kind Manager is resolved.

**Then** the responsible is **Olga's user**, no error is raised, and the warning "The manager
of Nadia Farrell should be linked to a user." is returned so the interface can explain the
substitution.

**And given** nobody in the chain has a user, **then** the responsible is the **acting user**
and the same warning is returned.

### J3. No manager at all

**Given** Nadia has no Manager.

**When** a template with responsible kind Manager is resolved.

**Then** the error "Manager of employee Nadia Farrell is not set." is returned.

### J4. The suggested plan date

**Given** three employees selected with Effective Start Dates 1 August 2026, 1 October 2026 and
1 December 2026; today is 11 September 2026.

**Then** the earliest is 1 August 2026, which is before today, so the suggested date is
**11 October 2026** — today plus thirty days.

**And given** the earliest were 1 November 2026, which is fifty-one days away, **then** the
suggested date is **1 November 2026** itself.

**And given** the earliest were 25 September 2026, which is fourteen days away, **then** the
suggested date is **11 October 2026**.

### J5. Plan department filtering

**Given** plans: "General Onboarding" with no department, "Research Onboarding" scoped to
Research, "Sales Onboarding" scoped to Sales.

| Selection | Resolved department | Plans offered |
|---|---|---|
| One employee of Research | Research | General Onboarding, Research Onboarding |
| Two employees, one of Research and one of Sales | empty | General Onboarding only |
| One employee with no department | empty | General Onboarding only |

### J6. Contract expiry reminder

**Given** Northwind SA with a Contract Expiry Notice Period of 7 days; today 11 September 2026.

| Employee | Contract Start | Contract End | Reminder raised today? |
|---|---|---|---|
| A | 1 January 2026 | 18 September 2026 | **yes**, deadline 18 September 2026 |
| B | 1 January 2026 | 19 September 2026 | no |
| C | 1 January 2026 | 17 September 2026 | no — it was raised yesterday |
| D | 1 October 2026 | 18 September 2026 | no — the start date is not earlier than today |
| E | empty | empty | no |

The activity summary reads "The contract of *the employee name* is about to expire." and is
assigned to the version's Human Resources Responsible, or to the acting user when there is
none.

### J7. Work permit expiry reminder

**Given** a Work Permit Expiry Notice Period of 60 days; today 11 September 2026.

| Employee | Work Permit Expiration | Reminder raised today? |
|---|---|---|
| A | 10 November 2026 | **yes**, deadline 10 November 2026 |
| B | 11 November 2026 | no |
| C | empty | no |

The summary reads "The work permit of *the employee name* is about to expire."

**And when** the Work Permit Expiration Date is written afterwards, **then** the Work Permit
Reminder Scheduled flag is reset to false by that same write.

---

## Part K — Bank accounts and the salary distribution

### K1. Adding a third account to a full distribution

**Given** Nadia with accounts A (60 %, order 1) and B (40 %, order 2), the two percentages
summing to 100.

**When** account C is added to her bank accounts.

**Then** the distribution becomes A 60 % (order 1), B 40 % (order 2), C **0 %** (order 3),
because the remaining share is max(0, 100 − 100) = 0. The total is still 100 and the
constraint passes.

### K2. Removing an account redistributes its share to the first

**Given** Nadia with A (60 %, order 1) and B (40 %, order 2).

**When** account A is removed.

**Then** B's amount becomes **100 %** and the distribution has one entry. The redistribution
target is the first entry of the ordered remainder, and it is applied only because that first
entry is itself a percentage entry.

**And given instead** that B were a **fixed** entry of 500.00, **then** the removed 60 % is
**not** redistributed, and the distribution contains one fixed entry of 500.00 with no
percentage entries, so the sum-to-100 constraint does not apply.

### K3. Ordering of the rebalanced map

**Given** a distribution with A (fixed 300.00, order 1), B (30 %, order 5), C (70 %, order 3),
and account D is added.

**Then** the ordered remainder is B and C first — because percentage entries sort before fixed
ones — ordered by their own order numbers, giving C (order 3), B (order 5), then A (order 1)
last. The allocated percentage total is 30 + 70 = 100, the remaining is 0, and D is added with
0 % at order 6.

### K4. Percentage validation

**When** the distribution contains a percentage entry with amount 120.

**Then** the save is refused with "Each amount percentage must be a number between 0 and 100."

**When** the percentage entries sum to 99.5.

**Then** the save is refused with "Total salary distribution on bank accounts must be exactly
100%."

**When** the percentage entries sum to 99.99995, which differs from 100 by 0.00005.

**Then** the save **passes**, because the tolerance is four decimal places.

### K5. Saving the allocation wizard

**Given** Nadia with accounts A and B, and the wizard opened.

**When** Olga sets A to 66.666 % and B to 33.334 %, and saves.

**Then** each amount is rounded **down** to two decimal places, giving A 66.66 and B 33.33,
whose sum is 99.99. That differs from 100 by 0.01, which exceeds the four-decimal tolerance, so
the save is refused with "Total percentage allocation must equal 100%."

**And when** she sets A to 66.67 and B to 33.33, **then** the sum is exactly 100.00 and the save
succeeds, writing the map and pushing each line's trust flag onto its bank account.

### K6. The wizard refuses to open on an inconsistent state

**Given** Nadia has account C among her bank accounts but the distribution map has no entry for
C.

**When** the wizard is created.

**Then** it fails with "Bank account *the account* not found within the salary distribution of
the employee".

### K7. The primary account

**Given** the distribution A (order 3), B (order 1), C (order 7).

**Then** the Primary Bank Account is **B**, and Primary Account is Trusted mirrors B's
outgoing-payment permission.

**And given** account D is among the employee's accounts but absent from the map, **then** D
sorts last and never becomes the primary account while any mapped account exists.

### K8. Changing the work contact moves and distrusts the accounts

**Given** Nadia's Work Contact is contact P, and her account A is owned by P with the
outgoing-payment permission granted.

**When** Olga changes her Work Contact to contact Q.

**Then** account A's owner becomes Q **and** its outgoing-payment permission is revoked.

---

## Part L — Validation of identifiers

### L1. Badge identifier format

| Value written | Outcome |
|---|---|
| `041123456789` | accepted |
| `ABC123` | accepted |
| empty | accepted |
| `ABC-123` | refused: "The Badge ID must be alphanumeric without any accents and no longer than 18 characters." |
| `Café1` | refused, same message |
| a nineteen-character alphanumeric value | refused, same message |
| a value already used by another employee | refused: "The Badge ID must be unique, this one is already assigned to another employee." |

### L2. Generated badge identifiers

**When** Olga presses the generate operation on three selected employees.

**Then** each receives a twelve-character value beginning with `041` followed by nine digits,
and each satisfies the format rule of L1.

### L3. Personal identification number format

| Value written | Outcome |
|---|---|
| `1234` | accepted |
| empty | accepted |
| `12a4` | refused: "The PIN must be a sequence of digits." |
| `12 34` | refused, same message |

### L4. Employee tag uniqueness

**When** Olga creates a second employee tag named "Remote".

**Then** the save is refused with "Tag name already exists!"

---

## Part M — Derived dates and schedules

### M1. Home-to-work distance conversion

| Distance entered | Unit | Home-Work Distance in Kilometres |
|---|---|---|
| 20 | miles | 20 × 1.609 = 32.18 → stored as **32** |
| 20 | kilometres | **20** |
| 0 | miles | **0** |

**And when** the kilometre field is written directly with 50 while the unit is miles, **then**
the distance becomes 50 ÷ 1.609 = 31.075… → stored as **31**.

### M2. Normalised wage

**Given** a version with Wage 4 000.00 and a working schedule of 38 hours per week.

**Then** the normalised wage is 4 000 × 12 ÷ 52 ÷ 38 = **24.29** after rounding to 0.01.

**And given** the schedule has 0 weekly hours, **then** the normalised wage is **0**.

**And given** the version has **no** working schedule at all, **then** the normalised wage is
the wage itself, **4 000.00**.

### M3. Age

| Date of Birth | Target date | Age |
|---|---|---|
| 14 March 1990 | 11 September 2026 | 36 |
| 14 March 1990 | 1 February 2026 | 35 |
| empty | any | 0 |

### M4. Newly hired

**Given** today 11 September 2026, so the cut-off is 13 June 2026.

| Creation timestamp | Newly Hired |
|---|---|
| 1 September 2026 | true |
| 13 June 2026 at 09:00 | true if strictly later than the cut-off moment |
| 1 March 2026 | false |
| empty | false |

### M5. First version date with gap removal

**Given** employee **Zara Toth** with three versions:

| Version | Effective Start | Effective End | Contract Start |
|---|---|---|---|
| P | 1 March 2021 | 31 August 2022 | 1 March 2021 |
| Q | 1 September 2024 | 31 December 2025 | 1 September 2024 |
| R | 1 January 2026 | empty | 1 September 2024 |

**When** Olga asks for the first version date with gap removal on.

**Then**:

1. Sorted descending, the walk starts at R with reference date 1 January 2026.
2. Q's gap is 1 January 2026 − 31 December 2025 = 1 day, below the four-day threshold, so the
   walk continues with reference date 1 September 2024.
3. P's gap is 1 September 2024 − 31 August 2022 = 732 days, at or above the threshold, so the
   result is Q plus R.
4. The first version date is **1 September 2024** and the first contract date is
   **1 September 2024**.

**And when** she asks with gap removal **off**, **then** the first version date is
**1 March 2021**.

**And when** Lin Chen, who is not an officer, asks, **then** the operation fails with "Only HR
users can access first version date on an employee."

### M6. A version with no end date never triggers the gap cut

**Given** two versions, the older of which has **no** Effective End Date.

**Then** the gap for that older version is computed against 1 January 2100, producing a large
negative number, which is below the four-day threshold, so the cut never fires there.

### M7. Unusual days across two versions with a gap

**Given** Ruth Nagy of B13 and a requested window of 1 October 2026 to 30 November 2026.

**Then**:

1. The overlapping versions are V2 (contract to 31 October 2026) and, for the tail, none.
2. For 1 to 31 October, the unusual days come from V2's working schedule.
3. For 1 to 30 November, the cursor has passed V2's Effective End Date, so **every** day is
   marked unusual, because no version covers the period.

**And given** an employee with no overlapping version at all, **then** every day of the window
is marked unusual.

### M8. Calendar periods split at a schedule change

**Given** Marek of B1, with V1 on Standard 40 Hours until 30 June 2026 and V2 on Half-Time
Mornings from 1 July 2026, both under contract from 1 January 2026 with no end.

**When** calendar periods are requested for the window 1 June 2026 to 31 July 2026.

**Then** two periods are produced: 1 June 2026 to the end of 30 June 2026 with Standard 40
Hours, and 1 July 2026 to 31 July 2026 with Half-Time Mornings — each expressed in that
schedule's own time zone.

### M9. A fully flexible version

**Given** a version with **no** working schedule.

**Then** Is Fully Flexible is true, Is Flexible is true, and the calendar period for that
stretch carries an empty schedule; the time zone used for that stretch is the employee's own
Resource time zone rather than a schedule's.

---

## Part N — Meeting availability

### N1. An attendee outside their working hours is flagged

**Given** a meeting from 11 September 2026 18:00 to 19:00 Brussels time, with attendee Nadia
whose schedule ends at 17:00.

**Then** Nadia is added to the meeting's unavailable attendees, because the total duration of
(her schedule ∩ the meeting) is 0 and the meeting's duration is one hour.

### N2. An attendee fully inside their working hours is not flagged

**Given** the same meeting from 10:00 to 11:00.

**Then** Nadia is not flagged.

### N3. A meeting straddling lunch

**Given** a meeting from 11:30 to 13:30, and Nadia's schedule 08:00–12:00 and 13:00–17:00.

**Then** the intersection is 11:30–12:00 plus 13:00–13:30, totalling one hour, while the
meeting lasts two hours, so Nadia **is** flagged.

### N4. An all-day meeting spanning a closed day

**Given** an all-day meeting covering Friday 11 and Saturday 12 September 2026, and the
company's schedule is closed on Saturday.

**Then** the meeting's interval is computed as **empty** and the availability check is skipped
entirely, so nobody is flagged for that meeting.

### N5. Working hours common to all attendees

**Given** attendees Nadia (08:00–12:00, 13:00–17:00 Brussels) and Marek (09:00–13:00,
14:00–18:00 Brussels), asked for the range 14 to 18 September 2026, with the reading user in
Brussels.

**Then** the returned display rules describe, for each weekday of the range, the intersection
of the two schedules — 09:00–12:00 and 14:00–17:00 — with each weekday expressed with Sunday
as 0 (so Monday is 1, Friday is 5) and the times in hours and minutes.

**And given** the two schedules had no overlap at all, **then** a single rule with the weekday
value 7 and a zero-length time range is returned, which the client renders as "the whole week
is outside working hours".

---

## Part O — Contract templates

### O1. Applying a template through the wizard

**Given** a contract template "Standard Developer" with Job Position = Developer, Department =
Research, Contract Type = Permanent, Salary Structure Type = Employee, Wage = 3 200.00,
Working Hours = Standard 40 Hours, Human Resources Responsible = Olga; and it also carries a
Nationality of Belgium and a Marital Status of married.

**When** Olga applies it to employee Tom Bauer through the wizard.

**Then**:

1. Tom's current version receives Job Position = Developer, Department = Research, Contract
   Type = Permanent, Salary Structure Type = Employee, Wage = 3 200.00, Working Hours =
   Standard 40 Hours, Human Resources Responsible = Olga.
2. Tom's Nationality and Marital Status are **not** changed, because they are outside the
   seven-field whitelist.
3. Tom's version's Contract Template field records "Standard Developer".

### O2. Explicit values beat the template at version creation

**When** a version is created with Contract Template = "Standard Developer" **and** Wage =
3 500.00.

**Then** the created version carries Wage **3 500.00**, and the other six whitelisted values
from the template.

### O3. A template is not a version of anybody

**Given** "Standard Developer" with no Employee link.

**Then**:

1. Its display name is its Name, not a date.
2. It is excluded from every version-selection algorithm.
3. It is listed by the Contract Templates action and excluded from the Employee Records
   action.
4. Following a link to it opens the contract template form rather than an employee form.

---

## Part P — Discussion channels, aliases and equipment

### P1. Department auto-subscription

**Given** a channel of the open kind whose department list contains Research, and Research has
members Bob and Carol, both with users.

**When** an employee Dan is created with department Research and a linked user.

**Then** Dan's user's contact is added to the channel's members, alongside Bob's and Carol's.

**And when** Carol's user is deactivated, **then** her contact is not added by a later
recomputation, because only active contacts are considered.

**And when** the channel is not of the open kind, **then** the constraint refuses with "For
*the channel name*, channel_type should be 'channel' to have the department
auto-subscription."

### P2. The authenticated-employees alias

**Given** an alias with the contact policy "Authenticated Employees".

| Sender address | Matches an employee's work email or an employee's user's electronic mail address | Accepted? |
|---|---|---|
| `nadia.farrell@northwind.example` | yes | yes |
| `Nadia.Farrell@Northwind.Example` | yes, case-insensitively | yes |
| `stranger@example.org` | no | no — rejected with the reason code `error_hr_employee_restricted` and the reason "restricted to employees" |

### P3. Equipment assignment

**Given** equipment "Laptop 42" with Used By = Employee and Assigned Employee = Ruben Alba,
whose user is Ruben.

**Then** the Assigned Department is empty, the Assignment Date is today, the Owner is Ruben's
user, and Ruben's user's contact is subscribed to the equipment's thread.

**And when** the Used By is changed to Department with Assigned Department = Research whose
manager is Alice, **then** Assigned Employee is cleared, the Owner becomes Alice's user, the
Assignment Date is set to today, Alice's user's contact is subscribed, and the change is
recorded under the "assigned" message subtype.

### P4. Freeing equipment at departure

**Given** Ruben with two equipment items.

**When** a departure is registered with "Free Equipment" ticked.

**Then** both items lose their Assigned Employee, and their Owner recomputes to the acting
user, because the assignment kind stays `employee` with no employee named.

---

## Part Q — Recognition badges

### Q1. Granting a badge

**Given** Lin Chen wants to grant the badge "Problem Solver" to Nadia Farrell.

**When** she opens the granting wizard, picks Nadia as the employee, and confirms.

**Then** the wizard's User field resolves to Nadia's user; a grant is created carrying the
badge, Nadia's user, Lin as the sender, Nadia as the employee, and the comment; and the grant
is sent, its notification carrying an action button labelled "View Your Badge" that opens
Nadia's public profile with the badges tab selected and this grant highlighted.

### Q2. Self-granting is refused

**When** Lin picks her own employee record.

**Then** the operation fails with "You can not send a badge to yourself."

### Q3. The employee must match the user

**When** a grant is created naming user Nadia and employee Marek.

**Then** the save is refused with "The selected employee does not correspond to the selected
user."

### Q4. Who may edit a grant

**Given** a grant created by Lin.

**Then** Lin may change and delete it; Pia Lund, a Human Resources Officer, may change and
delete it; Marek, an ordinary internal user who did not create it, may only read it.

### Q5. Badge counters

**Given** the badge "Problem Solver" granted three times, two of the grants naming an employee
and one naming only a user.

**Then** Granted Employees Count reads **2**.

**And** the employee whose grant named only their user still sees the badge in their Badges
collection, because that collection unions the grants naming the employee with the grants
naming the user and no employee.

---

## Part R — Reports and printing

### R1. Printing a badge

**Given** employees Nadia (badge identifier `041123456789`) and Marek (no badge identifier),
and the company has a logo.

**When** Olga prints badges for both.

**Then** both cards show the logo, the avatar, the name and the job position name; Nadia's card
additionally shows a bar code of `041123456789`; Marek's shows no bar code; both cards are
243 by 153 points, bordered, and neither is split across a page break; the file is named
"Badge - Nadia Farrell" when printed for her alone.

### R2. Printing a curriculum vitae

**Given** Nadia with resume lines of types Education and Training, plus one line with **no**
type.

| Wizard state | Sections rendered |
|---|---|
| "Show other sections" on | Education, Training, and an "Other" section containing the untyped line |
| "Show other sections" off | Education and Training only |

**And** the "show skills" switch is only offered when at least one selected employee has a
skill; the "show other sections" switch is only offered when at least one selected employee has
an untyped resume line.

### R3. The curriculum vitae route guards

| Caller | Requested employees | Outcome |
|---|---|---|
| Olga, a Human Resources Officer | any list | rendered |
| Lin Chen, an internal user | her own employee only | rendered |
| Lin Chen | any other employee, or a list containing one | not found |
| a portal or public visitor | any | not found |
| any caller | an employee list containing a character other than a digit or a comma | not found |
| any caller | an empty employee list | not found |

The downloaded file is named "Resume *the employee name*.pdf" for one employee and
"Resumes.pdf" for several.

### R4. The skill inventory report and the manager filter

**Given** Pia Lund is not a department manager and not an officer; and Alice manages Research.

| Reader | Rows visible in the skill inventory report |
|---|---|
| Olga, an officer | every row of her allowed companies |
| Alice, a department manager | her own row plus every row of Research and its descendant departments |
| Lin Chen, neither | her own row only |

### R5. Report values

**Given** three employees of Research holding a skill at levels with progress 25, 50 and 100.

**Then** the skill inventory report shows 0.25, 0.50 and 1.00, and the Research group shows the
average, (0.25 + 0.50 + 1.00) ÷ 3 = **0.5833…**, rendered as 58.33 per cent. Departments in the
pivot are named without their ancestor chain.

**And** the skill history report shows the raw percentages 25, 50 and 100 for the same records.

### R6. Certification report validity

**Given** a certification valid from 1 January 2026 to 31 December 2026, and the projection was
last rebuilt on 11 September 2026.

**Then** its Still Valid indicator reads true, because 31 December 2026 is on or after the
build date and 1 January 2026 is on or before it.

**And given** a certification that expired on 1 August 2026, **then** its indicator reads false.

---

## Part S — Multi-company

### S1. One person, two companies, two employees

**Given** user Sara Loft with employee Sara Loft (Northwind SA) and employee Sara Loft
(Southwind SA).

**Then**:

1. Both employees exist and the unique constraint is satisfied, because it is scoped per
   company.
2. Reading the user's Company Employee with Northwind SA as the acting company returns the
   Northwind employee; with Southwind SA, the Southwind one.
3. The user's Employee Count reads 2, archived employees included.
4. The delegated employee fields on the user resolve against the acting company's employee, so
   the same user shows different private telephone numbers in the two companies if they were
   entered separately.

### S2. Creating employees for two companies in one call

**When** four employees are created in one call, two for Northwind SA and two for Southwind SA,
in the order N1, S1, N2, S2.

**Then**:

1. Each employee's Employee Version carries the same company as its employee.
2. The returned record set is in the original order N1, S1, N2, S2, even though the creation
   was internally grouped by company.

### S3. Moving an employee between companies warns

**When** Olga changes an existing employee's Company.

**Then** a non-blocking warning is shown, titled "Warning", reading "To avoid multi company
issues (losing the access to your previous contracts, leaves, ...), you should create another
employee in the new company instead."

### S4. Version visibility across companies

**Given** Olga's allowed companies contain only Northwind SA, and an Employee Version belongs
to Southwind SA.

**Then**:

1. Pia Lund, an officer, cannot see that version.
2. Olga, an administrator, **can** see it, because the administrator rule is unrestricted and
   rules of different groups combine permissively.

### S5. Salary structure type visibility is global

**Given** a salary structure type whose country is France, and the reader's companies are all
Belgian.

**Then** **no** reader sees that structure type, not even an administrator, because the rule is
global.

---

## Part T — Contacts and deletion protection

### T1. Deleting a contact linked to one employee

**When** Olga deletes the contact that is Nadia's work contact.

**Then** the deletion is refused with "You cannot delete contact that are linked to an
employee, please archive them instead."

### T2. Deleting several contacts, some linked

**When** Olga deletes three contacts, two of which are work contacts of employees named "Nadia
Farrell" and "Marek Dolny".

**Then** the deletion is refused with:

> You cannot delete contact(s) linked to employee(s).
> Please archive them instead.
>
> Affected contact(s): Nadia Farrell, Marek Dolny

offered together with a link that opens those contacts.

### T3. A contact's address list gains an employee entry

**Given** contact P is the work contact of Nadia, whose private address is "12 Rue Neuve,
1000 Brussels, Belgium".

**Then** P's address list begins with an entry of kind "employee" carrying that private street,
city, postal code, state code and country code, followed by P's own addresses.

### T4. Work detail synchronisation only for a single-employee contact

**Given** contact P is the work contact of exactly one employee.

**When** the employee's Work Telephone is written.

**Then** P's telephone is written too, and conversely a change to P's telephone shows on the
employee.

**And given** P is the work contact of **two** employees, **then** neither direction
synchronises: writing the employee's Work Telephone does not touch P, and P's telephone does
not overwrite the employee's.

---

## Part U — Miscellaneous invariants

### U1. Coach follows the manager only when it was the manager

| Before | Manager written | Coach after |
|---|---|---|
| Manager Alice, Coach Alice | Erin | Erin |
| Manager Alice, Coach empty | Erin | Erin |
| Manager Alice, Coach Bob | Erin | **Bob** (unchanged) |
| Manager empty, Coach empty | empty | empty |

### U2. Job title follows the job position unless customised

| Before | Action | Job Title after | Job Title is Custom after |
|---|---|---|---|
| Job Position Developer, Job Title "Developer" | change Job Position to Architect | "Architect" | false |
| Job Position Developer, Job Title "Lead Developer" (customised) | change Job Position to Architect | "Architect" | false — changing the job position resets the custom flag first |
| Job Position Developer, Job Title "Developer" | write Job Title "Lead Developer" | "Lead Developer" | **true** |
| Job Position Developer, Job Title "Lead Developer" (custom) | write nothing | "Lead Developer" | true |

### U3. Legal name only fills when empty

| Before | Name written | Legal Name after |
|---|---|---|
| Legal Name empty | "Nadia Farrell" | "Nadia Farrell" |
| Legal Name "Nadia Anne Farrell" | "Nadia F." | "Nadia Anne Farrell" (unchanged) |

### U4. Work permit file name

| Employee Name | Work Permit Number | Work Permit File Name |
|---|---|---|
| "Nadia Farrell" | `WP-9981` | `Nadia_Farrell_work_permit_WP-9981` |
| "Nadia Farrell" | empty | `Nadia_Farrell_work_permit` |
| empty | `WP-9981` | `work_permit_WP-9981` |

### U5. Contract type code fills once

| Before | Name written | Code after |
|---|---|---|
| Code empty | "Permanent" | "Permanent" |
| Code "PERM" | "Permanent Contract" | "PERM" (unchanged) |

### U6. Employee tag colour

**When** three employee tags are created with no colour given.

**Then** each receives a whole number between 1 and 11 inclusive, drawn independently.

### U7. Version display name depends on the reading language

**Given** a version dated 1 July 2026 attached to an employee.

**Then** its display name is that date rendered in the reader's language using the medium date
pattern of that language's locale — so the same record shows different text to readers of
different languages, and the value is recomputed when the reading language changes.

### U8. Version revision signal

**Given** an employee with versions V1 and V2.

**Then** Version Revision reads the concatenation, comma-separated, of each version's
identifier followed by its last write moment, in the employee's version ordering.

**And when** any version is written, **then** the signal changes, which is what makes the
client reload the version timeline.

### U9. Deleting an employee deletes its resource

**When** Olga deletes an employee that nothing else restrict-references.

**Then** the employee is deleted first and its Resource is deleted immediately afterwards; its
versions, skill assertions, resume lines and home-working exceptions cascade away.

### U10. Import creates full employees

**When** twelve rows are imported through the employee import template.

**Then** twelve Resources, twelve Employees, twelve Employee Versions and up to twelve Contacts
are created, each with the defaults of scenario A1, and every constraint of this document is
enforced row by row.
