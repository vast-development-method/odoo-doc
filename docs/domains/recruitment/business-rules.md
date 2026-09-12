# Recruitment — Business Rules

Every validation, constraint, invariant, permission check and locking rule of the domain,
each with a stable identifier, the condition that makes it fire and the exact text the
system shows when it refuses. Messages are reproduced verbatim; a placeholder inside a
message is described in words so that a rebuild can substitute its own values.

## 1. How the rules are numbered

Rules carry the prefix `REC-` followed by a three-digit sequence number. The numbering is
stable within this file and is used by [workflows.md](workflows.md),
[acceptance-criteria.md](acceptance-criteria.md), [configuration.md](configuration.md) and
[interfaces.md](interfaces.md). A rule that only restates a rule owned by another domain
says so and links to it. Chapter 19 lists every identifier with a one-line summary; chapter
20 maps the identifiers used by the two earlier drafts of this folder onto the present
scheme.

Three markers are used, exactly as the writing rules of this repository define them:

- **industry-standard default** — the observed behaviour does not determine the answer, and
  the stated resolution is the conventional one.
- **compatibility finding** — the observed behaviour looks like a defect; it is recorded as
  observed, and the corrected behaviour is stated next to it.
- A rule with neither marker is observed behaviour that a rebuild must reproduce.

---

## 2. Application data integrity

**REC-001.** An Application has **no required field at the storage level**. Every screen
that creates one requires the applicant's name (`partner_name`); every automatic path — the
inbound message gateway, the public form — supplies a name from the parsed sender or from
the submitted form. A rebuild must keep the storage permissive and the screens strict,
because applications created by integrations rely on it.

**REC-002.** The display name of an Application is the applicant's name. An Application with
an empty name displays as an empty string in every reference field, every report and every
message. This is the reason the name is required on screens rather than in storage.

**REC-003.** The electronic mail address (`email_from`) holds at most 128 characters and the
telephone number (`partner_phone`) at most 32. Longer input is rejected by the storage
layer with its own length refusal.

**REC-004.** On creation, leading and trailing whitespace is stripped from the electronic
mail address before anything else happens, including before the address inverse of chapter
5 runs.

**REC-005.** The normalised address (`email_normalized`) is the canonical form of the
electronic mail address: the address alone, lower case, with the display name removed. It is
empty when the address contains nothing parsable. It is the value used for duplicate
detection, for blacklist matching and for attaching applications to a Contact.

**REC-006.** The sanitised telephone number (`partner_phone_sanitized`) is the telephone
number in international form when the number can be formatted, using the country of the
Application's company as the default country, and the **raw** telephone number otherwise. A
number that cannot be formatted therefore still takes part in duplicate detection, in its
raw spelling.

**REC-007.** The blacklist-side sanitised number (`phone_sanitized`) is empty when the
number cannot be formatted. `partner_phone_sanitized` and `phone_sanitized` are two distinct
stored values that differ exactly in that case, and a rebuild must keep both: the first
feeds duplicate detection, the second feeds the telephone blacklist and the text-message
capability.

**REC-008.** Nothing prevents two Applications carrying the same address, the same telephone
number or the same professional network profile address, for the same Job Position or for
different ones. Repetition is **reported** — by the application counter, by the live warning
of the public form and by the duplicate list of the refusal dialog — and never blocked.

**REC-009.** The company of an Application is derived, in this order: the Department's
company when it equals the Job Position's company; otherwise the Job Position's company when
a position is set; otherwise the acting user's current company. The field stays writable, so
a manual value survives until one of those inputs changes again.

**REC-010.** The Department offered on an Application is limited to departments with no
company or with the Application's company. The Job Positions offered are limited to those of
the Application's company when a company is set, and to all of them otherwise. The
recruiters and the interviewers offered are limited to non-shared users belonging to the
Application's company.

**REC-011.** A global record rule limits every reader to Applications whose company is among
the reader's allowed companies **or is empty**. It applies to every privilege including the
Administrator privilege, and it is the only company restriction in the domain: Talent Pools,
stages, reasons, degrees, tags and job boards are not company-scoped.

**REC-012.** The stages offered on an Application are those attached to no position plus
those attached to the Application's Job Position. The rule lives in the field's selection
domain; no storage constraint enforces it. **compatibility finding** — a write that bypasses
the screen may place an Application in a stage restricted to another position; a corrected
behaviour refuses such a write.

**REC-013.** The custom application properties follow the definition stored on the Job
Position. Changing the position changes the set of available properties, and values whose
definition disappears stop being displayed.

**REC-014.** The four salary fields — expected amount, expected extra advantages, proposed
amount, proposed extra advantages — are readable and writable only by the Officer privilege
and above. See chapter 3.

**REC-015.** The salary amounts carry **no currency field**. They are expressed in the
currency of the Application's company by convention, and no conversion is performed anywhere
in this domain. **industry-standard default** — a rebuild that needs multi-currency salary
reporting adds a currency field; that is an extension, not the reference behaviour.

**REC-016.** The employee name shown on the Application is writable and writes through to
the linked Employee, and the change is deliberately **not** recorded in the Application's
thread.

**REC-017.** The evaluation is one of four stored values, `0`, `1`, `2` and `3`, labelled
*Normal*, *Good*, *Very Good* and *Excellent*. It defaults to `0` and is the first sort key,
descending.

**REC-018.** The readiness colour is required and defaults to `normal`. It is not copied when
an Application is duplicated.

**REC-019.** Applications are ordered by evaluation descending, then by manual sequence
ascending, then by identifier descending, wherever no other order is requested.

**REC-020.** The probability field is a free decimal slot. No operation of this domain writes
it and no screen shows it. A rebuild must keep the column so that imported data and external
scoring tools can use it.

**REC-021.** A talent whose canonical pool copy is itself must belong to at least one Talent
Pool. Removing its last pool is refused with:
`Talent must belong to at least one Talent Pool.`

**REC-022.** A talent may not be duplicated. The generic duplicate operation on a record
whose pool list is non-empty is refused with:
`You cannot duplicate the talent(s).`

---

## 3. The interviewer restriction: a security boundary

The Interviewer privilege is a deliberately narrow, record-scoped grant that lets a person
who has no recruitment responsibility take part in the evaluation of the candidates they
actually interview. It is enforced in four independent layers, and a rebuild that implements
only some of them leaks data.

| Layer | What it does |
|---|---|
| Model access rights | Grant read and write on the Application entity, and withhold create and delete |
| Record rule | Narrow those rights to the Applications the user is attached to |
| Field-level restriction | Remove the salary fields from the payload entirely |
| Guard on the hire action | Refuse the creation of an Employee even for a user who can otherwise write the record |

**REC-023.** An Interviewer may **read and update** an Application when the acting user is
among that Application's interviewers, **or** among the interviewers of that Application's
Job Position. The record rule grants no create right and no delete right.

**REC-024.** An Interviewer may **not create** an Application. The pipeline screens hide the
creation control, and a direct create is refused by the access rights.

**REC-025.** An Interviewer may **not delete** an Application, for the same reason.

**REC-026.** The four salary fields are removed from an Interviewer's read of the record
entirely: they are not returned, not merely hidden. Reading them explicitly raises the
platform's field-level access error.

**REC-027.** An Interviewer who is not an Officer is shown a reduced Application form from
which the whole salary group has been removed. The reduced form is selected by the server
when the form view is requested, not by the client.

**REC-028.** An Interviewer may **refuse** an Application they can reach, and may open the
refusal dialog. The dialog is granted to the Interviewer privilege exactly like the Officer
privilege.

**REC-029.** An Interviewer may **schedule a meeting** for an Application they can reach.
Because a bare Interviewer has no create right on Applications, the calendar screen is
opened with an explicit permission to create, which the calendar would otherwise refuse.

**REC-030.** An Interviewer may **send and read a written interview** for an Application they
can reach; the questionnaire record rules restate the same condition. An Interviewer may not
read the answers of an Application they are not attached to.

**REC-031.** An Interviewer may **not create an Employee**. A user who holds the Interviewer
privilege and not the Officer privilege is refused with:
`You are not allowed to perform this action.`

**REC-032.** An Interviewer may **write the interviewer list** of an Application they can
reach, and thereby grant the Interviewer privilege to another user. This is intended: it is
how an interviewer brings a colleague into an evaluation.

**REC-033.** The Interviewer privilege is granted automatically to every user named as an
interviewer on a Job Position or on an Application, with elevated rights, because granting a
privilege is not something a recruiter may normally do. A user who already holds the Officer
privilege is never given the narrower one, because the Officer privilege implies it.

**REC-034.** The Interviewer privilege is revoked automatically from a user who is removed
from an interviewer list and who is then named neither on any Job Position nor on any
Application, and who does not hold the Officer privilege. A user still named anywhere keeps
the privilege.

**REC-035.** An Interviewer may see the Job Positions they interview for. Two paths make
that work: the position's own interviewer list, and the position's derived *extended
interviewer* list, which is the union of the interviewers named on the position's individual
Applications. The second exists so that a person attached to a single Application can still
reach the position without a scan of every Application.

---

## 4. Pipeline, stage and status rules

**REC-036.** An Application that gains a Job Position and has no stage yet receives the
available stage with the lowest sequence among those **not folded**. An Application with no
position has no stage. Clearing the position clears the stage.

**REC-037.** An Application that already has a stage keeps it when its Job Position changes.
This is why the stage's selection domain also accepts the stage currently held.

**REC-038.** A stage is available to a position when the stage is attached to no position at
all, or when the position is among the positions the stage names. A stage attached to at
least one position is invisible to every other position.

**REC-039.** Stages are ordered by sequence ascending. The available stage with the lowest
sequence is the position's *first stage*.

**REC-040.** Every write that changes the stage performs, in the same operation and in this
order: the last-stage-update moment is set to now; the previous stage is recorded; the
readiness colour is reset to `normal` unless the same write names a colour; the position's
remaining target is adjusted by REC-042; and the hire date is recomputed by REC-041.

**REC-041.** The hire date is set to the current moment when the Application enters a stage
flagged as a hired stage **and no hire date is stored yet**, and is cleared as soon as the
current stage is not flagged as a hired stage. A hire date written by hand therefore
survives until the Application leaves the hired stage.

**REC-042.** Moving an Application from a non-hired stage into a hired stage decrements the
position's remaining target by one, **only when that value is greater than zero**. Moving it
from a hired stage into a non-hired stage increments the remaining target by one, without
any upper bound. A move between two stages carrying the same hired flag changes nothing. An
Application with no position adjusts nothing.

**REC-043.** The remaining target may never become negative. The storage check that enforces
it belongs to [Human Resources Core](../human-resources-core/README.md) and refuses with
`The expected number of new employees must be positive.` REC-042 respects it by declining to
decrement a zero.

**REC-044.** Writing a non-empty recruiter, at creation or later, stamps the assignment
moment with the current moment. Clearing the recruiter does **not** clear the assignment
moment.

**REC-045.** Every write that changes the readiness colour also sets the last-stage-update
moment to now, which restarts the staleness countdown.

**REC-046.** The derived application status is evaluated in this order and the first match
wins: a refusal reason is set → `refused`; the record is inactive → `archived`; a hire date
is set → `hired`; otherwise → `ongoing`. It is never stored.

**REC-047.** Searching on the application status uses the stored columns and is **not** the
mirror of REC-046. Only the "is one of" operator is supported; any other operator is
rejected as not implemented. The translations are: `refused` means inactive **and** a
refusal reason is set; `hired` means **active** and a hire date is set; `archived`, and the
empty value, mean inactive; `ongoing` means active and no hire date. Several requested
values are combined with a logical or.

**REC-048.** Two consequences of REC-047 must be reproduced, because reports depend on them:
a refused Application matches both `refused` and `archived`; an Application that carried a
hire date and was then archived matches `archived` only, not `hired`.

**REC-049.** Restoring an Application sets it active, clears the refusal reason and moves it
to the first non-folded available stage of its Job Position, or to no stage when it has no
position. No stage message is posted for that move.

**REC-050.** Archiving and restoring both run with an internal marker that suppresses the
automatic stage message for any stage change performed in the same operation.

**REC-051.** The automatic message of a stage is posted when **all** of the following hold:
the write changed the stage; the destination stage carries a message template; the operation
is not an archival or a restoration; and the operation is not the *add to job* operation of
the skills companion. The message uses the internal-note subtype, the light notification
layout without background, and its technical log copy is retained rather than deleted, so
that the message stays visible in the thread.

**REC-052.** A stage computed at creation — as opposed to written afterwards — produces no
stage message, because no stage change occurred. A stage written into the creation values,
as the public form and the add-to-job dialog do, **does** produce one.

**REC-053.** A stage may not be deleted while any Application points at it; the reference is
declared with restricted deletion. The Applications must be moved, archived or deleted
first.

**REC-054.** Clearing the hired flag on a stage that holds Applications is allowed. The form
warns beforehand with:
`All applications will lose their hired date and hired status.`
After the change, every Application in that stage loses its hire date at the next
recomputation and its status returns to `ongoing`. The targets already consumed are **not**
given back.

**REC-055.** An Application is stale when its derived status is `ongoing`, its hire date is
empty, its stage's staleness threshold is not zero, and the last-stage-update moment plus
that number of days is already past. The day count is given in
[calculations.md](calculations.md#7-staleness-rotting).

**REC-056.** Staleness may be searched only with equality operators; any other operator is
refused with:
`For performance reasons, use "=" operators on rotting fields.`

**REC-057.** When no stage in the installation defines a staleness threshold, the feature is
inactive and a search on it is refused with:
`Model configuration does not support the rotting feature`

**REC-058.** The time spent per stage is accumulated from the stage-change entries of the
thread plus the time since the last change. For a refused Application, the interval between
the refusal moment and now is subtracted from the current stage's total at every read, so a
refused Application does not accumulate time for ever.

**REC-059.** Archiving an Application changes nothing but the active flag. The stage, the
hire date and the refusal reason, if any, all survive.

**REC-060.** Archiving a Job Position archives every Application of that position, and, when
the public job pages companion is installed, clears the publication flag of positions that
were still active.

---

## 5. Contact synchronisation

The Contact is the shared, de-duplicated identity of a person; the Application is the local,
per-position copy. Two rules keep them in step, and their asymmetry is deliberate.

**REC-061. Forward, Contact to Application.** Writing the Contact overwrites the
Application's electronic mail address with the Contact's address **unconditionally**, and
fills the Application's telephone number from the Contact's number **only when the
Application's telephone number is empty**. An existing telephone number on the Application is
never overwritten by this direction. Applications with no Contact are left untouched, which
is what allows an Application to carry an address with no Contact behind it.

**REC-062. Inverse, Application to Contact.** Writing the electronic mail address or the
telephone number directly runs the inverse rule, per Application:

1. Normalise the written address. When it does not normalise to anything, do nothing for
   that Application.
2. When the Application has **no** Contact, apply REC-063 and REC-064.
3. When the Application **has** a Contact, apply REC-065.

**REC-063.** Creating a Contact from an Application requires a name. When the applicant's
name is empty, the write is refused with:
`You must define a Contact Name for this applicant.`

**REC-064.** With a name present, look for an existing Contact carrying that normalised
address. If one exists, attach it. If none exists, create one carrying the language of the
acting request, the applicant's name and the Application's telephone number, and attach it.
This is the **only** path in the domain that reuses an existing Contact; see REC-069.

**REC-065.** When the Application already has a Contact, push the three values down: rename
the Contact when the applicant's name differs from the Contact's name; write the
Application's address onto the Contact when the normalised addresses differ; write the
Application's telephone number onto the Contact when the numbers differ.

**REC-066.** A Contact that was created on an **earlier** day is never renamed by the thread
mechanism of REC-067. Only a Contact created today may be renamed that way. This protects an
existing customer or an existing internal user who applies for a position. Note the contrast
with REC-065, which does rename through the field inverse; the two paths differ and both are
observed behaviour.

**REC-067.** After a message is posted on an Application that carries an address and no
Contact, the recipients of that message are searched for a Contact whose address equals the
Application's address, comparing both the raw and the normalised forms. When one is found:
its name is overwritten with the applicant's name — or with the address when the applicant's
name is empty — **only if that Contact was created today**; and every Application that has
no Contact, whose address matches that Contact's address raw or normalised, and whose stage
is **not folded**, is attached to that Contact.

**REC-068.** Five operations create a Contact on demand, and they do not behave alike. The
differences are observable and must be reproduced:

| Operation | Refusal when the applicant's name is empty | Values written on the new Contact | Rights used | Reuses an existing Contact |
|---|---|---|---|---|
| Writing the address or the telephone number on an Application | `You must define a Contact Name for this applicant.` | language of the acting request, applicant's name, telephone number | the acting user's | **yes** |
| Scheduling a meeting | `You must define a Contact Name for this applicant.` | not an organisation, applicant's name, electronic mail address | the acting user's | no |
| Creating an Employee | `Please provide an applicant name.` | not an organisation, applicant's name, electronic mail address | the acting user's | no |
| Sending a written interview | `Please provide an applicant name.` | not an organisation, applicant's name, electronic mail address, telephone number | elevated | no |
| Sending a message from the message dialog | none: the Contact is created with whatever name is present | not an organisation, applicant's name, electronic mail address, telephone number | the acting user's, with the reading context cleaned | no |

**REC-069.** **compatibility finding** — because four of the five paths of REC-068 create a
Contact unconditionally, sending a message to an Application that has no Contact creates a
second Contact even when one with the same address already exists. The corrected behaviour
resolves the address first, exactly as the field inverse does, and creates only when nothing
matches.

**REC-070.** The customer information the messaging layer derives from an Application is
keyed on the normalised address, falling back to the raw address, and carries the applicant's
name — or, when that is empty, the name parsed from the address, or the address itself — and
the Application's telephone number. When several Applications are read at once, those with
no address key are skipped so that unrelated records are not merged.

---

## 6. Intake through the inbound address and through job boards

**REC-071.** Every Job Position owns exactly one inbound electronic mail alias, created with
the position. Its target entity is the Application entity.

**REC-072.** The alias default values are the position, the position's department, the
department's company when it has one and otherwise the position's company, and the
position's recruiter. They are written at creation and rebuilt whenever the department or
the recruiter of the position changes, because the alias is created once and its defaults
would otherwise go stale.

**REC-073.** An Application created by the inbound gateway never inherits the gateway's
technical user as recruiter: the default recruiter of the acting context is forced to empty
before creation, and the recruiter is then derived from the position.

**REC-074.** The stage proposed for an Application created through a position's alias is that
position's available stage with the lowest sequence, **folded stages included**. Every other
creation path excludes folded stages. With the shipped configuration this makes no
difference, because the folded stage has the highest sequence.

**REC-075.** The alias default values are applied **after** every value proposed by the
intake logic and therefore win over all of them.

**REC-076.** When the normalised sender address of an inbound message equals the address of a
Job Board, the created Application receives **no** electronic mail address and **no**
Contact, because the address belongs to the board and not to the person. The sender address
is removed from the parsed message so that the generic creation cannot restore it.

**REC-077.** For such a message the board's extraction pattern is applied first to the
subject and then to the body; the captures are concatenated in that order and the **first**
capture becomes the applicant's name. When nothing is captured, the display name parsed from
the sender is kept. An empty pattern captures nothing.

**REC-078.** The address of a Job Board is normalised on creation and on every write that
supplies it; when it cannot be normalised the raw value is kept. It must be unique:
`The Email must be unique, this one already corresponds to another Job Platform.`

**REC-079.** A priority carried by the inbound message becomes the Application's evaluation.

**REC-080.** The generic creation fills the display field from the message subject **only
when the applicant's name is still empty**, and fills the primary electronic mail field from
the sender address **only when that address is still present in the parsed message**. This is
exactly why REC-076 removes it.

**REC-081.** Immediately after an Application is created from an inbound message, the
address-and-telephone computation is re-run explicitly, so that an Application that received
a Contact from the messaging layer picks up that Contact's address and telephone number.

**REC-082.** The reply address of a message posted on an Application is the inbound alias of
its Job Position when the position has one; an Application with no position falls back to the
generic reply resolution of the messaging layer.

**REC-083.** The company of an Application created from an inbound message is derived by
REC-009 and is therefore **not** the technical user's company. A position belonging to a
second company, whose department has no company, still produces an Application in that
second company.

**REC-084.** The attachments of the inbound message become attachments of the Application.
The first suitable document becomes the main attachment shown in the preview panel, and the
extracted text of the attachments is indexed so that applications can be searched by the
content of the curriculum vitae.

**REC-085.** A Recruitment Source may own its own inbound alias. Its default values are the
position, the shipped campaign `Job Campaign`, the shared medium named `email` — fetched or
created on demand — and this source's Tracking Source. Its local part is the position's own
local part when it has one, otherwise the position's name, followed by a plus sign and the
source's name. Before the alias is created with elevated rights, the acting user's right to
**create** a Recruitment Source is verified explicitly.

---

## 7. The public application form

Present when the public job pages companion is installed.

**REC-086.** The public form may write exactly these seven fields, and nothing else: the
electronic mail address, the applicant's name, the telephone number, the Job Position, the
Department, the professional network profile address and the custom application properties.
Anything else posted becomes a custom entry in a note, never a field.

**REC-087.** The forgery token is validated **only** when the visitor has a signed-in
session. An anonymous submission is accepted without a token, because an embedded form
routinely loses its session marker. A signed-in visitor whose token fails is refused with the
platform's expired-session message.

**REC-088.** A submission naming an archived Job Position is refused with:
`The job offer has been closed.`
The position is loaded with elevated rights for that check, because a visitor cannot read an
archived position.

**REC-089.** A submission naming a Job Position receives that position's available stage with
the lowest sequence among those **not folded**, written into the creation values.

**REC-090.** When the submitted values already name a Contact, the electronic mail address
and the telephone number are removed from the values before creation, so that a submitted
Contact is never overwritten by form input.

**REC-091.** Fields declared required on the entity and absent from the submission cause the
submission to be rejected, and the answer names the offending fields.

**REC-092.** The Application is created with full rights and with the automatic subscription
of the submitting user suppressed.

**REC-093.** The custom entries are posted as one note in the thread: the line
`Other Information:`, then a line of underscores, then an empty line, then one line per
custom entry of the form *name*, a space, a colon, a space, *value*. The custom entry named
`short_introduction` is relabelled `Short introduction from applicant` before the note is
built.

**REC-094.** When the platform-wide metadata switch is on, the same note also lists, under
the heading *Metadata*, the visitor's network address, browser identification, accepted
languages and referring page.

**REC-095.** Files that match no writable field of the entity are stored as attachments of the
new Application and announced by a second note whose body is `Attached files: `.

**REC-096.** No "authenticated as" note is logged on an Application submitted by a visitor
with no session.

**REC-097.** The form requires **either** a file in the résumé field **or** a value in the
professional network field. When both are empty and the visitor presses the submission
control, both fields become required and the page refuses to send the form.

**REC-098.** A professional network profile address that does not have the shape of such an
address produces, under the field, the warning:
`The profile that you gave us doesn't seems like a linkedin profile`
The field is highlighted and the submission is **not** blocked. **compatibility finding** —
the wording is ungrammatical; a corrected wording reads "The profile you gave us does not
look like a professional network profile". The stored text must nevertheless be reproduced
by a rebuild that wants identical behaviour.

**REC-099.** The live duplicate check answers exactly one of four things, evaluated in this
order. It never blocks the submission.

| Order | Condition | Answer |
|---|---|---|
| 1 | Some matching Application is refused, inactive, belongs to this very position and was created within the last six months | `We've found a previous closed application in our system within the last 6 months. Please consider before applying in order not to duplicate efforts.` |
| 2 | No matching Application has the status `ongoing` | No message at all; any warning already shown is hidden |
| 3 | The newest matching ongoing Application belongs to this very position | `An application already exists for ` + the value typed + `. Duplicates might be rejected. ` followed, when that Application has a recruiter, by ` In case of issue, contact ` and the recruiter's name, electronic mail address and telephone number joined by a comma and a space, omitting the parts that are empty |
| 4 | Otherwise | `We found a recent application with a similar name, email, phone number. You can continue if it's not a mistake.` |

**compatibility finding** — the third answer is assembled from a sentence that already ends
with a space and a fragment that begins with a space, so the delivered text contains two
consecutive spaces before *In case of issue*, and ends with a single trailing space when the
Application has no recruiter. A corrected behaviour trims the result. A rebuild that asserts
on the exact text must reproduce the spacing.

**REC-100.** The matching condition of the live check compares the **raw** telephone number,
where every other duplicate rule of the domain compares the sanitised number. See
[calculations.md](calculations.md#45-the-live-warning-on-the-public-application-form).

---

## 8. Refusal

**REC-101.** A refusal always carries a reason. The reason field of the dialog is required,
and its default is the **first** Refusal Reason in sequence order.

**REC-102.** Choosing a reason adopts that reason's message template, but only when the
template is **active**. An archived template leaves the dialog without a template, which in
turn proposes not to send a message.

**REC-103.** Adopting a template copies its subject, its body, its attachments and its
scheduled date into the dialog. When exactly one Application is selected, the subject and the
body are rendered against that Application immediately, so the user sees the final text.

**REC-104.** The message switch is proposed as on when a template is set, that template is
active, and **every** selected Application has an address; it is proposed as off otherwise.

**REC-105.** When at least one selected Application has neither its own address nor an
address on its Contact, the dialog shows a warning made of three lines: the sentence
`You can't select Send email option.`, then the sentence
`The email will not be sent to the following applicant(s) as they don't have an email address:`,
then the names of those Applications separated by a comma and a space. An Application with no
name contributes its display name, which for an unnamed Application is the empty string.

**REC-106.** Applying a refusal with the message switch on, while the acting user has no
electronic mail address, is refused with:
`Unable to post message, please configure the sender's email address.`

**REC-107.** Applying a refusal with the message switch on, while at least one selected
Application has neither its own address nor an address on its Contact, is refused with:
`At least one applicant doesn't have a email; you can't use send email option.`

**REC-108.** The duplicate candidates of a refusal are the Applications of the same people,
matched by the condition of
[calculations.md](calculations.md#42-refusal-duplicate-set), excluding the selected
Applications themselves and keeping only those whose derived status is neither `hired` nor
`refused` nor `archived`.

**REC-109.** Each refused duplicate receives the log line:
`Refused automatically because this application has been identified as a duplicate of ` +
a link bearing the original Application's display name. The original is resolved by testing,
in this order, the identifier, the normalised address, the sanitised telephone number and the
professional network profile address, keeping the first selected Application that matches. A
duplicate matching none of the four receives no line.

**REC-110.** Applying a refusal writes, on every Application of the refusal set in a single
operation: the chosen reason, the active flag set to false, and the refusal moment set to the
current moment.

**REC-111.** The refusal message is posted only on the **selected** Applications, never on
the duplicates that were added automatically, so that the person receives exactly one
message.

**REC-112.** Each refusal message is rendered per recipient: the language is resolved per
Application from the template's language expression, and the subject and the body are
rendered against that Application in that language.

**REC-113.** The sender of a refusal message is the template's own sender address when the
template defines one, and the acting user's formatted address otherwise. The author is always
the acting user's Contact, and the only recipient is the Application's Contact.

**REC-114.** The scheduled date carried by the dialog is a text value read as coordinated
universal time. The message may not leave before that moment.

**REC-115.** A refusal does **not** change the stage, and therefore does not change the
position's remaining target and does not clear a hire date. A hired Application that is
refused keeps its hire date and reads as `refused`, because the refusal branch of REC-046 is
evaluated first.

---

## 9. Multi-record stage writes

**REC-116.** The target arithmetic of REC-042 is applied **per record**, correctly, when a
stage is written onto several Applications at once.

**REC-117.** **compatibility finding** — the previous-stage stamp is not applied per record.
When a stage is written onto several Applications in one operation, the previous-stage value
computed for each record in turn is written into the shared value set, so all the records
end up carrying the previous stage of the **last** record processed. Only the lost-case
analysis is affected; no other rule reads the previous stage. The corrected behaviour writes
each record's own previous stage.

**REC-118.** The last-stage-update moment, the readiness reset and the hire-date
recomputation are applied to every record of a multi-record write, without anomaly.

---

## 10. Hiring and employee creation

**REC-119.** The *Create Employee* control is offered only when the Application is active,
carries a hire date and has no Employee yet, and only to the Human Resources Officer
privilege.

**REC-120.** A user holding the Interviewer privilege and not the Officer privilege is
refused with:
`You are not allowed to perform this action.`

**REC-121.** When the Application has no Contact, one is created as a person — not an
organisation — with the applicant's name and the Application's electronic mail address. When
the applicant's name is empty the operation is refused with:
`Please provide an applicant name.`

**REC-122.** The Employee is created with the field mapping listed in
[workflows.md](workflows.md#83-the-field-mapping-named-field-by-field). Three of its entries
are counter-intuitive and are restated here: the work address is the contact record of the
Application's **company**; the work electronic mail address is the address of the
**department's** company, falling back to the Application's own address when that company has
none; the work telephone number is the telephone number of the department's company.

**REC-123.** The Employee is created with the creation context cleaned, so that no default
value inherited from the screen the operation was launched from leaks into the new record.

**REC-124.** Attachments are copied from the Application to the new Employee, skipping any
file whose binary content is already attached to that Employee. The comparison is on the
content itself, so running the operation twice does not duplicate files.

**REC-125.** After the Employee exists, a second write re-applies the Job Position, the job
title, the department, the work electronic mail address and the work telephone number from
the same sources. The Employee recomputes several of those fields from the employment version
created alongside it, and the second write re-imposes the values taken from the Application.
Both writes are observable and must be applied in this order.

**REC-126.** Creating an Employee that carries Applications posts on each of those
Applications the log line `Employee created: ` followed by a link bearing the Employee's
name.

**REC-127.** The Application and the Employee point at each other after the operation: the
Employee's applicant list contains the Application, and the Application's employee reference
is the new Employee.

**REC-128.** An Application may hold the status `hired` without an Employee ever being
created, for instance when the person is enrolled in another system. The status depends on
the hire date alone.

**REC-129.** The Hired counter of a position counts Applications carrying a hire date,
**archived ones included**, which keeps the historical number of hires stable after
applications are archived.

**REC-130.** The set of contacts considered related to an Employee is extended with the
contacts of the Employee's Applications, read with elevated rights. This is what keeps the
Employee's duplicate-contact counter correct and what makes the unsubscription rule of
REC-215 skip a recruiter who is also the department manager's related contact.

---

## 11. Talent pools

**REC-131.** An Application whose pool list is non-empty is a **talent**. A talent's
canonical pool copy is itself.

**REC-132.** An Application created with pools and without a canonical pool copy is made to
point at itself at creation time.

**REC-133.** The Applications offered by the add-to-pool dialog are those that are already
talents, or that are linked to no pool directly or indirectly. An Application already linked
cannot be pooled a second time through a different copy.

**REC-134.** Adding a record that **is already a talent** links the chosen pools and the
chosen tags onto it. No record is created.

**REC-135.** Adding a record that is **not** a talent duplicates it: the copy carries no Job
Position, the chosen pools, and the union of the original's tags and the chosen tags. The
copy's canonical pool copy is then pointed at itself, and the original's canonical pool copy
is pointed at the copy.

**REC-136.** The duplication performed by the pool mechanism does **not** append the copy
suffix to the applicant's name. Every other duplication of an Application appends ` (copy)`.

**REC-137.** Tags chosen in the add-to-pool dialog are written on the talent only. The
originating Application keeps its own tags unchanged.

**REC-138.** The whole add-to-pool operation runs with elevated rights, so an Officer who has
no rights on the Employee register can still pool a candidate.

**REC-139.** Writing the electronic mail address, the telephone number, the professional
network profile address or the degree on an Application that points at a canonical pool copy,
and that is not itself a talent, copies the new value onto that canonical pool copy. Only the
fields that were part of the write are copied.

**REC-140.** Writing skills on such an Application writes the equivalent operations onto the
canonical pool copy, matched by skill: an update becomes an update when the talent holds that
skill and a creation otherwise; a deletion becomes a deletion when the talent holds that
skill and is dropped otherwise; anything else is passed through unchanged.

**REC-141.** Creating Applications out of talents clears the pool list on the new records,
sets the chosen Job Position, and sets the stage explicitly to the lowest-sequence non-folded
stage among those available to that position. Because the stage is part of the creation
values, the destination stage's message template **is** applied at creation.

**REC-142.** The new Applications created out of a talent keep their canonical pool copy
pointing at the talent, which keeps them linked to the pool.

**REC-143.** A record is "in a pool" when it carries at least one pool, when it points at a
canonical pool copy, or when it shares a normalised address, a sanitised telephone number or a
professional network profile address with a record satisfying one of the first two
conditions. Linking through a record that is itself only indirectly linked is deliberately
**not** transitive.

**REC-144.** The statistic control that opens the talent, when pressed on an Application that
has no canonical pool copy but is nevertheless linked to a pool indirectly, first resolves and
stores the link by searching for a talent matching any of the three keys, and then opens it.

**REC-145.** When the add-to-pool operation produces exactly one talent, the interface opens
it; otherwise the current screen is reloaded. When the add-to-job operation produces exactly
one Application, the interface opens it; otherwise it shows the notification
`Created ` + the number created + ` new applications for: ` + the distinct applicant names
separated by a comma and a space.

---

## 12. Protection of attribution records

**REC-146.** The shipped recruitment campaign, whose external identifier is
`hr_recruitment.utm_campaign_job` and whose name is `Job Campaign`, may not be deleted,
because every Recruitment Source alias stamps it on the Applications it creates. Deleting it
is refused with:
`The UTM campaign '%s' cannot be deleted as it is used in the recruitment process.`
The placeholder carries that campaign's name. The three capital letters at the start of the
message are the abbreviation the attribution convention uses for its own vocabulary; the
message is reproduced because support procedures and tests key on it.

**REC-147.** A Tracking Source referenced by at least one Recruitment Source may not be
deleted. Deleting it is refused with:
`You cannot delete these UTM Sources as they are linked to the following recruitment sources in Recruitment:`
followed by a new line and the names of the Job Positions concerned, each between double
quotation marks and separated by a comma and a space. Same remark on the abbreviation.

**REC-148.** REC-147 duplicates a storage-level deletion restriction on the same reference.
Its only purpose is to replace a technical reference error by a readable sentence, and a
rebuild must keep both, because the readable message is only reachable when the guard runs
first.

**REC-149.** The three attribution references on an Application — campaign, medium and source
— are declared so that deleting the referenced record **clears the reference** rather than
blocking the deletion. This domain changes them from the shared definition's default for
exactly that reason: deleting a campaign or a medium must never be blocked by existing
applications.

---

## 13. Skills

Present when the skills companion is installed.

**REC-150.** Skill lines are never updated in place. Adding a skill, or changing the level of
a skill the person already holds, closes the previous line for the same skill with a validity
end of yesterday, or deletes it when it was created today or has already expired, and creates
a new line starting today.

**REC-151.** Two lines of the same Application for the same non-certification skill may not
have overlapping validity windows, and two identical certification lines — same skill, same
level, same validity start, same validity end — may not coexist. The refusal is:
`The following skills can't be created as they overlap or exactly match existing skills:`
followed by one bullet line per conflict reading the new line's display name, then
` conflicts with the existing skill/certification `, then the existing line's display name,
then ` from `, the existing validity start, ` to `, and the existing validity end.

**REC-152.** A validity end earlier than the validity start is refused with:
`The following skills have their valid stop date prior to their valid start date:`
followed by one bullet line per offending record reading the skill's display name, then
` from `, the validity start, ` to `, and the validity end.

**REC-153.** The skill must belong to the chosen skill type:
`The skill %(name)s and skill type %(type)s don't match`
The first placeholder carries the skill's name, the second the skill type's name.

**REC-154.** The level must belong to the chosen skill type:
`The skill level %(level)s is not valid for skill type: %(type)s`
The first placeholder carries the level's name, the second the skill type's name.

**REC-155.** The current skills of an Application are the lines whose validity end is empty or
not earlier than today. For a certification type with no currently valid line, the line with
the latest validity end is kept instead, so that an expired certification stays visible.

**REC-156.** Adding the same skill twice in one save keeps one line; the repetition is dropped
before the operations are executed.

**REC-157.** Creating an Employee from an Application copies every skill line of the
Application as an Employee skill carrying the same skill, the same level and the same skill
type. Validity windows are **not** carried over; the Employee skill lines are created without
dates.

**REC-158.** The stored distinct-skill list of an Application is derived from its skill lines
and exists so that "applications holding any of these skills" can be searched efficiently. It
includes the skills of expired lines, because it is derived from the full line set and not
from the current set.

**REC-159.** The matching skills, the missing skills and the match score are computed against
the Job Position named by the reading context, and against the Application's own position
when the context names none. With no position, or with a position that requires neither a
skill nor a degree, all three are empty or zero.

**REC-160.** The *Add to job* operation of the matching list writes the Job Position and the
shipped first stage onto the selected Application with the *just moved* marker set, which
suppresses the destination stage's message, and then opens the applications of that position.

**REC-161.** The matching-applicants search from a position excludes the Applications of that
position itself, and keeps only Applications holding at least one of the skills the position
requires. When nothing matches, the list shows `No Matching Applicants` followed by
`We do not have any applicants who meet the skill requirements for this job position in the database at the moment.`

**REC-162.** An Interviewer may act on the skill lines of the Applications they interview,
directly or through the position; an Officer may act on all of them.

---

## 14. Access rights, record rules and menu visibility

The complete matrices are in [configuration.md](configuration.md#6-access-rights-matrix) and
[configuration.md](configuration.md#7-record-rules). The rules below state the decisions those
matrices encode.

**REC-163.** Three privileges form a chain: Interviewer implies the internal-user privilege;
Officer implies Interviewer; Administrator implies Officer. All three belong to one privilege
group of the human-resources category.

**REC-164.** Recruitment Stages are readable by the Interviewer and Officer privileges and
writable, creatable and deletable only by the Administrator privilege. An Officer configures
the pipeline only by asking an Administrator.

**REC-165.** Job Boards are readable, writable, creatable and deletable only by the
Administrator privilege.

**REC-166.** Recruitment activity plans and their step templates are managed by the
Administrator privilege, and only for plans whose target entity is the Application entity.

**REC-167.** Application Tags may be read, created and updated by **every internal user**, and
deleted only by the Officer privilege. This is deliberate: a tag is a shared vocabulary and
any colleague may extend it.

**REC-168.** Recruitment Sources may be read by every internal user and managed by the Officer
privilege.

**REC-169.** Talent Pools are readable by the Interviewer privilege and managed by the Officer
privilege. The talent list of a pool is readable by every internal user.

**REC-170.** Degrees are managed by the Officer privilege. Refusal Reasons are readable by the
Interviewer privilege and managed by the Officer privilege.

**REC-171.** The Officer privilege carries full rights on Contacts and on Meetings, because a
recruiter must be able to create the Contact of a candidate and to schedule an interview. It
also carries a record rule granting it every message, so that a recruiter can read a thread
whose record they reached through the pipeline.

**REC-172.** The add-to-pool dialog and the add-to-job dialog are **explicitly denied** to the
Interviewer privilege, with all four rights set to none, and granted to the Officer privilege.
The refusal dialog and the message dialog are granted to both.

**REC-173.** Every internal user is granted the display privilege whose reproduced name is
`Display CV on application form`, which is what makes the preview panel of the curriculum
vitae appear beside the application form.

**REC-174.** Installing the public job pages companion grants the restricted site-editor
privilege to the Officer privilege, so that a recruiter can edit the public job pages.

**REC-175.** Public visitors and portal users may read exactly the Job Positions whose
publication flag is true. Internal users may read every position through the base access
right. The Officer privilege may read every position through the public site but may not
write, create or delete through it.

**REC-176.** A public visitor may read a Department when at least one of its positions is
published, or when the department has child departments. This is what lets the public job
list build its department filter.

**REC-177.** The department's display name is computed with elevated rights when the public
job pages companion is installed, because a visitor browsing the job list must see department
names without being able to read departments.

**REC-178.** Menu visibility follows the privileges: the root recruitment menu is visible to
the Officer and Interviewer privileges; the reporting menu and the configuration menu to the
Officer privilege; activity plans and written interviews to the Administrator privilege; the
settings screen to the platform administrator. Exactly one of the two *By Job Positions*
entries is shown to each reader, as described in
[entities.md](entities.md#1210-menu-entry-iruimenu).

**REC-179.** Posting a message on an Application requires read access only, not write access,
which lets interviewers and followers answer in the thread.

**REC-180.** Record rules on the written-interview entities restate the interviewer
restriction: an Administrator acts on every recruitment questionnaire, its questions, its
answers, its answer sets, its answer lines and its invitations, with deletion granted
everywhere except on invitations; an Officer reads the answer sets and answer lines of
recruitment questionnaires that are unrestricted or that name them among their restricted
users, and may create and update invitations under the same condition; an Interviewer reads
the answer sets and answer lines of the Applications they interview, reads the questionnaires
and questions of the positions they interview for, and may create and update invitations for
those positions.

---

## 15. Rules of the configuration entities

**REC-181.** Application Tag names are unique. A duplicate is refused with:
`Tag name already exists!`

**REC-182.** Degree names are unique. A duplicate is refused with:
`The name of the Degree of Recruitment must be unique!`

**REC-183.** A Degree's score must lie between 0 and 1 inclusive. A value outside that range
is refused with:
`Score should be between 0 and 100%`
The message states the range as a percentage while the stored value is a fraction; both are
reproduced as they are.

**REC-184.** A stage created from inside a position's pipeline is **global**: the default
position carried by the screen is removed from the creation defaults, unless the caller
explicitly asks for a position-specific stage through the single-position marker. The
configurator restricts a stage deliberately by naming positions on it.

**REC-185.** The four readiness labels of a stage are required and translatable. Their
defaults are `In Progress`, `Ready for Next Stage`, `Waiting` and `Blocked`.

**REC-186.** Archived Refusal Reasons are not offered in the refusal dialog. Reasons are
ordered by sequence ascending, and the sequence is **not** copied when a reason is
duplicated.

**REC-187.** A Recruitment Source can produce an inbound address only when an alias domain
exists on the position's company or on the acting user's company. Otherwise the derived
*has domain* value is false and the address control is hidden.

**REC-188.** Deleting a Recruitment Source deletes its alias afterwards, with elevated rights,
so that no orphan inbound address survives. Deleting a Job Position deletes its Recruitment
Sources.

**REC-189.** The default medium of a new Recruitment Source is the shared medium named
`website`, fetched or created on demand.

**REC-190.** A Job Position's name must be unique per department within a company. A
duplicate is refused with:
`The name of the job position must be unique per department in company!`
The rule is owned by [Human Resources Core](../human-resources-core/README.md) and is
restated here because every recruitment screen can trigger it.

**REC-191.** The default job location of a new position is the location of the most recently
created position among the acting user's companies, or, when there is none, the contact
record of the acting user's current company.

**REC-192.** The job locations offered are the child contacts of the allowed companies'
contacts that are neither of the *contact* kind nor of the *private* kind, plus the
companies' own contacts.

**REC-193.** The favourites list of a new Job Position is forced to the value submitted or,
when nothing was submitted, to the empty list. This cancels the field's own default, so a
position created through the interface starts with **no** favourite.

**REC-194.** Sorting positions by the favourite flag is translated into a test of membership
in the acting user's favourites, which lets a list be sorted with the reader's own favourites
first.

**REC-195.** The default ordering of Job Positions is changed by this domain from sequence
alone to sequence ascending, then name ascending.

---

## 16. Rules of the public job list

Present when the public job pages companion is installed.

**REC-196.** Publishing a Job Position sets both the recruitment publication flag and the
generic publication flag, and stamps the published date with today's date. Unpublishing
clears all three.

**REC-197.** The *set open* operation of the publication mixin clears the recruitment
publication flag first, and the mixin then clears the generic flag.

**REC-198.** The public page of a position is at the relative path `/jobs/` followed by the
position's readable identifier; its application form at `/jobs/apply/` followed by the same
identifier; and the legacy path `/jobs/detail/` followed by the same identifier answers with
a permanent redirection to the first.

**REC-199.** The public job list shows twelve positions per page.

**REC-200.** The list searches at most six hundred positions, that is twelve multiplied by
fifty, ordered by publication state descending, then sequence ascending, then remaining target
descending.

**REC-201.** When the visitor applies no country, department, office or employment-type filter
and does not ask for every country, the country detected from the visitor's network address is
applied as a default filter, but **only** when at least one published position has a job
location in that country; otherwise no country filter is applied.

**REC-202.** A position published on one website is visible only on that website; a position
restricted to no website is visible on every website.

**REC-203.** The thank-you page is never served from the page cache, because it renders
information about the application just submitted.

**REC-204.** Creating a position from the site editor creates it with the name `Job Title` and
answers with the relative address of its new public page.

**REC-205.** Job Positions join the site-wide search under the search kind `jobs`. The
searchable fields are the position name and, when the caller asks for descriptions, the job
description. A search filtered by country is executed with elevated rights, and the published
condition is therefore re-imposed explicitly for any reader who is not an Officer.

**REC-206.** The public job list is offered among the suggested pages when a website is being
built, under the name *Jobs*, pointing at the path `/jobs`.

---

## 17. Messaging, notification and reporting rules

**REC-207.** The message subtypes of this domain are: on the Application entity,
*New Applicant* (hidden, off by default), *Stage Changed* (off by default), *Applicant Hired*
(on by default) and *New Talent* (off by default); on the Job Position entity,
*Job Position created* (hidden, off by default), *Applicant Stage Changed*, *Applicant Hired*
and *New Applicant*, each linked to its Application-level counterpart through the position
reference; on the Department entity, *Job Position Created*.

**REC-208.** The creation message of an Application uses the subtype *New Applicant*, or
*New Talent* when the record is a talent. A stage change uses *Stage Changed*.

**REC-209.** Naming a new interviewer on an Application notifies that user, unless the user is
the one performing the change, with the subject
`You have been assigned as an interviewer for ` followed by the Application's display name,
and the body `You have been assigned as an interviewer for the Applicant ` followed by the
applicant's name. The notification is authored by the acting user, sent from the acting user's
formatted address, and uses the standard notification layout with the model described as
*Applicant*.

**REC-210.** Naming a new interviewer on a Job Position grants the privilege but sends **no**
notification. Only the Application-level list notifies.

**REC-211.** Sending a message from the bulk message dialog is aborted, with nothing written,
when any selected Application has no address of its own or has a Contact with no address. The
notification shown is:
`The following applicants are missing an email address: ` followed by the names separated by a
comma and a space, and a full stop.

**REC-212.** Each attachment of the bulk message dialog is duplicated once per recipient
Application and re-owned by that Application before the message is posted, so that every
recipient's thread carries its own copy.

**REC-213.** The recruitment indicator of the periodic digest raises an access error carrying
`Do not have access, skip this data for user's digest email` for a reader below the Officer
privilege, which makes the digest engine skip the indicator for that reader instead of failing
the whole digest.

**REC-214.** The personal activity counter of a position counts the activities of the acting
user, on active Applications of that position, whose stage is not flagged as hired, and whose
activity is itself active.

**REC-215.** Changing the recruiter of a Job Position unsubscribes the previous recruiter's
contacts from the position's thread, except those that are related contacts of the department
manager; applies the same unsubscription to the position's Applications whose status is
`ongoing` and whose recruiter was the previous one; and reassigns exactly those Applications
to the new recruiter with the assignment notification suppressed. Applications that are hired,
refused or archived keep the previous recruiter, which preserves the history.

**REC-216.** The fields tracked in an Application's thread are: the stage, the recruiter, the
interviewers, the company, the department, the Job Position, the availability, the hire date,
the refusal reason and the four salary fields. The tracked fields of a Talent Pool are the
company and the pool manager. The fields this domain adds to the tracking of a Job Position
are the job location, the industry and the publication flag.

**REC-217.** An Application entering a stage that carries a message template receives the
rendered template as an internal note, addressed to the candidate's Contact, with the light
notification layout without background.

**REC-218.** Completing a written interview linked to an Application posts on that Application
the message `The applicant "` + the applicant's name + `" has finished the survey.`, authored
by the platform's system contact rather than by the candidate.

---

## 18. Locking, concurrency and ordering of side effects

**REC-219.** The domain takes no explicit lock. Concurrency is handled by the storage layer's
own record-level protection, and a rebuild must reproduce the **order** of the side effects
rather than any locking scheme.

**REC-220.** On an Application write the order is: adjust the value set (assignment moment,
last-stage-update moment, readiness reset, previous stage); adjust the position's remaining
target; perform the write; propagate the four fields to the canonical pool copy; then grant,
revoke and notify interviewers. A rebuild that performs the target adjustment after the write
produces a different result when the same operation also changes the position.

**REC-221.** On a Job Position write the order is: remember the interviewers, the department
manager and the recruiter; deactivate the Applications when the position is being deactivated;
perform the write; adjust the interviewer privileges; apply the recruiter reassignment of
REC-215; rewrite the alias default values when the department or the recruiter changed.

**REC-222.** On an Application creation the order is: stamp the assignment moment and strip
the address in the submitted values; create the records; grant the Interviewer privilege;
point a pooled record's canonical pool copy at itself; notify the new interviewers.

---

## 19. Index of rule identifiers

| Identifier | Subject |
|---|---|
| REC-001 … REC-022 | Application data integrity: required fields, lengths, normalisation, company derivation, selection domains, salaries, talents |
| REC-023 … REC-035 | The interviewer restriction, in four enforcement layers |
| REC-036 … REC-060 | Pipeline: landing stage, stage writes, target arithmetic, hire date, derived status and its search, staleness, archival |
| REC-061 … REC-070 | Contact synchronisation in both directions, and the five on-demand creation paths |
| REC-071 … REC-085 | Intake through the position's inbound address, job boards, and per-source addresses |
| REC-086 … REC-100 | The public application form: writable fields, guards, notes, attachments, live duplicate warning |
| REC-101 … REC-115 | Refusal: reason, template, message guards, duplicates, what refusal does and does not change |
| REC-116 … REC-118 | Multi-record stage writes |
| REC-119 … REC-130 | Hiring, the Employee mapping, attachments, the second write, the two-way link |
| REC-131 … REC-145 | Talent pools: talents, pooling, propagation, pushing talents into positions |
| REC-146 … REC-149 | Protection of the attribution records |
| REC-150 … REC-162 | Skills: versioning, overlap, validity, current set, transfer at hire, matching |
| REC-163 … REC-180 | Access rights, record rules, menu visibility, public read access |
| REC-181 … REC-195 | Configuration entities: uniqueness, scores, stage defaults, sources, positions |
| REC-196 … REC-206 | The public job list: publication, paths, paging, ordering, default country, caching, search |
| REC-207 … REC-218 | Message subtypes, notifications, digest, tracked fields, recruiter reassignment |
| REC-219 … REC-222 | Ordering of side effects |

---

## 20. Mapping of the identifiers used by the two earlier drafts

The two independently written descriptions of this domain that were merged into this folder
used different schemes. One numbered its rules `REC-RULE-nnn`; the other used chapter numbers
inside its own business-rules file, which the rest of that draft referenced by name. Both are
mapped here so that no earlier reference is orphaned.

### 20.1 From the numbered scheme

| Former identifier | Present identifier |
|---|---|
| REC-RULE-001 | REC-001 |
| REC-RULE-002 | REC-002 |
| REC-RULE-003 | REC-003 |
| REC-RULE-004 | REC-004 |
| REC-RULE-005 | REC-005 |
| REC-RULE-006 | REC-006 |
| REC-RULE-007 | REC-061 |
| REC-RULE-008 | REC-065 |
| REC-RULE-009 | REC-064 |
| REC-RULE-010 | REC-063 |
| REC-RULE-011 | REC-008 |
| REC-RULE-012 | REC-009 |
| REC-RULE-013 | REC-010 |
| REC-RULE-014 | REC-011 |
| REC-RULE-015 | REC-012 |
| REC-RULE-016 | REC-013 |
| REC-RULE-017 | REC-014, REC-026, REC-027 |
| REC-RULE-018 | REC-015 |
| REC-RULE-019 | REC-016 |
| REC-RULE-020 | REC-036 |
| REC-RULE-021 | REC-040 |
| REC-RULE-022 | REC-045 |
| REC-RULE-023 | REC-041 |
| REC-RULE-024 | REC-042 |
| REC-RULE-025 | REC-043 |
| REC-RULE-026 | REC-044 |
| REC-RULE-027 | REC-046 |
| REC-RULE-028 | REC-047, REC-048 |
| REC-RULE-029 | REC-049 |
| REC-RULE-030 | REC-050 |
| REC-RULE-031 | REC-051 |
| REC-RULE-032 | REC-053 |
| REC-RULE-033 | REC-054 |
| REC-RULE-034 | REC-055 |
| REC-RULE-035 | REC-056, REC-057 |
| REC-RULE-036 | REC-058 |
| REC-RULE-037 | REC-019 |
| REC-RULE-038 | REC-039 |
| REC-RULE-039 | REC-038 |
| REC-RULE-040 | REC-071 |
| REC-RULE-041 | REC-072 |
| REC-RULE-042 | REC-073 |
| REC-RULE-043 | REC-074 |
| REC-RULE-044 | REC-076 |
| REC-RULE-045 | REC-077 |
| REC-RULE-046 | REC-078 |
| REC-RULE-047 | REC-079 |
| REC-RULE-048 | REC-075 |
| REC-RULE-049 | REC-082 |
| REC-RULE-050 | REC-086 |
| REC-RULE-051 | REC-097 |
| REC-RULE-052 | REC-088 |
| REC-RULE-053 | REC-089 |
| REC-RULE-054 | REC-090 |
| REC-RULE-055 | REC-093 |
| REC-RULE-056 | REC-095 |
| REC-RULE-057 | REC-066 |
| REC-RULE-058 | REC-096 |
| REC-RULE-059 | REC-099 |
| REC-RULE-060 | REC-101 |
| REC-RULE-061 | REC-106 |
| REC-RULE-062 | REC-107 |
| REC-RULE-063 | REC-105 |
| REC-RULE-064 | REC-108 |
| REC-RULE-065 | REC-109 |
| REC-RULE-066 | REC-110 |
| REC-RULE-067 | REC-111 |
| REC-RULE-068 | REC-102 |
| REC-RULE-069 | REC-113 |
| REC-RULE-070 | REC-114 |
| REC-RULE-071 | REC-115 |
| REC-RULE-072 | REC-129, and the counter definitions in [calculations.md](calculations.md#21-the-four-straightforward-counts) |
| REC-RULE-080 | REC-120 |
| REC-RULE-081 | REC-121 |
| REC-RULE-082 | REC-122 |
| REC-RULE-083 | REC-124 |
| REC-RULE-084 | REC-126 |
| REC-RULE-085 | REC-119 |
| REC-RULE-086 | REC-128 |
| REC-RULE-087 | REC-129 |
| REC-RULE-100 | REC-021 |
| REC-RULE-101 | REC-132 |
| REC-RULE-102 | REC-022 |
| REC-RULE-103 | REC-136 |
| REC-RULE-104 | REC-137 |
| REC-RULE-105 | REC-133 |
| REC-RULE-106 | REC-139 |
| REC-RULE-107 | REC-140 |
| REC-RULE-108 | REC-141 |
| REC-RULE-109 | REC-143 |
| REC-RULE-120 | REC-150 |
| REC-RULE-121 | REC-151 |
| REC-RULE-122 | REC-152 |
| REC-RULE-123 | REC-153 |
| REC-RULE-124 | REC-154 |
| REC-RULE-125 | REC-155 |
| REC-RULE-126 | REC-156 |
| REC-RULE-127 | REC-157 |
| REC-RULE-140 | REC-163 |
| REC-RULE-141 | [configuration.md](configuration.md#6-access-rights-matrix), and REC-164 to REC-172 |
| REC-RULE-142 | REC-023, REC-024, REC-025 |
| REC-RULE-143 | REC-171 |
| REC-RULE-144 | REC-162 |
| REC-RULE-145 | REC-180 |
| REC-RULE-146 | REC-027 |
| REC-RULE-147 | REC-178 |
| REC-RULE-148 | REC-033, REC-034 |
| REC-RULE-149 | REC-175 |
| REC-RULE-150 | REC-176 |
| REC-RULE-151 | REC-173 |
| REC-RULE-152 | REC-174 |
| REC-RULE-170 | REC-181 |
| REC-RULE-171 | REC-182, REC-183 |
| REC-RULE-172 | REC-147, REC-148 |
| REC-RULE-173 | REC-146 |
| REC-RULE-174 | REC-188 |
| REC-RULE-175 | REC-184 |
| REC-RULE-176 | REC-185 |
| REC-RULE-177 | REC-186 |
| REC-RULE-178 | REC-187 |
| REC-RULE-190 | REC-196 |
| REC-RULE-191 | REC-060 |
| REC-RULE-192 | REC-198 |
| REC-RULE-193 | REC-199 |
| REC-RULE-194 | REC-201 |
| REC-RULE-195 | REC-200 |
| REC-RULE-196 | REC-203 |
| REC-RULE-197 | REC-202 |
| REC-RULE-200 | REC-207, REC-208 |
| REC-RULE-201 | REC-209 |
| REC-RULE-202 | REC-213 |
| REC-RULE-203 | REC-214 |
| REC-RULE-204 | REC-216 |
| REC-RULE-205 | REC-179 |

### 20.2 From the chapter-numbered draft

| Former reference | Present location |
|---|---|
| business-rules chapter 3, "The interviewer restriction: a security boundary" | Chapter 3, REC-023 to REC-035 |
| business-rules chapter 5, "Contact synchronisation" | Chapter 5, REC-061 to REC-070 |
| business-rules chapter 9, "Multi-record stage writes" | Chapter 9, REC-116 to REC-118 |
| business-rules chapter 12, "Protection of attribution records" | Chapter 12, REC-146 to REC-149 |
| business-rules, duplicate detection | REC-008, REC-099, REC-108, and [calculations.md](calculations.md#4-duplicate-and-similar-application-detection) |
| business-rules, protections on the public form | Chapter 7, REC-086 to REC-100 |

---

## 21. Reconciliation notes

| Point | Resolution |
|---|---|
| Whether the stage message keeps its technical log copy | One draft said the log copy is not retained, the other that automatic deletion of the sent message is disabled so the message stays visible. The second is correct and is now stated in REC-051: the log copy is kept. |
| Whether a Job Position interviewer is notified | One draft notified every newly named interviewer, on positions and on applications alike. Only the Application-level list notifies; REC-210 records the difference. |
| The refusal warning's line count | One draft described two lines, the other three. The stored text is one sentence, a line break, a second sentence; the names follow after a further line break, giving three displayed lines. REC-105 states it. |
| Whether refusing changes the recruitment target | Both drafts agree it does not, for the same reason: refusal does not touch the stage. REC-115. |
| Whether the message dialog reuses an existing Contact | One draft said it creates one unconditionally, the other said nothing. It creates one unconditionally; recorded as a compatibility finding in REC-069. |
| Whether Application Tags may be created by any internal user | Both drafts agree, and the access matrix confirms it: read, write and create for every internal user, delete for the Officer privilege. REC-167. |
| The name of the curriculum-vitae display privilege | One draft spelled out the abbreviation in the privilege name. The stored name is reproduced in REC-173 in code font, because it is the string an installation carries. |
