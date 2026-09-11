# Recruitment — Workflows

This file gives the end-to-end operational procedures of the domain, step by step, naming
the role that performs each step, the preconditions, the records created or updated at each
step, and the failure conditions. Formulas referred to here are specified in
[calculations.md](calculations.md); validations and error messages in
[business-rules.md](business-rules.md); access rights in
[configuration.md](configuration.md).

## Roles used in this file

| Role | Privilege held | What the role may do in this domain |
|---|---|---|
| Administrator | Recruitment Administrator | Everything an Officer may do, plus configuring stages, activity plans and job boards. |
| Officer | Recruitment Officer | Create, read, update and delete every application, job position and talent pool; see the salary fields; refuse; hire; manage pools. |
| Interviewer | Recruitment Interviewer | Read and update **only** the applications of the positions they are attached to and the applications they are individually attached to; never create or delete applications; never see the salary fields; may refuse; may schedule meetings; may not create an employee. |
| Human Resources Officer | Human Resources Officer | Create the Employee record. The *Create Employee* control is shown only to this role. |
| Internal user | none of the above | May read job positions and recruitment sources; sees no application. |
| Candidate | none — a public visitor or an inbound message sender | Submits the public form, answers the written interview, replies to messages. |
| System | the scheduled and inbound-message machinery | Converts inbound messages into applications, sends queued messages. |

---

## 1. Opening a Job Position

**Performed by:** Officer or Administrator.
**Precondition:** a Department exists if the position is to belong to one.

1. The Officer creates a Job Position with at least a name. The following defaults apply:
   - *Target* (the number of people to hire) is 1;
   - *Recruiter* is the acting user;
   - *Company* is the acting user's current company;
   - *Job Location* is the location of the most recently created position among the acting
     user's companies, or, when there is none, the contact record of the acting user's
     current company;
   - *Active* is true;
   - *Sequence* is 10;
   - the favourites list is emptied by the creation rule, so the position is not
     automatically a favourite of its creator.
2. The system creates the position's inbound electronic mail alias, with the Application
   entity as the target model and with default values carrying the position, its
   department, the company (the department's company when it has one, otherwise the
   position's company) and the recruiter. From now on, any message sent to that address
   creates an Application already attached to the position.
3. The system grants the Interviewer privilege to every user named in the position's
   interviewer list.
4. The uniqueness rule is checked: the triple (name, company, department) must not already
   exist. Failure message: `The name of the job position must be unique per department in company!`
5. The database check on the target is applied: the target must not be negative. Failure
   message: `The expected number of new employees must be positive.`

**Optional configuration, each performed by the Officer:**

| Step | Effect |
|---|---|
| Name interviewers on the position | Those users may see every application of the position and are granted the Interviewer privilege. |
| Attach a written interview questionnaire | Every application of the position may be sent that questionnaire. |
| Declare required skills and an expected degree | Applications get a match score against the position. |
| Declare custom application properties | Every application of the position carries those extra fields. |
| Add Recruitment Sources | Each source can own a dedicated inbound address and exposes a tracking address for public links. |
| Publish the position | The position appears on the public job list and its public page becomes reachable. |

**Postcondition:** the position exists, is reachable by electronic mail, and its remaining
target states how many people are still to be hired.

---

## 2. Configuring the pipeline

**Performed by:** Administrator (stages are writable only by the Administrator privilege).

1. The Administrator opens the stage configuration. The six shipped stages are present.
2. Creating a stage from inside one position's pipeline produces a **global** stage, because
   the default-position context entry is stripped before defaults are computed, unless the
   caller sets the single-position flag. To restrict a stage, the Administrator names the
   positions explicitly in *Job Specific*.
3. Setting *Hired Stage* on a stage makes entering it the hire event. Clearing that flag on
   a stage that currently holds applications raises the form warning driven by the derived
   flag *Is Warning Visible*; confirming the change clears the hire date of every
   application sitting in that stage on the next recomputation, and does **not** give the
   consumed targets back.
4. Setting *Days to rot* to a non-zero number enables the staleness report for that stage.
5. Attaching a message template to a stage makes every entry into that stage post a message
   built from the template.

---

## 3. Creating an application by hand

**Performed by:** Officer (an Interviewer cannot create).

1. The Officer opens the applications view and creates a record, giving at least the
   applicant's name (required on the form).
2. On save:
   - if a recruiter was given, the assignment moment is stamped;
   - the electronic mail address is stripped of surrounding whitespace;
   - the company, the department, the stage and the recruiter are computed from the
     position as specified in [entities.md](entities.md#24-computed-fields-in-detail);
   - the Contact synchronisation runs if an address was given (see §12);
   - the Interviewer privilege is granted to any named interviewer, and each named
     interviewer other than the acting user is notified.
3. The message subtype *New Applicant* (or *New Talent* for a pooled record) is used for the
   creation message, so followers of the position who subscribed to the corresponding
   parent subtype are notified.

---

## 4. An application arriving by electronic mail

**Performed by:** the candidate (who simply sends a message) and the System.
**Precondition:** the position has an inbound alias, and the message is addressed to it.

### 4.1 Routing

1. The inbound message is received and parsed into a message description: sender header,
   normalised sender address, subject, body, message identifier, references, attachments,
   and any recognised priority.
2. The routing layer resolves the destination address to the position's alias, and therefore
   to the Application entity, with the alias's default values as the custom values: the
   position, the department, the company and the recruiter.
3. Because the message does not reply to an existing thread, the "create a new record"
   path is taken.

### 4.2 Creating the Application

The recruitment domain overrides that path as follows:

1. The default recruiter is forced to *empty* for this creation. This is deliberate: the
   inbound-message machinery runs as a technical user, and without this the technical user
   would become the responsible recruiter. The alias default may still supply a recruiter.
2. If the custom values name a position, the landing stage is resolved: the first stage in
   ascending sequence that is unrestricted or attached to that position. **On this path the
   folded filter is not applied** (see [state-machines.md](state-machines.md#13-choosing-the-landing-stage)).
3. The sender header is parsed into a display name and a normalised address.
4. The default applicant's name is set to the parsed display name.
5. The normalised sender address is looked up in the Job Board list.
   - **No job board matches** (the ordinary case): the Application's electronic mail address
     is set to the raw sender header, and the Contact is set to the author contact the
     messaging layer resolved, if any.
   - **A job board matches**: the extraction pattern of that job board is applied first to
     the subject and then to the body; all captures are concatenated in that order and the
     **first** capture becomes the applicant's name. If nothing is captured, the parsed
     display name is kept. The sender address is then **removed** from the message
     description, so the Application ends up with no electronic mail address and no Contact.
6. If the message carried a recognised priority, it becomes the Application's evaluation
   priority.
7. If a landing stage was resolved, it is set.
8. The custom values from the alias are applied last and therefore win over everything
   above.
9. The generic creation then fills the display field from the message subject **only if the
   applicant's name is still empty**, and fills the primary electronic mail field from the
   message's sender address **only if it is still present in the message description** —
   which is exactly why the job-board branch removes it.
10. The record is created. All the ordinary creation rules of §3 step 2 run.
11. Immediately after creation, the address-and-telephone computation is re-run explicitly,
    so that an Application that received a Contact from the messaging layer picks up that
    Contact's address and telephone number.

### 4.3 Attachments and the thread

1. The message is posted on the new Application; its attachments — the curriculum vitae, the
   cover letter — are stored as attachments of the Application.
2. The first suitable document becomes the Application's main attachment and is shown in
   the side preview panel of the form for every user (the preview panel is governed by a
   dedicated group implied for all internal users).
3. The extracted text content of the attachment is indexed, so the recruiter can later
   search applications by résumé content.
4. The reply address for any answer posted on the Application is the alias of its position.

### 4.4 Company resolution for inbound applications

The company is not taken from the technical user. It is computed as specified in
[entities.md](entities.md#243-company): the department's company when it matches the
position's company, otherwise the position's company, otherwise the acting user's company.
A position in a second company whose department has no company therefore still produces an
application in the position's company.

### 4.5 Worked outcome

An inbound message from `Mr. Richard Anderson <Richard_Anderson@yahoo.com>` to the alias of
the position *Experienced Developer*, carrying one attachment named `resume.pdf`, produces:

| Field | Value |
|---|---|
| Applicant's Name | `Mr. Richard Anderson` |
| Email | `Mr. Richard Anderson <Richard_Anderson@yahoo.com>` (the raw header) |
| Normalized Email | `richard_anderson@yahoo.com` |
| Job Position | Experienced Developer |
| Department | the position's department |
| Company | the position's company |
| Stage | *New* |
| Attachments | one, named `resume.pdf`, with its text content indexed |
| Applied on | the moment the message was processed |

A message from the shipped job board *Linkedin*, address `jobs-listings@linkedin.com`, with
the subject `New application: ERP Implementation Consultant from John Doe`, produces an
Application whose name is `John Doe`, whose electronic mail address is **empty** and whose
Contact is **empty**.

---

## 5. An application arriving through the public form

**Performed by:** the candidate (a public visitor) and the System.
**Precondition:** the public job pages companion is installed and the position is published.

### 5.1 What the visitor sees

1. The visitor opens the public job list. The list shows published positions, paged twelve
   per page, and offers filters by country, office, department, employment type and
   industry, plus a text search over the position name and description.
2. The visitor opens a position's public page, which renders the position's website
   description and its process details.
3. The visitor opens the application page. The form offers:

| Control | Submitted name | Required | Target |
|---|---|---|---|
| Your Name | `partner_name` | yes | the Application's applicant name |
| Your Email | `email_from` | yes | the Application's electronic mail address |
| Your Phone Number | `partner_phone` | yes | the Application's telephone number |
| LinkedIn Profile | `linkedin_profile` | no | the Application's professional network profile address |
| Resume (a file) | `Resume` | no | an attachment on the Application |
| Short Introduction | `short_introduction` | no | a custom entry, not a field |
| Job (hidden) | `job_id` | filled from the page | the position |
| Department (hidden) | `department_id` | filled from the page | the department |

### 5.2 Live duplicate warning, before submission

While the visitor types, the page asks the server whether a recent application already
exists. The question carries one of four field names — name, electronic mail address,
telephone number, professional network profile — its value, and the position identifier.
The server answers with a single message, or with nothing:

1. Build the matching condition for the given field: the applicant name matched
   case-insensitively; or the normalised electronic mail address matched exactly; or the
   telephone number matched exactly; or the professional network profile matched
   case-insensitively. An unknown field name yields a condition that matches nothing.
2. Intersect it with: the position belongs to this website or to no website, and the
   application status is `ongoing` or `refused`.
3. Read the matches with elevated rights, newest first, and group them by status.
4. **If any refused match is inactive, belongs to this very position, and was created within
   the last six months**, answer:
   `We've found a previous closed application in our system within the last 6 months. Please consider before applying in order not to duplicate efforts.`
5. Otherwise, if there is no ongoing match, answer nothing.
6. Otherwise take the newest ongoing match:
   - **If it belongs to this very position**, answer
     `An application already exists for ` + the submitted value + `. Duplicates might be rejected.`
     followed, when that application has a recruiter, by
     ` In case of issue, contact ` and the recruiter's name, address and telephone number,
     joined by commas, omitting the empty ones.
   - **Otherwise** answer
     `We found a recent application with a similar name, email, phone number. You can continue if it's not a mistake.`

The warning never blocks the submission. It is advisory.

### 5.3 Submission

1. The form is posted to the generic public form endpoint for the Application entity. The
   endpoint verifies that the entity is declared as accepting public form submissions.
2. A partial forgery check is applied: the token is validated **only when the visitor has a
   signed-in session**. An anonymous submission is accepted without a token, because
   embedded forms routinely lose their session cookie. A failed check for a signed-in
   visitor is refused with `Session expired (invalid CSRF token)`.
3. A challenge-response check is applied according to the platform's public-form challenge
   configuration.
4. The submitted values are sorted into three buckets:
   - **record values**: entries whose name is in the writable-field list declared for the
     Application entity, namely the electronic mail address, the applicant's name, the
     telephone number, the position, the department, the professional network profile
     address and the custom application properties. Each is converted according to its type
     (references become whole numbers, files become encoded binary, and so on).
   - **attachments**: uploaded files whose name is not a binary field of the entity.
   - **custom entries**: everything else, kept as a `name : value` text block.
5. The recruitment domain then filters the record values:
   - if a Contact was somehow submitted, the electronic mail address and the telephone
     number are dropped, so that a submitted Contact is never overwritten by form input;
   - if a position was submitted, the position is loaded with elevated rights and its
     active flag checked. **A submission for an archived position is refused with**
     `The job offer has been closed.`;
   - the landing stage is resolved as the first non-folded stage, in ascending sequence,
     that is unrestricted or attached to the submitted position, and written into the record
     values.
6. Required fields that are declared required on the entity and absent from the submission
   cause the submission to be rejected with the list of offending field names.
7. The Application is created with full rights and with automatic subscription of the
   creator suppressed.
8. The custom entries are turned into a log message on the Application, because the
   Application entity declares no default text field for form overflow. The label
   `short_introduction` is replaced by the readable label
   `Short introduction from applicant` before the block is built. When the platform-wide
   metadata switch is on, the visitor's network address, browser identification, accepted
   languages and referring page are appended under the heading *Metadata*.
9. The uploaded file is stored as an attachment of the new Application. Because the field
   name `Resume` is not a field of the entity, the file is an **orphan attachment**: it is
   attached to the record and announced by a log message whose body is `Attached files: `.
10. The Application is **not** given an authentication log line: the domain suppresses the
    "this record was created by an authenticated user" log line for applications created by
    a visitor with no session.
11. The visitor is redirected to the thank-you page. That page is excluded from the page
    cache, so each candidate sees a freshly rendered confirmation.

### 5.4 What the recruiter sees afterwards

A new Application in the landing stage of the position, with the candidate's name, address,
telephone number and professional network profile filled, the résumé attached and
previewable, the short introduction visible as the first log message, and a Contact created
or reused by the address synchronisation of §12.

---

## 6. Moving an application through the pipeline

**Performed by:** Officer or Interviewer (an interviewer may write on the applications they
can reach).

1. The user drags the card to another column, clicks the stage in the progress bar of the
   form, or edits the stage in a list.
2. The platform applies, in order, the write rules of
   [entities.md](entities.md#26-modification-rules): the last-stage-update moment, the
   readiness reset, the previous-stage record, the target adjustment, then the write itself,
   then the hire-date recomputation.
3. A tracked-change entry records the stage change in the thread, under the subtype *Stage
   Changed*. Followers of the position who subscribed to the parent subtype *Applicant Stage
   Changed* are notified.
4. **If the destination stage carries a message template**, a message rendered from that
   template is posted on the Application, addressed to the candidate, as an internal note
   with the light layout. This is the mechanism that sends the acknowledgement message when
   an application reaches *New*.

### 6.1 Worked example — the acknowledgement message

Given the shipped configuration, where the *New* stage carries the template *Recruitment:
Application Acknowledgement*:

1. An Officer creates an Application by hand with no position, then sets the position
   *Experienced Developer*. The stage is computed to *New* at that moment. Because the stage
   was **computed at creation** rather than written afterwards, the template message is not
   sent.
2. An Officer takes an existing application sitting in *Qualification* and drags it back to
   *New*. The stage is **written**, so the template message is sent: subject
   `Your Job Application: Experienced Developer`, addressed to the candidate's Contact,
   body confirming receipt of the application, naming the company, offering the link to the
   public job page when the position is published, naming the recruiter with their address
   and telephone number when a recruiter is set, and printing the job location's address
   block.
3. The message is recorded in the thread as a note and is sent to the candidate's Contact.

### 6.2 Readiness colour

Inside a stage, the user sets the readiness colour to *Ready for Next Stage*, *Waiting* or
*Blocked*. Each change refreshes the last-stage-update moment and therefore restarts the
staleness clock. The labels shown are the four label fields of the current stage.

---

## 7. Refusing an application

**Performed by:** Officer or Interviewer.

### 7.1 Opening the dialog

The *Refuse* control on the form, the *Refuse* entry in the card menu, or the *Refuse*
header button of the list opens the refusal dialog with the selected applications, with
archived records included in the selection context, and with the template-management
options hidden.

### 7.2 Filling the dialog

1. The reason defaults to the **first** Refusal Reason in sequence order.
2. Choosing a reason adopts that reason's message template — unless the template is
   archived, in which case no template is adopted.
3. Adopting a template copies its subject, its body, its attachments and its scheduled date
   into the dialog. When exactly one application is selected, the subject and body are
   rendered against that application immediately, so the user sees the final text.
4. The *Send Email* switch is set automatically to true when a template is adopted, the
   template is active, and **every** selected application has an address. It is set to false
   otherwise.
5. When at least one selected application has neither its own address nor an address on its
   Contact, the dialog shows the sentence
   `You can't select Send email option.` then, on the next line,
   `The email will not be sent to the following applicant(s) as they don't have an email address:`
   followed by the comma-separated names of those applications.
6. The dialog counts the **duplicates**: other applications that match the selection by any
   of the three duplicate keys or by sharing a canonical pool copy, that are not themselves
   selected, and whose status is neither hired nor refused nor archived. Turning on *Refuse
   Duplicate Applications* loads them into an editable list; turning it off empties the list.

### 7.3 Applying

1. **Guard:** if the message option is on and the acting user has no electronic mail
   address, the operation is refused with
   `Unable to post message, please configure the sender's email address.`
2. **Guard:** if the message option is on and any selected application has neither its own
   address nor an address on its Contact, the operation is refused with
   `At least one applicant doesn't have a email; you can't use send email option.`
3. The set of applications to refuse is the selection. If the duplicate switch is on and the
   duplicate count is not zero, the duplicate list is added to it, and, for every duplicate,
   the original it matched is resolved (see
   [calculations.md](calculations.md#42-refusal-duplicate-set)) and a log line is written on
   the duplicate reading
   `Refused automatically because this application has been identified as a duplicate of ` +
   a link bearing the original's display name.
4. Every application of the combined set is written in one operation with: the chosen
   refusal reason, the active flag set to false, and the refusal moment set to the current
   moment.
5. **If the message option is on**, one message per *selected* application (not per
   duplicate) is posted, each rendered in the recipient's language:
   - the language is resolved per application from the template's language expression;
   - the subject and the body are rendered against that application in that language;
   - the sender is the template's own sender address when it defines one, otherwise the
     acting user's formatted address;
   - the author is the acting user's contact;
   - the scheduled date of the dialog is carried over, so a template with a scheduled date
     delays the delivery;
   - the dialog's attachments are linked to the message;
   - the recipient is the application's Contact.
6. The dialog closes.

### 7.4 Consequences

| Consequence | Detail |
|---|---|
| Derived status | `refused` |
| Visibility | The application disappears from the default pipeline; it is found again with the *Refused* filter |
| Stage | Unchanged |
| Hire date | Unchanged — refusing a hired application leaves the hire date, and the derived status becomes `refused` because the refusal branch is evaluated first |
| Position target | Unchanged — refusal never gives a target back |
| Stage clock | Stopped: the time spent in the current stage is reduced by the interval between the refusal moment and now, every time the durations are read |
| Duplicate applications | Refused too, when the switch was on, each with the explanatory log line |

### 7.5 Worked example — a refusal with a reason

An Officer selects the application of *Laurie Poiret*, `laurie.poiret@aol.ru`, chooses the
reason *Does not fit the job requirements*, which carries the template *Recruitment:
Refuse*, leaves *Send Email* on, and turns on *Refuse Duplicate Applications*. There is one
other ongoing application from the same address.

Result: two applications become inactive with that reason and the current moment as the
refusal moment; the second one receives the log line naming the first; one message with
subject `Your Job Application: ` + the position name is sent to Laurie Poiret's Contact
from the acting user's address; the pipeline shows neither application any more.

---

## 8. Hiring: from accepted candidate to Employee

**Performed by:** Officer for the stage move; Human Resources Officer for the employee
creation.

### 8.1 Step one — record the hire

1. The Officer moves the application into a stage flagged as a hired stage (the shipped one
   is *Contract Signed*).
2. The hire date is stamped with the current moment.
3. The position's remaining target is decremented by one, clamped at zero.
4. The position's stored *Hired* counter is recomputed: the number of applications of the
   position that carry a hire date, active or archived.
5. The derived status becomes `hired` and the *Hired* ribbon appears on the form.

### 8.2 Step two — create the Employee

The *Create Employee* control appears on the application form only when: the application has
no employee yet, is active, and has a hire date. It is visible only to the Human Resources
Officer role.

1. **Guard:** the acting user must not be a bare Interviewer. A user who holds the
   Interviewer privilege but not the Officer privilege is refused with
   `You are not allowed to perform this action.`
2. **Contact:** if the application has no Contact, one is created with the applicant's name
   and the application's electronic mail address, as a person rather than an organisation.
   If the applicant's name is also empty, the operation is refused with
   `Please provide an applicant name.`
3. **Employee creation:** an Employee is created, with the creation context cleaned of any
   inherited default values, using the field mapping of §8.3.
4. **Attachment copying:** the attachments of the application whose binary content is not
   already present among the new employee's attachments are duplicated onto the employee.
   The comparison is on the content itself, so re-running the operation does not duplicate
   files.
5. **Post-creation write:** a second write is applied to the employee, setting the position,
   the job title, the department, the work electronic mail address and the work telephone
   number again — see §8.3 for why.
6. **Log line:** because the new employee was created with a non-empty application list, a
   log line is written on the application reading `Employee created: ` followed by a link
   bearing the employee's name.
7. The interface navigates to the new employee's form.

### 8.3 The field mapping, named field by field

The Employee is created with exactly these values. The left column is the Employee field,
the right column the source.

| Employee field (storage name) | Source on the Application |
|---|---|
| Name (`name`) | the applicant's name; if empty, the Contact's display name |
| Work Contact (`work_contact_id`) | the Application's Contact |
| Job Position (`job_id`) | the Application's position |
| Job Title (`job_title`) | the **name** of the Application's position |
| Private Street (`private_street`) | the *contact* address of the Application's Contact, street line 1 |
| Private Street 2 (`private_street2`) | the same address, street line 2 |
| Private City (`private_city`) | the same address, city |
| Private State (`private_state_id`) | the same address, state or province |
| Private Zip (`private_zip`) | the same address, postal code |
| Private Country (`private_country_id`) | the same address, country |
| Private Phone (`private_phone`) | the same address, telephone number |
| Private Email (`private_email`) | the same address, electronic mail address |
| Language (`lang`) | the same address, language |
| Department (`department_id`) | the Application's department |
| Work Address (`address_id`) | the contact record of the Application's **company** |
| Work Email (`work_email`) | the electronic mail address of the **department's company** when it has one, otherwise the Application's own electronic mail address |
| Work Phone (`work_phone`) | the telephone number of the department's company |
| Applicants (`applicant_ids`) | this Application |
| Phone (`phone`) | the Application's telephone number |
| Skills (`employee_skill_ids`) | one employee skill line per application skill line, carrying the skill, the level and the skill type (present when the skills companion is installed) |

The *contact* address referred to above is obtained by asking the Application's Contact for
its address of the *contact* kind, resolved with elevated rights so that private address
fields can be read.

**The second write.** After the employee exists, the operation writes the position, the job
title, the department, the work electronic mail address and the work telephone number a
second time, with the same sources. This is not redundant in effect: the Employee entity
recomputes several of those fields from the employment version created alongside the
employee, and the second write re-imposes the values taken from the application. A rebuild
must apply both writes in this order, because the observable result of a single write
differs.

**Skill transfer.** The skill lines are passed as part of the creation values, so the new
employee starts with the skills claimed on the application, each at the level recorded
there. Validity windows are **not** carried over; the employee skill lines are created
without dates.

### 8.4 Worked example

An application for *Experienced Developer* in the department *Research & Development* of
the company *Company Test*, whose candidate is *Sharlene Rhodes*, address
`sharlene@example.com`, telephone `+1 650-123-4567`, with a Contact whose contact address
is `250 Executive Park Blvd, Suite 3400, San Francisco, California 94134, United States`,
and whose department's company has the address `info@companytest.com` and the telephone
`+1 555-0100`, produces an Employee named `Sharlene Rhodes`, with job title
`Experienced Developer`, department *Research & Development*, private address copied from
the Contact, work address = the contact record of *Company Test*, work electronic mail
address `info@companytest.com` (the company's, not the candidate's, because the company has
one), work telephone `+1 555-0100`, personal telephone `+1 650-123-4567`, and the
application in its applicant list. The application's attachments, if any and if not already
present, are copied onto the employee, and the application's thread receives the line
`Employee created: Sharlene Rhodes`.

---

## 9. Archiving and restoring

**Archiving.** Performed by the Officer. The record becomes inactive. The context flag that
suppresses the stage message is set, so no message is sent. Nothing else changes: the stage,
the hire date and the refusal reason, if any, survive.

**Restoring.** Performed by the Officer. In order:

1. The record becomes active again, with the same message-suppressing flag set.
2. For each distinct position among the restored applications, the first non-folded stage,
   ascending by sequence, that is unrestricted or attached to that position is resolved.
3. Each application is written with that stage and with its refusal reason cleared. An
   application with no position receives an empty stage.
4. Because the stage is **written**, the full stage-write machinery runs: the
   last-stage-update moment is refreshed, the readiness colour is reset, the previous stage
   is recorded and the target arithmetic applies. Because the suppressing flag is set, no
   template message is sent.

**Archiving a Job Position** deactivates every application of the position, and, when the
public job pages companion is installed, unpublishes it.

---

## 10. Talent pools

### 10.1 Adding people to a pool

**Performed by:** Officer (the dialog is not accessible to the Interviewer privilege).

1. The Officer selects applications in a list or a card view and chooses *Add to Pool*, or
   opens the dialog from a pool.
2. The dialog offers only applications that are **either already talents or not linked to
   any pool**, so that the same person cannot be pooled twice through two different copies.
3. The Officer names the pools and, optionally, tags.
4. Applying, with elevated rights:
   - **For a selected record that is already a talent:** the named pools and the named tags
     are linked onto it. No copy is made.
   - **For a selected record that is not a talent:** a copy is made with the position
     cleared, the named pools set, and the tags set to the union of the original's tags and
     the named tags. The copy is made with the "no copy suffix" flag, so the name is not
     changed. The copy's canonical pool copy is then pointed at itself, and the original's
     canonical pool copy is pointed at the copy.
5. If exactly one talent results, the interface opens it; otherwise the current view is
   reloaded.

### 10.2 Keeping the talent in step

Whenever a pooled application — one whose canonical pool copy is another record — is
written, the following fields are copied onto the talent if they were part of the write:
the electronic mail address, the telephone number, the professional network profile address
and the degree. When the skills companion is installed, skill line changes are translated
first, because the skill line identifiers differ between the two records: an update of a
skill the talent already has becomes an update of the talent's own line; an update of a
skill the talent does not have becomes a creation on the talent; a deletion becomes a
deletion of the talent's corresponding line, if it has one; anything else is passed through
unchanged.

### 10.3 Pushing talents into positions

**Performed by:** Officer.

1. The Officer selects talents and chooses *Create Applications*, or uses the same control
   from a talent's form.
2. The dialog asks for one or more positions.
3. Applying: the selected records are read as creation values with the "no copy suffix"
   flag; for each (record, position) pair a new Application is created with the position
   set, the pool list cleared, and the stage set to the **lowest-sequence non-folded stage**
   among those that are unrestricted or attached to that position.
4. If exactly one Application results, the interface opens it. Otherwise it shows the
   notification `Created ` + the count + ` new applications for: ` + the comma-separated
   distinct names.

### 10.4 Reaching the talent from an application

The *Talent Pools* statistic button on an application opens the canonical pool copy. If the
application has no canonical pool copy yet but is nevertheless linked to a pool indirectly,
the button first resolves and stores the link by searching for a talent matching any of the
three duplicate keys, then opens it.

---

## 11. Duplicate handling in daily work

**Performed by:** Officer.

1. The application form shows an *Applications* statistic button whenever the duplicate
   count is greater than one (or whenever the record is a talent). The number shown is the
   count of applications that are the same person, including this one.
2. Clicking it opens a list of exactly those applications, with archived records included,
   grouped by stage by default, and with the create-application header button suppressed.
3. From that list the Officer can compare the applications, refuse the redundant ones — the
   refusal dialog's duplicate switch does this in one operation — or pool the person.

### 11.1 Worked example — a duplicate detected by address

Three applications are created with the same electronic mail address `test@example.com` and
the same name. The first one's application count reads **3**. Archiving the second one does
**not** change the count, because the count reads archived records too. Refusing the third
one does not change it either. Clicking the statistic button opens a list of three records.

A fourth application is created with the address `laurie.POIRET@aol.ru` while an existing
one carries `laurie.poiret@aol.ru`. Because the comparison is on the **normalised**
address, which is lowercase, the two match and both report a count of 2.

---

## 12. Contact record creation on demand

Several operations need a Contact and create one if it is missing. They do not all behave
the same way, and the differences matter:

| Operation | Guard when the applicant's name is empty | Values written on the new Contact | Rights |
|---|---|---|---|
| Writing the electronic mail address or the telephone number on an Application | Refused with `You must define a Contact Name for this applicant.` | language of the acting user, applicant's name, telephone number. An existing Contact with the same normalised address is reused rather than duplicated. | the acting user's |
| Scheduling a meeting | Refused with `You must define a Contact Name for this applicant.` | not an organisation, applicant's name, electronic mail address | the acting user's |
| Creating an Employee | Refused with `Please provide an applicant name.` | not an organisation, applicant's name, electronic mail address | the acting user's |
| Sending an interview invitation | Refused with `Please provide an applicant name.` | not an organisation, applicant's name, electronic mail address, telephone number | elevated |
| Sending a message from the message dialog | none — the Contact is created with whatever name is present | not an organisation, applicant's name, electronic mail address, telephone number | the acting user's, with the context cleaned |

Only the first of these reuses an existing Contact; the others create one unconditionally.
A rebuild must reproduce this, because it is observable: sending a message to an application
that has no Contact creates a new Contact even when one with the same address exists.

---

## 13. Scheduling an interview meeting

**Performed by:** Officer or Interviewer.

1. The user presses the meeting statistic button on the application form.
2. If the application has no Contact, one is created (see §12), failing with
   `You must define a Contact Name for this applicant.` when the name is empty.
3. The participant list is built: the Application's Contact, plus the contact of the
   department manager's login user, plus **either** the acting user's contact (when the
   acting user is a bare Interviewer) **or** the recruiter's contact (in every other case).
4. The calendar action is opened with a context that:
   - forces creation to be allowed, because a bare Interviewer has no create right on
     applications and the calendar would otherwise refuse to let them create a meeting for
     one;
   - sets the meeting's application, participants, organiser (the acting user) and name (the
     applicant's name);
   - carries the application's attachments.
5. On saving the meeting, the platform links it to the Application and duplicates every
   attachment of the Application onto the meeting, provided the acting user may read
   applications.
6. The application's meeting summary fields update: the text becomes *1 Meeting*, *Next
   Meeting* or *Last Meeting* and the date becomes the earliest future meeting or, when all
   meetings are past, the latest one.

---

## 14. Granting and revoking the Interviewer privilege

This is a side effect of ordinary editing and must be reproduced exactly, because it is how
a user with no recruitment privileges becomes able to see an application.

**Grant.** Triggered after creating an Application, after creating a Job Position, and after
writing the interviewer list of either. Given the set of users now named as interviewers:
drop those who already hold the Officer privilege, then add the Interviewer privilege to the
rest, with elevated rights.

**Revoke.** Triggered after writing the interviewer list of an Application or a Job Position,
on the users that were removed. For that set:

1. Collect every user still named as an interviewer on at least one Job Position.
2. Add every user still named as an interviewer on at least one Application.
3. Add every user holding the Officer privilege.
4. Remove the Interviewer privilege from every user of the input set that is in none of
   those three groups, with elevated rights.

Consequence, worked: a user who is an interviewer on a position **and** on one of its
applications keeps the privilege when removed from the application, and keeps it when
removed from the position while still on the application; the privilege disappears only
when the last attachment is removed.

---

## 15. Changing the recruiter of a position

**Performed by:** Officer.

1. The Officer writes a new recruiter on the Job Position.
2. The platform remembers the previous recruiter and the department manager.
3. After the write, for each position:
   - the contacts of the **previous** recruiter are unsubscribed from the position's message
     thread, except those that are related contacts of the department manager;
   - the same unsubscription is applied to the position's applications that are `ongoing`
     **and** were assigned to the previous recruiter;
   - those same applications are reassigned to the new recruiter, with assignment
     notifications suppressed.
4. Applications that are hired, refused or archived keep their old recruiter. Applications
   that had a different recruiter keep theirs.
5. Because the recruiter is one of the alias default values, the alias defaults are
   rewritten so that new inbound applications get the new recruiter.

---

## 16. Sending a message to one or several applications

**Performed by:** Officer or Interviewer.

1. The user selects applications in a list or card view and chooses *Send Email*, or uses
   the control on the form.
2. The dialog opens with the selected applications (archived ones included), the acting
   user's contact as the author, and the rendering model fixed to the Application entity.
3. **Guard:** if any selected application has no electronic mail address, or has a Contact
   with no address, the send is aborted and a danger notification is shown reading
   `The following applicants are missing an email address: ` followed by the comma-separated
   names.
4. If a template was chosen, the subject and the body are rendered per application;
   otherwise the typed subject and body are used unchanged for all of them.
5. For each application:
   - if it has no Contact, one is created with the applicant's name, the address and the
     telephone number, with the context cleaned;
   - each dialog attachment is duplicated and re-owned by that application;
   - a comment message is posted on the application, authored by the chosen author, with the
     light notification layout, addressed to the application's Contact, carrying the
     duplicated attachments.

---

## 17. The written interview

**Performed by:** Officer or Interviewer for sending; the candidate for answering.
**Precondition:** the interview companion is installed and the position has a questionnaire.

### 17.1 Attaching a questionnaire to a position

The Officer either picks an existing questionnaire of the recruitment kind, or presses *New*
on the position, which creates a questionnaire titled `Interview Form: ` + the position name
and attaches it, then opens it for editing.

### 17.2 Sending the invitation

1. The user presses *Send Interview* on the application.
2. If the application has no Contact, one is created with elevated rights from the
   applicant's name, address and telephone number; if the name is empty the operation is
   refused with `Please provide an applicant name.`
3. The questionnaire's own validity check runs (it refuses, for example, a questionnaire
   with no question).
4. The invitation dialog opens pre-filled with: this application, the candidate's Contact as
   the sole recipient, the position's questionnaire, the shipped invitation template
   *Applicant: Interview*, the light notification layout, and a deadline of **fifteen days
   from now**.
5. On sending:
   - if the application has no answer set for this questionnaire yet, one is created for the
     candidate's Contact with the dialog's answer values, and appended to the application's
     answer list, with elevated rights;
   - a message is posted on the application reading `The survey ` + a link bearing the
     questionnaire title + ` has been sent to ` + a link bearing the contact's name;
   - the invitation message is generated; its rendered body is **also** posted on the
     application, and the message is sent immediately rather than left in the queue.
6. When the invitation is re-sent in *resend* mode and the application already has an answer
   set for that questionnaire, the candidate's Contact is treated as already-answered, which
   is what makes the resend mode behave correctly.

### 17.3 Answering

The candidate follows the link in the invitation and answers. On submission the answer set
becomes complete and a message is posted on the application reading `The applicant "` + the
applicant's name + `" has finished the survey.`, authored by the platform's system contact.

### 17.4 Reading the answers

- The *Print Interview* control on the application opens the printable view of the most
  relevant answer set (see [state-machines.md](state-machines.md#52-which-answer-set-is-printed)).
- An Officer may read the answers of recruitment questionnaires that are unrestricted or
  that name them in the questionnaire's restricted-user list.
- An Interviewer may read the answers only of applications they can reach — that is, of
  applications on which they are an interviewer, or whose position names them as an
  interviewer.
- An Administrator may read, write and delete every recruitment answer.

### 17.5 Worked example

The position *Experienced Developer* carries the questionnaire *Recruitment Form*. The
Officer opens the application of *John Doe*, presses *Send Interview*, and sends the
invitation with the default deadline. The application's thread gains two entries: the
sentence naming the questionnaire and the recipient, and the rendered invitation. The
candidate receives a message subject `Participate to Recruitment Form interview` with a
button *Start the written interview* and the sentence `Please answer the interview for ` +
the formatted deadline date. When John Doe submits, the thread gains
`The applicant "John Doe" has finished the survey.` and the Officer can print the completed
answers from the application.

---

## 18. Skills

**Performed by:** Officer or Interviewer.

1. Skills are recorded on the application as lines, each naming a skill, its type and a
   level, optionally with a validity window.
2. A position declares required skills the same way, plus an expected degree.
3. The application form shows the match score against its own position; the matching-jobs
   view shows the score of one application against every position; the matching-applicants
   view shows the score of every application against one position. All three use the
   formulas of [calculations.md](calculations.md#8-skill-matching).
4. From the matching-applicants list, the Officer may press *Add to job*, which writes the
   position and the shipped *New* stage onto the selected application with the "just moved"
   flag set, so the stage's template message is **not** sent, and then opens the position's
   application view.
5. At hire, the application's skill lines are transferred to the new employee (see §8.3).

---

## 19. Sending a text message to applicants

**Performed by:** Officer or Interviewer, when the text-message companion is installed.

The list and card views expose a *Send SMS* action bound to the Application entity. It opens
the platform's text-message composer in mass mode, with the log-keeping option on and the
selected applications as the targets. The number used is the application's telephone number;
the telephone blacklist of the platform applies, and applications whose sanitised number is
blacklisted are skipped by the composer.

---

## 20. Reporting

**Performed by:** Officer.

| Report | Contents |
|---|---|
| Recruitment Analysis | Applications shown as a graph and as a matrix. Default grouping: creation month, then position. Preset groupings by recruiter, by position and by department are shipped as saved filters. |
| Per-department analysis | The same report opened from a department, pre-filtered to that department. |
| Per-position analysis | The same report opened from a position, pre-filtered to that position and to the current creation month. |
| Pipeline by position | The matrix of applications with the position on rows and the stage on columns. |
| New applications of a department | The list of applications of the department whose stage sequence is at most 1. |
| Periodic digest indicator | The count of employees created in the period, shown to Officers only. |
