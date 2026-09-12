# Recruitment

## 1. Purpose of this domain

This domain is the hiring pipeline of the platform. It holds the openings the organisation
wants to fill, the people who present themselves for those openings, the staged evaluation
of each of those people, the written interview questionnaires they answer, the skills they
claim, the reasons they are turned down, the reusable pool in which promising people are
kept for later, and the final act of turning an accepted candidate into an employee record
in the people register.

The domain is deliberately built around three record kinds and one boundary:

1. **Job Position** — an opening with a hiring target, a recruiter, a department, a
   location, a set of interviewers, an optional written interview questionnaire, an
   optional list of required skills, an electronic mail alias, and an optional public page
   on the organisation's website.
2. **Application** — one person applying to at most one Job Position. The application
   carries the person's contact data, the evaluation stage, the salary discussion, the
   sourcing attribution, the attachments (curriculum vitae, cover letters), the skills, the
   interview answers, and the outcome. An application with no Job Position is a
   *spontaneous application*; an application that belongs to one or more Talent Pools is a
   *talent*, the canonical copy of a person kept for future openings.
3. **Recruitment Stage** — the ordered columns of the pipeline, one of which may be marked
   as the hired stage. Moving an application into the hired stage is the single event that
   records a hire, sets the hire date, and decrements the Job Position's remaining target.

The boundary is the **interviewer restriction**. The platform has three recruitment
privilege levels (Interviewer, Officer, Administrator) plus two non-recruitment audiences
(the public visitor on the website, and the ordinary internal user). The Interviewer level
is a deliberately narrow, record-scoped grant: an interviewer sees only the applications of
the job positions they are attached to, or the individual applications they are personally
attached to; the interviewer cannot create or delete applications; and the salary fields are
removed from the interviewer's form entirely. This is a security boundary, is enforced in
four independent layers (model access rights, record rules, field-level group restrictions,
and a guard method on the hire action), and is documented rule by rule in
[Business Rules, chapter 3](business-rules.md#3-the-interviewer-restriction-a-security-boundary)
and [Configuration, chapter 6](configuration.md#6-access-rights-matrix).

Two further design decisions dominate the domain and are documented exhaustively:

- **Duplicate detection is by three comparison keys, not by identity.** Two applications
  are considered to be the same person when their normalised electronic mail address, or
  their sanitised telephone number, or their professional network profile address are
  equal. The platform never merges them silently; it counts them, offers to refuse them
  together, and warns the public applicant before they submit. The full algorithm is in
  [Calculations, chapter 4](calculations.md#4-duplicate-and-similar-application-detection).
- **The pipeline has no explicit state column.** The lifecycle of an application is
  derived, at read time, from three stored facts: whether the record is active, whether a
  refusal reason is set, and whether the hire date is set. The derived value is exposed as
  the application status, and the search on that derived value is translated back into
  conditions on the three stored facts. This is specified in
  [State Machines, chapter 2](state-machines.md#2-application-status-the-derived-lifecycle).

## 2. Capabilities covered

| Capability | Where documented |
|---|---|
| Job positions with recruitment target, recruiter, interviewers, location, industry, employment type and custom properties | [entities.md](entities.md), [configuration.md](configuration.md) |
| Counting applications per job position: total, all, open, new, old, hired | [calculations.md](calculations.md) |
| Recruitment stages, their ordering, their job restriction, their folding, the hired flag, the staleness threshold and the kanban legends | [entities.md](entities.md), [state-machines.md](state-machines.md) |
| Applications: contact data, contact record synchronisation, evaluation priority, degree, availability, tags, notes, custom properties | [entities.md](entities.md), [business-rules.md](business-rules.md) |
| The derived application status and the stage machine | [state-machines.md](state-machines.md) |
| The kanban readiness state and its four legends | [state-machines.md](state-machines.md) |
| Duplicate detection by electronic mail address, telephone number and professional network profile | [calculations.md](calculations.md), [business-rules.md](business-rules.md) |
| Talent pools, talents, the canonical pool application, and the two-way field propagation | [entities.md](entities.md), [workflows.md](workflows.md) |
| Refusal with a reason, optional refusal message, and cascading refusal of duplicates | [workflows.md](workflows.md), [business-rules.md](business-rules.md) |
| Hiring: hired stage, hire date, target decrement, employee creation and the complete field mapping | [workflows.md](workflows.md), [calculations.md](calculations.md) |
| Applications arriving by electronic mail through a job alias, including job-board senders and name extraction | [workflows.md](workflows.md), [interfaces.md](interfaces.md) |
| Sourcing attribution: campaign, medium, source, per-source aliases and per-source tracking addresses | [entities.md](entities.md), [interfaces.md](interfaces.md) |
| The public job list, the public job page, the public application form, and the protections on them | [interfaces.md](interfaces.md), [business-rules.md](business-rules.md) |
| Written interview questionnaires: assignment to a job, invitation, answering, reading answers, printing | [workflows.md](workflows.md), [interfaces.md](interfaces.md) |
| Skills on applications and on job positions, the match score, and skill transfer at hire | [calculations.md](calculations.md), [workflows.md](workflows.md) |
| Text-message sending to applicants | [interfaces.md](interfaces.md) |
| Meetings and activities scheduled on an application | [workflows.md](workflows.md), [interfaces.md](interfaces.md) |
| Staleness ("rotting") of applications per stage | [calculations.md](calculations.md) |
| Stage-change messages from templates, refusal messages, acknowledgement messages, interviewer notifications | [interfaces.md](interfaces.md), [workflows.md](workflows.md) |
| The recruitment analysis report and the per-department and per-job filtered reports | [interfaces.md](interfaces.md) |
| Settings, groups, access rights, record rules, default records | [configuration.md](configuration.md) |
| Every validation, error message and permission check | [business-rules.md](business-rules.md) |
| Numbered end-to-end acceptance scenarios with concrete numbers | [acceptance-criteria.md](acceptance-criteria.md) |

## 3. What is in scope and what is not

**In scope.**

- The Application record in full: contact details, attribution, recruiter and interviewers,
  stage, readiness colour, evaluation, salaries, degree, availability, tags, private notes,
  custom properties, attachments, the derived status, the duplicate counter, and the complete
  lifecycle from creation through stage progression, refusal, archival, restoration and
  hiring to deletion.
- Recruitment Stages: ordering, folding, the hired flag, the per-stage message template, the
  per-stage requirements note, the four readiness labels, the staleness threshold, and the
  rule that a stage is either shared by every position or restricted to named ones.
- Refusal Reasons and the refusal operation, including the optional message, the per-reason
  template, scheduled delivery, attachments and the bulk refusal of duplicates.
- Application Tags, Degrees with their score, Job Boards with their extraction pattern, and
  Recruitment Sources with their tracking address and their dedicated inbound address.
- The talent pool: pools, talents, the two-way propagation between a pooled Application and
  its talent, and the creation of new Applications out of a talent.
- Skills on Applications and on positions, the match score in both directions, and the
  transfer of skills at hire.
- Written interviews attached to a position: invitation, answering, the completion note and
  the printable answers.
- Text messages sent in bulk to candidates.
- The public job list, the public detail page, the public application form with its file
  upload and its live duplicate warning, the thank-you page, the publication state of a
  position, and the guided chat script that helps a visitor find an opening.
- Recruitment reporting: the analysis screens, the stage funnel, the groupings by source,
  medium, campaign, recruiter, department and position, the time spent per stage, and the
  new-employee indicator of the periodic digest.
- The three privileges, the record rules that narrow an interviewer to the Applications they
  interview, and the automatic granting and revocation of the Interviewer privilege.

**Out of scope, and owned by the domain named against each entry.**

| Not owned here | Owner |
|---|---|
| The Employee, the Employee Version, the Department, the Work Location, the Employment Type and the skills catalogue | [Human Resources Core](../human-resources-core/README.md) |
| The Job Position record itself — its name, description, remaining target, department and company. This domain adds fields to it | [Human Resources Core](../human-resources-core/README.md) |
| Threads, followers, message subtypes, activities, activity plans, message templates, inbound aliases and the inbound message gateway | [Messaging and Activities](../messaging-and-activities/README.md) |
| Contacts, and the normalisation of electronic mail addresses and telephone numbers | [Contacts and Organizations](../contacts-and-organizations/README.md) |
| Users, privileges, record rules and the evaluation of access rights | [Identity and Access](../identity-and-access/README.md) |
| Meetings and calendars | [Calendar and Scheduling](../calendar-and-scheduling/README.md) |
| Questionnaires, their questions, their answers, their scoring and their answer sets | [Learning, Surveys and Gamification](../learning-surveys-and-gamification/README.md) |
| Campaigns, tracking sources and tracking media | [Marketing and Mass Mailing](../marketing-and-mass-mailing/README.md) |
| Web pages, page publication, the public form endpoint, site search and the visual editor | [Website and Storefront](../website-and-storefront/README.md) |
| The periodic digest message itself | [Spreadsheets and Dashboards](../spreadsheets-and-dashboards/README.md) |
| Attachment storage and the indexing of attachment content | [Platform Foundation](../platform-foundation/README.md) |
| Everything a ledger entry needs. This domain posts none; see [accounting-effects.md](accounting-effects.md) | [General Ledger](../general-ledger/README.md) |

## 4. The people who use this domain

| Role | Privilege held | What the role may do here |
|---|---|---|
| Candidate | none: a public visitor, or the sender of a message | Submit the public application form; answer a written interview; reply to messages; be counted by the live duplicate warning |
| Portal user | the portal privilege | Read the published Job Positions, exactly as a public visitor does |
| Internal user | none of the recruitment privileges | Read Job Positions and Recruitment Sources; create, read and update Application Tags; see the preview panel of the curriculum vitae on an application form. Sees no Application |
| Interviewer | Interviewer | Read and update **only** the Applications of the positions they are attached to and the Applications they are individually attached to; refuse them; schedule meetings for them; send and read their written interviews. Never create or delete an Application, never see a salary field, never create an Employee |
| Officer | Officer: Manage all applicants | Create, read, update and delete every Application, Job Position and Talent Pool; see and edit the salary fields; refuse; record a hire; manage pools, sources, reasons, degrees and tags |
| Administrator | Administrator | Everything an Officer may do, plus configure the stages, the job boards, the recruitment activity plans and the written interview questionnaires |
| Human Resources Officer | Officer: Manage all employees, of [Human Resources Core](../human-resources-core/README.md) | Read Job Positions and perform the create-employee operation on an Application that carries a hire date |
| Platform administrator | the platform administration privilege | Open the settings screen and install or remove the companions |
| The system | the inbound gateway, the derived-value engine and the notification engine | Turn inbound messages into Applications, recompute derived values, post automatic messages, deliver templated messages |

## 5. The packages the domain is delivered in

| Package | What it adds |
|---|---|
| Recruitment | The core: every owned entity, the three privileges, the pipeline, the refusal, the hiring hand-over and the analysis screens |
| Recruitment — Skills Management | Application Skills, the required skills on a position, the match score in both directions, the transfer at hire and the two matching searches |
| Recruitment — Text Message | The bulk text-message action on Applications |
| Recruitment Interview Forms | The written interview: the questionnaire on a position, the invitation, the answer set link and the printable answers |
| Online Jobs | The public job list, detail page, application form and thank-you page, the publication fields, the tracking address, and the public read rules |
| Recruitment Live Chat | The guided chat script on the public job list |

Details, including what each package requires, are in
[configuration.md](configuration.md#1-capability-packages).

## 6. Entities of this domain

Entities owned by this domain (created here, and not meaningful outside it):

| Entity | Transport name | Storage name | One-line purpose |
|---|---|---|---|
| Application | `hr.applicant` | `hr_applicant` | One person presenting themselves for one opening, with stage, outcome, attachments and evaluation. |
| Recruitment Stage | `hr.recruitment.stage` | `hr_recruitment_stage` | One ordered column of the hiring pipeline, optionally restricted to given job positions and optionally flagged as the hired stage. |
| Refusal Reason | `hr.applicant.refuse.reason` | `hr_applicant_refuse_reason` | A named reason for turning an application down, optionally carrying the message template to send. |
| Application Tag | `hr.applicant.category` | `hr_applicant_category` | A free label attachable to applications and to talent pools. |
| Degree | `hr.recruitment.degree` | `hr_recruitment_degree` | A level of education with a numeric score used in skill matching. |
| Recruitment Source | `hr.recruitment.source` | `hr_recruitment_source` | A named origin of applications for one job position, with its own electronic mail alias and its own tracking address. |
| Job Board | `hr.job.platform` | `hr_job_platform` | An external job-board sender address, with the pattern used to extract the candidate's name from its notification messages. |
| Talent Pool | `hr.talent.pool` | `hr_talent_pool` | A named, managed collection of candidates kept for future openings. |
| Application Skill | `hr.applicant.skill` | `hr_applicant_skill` | One skill claimed by one application, at one level, optionally with a validity window. |
| Refusal Wizard | `applicant.get.refuse.reason` | not stored | The transient dialog that applies a refusal reason, optionally sends the message, and optionally refuses the duplicates. |
| Application Message Wizard | `applicant.send.mail` | not stored | The transient dialog that sends one electronic mail message to a set of applications. |
| Add-to-Pool Wizard | `talent.pool.add.applicants` | not stored | The transient dialog that adds applications to one or more talent pools, creating the canonical talent copy when needed. |
| Add-to-Job Wizard | `job.add.applicants` | not stored | The transient dialog that copies talents into one or more job positions as fresh applications. |

Entities owned by other domains but extended here, and therefore described in this folder
only for the parts this domain adds:

| Entity | Transport name | What this domain adds |
|---|---|---|
| Job Position | `hr.job` | Recruitment target usage, applications, application counters, interviewers, extended interviewers, electronic mail alias, location, industry, expected degree, required skills, written interview questionnaire, favourites, sourcing children, custom properties and the public website page. |
| Employee | `hr.employee` | The back-link to the applications that produced the employee, and the automatic log line when an employee is created from an application. |
| Department | `hr.department` | The new-application counter and the hired/expected headcount roll-up. |
| Contact | `res.partner` | The back-link to the applications that reference the contact. |
| Login User | `res.users` | Automatic grant and revocation of the Interviewer privilege. |
| Meeting | `calendar.event` | The link to the application, the default values when scheduling from an application, and the copying of the application's attachments onto the meeting. |
| Questionnaire | `survey.survey` | A recruitment kind of questionnaire and the job positions that use it. |
| Questionnaire Answer Set | `survey.user_input` | The link to the application and the message posted when the candidate finishes. |
| Periodic Digest | `digest.digest` | The new-colleagues indicator. |
| Tracking Campaign / Tracking Source | `utm.campaign` / `utm.source` | Deletion protection for the records the recruitment flow depends on. |

## 7. Reading order, and every file in this folder

1. **[entities.md](entities.md)** — every entity with its complete field table, relations,
   defaults, computed rules, ordering, display rule, uniqueness and archival behaviour.
   Read this first; every other file refers back to field names defined here.
2. **[state-machines.md](state-machines.md)** — the stage machine, the derived application
   status, the kanban readiness state, the publication state of a job position, and the
   questionnaire answer state as recruitment uses it.
3. **[workflows.md](workflows.md)** — the end-to-end procedures: opening a position,
   receiving an application through each of the four channels, moving it through the
   pipeline, interviewing, refusing, hiring, pooling and re-applying.
4. **[business-rules.md](business-rules.md)** — every validation, constraint, invariant,
   exact error message and permission check, including the interviewer restriction stated
   rule by rule.
5. **[calculations.md](calculations.md)** — every formula and algorithm: the counters, the
   duration measures, the duplicate detection, the skill match score, the staleness
   computation and the target arithmetic, each with worked numbers.
6. **[configuration.md](configuration.md)** — settings, groups, the access rights matrix,
   the record rules, the default records shipped with the domain, and the scheduled work.
7. **[interfaces.md](interfaces.md)** — menus, views, buttons, filters, groupings, named
   remote operations, public routes, message templates and external integrations.
8. **[accounting-effects.md](accounting-effects.md)** — why this domain posts nothing to
   the ledger and how it nevertheless reaches the financial domains indirectly.
9. **[acceptance-criteria.md](acceptance-criteria.md)** — numbered scenarios with concrete
   numbers that a rebuild must satisfy.
10. **[glossary.md](glossary.md)** — every term used in this folder, defined in full.

The folder holds exactly these eleven files and no others:

| File | What it contains |
|---|---|
| [README.md](README.md) | This file: the scope, the capabilities, the entities owned and extended, the roles, the packages, the reading order, the dependencies and the boundaries |
| [entities.md](entities.md) | Every entity in full: purpose, identity, ordering, display rule, complete field table with identifier, full name, type, target, requiredness, default, derivation and meaning, relations, uniqueness, indexes, archival, company behaviour, the extensions other packages contribute, and the lifecycle of every configuration entity |
| [state-machines.md](state-machines.md) | The five state-bearing fields and the talent condition: every state with its stored value, label and meaning; every transition with origin, destination, trigger, guards and side effects; the guards of each transition in order with their exact refusal message; a diagram per machine |
| [workflows.md](workflows.md) | Twenty-seven end-to-end procedures, step by step, with the records each step creates or changes, the operations invoked and the failure conditions |
| [business-rules.md](business-rules.md) | Two hundred and twenty-two numbered rules with their exact messages, the rule index, and the mapping of the identifiers used by the two earlier drafts |
| [calculations.md](calculations.md) | Every counter, duration, duplicate condition, match score, staleness rule, filter counter and rounding rule, each with a worked numeric example |
| [accounting-effects.md](accounting-effects.md) | The reasoned statement that this domain posts nothing to the ledger, the per-event boundary table, and what a rebuild must guarantee |
| [configuration.md](configuration.md) | The packages, the settings, the privileges, the access rights matrix, the record rules, every shipped record, the message templates and subtypes, the constants, the menus and the master data a new installation needs |
| [interfaces.md](interfaces.md) | The screens, the named operations, the public routes, the addresses the domain builds, the reports, the message subtypes, the notifications, the external integrations, the import and export surfaces, the control matrix and the reporting measures |
| [acceptance-criteria.md](acceptance-criteria.md) | Two hundred and forty-three numbered Given / When / Then scenarios with concrete records, inputs, amounts, states and messages |
| [glossary.md](glossary.md) | Every term of the domain, defined, plus the terms that mean something narrower here than elsewhere |

## 8. Dependencies on other domains

| Domain | What this domain needs from it |
|---|---|
| [Human Resources Core](../human-resources-core/README.md) | The Job Position entity (this domain extends it), the Department entity, the Employee entity and the employee-creation contract, the Contract Type, the Degree-adjacent skill model (skills, skill types, skill levels). |
| [Messaging and Activities](../messaging-and-activities/README.md) | The message thread and follower mechanics on applications and job positions, the electronic mail alias mechanism that turns inbound messages into applications, the message templates, the message subtypes, the activity and activity-plan mechanics, the blacklist mixin, the telephone-number sanitisation mixin, and the stage-duration tracking mixin. |
| [Contacts and Organizations](../contacts-and-organizations/README.md) | The Contact entity used as the applicant's contact record, the address resolution used when creating the employee, and the industry list. |
| [Identity and Access](../identity-and-access/README.md) | The login user entity, the group mechanism, the record rule mechanism and the company scoping. |
| [Calendar and Scheduling](../calendar-and-scheduling/README.md) | The Meeting entity used for interviews. |
| [Learning, Surveys and Gamification](../learning-surveys-and-gamification/README.md) | The questionnaire, its questions, its answer sets and the invitation wizard used for written interviews. |
| [Website and Storefront](../website-and-storefront/README.md) | The public page mechanism, the generic public form handler, the search mechanism and the page-publication mixin used for the public job pages. |
| [Marketing and Mass Mailing](../marketing-and-mass-mailing/README.md) | The tracking campaign, medium and source entities used for sourcing attribution, and the text-message composer used to reach applicants. |
| [Spreadsheets and Dashboards](../spreadsheets-and-dashboards/README.md) | The periodic digest that carries the new-colleagues indicator. |

## 9. What this domain does not do

- It does not create or maintain employment terms. Creating an employee from an application
  produces an Employee record with an initial set of values; everything about wages,
  schedules and contract dates belongs to
  [Human Resources Core](../human-resources-core/README.md).
- It does not post to the general ledger. See
  [accounting-effects.md](accounting-effects.md).
- It does not define the questionnaire engine. It only marks a questionnaire as being of
  the recruitment kind, links it to job positions and applications, and restricts who may
  read the answers.
- It does not define the public website rendering engine, the form builder, or the search
  engine; it registers a model, a set of writable fields, a few routes and a few page
  templates with them.

## 10. Reconciliation notes

Two independently written descriptions of this domain were merged into this folder. The
differences that mattered are recorded at the end of the file they affect: see the last
chapter of [entities.md](entities.md), [state-machines.md](state-machines.md),
[workflows.md](workflows.md), [business-rules.md](business-rules.md),
[calculations.md](calculations.md), [configuration.md](configuration.md),
[interfaces.md](interfaces.md), [accounting-effects.md](accounting-effects.md),
[acceptance-criteria.md](acceptance-criteria.md) and [glossary.md](glossary.md).

Three decisions were taken once and applied to every file:

1. **Identifiers are the stored ones.** Where the two drafts disagreed on a name, this folder
   reproduces the identifier the system stores — `kanban_state` rather than a descriptive
   name, `stage_id` rather than `stage`, `job_id` rather than `job_position` — because those
   strings are contractual. The descriptive wording of the other draft survives as the
   meaning column of the field tables and as entries in the glossary.
2. **Rule identifiers are one sequence per file.** The rules are numbered `REC-001` upward in
   [business-rules.md](business-rules.md), and the scenarios `REC-AC-001` upward in
   [acceptance-criteria.md](acceptance-criteria.md). Chapter 20 of the rules file maps every
   identifier the two drafts used onto the present scheme.
3. **Where the drafts contradicted each other, the system decided.** Each such point is
   recorded in the reconciliation chapter of the affected file, with the resolution and the
   reason.
