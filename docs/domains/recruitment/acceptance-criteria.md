# Recruitment — Acceptance Criteria

Numbered scenarios in Given / When / Then form, with concrete starting records, concrete
inputs and the exact records, amounts, states and messages that must result. A rebuild is
correct for this domain when every scenario below passes.

Identifiers are `REC-AC-` followed by a three-digit number and are stable. Rule identifiers
in the text refer to [business-rules.md](business-rules.md); formulas to
[calculations.md](calculations.md).

## The standing fixture

Unless a scenario says otherwise, the starting point is:

- one company, `Company Test`, with a contact record and the alias domain `example.com`;
- one department, `Research and Development`, belonging to that company, with no manager;
- one Job Position, `Experienced Developer`, in that department, with a remaining target of
  five, the recruiter `Anna`, the inbound local part `experienced-developer`, and no
  interviewer;
- the six shipped Recruitment Stages, the six shipped Refusal Reasons, the four shipped
  Degrees, the four shipped Application Tags and the three shipped Job Boards;
- the acting user holds the Officer privilege and has the electronic mail address
  `officer@example.com`.

---

## A. Creating an Application and synchronising the Contact

**REC-AC-001. Creating an Application creates a Contact.**
Given no Contact exists carrying the address `test@thisisatest.com`,
when an Application is created with the applicant's name `Test` and that address,
then exactly one Contact exists with that address, its name is `Test`, and the Application's
Contact reference points at it.

**REC-AC-002. A second Application reuses that Contact.**
Given REC-AC-001 has run,
when a second Application is created with the same name and the same address,
then still exactly one Contact carries that address, and both Applications point at it
(REC-064).

**REC-AC-003. The Contact inherits the language of the request.**
Given two languages are active and the default Contact language is the second of them,
when an Application is created in a request running in the first language, with the name
`Test Applicant` and the address `test_applicant@example.com`,
then the created Contact carries the first language, not the default one.

**REC-AC-004. Changing the address and the number updates the Contact.**
Given an Application named `Mary Applicant` with the address `applicant@example.com`, the
telephone number `123456789` and a Contact carrying both,
when the address is changed to `applicant_diff@example.com` and then the telephone number to
`987654321`,
then the Contact's address is `applicant_diff@example.com` and the Contact's telephone number
is `987654321` (REC-065).

**REC-AC-005. An address without a name is refused.**
Given an Application with an empty applicant's name and no Contact,
when the address `someone@example.com` is written on it,
then the write fails with `You must define a Contact Name for this applicant.` and no Contact
is created (REC-063).

**REC-AC-006. Surrounding spaces are stripped at creation.**
Given nothing,
when an Application is created with the address `  spaced@example.com  `,
then the stored address is exactly `spaced@example.com`, with no leading and no trailing
space (REC-004).

**REC-AC-007. The telephone number is not overwritten by the Contact.**
Given a Contact carrying the telephone number `111`,
when an Application is created naming that Contact and carrying the telephone number `222`,
then the Application's telephone number is `222`, because the forward rule fills the number
only when the Application's number is empty (REC-061).

**REC-AC-008. The address is overwritten by the Contact.**
Given a Contact carrying the address `contact@example.com`,
when an Application is created naming that Contact and carrying the address
`typed@example.com`,
then the Application's address is `contact@example.com`: the forward rule overwrites the
address unconditionally (REC-061).

**REC-AC-009. A Contact created before today is never renamed by the public form.**
Given an internal user `Internal User` whose Contact was created a week ago and carries the
address `internal.user@example.com`,
when a visitor submits the public application form with the name `Impersonated User` and that
address,
then the Application points at the existing Contact, the Application's name is
`Impersonated User`, and the Contact's name is still `Internal User` (REC-066).

**REC-AC-010. The normalised address is lower case and carries no display name.**
Given nothing,
when an Application is created with the address `Mr. Richard Anderson <Richard_Anderson@yahoo.com>`,
then the stored address is that whole string and the normalised address is
`richard_anderson@yahoo.com` (REC-005).

**REC-AC-011. An unformattable telephone number is still sanitised.**
Given nothing,
when an Application is created with the telephone number `123`,
then `partner_phone_sanitized` holds `123`, the raw value, and `phone_sanitized` is empty
(REC-006, REC-007).

---

## B. Derivation from the Job Position

**REC-AC-020. Stage, recruiter, department and company follow the position.**
Given the fixture,
when an Application is created naming `Experienced Developer` and nothing else,
then its stage is `New`, its recruiter is `Anna`, its department is `Research and
Development` and its company is `Company Test` (REC-036, REC-009).

**REC-AC-021. The landing stage skips folded stages.**
Given a stage `Screening` with sequence 0, marked folded, and a stage `New` with sequence 1,
not folded, both unrestricted,
when an Application is created naming a position,
then its stage is `New` (REC-036).

**REC-AC-022. Clearing the position clears the stage.**
Given an Application sitting in `Qualification`,
when its Job Position is cleared,
then its stage is empty.

**REC-AC-023. Changing the position keeps the stage.**
Given an Application of `Experienced Developer` sitting in `Second Interview`,
when its Job Position is changed to another position that also uses the shipped stages,
then its stage is still `Second Interview` (REC-037).

**REC-AC-024. The company falls back to the position's company.**
Given a department with no company, and a position of `Mystery Company` using that
department,
when an Application is created for that position through the position's inbound address,
then the Application's department is that department and its company is `Mystery Company`
(REC-009).

**REC-AC-025. An Application for another company is created in that company.**
Given a position `Experienced Developer (Other Company)` belonging to `Other Company`, while
the acting company is `Company Test`,
when a message addressed to that position's inbound address is processed,
then the created Application's company is `Other Company`, not `Company Test` (REC-083).

**REC-AC-026. The recruiter selection is limited to the company.**
Given `User A` belongs to `Company A` only and `User B` to `Company B` only, and an
Application belongs to `Company A`,
then `User A` is offered as recruiter and `User B` is not; and when the Application has no
company, both are offered (REC-010).

**REC-AC-027. A stage restricted to another position is not offered.**
Given a stage restricted to the position `Job 2`,
when the pipeline of the position `Job 3` is opened,
then that stage is not among the columns (REC-038).

---

## C. Pipeline, stage and status

**REC-AC-030. Entering a hired stage stamps the hire date and decrements the target.**
Given a position with a remaining target of 1, an Application of that position sitting in
`New`, and a stage `Hired` with sequence 1 flagged as a hired stage,
when the Application is moved to `Hired`,
then the Application's hire date is the current moment, its derived status is `hired`, and
the position's remaining target is 0 (REC-041, REC-042).

**REC-AC-031. Leaving a hired stage clears the hire date and increments the target.**
Given REC-AC-030 has run,
when the Application is moved back to `New`,
then the hire date is empty, the derived status is `ongoing`, and the position's remaining
target is 1 again.

**REC-AC-032. The target never goes below zero.**
Given a position with a remaining target of 0 and an Application sitting in `New`,
when the Application is moved to a hired stage,
then the hire date is stamped and the remaining target is still 0 (REC-042).

**REC-AC-033. The increment has no upper bound.**
Given a position with a remaining target of 0 and three Applications sitting in a hired
stage,
when all three are moved out of the hired stage, one at a time,
then the remaining target is 3.

**REC-AC-034. A move between two non-hired stages changes nothing.**
Given a position with a remaining target of 5 and an Application in `Qualification`,
when the Application is moved to `First Interview`,
then the remaining target is still 5 and the hire date is still empty.

**REC-AC-035. A stage change resets the readiness colour.**
Given an Application whose readiness colour is `blocked`,
when it is moved to another stage without naming a readiness colour in the same operation,
then its readiness colour is `normal` and its last-stage-update moment is the current moment
(REC-040).

**REC-AC-036. A stage change that names a colour keeps that colour.**
Given the same Application,
when the stage and the readiness colour `done` are written in one operation,
then the readiness colour is `done`, not `normal`.

**REC-AC-037. A stage change records the previous stage and posts the tracked change.**
Given an Application in `Qualification`,
when it is moved to `First Interview`,
then its previous-stage reference holds `Qualification` and its thread carries a tracked
change under the subtype *Stage Changed* (REC-040, REC-208).

**REC-AC-038. A stage template is posted when the stage is written into the creation values.**
Given the shipped stage `New` carrying the template *Recruitment: Application
Acknowledgement*,
when an Application is created with that stage written into the creation values,
then a message rendered from that template is posted in the Application's thread and
addressed to the candidate's Contact, with the subject
`Your Job Application: Experienced Developer` (REC-052).

**REC-AC-039. A stage merely computed at creation posts nothing.**
Given the same configuration,
when an Application is created with the position and no stage, so that the stage is computed
to `New`,
then no message rendered from that template is posted (REC-052).

**REC-AC-040. Restoring does not post the landing stage's template.**
Given a refused Application whose position's first stage carries a template,
when the Application is restored,
then it is active, its refusal reason is empty, it sits in the first non-folded stage, and
**no** message rendered from that template is posted (REC-049, REC-050).

**REC-AC-041. The derived status follows the three stored facts.**
Given four Applications — one untouched, one carrying a hire date, one archived with no
reason, one archived with a reason —
then their derived statuses are `ongoing`, `hired`, `archived` and `refused` respectively
(REC-046).

**REC-AC-042. An active Application carrying a reason reads as refused.**
Given an Application that was refused and then reactivated without clearing the reason,
then its derived status is `refused`, and a search for the status `refused` does **not**
find it, because that search requires inactivity (REC-046, REC-047).

**REC-AC-043. The archived search returns refused records too.**
Given an Application archived with a reason,
when Applications are searched with the status `archived`,
then it is in the result; and when they are searched with the status `refused`, it is in the
result as well (REC-048).

**REC-AC-044. A hired Application that is archived is not found by the hired search.**
Given an Application carrying a hire date that is then archived with no reason,
when Applications are searched with the status `hired`,
then it is **not** in the result (REC-048).

**REC-AC-045. An unsupported operator on the status search is refused.**
Given any set of Applications,
when the status is searched with an operator other than "is one of",
then the search is rejected as not implemented (REC-047).

**REC-AC-046. A stage holding Applications cannot be deleted.**
Given one Application sitting in `Qualification`,
when that stage is deleted,
then the deletion is refused and the stage still exists (REC-053).

**REC-AC-047. Clearing the hired flag clears the hire dates.**
Given a stage flagged as hired holding two Applications carrying hire dates,
when the flag is cleared, after the form has shown
`All applications will lose their hired date and hired status.`,
then both Applications lose their hire date and both derived statuses become `ongoing`, and
the position's remaining target is unchanged (REC-054).

**REC-AC-048. Staleness.**
Given the stage `Qualification` carrying a staleness threshold of 10 days, and an ongoing
Application with no hire date whose last stage update was 11 days ago,
then the Application is reported as stale with 11 whole days of staleness; and when the
threshold is set back to 0, it is no longer stale and its day count is 0 (REC-055).

**REC-AC-049. A hired Application is never stale.**
Given the same stage and an Application whose last stage update was 30 days ago but which
carries a hire date,
then the Application is not stale.

**REC-AC-050. Searching staleness with the wrong operator is refused.**
Given at least one stage carrying a threshold,
when the staleness field is searched with an inequality operator,
then the search is refused with
`For performance reasons, use "=" operators on rotting fields.` (REC-056).

**REC-AC-051. Staleness with no threshold anywhere is inactive.**
Given no stage in the installation carries a threshold,
when the staleness field is searched,
then the search is refused with
`Model configuration does not support the rotting feature` (REC-057).

**REC-AC-052. Time in stage excludes the period after refusal.**
Given an Application created on 1 March at 08:00, moved to `Qualification` on 3 March at
08:00, refused on 5 March at 08:00, read on 9 March at 08:00,
then the recorded time is 172 800 seconds for `New` and 172 800 seconds for `Qualification`
(REC-058).

**REC-AC-053. Archiving changes nothing but the active flag.**
Given an Application in `Second Interview` carrying a hire date and a refusal reason,
when it is archived,
then it is inactive, its stage is still `Second Interview`, its hire date is unchanged and
its refusal reason is unchanged (REC-059).

**REC-AC-054. Archiving a position archives its Applications.**
Given a published position carrying three active Applications,
when the position is archived,
then the position's publication flag is false and all three Applications are inactive
(REC-060).

**REC-AC-055. A multi-record stage write adjusts the target per record.**
Given a position with a remaining target of 5 and three Applications of that position sitting
in `Qualification`,
when all three are moved into a hired stage in one operation,
then the remaining target is 2, and all three carry a hire date (REC-116).

**REC-AC-056. A multi-record stage write leaves one previous-stage value on every record.**
Given the same three Applications, sitting respectively in `New`, `Qualification` and
`First Interview`,
when all three are moved into `Second Interview` in one operation,
then all three carry the same previous stage, that of the record processed last. This is the
compatibility finding of REC-117; a rebuild that writes each record's own previous stage is
correct but differs from the reference here.

---

## D. Intake through the inbound address and through job boards

**REC-AC-060. A message to a position's address creates an Application with its attachment.**
Given the position `Experienced Developer` whose inbound address is
`experienced-developer@example.com`, and a message from
`Mr. Richard Anderson <Richard_Anderson@yahoo.com>` carrying one attachment named
`resume.pdf`,
when the message is processed,
then one Application exists with the applicant's name `Mr. Richard Anderson`, the address
`Mr. Richard Anderson <Richard_Anderson@yahoo.com>`, the normalised address
`richard_anderson@yahoo.com`, the position `Experienced Developer`, the department
`Research and Development`, the company `Company Test`, the stage `New`, and one attachment
named `resume.pdf` whose extracted text is indexed.

**REC-AC-061. The gateway's technical user does not become the recruiter.**
Given REC-AC-060 processed by a technical user who happens to hold the Officer privilege,
then the Application's recruiter is `Anna`, the position's recruiter; and when the position
has no recruiter, the Application's recruiter is empty (REC-073).

**REC-AC-062. The inbound path does not exclude folded stages.**
Given a stage `Intake` with sequence 0, marked folded and unrestricted, and the shipped `New`
with sequence 0 as well but reordered to 1,
when a message is processed for a position,
then the created Application sits in `Intake`, the folded stage, because the inbound path
applies no folded filter (REC-074).

**REC-AC-063. A job board message takes the name from the subject.**
Given a Job Board named `YourJobPlatform` with the address `yourjobplatform@platform.com` and
the pattern `^New application:.*from (.*)`,
when a message arrives from `"Job Platform Application" <yourjobplatform@platform.com>` with
the subject `New application: Implementation Consultant from John Doe`,
then the created Application's name is `John Doe`, its electronic mail address is empty and
its Contact is empty (REC-076, REC-077).

**REC-AC-064. A job board message takes the name from the body when the subject does not
match.**
Given the same Job Board,
when a message arrives from the same address with the subject
`Very badly formatted subject :D` and a body containing
`New application: Implementation Consultant from John Doe`,
then the created Application's name is `John Doe` and its electronic mail address is empty.

**REC-AC-065. A job board pattern that captures nothing keeps the parsed display name.**
Given the same Job Board,
when a message arrives from `"Job Platform Application" <yourjobplatform@platform.com>` whose
subject and body match nothing,
then the created Application's name is `Job Platform Application` and its address is still
empty (REC-077).

**REC-AC-066. The job board address is unique and is normalised first.**
Given a Job Board carrying the address `cs@jobsdb.com`,
when a second Job Board is created with `CS@JobsDB.com`,
then the creation fails with
`The Email must be unique, this one already corresponds to another Job Platform.`, because
the address is normalised before the uniqueness check (REC-078).

**REC-AC-067. Replies are addressed to the position's inbound address.**
Given an Application of a position whose inbound address is
`experienced-developer@example.com`,
when a message is posted on that Application,
then the reply address of that message is `experienced-developer@example.com` (REC-082).

**REC-AC-068. An Application with no position falls back to the generic reply address.**
Given a spontaneous Application with no Job Position,
when a message is posted on it,
then its reply address is the platform's generic one, not a position address.

**REC-AC-069. A follower of the position is notified of a new Application.**
Given a user following the position with the subtype *New Applicant* enabled,
when an Application is created for that position,
then the creation message carries the subtype *New Applicant* and that user's Contact is
among the notified recipients (REC-207, REC-208).

**REC-AC-070. A talent's creation message carries the talent subtype.**
Given the pool `Reserve`,
when an Application is created directly with that pool,
then its creation message carries the subtype *New Talent*, not *New Applicant* (REC-208).

**REC-AC-071. A priority carried by the message becomes the evaluation.**
Given a message carrying a recognised priority of `2`,
when it is processed into an Application,
then the Application's evaluation is `2`, labelled *Very Good* (REC-079).

---

## E. The public job pages

**REC-AC-080. Applying through the public form creates the Application and logs the extra
values.**
Given the published position `Developer` of the department `Research and Development`,
when a visitor submits the form with the name `Georges`, the address `georges@test.com`, the
telephone number `12345678`, the hidden position and department, a custom entry named
`description` holding `This is a short introduction`, and a custom entry named
`Additional info` holding `Test`,
then one Application exists carrying those four field values, and its thread carries one note
reading `Other Information:`, a line of underscores, an empty line,
`description : This is a short introduction` and `Additional info : Test` (REC-093).

**REC-AC-081. The short-introduction entry is relabelled.**
Given the same form,
when the visitor fills the introduction area, whose submitted name is `short_introduction`,
then the note names that entry `Short introduction from applicant` and not
`short_introduction` (REC-093).

**REC-AC-082. The uploaded curriculum vitae becomes an attachment.**
Given the same form,
when a file is chosen in the résumé field,
then the created Application carries that file as an attachment, and a second note announces
it with the body `Attached files: ` (REC-095).

**REC-AC-083. Applying to a closed position is refused.**
Given an archived position,
when the form is submitted for it,
then the submission fails with `The job offer has been closed.` and no Application is created
(REC-088).

**REC-AC-084. The public form uses the first non-folded stage.**
Given a stage `Screening` with sequence 0 marked folded and `New` with sequence 1 not folded,
when the form is submitted,
then the created Application sits in `New` (REC-089).

**REC-AC-085. No authentication note is logged for an anonymous submission.**
Given a visitor with no session,
when the form is submitted,
then the created Application's thread carries no "authenticated as" note (REC-096).

**REC-AC-086. The résumé or the profile address is required.**
Given the public form with an empty résumé field and an empty professional network field,
when the visitor presses the submission control,
then both fields become required and the form is not sent (REC-097).

**REC-AC-087. A malformed profile address warns without blocking.**
Given the public form,
when the visitor types `not-a-profile` in the professional network field and leaves it,
then the warning `The profile that you gave us doesn't seems like a linkedin profile` appears
under the field, the field is highlighted, and the visitor can still submit (REC-098).

**REC-AC-088. Only published positions are public.**
Given one published and one unpublished position,
when an unauthenticated visitor reads positions,
then only the published one is returned (REC-175).

**REC-AC-089. Publishing stamps the published date.**
Given an unpublished position,
when it is published,
then both publication flags are true and the published date is today; and when it is
unpublished, both flags are false and the published date is empty (REC-196).

**REC-AC-090. The job list survives a position with no location.**
Given three published positions: one whose job location is in `Paris`, one whose job location
carries no city, and one with no job location at all,
when the public job list is requested,
then the page is produced without failure and the last two show `Remote` as their location.

**REC-AC-091. Pagination.**
Given twenty-seven published positions matching the search,
when the job list is requested for page 1, then page 3,
then page 1 shows twelve positions and page 3 shows three; and when a department filter
reduces the result to nine, the pager offers one page only (REC-199).

**REC-AC-092. The default country filter is applied only when it matches something.**
Given a visitor whose network address resolves to a country in which no published position
has a job location, and no filter set,
when the job list is requested,
then no country filter is applied and every published position of the website is considered
(REC-201).

**REC-AC-093. The default country filter is skipped when another filter is set.**
Given the same visitor and a department filter,
when the job list is requested,
then no country is detected and only the department filter applies (REC-201).

**REC-AC-094. A position published on one website is invisible on another.**
Given a position restricted to website A and published,
when the job list of website B is requested,
then the position is absent; and a position restricted to no website appears on both
(REC-202).

**REC-AC-095. The legacy detail address redirects permanently.**
Given a published position,
when the address `/jobs/detail/` followed by its readable identifier is requested,
then the answer is a permanent redirection to `/jobs/` followed by the same identifier
(REC-198).

**REC-AC-096. Live warning: an ongoing Application on the same position.**
Given an ongoing Application for `Developer` with the address `georges@test.com`, whose
recruiter `Anna` has the address `anna@example.com` and no telephone number,
when the visitor types that address in the form of the same position,
then the answer is `An application already exists for georges@test.com. Duplicates might be
rejected.` followed by ` In case of issue, contact Anna, anna@example.com` (REC-099).

**REC-AC-097. Live warning: a refused Application within six months.**
Given a refused, inactive Application for `Developer` created two months ago with the address
`georges@test.com`,
when the visitor types that address for that position,
then the answer is `We've found a previous closed application in our system within the last 6
months. Please consider before applying in order not to duplicate efforts.` (REC-099).

**REC-AC-098. Live warning: a refusal older than six months is ignored.**
Given the same Application created seven months ago,
when the visitor types that address for that position,
then the answer carries no message, provided no ongoing Application matches.

**REC-AC-099. Live warning: an ongoing Application on another position.**
Given an ongoing Application of the same person for a different position,
when the visitor types the value,
then the answer is `We found a recent application with a similar name, email, phone number.
You can continue if it's not a mistake.` (REC-099).

**REC-AC-100. Live warning: nothing matches.**
Given no matching Application,
then the answer carries no message.

**REC-AC-101. Live warning: an unknown key matches nothing.**
Given any set of Applications,
when the check is called with a key name that is none of the four,
then the answer carries no message.

**REC-AC-102. The thank-you page shows the recruiter block.**
Given a submission for a position whose recruiter is `Anna`,
when the visitor is redirected to the thank-you page,
then the page shows `Congratulations!`, the two sentences
`Your application has been posted successfully,` and `We usually respond within 3 days...`,
and a block carrying Anna's photograph, name, job title, work telephone number and electronic
mail address.

**REC-AC-103. The thank-you page is not cached.**
Given two visitors submitting for two different positions,
when each is redirected to the thank-you page,
then each sees the recruiter of their own position; the page is never served from the cache
(REC-203).

---

## F. Duplicate detection

**REC-AC-110. Address matching ignores case.**
Given three Applications carrying the addresses `laurie.poiret@aol.ru`,
`laurie.POIRET@aol.ru` and `laure.poiret@aol.ru`,
then the first two report an application count of 2 and the third reports 1.

**REC-AC-111. Matching by address, by telephone number or by both.**
Given the seven Applications of the worked example in
[calculations.md](calculations.md#41-application-count),
then their application counts are 3, 2, 2, 3, 0, 1 and 0, in that order.

**REC-AC-112. An Application with no key counts zero.**
Given an Application with no address, no telephone number, no profile address and no
canonical pool copy,
then its application count is 0, not 1.

**REC-AC-113. The count ignores archiving and refusal.**
Given three Applications carrying the address `test@example.com`,
when one is archived and another is refused,
then all three still report a count of 3.

**REC-AC-114. The related-applications list holds exactly the matches.**
Given the same three Applications,
when the related-applications operation is run on the first,
then the list holds exactly three records, archived ones included, grouped by stage, and the
creation control is suppressed.

**REC-AC-115. Two unformattable spellings of one number do not match.**
Given one Application carrying `123` and another carrying `+49 123`, neither of which can be
formatted,
then each reports an application count of 1.

**REC-AC-116. The statistic control is hidden for a single application.**
Given an Application whose application count is 1 and which is not a talent,
then the related-applications control is not shown; and when the count reaches 2, it is
shown.

---

## G. Refusal

**REC-AC-120. Refusing archives the Application and stores the reason.**
Given an ongoing Application and the reason `Spam`,
when the refusal is applied with the message switch off,
then the Application is inactive, carries that reason, carries a refusal moment equal to the
current moment, and its derived status is `refused` (REC-110).

**REC-AC-121. Refusing does not change the stage or the target.**
Given an Application in `Second Interview` of a position whose remaining target is 2,
when it is refused,
then its stage is still `Second Interview` and the remaining target is still 2 (REC-115).

**REC-AC-122. Refusing a hired Application keeps the hire date.**
Given an Application carrying a hire date,
when it is refused,
then its hire date is unchanged and its derived status is `refused`, not `hired` (REC-115,
REC-046).

**REC-AC-123. Refusing duplicates refuses every live Application of the same person.**
Given the Applications of `Laurie` (`laurie.poiret@aol.ru`) and of `Mitchell`
(`mitchell_admin@example.com`), both ongoing, and the reason `Duplicate`,
when Laurie's Application is refused with the duplicate switch on,
then no active Application carrying `laurie.poiret@aol.ru` remains, and Mitchell's
Application is untouched (REC-108).

**REC-AC-124. A refused duplicate receives the explanatory note.**
Given two ongoing Applications of the same person, one of them selected,
when the selection is refused with the duplicate switch on,
then the unselected one carries the log line
`Refused automatically because this application has been identified as a duplicate of `
followed by a link to the selected one (REC-109).

**REC-AC-125. Only the selected Applications receive the message.**
Given REC-AC-124 with the message switch on,
then exactly one message is sent, and it is posted on the selected Application (REC-111).

**REC-AC-126. The duplicate set excludes hired, refused and archived records.**
Given three other Applications of the same person: one ongoing, one already refused and one
archived without a reason,
when the refusal dialog is opened on the original,
then the duplicate count is 1 (REC-108).

**REC-AC-127. The sender comes from the template when the template defines one.**
Given a reason whose template defines the sender `test@test.test`,
when the message values are prepared for a candidate,
then the sender is `test@test.test`; and given a reason whose template defines no sender, the
sender is the acting user's formatted address (REC-113).

**REC-AC-128. An archived template is not adopted.**
Given a reason whose template is archived,
when the refusal dialog is opened,
then the dialog's template is empty and the message switch is proposed as off (REC-102,
REC-104).

**REC-AC-129. Refusing with no sender address is refused.**
Given an acting user with no electronic mail address,
when a refusal with the message switch on is applied,
then the operation fails with
`Unable to post message, please configure the sender's email address.` (REC-106).

**REC-AC-130. Refusing with the message switch on and a candidate without an address is
refused.**
Given two selected Applications, one of which has neither its own address nor an address on
its Contact,
when the refusal with the message switch on is applied,
then the operation fails with
`At least one applicant doesn't have a email; you can't use send email option.` (REC-107).

**REC-AC-131. The warning lists the Applications without an address.**
Given the same selection while the dialog is open,
then the dialog shows `You can't select Send email option.`, then
`The email will not be sent to the following applicant(s) as they don't have an email
address:`, then the names of those Applications separated by a comma and a space (REC-105).

**REC-AC-132. The dialog opens for an Application with no name.**
Given an Application carrying only the telephone number `123`,
when the refusal dialog is opened on it,
then the dialog opens and the Application appears in it with an empty name.

**REC-AC-133. The refusal message is rendered per candidate.**
Given a template whose subject is the words `Application refused: ` followed by the
applicant's name, and an Application named `Mario` with the address `super@mario.bros`,
when the refusal is applied with the message switch on,
then a message exists whose subject is `Application refused: Mario` and whose only recipient
is that Application's Contact (REC-112, REC-113).

**REC-AC-134. The default reason is the first in sequence order.**
Given the six shipped reasons,
when the refusal dialog is opened,
then the reason proposed is `Refused by applicant: salary`, whose sequence is 10 (REC-101).

**REC-AC-135. The duplicate switch is hidden when there is nothing to refuse.**
Given a selection with no live duplicate,
when the refusal dialog is opened,
then the duplicate count is 0 and the duplicate switch is not shown.

**REC-AC-136. The scheduled date delays the message.**
Given a reason whose template carries a scheduled date one day in the future,
when the refusal is applied with the message switch on,
then the message carries that scheduled date and is not sent before it (REC-114).

---

## H. Hiring and creating the Employee

**REC-AC-140. Creating the Employee copies the mapped values.**
Given an Application named `Applicant 1` with the address `test_applicant@example.com`, a
hire date, the position `Experienced Developer` and the department `Research and
Development`,
when the Employee is created from it,
then an Employee exists whose name is `Applicant 1`, whose work contact is the Application's
Contact, whose Job Position is `Experienced Developer`, whose job title is
`Experienced Developer`, whose department is `Research and Development`, and the Application
points at that Employee (REC-122, REC-127).

**REC-AC-141. The work address is the company's contact and the work address values come from
the department's company.**
Given the Application of REC-AC-140, whose company is `Company Test` and whose department's
company carries the address `info@companytest.com` and the telephone number `+1 555-0100`,
then the Employee's work address is the contact record of `Company Test`, its work electronic
mail address is `info@companytest.com` and its work telephone number is `+1 555-0100`
(REC-122).

**REC-AC-142. The work address falls back to the candidate's own address.**
Given the same Application whose department's company carries no electronic mail address,
when the Employee is created,
then the Employee's work electronic mail address is `test_applicant@example.com`, the
candidate's own (REC-122).

**REC-AC-143. Attachments are copied once.**
Given the same Application carrying the file `textFile.txt`,
when the Employee is created, and the operation is then repeated on a second Application
carrying an identical file,
then the Employee carries exactly one attachment with that content: the second copy is
skipped because the content is already present (REC-124).

**REC-AC-144. A note records the creation of the Employee.**
Given REC-AC-140,
then the Application's thread carries the note `Employee created: Applicant 1` with a link to
the Employee (REC-126).

**REC-AC-145. An Interviewer may not create an Employee.**
Given a user holding the Interviewer privilege and not the Officer privilege, and an
Application that user may read,
when the create-employee operation is invoked,
then it fails with `You are not allowed to perform this action.` (REC-120).

**REC-AC-146. Creating the Employee needs a name.**
Given an Application with an empty applicant's name and no Contact,
when the create-employee operation is invoked,
then it fails with `Please provide an applicant name.` (REC-121).

**REC-AC-147. The name falls back to the Contact's display name.**
Given an Application with an empty applicant's name but with a Contact named `Sharlene
Rhodes`,
when the Employee is created,
then the Employee's name is `Sharlene Rhodes` (REC-122).

**REC-AC-148. Skills are transferred without their validity windows.**
Given an Application holding `Test Skill 1` at `Level 2` with a validity start three months
ago,
when the Employee is created,
then the Employee holds one skill line carrying the same skill, the same level and the same
skill type, and that line carries no validity dates (REC-157).

**REC-AC-149. The forecast headcount is stable across the hire.**
Given a position with five employees and a remaining target of two, so a forecast of seven,
when one Application is moved into a hired stage and the Employee is then created,
then after the move the target is 1 and the forecast 6, and after the creation the employee
count is 6 and the forecast 7 again.

**REC-AC-150. An Application may be hired without an Employee.**
Given an Application moved into a hired stage and no create-employee operation run,
then its derived status is `hired` and its Employee reference is empty (REC-128).

**REC-AC-151. The private address comes from the Contact's contact address.**
Given an Application whose Contact carries the contact address
`250 Executive Park Blvd, Suite 3400, San Francisco, California 94134, United States` and the
language of that address,
when the Employee is created,
then the Employee's private street, second street line, city, state, postal code, country,
telephone number, electronic mail address and language are copied from that address
(REC-122).

**REC-AC-152. The Create Employee control is hidden when it does not apply.**
Given three Applications — one active with a hire date and an Employee, one archived with a
hire date, one active with no hire date —
then none of them shows the create-employee control (REC-119).

---

## I. Interviewers and access

**REC-AC-160. Naming an interviewer grants the privilege.**
Given `Simple User` holding no recruitment privilege,
when that user is named as interviewer on a Job Position,
then the user holds the Interviewer privilege (REC-033).

**REC-AC-161. Removing the last attachment revokes the privilege.**
Given REC-AC-160,
when the user is removed from the position's interviewers,
then the user no longer holds the Interviewer privilege (REC-034).

**REC-AC-162. A remaining attachment keeps the privilege.**
Given `Simple User` named as interviewer on a Job Position **and** on an Application of
another position,
when the user is removed from the Application only,
then the user still holds the Interviewer privilege; and when the user is then removed from
the position as well, the privilege is revoked (REC-034).

**REC-AC-163. An Officer is never given the narrower privilege explicitly.**
Given a user holding the Officer privilege,
when that user is named as interviewer on an Application,
then the user's privileges are unchanged: the Interviewer privilege is implied by the Officer
privilege and is not added (REC-033).

**REC-AC-164. An Interviewer sees only the Applications they are attached to.**
Given an Interviewer and an Application of a position with no interviewers,
when the Interviewer reads that Application,
then the read fails with the platform's access refusal; and when the Interviewer is added to
the Application's interviewers, or to the position's interviewers, the read succeeds
(REC-023).

**REC-AC-165. An Interviewer may not create or delete an Application.**
Given an Interviewer,
when a creation and a deletion are attempted,
then both are refused (REC-024, REC-025).

**REC-AC-166. An Interviewer may change the interviewer list.**
Given an Application the Interviewer may reach,
when the Interviewer writes another user into the interviewers,
then the write succeeds and the list holds that user (REC-032).

**REC-AC-167. Naming interviewers on several Applications notifies each new one.**
Given two Applications carrying different interviewers,
when a third user is added to both in one operation,
then all three interviewers are named on the Applications and a notification exists whose
subject is `You have been assigned as an interviewer for ` followed by the Application's
display name and whose body is
`You have been assigned as an interviewer for the Applicant ` followed by the applicant's
name (REC-209).

**REC-AC-168. The acting user is not notified of their own assignment.**
Given an Officer,
when that Officer adds themselves to an Application's interviewers,
then no notification is created for them (REC-209).

**REC-AC-169. Naming an interviewer on a position sends no notification.**
Given a Job Position,
when a user is named as interviewer on it,
then the privilege is granted and **no** notification is sent (REC-210).

**REC-AC-170. Salary fields are absent from an Interviewer's read.**
Given an Interviewer who is not an Officer,
when the Application form is opened,
then the whole salary group is absent from the form, and reading any of the four salary
fields explicitly fails with the platform's field access refusal (REC-026, REC-027).

**REC-AC-171. Changing a position's recruiter moves only the ongoing Applications.**
Given a position whose recruiter is `Manager`, carrying two Applications both assigned to
`Manager`, one ongoing and one carrying a hire date,
when the position's recruiter is set to `New Manager`,
then the ongoing Application's recruiter is `New Manager` and the hired Application's
recruiter is still `Manager` (REC-215).

**REC-AC-172. The previous recruiter is unsubscribed, unless they are the department
manager's contact.**
Given the same position, where `Manager` is also the department manager's related contact,
when the recruiter changes,
then `Manager` remains a follower of the position and of its ongoing Applications (REC-215,
REC-130).

**REC-AC-173. The alias defaults are rewritten when the recruiter changes.**
Given the same position,
when the recruiter changes to `New Manager`,
then a subsequent message to the position's inbound address produces an Application whose
recruiter is `New Manager` (REC-072).

**REC-AC-174. The extended interviewer list reaches the position.**
Given an Interviewer named on one Application of a position and on nothing else,
then that user may read the position itself, through the position's derived extended
interviewer list (REC-035).

**REC-AC-175. The multi-company rule hides other companies' Applications.**
Given an Application of `Other Company` and a reader whose allowed companies are
`Company Test` only,
then the reader does not see it; and an Application with **no** company is visible to every
reader (REC-011).

**REC-AC-176. Any internal user may create a tag.**
Given a user holding no recruitment privilege,
when the user creates an Application Tag,
then the creation succeeds; and when the same user deletes a tag, the deletion is refused
(REC-167).

---

## J. Talent pools

**REC-AC-180. Adding an Application to a pool creates a talent.**
Given the pool `Test Talent Pool 1` and the Application `Test Applicant 1`,
when the Application is added to the pool,
then a second record exists, it belongs to that pool, it is not the original, its canonical
pool copy is itself, and the original's canonical pool copy points at it (REC-135).

**REC-AC-181. One talent for several pools.**
Given two pools,
when one Application is added to both in one operation,
then exactly one talent exists and it belongs to both pools.

**REC-AC-182. Several Applications produce one talent each.**
Given two pools and two Applications,
when both Applications are added to both pools in one operation,
then exactly two talents exist, each belonging to both pools.

**REC-AC-183. An Application is never pooled twice.**
Given an Application already added to a first pool,
when the add-to-pool dialog is opened again,
then that Application is not offered; and when the **talent** is added to a second pool, no
new record is created and the talent belongs to both pools (REC-133, REC-134).

**REC-AC-184. Creating an Application directly inside a pool points it at itself.**
Given a request naming the pool `Test Talent Pool 1`,
when an Application named `Talent in a pool` is created in that request,
then the record belongs to that pool and its canonical pool copy is itself (REC-132).

**REC-AC-185. Tags chosen in the dialog land on the talent only.**
Given the tag `Reserve`,
when an Application is added to a pool with that tag,
then the talent carries the tag and the original Application's tags are unchanged (REC-137).

**REC-AC-186. The talent's name carries no copy suffix.**
Given an Application named `Test Applicant 1`,
when it is added to a pool,
then the talent is named `Test Applicant 1`, with no suffix (REC-136).

**REC-AC-187. A talent must keep at least one pool.**
Given a talent belonging to one pool,
when every pool is removed from it,
then the write fails with `Talent must belong to at least one Talent Pool.` (REC-021).

**REC-AC-188. A talent may not be duplicated.**
Given a talent,
when the generic duplicate operation is invoked on it,
then it fails with `You cannot duplicate the talent(s).` (REC-022).

**REC-AC-189. Creating an Application out of a talent applies the stage template.**
Given a talent, and the position `Job 2` with a stage `Recruitment Stage` restricted to it, of
sequence 0, not folded, carrying a message template,
when an Application is created out of the talent for that position,
then the new Application belongs to `Job 2`, sits in `Recruitment Stage`, has an empty pool
list, keeps its canonical pool copy pointing at the talent, and its thread carries the
message rendered from that stage's template (REC-141, REC-142).

**REC-AC-190. One talent and two positions produce two Applications.**
Given one talent, the positions `Job 2` and `Job 3`, and the original Application the talent
came from,
when Applications are created for both positions,
then four records carry the applicant's name: the original Application, the talent, and one
new Application per position.

**REC-AC-191. The notification names the count and the people.**
Given the previous scenario,
then the interface shows `Created 2 new applications for: ` followed by the distinct
applicant names (REC-145).

**REC-AC-192. Updating a linked Application updates the talent.**
Given an Application whose canonical pool copy is a talent,
when the Application's address is set to `updated@gmail.com`,
then the Application carries that address and the talent carries it as well (REC-139).

**REC-AC-193. Only the fields in the write are propagated.**
Given the same pair,
when only the degree is written on the Application,
then the talent's degree changes and the talent's address, telephone number and profile
address are unchanged (REC-139).

**REC-AC-194. Talent pool counts, direct and indirect.**
Given the fixture of the worked example in
[calculations.md](calculations.md#44-talent-pool-count),
then the counts are: talent *tA* 2, talent *tB* 1, the Application pointing at *tA* 2, the
second Application pointing at *tA* 2, the Application sharing the address 2, the one sharing
the telephone number 2, the one sharing the profile address 2, the unrelated one 0, and the
one pointing at *tB* 1.

**REC-AC-195. Pool membership is not transitive.**
Given a talent *T* with the address `a@example.com`, an Application *X* with the same
address, and an Application *Y* sharing only *X*'s telephone number,
then *X* is in a pool and *Y* is **not** (REC-143).

**REC-AC-196. Pooling needs no rights on the Employee register.**
Given a user holding only the Officer privilege of this domain,
when that user adds an Application to a pool,
then the operation succeeds and the pool holds one talent (REC-138).

**REC-AC-197. The talent statistic resolves a missing link.**
Given an Application that shares an address with a talent but whose canonical pool copy is
empty,
when the talent statistic control is pressed,
then the canonical pool copy is stored and the talent's form opens (REC-144).

---

## K. Skills and matching

**REC-AC-200. Adding a skill.**
Given an Application and the skill `Test Skill 1` of the type `Skills for tests` at
`Level 1`,
when the skill is added,
then the Application holds exactly one skill line carrying that skill, that level and that
type, with a validity start of today and no validity end.

**REC-AC-201. Adding the same skill twice in one save keeps one line.**
Given the previous scenario,
when the same skill and level are added again in the same save,
then the Application still holds exactly one skill line (REC-156).

**REC-AC-202. Changing the level versions the line.**
Given an Application holding `Test Skill 1` at `Level 2` since three months ago with no
validity end,
when `Test Skill 1` at `Level 3` is added today,
then the Application holds two lines for that skill, the first ending yesterday and the
second starting today, and the current-skill set holds only the second (REC-150, REC-155).

**REC-AC-203. A line created today is replaced rather than closed.**
Given an Application whose line for `Test Skill 1` at `Level 1` was created today,
when `Test Skill 1` at `Level 2` is added today,
then the Application holds exactly one line for that skill, at `Level 2`, starting today
(REC-150).

**REC-AC-204. Overlapping non-certification lines are refused.**
Given an Application holding `Test Skill 1` valid from 1 January to 31 December,
when a second line for the same skill valid from 1 June to 30 June is created directly,
then the operation fails with
`The following skills can't be created as they overlap or exactly match existing skills:`
followed by the bullet line naming the new line, the existing line and the existing validity
window (REC-151).

**REC-AC-205. A validity end before the validity start is refused.**
Given an Application,
when a skill line is created with a validity start of 1 June and a validity end of 1 May,
then the operation fails with
`The following skills have their valid stop date prior to their valid start date:` followed
by the bullet line naming the skill and the two dates (REC-152).

**REC-AC-206. A skill that does not belong to the type is refused.**
Given a skill of the type `Languages` and the type `Skills for tests`,
when a line pairs them,
then the operation fails with `The skill %(name)s and skill type %(type)s don't match`, the
placeholders carrying the skill's name and the type's name (REC-153).

**REC-AC-207. A level that does not belong to the type is refused.**
Given a level of another type,
when a line uses it,
then the operation fails with
`The skill level %(level)s is not valid for skill type: %(type)s` (REC-154).

**REC-AC-208. An expired certification stays current.**
Given an Application holding one certification whose validity end is last month, and no other
line for that skill,
then the current-skill set still holds that line (REC-155).

**REC-AC-209. Skills are copied to the talent.**
Given an Application holding one skill,
when it is added to a pool,
then the talent holds one skill line carrying the same skill and level; and with three skills
on the Application, the talent holds three (REC-140).

**REC-AC-210. Skill changes are propagated to the talent.**
Given a linked Application and its talent, both holding `Test Skill 1` at `Level 1`,
when the Application's line is changed to `Level 3`,
then the talent's line for that skill is changed as well (REC-140).

**REC-AC-211. Match score with an expected degree.**
Given a position requiring `Skill A` at a level of progress 50 and `Skill B` at a level of
progress 100, expecting the degree `Master Degree` of score 0.90, and a candidate holding
`Skill A` at a level of progress 100 with the degree `Bachelor Degree` of score 0.70,
then the candidate's match score is 71, the matching skills hold `Skill A` and the missing
skills hold `Skill B`.

**REC-AC-212. Match score without an expected degree.**
Given the same position with no expected degree,
then the score is 67.

**REC-AC-213. The position side is not rounded.**
Given the fixture of REC-AC-211 and a reader browsing positions with that candidate named in
the reading context,
then the position reports 70.8333…, displayed as 70.83, while the candidate reports 71.

**REC-AC-214. No expectation, no score.**
Given a position requiring neither a skill nor a degree,
then the candidate's match score is 0 and both skill lists are empty.

**REC-AC-215. The cap on an over-qualified skill.**
Given a position requiring `Skill A` at a level of progress 20, with no expected degree, and
a candidate holding `Skill A` at a level of progress 100,
then the score is 200, because the contribution is capped at twice the required progress and
the result is not clamped. This is the compatibility finding of
[calculations.md](calculations.md#81-applicant-to-position-match-score).

**REC-AC-216. The degree is ignored when the expectation is negligible.**
Given a position requiring `Skill A` at a level of progress 50 and expecting a degree of
score 0.01, and a candidate holding `Skill A` at that level and a degree of score 1.00,
then the candidate's degree contributes nothing, because the position's degree points equal 1
and the rule requires more than 1.

**REC-AC-217. The matching search excludes the position's own Applications.**
Given a position requiring `Skill A`, and two Applications holding `Skill A`, one of them for
that position,
when matching applicants are searched from the position,
then only the Application of the other position is listed (REC-161).

**REC-AC-218. Adding a matched candidate to the job posts no stage message.**
Given a matched Application and a position whose first stage carries a template,
when *Add to job* is used from the matching list,
then the Application carries that position and the shipped first stage, and **no** message
rendered from that template is posted (REC-160).

---

## L. Written interviews

**REC-AC-230. Sending an interview creates the answer set.**
Given the position `Technical worker` carrying the questionnaire
`Questions for Sysadmin job offer`, and an Application `Jane Doe` with the address
`customer@example.com`,
when the interview is sent,
then exactly one answer set exists for that questionnaire, it belongs to the Application, its
address is `customer@example.com`, and the Application's thread carries the note
`The survey Questions for Sysadmin job offer has been sent to Jane Doe`.

**REC-AC-231. The invitation body is posted in the thread and sent immediately.**
Given the previous scenario,
then the Application's thread carries a second entry holding the rendered invitation body,
and the message is sent immediately rather than left in the queue.

**REC-AC-232. The deadline defaults to fifteen days.**
Given the same Application,
when the invitation dialog is opened,
then the answer deadline is the current moment plus fifteen days.

**REC-AC-233. Sending an interview needs a name when there is no Contact.**
Given an Application with no Contact and an empty applicant's name,
when the interview is sent,
then it fails with `Please provide an applicant name.` (REC-068).

**REC-AC-234. Re-sending does not create a second answer set.**
Given an Application that already has an answer set for the questionnaire,
when the invitation is re-sent in resend mode,
then no second answer set is created, because the candidate's Contact is treated as already
answered.

**REC-AC-235. An Administrator may act only on recruitment questionnaires.**
Given a recruitment Administrator,
then that user may send the invitation for a recruitment questionnaire, and reading a
questionnaire of another kind fails with an access refusal (REC-180).

**REC-AC-236. An Officer or an Interviewer must be attached.**
Given an Officer and an Interviewer, neither named on the position nor on the Application,
when either sends the invitation,
then it fails with an access refusal; and once the user is named on the position, or on the
Application, the invitation succeeds (REC-180).

**REC-AC-237. Printing with no answer set prints the blank questionnaire.**
Given an Application whose position carries a questionnaire and which has no answer set,
when the print operation is invoked,
then a printable view of the blank questionnaire opens in a dialog.

**REC-AC-238. Printing prefers the most recent completed answer set.**
Given an Application with three answer sets for the position's questionnaire — one completed
last week, one completed yesterday and one still in progress created today —
when the print operation is invoked,
then the printable view shows the one completed yesterday.

**REC-AC-239. Printing falls back to the most recent answer set of any state.**
Given an Application with two answer sets, neither completed,
when the print operation is invoked,
then the printable view shows the most recently created one.

**REC-AC-240. An Interviewer may not print without an attachment to the Application.**
Given an Interviewer named neither on the Application nor on its position,
when the print operation is invoked,
then it fails with an access refusal (REC-180).

**REC-AC-241. Completing an answer set posts the note.**
Given an answer set linked to the Application `Jane Doe`,
when the answer set is completed,
then the Application's thread carries `The applicant "Jane Doe" has finished the survey.`,
authored by the platform's system contact (REC-218).

**REC-AC-242. A retry keeps the Application link.**
Given a completed answer set linked to an Application,
when a retry is granted,
then the new answer set carries the same Application reference.

---

## M. Messaging

**REC-AC-250. A bulk message reaches two Applications.**
Given two Applications with addresses and Contacts,
when a message with the subject `Test` and a body is sent through the bulk dialog,
then each Application's thread carries one message addressed to its own Contact, authored by
the chosen author, with the light notification layout.

**REC-AC-251. A missing Contact is created for the recipient that lacks one.**
Given two Applications, the first of which has no Contact,
when a bulk message is sent to both,
then a new Contact is created for the first and the second keeps its own.

**REC-AC-252. A recipient with no address blocks the whole send.**
Given a selection containing one Application with no address,
when the bulk message is sent,
then nothing is posted and the notification
`The following applicants are missing an email address: ` followed by the names and a full
stop is displayed (REC-211).

**REC-AC-253. Attachments are duplicated per recipient.**
Given a bulk message carrying one attachment and three recipients,
when it is sent,
then three copies of the attachment exist, one owned by each recipient Application (REC-212).

**REC-AC-254. Answering in the thread links the pending Applications.**
Given two Applications of the same person carrying the address `person@example.com`, neither
with a Contact, both in stages that are not folded,
when a message is posted on one of them with the applicant as recipient, which creates the
Contact,
then both Applications point at that Contact (REC-067).

**REC-AC-255. An Application parked in a folded stage is not linked.**
Given the same pair where the second Application sits in a folded stage,
when the message is posted on the first,
then only the first is linked to the new Contact (REC-067).

**REC-AC-256. A Contact created today is renamed, one created earlier is not.**
Given the same scenario where the Contact created by the composer was created today,
then its name becomes the applicant's name; and given a Contact created last week, its name
is unchanged (REC-066).

**REC-AC-257. Posting a message needs read access only.**
Given a follower who may read an Application but not write it,
when that user posts a message in the thread,
then the message is posted (REC-179).

---

## N. Recruitment Sources and attribution

**REC-AC-260. Creating a Recruitment Source creates its Tracking Source.**
Given nothing,
when a Recruitment Source named `Recruitment Source` is created on a position,
then a Tracking Source with that name exists and the Recruitment Source points at it.

**REC-AC-261. A Tracking Source in use may not be deleted.**
Given the previous scenario,
when the Tracking Source is deleted,
then the deletion fails with
`You cannot delete these UTM Sources as they are linked to the following recruitment sources
in Recruitment:` followed by the quoted names of the affected positions (REC-147).

**REC-AC-262. The shipped campaign may not be deleted.**
Given the shipped campaign `Job Campaign`,
when it is deleted,
then the deletion fails with `The UTM campaign 'Job Campaign' cannot be deleted as it is used
in the recruitment process.` (REC-146).

**REC-AC-263. Deleting a campaign that is not the shipped one succeeds and clears the
references.**
Given a campaign named `Spring Drive` referenced by three Applications,
when it is deleted,
then the deletion succeeds and the three Applications carry an empty campaign (REC-149).

**REC-AC-264. The tracking address carries the three parameters.**
Given the position page `/jobs/sales-manager-3`, the base web address `https://example.com`
and a Recruitment Source named `LinkedIn` whose medium is `website`,
then the tracking address is
`https://example.com/jobs/sales-manager-3?utm_campaign=Job+Campaign&utm_medium=website&utm_source=LinkedIn`.

**REC-AC-265. The source's inbound address stamps its values.**
Given a Recruitment Source `Indeed` on the position `Sales Manager` whose local part is
`sales-manager`, and the alias domain `example.com`,
when the address is created,
then its local part is `sales-manager+Indeed`, its full address is
`sales-manager+Indeed@example.com`, and an Application received there carries the position,
the campaign `Job Campaign`, the medium `email` and the source `Indeed` (REC-085).

**REC-AC-266. A position with no local part uses its name.**
Given a position named `Sales Manager` with no local part, and the source `Indeed`,
when the address is created,
then the local part is `Sales Manager+Indeed`.

**REC-AC-267. No alias domain, no address.**
Given a position whose company has no alias domain, and an acting company with none either,
then the source reports that it has no domain and the address control is hidden (REC-187).

**REC-AC-268. Deleting a Recruitment Source deletes its address.**
Given a source carrying an inbound address,
when the source is deleted,
then the alias no longer exists (REC-188).

**REC-AC-269. Deleting a position deletes its sources.**
Given a position carrying two sources,
when the position is deleted,
then both sources and both of their aliases are gone (REC-188).

**REC-AC-270. A visitor arriving through the tracking address is attributed.**
Given the tracking address of REC-AC-264,
when a visitor follows it and submits the application form,
then the created Application carries the campaign `Job Campaign`, the medium `website` and
the source `LinkedIn`.

---

## O. Counters, times and reporting

**REC-AC-280. Position counters.**
Given a position holding ten active Applications, of which three sit in its first stage, plus
four archived Applications carrying a refusal reason and one archived Application carrying
none,
then the application count is 10, the new count is 3, the old count is 7 and the
all-applications count is 14.

**REC-AC-281. The hired counter includes archived records.**
Given a position with two Applications carrying a hire date, one of them archived,
then the position's hired counter is 2 (REC-129).

**REC-AC-282. The open counter excludes the hired stage.**
Given a position holding five active Applications, one of them in a hired stage,
then the open application count is 4.

**REC-AC-283. Department counters.**
Given a department with two positions whose hired counters are 2 and 1 and whose remaining
targets are 3 and 0,
then the department reports 3 new hired employees and 3 expected employees.

**REC-AC-284. The department's new-applicant counter respects the privilege.**
Given a reader who does not hold at least the Interviewer privilege,
then the department's new-applicant counter is 0 for that reader, and no refusal is raised.

**REC-AC-285. The personal activity counter is per reader.**
Given a position with two active Applications not in a hired stage, one carrying an open
activity owned by `Anna` and one carrying an open activity owned by `Bob`,
then Anna's badge shows 1 and Bob's badge shows 1.

**REC-AC-286. The activity counter ignores hired Applications.**
Given the same position where Anna's Application is then moved into a hired stage,
then Anna's badge shows 0.

**REC-AC-287. Elapsed times.**
Given an Application created on 2 March at 09:00, assigned on 4 March at 15:00 and hired on
20 March at 09:00,
then the days to open are 2.25, the days to close are 18.00 and the delay to close is 15.75.

**REC-AC-288. Days to open survives the recruiter being cleared.**
Given the same Application,
when its recruiter is cleared,
then the assignment moment and the days-to-open value are unchanged (REC-044).

**REC-AC-289. Meeting summary.**
Given three meetings on 2, 10 and 20 March, read on 12 March,
then the displayed date is 20 March and the displayed text is *Next Meeting*; with a single
meeting the text is *1 Meeting*; with no meeting the text is *No Meeting* and the date is
empty.

**REC-AC-290. Meeting summary with only future meetings.**
Given two meetings on 20 and 25 March, read on 12 March,
then the displayed date is 20 March, the earliest, and the text is *Next Meeting*.

**REC-AC-291. The document count excludes Applications that produced an Employee.**
Given a position carrying one attachment of its own and two Applications each carrying one
attachment, one of which has produced an Employee,
then the position's document count is 2.

**REC-AC-292. The digest indicator is skipped below the Officer privilege.**
Given a digest recipient who does not hold the Officer privilege,
when the digest is computed,
then the new-employees indicator is omitted for that recipient and the rest of the digest is
sent (REC-213).

---

## P. Configuration entities

**REC-AC-300. Tag names are unique.**
Given the shipped tag `Sales`,
when a second tag with the same name is created,
then it fails with `Tag name already exists!` (REC-181).

**REC-AC-301. Degree names are unique.**
Given the shipped degrees,
when a degree named `Graduate` is created,
then it fails with `The name of the Degree of Recruitment must be unique!` (REC-182).

**REC-AC-302. The degree score is bounded.**
Given nothing,
when a degree carrying the score 1.5 is created,
then it fails with `Score should be between 0 and 100%` (REC-183).

**REC-AC-303. A stage created from a pipeline is global.**
Given the pipeline of `Experienced Developer`,
when a stage is created from that screen without naming a position,
then the stage is available to every position (REC-184).

**REC-AC-304. A stage created with the single-position marker is restricted.**
Given the same pipeline and the single-position marker set,
when a stage is created,
then the stage names `Experienced Developer` and is invisible to every other position
(REC-184).

**REC-AC-305. Duplicating a Refusal Reason resets the sequence.**
Given the reason `Spam` with sequence 15,
when it is duplicated,
then the copy carries the sequence 10, the field's default.

**REC-AC-306. Archived reasons are not proposed.**
Given the reason `Spam` archived,
when the refusal dialog is opened,
then `Spam` is not selectable, and Applications already refused for it keep it (REC-186).

**REC-AC-307. A position name must be unique per department and company.**
Given the position `Experienced Developer` in `Research and Development` of `Company Test`,
when a second position with the same three values is created,
then it fails with
`The name of the job position must be unique per department in company!` (REC-190).

**REC-AC-308. A negative target is refused.**
Given a position,
when its remaining target is set to −1,
then it fails with `The expected number of new employees must be positive.` (REC-043).

**REC-AC-309. A new position has no favourite.**
Given an Officer,
when a position is created through the interface with nothing named in the favourites,
then its favourites list is empty and the Officer's favourite marker is off (REC-193).

**REC-AC-310. The default job location follows the last position created.**
Given a position created earlier whose job location is the contact `Chicago Office`,
when a new position is created,
then its job location defaults to `Chicago Office`; and given no earlier position, it
defaults to the contact record of the acting user's company (REC-191).

**REC-AC-311. Every shipped stage carries the four default labels.**
Given a fresh installation,
then each of the six shipped stages carries the labels `In Progress`,
`Ready for Next Stage`, `Waiting` and `Blocked` (REC-185).

**REC-AC-312. Stages are readable but not writable by an Officer.**
Given an Officer,
when a stage is read, then written,
then the read succeeds and the write is refused (REC-164).

---

## Q. Duplication, deletion and archival of Applications

**REC-AC-320. Duplicating an Application.**
Given an Application named `Helen Lee` carrying an address, a telephone number, a Contact, a
Job Position, a stage, a hire date, an Employee, tags, custom properties and skill lines,
when it is duplicated through the generic duplicate operation,
then the copy is named `Helen Lee (copy)`, keeps the address, the telephone number, the tags,
the custom properties and the skill lines, and carries no Contact, no Job Position, no stage,
no hire date, no Employee and the readiness colour `normal`.

**REC-AC-321. Deleting an Application removes its skill lines.**
Given an Application holding two skill lines and referenced by one meeting,
when the Application is deleted,
then both skill lines are gone and the meeting still exists with an empty Application
reference.

**REC-AC-322. An Interviewer cannot delete.**
Given an Interviewer and an Application that user may reach,
when the deletion is attempted,
then it is refused (REC-025).

**REC-AC-323. Restoring puts the Application back at the start.**
Given a refused Application of `Experienced Developer` that was refused while in
`Second Interview`,
when it is restored,
then it is active, its refusal reason is empty, its stage is `New`, its readiness colour is
`normal` and its last-stage-update moment is the current moment (REC-049).

**REC-AC-324. Restoring an Application with no position clears the stage.**
Given a refused spontaneous Application with no Job Position,
when it is restored,
then it is active, its refusal reason is empty and its stage is empty (REC-049).

**REC-AC-325. Restoring a hired Application clears the hire date.**
Given an Application archived while sitting in the hired stage,
when it is restored,
then it sits in `New`, its hire date is empty and its derived status is `ongoing`; and the
position's remaining target has been incremented by one, because the stage write left the
hired stage.

---

## R. Multiple companies

**REC-AC-330. An Application takes the position's company, not the acting user's.**
Given an Officer whose current company is `Company Test`, and the position
`Experienced Developer (Other Company)` of `Other Company`,
when an Application is created for that position,
then its company is `Other Company` (REC-009).

**REC-AC-331. The department wins when it agrees with the position.**
Given a department of `Company Test` and a position of `Company Test`,
when an Application is created for that position,
then its company is `Company Test`, taken from the department.

**REC-AC-332. A spontaneous Application takes the acting user's company.**
Given an Officer whose current company is `Company Test`,
when an Application is created with no Job Position and no Department,
then its company is `Company Test`.

**REC-AC-333. Positions offered are limited to the Application's company.**
Given an Application whose company is `Company Test`,
then only positions of `Company Test` are offered; and when its company is cleared, every
position is offered (REC-010).

**REC-AC-334. Talent Pools are not company-scoped.**
Given a pool created in `Other Company`,
when an Officer whose allowed companies are `Company Test` only lists the pools,
then that pool is visible, because no record rule restricts pools by company.

**REC-AC-335. The multi-company rule admits Applications with no company.**
Given an Application whose company is empty,
then every reader sees it, whatever their allowed companies (REC-011).

---

## S. Reconciliation notes

| Point | Resolution |
|---|---|
| Numbering | One draft numbered its scenarios `REC-AC-nnn` in a single sequence and one supplied worked examples inside its narrative files. Both were merged into the single `REC-AC-` sequence of this file; the worked examples of the narrative draft became REC-AC-038, REC-AC-039, REC-AC-055, REC-AC-056, REC-AC-060, REC-AC-141, REC-AC-151 and REC-AC-171. |
| The refusal duplicate example | One draft used two applications of the same person for two different positions; the other used three applications sharing one address. Both are kept, as REC-AC-123 and REC-AC-113, because they exercise different parts of the matching condition. |
| Whether an unmatched Application counts itself | Settled by REC-AC-112, and by the worked example the counter file carries. |
| The wording of the live warning for a recruiter with no telephone number | One draft showed the message with a trailing comma. The empty parts are omitted, so the message ends after the address; REC-AC-096 states it. |
| The name of the job board in the extraction scenarios | One draft used a shipped board, the other a purpose-built one. The purpose-built board is used in REC-AC-063 to REC-AC-065, so that the scenarios do not depend on the shipped data, and the shipped patterns are exercised through [configuration.md](configuration.md#55-job-boards). |
| Scenarios for several companies | Only one draft had them, and only in passing. Chapter R now covers the five cases the company derivation can produce. |
