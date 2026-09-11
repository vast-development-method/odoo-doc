# Business rules of the Human Resources Core domain

Every validation, constraint, invariant, error message, permission check and edge-case
behaviour of the domain. Error messages are reproduced exactly as the system produces them,
with placeholders translated into words.

---

## 1. The invariants

These are the statements that must hold at every moment. Everything else in this file exists
to preserve one of them.

| # | Invariant |
|---|---|
| I1 | Every Employee has at least one Employee Version at all times. |
| I2 | Every Employee has exactly one Current Version at all times, unless it has no versions at all — which only happens transiently during creation. |
| I3 | No two **active** Employee Versions of the same employee share an Effective Date. |
| I4 | No two contract periods of the same employee overlap, not even by one day. |
| I5 | A contract end date never exists without a contract start date. |
| I6 | A contract start date is never later than its contract end date. |
| I7 | All the versions belonging to one contract period carry the same pair of contract dates. |
| I8 | A user is linked to at most one employee per company. |
| I9 | A badge identifier is unique across the whole database. |
| I10 | A field that is not declared on the Public Employee model is never returned to a user who lacks read access on the private Employee model — not through a read, not through a search, not through a view, not through a display name. |
| I11 | For a regular (non-certification) skill type, a holder has at most one **valid** assertion per skill at any moment. |
| I12 | A skill assertion's validity stop is never earlier than its validity start. |
| I13 | At most one Employee Home-Working Location exception exists per employee per date. |
| I14 | At most one Skill Level per Skill Type carries the default flag. |
| I15 | A Skill Type always contains at least one Skill and at least one Skill Level. |

---

## 2. The private/public field split

This is the security boundary of the domain. It is enforced on three independent axes, and
a reimplementation that omits any one of them leaks data.

### 2.1 Axis one — model-level access rights

| Model | Group | Read | Write | Create | Delete |
|---|---|---|---|---|---|
| Employee | Human Resources Officer | yes | yes | yes | yes |
| Employee | System Administrator | yes | no | no | no |
| Employee | any other internal user | **no** | no | no | no |
| Public Employee | any internal user | yes | no | no | no |

An ordinary internal user therefore has **no access rights whatsoever** on the private
Employee model. This is the primary defence: every operation on it would raise an access
error. The redirects of [section 2.3](#23-axis-three--the-six-redirects) exist so that the
user gets useful data instead of that error, not so that they get private data.

### 2.2 Axis two — field-level groups on the private model

Within the private model, individual fields carry a readability group, so that even a
Human Resources Officer's data set is further narrowed for an Administrator-only subset, and
so that a prefetch triggered by some other model never pulls a restricted column.

The design rule stated in the model's own documentation is: **any field available on the
private Employee but not on the Public Employee must carry the Human Resources Officer
group**, precisely so that prefetching — which loads every field a user is allowed to read —
cannot accidentally load private data for a user who has no business seeing it.

Four fields are exempted from that rule by a special-case filter rather than by a declared
group, because they are inherited from mixins and cannot carry a group at their point of
declaration: the activity's calendar meeting, the ratings, the website messages and the
text-message error flag. For any reader who is neither operating with elevated rights nor a
Human Resources Officer, those four are removed from the readable field set at access-check
time, producing exactly the same effect as a declared group.

### 2.3 Axis three — the six redirects

When the reading user cannot read the private Employee model, six operations silently
substitute the Public Employee:

| Operation | Behaviour without read access on the private model |
|---|---|
| **Search** | The search condition is rewritten and run against the Public Employee, then the resulting identifiers are browsed on the private model with elevated rights, so that the caller still receives a private-model recordset. This works only because the two models share identifiers. Two rewrites happen first: a condition on the stored Current Version pointer is rewritten to a condition on the identifier, because the public model has no such column; and a condition on the Current Version using the plain "any" or "not any" comparison — which cannot be rewritten safely — is refused with "You do not have access to this document." A malformed condition that the public model rejects is likewise converted into "You do not have access to this document." |
| **Search and fetch together** | The requested field list is first checked against the public model's field set; any field not present there aborts the call (see the message below). The remaining fields are searched and fetched on the public model and the values are copied into the private recordset's cache, so the caller reads public values through a private handle. The stored Current Version pointer is always stripped from the requested list. |
| **Fetch** | Same as above without the search. In addition, before copying, any requested field that is a mirror of a private employee field or a delegated version field is explicitly resolved on the public side, so that the cache is complete. |
| **View retrieval (one view)** | The public model's view of the same kind is returned instead. |
| **View retrieval (several views)** | Returning public data inside a private view would produce an inconsistent client, so the call is refused with a redirection offer instead (see the message below). |
| **Display name** | Computed from the matching Public Employee record. |

Two further redirects apply to links rather than to reads:

| Operation | Behaviour |
|---|---|
| **Form view resolution for a link** | For a user outside the Human Resources Officer group, the public employee form is returned instead of the private one. |
| **Form action resolution for a link** | For a user outside the Human Resources Officer group, the action's target model is switched to the Public Employee. |

### 2.4 The private-field check and its message

Before any redirected fetch, the requested field names are compared with the Public
Employee's field names. Any requested name absent from the public model aborts the call:

> The fields “*the comma-separated list of field names*”, which you are trying to read, are
> not available for employee public profiles.

The exact character used around the list is the typographic double quotation mark.

### 2.5 The definitive field lists

The boundary is defined operationally: **a field is public exactly when a field of that name
is declared on the Public Employee model.** The following two lists enumerate the result.

#### 2.5.1 Public — readable by every internal user

Identity and contact: Employee Name, Active, Created On, Company, Country Code, Work
Address, Work Contact, Work Telephone, Work Mobile, Work Email, Telephone, Electronic Mail,
Instant Messaging Status, Is a Portal or Public Account, User, User's Contact, Resource,
Time Zone, Colour Index.

Placement: Department, Member of My Department, Job Position, Job Title, Work Location, Work
Location Name, Work Location Kind, Working Schedule, Monday Location through Sunday
Location, Today's Location Name.

Hierarchy: Manager, Coach, Direct Subordinates, Subordinates, Is a Subordinate of Me, Direct
Subordinates Count, Indirect Subordinates Count, Department Colour, Is a Manager, Is Me.

Images: Image at all five sizes, Avatar at all five sizes.

Presence: Presence State, Presence Icon, Show Presence Icon, Last Activity Date, Last
Activity Time, Newly Hired, Message Sent Today, Network Address Seen Today, Manually Set
Present, Manual Presence Override Active, Stored Presence State.

Career and recognition: Resume Lines, Skills, Current Skills, Certifications, Show
Certification Page, Badges, Has Badges, Equipment Count.

The single exception that carries partial private data: **Public Date of Birth** — the day
and month of the date of birth when the employee has consented by ticking "Show to all
employees", and the literal text `hidden` otherwise.

#### 2.5.2 Private — Human Resources Officer only

Personal identity: Legal Name, Date of Birth, Show Date of Birth to All Employees, Place of
Birth, Country of Birth, Language, Private Telephone, Private Email, Identity Card Copy,
Driving Licence, Private Car Plate, Emergency Contact, Emergency Telephone, User is Active.

Carried on the version and therefore equally private: Nationality, National Identification
Number, Social Security Number, Passport Number, Passport Expiration Date, Gender, Private
Street, Private Street Second Line, Private City, Private State, Private Postal Code, Private
Country, Allowed States, Home-Work Distance, Home-Work Distance in Kilometres, Home-Work
Distance Unit, Marital Status, Spouse Legal Name, Spouse Date of Birth, Dependent Children,
Additional Note, Employee Kind, Job Title is Custom, Is Flexible, Is Fully Flexible, Human
Resources Responsible, Effective Date, Last Modified By, Last Modified On, Employee is
Active, Contract Template.

Identification and access: Badge Identifier, Personal Identification Number.

Work permit and education: Work Permit Number, Visa Number, Visa Expiration Date, Work Permit
Expiration Date, Work Permit Document, Work Permit Reminder Scheduled, Work Permit File Name,
Certificate Level, Field of Study, School.

Money: Bank Accounts, Primary Bank Account, Primary Account is Trusted, Has Several Bank
Accounts, Salary Distribution, Currency, Hourly Cost.

Structure: Tags, Properties, Versions, Number of Versions, Version Revision, Version pointer,
Current Effective Date, Exceptional Location Today, Distinct Skills, Employee Goals, Directly
Granted Badges, Assigned Equipment, Related Contacts Count.

Departure: Departure Reason, Additional Information, Departure Date.

Messaging: the main attachment, every follower field, every message field, every activity
field, plus the four special-cased mixin fields listed in
[section 2.2](#22-axis-two--field-level-groups-on-the-private-model).

Company context: Company Country and Company Country Code, which are additionally readable
by the System Administrator group.

#### 2.5.3 Human Resources Administrator only

A twelve-field subset is narrower still. Re-declared on the Employee with the Administrator
group, and declared with it on the Version:

Contract Start Date, Contract End Date, End of Trial Period, Wage, Contract Wage, Salary
Structure Type, Contract Type, Effective Start Date, Effective End Date, Is Current, Is Past,
Is Future, Is Under Contract.

In words: **a Human Resources Officer can manage people but cannot see pay or contract
dates; a Human Resources Administrator can.**

### 2.6 The manager-only tier

The Public Employee declares an extension point listing fields that are revealed only when
the reading user is somewhere in the chain of managers above the record. For each such
field: if the reader manages this person directly or indirectly, the value is copied from
the private employee with elevated rights; otherwise the field reads as empty — not as an
error. The shipped list is empty; capabilities populate it. This is a third tier between
"public" and "officer only" and it is evaluated per reader, so the same record shows
different values to different people.

### 2.7 Two deliberate widenings of employee visibility

The record rule on both the private and the public Employee is not simply "my companies".
It is:

```
company is among my allowed companies (or the employee has no company)
OR  this employee's manager is me
OR  this employee is my own manager
OR  this employee is me
```

The second and third clauses exist so that cross-company reporting lines remain navigable:
a person can always see their own manager, and a manager can always see their reports, even
across a company boundary.

### 2.8 The read-access escape hatch for relation editing

Assigning a link-to-many field whose target is the Employee requires read access on the
Employee. Ordinary internal users have none. A dedicated abstract behaviour therefore exists
that other models mix in: while a create or a write runs on such a model, a sentinel value is
placed in the reading context, and the Employee's access check returns "allowed" for the
**read** operation when — and only when — that exact sentinel object is present. Because the
sentinel is an object identity rather than a value, it cannot be forged from outside the
running process, and it grants nothing but read.

### 2.9 Bank account confidentiality

Two record rules on the Bank Account model implement the rule "ordinary users must not see
employee bank accounts":

| Group | Rule |
|---|---|
| Every internal user | Only bank accounts whose owning contact has **no** employees at all. |
| Human Resources Officer | Every bank account. |

Because record rules of different groups are combined permissively, an officer sees
everything and everyone else sees only non-employee accounts. On top of that, the display
name of an employee bank account is masked for non-officers by the rule in
[calculations, section 26](calculations.md#26-bank-account-display-name-masking) — a second
line of defence for the case where such an account reaches a non-officer through a
relation they are allowed to traverse.

---

## 3. Employee identity rules

### 3.1 Uniqueness

| Rule | Message |
|---|---|
| The Badge Identifier is unique across the whole table. | "The Badge ID must be unique, this one is already assigned to another employee." |
| The pair (User, Company) is unique. | "A user cannot be linked to multiple employees in the same company." |

### 3.2 Badge identifier format

Checked on every write:

- Allowed characters: the letters A to Z in either case and the digits 0 to 9 only. No
  accented characters, no spaces, no punctuation.
- Maximum length: eighteen characters.
- An empty badge identifier always passes.

Message on failure:

> The Badge ID must be alphanumeric without any accents and no longer than 18 characters.

### 3.3 Personal identification number format

Checked on every write. The value must consist of digits only; an empty value passes.

> The PIN must be a sequence of digits.

### 3.4 Work contact rules

- Creating work contacts in bulk for employees that already have one is refused with "Some
  employee already have a work contact".
- The Work Telephone and Work Email are synchronised with the work contact **only when the
  work contact is attached to at most one employee**. A contact shared by two employees is
  never overwritten by either of them, and neither reads from it.
- Assigning a user to an employee detaches the work contact from any *other* employee of the
  same company that has **no** user of its own and whose work contact is that user's contact.
  This prevents the unique (user, company) constraint from being circumvented through the
  shared contact.
- When the work contact is changed on an employee, its previous work contact is unsubscribed
  from the employee's discussion thread.
- When the work contact is changed and the employee has bank accounts, each of those bank
  accounts is moved to the new work contact — and, if the account permitted outgoing
  payments, that permission is **revoked**, because the account's owner changed and the trust
  decision must be retaken.

### 3.5 Deleting a contact linked to an employee

Refused. For a single contact:

> You cannot delete contact that are linked to an employee, please archive them instead.

For several contacts, with a link offered to open them:

> You cannot delete contact(s) linked to employee(s).
> Please archive them instead.
>
> Affected contact(s): *the comma-separated names*

### 3.6 Company change warning

Changing the Company of an existing employee is allowed but produces a non-blocking warning:

> **Warning**
>
> To avoid multi company issues (losing the access to your previous contracts, leaves, ...),
> you should create another employee in the new company instead.

---

## 4. Contract period rules

### 4.1 Start before end

Checked on every create and write touching the employee, the contract start date or the
contract end date, and evaluated with elevated rights so that an officer without wage
visibility still triggers it:

> Start date (*the contract start date*) must be earlier than contract end date (*the
> contract end date*).

A version with no contract start date, or with no employee, is skipped entirely.

### 4.2 No overlapping contract periods

The core rule of the domain. For each version being validated, with a contract start date,
an employee, and Active true:

1. Load every **other** version of the same employee that has a contract start date, grouped
   by (employee, contract start date to the day, contract end date to the day).
2. Let the version's own period be (its contract start date, its contract end date or the
   largest representable date).
3. For each existing period (start, end or the largest representable date):
   a. If the existing period is **exactly** the version's own period, mark it as an already
      existing period and continue — this is the normal case of several versions sharing one
      contract, and it is not an overlap.
   b. Otherwise, if the existing start is on or before the version's end **and** the
      version's start is on or before the existing end, the two periods overlap. Fail.
4. If the version's period was not found among the existing ones, add it to the working set
   so that a batch containing several new versions is checked against itself too.

The failure message is:

> *the employee's display name* already has a contract running during the selected period.
>
> Please either:
>
> - Change the start date so that it doesn't overlap with the existing contract, or
> - Create a new employee if this employee should have multiple active contracts.

Note that inactive versions are skipped by the loop (step 3 is only reached when the version
being validated is active), so archiving a version releases its period.

### 4.3 The check constraint

> The contract must have a start date.

Raised by the database whenever a contract end date exists and the contract start date does
not.

### 4.4 Closing before opening a new contract

An explicit guard exists for the flow "start a new contract for a person who already has an
open-ended one":

> Before creating a new contract, close the current one by setting an end date.

Raised when the version under examination has a contract start date and no contract end
date.

### 4.5 Creating a contract on a date already covered

A second guard, used before offering to create a contract at a chosen date:

> The employee is already in contract on *the date, rendered in the reader's language in the
> medium date format*. Please select a date outside existing contracts

### 4.6 Writing contract dates across several contracts at once

Writing a contract date onto a selection of versions whose contract start dates are not all
the same is refused:

> Cannot modify multiple versions contract dates with different contracts at once.

### 4.7 Departure date versus contract start

When a departure is registered:

> Departure date can't be earlier than the start date of current contract.

Raised when any selected employee's current version has a contract start date later than the
chosen departure date.

---

## 5. Employee Version record rules

| Rule | When checked | Message |
|---|---|---|
| An employee always keeps at least one version. | On delete. | "Employee *the employee name* must always have at least one active version." |
| All of an employee's versions cannot be unassigned at once. | On write of the Employee link. | "Cannot unassign all the active versions of an employee." |
| All of an employee's versions cannot be archived at once. | On write of Active to false. | "Cannot archive all the active versions of an employee." |
| Two active versions of one employee cannot share an effective date. | By a unique database index restricted to rows where Active is true and the Employee link is not empty. | "An employee cannot have multiple active versions sharing the same effective date." |

Three consequences worth stating explicitly:

- The unique index **excludes** archived versions and templates. Archiving a version frees
  its effective date for another version; reactivating a version can therefore fail.
- The "at least one version" guard is evaluated per employee across the whole deletion set,
  so deleting several versions of one employee at once is refused when they are all of them,
  and allowed when at least one remains.
- Creating a version whose Effective Date equals that of an existing active version does not
  raise the domain-level messages; it is stopped by the index.

---

## 6. What a person may change about themselves

A user editing their own preferences is granted, by two explicit lists, the right to read and
to write a set of fields delegated to their own employee — even though they have no rights
on the Employee model at all.

### 6.1 Readable about oneself but not writable

Active, Direct Subordinates, Company Employee, Related Employees, Is a Human Resources
Officer, Is a System Administrator, Employee's Working Hours, Work Contact, Bank Accounts.

### 6.2 Readable **and** writable about oneself

Additional Note, Private Street, Private Street Second Line, Private City, Private State,
Private Postal Code, Private Country, Private Telephone, Private Email, Badge Identifier,
Employee Tags, Display Name, Emergency Contact, Emergency Telephone, Employee's Bank
Accounts, Job Title, Home-Work Distance in Kilometres, Work Mobile, Personal Identification
Number, Visa Expiration Date, Work Email, Work Location, Work Telephone, and the seven
weekday home-working location fields.

### 6.3 Notification when a person edits their own personal data

Writing any field on a user that is delegated to the Employee triggers, **before** the write
is applied, a notification on each affected employee's thread. The notification is addressed
to the employee's Human Resources Responsible — and to nobody when there is none — and reads:

> Personal information update.
>
> The following fields were modified by *the employee's name*
>
> - *the label of each modified field, one per list item*
>
> *You are receiving this message because you are the HR Responsible of this employee.*

with the last line in italics.

### 6.4 Data pushed the other way, from user to employee

Writing certain fields on a user propagates to that user's employees in the current company.
The propagated set is: Name, Electronic Mail (which becomes the employee's Work Email), Image
and Time Zone — plus, when the home-working capability is present, the seven weekday
location fields.

The image case is handled specially: employees that already have an image and employees that
do not are written in two separate operations, so that an employee with its own photograph is
not silently overwritten by the user's.

### 6.5 The preferences form's field visibility

Because the preferences form must show fields protected by groups the user does not hold, the
form's definition is fetched with elevated rights **only** when the view being fetched is
exactly the preferences form. The elevation is deliberately narrowed to that one view so the
group mechanism is not disabled on the ordinary user form. In addition, the preferences form
is moved to the **end** of the list of views being fetched, so that the search view's field
set does not take precedence and omit the elevated fields.

---

## 7. Skill rules

### 7.1 Skill type composition

> The following skills type must contain at least one skill and one level: *the names, one
> per line*

Raised whenever a skill type is left with no skill or with no level.

### 7.2 Skill level progress range

> Progress should be a number between 0 and 100.

A database check.

### 7.3 Skill and level must belong to the chosen type

> The skill *the skill name* and skill type *the skill type name* don't match

> The skill level *the level name* is not valid for skill type: *the skill type name*

### 7.4 Validity window ordering

> The following skills have their valid stop date prior to their valid start date:
> • *the display name* from *the validity start* to *the validity stop*

with one bullet per offending record, concatenated onto the heading line.

### 7.5 Overlapping or duplicate assertions

> The following skills can't be created as they overlap or exactly match existing skills:
> • *the offending value sets* conflicts with the existing skill/certification *the stored
> record's display name* from *its validity start* to *its validity stop*

with one bullet per conflicting stored record.

The conflict definition differs by kind:

| Kind | Conflicts with |
|---|---|
| Regular skill, or certification where validity editing is not allowed | Any other assertion of the same holder and skill whose window overlaps, by the asymmetric test of [calculations, section 15.4](calculations.md#154-detecting-a-conflict). |
| Certification where validity editing is allowed | Only another assertion of the same holder, skill, level, validity start **and** validity stop. |

Validity editing is allowed for Employee Skill and **not** for Job Skill.

### 7.6 Deletion of a skill assertion

Genuine deletion is only performed when the assertion's Validity Start is on or after
yesterday, or when the assertion is already stopped on or before yesterday. Otherwise the
assertion is stopped at yesterday and kept. The rationale is stated in the model's own
documentation: *skills should not be deleted, so that the history of the record's skills is
preserved; skills should not be written to either, the previous one should be stopped and a
new one created*.

### 7.7 Single default level

Setting a Skill Level as the default clears the flag on every other level of the same skill
type. Enforced on create, on write, and through a client-side marker consumed by the skill
type's on-change so that the rule also holds in an unsaved form.

---

## 8. Department rules

| Rule | Message |
|---|---|
| The parent chain may not form a cycle. | "You cannot create recursive departments." |
| The complete-name search supports only a fixed operator set. | "Operation not Supported." |

Supported operators for the complete-name search: equals, not equals, contains, does not
contain, in, not in, pattern-match. The search is resolved by loading every department and
filtering in memory, because the complete name is not stored.

### 8.1 Manager change propagation

Writing a department's Manager re-points the department's members: every employee of that
department whose Manager is the department's **outgoing** manager, and which is not the
incoming manager itself, has its Manager set to the incoming manager. Employees who report to
someone else are left alone, so an intra-department reporting line survives a change of
department head.

The re-pointing happens **before** the new manager is stored, so the comparison uses the old
manager.

### 8.2 Company inheritance

A department with a parent that has a company takes the parent's company. The computation is
recursive and stored.

### 8.3 Menu visibility

- A Human Resources Officer never sees the plain "Employees" leaf menu.
- A user who is neither an officer nor the manager of any department never sees the
  department card menu.

---

## 9. Job position rules

| Rule | Message |
|---|---|
| Unique job name per department per company. | "The name of the job position must be unique per department in company!" |
| Non-negative recruitment target. | "The expected number of new employees must be positive." |

Editing the rich-text Job Description goes through a revision-divergence guard, so that two
people editing it at the same time do not silently lose one another's work; the guard is
applied only when exactly one job is being written.

---

## 10. Work location rules

Deleting a Work Location still used as a weekday default by any employee is refused:

> You cannot delete locations that are being used by your employees

When deletion is allowed, every single-date home-working exception referencing that location
is deleted first.

---

## 11. Home-working rules

| Rule | Message |
|---|---|
| One exception per employee per date. | "Only one default work location and one exceptional work location per day per employee." |

Behavioural rules:

- Setting a single date's location to the value the weekly plan already gives for that
  weekday **deletes** the exception rather than storing it.
- Setting a recurring location for a weekday deletes the exception on the date the user
  clicked **before** writing the new weekday default, so that the clicked day immediately
  shows the new default rather than the old exception.
- The recurring write is addressed to the employee's **user**, not directly to the employee,
  so it flows through the self-service writable-field list and works for a person editing
  their own plan.

---

## 12. Badge granting rules

| Rule | Message |
|---|---|
| The named employee must belong to the named user. | "The selected employee does not correspond to the selected user." |
| A person cannot grant a badge to themselves. | "You can not send a badge to yourself." |

Editing or deleting a badge grant requires being a Human Resources Officer or being the
person who granted it.

---

## 13. Departure rules

| Rule | Message |
|---|---|
| The departure date may not precede the current contract's start date. | "Departure date can't be earlier than the start date of current contract." |

Safety rule on archiving the linked user: a user is archived **only when every employee
linked to that user is in the departure selection**. If the user still has another active
employee outside the selection, it is left alone and the operator is told so:

> The following users have not been archived as they are still linked to another active
> employees: *the comma-separated user names*

and, for the users that were archived:

> The following users have been archived: *the comma-separated user names*

Both messages are delivered as client notifications, the first in the danger style and the
second in the success style.

---

## 14. User creation rules

### 14.1 Creating a user from an employee

Refused when the employee already has one:

> This employee already has an user.

### 14.2 Creating users in bulk from employees

The bulk operation classifies every selected employee and reports each class separately. The
classification, applied in order, is:

| Order | Condition | Class | Message style |
|---|---|---|---|
| 1 | The employee already has a user. | Skipped | warning: "User already exists for Those Employees *the names*" |
| 2 | The employee has no work electronic mail address. | Skipped | danger: "You need to set the work email address for *the names*" |
| 3 | The work electronic mail address does not normalise to a valid address. | Skipped | danger: "You need to set a valid work email address for *the names*" |
| 4 | A user already exists whose normalised address or login equals it. | Skipped | warning: "User already exists with the same email for Employees *the names*" |
| 5 | Two or more selected employees share the same normalised address. | Skipped | warning: "The following employees have the same work email address: *the names*" |
| 6 | None of the above. | Created | success: "Users *the names* creation successful" |

The created user carries: the employee link, the employee's name, the employee's work
telephone number as its telephone, the normalised work electronic mail address as its login,
and the employee's work contact as its contact.

The notifications are chained so that all applicable ones are shown in sequence, ending with
a soft reload when at least one user was created.

### 14.3 The confirmation prompt before bulk creation

Before the bulk creation runs, the operator is warned, because creating users may have a
commercial consequence:

> You're about to invite new users. *the number of selected employees* users will be created
> with the default user template's rights. Adding new users may increase your subscription
> cost. Do you wish to continue?

with a button labelled "Confirm".

### 14.4 Creating an employee from a user

Refused when the acting company is not one of the user's companies:

> You are not allowed to create an employee because the user does not have access rights for
> *the company name*

---

## 15. Access checks embedded in operations

| Operation | Check | Failure |
|---|---|---|
| Reading the first version date or first contract date of an employee | Elevated rights, or the Human Resources Officer group. | "Only HR users can access first version date on an employee." |
| Reading the internal career history of an employee | Read access on the matching Public Employee. | "You cannot access the resume of this employee." |
| Declaring an employee present or absent by hand | The Human Resources Administrator group. | "You don't have the right to do this. Please contact an Administrator." |
| Sending a text message to absent employees | The Human Resources Administrator group. | "You don't have the right to do this. Please contact an Administrator." |
| Logging a presence note on employees | The Human Resources Administrator group. | "You don't have the right to do this. Please contact an Administrator." |
| Creating a new version | Write access on the employee **and** write access on the version being copied. | The platform's generic access error. |
| Opening several views of the private Employee without read access | Read access on the private Employee. | A refusal offering a redirection: "You are not allowed to access \"Employee\" (hr.employee) records.\nWe can redirect you to the public employee list." with a button labelled "Employees profile". |

---

## 16. Alias rules

An email alias configured with the "Authenticated Employees" contact policy accepts an
incoming message only when the sender's normalised address matches, case-insensitively and
partially, either some employee's Work Email or some employee's user's electronic mail
address. Otherwise the message is rejected with the reason code
`error_hr_employee_restricted` and the human-readable reason "restricted to employees". The
policy's description in words is "addresses linked to registered employees".

---

## 17. Activity plan rules

| Rule | Message |
|---|---|
| A plan whose target entity is not the Employee may not name a department. | "Plan *the plan names* cannot use a department as it is used only for some HR plans." |
| A plan whose target entity is not the Employee may not contain a template with a coach, manager or employee responsible kind. | "Plan activities *the activity type names* cannot use coach, manager or employee responsible as it is used only for employee plans." |
| A template may not use a coach, manager or employee responsible kind inside a plan whose target entity is not the Employee. | "Those responsible types are limited to Employee plans." |
| A reporting loop in which nobody has a user blocks responsible resolution. | "Oops! It seems there is a problem with your team structure. We found a circular reporting loop and no one in that loop is linked to a user. Please double-check that everyone reports to the correct manager." |
| A coach without a user, and no user anywhere up the chain. | Warning, not error: "The user of *the employee name*'s coach is not set." — the acting user becomes the responsible. |
| A manager without a user, and no user anywhere up the chain. | Warning: "The manager of *the employee name* should be linked to a user." |
| An employee without a user, and no user anywhere up the chain. | Warning: "The employee *the employee name* should be linked to a user." |
| No coach at all. | Error: "Coach of employee *the employee name* is not set." |
| No manager at all. | Error: "Manager of employee *the employee name* is not set." |

---

## 18. Discussion channel rules

| Rule | Message |
|---|---|
| Only an open channel may carry department auto-subscription. | "For *the comma-separated channel names*, channel_type should be 'channel' to have the department auto-subscription." |

---

## 19. Bank account allocation rules

| Rule | Message |
|---|---|
| Every bank account of the employee must appear in the salary distribution before the wizard can open. | "Bank account *the account* not found within the salary distribution of the employee" |
| Percentage amounts must be numbers in the closed range 0 to 100. | "Each amount percentage must be a number between 0 and 100." |
| When any percentage entry exists, the percentages must sum to exactly 100, to within four decimal places. | "Total salary distribution on bank accounts must be exactly 100%." |
| The same check applied when saving the wizard. | "Total percentage allocation must equal 100%." |

---

## 20. Departure reason and structure type deletion

| Rule | Message |
|---|---|
| The three shipped departure reasons cannot be deleted. | "Default departure reasons cannot be deleted." |
| A departure reason still referenced by a version cannot be deleted. | The relation's restrict rule refuses the deletion. |

---

## 21. Concurrency and ordering considerations

This domain has no explicit record locking. The points where concurrent activity matters
are:

| Situation | Behaviour |
|---|---|
| Two officers create a version for the same employee on the same effective date | The unique index rejects the second one. |
| Two officers write different contract dates on versions of the same contract period | Each write propagates across the period; the last write wins, and the overlap check runs against the state as it stands at that moment. |
| The nightly current-version refresh runs while an officer is editing | The refresh only writes the pointer when it actually changes, so an unrelated edit is not disturbed; an edit that itself changes the pointer makes the nightly pass a no-operation. |
| Two people edit a job description simultaneously | The revision-divergence guard on the rich-text field detects the divergence rather than silently overwriting. |
| The advanced presence job runs while presence is being read | The read uses the company's Last Presence Computation stamp to decide whether the stored indicators are fresh; a stamp from a previous day is ignored. |
| A network address log row is written during a presence refresh | The row is created in its own separate transaction, so it survives even when the surrounding request is rolled back, and a duplicate is avoided by checking for an existing row for that user, address and day first. |
| An employee is created in several companies in one batch | The batch is split by company, each group is created with that company as the acting company, and the original ordering is restored afterwards. This matters because the version created underneath takes its company from the acting company. |

---

## 22. Edge cases

| Case | Behaviour |
|---|---|
| An employee with only future versions | The Current Version falls back to the earliest version. Every delegated read therefore shows values that are not yet supposed to apply. This is deliberate: the alternative would be an employee with no readable department, job or schedule at all. |
| An employee with no active versions | Version selection falls back to the full set, archived versions included, so a Current Version still exists. |
| A version in force with no contract | Perfectly legal. Is Under Contract is false; the person appears in the directory, has a department and a schedule, but no employment period. Work entry generation and payroll treat them as not employed. |
| A gap between a contract end and the next version's start | The version in force during the gap is still the older one, but Is Under Contract is false for those days. See the worked example in [calculations, section 4.3](calculations.md#43-worked-example). |
| A person who is their own manager, or a reporting cycle | The subordinate walk carries the set of already-visited nodes and excludes them, so the recursion terminates and nobody is counted as their own subordinate. The organisation-chart ancestor walk breaks on the same condition. The activity-plan responsible resolution detects it and reports the circular-loop error. |
| An employee whose user has several employees across companies | Permitted. The user's "Company Employee" resolves per acting company. The departure wizard refuses to archive such a user unless every one of its employees is in the departure selection. |
| An employee with no work contact | Created automatically at employee creation. When an employee somehow reaches the work-detail inverse without one, a work contact is created on the spot with elevated rights, carrying the work email, the work telephone, the name, the image and the company. |
| A contact shared as the work contact of two employees | Neither employee synchronises its work telephone or work email with it, in either direction. |
| An employee archived while being someone's manager or coach | Those links are emptied on the other employees. |
| An employee unarchived after a departure | The three departure fields are cleared; the contract end date written at departure is **not** cleared, so the contract remains closed until an officer reopens it. |
| A skill removed the same day it was added | Genuinely deleted, leaving no trace, because its validity start is not earlier than yesterday. |
| A certification with no expiry date | Treated as permanent: the certification reminder job skips it entirely. |
| A skill type deactivated | Its assertions disappear from the employee's skill collection, which is filtered on active skill types, but the records remain and the history reports still see them. |
| A work location deleted while used only by exceptions | Allowed; the exceptions are deleted and those days silently return to their weekday default. |
| An employee with no time zone anywhere | The time-zone resolution chain is: the working schedule's time zone, then the employee's own, then the company's default schedule's, then the reference frame. |
| An employee with no working schedule | Fully flexible. Interval computations use the employee's own time zone and treat the whole window as available minus leaves. |
| An all-day meeting spanning a day the company is closed | The meeting's interval is computed as empty and the attendee-availability check is skipped for it entirely, so nobody is marked unavailable. |
| The salary distribution left empty | No constraint applies; the primary account resolution then simply orders the accounts by "absent from the map", which sorts them all last, and picks the first by identifier order. |
