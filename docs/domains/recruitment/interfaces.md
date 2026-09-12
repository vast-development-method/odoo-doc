# Recruitment — Interfaces

Everything through which a person, another system or a scheduled mechanism reaches this
domain: the screens, the named operations, the public routes, the addresses the domain
builds, the printable documents, the message subtypes, the notifications, the external
integrations and the import and export surfaces. Rule identifiers refer to
[business-rules.md](business-rules.md).

Operation names in this file are the domain's stable operation names, reproduced in code
font where an integration keys on them, and described in words otherwise.

## 1. The shape of the interface

The domain presents four working surfaces and one public surface.

| Surface | Who uses it | Entry point |
|---|---|---|
| The position cards | Officer, Interviewer | The recruitment menu |
| The pipeline of one position | Officer, Interviewer | A position card |
| The Application form and its list, calendar, activity, graph and matrix views | Officer, Interviewer | A card, or the *All Applications* menu |
| The configuration screens | Officer, Administrator | The configuration menu |
| The public job list, detail page, application form and thank-you page | Any visitor | The site menu entry *Jobs* |

The menu tree and its visibility are in
[configuration.md](configuration.md#12-menus). The conventions these screens follow — what a
card view, a list, a form, a graph and a matrix are, and how an action reaches them — are
defined once in [views and actions](../../overview/views-and-actions.md). The public routes of
§5 are listed alongside every other route of the platform in
[the endpoint catalogue](../../interfaces/endpoint-catalog.md), and the operation style of §3
and §4 follows [the service layer](../../interfaces/service-layer.md).

## 2. Screens

### 2.1 Position cards

- **Shape.** Cards, forty per page, ordered with published positions first, then the reader's
  favourites, then the most recently created. A card opens the applications of that position.
- **On each card.** The favourite marker; the position name; the inbound address when the
  position has one; the recruiter with an avatar; the company in a multi-company
  installation; the remaining target with the caption `to recruit`; the number of new
  applications with the caption `new applications`; the number of applications in progress
  with the caption `in progress`; a badge carrying the reader's own activity count; a control
  labelled `Job Page` when the public job pages companion is installed; and a control
  labelled `Configure`. A green banner reading `Published` marks a published position.
- **Card menu.** Applications, Activities, Trackers, create an Application, Analysis, the
  colour picker, Configuration, Archive or Unarchive, and the control that copies the public
  address.
- **Filters.** `My Favorites`, `My Job Positions`, `Published`, `Unread Messages`,
  `Archived`; grouping by department, by company and by publication state; a side panel that
  filters by company and by department with counters.
- **Creation.** Opens the two-field dialog of [workflows.md](workflows.md#1-opening-a-job-position).

### 2.2 The pipeline of one position

- **Shape.** Cards grouped by stage. The grouping expands to every stage available to the
  position, empty ones included, by the rule in
  [entities.md](entities.md#2412-kanban-column-expansion). A folded stage is collapsed while
  it holds no record. Each column carries a progress bar counting the four readiness values,
  coloured green for *Ready for Next Stage*, orange for *Waiting* and red for *Blocked*; the
  grey value is not counted in the bar.
- **On each card.** The applicant's name; the position, hidden when the screen is already
  filtered on one position; the tags; the custom properties; the evaluation stars; the
  activity marker; the staleness marker; the number of attachments, which is clickable; the
  readiness selector; and the recruiter's avatar. Banners mark `Hired`, `Refused` and
  `Archived`.
- **Card menu.** `Schedule Interview`, `Refuse`, `Archive` or `Unarchive`, `Delete` when the
  reader may delete, and `Add to Pool`.
- **Quick creation.** A small card asking only for the applicant's name, the position and the
  company.
- **Header controls.** `Add Applicants` when the screen is a talent pool, `Create
  Applications`, and `Refuse`.
- **Empty-list text.** `No applications found.` followed, when the position has an inbound
  address, by `Send applications to` and that address.

### 2.3 The Application list

- **Columns.** Name, applied-on date, electronic mail address, telephone number, Job
  Position, stage with its staleness badge, evaluation, tags, recruiter, application status
  (hidden while it is `ongoing`), refusal reason, activities, interviewers, medium, source,
  expected salary, proposed salary, availability, department and company. Several columns are
  optional and hidden until the reader adds them.
- **Behaviour.** Editing several rows at once is allowed; refused rows are shown in red; the
  default grouping is by stage.

### 2.4 The Application form

- **Status bar.** The stages available to the position, clickable. Hidden for a talent, and
  for an archived Application that produced no Employee.
- **Header controls.** `Create Employee` for the Human Resources Officer privilege, visible
  only when the Application is active, carries a hire date and has no Employee; `Send
  Interview` with the written interview companion, visible when the Application is active and
  its position carries a questionnaire; `Refuse`, hidden for a talent and for an archived
  Application; `Restore`, visible when archived; `Create Applications`, on a talent; `Add to
  Pool`, hidden for a talent and for an Application already linked to a pool.
- **Statistic controls.** The linked Employee; the other applications of the same person with
  their count; the meetings with their summary text and date; the talent pools with their
  count; and `Consult Interview` with the written interview companion.
- **Body.** The applicant's name as the title; then the electronic mail address, the
  telephone number and the professional network profile address on the left, and the
  evaluation, the position, the recruiter, the interviewers, the hire date and the tags on
  the right; then the custom properties; then two tabs, one holding the private notes and one
  holding the degree, the availability, the department, the company, the salary group
  restricted to the Officer privilege, and the sourcing group with campaign, medium and
  source.
- **Side panel.** The preview of the curriculum vitae, for readers holding the display
  privilege of [configuration.md](configuration.md#3-access-privileges).
- **Thread.** Attached below, opening on the attachments and reloading when an attachment
  changes.
- **Interviewer variant.** The same form with the whole salary group removed (REC-027).

### 2.5 The other views on Applications

| View | Content |
|---|---|
| Calendar | Applications placed on their next activity deadline, coloured by recruiter, showing the applicant's name, the position, the evaluation and the activity summary |
| Activity | One row per Application with the recruiter's avatar and the applicant's name, one column per activity type |
| Graph, pipeline | Count of Applications by stage |
| Matrix, pipeline | Positions on the rows, stages on the columns |
| Graph, analysis | Count of Applications by stage and by position |
| Matrix, analysis | Stages on the rows, positions on the columns, applicant names as the measure list |

### 2.6 Search, filters and groupings on Applications

- **Searchable.** The applicant's name, which also searches the electronic mail address; the
  electronic mail address; the Job Position; the department, including its child departments;
  the company; the recruiter; the stage; the tags; the refusal reason; the application status;
  the hire date; the activity owner; the activity type; and the indexed text content of the
  attachments, which is how a recruiter searches inside a curriculum vitae.
- **Filters.** `My Applications`, `Unassigned`, `Applicants`, `Talents`, `In Progress`,
  `Hired`, `Ready for Next Stage`, `Waiting`, `Blocked`, `Rotting`, `Directly Available`
  (availability empty or not later than today), `Creation Date`, `Last Stage Update`,
  `Unread Messages`, `Archived`, `Refused`, and the activity filters `My Activities`,
  `Late Activities`, `Today Activities`, `Future Activities` and `Running Applicants`.
- **Groupings.** Position, stage, recruiter, talent pool, creation date, hire date, last stage
  update, refusal reason, company, custom properties, tags and department.

### 2.7 The configuration screens

| Screen | Shape | Notable columns and controls |
|---|---|---|
| Stages | A list with a drag handle, plus cards and a form | Name, staleness threshold, folded flag, hired flag. The form adds the message template, the position restriction, the four readiness labels and the requirements text, and shows the warning of REC-054 |
| Refusal Reasons | An editable list and a form | Sequence handle, description, message template |
| Degrees | An editable list and a form | Sequence handle, name, score displayed as a percentage |
| Tags | An editable list and a form | Name, colour picker |
| Trackers | An editable list | Campaign, source name, medium, and the address cell with a copy control that creates the inbound address on demand. The address column is hidden when no alias domain exists |
| Job boards | A list and a form | Name, sender address, extraction pattern. Its empty-list text is four sentences: `No rules have been defined.`, `Create a new rule to process emails from specific job boards.`, `Define a regular expression: Extract the applicant's name from the email's subject or body.`, `Without a regular expression: The applicant's name will be the email's subject.` |
| Talent Pools | Cards, a list and a form | Name, pool manager, talent count, tags, company, description, colour. Each card carries a `New Talent` control |
| Documents | A list | The file, the applicant's name, the creation date. The search adds a *Content* key that searches inside the indexed text of the files |

### 2.8 The public pages

Present when the public job pages companion is installed.

| Page | Content |
|---|---|
| Job list | The heading `Our Job Offers`; an optional search bar; the five filter controls, each showing the counters of [calculations.md](calculations.md#9-job-list-filter-counters-and-pagination); one card per position showing the name, the number of openings with the wording `open position` or `open positions`, the short description, the city of the job location or `Remote`, the department and the employment type; a pager; and two editable areas, one above and one below. An unpublished position seen by an editor carries a red badge reading `unpublished`. The empty states are listed in [workflows.md](workflows.md#23-browsing-the-public-job-list). |
| Job detail | A bar linking back with the wording `All Jobs`; the position name; the city or `Remote`; two controls labelled `Apply Now!`; the website description; and the structured job-posting description for search engines |
| Application form | The heading `Job Application Form`; the fields of [workflows.md](workflows.md#5-an-application-arriving-through-the-public-form); the two warning areas, one under the professional network field and one for the live duplicate check; a submission control labelled `I'm feeling lucky`; and a side column repeating the position, the location, the department, the employment type and the process details |
| Thank you | `Congratulations!`, then `Your application has been posted successfully,` and `We usually respond within 3 days...`; an illustration; a block inviting the visitor to look around the site; and, when the position has a recruiter, that recruiter's photograph, name, job title, work telephone number and electronic mail address, followed by the sentences explaining that applications are usually answered within three days, that the next step is a call or a meeting, and that the candidate may make contact for faster feedback |

## 3. Named operations on the Application

| Operation | Input | Result | Side effects | Refusals |
|---|---|---|---|---|
| Open the refusal dialog | One or several Applications | A dialog carrying the selection, read with archived records included, and with the template-management controls hidden | none until applied | none |
| Apply the refusal | The dialog: reason, selection, duplicate switch, duplicate list, message switch, subject, body, attachments, scheduled date | Closes the dialog | REC-108 to REC-115 | REC-106, REC-107 |
| Archive | One or several Applications | none | Sets the records inactive with the marker that suppresses stage messages | none |
| Restore | One or several Applications | none | Sets the records active, then runs the reset below | none |
| Reset to the first stage | One or several Applications | none | Writes the first non-folded available stage of each Application's position and clears the refusal reason | none |
| Create the Employee | Exactly one Application | Opens the new Employee | REC-121 to REC-127 | REC-120, REC-121 |
| Open the Employee | Exactly one Application | Opens the linked Employee | none | none |
| Open the attachments | One or several Applications | Lists the attachments of those Applications, with the applicant's name shown on each row | none | none |
| Open the related applications | Exactly one Application | Lists every Application of the same person, archived ones included, grouped by stage, with the creation control suppressed | none | none |
| Open the talent | Exactly one Application | Opens the talent of the same person | Stores the link when it was missing (REC-144) | none |
| Link the Application to its talent | One Application | none | Searches the talent of the same person and stores the link | none |
| Open the add-to-pool dialog | One or several Applications, optionally a target pool | A dialog | none | none |
| Add applicants to pools | The dialog: Applications, pools, tags | Opens the talent when exactly one results, otherwise reloads the screen | REC-134 to REC-138 | none |
| Open the add-to-job dialog | One or several Applications or talents | A dialog | none | none |
| Add applicants to positions | The dialog: records, positions | Opens the new Application when exactly one results, otherwise shows the notification of REC-145 | REC-141, REC-142 | none |
| Create a meeting | Exactly one Application | Opens the calendar with the defaults of [workflows.md](workflows.md#13-scheduling-an-interview-meeting) | Creates the Contact when missing | `You must define a Contact Name for this applicant.` |
| Open the message dialog | One or several Applications | A dialog | none | none |
| Send the messages | The dialog: Applications, author, subject, body, attachments, optional template | none | REC-212 and the per-recipient posting | REC-211, shown as a notification without writing anything |
| Open the text-message composer | One or several Applications | Opens the composer in bulk mode with the log-keeping option on | none | none |
| Send the written interview | Exactly one Application | Opens the invitation dialog | Creates the Contact when missing; checks the questionnaire's validity | `Please provide an applicant name.`, plus the questionnaire engine's own validity refusals |
| Print the written interview | Exactly one Application | Opens the printable answers in a dialog | none | none |
| Add to job, from the matching list | One or several Applications and a position named in the reading context | Opens the applications of that position | REC-160 | none |
| Find matching positions | Exactly one Application | Lists the positions with their match score against that person | none | none |

## 4. Named operations on the other entities

| Entity | Operation | Input | Result | Side effects |
|---|---|---|---|---|
| Job Position | Open the applications | One position | The applications of the position, filtered to non-talent records, with the position as the default for new ones | none |
| Job Position | Open the activities | One position | The applications of the position in the activity view, filtered to applications whose stage is not a hired stage | none |
| Job Position | Open the documents | One position | The attachments of the position and of its applications | none |
| Job Position | Open the related employees | One position | The employees of the position, grouped by position. The reduced public employee entity is used when the reader may not read employees | none |
| Job Position | Open the trackers | One position | The Recruitment Sources of the position | none |
| Job Position | Create the interview form | One position | Opens the new questionnaire | Creates a questionnaire titled `Interview Form: ` followed by the position name and links it to the position |
| Job Position | Test the interview form | One position | Opens a test run of the questionnaire | Creates a test answer set, owned by the questionnaire engine |
| Job Position | Search matching applicants | One position | The applications of other positions holding at least one required skill, scored against this position | none |
| Job Position | Load the demonstration scenario | none | Reloads the screen | Loads the data set of [workflows.md](workflows.md#26-loading-the-demonstration-scenario) |
| Job Position | Open the public page | One position | Opens the position's public page | none |
| Recruitment Source | Create the address | One or several sources | none | Creates the inbound alias of every source that has none, with the values of REC-085 |
| Recruitment Source | Create the address and return it | Exactly one source | The full address, for the copy control | The same |
| Talent Pool | Open the talents | One pool | The talents of the pool | none |
| Talent Pool | Create a talent | One pool | A blank Application form carrying that pool | none |

## 5. Public routes

Every route below belongs to the public job pages companion. The access column states who
may reach it.

| Path | Kind | Access | Purpose | Request | Answer |
|---|---|---|---|---|---|
| `/jobs` and `/jobs/page/<page number>` | page | public | The job list | Optional values: country, a switch asking for every country, department, office address, employment type, a remote switch, an "other department" switch, an "unspecified employment type" switch, industry, an "unspecified industry" switch, a switch disabling tolerant search, the page number and the search text | The rendered list with the positions of the page, the pager, the search text actually used, and one counter map per filter |
| `/jobs/add` | remote call | signed-in internal user | Creates a position from the site editor | none | The relative address of the new position's page. The position is created with the name `Job Title` |
| `/jobs/detail/<position>` | page | public | Compatibility address | none | A permanent redirection to `/jobs/<position>` |
| `/jobs/<position>` | page | public | The detail page of one position | none | The rendered page, including the structured job-posting description |
| `/jobs/apply/<position>` | page | public | The application form | none | The rendered form, pre-filled from the values kept in the session when a previous submission failed |
| `/website_hr_recruitment/check_recent_application` | remote call | public | The live duplicate warning | The key name, one of `name`, `email`, `phone` or `linkedin`; the value typed; the position identifier | One message, or none, following REC-099 |
| `/website/form/<entity>` | submission | public | The generic public form endpoint, used with the Application entity | The seven writable names of REC-086, plus any custom entry and any file | The identifier of the created Application, or a description naming the fields in error |
| `/job-thank-you` | page | public | The confirmation page | none | The rendered page, showing the recruiter block of the position of the last submission recorded in the session |

The job list is offered to search engines through the site map, which lists the path `/jobs`
once; the detail page and the application form of each position are listed individually.

## 6. Addresses the domain builds

### 6.1 The public address of a Job Position

```formula
relative address = "/jobs/" + the position's readable identifier
absolute address = the site's base web address + relative address
                   ( or the base web address + "/jobs" when the position has no
                     readable identifier yet )
```

The readable identifier is produced by the website engine from the position's name and its
numeric identifier, for example `sales-manager-3`.

### 6.2 The inbound address of a Job Position

```formula
inbound address = the position's local part + "@" + the company's alias domain
```

The local part is typed by the Officer when the position is created and may be changed
afterwards. A position with no local part has no working inbound address.

### 6.3 Per-source tracking address

A Recruitment Source exposes a tracking address that stamps the campaign, the medium and the
source on every application produced through it. It exists only when the public job pages
companion is installed.

**Inputs.** The base web address of the position's site; the position's relative public
address; the name of the shipped campaign; the name of the source's medium, or the name of
the shared medium `website` when the source names none; the name of the source's Tracking
Source.

```formula
tracking address = base web address
                 + the position's relative public address
                 + "?"
                 + the three parameters, joined by "&", each written as
                   its name, "=", and its value with the characters that are
                   not permitted in an address replaced by their escaped form:

                   utm_campaign = the name of the shipped campaign
                   utm_medium   = the source's medium name, or "website"
                   utm_source   = the Tracking Source's name, or nothing
```

The three parameter names are the literal names of the attribution convention and must be
reproduced exactly; they carry, in order, the campaign, the medium and the source.

**Worked example.** The base web address is `https://example.com`, the position's public page
is `/jobs/sales-manager-3`, the source is named `LinkedIn` and its medium is `website`. A
space in a value is escaped as a plus sign, so the campaign name `Job Campaign` becomes
`Job+Campaign`:

```formula
https://example.com/jobs/sales-manager-3?utm_campaign=Job+Campaign&utm_medium=website&utm_source=LinkedIn
```

A visitor arriving through that address and applying produces an Application carrying the
campaign `Job Campaign`, the medium `website` and the source `LinkedIn`, because the
attribution values of the visit are read at creation by the shared attribution definition.

### 6.4 The inbound address of a Recruitment Source

```formula
local part = ( the position's own local part, or the position's name when it has none )
             + "+"
             + the source's name

inbound address = local part + "@" + the alias domain of the position's company,
                  or of the acting user's company when the position's company has none
```

**Worked example.** The position `Sales Manager` has the local part `sales-manager`, the
source is named `Indeed`, and the company's alias domain is `example.com`. The address is
`sales-manager+Indeed@example.com`. A position with no local part produces
`Sales Manager+Indeed` as the local part.

## 7. Reports and printable documents

| Report | Shape | Content | Grouping and totals |
|---|---|---|---|
| Recruitment Analysis | Graph and matrix over Applications | The count of Applications, and the averages of the expected and proposed salaries when those measures are added | Default: creation month on the rows and Job Position on the columns. Three shipped layouts: by recruiter, by position and by department, each crossed with the creation month. Filters include the creation year, unassigned, new (stage sequence 1), ongoing, refused and archived |
| Recruitment Analysis for one department | The same views | The same, restricted to one department | Opened from the department card |
| Recruitment Analysis for one position | The same views | The same, restricted to one position, with the creation-month filter applied | Opened from the position card |
| Pipeline analysis | Graph and matrix reachable from the applications screen | The count by stage; positions crossed with stages | Read in stage order, the count per stage is the hiring funnel |
| Source analysis | The analysis screens grouped by source and by medium | The count of Applications per source, per medium and per campaign, crossed with the application status | Built by grouping; no separate screen is shipped |
| Time in stage | The stage-duration values carried by each Application | The seconds spent per stage, from which an average per stage and per position is derived | Built by grouping on the stage |
| Written interview answers | A printable view of one answer set | The questions and the answers of one candidate, or the blank questionnaire when no answer set exists | Owned by [Learning, Surveys and Gamification](../learning-surveys-and-gamification/README.md); which answer set is chosen is specified in [state-machines.md](state-machines.md#52-which-answer-set-is-printed) |
| Documents | A list of attachments | Curricula vitae and other files, with the applicant's name and the creation date, searchable by indexed content | none |

**This domain ships no printable document of its own.** There is no application sheet and no
position sheet to print; the only printable output is the written interview, which belongs to
the questionnaire engine.

## 8. Message subtypes

A message subtype decides which followers are notified by which event, and whether a follower
is subscribed to it by default.

| Subtype | Carried by | Hidden | On by default | Used for |
|---|---|---|---|---|
| New Applicant | The Application | yes | no | The creation message of an ordinary Application |
| New Talent | The Application | no | no | The creation message of a talent |
| Stage Changed | The Application | no | no | The tracked change written when the stage changes |
| Applicant Hired | The Application | no | **yes** | Offered to followers who want to hear about hires |
| Job Position created | The Job Position | yes | no | The creation message of a position |
| New Applicant | The Job Position | no | no | The parent of the Application-level subtype, linked through the position reference, so that a follower of the position hears about new applications |
| Applicant Stage Changed | The Job Position | no | no | The parent of *Stage Changed*, linked the same way |
| Applicant Hired | The Job Position | no | **yes** | The parent of the Application-level *Applicant Hired* |
| Job Position Created | The Department | no | no | The parent used when a position is created in a department, linked through the department reference |

The parent links are what make a follower of a Job Position receive the events of its
Applications without following each Application individually.

## 9. Notifications and messages

| Event | Who receives it | What is sent |
|---|---|---|
| An Application is created | Followers of the position subscribed to the parent subtype | The creation message, with the subtype *New Applicant*, or *New Talent* for a talent |
| An Application changes stage | Followers subscribed to *Stage Changed*, and followers of the position subscribed to *Applicant Stage Changed* | The tracked change naming the old and the new stage |
| An Application enters a stage carrying a template | The candidate's Contact | The rendered template, posted as an internal note with the light layout without background. The acknowledgement template on the stage `New` is the shipped example |
| A user is named interviewer on an Application | That user, unless it is the acting user | Subject `You have been assigned as an interviewer for ` followed by the Application's display name; body `You have been assigned as an interviewer for the Applicant ` followed by the applicant's name |
| An Application is refused with the message option | The candidate's Contact | The rendered refusal template, per recipient, in the recipient's language |
| An Application is refused as a duplicate | Nobody; a log line is written | `Refused automatically because this application has been identified as a duplicate of ` followed by a link to the original |
| A written interview is sent | The candidate's Contact | The invitation template, with the answer link and the deadline, sent immediately rather than queued, plus two entries in the Application's thread |
| A written interview is completed | Followers of the Application | `The applicant "` + the applicant's name + `" has finished the survey.`, authored by the platform's system contact |
| An Employee is created from Applications | Followers of each Application | `Employee created: ` followed by a link bearing the Employee's name |
| A public application is submitted | Nobody | Two notes: the custom entries under `Other Information:`, and the uploaded files under `Attached files: ` |
| A message is sent from the bulk dialog | Each selected candidate's Contact | The rendered subject and body with the light layout, and a copy of each dialog attachment |
| A text message is sent | Each selected candidate's telephone number | The composed text, logged in each thread |
| The periodic digest is sent | The digest's recipients | The new-employees indicator, skipped for readers below the Officer privilege |

## 10. External integrations

| Integration | Direction | Contract |
|---|---|---|
| Inbound electronic mail to a Job Position | inbound | Any message sent to the position's address creates an Application carrying the alias default values. The contract is the address and the parsing described in [workflows.md](workflows.md#4-an-application-arriving-by-electronic-mail) |
| Inbound electronic mail to a Recruitment Source | inbound | The same, with the source's campaign, medium and Tracking Source stamped in addition |
| External job boards | inbound | A board forwards applications from its **own** address. The contract is the board's sender address and the extraction pattern; the three shipped boards are `jobs-listings@linkedin.com`, `cs@jobsdb.com` and `no-reply@indeed.com` |
| Tracking addresses published on external boards | outbound | The address of §6.3. What crosses the boundary is the three attribution parameters |
| The public application form | inbound | The generic public form endpoint with the seven writable names of REC-086, any number of custom entries and any number of files |
| The live duplicate check | inbound | A remote call taking a key name, a value and a position identifier, and answering one message or none |
| Text messages | outbound | The platform's text-message gateway, addressed to the sanitised telephone number, skipping blacklisted numbers |
| Search engines | outbound | The site map lists the job list and each published position; each detail page carries a structured job-posting description with the employment type, the publication date, the title, the direct-application flag, the hiring organisation's name and logotype, and either the workplace address or the remote indication with the company's country |

## 11. Import and export

The domain adds no import or export format of its own. Three things are worth stating for a
rebuild:

1. **Every entity of the folder is importable and exportable through the platform's generic
   mechanism.** The identifiers a file must carry are the ones in
   [entities.md](entities.md); the external identifiers of the shipped records are in
   [configuration.md](configuration.md#5-default-records-shipped-with-the-domain).
2. **Importing Applications bypasses no rule.** The creation rules run, which means the
   Contact synchronisation runs, the Interviewer privilege is granted, the canonical pool copy
   of a pooled record is pointed at itself, and interviewers are notified. An import of many
   records therefore produces many notifications; an installation that does not want them
   removes the interviewers from the file.
3. **The analysis screens export their own grids.** Nothing else in the domain produces a
   file.

## 12. Controls and the conditions under which they appear

| Control | Where | Shown when | Privilege | Operation |
|---|---|---|---|---|
| `Create Employee` | Application form header | The Application is active, has a hire date and has no Employee | Human Resources Officer | Create the Employee |
| `Send Interview` | Application form header | The Application is active and its position carries a questionnaire | Interviewer and above | Send the written interview |
| `Refuse` | Application form header, card menu, list header | The Application is active and is not a talent | Interviewer and above | Open the refusal dialog |
| `Restore` | Application form header | The Application is archived | Officer | Restore |
| `Create Applications` | Application form header, list header | The record is a talent (form), or the screen is not already filtered on one position (list) | Officer | Open the add-to-job dialog |
| `Add to Pool` | Application form header, card menu | The record is not a talent and is not already linked to a pool | Officer | Open the add-to-pool dialog |
| `Add Applicants` | Application list and card headers | The screen is filtered on a talent pool | Officer | Open the add-to-pool dialog |
| Employee statistic | Application form | An Employee is linked | Human Resources Officer | Open the Employee |
| Applications statistic | Application form | The application count is not zero, and is greater than one unless the record is a talent | Interviewer and above | Open the related applications |
| Meetings statistic | Application form | The record is stored | Interviewer and above | Create a meeting |
| Talent Pools statistic | Application form | The record is stored, is not a talent and is linked to a pool | Officer | Open the talent |
| `Consult Interview` | Application form | The position carries a questionnaire and the Application has at least one answer set | Interviewer and above | Print the written interview |
| Attachment counter | Application card | always | Interviewer and above | Open the attachments |
| `Schedule Interview` | Application card menu | always | Interviewer and above | Create a meeting |
| `Archive` / `Unarchive` | Application card menu | According to the active flag | Officer | Archive / Restore |
| `Delete` | Application card menu | The reader may delete | Officer | Generic deletion |
| `Send Email` | Application list and cards, as a contextual action | One or more records are selected | Interviewer and above | Open the message dialog |
| `Send SMS` | Application list and cards, as a contextual action | One or more records are selected and the text-message companion is installed | Officer | Open the text-message composer. The name of the action is reproduced as it is stored |
| Add or remove followers | Application list and cards, as a contextual action | One or more records are selected | Officer | The follower dialog of [Messaging and Activities](../messaging-and-activities/README.md) |
| `Matching Positions` | Application form, as a contextual action | The skills companion is installed | Officer | Find matching positions |
| `Search Matching Applicants` | Position form, as a contextual action | The skills companion is installed | Officer | Search matching applicants |
| `Applications` | Position card menu and statistic | always | Interviewer and above | Open the applications |
| `Activities` | Position card menu and the activity badge | always | Interviewer and above | Open the activities |
| `Trackers` | Position card menu and tab | always | Officer | Open the trackers |
| `Analysis` | Position card menu | always | Officer | The position-filtered analysis screen |
| `Interviews` | Position card menu | The position carries a questionnaire | Interviewer and above | Test the interview form |
| `Interview Form` | Position card menu | The position carries no questionnaire | Interviewer and above | Create the interview form |
| `Job Page` | Position card | The public job pages companion is installed | Interviewer and above | Open the public page |
| `Configure` | Position card | always | Officer | Opens the position form |
| Documents statistic | Position form | The document count is not zero | Interviewer and above | Open the documents |
| Employees statistic | Position form | The employee count is not zero | Interviewer and above | Open the related employees |
| Applications statistic | Position form | always | Interviewer and above | Open the applications, with archived records visible |
| `New Talent` | Talent pool card | always | Officer | Create a talent |
| `Refuse` / `Cancel` | The refusal dialog | always | Whoever opened it | Apply the refusal / close |
| `Send` / `Cancel` | The message dialog | always | Whoever opened it | Send the messages / close |
| `Create Applications` / `Discard` | The add-to-job dialog | always | Officer | Add applicants to positions / close |
| `Add to Pool` / `Cancel` | The add-to-pool dialog | always | Officer | Add applicants to pools / close |
| `Apply Now!` | Public detail page | always | Any visitor | Opens the public application form |
| `I'm feeling lucky` | Public application form | always | Any visitor | Submits the form |

## 13. Reporting measures and groupings

| Measure | Aggregation | Available on |
|---|---|---|
| Count of Applications | Sum of records | Every analysis screen |
| Expected salary | Average | The analysis screens, for the Officer privilege only |
| Proposed salary | Average | The analysis screens, for the Officer privilege only |
| Delay to Close | Average | The analysis screens |
| Days to Open, Days to Close | Average | The analysis screens |
| Probability | Sum | The analysis screens |
| Time spent per stage | Read from the stage-duration map of each record | The stage columns of the pipeline screens |

| Grouping | What it answers |
|---|---|
| Stage | The hiring funnel: how many applications sit at each step, read in stage order |
| Job Position | Volume and outcome per opening |
| Department | Volume and outcome per organisational unit |
| Recruiter | Workload and outcome per recruiter |
| Source, medium, campaign | Where applications come from, and which channel converts |
| Creation date, by month | Intake over time |
| Hire date, by month | Hires over time |
| Last stage update | Where applications are sitting |
| Refusal reason | Why applications are lost |
| Tags and custom properties | Any classification the organisation added |
| Talent pool | The size and composition of each pool |

## 14. Reconciliation notes

| Point | Resolution |
|---|---|
| Whether the domain ships a printable document | One draft listed an interview report as a report of this domain. The printable answers belong to the questionnaire engine; this domain ships no printable document, and §7 says so. |
| The name of the text-message action | Reproduced as stored in §12, because a contextual action's name is what an integration binds to. |
| The public form's submission control | One draft did not name it. Its label is reproduced in §2.8 and §12. |
| The route that answers the live duplicate check | Both drafts describe the same three inputs and the same four answers; §5 states the path and REC-099 the answers. |
| Whether the analysis screens expose the salary measures to everyone | They are field-restricted like the fields themselves, so a reader below the Officer privilege does not see the measures at all. §13. |
| The tracking address parameters | One draft wrote the three parameter names in prose. They are reproduced in code font in §6.3, because an external board keys on them character for character. |
