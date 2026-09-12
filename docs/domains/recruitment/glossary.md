# Recruitment — Glossary

Every term this folder uses with a meaning of its own, defined in full. Entity names are
written with initial capitals. Reproduced identifiers are in code font and carry their full
name in words.

**Add-to-job dialog.** The transient dialog (`job.add.applicants`) that creates one new
Application per chosen Job Position out of each selected record, clearing the pool list and
writing the landing stage explicitly. See [entities.md](entities.md#134-add-to-job-dialog-jobaddapplicants).

**Add-to-pool dialog.** The transient dialog (`talent.pool.add.applicants`) that adds
Applications to one or more Talent Pools, creating the talent when the person is not yet
pooled.

**Application.** One person presenting themselves for one opening (`hr.applicant`). It
carries the person's contact details, the evaluation, the pipeline position, the salary
discussion, the attribution, the attachments, the outcome and the pooling links. The same
person applying to two openings produces two Applications.

**Application count.** The number of Applications, archived ones included, that belong to the
same person as the record being read, talents excluded. It is the visible face of duplicate
detection, and a record carrying none of the three comparison keys counts zero.

**Application status.** The single derived value shown on an Application: `ongoing`, `hired`,
`refused` or `archived`. It is computed from the refusal reason, the active flag and the hire
date, and is never stored. Its search translation is deliberately not the mirror of its
derivation.

**Application Tag.** A free classification label (`hr.applicant.category`) attachable to
Applications and, separately, to Talent Pools. Tag names are unique and every internal user
may create one.

**Attribution.** The three references — campaign, medium and source — stamped on an
Application to record where it came from. The vocabulary belongs to
[Marketing and Mass Mailing](../marketing-and-mass-mailing/README.md); this domain fills the
three references from tracking addresses, from source addresses and from manual entry.

**Availability.** The date from which the person can start working. Tracked in the thread.

**Canonical pool copy.** The talent that represents a person across pools
(`pool_applicant_id`). A talent points at itself; an Application that was pooled points at the
talent.

**Curriculum vitae.** The document describing the person's career, normally attached to the
Application at intake and shown in the preview panel of the application form. Its text is
indexed, which is what makes the *Resume's content* search work.

**Degree.** A level of education (`hr.recruitment.degree`) with a score between 0 and 1 used
by the education part of the skill match. Four are shipped: Graduate, Bachelor Degree, Master
Degree and Doctoral Degree.

**Delay to Close.** The stored difference between Days to Close and Days to Open, that is the
number of days between the assignment of a recruiter and the hire, expressed as a decimal.

**Duplicate application.** Another Application belonging to the same person, matched on the
normalised electronic mail address, the sanitised telephone number or the professional
network profile address.

**Extended interviewer.** A user named as interviewer on at least one Application of a Job
Position. The position's derived extended interviewer list exists so that such a user may
read the position itself.

**First stage.** The available stage of a position with the lowest sequence. Every intake
path except the position's inbound address excludes folded stages when choosing it.

**Folded stage.** A stage that is collapsed in the pipeline while it holds no record, and that
is skipped when a first stage is chosen. The shipped hired stage is folded, which is why a new
Application never lands in it.

**Hire date.** The moment the Application entered a stage flagged as a hired stage. Its
presence makes the derived status `hired`; leaving such a stage clears it.

**Hired stage.** A stage whose hired flag is set. Entering it stamps the hire date and
decrements the position's remaining target; leaving it does the opposite. It is the only stage
attribute with a business meaning rather than a presentational one.

**Inbound address.** The electronic mail address of a Job Position, or of a Recruitment
Source, at which messages become Applications.

**Interviewer.** A user named on a Job Position or on an Application in order to take part in
the evaluation. Naming a user grants the Interviewer privilege automatically; removing the
last such attachment revokes it.

**Interviewer restriction.** The four-layer security boundary that limits a bare Interviewer
to the Applications they are attached to, without create or delete rights, without the salary
fields, and without the ability to create an Employee. See
[business-rules.md](business-rules.md#3-the-interviewer-restriction-a-security-boundary).

**Job Board.** An external forwarding address (`hr.job.platform`) from which applications
arrive under the board's own address rather than the candidate's. Applications received from
one carry no electronic mail address and no Contact, and their applicant name is extracted
from the message with a pattern.

**Job list.** The public page that lists the published Job Positions of a website, with its
five filters, its counters and its pager.

**Job Position.** The opening being recruited for (`hr.job`). The entity belongs to
[Human Resources Core](../human-resources-core/README.md); this domain adds everything a
position needs while it is being recruited for.

**Landing stage.** The stage an Application is placed in when it is created. Which stage that
is depends on the intake path; see
[state-machines.md](state-machines.md#13-choosing-the-landing-stage).

**Match score.** A percentage expressing how far a candidate's current skills and degree cover
the skills and degree a position requires, with each skill capped at twice the required level.
It is rounded on the candidate side and not on the position side.

**Message dialog.** The transient dialog (`applicant.send.mail`) that posts one message per
selected Application, each addressed to that Application's Contact.

**Normalised address.** The canonical form of an electronic mail address: lower case, with the
display name removed. It is one of the three duplicate comparison keys.

**Pipeline.** The ordered set of stages an Application moves through. Stages are global unless
restricted to named positions, and the pipeline is configuration, not code.

**Pooled application.** An Application that is not itself a talent but whose canonical pool
copy points at one. Writing its address, telephone number, professional network profile
address, degree or skills propagates the change to the talent.

**Professional network profile address.** The address of the person's profile page on the
professional network (`linkedin_profile`). Free text, never validated on the server, and the
third duplicate comparison key.

**Readiness colour.** The four-value marker on an Application inside its current stage:
`normal`, `done`, `waiting` and `blocked`, whose labels each stage may rename. Every stage
change resets it to `normal` unless the same operation names a value.

**Recruiter.** The user responsible for an Application, defaulting to the recruiter of its Job
Position. Setting one stamps the assignment moment.

**Recruitment Source.** A per-position attribution record (`hr.recruitment.source`) that
produces a tracking address and, on demand, a dedicated inbound address, both of which stamp
campaign, medium and source on the Applications they produce. The screens call it a tracker.

**Recruitment Stage.** One column of the pipeline (`hr.recruitment.stage`): a name, an order,
an optional message template, an optional restriction to named positions, a folding flag, a
hired flag, a staleness threshold, four readiness labels and a requirements note.

**Refusal dialog.** The transient dialog (`applicant.get.refuse.reason`) that applies a
refusal reason to a selection, optionally sends one message per selected Application, and
optionally refuses the live duplicates as well.

**Refusal Reason.** A documented reason for turning an Application down
(`hr.applicant.refuse.reason`), carrying the message template proposed when it is chosen.
Refusing stores the reason, stamps the refusal moment and archives the Application.

**Remaining target.** The number of people still to be hired for a Job Position
(`no_of_recruitment`). It is decremented when an Application reaches a hired stage and
incremented when one leaves it, may never go below zero, and may be edited by hand.

**Sanitised telephone number.** The telephone number of an Application in international form
when it can be formatted and in its raw spelling otherwise (`partner_phone_sanitized`). It is
the second duplicate comparison key. It differs from the blacklist-side sanitised number,
which stays empty when the number cannot be formatted.

**Spontaneous application.** An Application with no Job Position.

**Stale application.** An ongoing Application with no hire date that has stayed in its stage
longer than that stage's staleness threshold allows. Stale Applications are marked on the
cards and can be filtered.

**Talent.** An Application whose pool list is non-empty and whose canonical pool copy is
itself: the positionless, canonical record of a person kept for future openings. It cannot be
duplicated, is not shown a pipeline status bar, and must always belong to at least one pool.

**Talent Pool.** A named, managed collection of talents (`hr.talent.pool`) kept independently
of any opening, with a manager, tags, a description and a colour.

**Tracker.** See Recruitment Source.

**Tracking address.** The address published on an external board, carrying the three
attribution parameters, so that Applications produced through it are attributed
automatically. Its formula is in
[interfaces.md](interfaces.md#63-per-source-tracking-address).

**Written interview.** A questionnaire of the recruitment kind attached to a Job Position,
sent to a candidate with an answer deadline, answered by the candidate, and readable or
printable from the Application afterwards. The screens call it an interview form.

---

## Terms that mean something narrower here than elsewhere

| Term | Meaning in this folder |
|---|---|
| Archived | Inactive, whatever the reason. An Application archived **with** a refusal reason reads as *refused*, not as *archived*, even though both are inactive. |
| Contact | The shared address-book record of the person. It is not the Application, and one Contact may carry several Applications. |
| Duplicate | Another Application of the same person, decided by three comparison keys. Nothing is ever merged; duplicates are counted, listed and optionally refused together. |
| Interviewer | Both a privilege and a per-record attachment. Holding the privilege without an attachment reaches nothing. |
| Source | A Recruitment Source of this domain, and, through it, the Tracking Source of the attribution vocabulary. The two are distinct records with the same name. |
| Stage | A configuration record, not an enumerated value. Adding a column to the pipeline is a configuration act. |
| Target | The **remaining** number of people to hire, not the original plan. It falls as people are hired and rises when a hire is undone. |

---

## Reconciliation notes

| Point | Resolution |
|---|---|
| The name of the four-value marker | One draft called it a progress indicator, the other a readiness colour or kanban state. This folder uses *readiness colour* in prose and reproduces `kanban_state` and the four `legend_` identifiers where they matter. Both former names appear in this glossary so that a reader of either draft finds the entry. |
| Talent against talent entry | One draft said *talent entry*, the other *talent*. This folder says **talent** for the record and **Talent Pool** for the collection, and defines *pooled application* for the record that points at a talent. |
| Tracker against Recruitment Source | Both names are in use; the entity is the Recruitment Source and the screens say tracker. Both entries are present and cross-referenced. |
| Rotting against stale | One draft used the word the screens use. This folder says **stale** in prose and reproduces the stored identifiers where they appear. |
