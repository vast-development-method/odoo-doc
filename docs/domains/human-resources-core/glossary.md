# Glossary of the Human Resources Core domain

Every term used anywhere in this folder, defined in full. Terms are listed alphabetically.
Where a term corresponds to a reproduced storage or transport name, that name is given in
code font after the definition.

---

## A

**Access right** — A permission granted to a security group on an entity, expressed as four
independent flags: read, write, create and delete. Access rights are checked before record
rules. A group with no access right on an entity cannot touch it at all, whatever the record
rules say.

**Activity** — A task scheduled against a record, carrying a kind, a summary, a deadline and a
responsible user. This domain raises activities automatically for expiring contracts,
expiring work permits and missing or expiring certifications.

**Activity plan** — A named, ordered set of activity templates that can be launched in one
action against one or several records. This domain ships an onboarding plan and an
offboarding plan for employees and adds department scoping to plans. (`mail.activity.plan`)

**Activity plan template** — One line of an activity plan: an activity kind, a summary, an
ordering and a rule saying who becomes responsible. This domain adds three rules specific to
employees: the coach, the manager and the employee themselves.
(`mail.activity.plan.template`)

**Administrator** — Short for Human Resources Administrator. See that entry.

**Advanced presence** — The optional capability that derives presence from the number of
messages a person authored today and from the network addresses they connected from today,
in addition to their messaging status.

**Archived** — Not active. An archived record is excluded from ordinary searches but is not
deleted. Archiving an employee archives its resource; archiving a version removes it from
version selection and from the unique-effective-date index.

**Assignment kind** — On an equipment item, whether it is used by an employee, by a
department, or by neither. (`equipment_assign_to`)

**Avatar** — The image shown for a person in cards and lists. Resolved from the employee's own
image, then from the linked user's avatar, then from a generated placeholder built from the
name.

---

## B

**Badge** — Two unrelated meanings in this domain, always disambiguated in the text:
(1) the printable identification card carrying a person's name, photograph, job position and
bar code; (2) a recognition badge, a token granted by one colleague to another.

**Badge identifier** — The alphanumeric value printed as a bar code on a person's
identification card and scanned at attendance kiosks. Unique across the database, at most
eighteen characters, letters and digits only. (`barcode`)

**Bank account** — An account into which a salary may be paid. Owned by a contact, restricted
to a company or to none, and carrying a permission to be used for outgoing payments.
(`res.partner.bank`)

**Base internal user** — Any authenticated non-share user of the platform, with no
human-resources group. Such a user reads the public employee directory and manages their own
delegated employee fields.

---

## C

**Calendar period** — One stretch of a requested time window during which a single working
schedule applies to one employee, expressed as a triple of a start moment, a stop moment and
a schedule. An employee's window is tiled by calendar periods, one per version that overlaps
it.

**Certificate level** — The employee's highest educational attainment, chosen among graduate,
bachelor, master, doctor and other. (`certificate`)

**Certification** — A skill assertion whose skill type carries the certification flag.
Certifications differ from regular skills in three ways: several may coexist for the same
skill with different validity windows; they may be deleted freely; and a lapsed one remains
visible when no valid one exists.

**Certification type** — A skill type flagged as holding certifications rather than ordinary
proficiency levels. Its display name carries a medal character. (`is_certification`)

**Coach** — An employee designated as another employee's coach. The coach has no rights or
duties by default; the role exists so that activity plans can address them. The coach follows
the manager automatically when it was previously equal to the old manager or was empty.
(`coach_id`)

**Company** — The legal entity an employee belongs to. Required on every employee. A person
employed by two companies of the same database is two employee records.

**Complete name** — A department's name prefixed by the chain of its ancestors' names,
separated by " / ". Not stored; searched by loading every department and filtering in
memory. (`complete_name`)

**Contract** — A continuous period of employment, identified by the pair (contract start date,
contract end date). A contract is normally described by several employee versions, all
carrying that same pair. This domain never stores a separate contract record.

**Contract period** — Synonym for contract in this folder, used when the emphasis is on the
pair of dates rather than on the employment relationship.

**Contract template** — An employee version with **no** employee link: a reusable set of
employment terms, stored under a name, that can be applied to an employee. Only seven of its
fields are copied when it is applied. (`contract_template_id` points at one)

**Contract type** — A classification of employment agreements — permanent, temporary,
seasonal, intern, and so on — optionally restricted to a country. (`hr.contract.type`)

**Contract wage** — The wage figure a payroll capability treats as contractual. In this domain
it is simply the wage; the indirection exists so payroll can substitute another field.
(`contract_wage`)

**Current skills** — The subset of a holder's skill assertions still valid today, with the
special rule that a certification skill with no valid assertion contributes its most recently
expired one. (`current_employee_skill_ids`)

**Current version** — The stored pointer to the employee version in force today: the version
with the greatest effective date not in the future, or the employee's first version when they
all lie in the future. (`current_version_id`)

**Curriculum vitae** — The printable document assembling a person's resume lines, grouped by
resume line type, optionally with their skills and contact details, and interleaved with the
internal career history derived from their versions.

---

## D

**Delegation** — The mechanism by which the employee reads and writes most of its fields
through a pointed-at employee version. Reading a delegated field returns the version's value;
writing one writes the version; creating an employee creates its first version. Also called
the inheritance-by-pointer relationship.

**Department** — A node of the organisational tree, with a parent, children, a manager,
members, jobs and activity plans. (`hr.department`)

**Department manager** — A person who is the Manager of at least one department. Not a
security group; recognised dynamically, and used to widen menu visibility and report
visibility.

**Departure** — The recorded end of a person's employment: a reason, a description and a date
written onto the current version, optionally accompanied by archiving the employee, closing
the contract, freeing the equipment and archiving the linked user.

**Departure reason** — A named reason a person left, optionally restricted to a country. Three
ship with the platform and cannot be deleted. (`hr.departure.reason`)

**Derived value** — A value computed rather than stored, or computed **and** stored. Where the
distinction matters, this folder says "computed, not stored" or "computed and stored".

**Direct subordinate** — An employee whose Manager is this employee. (`child_ids`)

**Distinct skills** — The set of skills referenced by any of an employee's skill assertions,
regardless of validity. (`skill_ids`)

---

## E

**Effective date** — The date from which an employee version's terms apply. Required; unique
per employee among active versions. (`date_version`)

**Effective end date** — A version's derived last day: the earlier of "the day before the next
version's effective date" and the contract end date, whichever of the two exists. Empty when
neither exists. (`date_end`)

**Effective start date** — A version's derived first day: the later of its effective date and
its contract start date, or simply its effective date when there is no contract start date.
(`date_start`)

**Elevated rights** — Running an operation as if by the system itself, bypassing access rights
and record rules. Used deliberately in this domain to read a photograph, to read presence
data, to count employees for a department, to create a work contact for an officer without
contact rights, and to copy a version whose fields the acting user cannot read.

**Employee** — The private, complete record of one person employed by one company. Delegates
its employment terms to an employee version. (`hr.employee`, table `hr_employee`)

**Employee home-working location** — An exception overriding, for one single date, the weekday
default of the weekly home-working plan. (`hr.employee.location`, table
`hr_employee_location`)

**Employee kind** — A classification of the working relationship: employee, worker, student,
trainee, contractor or freelancer. (`employee_type`)

**Employee skill** — A dated assertion that one employee holds one skill at one level over a
validity period. (`hr.employee.skill`)

**Employee tag** — A free colour-coded label attachable to employees. (`hr.employee.category`)

**Employee version** — A dated snapshot of one employee's employment terms, or, when detached
from any employee, a reusable contract template. (`hr.version`, table `hr_version`)

**Equipment** — A physical item assigned to an employee or a department, with an assignment
date and an owner. (`maintenance.equipment`)

**Evidence** — In the advanced presence rule, the disjunction of three stored indicators:
message sent today, network address seen today, manually set present.

**Expired** — A skill assertion whose validity stop lies in the past. Kept for history, not
deleted.

---

## F

**Fixed-term contract** — A contract with both a start date and an end date.

**Flexible** — A version whose working schedule is marked as flexible-hours, or that has no
working schedule at all. (`is_flexible`)

**Fully flexible** — A version with **no** working schedule. Interval computations for such a
version use the employee's own time zone and treat the whole window as available minus leaves.
(`is_fully_flexible`)

---

## G

**Gap removal** — The rule that cuts a person's version history at the point where they were
away for four days or more, so that the "joined on" date reflects the current spell of
employment rather than a previous one.

**Granted recognition badge** — One instance of a badge given by one person to another, naming
the recipient's user and, optionally, their employee. (`gamification.badge.user`)

---

## H

**Home-working plan** — The seven weekday fields on the employee giving the default work
location for each day of the week. Deviations from it are home-working exceptions.

**Home-to-work distance** — The distance between a person's private address and their
workplace, stored twice: in the unit the user chose, and converted to kilometres for searching
and aggregation. (`distance_home_work`, `km_home_work`)

**Hourly cost** — The internal cost of one hour of a person's time, expressed in the company
currency, stored on the employee and defaulting to zero. Independent of the wage. Consumed by
timesheets and project profitability. (`hourly_cost`)

**Human Resources Administrator** — The higher of the two security groups. Implies the officer
group and adds: wage and contract dates, contract types, salary structure types, work
locations, employee activity plans, unrestricted version visibility across companies, and
manual presence declarations. (`hr.group_hr_manager`)

**Human Resources Officer** — The lower of the two security groups. Full management of
employees, versions, departments, jobs, tags, departure reasons, resources and working
schedules; sees every private employee field except the twelve administrator-only ones.
(`hr.group_hr_user`)

**Human Resources Responsible** — The user responsible for validating a person's contracts.
Required on every version, defaulting to the acting user. Receives the expiry reminders and
the personal-information change notifications. (`hr_responsible_id`)

---

## I

**Identification number, national** — The number issued by the government for official records
and statutory compliance. (`identification_id`)

**In force** — Said of the employee version whose effective period contains a given date. See
also *version in force*.

**Indirect subordinate** — An employee in the transitive closure of the direct-subordinate
relation, excluding the direct subordinates themselves when the distinction is made.

**Instant messaging status** — The presence indicator maintained by the messaging layer:
online, away, busy or offline. Decorated by the home-working capability with today's work
location kind, giving values such as `home_online`. (`im_status`)

**Internal career history** — The sequence of job titles a person held internally, derived
from their versions rather than from hand-entered resume lines, with consecutive versions
sharing a title collapsed into one entry.

**Internal user** — See *base internal user*.

**Interval** — A pair of moments with a payload, used throughout the working-time computations.
Interval sets support union, intersection and difference.

---

## J

**Job position** — A named position within a company and optionally a department, carrying a
description, requirements, a recruitment target and derived headcount numbers. (`hr.job`)

**Job skill** — A skill, at a level, expected of holders of a job position. (`hr.job.skill`)

**Job title** — The free text describing a person's role, defaulting to their job position's
name and marked as customised as soon as it is written by hand. (`job_title`)

---

## L

**Last activity** — The date, and on the same day also the clock time, of a person's last
recorded messaging presence, expressed in their time zone.

**Last presence computation** — The moment the advanced presence job last ran for a company.
Read by the presence rule to decide whether the stored evidence is fresh.
(`hr_presence_last_compute_date`)

**Legal name** — The person's name as it appears on official documents, defaulting to the
employee name and never overwritten once set. (`legal_name`)

**Level** — See *skill level*.

---

## M

**Manager** — An employee designated as another employee's manager. Forms the reporting tree
that the organisation chart renders. Emptied automatically on every employee that pointed at
an archived employee. (`parent_id`)

**Manager-only field** — A field of the public employee revealed only to the reading user's
managers. The shipped list is empty; capabilities populate it.

**Marital status** — Single, married, legal cohabitant, widower or divorced. Required, default
single. (`marital`)

**Master department** — The root of a department's tree, derived from the hierarchy path.
(`master_department_id`)

**Member of my department** — A derived indicator, evaluated per reading user, that is true
when a record's department is the reader's own department or one of its descendants.
(`member_of_department`)

---

## N

**Network address** — An internet protocol address a user connected from. Logged at most once
per user per address per day and consulted by the advanced presence rule. (`ip` on
`res.users.log`)

**Newly hired** — A derived indicator, true when the employee's new-hire reference value — the
creation timestamp by default — is later than ninety days before now. (`newly_hired`)

**Normalised wage** — An hourly equivalent of a monthly wage: the wage times twelve, divided by
fifty-two, divided by the weekly hours; or simply the wage when the version is fully flexible.

---

## O

**Off-hours** — The presence state meaning "not expected to be working right now".
(`out_of_working_hour`)

**Officer** — Short for Human Resources Officer. See that entry.

**Onboarding plan** — The shipped activity plan launched for a new employee: setting up
materials, planning training, and the training itself.

**Open-ended contract** — A contract with a start date and no end date.

**Organisation chart** — The rendering of the reporting tree around one employee: their
ancestors up to five levels, themselves, and their direct subordinates.

**Overlap** — Two contract periods overlap when the first starts on or before the second ends
**and** the second starts on or before the first ends. Two skill assertions overlap when their
validity windows intersect under the asymmetric test documented in the calculations file.

---

## P

**Passive field** — A field of a skill assertion that must survive the create-instead-of-write
rewriting without itself triggering it. The shipped list is empty; capabilities add to it.

**Personal identification number** — A digits-only value used to confirm a check-in or
check-out at an attendance kiosk and to change the cashier at a point of sale. (`pin`)

**Presence icon** — The derived indicator driving the badge shown on a person's card. Its value
is the literal text `presence_` followed either by the presence state or, when the
home-working capability is active and a location is known for today, by that location's kind.
(`hr_icon_display`)

**Presence state** — Whether a person is considered present, absent, off-hours or archived.
Computed, never stored on the private model except as a display copy.
(`hr_presence_state`)

**Primary bank account** — The employee bank account with the lowest order number in the salary
distribution; accounts absent from the distribution sort last.
(`primary_bank_account_id`)

**Privilege** — A grouping of security groups shown as one choice in the user form. This domain
declares one, named Employees.

**Properties** — Free extra fields whose definition lives on another record — on the company for
employee properties, on the resume line type for resume line properties. Stored as a keyed
map.

**Public date of birth** — The day and month of a person's date of birth, published to every
internal user **only** when that person has ticked "Show to all employees"; otherwise the
literal text `hidden`. The only personal datum that crosses the privacy boundary.
(`birthday_public_display_string`)

**Public employee** — The read-only projection over the employee joined to its current version,
exposing only the fields every internal user may see. A database view, not a table, whose row
identifiers coincide with the employee's. (`hr.employee.public`)

---

## R

**Record rule** — A filter applied per group that narrows which records of an entity a user may
act on. Rules of different groups combine permissively: a user who holds two groups sees the
union. A **global** rule applies to every group and can never be widened.

**Recognition badge** — A token of appreciation granted by one colleague to another, with a
comment. (`gamification.badge`)

**Recruitment target** — The number of new employees expected to be recruited for a job
position. Must be zero or greater; defaults to one. (`no_of_recruitment`)

**Regular skill** — A skill assertion whose skill type is **not** flagged as a certification
type. At most one such assertion may be valid per holder per skill at any moment.

**Resource** — The abstract schedulable entity every employee owns, carrying the name, company,
working schedule, time zone and active flag, and used by every scheduling computation in the
platform. (`resource.resource`)

**Responsible kind** — On an activity plan template, the rule that turns the plan into an
assignment: a fixed user, or — for employee plans only — the coach, the manager or the employee
themselves. (`responsible_type`)

**Resume line** — One dated entry of a person's curriculum vitae: a past job, a degree, an
internal certification or a course. (`hr.resume.line`)

**Resume line type** — A classification of resume lines with an ordering, a course flag and its
own free-extra-field definition. (`hr.resume.line.type`)

**Rewriting discipline** — The rule that every create, write and delete instruction addressed
to a skill collection is transformed before it reaches storage, so that history is preserved:
deletes become expiries, creates expire their predecessor, and writes that touch an identifying
field become an expiry plus a create.

---

## S

**Salary cost factor** — The constant twelve: the number of wage payments per year assumed when
annualising a monthly wage.

**Salary distribution** — The keyed map saying how a person's pay is split across their bank
accounts, each entry carrying an amount, a flag saying whether the amount is a percentage, and
an order number. (`salary_distribution`)

**Salary structure type** — A classification of pay rules for a country, carrying a default
working schedule. Referenced by every version; the hook by which payroll capabilities attach
country-specific rule sets. (`hr.payroll.structure.type`)

**Security group** — A named set of access rights and record rules that users are members of.
This domain declares two.

**Self-readable field** — A field of the user entity a person may read about themselves even
though they have no rights on the underlying employee.

**Self-writable field** — A field of the user entity a person may **write** about themselves,
which lands on their own employee.

**Share user** — A user who is not internal: a portal account or a public visitor. Excluded
from every human-resources selection list.

**Skill** — A single named competence inside a skill type. (`hr.skill`)

**Skill assertion** — A dated statement that a holder — an employee or a job position — holds a
skill at a level over a validity window. Generic term covering both employee skills and job
skills.

**Skill level** — A named proficiency step inside a skill type, carrying a progress percentage
between zero and one hundred and, for at most one level per type, a default flag.
(`hr.skill.level`)

**Skill type** — A family of competences owning both the skills inside it and the levels they
are graded on, optionally flagged as a certification type. (`hr.skill.type`)

**Sentinel** — The object identity placed in the reading context to grant read access on the
employee for the duration of a relation-editing operation. Cannot be forged from outside the
running process.

**Subordinate** — Any employee in the transitive closure of the direct-subordinate relation,
with a cycle guard that excludes nodes already visited on the current path.

---

## T

**Tag** — See *employee tag*.

**Termination mode** — The calling context in which the departure registration wizard also
archives the selected employees and their eligible users. Outside it, the wizard records the
departure without archiving anything.

**Time zone** — The zone in which a person's working hours are interpreted. Resolved in order:
the working schedule's, then the employee's own, then the company's default schedule's, then
the reference frame. (`tz`)

**Trial period end** — The date a probation period ends. Informational in this domain; no
behaviour is attached to it. (`trial_date_end`)

**Trusted bank account** — A bank account whose permission to be used for outgoing payments has
been granted. Revoked automatically when the account's owner changes.

---

## U

**Unusual day** — A day on which a person is not expected to work, used to shade planning and
calendar views. Derived per version from that version's working schedule, with the days not
covered by any version marked unusual wholesale.

**User** — A login account. At most one employee per company may be linked to a given user.
(`res.users`)

---

## V

**Validity window** — The pair (validity start, validity stop) of a skill assertion. An empty
validity stop means "still valid".

**Version** — Short for employee version. See that entry.

**Version in force** — The employee version selected for a given date by the algorithm that
takes the greatest effective date not later than the date, among the employee's active
versions — or among all its versions when none is active — falling back to the earliest version
when none qualifies.

**Version pointer** — The non-stored link on the employee designating the version its delegated
fields read through: the version named in the reading context when it belongs to this employee,
and otherwise the current version. (`version_id`)

**Version revision** — A derived text concatenating each version's identifier and last write
moment, used by the client as a cheap signal that the version timeline must be reloaded.
(`version_revision`)

---

## W

**Wage** — The person's monthly gross wage, expressed in the company currency, visible only to
the Human Resources Administrator group, and aggregated by averaging when grouped. (`wage`)

**Weekday default** — One of the seven work-location fields on the employee giving the usual
location for that day of the week.

**Work contact** — The contact record representing an employee for business purposes: it
carries the work electronic mail address, the work telephone number and the photograph, and it
receives notifications addressed to the employee. (`work_contact_id`)

**Work location** — A named place of work attached to a work address and typed as home, office
or other. (`hr.work.location`)

**Work permit** — The authorisation to work in a country, recorded as a number, an expiration
date and a scanned document, with an automatic reminder before expiry.

**Working schedule** — The definition of a person's expected working hours: attendance lines
per weekday, a time zone, weekly hours, and flags for flexible hours and duration-based
counting. Owned by the working-time domain and referenced by every employee version.
(`resource.calendar`)

**Working schedule leave** — A period of absence attached to a schedule, optionally to a
specific resource. This domain assigns such a leave to the schedule of the employee's current
version when the leave falls inside that version's effective period.
(`resource.calendar.leaves`)
