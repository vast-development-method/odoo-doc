# Recruitment — Configuration

Everything an installation has to configure or ships already configured: the capability
packages, the settings, the privileges, the access rights matrix, the record rules, the
default records, the message templates and subtypes, the activity configuration, the
constants, the menus and the master data the domain needs before it can be used. Rule
identifiers refer to [business-rules.md](business-rules.md).

## 1. Capability packages

The domain is delivered as one core package and five companions. Each companion is optional
and adds a bounded set of entities, fields and screens; nothing in the core depends on a
companion being present.

| Package (business name) | What it adds | What it needs |
|---|---|---|
| Recruitment | The whole core: the Application, the Recruitment Stage, the Refusal Reason, the Application Tag, the Degree, the Recruitment Source, the Job Board, the Talent Pool, the four transient dialogs, the three privileges, the analysis screens, and every field added to the Job Position, the Department, the Employee, the Contact, the Login User, the Meeting, the Company, the settings record and the periodic digest | The people register, the calendar, the attribution trackers, attachment content indexing, the guided-tour mechanism and the periodic digest |
| Recruitment — Skills Management | The Application Skill entity, the skill fields on the Application and the Job Position, the match score on both sides, the transfer of skills at hire, and the two matching searches | The skills catalogue of [Human Resources Core](../human-resources-core/README.md) |
| Recruitment — Text Message | The bulk text-message action on Applications | The text-message gateway |
| Recruitment Interview Forms | The written interview questionnaire on the position, the invitation from the Application, the answer-set link, the completion note and the printable answers | The questionnaire engine of [Learning, Surveys and Gamification](../learning-surveys-and-gamification/README.md) |
| Online Jobs | The public job list, the public detail page, the public application form, the thank-you page, the publication fields on the position, the tracking address on the Recruitment Source, the exposure of department names to visitors and the public record rules | The website engine and its messaging companion, from [Website and Storefront](../website-and-storefront/README.md) |
| Recruitment Live Chat | A guided chat script that helps a visitor of the job list find a position and hands over to the recruiting team | Online Jobs and the live-chat capability |

## 2. Settings

Three switches live in the recruitment section of the settings screen, which is visible to
the platform administrator. Each is a truth value whose only effect is to install or remove
a companion.

| Setting (storage name) | Interface label, reproduced | Type | Default | Effect |
|---|---|---|---|---|
| `module_website_hr_recruitment` | "Online Posting" | true or false | false | Installs or removes the public job pages companion |
| `module_hr_recruitment_survey` | "Interview Forms" | true or false | false | Installs or removes the written interview companion, which in turn installs the questionnaire engine |
| `module_hr_recruitment_extract` | "Send CV to OCR to fill applications" | true or false, company-dependent | false | Installs or removes the companion that sends an uploaded curriculum vitae to an optical character recognition service and fills the applicant's name, telephone number and electronic mail address from the result. That companion is outside the scope of this folder; the switch is documented because it appears on this screen. |

The screen also shows an informational block about the credit balance of the text-message
gateway, with a link to the gateway's credit purchase screen. It stores nothing.

## 3. Access privileges

| Privilege | Implies | How it is granted | What it allows |
|---|---|---|---|
| Interviewer | The internal-user privilege | **Automatically**, to every user named as an interviewer on a Job Position or on an Application, and revoked automatically when the last such attachment disappears (REC-033, REC-034) | Read and update the Applications the user is attached to, and the Applications of the positions the user is attached to; read those positions; refuse; schedule meetings; send and read written interviews; read stages and refusal reasons |
| Officer: Manage all applicants | Interviewer | Manually | Everything on Applications, Job Positions, Talent Pools, Refusal Reasons, Degrees, Recruitment Sources, Application Tags, Contacts and Meetings; read stages without changing them; see and edit the salary fields |
| Administrator | Officer | Manually. The platform's own technical user and the shipped administrator user are members | Everything the Officer may do, plus create, update and delete Recruitment Stages, Job Boards and recruitment activity plans, and manage the written interview questionnaires |
| Curriculum vitae display | none | Automatically, to every internal user | Show the preview panel of the curriculum vitae beside the application form. Its reproduced name is `Display CV on application form`. |

The three main privileges belong to one privilege group named `Recruitment`, sequence 11,
inside the human-resources category, so that they appear as one selector on a user's form.
Installing the public job pages companion additionally grants the restricted site-editor
privilege to the Officer privilege (REC-174).

## 4. Sequences and numbering

This domain defines **no sequence and no numbering scheme**. Applications, Job Positions,
Talent Pools and every configuration record are identified by their surrogate identifier and
by their name; nothing carries a human-readable reference number.

Ordering is carried by three manual sequence fields, each with its own default:

| Entity | Field | Default | What it orders |
|---|---|---|---|
| Application | `sequence` | 10 | The order inside one pipeline column, at equal evaluation |
| Recruitment Stage | `sequence` | 10 (0 to 5 on the shipped stages) | The pipeline columns, and which stage is a position's first stage |
| Refusal Reason | `sequence` | 10 (10 to 15 on the shipped reasons) | The order in the refusal dialog, and which reason is proposed by default |
| Degree | `sequence` | 1 (1 to 4 on the shipped degrees) | The display order of the degree list |
| Job Position | `sequence` | 10 | The order of the position cards and of the public job list |

## 5. Default records shipped with the domain

### 5.1 Degrees

| Name | Sequence | Score | External identifier |
|---|---|---|---|
| Graduate | 1 | 0.50 | `degree_graduate` |
| Bachelor Degree | 2 | 0.70 | `degree_bachelor` |
| Master Degree | 3 | 0.90 | `degree_licenced` |
| Doctoral Degree | 4 | 1.00 | `degree_bac5` |

The score is a fraction of one and is displayed as a percentage; a score of 1.00 means a
perfect match on the education criterion of the skill match.

### 5.2 Application tags

| Reproduced name | External identifier | Meaning |
|---|---|---|
| `Reserve` | `tag_applicant_reserve` | Candidates worth keeping for a later opening |
| `Manager` | `tag_applicant_manager` | Candidates for a management position |
| `IT` | `tag_applicant_it` | Candidates for an information-technology position. The shipped label is the two-letter abbreviation, reproduced here because it is a stored value; a rebuild may ship the phrase in full. |
| `Sales` | `tag_applicant_sales` | Candidates for a sales position |

A fifth tag named `Demo` exists in the demonstration data only.

### 5.3 Recruitment stages

| Name | External identifier | Sequence | Folded | Hired stage | Message template |
|---|---|---|---|---|---|
| New | `stage_job0` | 0 | no | no | Recruitment: Application Acknowledgement |
| Qualification | `stage_job1` | 1 | no | no | none |
| First Interview | `stage_job2` | 2 | no | no | none |
| Second Interview | `stage_job3` | 3 | no | no | none |
| Contract Proposal | `stage_job4` | 4 | no | no | none |
| Contract Signed | `stage_job5` | 5 | **yes** | **yes** | none |

All six are unrestricted: none of them names a Job Position, so every new position uses them
immediately. None of them defines a staleness threshold, so the staleness feature is inactive
until an administrator sets one. Every stage carries the four default readiness labels
`In Progress`, `Ready for Next Stage`, `Waiting` and `Blocked`.

Because the hired stage is folded, a new Application never lands in it (REC-036), and because
the first stage carries a template, every Application that is **moved** into it, or created
with it written into the creation values, receives the acknowledgement message (REC-051,
REC-052).

### 5.4 Refusal reasons

| Sequence | Name | External identifier | Template proposed |
|---|---|---|---|
| 10 | Refused by applicant: salary | `refuse_reason_8` | Recruitment: Not interested anymore |
| 11 | Refused by applicant: job fit | `refuse_reason_2` | Recruitment: Not interested anymore |
| 12 | Does not fit the job requirements | `refuse_reason_1` | Recruitment: Refuse |
| 13 | Job already fulfilled | `refuse_reason_5` | Recruitment: Refuse |
| 14 | Duplicate | `refuse_reason_6` | Recruitment: Refuse |
| 15 | Spam | `refuse_reason_7` | Recruitment: Refuse |

The refusal dialog proposes the first of them in sequence order, that is
*Refused by applicant: salary*.

### 5.5 Job boards

| Name | Sender address | Extraction pattern | What the pattern captures | External identifier |
|---|---|---|---|---|
| Linkedin | `jobs-listings@linkedin.com` | `New application:.*from (.*)` | Everything after the word *from* on the text that begins with *New application:* | `linkedin_job_platform` |
| Jobsdb | `cs@jobsdb.com` | `from (.+?) for` | The shortest run of characters between *from* and the next *for* | `jobsdb_job_platform` |
| Indeed | `no-reply@indeed.com` | `^([^ ]+ [^ ]+)` | The first two space-separated words of the text | `indeed_job_platform` |

### 5.6 Message templates

| Name | Rendered against | Subject | Used by | Log copy deleted after sending |
|---|---|---|---|---|
| Recruitment: Application Acknowledgement | The Application | `Your Job Application: ` followed by the position name | The stage *New* | yes |
| Recruitment: Interest | The Application | `Your Job Application: ` followed by the position name | Offered for a stage that shortlists a candidate; attached to no shipped stage | yes |
| Recruitment: Refuse | The Application | `Your Job Application: ` followed by the position name | Four refusal reasons | yes |
| Recruitment: Not interested anymore | The Application | `Your Job Application: ` followed by the position name | Two refusal reasons | yes |
| Applicant: Interview | The Answer Set | `Participate to ` followed by the questionnaire name and ` interview` | The written interview invitation | yes |

All five address the recipient computed from the record rather than a fixed address: no
recipient address and no recipient contact is stored on the template. Their bodies carry the
position name, the company name, the recruiter's name, address and telephone number when a
recruiter is set, the address block of the position's job location, and, when the public job
pages companion is installed, a control linking to the position's public page.

Note the interaction with REC-051: although these templates are marked for deletion of the
sent copy, the stage-change path posts them with that deletion disabled, so a stage message
stays visible in the thread.

### 5.7 Attribution records

One campaign named `Job Campaign`, external identifier `utm_campaign_job`, is shipped and may
not be deleted (REC-146). Two shared media are used by name and are created on demand when
they do not exist: `website`, the default medium of a new Recruitment Source, and `email`,
stamped by a Recruitment Source's inbound alias.

### 5.8 Periodic digest and guided tour

- The shipped periodic digest has the new-employees indicator switched on.
- A digest tip titled `Tip: Let candidates apply by email` is shipped for the Administrator
  privilege. It explains that giving a position an inbound address turns incoming messages
  into applications, mentions that several sources may be used to attribute applications, and
  offers a control that opens a message addressed to the first position address it finds.
- A guided tour of the domain is shipped. It ends with the message
  `Great job! You hired a new colleague!` followed by
  `Try the Website app to publish job offers online.`

### 5.9 Public site records, from the public job pages companion

| Record | Value |
|---|---|
| Site menu entry | `Jobs`, pointing at `/jobs`, sequence 59, under the main site menu |
| Page | `Thank you (Recruitment)` at `/job-thank-you`, published, excluded from search-engine indexing |
| Start-up action | Opens `/jobs` in the same window once the companion is installed |
| Public form registration | The Application entity is registered as a public form target with the key `apply_job` and the label `Apply for a Job` |
| Writable field list | The seven names of REC-086 |

## 6. Access rights matrix

Model-level rights, before record rules. A blank cell means the right is not granted by that
line.

| Entity | Privilege | Read | Write | Create | Delete |
|---|---|---|---|---|---|
| Job Position | Interviewer | yes | — | — | — |
| Job Position | Officer | yes | yes | yes | yes |
| Job Position | Human Resources Officer | yes | — | — | — |
| Job Position | Public visitor, portal user, internal user (public job pages companion) | yes | — | — | — |
| Application | Interviewer | yes | yes | — | — |
| Application | Officer | yes | yes | yes | yes |
| Talent Pool | Interviewer | yes | — | — | — |
| Talent Pool | Officer | yes | yes | yes | yes |
| Recruitment Stage | Interviewer | yes | — | — | — |
| Recruitment Stage | Officer | yes | — | — | — |
| Recruitment Stage | Administrator | yes | yes | yes | yes |
| Degree | Officer | yes | yes | yes | yes |
| Refusal Reason | Interviewer | yes | — | — | — |
| Refusal Reason | Officer | yes | yes | yes | yes |
| Contact | Officer | yes | yes | yes | yes |
| Meeting | Officer | yes | yes | yes | yes |
| Meeting kind | Officer | yes | yes | yes | — |
| Recruitment Source | Internal user | yes | — | — | — |
| Recruitment Source | Officer | yes | yes | yes | yes |
| Application Tag | Internal user | yes | yes | yes | — |
| Application Tag | Officer | yes | yes | yes | yes |
| Job Board | Administrator | yes | yes | yes | yes |
| Activity plan and activity plan step | Administrator | yes | yes | yes | yes |
| Refusal dialog | Interviewer | yes | yes | yes | — |
| Refusal dialog | Officer | yes | yes | yes | — |
| Message dialog | Interviewer | yes | yes | yes | — |
| Message dialog | Officer | yes | yes | yes | — |
| Add-to-pool dialog | Interviewer | — | — | — | — |
| Add-to-pool dialog | Officer | yes | yes | yes | yes |
| Add-to-job dialog | Interviewer | — | — | — | — |
| Add-to-job dialog | Officer | yes | yes | yes | yes |
| Application Skill (skills companion) | Interviewer | yes | yes | yes | yes |
| Job Skill (skills companion) | Officer | yes | yes | yes | yes |
| Department | Public visitor (public job pages companion) | yes | — | — | — |
| Answer Set, answer line, questionnaire, question, question answer (written interview companion) | Administrator | yes | yes | yes | yes |
| Answer Set, answer line (written interview companion) | Officer | yes | — | — | — |
| Interview invitation (written interview companion) | Officer | yes | yes | yes | — |
| Answer Set, answer line, questionnaire, question (written interview companion) | Interviewer | yes | — | — | — |
| Interview invitation (written interview companion) | Interviewer | yes | yes | yes | — |

The two lines that grant nothing to the Interviewer privilege on the two pooling dialogs are
deliberate and explicit: without them the Interviewer privilege would inherit nothing at all,
and the intent of the configuration would be unreadable.

## 7. Record rules

| Rule | Entity | Privilege | Condition | Rights it applies to |
|---|---|---|---|---|
| Application, multi-company | Application | every reader (global) | The Application's company is among the reader's allowed companies, or is empty | all four |
| Application, interviewer | Application | Interviewer | The acting user is among the Application's interviewers, or among the interviewers of the Application's Job Position | read and update only; create and delete are excluded |
| Application, officer | Application | Officer | always true | all four |
| Job Position, officer | Job Position | Officer | always true | all four |
| Talent Pool, officer | Talent Pool | Officer | always true | all four |
| Message, officer | Message | Officer | always true | all four |
| Activity plan, administrator | Activity plan | Administrator | The plan's target entity is the Application entity | write, create and delete; read is not restricted by this rule |
| Activity plan step, administrator | Activity plan step | Administrator | The plan's target entity is the Application entity | write, create and delete |
| Application Skill, interviewer | Application Skill | Interviewer | The skill line's Application names the reader as interviewer, or that Application's position does | all four |
| Application Skill, officer | Application Skill | Officer | always true | all four |
| Job Position, public | Job Position | Public visitor | The publication flag is true | read only |
| Job Position, portal | Job Position | Portal user | The publication flag is true | read only |
| Job Position, officer through the site | Job Position | Officer | always true | read only |
| Department, public | Department | Public visitor | At least one of the department's positions is published, or the department has child departments | read only |
| Answer Set, answer line, questionnaire, question, question answer, invitation — administrator | The written-interview entities | Administrator | The questionnaire kind is `recruitment` | read, write and create everywhere; delete everywhere except on invitations |
| Answer Set and answer line — officer | The written-interview entities | Officer | The questionnaire kind is `recruitment` **and** the questionnaire is unrestricted or names the reader among its restricted users | read only |
| Invitation — officer | Interview invitation | Officer | The same condition | read, write and create |
| Answer Set and answer line — interviewer | The written-interview entities | Interviewer | The answer set's Application names the reader as interviewer, or that Application's position does | read only |
| Questionnaire and question — interviewer | The written-interview entities | Interviewer | The questionnaire kind is `recruitment` and the reader is an interviewer of one of its positions, or of one of the applications of those positions | read only |
| Invitation — interviewer | Interview invitation | Interviewer | The same condition | read, write and create |

## 8. Message subtypes

Listed with their defaults in [business-rules.md](business-rules.md#17-messaging-notification-and-reporting-rules),
REC-207. Their operational meaning and the notifications they drive are in
[interfaces.md](interfaces.md#8-message-subtypes).

## 9. Activity configuration

- Activity types are configured from the recruitment configuration menu; the domain ships
  none of its own and reuses the platform's.
- Activity **plans** whose target entity is the Application entity are managed by the
  Administrator privilege, become department-assignable, and become department-filterable in
  the activity scheduling dialog. The domain ships no plan; an installation defines its own,
  for example a plan that schedules a screening call, a technical interview and a reference
  check.

## 10. Scheduled work

This domain owns **no scheduled job**. Three periodic mechanisms owned elsewhere touch it:

1. The periodic digest reads the new-employees indicator described in §5.8 and in
   [calculations.md](calculations.md#12-the-periodic-digest-indicator).
2. The platform's periodic cleanup of transient records removes stale dialog records,
   including the four dialogs of this domain.
3. The inbound message poller of
   [Messaging and Activities](../messaging-and-activities/README.md) delivers messages to the
   position addresses and to the source addresses, which is what creates applications by
   electronic mail.

Every derived value of this domain is recomputed when its inputs change; none depends on a
nightly run.

## 11. Constants and parameters

| Name | Kind | Value | Effect |
|---|---|---|---|
| Positions per page on the public job list | fixed constant | 12 | Pagination of the public list (REC-199) |
| Maximum positions searched for the public job list | fixed constant | 600, that is twelve multiplied by fifty | Upper bound of the search that feeds the list and its filter counters (REC-200) |
| Answer deadline proposed for a written interview invitation | fixed constant | the current moment plus fifteen days | Prefills the deadline of the invitation dialog |
| Recent-application window of the public live check | fixed constant | six months | The first branch of REC-099 |
| Staleness look-back window | platform-wide parameter, shared with the opportunity pipeline | twelve months | Applications whose last stage update is older are excluded from the staleness search |
| Public form metadata capture | platform-wide parameter | disabled | When enabled, the note carrying the custom entries of a public submission also lists the visitor's network address, browser identification, accepted languages and referring page (REC-094) |
| Colour index range for a new tag or pool | fixed constant | 1 to 11 inclusive | The pseudo-random colour drawn at creation |
| Electronic mail address length on an Application | fixed constant | 128 characters | REC-003 |
| Telephone number length on an Application | fixed constant | 32 characters | REC-003 |

## 12. Menus

| Menu entry | Parent | Visible to | Opens |
|---|---|---|---|
| Recruitment | the application root | Officer, Interviewer | The domain |
| Applications | Recruitment | Officer, Interviewer | A grouping entry |
| By Job Positions | Applications | Officer | The position cards |
| By Job Positions (interviewer variant) | Applications | Interviewer | The positions the reader interviews for, without the creation control |
| By Talent Pools | Applications | Officer | The talent pool cards |
| All Applications | Applications | Officer, Interviewer | Every Application, filtered to non-talent records |
| Reporting | Recruitment | Officer | A grouping entry |
| Recruitment Analysis | Reporting | Officer | The analysis graph and matrix |
| Configuration | Recruitment | Officer | A grouping entry |
| Settings | Configuration | Platform administrator | The settings screen |
| Job Positions | Configuration | Officer | A grouping entry holding Stages and Employment Types |
| Stages | Job Positions | Technical readers only | The stage list |
| Employment Types | Job Positions | Human Resources Officer | The employment type list |
| Trackers | Configuration | Technical readers only | A grouping entry holding Sources and Mediums |
| Applications | Configuration | Officer | A grouping entry holding Degrees, Refusal Reasons and Tags |
| Employees | Configuration | Officer | A grouping entry holding Departments, and, with the skills companion, Skill Types |
| Activities | Configuration | Officer | A grouping entry holding Activity Types and, for the Administrator privilege, recruitment activity plans |
| Job Boards | Configuration | Officer | A grouping entry holding the job board list, which the access rights limit to the Administrator privilege |
| Interviews | Configuration | Administrator | The recruitment questionnaires (written interview companion) |
| Jobs | The site content menu | Interviewer | The public job pages list (public job pages companion) |

Exactly one of the two *By Job Positions* entries reaches a given reader; the other is hidden
by REC-178.

## 13. Master data a new installation needs

| Prerequisite | Owned by | Why the domain needs it |
|---|---|---|
| At least one company with a contact record | [Contacts and Organizations](../contacts-and-organizations/README.md) | The default job location is the company's contact, and the created Employee's work address is the company's contact |
| An alias domain on the company | [Messaging and Activities](../messaging-and-activities/README.md) | Without it a position's inbound address has no domain, applications cannot arrive by electronic mail, and the address control of a Recruitment Source is hidden |
| At least one Recruitment Stage | this domain (six are shipped) | Without a stage, applications are created with no stage and the pipeline has no column |
| At least one Refusal Reason | this domain (six are shipped) | The refusal dialog requires a reason |
| Degrees | this domain (four are shipped) | Needed by the education part of the skill match |
| Departments and Job Positions | [Human Resources Core](../human-resources-core/README.md) | A position is the anchor of the whole pipeline |
| A skills catalogue with types, skills and levels | [Human Resources Core](../human-resources-core/README.md) | Needed by the skills companion and by the match score |
| A questionnaire of the recruitment kind | [Learning, Surveys and Gamification](../learning-surveys-and-gamification/README.md) | Needed by the written interview companion |
| A published website | [Website and Storefront](../website-and-storefront/README.md) | Needed by the public job list |
| Employment types | [Human Resources Core](../human-resources-core/README.md) | Shown on the public detail page and used as a public filter |
| Industries | [Contacts and Organizations](../contacts-and-organizations/README.md) | Used as a public filter and as a position attribute |

## 14. Reconciliation notes

| Point | Resolution |
|---|---|
| Which reason the refusal dialog proposes | One draft said *Does not fit the job requirements*, the other *Refused by applicant: salary*. The dialog takes the first reason in sequence order, and the shipped sequences run from 10 for *Refused by applicant: salary*. The second is correct and §5.4 gives every sequence. |
| Whether stages are writable by the Officer privilege | One draft granted write to the Officer privilege. The matrix grants the Officer privilege read only; write, create and delete belong to the Administrator privilege. §6 and REC-164. |
| The name of the curriculum-vitae display privilege | Reproduced in §3 as it is stored, with the abbreviation intact, because an installation carries that exact string. |
| Whether the domain ships an activity plan | One draft implied it did. It does not; the plan mechanism is configured, not populated. §9. |
| Whether the domain owns a scheduled job | Both drafts agree it does not. §10 lists the three external mechanisms that touch it. |
| The shipped tag written as an abbreviation | One draft spelled the third tag out in full, the other reproduced the two-letter form. The stored value is the two-letter form; §5.2 reproduces it and states the meaning in full. |
| The digest tip | Only one draft mentioned it. It is shipped and is documented in §5.8. |
