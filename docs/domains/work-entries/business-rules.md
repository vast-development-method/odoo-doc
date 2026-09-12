# Work Entries — Business Rules

Every validation, constraint, invariant, permission check and locking rule of the domain, numbered
with a stable identifier. The identifiers are of the form `WKE-nnn` and never change meaning; a rule
that is withdrawn keeps its number and is marked as such rather than being reused. The full table of
identifiers is [chapter 14](#14-the-table-of-rule-identifiers).

Every message is reproduced character for character. Where a message contains a placeholder, the
placeholder is described in words so that a rebuild can substitute its own values.

---

## 1. How the rules are numbered and where they fire

| Band | Subject |
|---|---|
| `WKE-001` to `WKE-009` | The Work Entry Type catalogue |
| `WKE-010` to `WKE-019` | The fields of a Work Entry |
| `WKE-020` to `WKE-029` | Conflicts |
| `WKE-030` to `WKE-034` | Validation |
| `WKE-035` to `WKE-040` | Deletion, archiving and removal |
| `WKE-041` to `WKE-046` | Interaction with absence requests |
| `WKE-047` to `WKE-053` | Regeneration |
| `WKE-054` to `WKE-059` | Permissions |
| `WKE-060` to `WKE-067` | Everything else |

A rule fires at one of five moments: on creating a record, on writing to it, on deleting it, when a
named operation is invoked, or when a record is read through a record rule. Each rule below states
which.

---

## 2. The Work Entry Type catalogue

### `WKE-001` — A work entry kind must carry a name

**Fires on** create and write. The name is required and translatable. A create without one is refused
by the platform's own required-field handling; the message is the platform's generic one for a missing
required field, naming the field by its label.

### `WKE-002` — A work entry kind must carry a payroll code

**Fires on** create and write. The payroll code is required. It is the identifier every downstream
payroll rule and every statutory export keys off, and its help text warns the user: "Careful, the Code
is used in many references, changing it could lead to unwanted changes."

### `WKE-003` — The display code is at most three characters

**Fires on** create and write. The field is declared with a maximum length of three characters and the
platform truncates or refuses according to its storage handling. Its help text reads: "This code can
be changed, it is only for a display purpose (3 letters max)". Nothing keys off the display code; it
appears only as a badge on a calendar cell and on a kanban card.

### `WKE-004` — The country of the shipped ordinary-attendance kind may not be changed

**Fires on** create and write, whenever the country of one or more kinds is written. If the set being
written includes the shipped ordinary-attendance kind, payroll code `WORK100`, the whole write is
refused with:

> "You can't change the country of this specific work entry type."

The reason is structural: that kind is the fallback of every working schedule line and of every
generated attendance interval, so it must remain usable by every company in every country.

### `WKE-005` — The country of a kind in use may not be changed

**Fires on** create and write, whenever the country of one or more kinds is written, and only after
`WKE-004` has passed. Unless the write is part of the installation of shipped data, the platform looks
for a single work entry referencing **any** of the kinds being written. If one exists, the whole write
is refused with:

> "You can't change the Country of this work entry type cause it's currently used by the system. You need to delete related working entries first."

The existence query runs with elevated rights, so a work entry the acting user cannot see still blocks
the change.

### `WKE-006` — A payroll code is unique within a country scope

**Fires on** create and write, whenever the payroll code or the country of one or more kinds is
written. The platform searches for other kinds — excluding those being written — whose payroll code is
one of the codes being written and whose country is either one of the countries being written or
empty. For each kind being written, that set is narrowed to those carrying the identical payroll code. The
write is refused whenever the narrowed set is not empty, with the message:

> "The same code cannot be associated to multiple work entry types (the offending payroll codes, comma separated)"

The placeholder is the comma-separated list of the payroll codes of the offending kinds, with
duplicates removed. Two consequences are easy to get wrong and are therefore stated:

- A universal kind, one with no country, and a country-specific kind may **not** share a payroll code,
  because the search deliberately includes kinds with no country.
- Two kinds belonging to **different** countries **may** share a payroll code, because the search only
  considers the countries present in the write.

This is a validation, not a database index: data loaded outside the ordinary write path is not
re-checked.

### `WKE-007` — The country of a kind is restricted to the countries of the acting companies

**Fires on** the selection list of the country field. Only countries that are the country of one of
the acting companies are offered. It is a restriction on choice, not a refusal: a country written
directly is not rejected by this rule.

### `WKE-008` — The absence flag and the working-time flag are exact complements

**Fires on** read and write of either. The working-time indicator is not stored: it is computed as the
negation of the absence flag, and writing it writes the negation back onto the absence flag. A rebuild
must not store both.

### `WKE-009` — Archiving a kind never cascades

**Fires on** archiving. An archived kind disappears from every selection list and from the default
search but every existing work entry keeps pointing at it, keeps its payroll code mirror and keeps its
pay rate. Nothing is deleted, nothing is re-pointed and no entry changes state.

---

## 3. The duration constraint

### `WKE-010` — A duration is strictly positive and at most twenty-four hours

**Fires on** create and on every write that changes the duration. The comparison is made to a
precision of three decimal places. The refusal is:

> "Duration must be positive and cannot exceed 24 hours."

A duration of exactly twenty-four hours is accepted. A duration of zero is refused; this is why the
generation algorithm skips a degenerate interval whose start equals its stop and why the
post-processing pass drops rows of zero duration before creating anything.

The relationship between this constraint and the day-total conflict test — they use different
precisions and disagree on a narrow band — is worked out in
[calculations.md, section 14.4](calculations.md#144-the-duration-constraint-for-comparison).

---

## 4. The required fields and the defaults of a Work Entry

### `WKE-011` — The employee is required

**Fires on** create and write. The link refuses deletion of the employee while entries exist.

### `WKE-012` — The version is required and is resolved from the employee and the date

**Fires on** create, and on the form whenever the employee or the date changes. The resolution is:

1. If the caller already supplied a version, keep it.
2. Otherwise, if both a date and an employee are present, take the version of that employee in force
   on that date, as defined by
   [Human Resources Core](../human-resources-core/calculations.md#33-the-version-in-force-on-an-arbitrary-date):
   the latest version whose version date is not later than the requested date, falling back to the
   employee's earliest version when the requested date precedes them all, and preferring active
   versions over archived ones.
3. Otherwise leave it unset, which makes the create fail because the field is required.

On an open form a validation error raised while resolving is swallowed and the field simply stays as
it was. On create no such swallowing happens.

### `WKE-013` — The date is required

**Fires on** create and write. The date is a calendar date in the **schedule's** zone, not in the
universal reckoning. A work entry carries no clock times at all.

### `WKE-014` — The company is required, read-only, and taken from the employee

**Fires on** create. The default is the acting company; on create, whenever the caller supplied no
company, the employee's company overrides it. A work entry therefore always carries the company of its
employee at the moment of creation, whatever company the acting user was in. The field is read-only
thereafter.

### `WKE-015` — The employee must belong to the entry's company or to no company

**Fires on** the selection list of the employee field. It is a restriction on choice, not a refusal.

### `WKE-016` — The kind offered depends on how many countries the acting companies span

**Fires on** the selection list of the kind field.

| Number of distinct countries among the acting companies | Kinds offered |
|---|---|
| more than one | only universal kinds, those with no country |
| exactly one | universal kinds and kinds of that country |
| none | universal kinds only, since the list of countries is empty |

The asymmetry is deliberate: it prevents an entry created in a multi-country context from silently
picking up a country-specific payroll code.

### `WKE-017` — The pay rate is copied from the kind once, at creation

**Fires on** create. When the caller supplied no pay rate and did supply a kind, the kind's rate is
copied onto the entry. It is never recomputed: changing the kind's rate does not move existing
entries, and changing an entry's kind does not change its rate. A rate a payslip has already used must
not move.

### `WKE-018` — The state and the archived flag are one fact

**Fires on** every write. The rewriting is specified in
[state-machines.md, section 1.2](state-machines.md#12-the-coupling-with-the-archived-flag). In short:
writing the state `draft` also unarchives; writing `cancelled` also archives; writing the archived
flag rewrites the state; and the flag rule runs last.

### `WKE-019` — The stored conflict indicator mirrors the state

**Fires on** every write of the state. The stored true/false indicator is true exactly when the state
is `conflict`. It exists only so that a list can sort conflicting entries first without joining on a
selection value, and nothing reads it for a decision.

---

## 5. The four conflict conditions

The four conditions are evaluated as one operation, in the order below, each writing state before the
next reads it. The operation reports whether anything at all was marked; the validation of
[chapter 7](#7-validation) uses that report.

### `WKE-020` — An entry with no kind is in conflict

**Fires on** every conflict check. Every entry of the selection with no work entry kind is moved to
`conflict`. The form explains it: "This work entry cannot be validated. The work entry type is
undefined."

### `WKE-021` — A day whose hours leave the range from zero to twenty-four puts its whole day in conflict

**Fires on** every conflict check. Over the date range spanned by the selection, and for the employees
of the selection, the platform totals the durations of every entry whose archived flag is true, grouped
by employee and date, and selects the groups whose total is at most zero hours or more than
twenty-four hours. **Every** entry of those groups whose archived flag is true is moved to `conflict`,
including entries that were not part of the selection.

The arithmetic and its worked examples are in
[calculations.md, chapter 14](calculations.md#14-the-conflict-arithmetic). The form explains the
condition: "The amount of work on the day should not exceed 24 hours."

Three details of reach matter:

- Archived and cancelled entries are excluded from the total, so cancelling an entry can lift a day
  out of conflict.
- Validated entries **are** included in the total.
- The range is the range of the selection, so an entry moved to a date outside that range is checked
  against the range recomputed after the change, while the reset pass that precedes the change used
  the range before it.

### `WKE-022` — An absence entry entirely outside the working schedule is in conflict

**Fires on** every conflict check. Among the entries of the selection, those whose kind carries the
absence flag and whose state is neither `validated` nor `cancelled` are grouped by the working
schedule of their version. For each group:

1. If the group's schedule is empty, or the schedule is a flexible-hours one, the group is skipped: a
   version with no fixed hours cannot have an absence "outside" them.
2. Otherwise the schedule is expanded over the range from the earliest date of the group at local
   midnight to the latest date of the group at the last representable microsecond of the day.
3. Each entry of the group is turned into an interval covering its whole date, from midnight to the
   last microsecond, on the universal time scale.
4. The entries whose interval intersects the expanded schedule at all are kept; the rest are moved to
   `conflict`.

The test is a whole-day one: an absence entry on a day the employee works at all survives, whatever
hours the absence claims.

### `WKE-023` — An entry on a day that already holds a validated entry is in conflict

**Fires on** every conflict check. The platform searches for validated entries whose date lies between
the earliest and the latest date of the selection and whose company is the **acting** company, indexes
them by employee and date, and moves to `conflict` every entry of the selection whose employee and
date appear in that index.

Note the company clause: the search is restricted to the acting company, not to the companies of the
entries being checked. In a multi-company session where the acting company is not the employee's, a
validated entry of the employee's company does not block. This is recorded as a **compatibility
finding**; a corrected behaviour would restrict the search to the companies of the entries being
checked.

Note also that an entry is put in conflict by its **own** validated twin: validating a day and then
adding a second entry to it makes the new entry conflict, which is the intended protection of a locked
payroll figure.

### `WKE-024` — The conflict check runs on create, and on write and delete only when one of five fields is involved

**Fires on** create, write and delete.

| Operation | When the check runs |
|---|---|
| create | always, over everything created |
| write | only when the values written include at least one of: the date, the duration, the employee, the kind, the archived flag |
| write of the state `cancelled` | additionally skipped when **every** entry of the selection is currently out of the conflict state; if any is in conflict the check runs |
| delete | always |

A write of the description alone, or of the pay rate alone, or of the state to `validated`, therefore
performs no check at all.

### `WKE-025` — The reset pass precedes every check

**Fires on** write and delete. Before the change is applied, the platform searches, with elevated
rights and with the check suppressed to avoid recursion, every entry whose date lies inside the
re-check window, whose state is neither `validated` nor `cancelled`, and whose employee is one of the
affected employees; and it writes the state `draft` onto those of them that are in conflict.

The affected employees are the employees of the records being changed, plus the employee being written
when the write moves an entry to another employee.

In the absence companion the reset pass has a second effect: every entry it touches that has a kind
**and** whose kind does not carry the absence flag has its absence link cleared. An attendance entry
must never keep a pointer to an absence.

After the change is applied, the same set of entries — refetched, so that deleted ones drop out — is
put through the four conditions again.

### `WKE-026` — The re-check window of an absence request is widened by one day at each end

**Fires on** create and write of a Time Off Request, in the absence companion.

```formula
window start = ( the earliest requested start date among the records and the values, minus one day )
               at 00:00:00
window stop  = ( the latest  requested end   date among the records and the values, plus  one day )
               at 23:59:59.999999
```

The widening exists because the instants of a request are derived from its requested dates in the
employee's zone and can therefore fall on the neighbouring calendar day. Without it, an absence
starting at 23:00 local on the last day of the window would leave the following day unchecked.

When neither the records nor the values carry a requested start date, the earliest is taken as the
largest representable date, and when neither carries a requested end date the latest is taken as the
smallest representable date; the resulting window is then empty and the check is skipped.

The check is skipped entirely unless the write touches the employee, the state, the requested start
date or the requested end date.

### `WKE-027` — A caller may suppress the conflict check

**Fires on** create, write and delete. A marker on the operating context suppresses the check for the
whole call. It is set by the reset pass itself, to avoid unbounded recursion, and may be set by a
bulk loader. A rebuild must offer the same escape, because without it a large import performs one
day-total query per record.

### `WKE-028` — A French part-time employee is exempt from the outside-schedule test

**Fires on** the conflict check, in the French part-time companion. An entry is exempt when its
company's country is France **and** the employee's working schedule differs from the company's
working schedule. If the selection consists only of exempt entries the test does not run at all; if
it is mixed, the test runs over the non-exempt entries only and the exempt ones are neither tested nor
marked.

The reason is the gap filling of [calculations.md, chapter 13](calculations.md#13-the-gap-filling-rule-for-french-part-time-absences):
it deliberately produces absence rows on days the employee does not work, which the outside-schedule
test would otherwise flag as conflicts on every one of them.

### `WKE-029` — Versions with no schedule and flexible-hours versions are exempt from the outside-schedule test

**Fires on** the conflict check. Stated separately from `WKE-022` because it is easy to overlook: the
exemption is by schedule, not by entry, so a single employee whose version has no schedule exempts
every absence entry of that version.

---

## 6. The conflict re-check window

The window over which conflicts are recomputed is not always the range of the records being changed.
The three cases:

| Change | Window start | Window stop |
|---|---|---|
| Create, write or delete of work entries | the earliest date among the records being changed | the latest date among them |
| Create or write of a Time Off Request | one day before the earliest requested start date | one day after the latest requested end date |
| Explicit range supplied by a caller | that range | that range |

In the first case the window is computed **before** the change is applied. An entry moved from inside
the window to outside it is therefore reset by the pre-change pass and re-examined by the post-change
pass, whose own range is recomputed from the new dates; but entries that were already on the
destination day are not reset, and are only re-marked if they still qualify. This asymmetry is
observed behaviour.

---

## 7. Validation

### `WKE-030` — Validation refuses as a whole when any condition holds

**Fires on** the validation operation. The operation:

1. Narrows the selection to entries whose state is not `validated`.
2. Runs the four conditions of [chapter 5](#5-the-four-conflict-conditions) over that narrowed
   selection.
3. If any condition marked anything, writes nothing and reports failure. The entries it marked stay in
   the conflict state; the operation does not undo them.
4. If nothing was marked, writes the state `validated` onto the whole narrowed selection in one
   operation and reports success.

There is no message. The failure is expressed by the state of the entries and by the return value,
which a payroll capability or an automation reads. A person sees the conflict badge and the form's
explanatory text.

### `WKE-031` — Entries already validated are excluded from a validation

**Fires on** the validation operation. They are removed from the selection before the conditions run,
so a day that already holds a validated entry is not made to conflict with itself by `WKE-023` — only
entries that are not yet validated are.

### `WKE-032` — A validated entry is not editable

**Fires on** the interface. The form marks the description, the kind, the employee, the date and the
duration read-only when the state is `validated`, and shows the notice: "Note: Validated work entries
cannot be modified." The calendar refuses to drag, resize, open or delete a validated event, shows a
padlock on it, and shows in its popover: "You cannot edit a validated work entry".

This is an interface rule and not a stored constraint: a write performed by an integration is not
refused. A rebuild that needs the guarantee must add a constraint of its own; the absence of one is
recorded as a **compatibility finding**.

### `WKE-033` — A validated entry survives every regeneration

**Fires on** generation. Every domain the generator uses to nullify or delete entries carries the
clause "the state is not `validated`". Forced regeneration, the retirement of entries beyond a version
end and the cancellation on version removal all honour it.

### `WKE-034` — A validated entry is never swallowed by an absence

**Fires on** the validation of an absence request. Both the archiving of entries the absence covers and
the clearing of the absence link on entries that overlap it are restricted to entries whose state is
not `validated`.

---

## 8. Deletion and archiving rules

### `WKE-035` — A validated entry may not be deleted

**Fires on** delete. If any entry of the selection is in the state `validated`, the whole deletion is
refused with:

> "This work entry is validated. You can't delete it."

### `WKE-036` — Archiving and cancelling are the same act

**Fires on** every write. See `WKE-018`. Every place in the platform that wants an entry to stop
counting writes the archived flag false; the state follows.

### `WKE-037` — Cancelling an entry refuses its absence request

**Fires on** a write of the state `cancelled`, in the absence companion, **before** the state is
written. Every absence request linked to an entry of the selection whose state is not already the
refused one is refused. Refusing a request in turn archives the entries it produced and regenerates
ordinary entries in their place, so cancelling one absence entry can rewrite a whole absence period.

### `WKE-038` — Resetting an attendance entry out of conflict clears its absence link

**Fires on** the reset pass. Specified under `WKE-025`.

### `WKE-039` — Entries outside the contract period are deleted, not archived

**Fires on** a write of the contract start date, the contract end date or the version date of a
version. The pass is specified in
[calculations.md, section 10.3](calculations.md#103-removal-outside-the-contract-period). It deletes,
so a validated entry outside the new period makes the whole write fail through `WKE-035`.

### `WKE-040` — Removing a version deletes its non-validated entries inside its window

**Fires on** deleting an Employee Version. Specified in
[calculations.md, section 10.4](calculations.md#104-cancellation-when-a-version-is-removed). Because
the entry-to-version link refuses deletion of the version while entries remain, a version still
holding a validated entry cannot be removed.

---

## 9. Interaction with absence requests

All of these belong to the absence companion.

### `WKE-041` — The exclusion an absence creates carries the work entry kind of the absence kind

**Fires on** preparing the values of the Working Time Exclusion that a validated request creates. The
work entry kind of the request's absence kind is copied onto the exclusion. This is what lets the
precedence ladder of [calculations.md, section 7.3](calculations.md#73-the-precedence-ladder-for-an-absence-interval)
label the produced row.

### `WKE-042` — Validating an absence generates rows only where generation has already reached

**Fires on** validating a Time Off Request. For each request, and for each version of the employee
overlapping the request's dates by at least one contracted day, rows are produced **only when** the
request's stop is not earlier than that version's generated-from marker **and** the request's start is
not later than its generated-to marker. An absence entirely in the future, beyond the generated
window, produces nothing at validation time; its rows appear when generation next reaches that period.

The values are computed over the whole of the request's days — from the request's start date at
midnight to the request's end date at the last representable microsecond — passed through the
post-processing pass, and created.

The computation is performed for the **whole set** of overlapping versions once for each version that
passes the marker test. When two versions of the same employee both overlap the request and both have
reached it, the same value sets are therefore produced twice, and the merge of the post-processing
pass adds their durations together, doubling the hours of the produced absence rows. This is recorded
as a **compatibility finding**; a corrected behaviour would compute the values once per qualifying
version, for that version alone.

### `WKE-043` — Entries an absence completely covers are archived; entries that overlap it lose their link

**Fires on** validating a Time Off Request, immediately after `WKE-042`. For each employee:

1. Read every entry of that employee dated between the earliest request start and the latest request
   stop of the batch.
2. Split them into the rows just created for the absence and the rows that existed before.
3. Turn both groups into whole-day intervals and subtract: the pre-existing rows whose day interval
   lies at least partly outside the new absence rows' day intervals are the **overlapping** ones; the
   rest are the **included** ones.
4. Every overlapping entry that is not validated has its absence link cleared.
5. Every included entry that is not validated is archived, and therefore cancelled.

Because the intervals compared are whole days, an entry on a day that the absence touches at all is
treated as included unless some other entry of that day extends the day interval beyond the absence's.
In practice this means: a full day of absence archives the day's attendance row; a partial absence
leaves the day's attendance row alone and merely clears any stale absence link on it.

### `WKE-044` — Refusing, reverting or cancelling an absence archives its rows and regenerates the days

**Fires on** three operations of a Time Off Request: refusing it, moving it back from validated to an
earlier approval step, and the requester cancelling it. In all three:

1. Find every work entry whose absence link points at one of the requests, with elevated rights.
2. Archive them all, which also cancels them. Note that this is **not** restricted to non-validated
   entries; see `WKE-045` for why that is nevertheless safe for a user-initiated cancellation, and
   note it as a **compatibility finding** for the refusal path, where a validated row can be
   cancelled. A corrected behaviour would refuse the refusal while a produced row is validated, as it
   already refuses the cancellation.
3. For each archived entry, recompute the values of its version over that entry's whole date, from
   midnight to the last representable microsecond, pass them through the post-processing pass and
   create them. The ordinary attendance of those days is thereby restored.

### `WKE-045` — An absence with a validated entry cannot be cancelled by its requester

**Fires on** computing whether a request may be cancelled. The platform searches for validated work
entries linked to the candidate requests; every request that appears becomes non-cancellable. The
message shown, and the rest of the cancellation flow, belong to
[Time Off](../time-off/business-rules.md).

### `WKE-046` — Three absence kinds are not displaced by a public holiday

**Fires on** the recomputation, in the absence domain, of the set of absences falling on a public
holiday. Requests whose absence kind's work entry kind carries the payroll code `LEAVE110` (sick time
off), `LEAVE210` (maternity time off) or `LEAVE280` (long-term sick) are removed from that set. The
business reason is that these absences continue to run across a public holiday rather than being
interrupted by it, so the platform must not shorten them.

The three codes are reproduced exactly because they are the contract: a rebuild that renames them
changes which absences survive a public holiday.

---

## 10. Regeneration guards

The three guards below fire only when the operation is called without the skip-validation marker and
without a list of day slots. Regeneration triggered by a version change, and regeneration triggered
from the calendar's multiple selection, both bypass all three.

### `WKE-047` — The search criteria must be complete

**Fires on** the regeneration operation. The criteria are complete when a from-date, a to-date, at
least one employee, an earliest available date and a latest available date are all present. Otherwise:

> "In order to regenerate the work entries, you need to provide the wizard with an employee_id, a date_from and a date_to."

The three names inside the message are reproduced exactly, because the message is asserted by tests
and quoted in support procedures; they are the stored names of the three fields.

### `WKE-048` — The requested range must lie inside the generated range

**Fires on** the regeneration operation, after `WKE-047`. If the from-date is earlier than the earliest
available date, or the to-date later than the latest available date:

> "The from date must be >= '(the earliest available date)' and the to date must be <= '(the latest available date)', which correspond to the generated work entries time interval."

The two placeholders are the earliest and the latest available date, each rendered in the date format
of the acting user's language and each enclosed in single quotation marks by the message itself.

### `WKE-049` — At least one selected employee must be regenerable

**Fires on** the regeneration operation, after `WKE-048`. The operation is valid only when at least one
selected employee holds no validated entry inside the requested range. Otherwise:

> "No work entry can be regenerated in this range of dates and these employees."

### `WKE-050` — Employees holding a validated entry in the range are skipped, not refused

**Fires on** the regeneration operation, in every mode. The selected employees are narrowed to those
that hold no validated entry between the from-date and the to-date. The form shows the excluded ones
in red with the notice: "Employees in red will be skipped because they have at least one validated
work entry." If the narrowing empties the set, the operation returns having done nothing at all — no
message, no error.

That silent return is what makes a schedule change safe: changing the working schedule of an employee
who has validated entries in the generated window regenerates nothing rather than destroying the
payroll figures.

### `WKE-051` — Regeneration triggered by a version change skips the three guards

**Fires on** a write to an Employee Version that changes the working schedule or the generation
source. The wizard is built for that version's employee over the overlap of the version's effective
window and its generated window, and run with the skip-validation marker and with archived records
included. `WKE-050` still applies, so the run still refuses to touch an employee with validated
entries.

### `WKE-052` — Regeneration from a list of day slots skips the three guards

**Fires on** the regeneration operation called with a list of pairs of an employee and a date. The
pairs are collapsed into maximal runs of consecutive days as specified in
[calculations.md, section 15.5](calculations.md#155-collapsing-selected-days-into-runs), and one
forced generation is issued per run. None of `WKE-047`, `WKE-048` or `WKE-049` is evaluated, and the
clamping of [calculations.md, section 15.4](calculations.md#154-the-clamping-applied-at-run-time) is
not applied either: the days are taken as given.

### `WKE-053` — What a forced regeneration writes onto the entries it supersedes

**Fires on** a forced generation. The superseded entries are not deleted. A single list names the
fields a nullifying write sets to false; in the specified system that list contains exactly the
archived flag. Writing the archived flag false also writes the state `cancelled`, so the superseded
entries end up cancelled and archived and remain visible to anybody who looks for archived records.

The list is an extension point. A country package may add its own fields to it, and a rebuild should
keep the indirection rather than hard-coding the archived flag.

---

## 11. Permission checks

### `WKE-054` — The access matrix

| Entity | Group | Read | Write | Create | Delete |
|---|---|---|---|---|---|
| Work Entry | Human Resources Officer | yes | yes | yes | **no** |
| Work Entry | Settings Administrator | yes | yes | yes | yes |
| Work Entry Type | Human Resources Officer | yes | no | no | no |
| Work Entry Type | Human Resources Administrator | yes | yes | yes | yes |
| Work Entry Employee Filter | Human Resources Officer | yes | yes | yes | yes |
| Work Entry Regeneration Wizard | Human Resources Administrator | yes | yes | yes | yes |

The single most consequential line is the first: **a human resources officer cannot delete a work
entry.** Officers archive instead, which cancels. Only a member of the Settings Administrator group can delete one,
and even then `WKE-035` protects validated entries. A rebuild that grants deletion to officers changes
the domain's safety properties.

### `WKE-055` — Work entries are restricted to the acting companies

**Fires on** every operation, for every group. A record rule named "HR Work Entry Contract: Multi
Company" limits visibility to entries whose company is one of the acting companies.

### `WKE-056` — Work entry kinds are restricted by country

**Fires on** every operation, for every group. A record rule named "HR Work Entry: Multi Company"
limits visibility to kinds whose country is one of the countries of the acting companies or is empty.

Note that the kinds a user can **see** are governed by this rule and the kinds a user can **choose**
on an entry by `WKE-016`; the two are different and deliberately so.

### `WKE-057` — A calendar filter row may be changed only by its own user

**Fires on** create, write and delete, for every internal user. A record rule named "Work
entries/Employee calendar filter: only self" restricts those three operations to rows whose user is
the acting user. The **read** operation is deliberately left unrestricted, because the calendar's
grouping query reads the whole table.

### `WKE-058` — Generation runs with elevated rights and under the group's company

**Fires on** the version-level entry point. Each group of versions is generated with elevated rights
and with the group's company as the acting company. The elevation is necessary because generation
reads Working Time Exclusions and absence requests that the acting user may not be allowed to see, and
writes entries for employees the acting user may not be allowed to write.

The nullifying search, the cancellation on version removal, the removal outside the contract period
and the regeneration triggered by a version change all likewise run with elevated rights.

### `WKE-059` — Field-level restrictions on the version and the employee

| Field | Entity | Visible to |
|---|---|---|
| Generated From, Generated To, Last Generation Date | Employee Version | Human Resources Officer and above |
| Generation source, invalid-source indicator | Employee Version | Human Resources Administrator only |
| Generation source, invalid-source indicator | Employee | Human Resources Administrator only |
| Has work entries | Employee | the Settings Administrator group and Human Resources Officer |
| Work entry kind on a working schedule line | Working Schedule Line | Human Resources Officer and above |
| Work entry kind on a working time exclusion | Working Time Exclusion | Human Resources Officer and above |

---

## 12. The generation source extension point

### `WKE-060` — The generation source is a one-value selection with a documented extension contract

**Fires on** create and write of an Employee Version. The field is required and defaults to `calendar`
"Working Schedule", which is the only value the specified system offers. Its help text names two
further sources that a package may add — generating from recorded attendances, and generating from a
published plan — and a rebuild should treat the field as the extension point it is.

The contract a new source must satisfy is:

1. **Declare whether it is static.** A source is static when the same day book would be produced every
   month from the same configuration. `calendar` is static. A source that is not static loses the
   cheap path: its attendance intervals are split per payload record so that overlapping slots produce
   two rows and a visible conflict, its absences are bounded by a second expansion of the schedule
   rather than by the produced attendances, and the scheduled job processes it after the static ones.
2. **Produce attendance intervals.** Only versions whose source is `calendar` contribute attendance
   intervals through the schedule expansion; another source must contribute its own.
3. **Leave the absence path alone.** The exclusion reading and the absence partition are not
   conditioned on the source, so absences continue to work unchanged.
4. **Honour the markers.** A source that produces a variable day book should let the markers be pushed
   by the produced values, as specified in
   [calculations.md, section 9.3](calculations.md#93-the-push-from-produced-values), rather than
   setting them ahead of production.

### `WKE-061` — A version whose source is the working schedule but which names no schedule is flagged, not blocked

**Fires on** reading the invalid-source indicator of a version or an employee. The indicator is true
exactly when the generation source is `calendar` and the version names no working schedule. It drives
a warning in the interface, whose text is: "Invalid option: For fully flexible calendars, the work
entry source cannot be 'Working Hours'." It does not block generation; such a version simply produces
one whole-window interval per the fully flexible rule of
[calculations.md, section 4.3](calculations.md#43-a-version-with-no-working-schedule).

---

## 13. Everything else

### `WKE-062` — A user may pin an employee into the calendar only once

**Fires on** create and write of a calendar filter row. A database uniqueness constraint spans the
pair of the user and the employee. Its violation message is:

> "You cannot have the same employee twice."

The constraint does not exclude archived rows, so an archived pin still occupies the slot; re-pinning
must reactivate the existing row rather than insert a new one.

### `WKE-063` — Generation refuses to run without a time zone

**Fires on** the post-processing pass. If the version's working schedule, the employee's working
schedule and the company's working schedule all name no time zone, the run fails with:

> "Missing timezone for work entries generation."

### `WKE-064` — A value set must carry either two instants or a date and a duration

**Fires on** the post-processing pass. A value set with no start or no stop must already carry both a
date and a duration; otherwise the run fails with:

> "Missing date or duration on work entry"

### `WKE-065` — The two guards on splitting an entry

**Fires on** the split operation, which acts on exactly one entry.

| Order | Condition | Message |
|---|---|---|
| 1 | the entry's duration is strictly below one hour | "You can't split a work entry with less than 1 hour." |
| 2 | the requested split duration is not strictly smaller than the entry's duration | "Split work entry duration has to be less than the existing work entry duration." |

When both pass, the entry's duration is reduced by the split duration, a copy of the entry is created —
which starts in `draft`, because the state is not copied — and the split values are written onto the
copy. The operation returns the identifier of the new entry.

### `WKE-066` — The version-level entry point takes dates, never instants

**Fires on** the version-level generation entry point. Passing an instant is an internal contract
breach and is refused before anything runs. It matters because the conversion from a period to a
window is the entry point's own job, and a caller that has already converted would have the conversion
applied twice.

### `WKE-067` — Concurrency and locking

The domain declares no explicit lock. Four properties keep concurrent generation safe, and a rebuild
must reproduce all four:

1. **Everything is one transaction.** A generation run writes its markers and its rows in the same
   transaction; a failure discards both, leaving the markers consistent with the rows.
2. **The markers are the guard against duplication.** Two concurrent runs over the same period for the
   same version both read the markers; the second to commit finds them already advanced only if it
   re-read them, which the platform's own row-level write conflict handling enforces by making one of
   the two transactions retry. A rebuild that lets both commit will produce duplicate rows.
3. **The day-total query flushes first.** The conflict check writes the pending changes to storage
   before running the day-total query, so that the query sees the values of the current transaction.
4. **A dead cursor aborts the check.** If the underlying connection fails during a checked operation,
   the check is abandoned rather than retried, so that the original failure is not masked by a second
   one.

---

## 14. The table of rule identifiers

| Identifier | Rule | Chapter |
|---|---|---|
| `WKE-001` | A work entry kind must carry a name | [2](#2-the-work-entry-type-catalogue) |
| `WKE-002` | A work entry kind must carry a payroll code | [2](#2-the-work-entry-type-catalogue) |
| `WKE-003` | The display code is at most three characters | [2](#2-the-work-entry-type-catalogue) |
| `WKE-004` | The country of the shipped ordinary-attendance kind may not be changed | [2](#2-the-work-entry-type-catalogue) |
| `WKE-005` | The country of a kind in use may not be changed | [2](#2-the-work-entry-type-catalogue) |
| `WKE-006` | A payroll code is unique within a country scope | [2](#2-the-work-entry-type-catalogue) |
| `WKE-007` | The country of a kind is restricted to the countries of the acting companies | [2](#2-the-work-entry-type-catalogue) |
| `WKE-008` | The absence flag and the working-time flag are exact complements | [2](#2-the-work-entry-type-catalogue) |
| `WKE-009` | Archiving a kind never cascades | [2](#2-the-work-entry-type-catalogue) |
| `WKE-010` | A duration is strictly positive and at most twenty-four hours | [3](#3-the-duration-constraint) |
| `WKE-011` | The employee is required | [4](#4-the-required-fields-and-the-defaults-of-a-work-entry) |
| `WKE-012` | The version is required and resolved from the employee and the date | [4](#4-the-required-fields-and-the-defaults-of-a-work-entry) |
| `WKE-013` | The date is required | [4](#4-the-required-fields-and-the-defaults-of-a-work-entry) |
| `WKE-014` | The company is required, read-only, and taken from the employee | [4](#4-the-required-fields-and-the-defaults-of-a-work-entry) |
| `WKE-015` | The employee must belong to the entry's company or to no company | [4](#4-the-required-fields-and-the-defaults-of-a-work-entry) |
| `WKE-016` | The kinds offered depend on how many countries the acting companies span | [4](#4-the-required-fields-and-the-defaults-of-a-work-entry) |
| `WKE-017` | The pay rate is copied from the kind once, at creation | [4](#4-the-required-fields-and-the-defaults-of-a-work-entry) |
| `WKE-018` | The state and the archived flag are one fact | [4](#4-the-required-fields-and-the-defaults-of-a-work-entry) |
| `WKE-019` | The stored conflict indicator mirrors the state | [4](#4-the-required-fields-and-the-defaults-of-a-work-entry) |
| `WKE-020` | An entry with no kind is in conflict | [5](#5-the-four-conflict-conditions) |
| `WKE-021` | A day outside the zero-to-twenty-four-hour range puts its whole day in conflict | [5](#5-the-four-conflict-conditions) |
| `WKE-022` | An absence entry entirely outside the working schedule is in conflict | [5](#5-the-four-conflict-conditions) |
| `WKE-023` | An entry on a day that already holds a validated entry is in conflict | [5](#5-the-four-conflict-conditions) |
| `WKE-024` | The conflict check runs on create, and on write and delete only for five fields | [5](#5-the-four-conflict-conditions) |
| `WKE-025` | The reset pass precedes every check | [5](#5-the-four-conflict-conditions) |
| `WKE-026` | The re-check window of an absence request is widened by one day at each end | [5](#5-the-four-conflict-conditions) |
| `WKE-027` | A caller may suppress the conflict check | [5](#5-the-four-conflict-conditions) |
| `WKE-028` | A French part-time employee is exempt from the outside-schedule test | [5](#5-the-four-conflict-conditions) |
| `WKE-029` | Versions with no schedule and flexible-hours versions are exempt from that test | [5](#5-the-four-conflict-conditions) |
| `WKE-030` | Validation refuses as a whole when any condition holds | [7](#7-validation) |
| `WKE-031` | Entries already validated are excluded from a validation | [7](#7-validation) |
| `WKE-032` | A validated entry is not editable | [7](#7-validation) |
| `WKE-033` | A validated entry survives every regeneration | [7](#7-validation) |
| `WKE-034` | A validated entry is never swallowed by an absence | [7](#7-validation) |
| `WKE-035` | A validated entry may not be deleted | [8](#8-deletion-and-archiving-rules) |
| `WKE-036` | Archiving and cancelling are the same act | [8](#8-deletion-and-archiving-rules) |
| `WKE-037` | Cancelling an entry refuses its absence request | [8](#8-deletion-and-archiving-rules) |
| `WKE-038` | Resetting an attendance entry out of conflict clears its absence link | [8](#8-deletion-and-archiving-rules) |
| `WKE-039` | Entries outside the contract period are deleted, not archived | [8](#8-deletion-and-archiving-rules) |
| `WKE-040` | Removing a version deletes its non-validated entries inside its window | [8](#8-deletion-and-archiving-rules) |
| `WKE-041` | The exclusion an absence creates carries the work entry kind of the absence kind | [9](#9-interaction-with-absence-requests) |
| `WKE-042` | Validating an absence generates rows only where generation has already reached | [9](#9-interaction-with-absence-requests) |
| `WKE-043` | Entries an absence covers are archived; entries that overlap it lose their link | [9](#9-interaction-with-absence-requests) |
| `WKE-044` | Refusing, reverting or cancelling an absence archives its rows and regenerates | [9](#9-interaction-with-absence-requests) |
| `WKE-045` | An absence with a validated entry cannot be cancelled by its requester | [9](#9-interaction-with-absence-requests) |
| `WKE-046` | Three absence kinds are not displaced by a public holiday | [9](#9-interaction-with-absence-requests) |
| `WKE-047` | The search criteria must be complete | [10](#10-regeneration-guards) |
| `WKE-048` | The requested range must lie inside the generated range | [10](#10-regeneration-guards) |
| `WKE-049` | At least one selected employee must be regenerable | [10](#10-regeneration-guards) |
| `WKE-050` | Employees holding a validated entry in the range are skipped, not refused | [10](#10-regeneration-guards) |
| `WKE-051` | Regeneration triggered by a version change skips the three guards | [10](#10-regeneration-guards) |
| `WKE-052` | Regeneration from a list of day slots skips the three guards | [10](#10-regeneration-guards) |
| `WKE-053` | What a forced regeneration writes onto the entries it supersedes | [10](#10-regeneration-guards) |
| `WKE-054` | The access matrix | [11](#11-permission-checks) |
| `WKE-055` | Work entries are restricted to the acting companies | [11](#11-permission-checks) |
| `WKE-056` | Work entry kinds are restricted by country | [11](#11-permission-checks) |
| `WKE-057` | A calendar filter row may be changed only by its own user | [11](#11-permission-checks) |
| `WKE-058` | Generation runs with elevated rights and under the group's company | [11](#11-permission-checks) |
| `WKE-059` | Field-level restrictions on the version and the employee | [11](#11-permission-checks) |
| `WKE-060` | The generation source is a one-value selection with an extension contract | [12](#12-the-generation-source-extension-point) |
| `WKE-061` | A version claiming a schedule it does not have is flagged, not blocked | [12](#12-the-generation-source-extension-point) |
| `WKE-062` | A user may pin an employee into the calendar only once | [13](#13-everything-else) |
| `WKE-063` | Generation refuses to run without a time zone | [13](#13-everything-else) |
| `WKE-064` | A value set must carry either two instants or a date and a duration | [13](#13-everything-else) |
| `WKE-065` | The two guards on splitting an entry | [13](#13-everything-else) |
| `WKE-066` | The version-level entry point takes dates, never instants | [13](#13-everything-else) |
| `WKE-067` | Concurrency and locking | [13](#13-everything-else) |
