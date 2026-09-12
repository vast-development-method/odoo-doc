# Recruitment — Calculations

Every number this domain produces, with its inputs, the order in which the operations are
evaluated, what is rounded and how, and at least one worked example carried to the last
decimal the rule produces. Nothing here depends on a particular technology: each rule is a
numbered procedure or a formula over quantities named in words.

## 1. Conventions used in this file

- **Quantities** are named in words in the formula blocks. Where a quantity is the value of
  a stored field, the field's identifier is given in code font in the surrounding prose.
- **Counts** are whole numbers and are never rounded, because nothing fractional enters
  them.
- **Durations** are computed from moments recorded with a resolution of one second. A day is
  86 400 seconds exactly; no calendar arithmetic, no time-zone shift and no working-calendar
  is applied anywhere in this domain.
- **Percentages** produced by the skill match are expressed on a scale of 0 to 100.
- **Elevated rights** means the computation ignores the reader's record rules. It is stated
  where it applies, because it changes which records enter a count.
- Unless a rule says otherwise, a search that feeds a calculation of this domain **includes
  archived records**; the rules that do not are named explicitly.

---

## 2. Counters on a Job Position

Seven counters live on a Job Position. All of them count Applications; none of them rounds
anything. All of them are restricted to the Interviewer privilege or above, except the
employee count, which is not restricted.

### 2.1 The four straightforward counts

| Counter (storage name) | Set counted |
|---|---|
| Application Count (`application_count`) | Applications of the position, archived ones excluded |
| All Application Count (`all_application_count`) | Applications of the position, archived ones included, keeping a record when it is **active**, or when it is **archived and carries a refusal reason**. An application archived without a reason is therefore excluded. |
| Open Application Count (`open_application_count`) | Applications of the position, archived ones excluded, whose stage is **not** flagged as a hired stage |
| Applicants Hired (`applicant_hired`) | Applications of the position, archived ones excluded, whose stage **is** flagged as a hired stage |
| Hired (`no_of_hired_employee`) | Applications of the position, **archived ones included**, that carry a hire date. This one is stored and is recomputed whenever the hire date of any application of the position changes. |

Worked example. A position holds ten active applications, of which three sit in its first
stage and one sits in the hired stage; it also holds four archived applications carrying a
refusal reason and one archived application carrying none. Two of the applications with a
hire date have since been archived.

```formula
Application Count      = 10 applications
Open Application Count = 10 − 1 = 9 applications
Applicants Hired       = 1 application
All Application Count  = 10 + 4 = 14 applications
Hired                  = 1 active with a hire date + 2 archived with a hire date = 3 applications
```

### 2.2 New application count

Depends on the position's stages and on the acting user's allowed companies. Computed with a
single grouped read over the applications, with elevated rights on the stage resolution.

1. For the position, take the stages attached to it and the stages attached to no position
   at all.
2. Order that set by sequence ascending and keep the **first** one. This is the position's
   *first stage*. Folded stages are **not** excluded here.
3. Count the applications of the position that are active, that sit in that first stage, and
   whose company is among the acting user's allowed companies **or** is empty.

```formula
New Application = number of active applications of the position
                  whose stage is the position's first stage
                  and whose company is allowed to the reader or is empty
```

Worked example. A position may use `New` (sequence 0, unrestricted), `Qualification`
(sequence 1, unrestricted) and `Screening` (sequence 0, restricted to this position). Three
active applications sit in `New`, one in `Screening` and six in `Qualification`. `New` and
`Screening` tie at sequence 0.

**industry-standard default** — the tie is not resolved by the rule as observed, so the
counter may report either 3 or 1 depending on the order the storage layer returns. A rebuild
must break the tie deterministically by ordering on sequence ascending and then on the
stage's identifier ascending, which selects `New` in this example and yields a New
Application count of 3.

### 2.3 Old application count

```formula
Old Application = Application Count − New Application
```

With the numbers of §2.1 and a New Application count of 3: `10 − 3 = 7 applications`.

### 2.4 Personal activity count

Depends on the acting user. It is the workload badge shown on the position card.

Count the scheduled activities such that all of the following hold:

1. the activity's owner is the acting user;
2. the activity's record is an Application;
3. that Application is active;
4. that Application belongs to this position;
5. the stage of that Application is **not** flagged as a hired stage;
6. the activity itself is active.

```formula
Activity Count = number of the acting user's own open activities
                 on active, not-yet-hired applications of the position
```

Two readers looking at the same position therefore see different badges. That is intended:
the badge is a personal workload, not a position statistic.

### 2.5 Document count

```formula
Documents = attachments whose owning record is the position
          + attachments whose owning record is an application of the position
            that has not yet produced an Employee

Document Count = number of entries in Documents
```

Applications that already produced an Employee are excluded because their files have been
copied onto the Employee and belong to the people register from then on.

### 2.6 Employee count

Counted with elevated rights, and **not** restricted to the Interviewer privilege:

```formula
Employee Count = number of employees whose job position is this one
                 and whose company is among the reader's allowed companies
```

### 2.7 Forecast headcount

This formula belongs to [Human Resources Core](../human-resources-core/README.md) and is
repeated here because the hiring procedure changes both of its inputs:

```formula
Total Forecasted Employees = Current Number of Employees + Target
```

Worked example, showing that the forecast is stable across a hire. A position has five
employees and a target of two, so the forecast is seven. An application is moved into the
hired stage: the target falls to one and the forecast falls to six. The Employee is then
created from that application: the employee count rises to six and the forecast returns to
seven. The dip to six between the two steps is real and visible, and lasts exactly as long
as the interval between recording the hire and creating the Employee.

---

## 3. Counters on a Department

```formula
New Applicant        = number of applications of the department
                       whose stage sequence is at most 1
New Hired Employee   = sum of the Hired counter over the positions of the department
Expected Employee    = sum of the Target over the positions of the department
```

The new-applicant count is computed with elevated rights, which is why the privilege test is
explicit: a reader who does not hold at least the Interviewer privilege receives **zero**
rather than a refusal. The threshold is the stage's **sequence**, not the position's first
stage, so with the shipped stages it covers `New` (sequence 0) and `Qualification`
(sequence 1).

Worked example. The department *Research and Development* holds two positions. The first
has a Hired counter of 2 and a Target of 3; the second has a Hired counter of 1 and a Target
of 0.

```formula
New Hired Employee = 2 + 1 = 3 employees
Expected Employee  = 3 + 0 = 3 employees
```

---

## 4. Duplicate and similar application detection

Everything this domain does about repeated candidates rests on one condition, defined once
here and reused by the application counter, the refusal dialog, the statistic button, the
pool resolution and the public warning.

### 4.1 Application count

**The matching condition.** Given a set of Applications, another Application belongs to the
same people when **any** of the following holds:

1. its identifier is in the set;
2. its normalised electronic mail address equals one of the non-empty normalised addresses
   of the set;
3. its sanitised telephone number equals one of the non-empty sanitised numbers of the set;
4. its professional network profile address equals one of the non-empty profile addresses of
   the set;
5. its canonical pool copy is one of the non-empty canonical pool copies of the set.

Two narrowings are used: *ignoring talents*, which adds "and the pool list is empty", and
*only talents*, which adds "and the pool list is not empty".

Three properties of the comparison must be reproduced exactly:

- Addresses are compared on their **normalised** form, which is lower case and carries no
  display name. `laurie.poiret@aol.ru` and `laurie.POIRET@aol.ru` are the same person.
- Telephone numbers are compared on `partner_phone_sanitized`, which is the international
  form when the number can be formatted and the **raw** value otherwise. Two spellings of the
  same unformattable number, for example `123` and `+49 123`, are therefore **not** matched.
- An empty value never matches. An Application with no address, no telephone number, no
  profile address and no canonical pool copy matches nobody — not even itself.

**The counter.** `application_count` on an Application is computed as follows:

1. Build the matching condition for the Applications being read, narrowed by *ignoring
   talents*, and search it **with archived records included**. Call the result the match
   set.
2. Over the match set build four maps: normalised address to identifiers, sanitised
   telephone number to identifiers, profile address to identifiers, canonical pool copy to
   identifiers.
3. For each Application being read, start from an empty set of identifiers and add, for each
   of the four keys the Application itself carries, the identifiers found in the
   corresponding map. Keys the Application does not carry are skipped.
4. The counter is the size of that set, and never less than zero.

Because the Application's own identifier is in every map entry built from a key it carries,
an Application that carries at least one key counts **itself**.

**Worked example.** Seven Applications are created together:

| Application | Active | Normalised address | Sanitised telephone number | Identifiers gathered | Count |
|---|---|---|---|---|---|
| A | no (archived) | `abc@example.com` | `123` | A, C, D | 3 |
| B | yes | empty | `456` | B, D | 2 |
| C | yes | `def@example.com` | `123` | C, A | 2 |
| D | yes | `abc@example.com` | `456` | D, A, B | 3 |
| E | yes | empty | empty | none | 0 |
| F | yes | `ghi@example.com` | `789` | F | 1 |
| G | yes | empty | empty | none | 0 |

A is archived and is still counted: a person who was refused once and applies again must be
visible as a repeat applicant. E and G carry no key at all and therefore report zero, not
one.

**Second worked example.** Three Applications carry the address `test@example.com`. Each
reports a count of 3. Archiving one of them and refusing another leaves all three counts at
3, because the search ignores both the active flag and the refusal reason.

### 4.2 Refusal duplicate set

The refusal dialog offers to refuse, together with the selection, every other live
application of the same people. The set is:

```formula
duplicate set = applications matching the selection by the condition of §4.1
              − the applications already selected
              − the applications whose derived status is hired, refused or archived
```

The status exclusion uses the **derived** status, so an application that is active and
carries no hire date qualifies, and one that is merely archived does not.

The count shown in the dialog is the size of that set. Turning the switch on loads the whole
set into an editable list; turning it off empties the list. The user may remove individual
lines before applying, and only the lines that remain are refused.

**Resolving the original of each duplicate.** For the explanatory note written on each
refused duplicate, the selected application it duplicates is found by testing, **in this
exact order** and stopping at the first hit:

1. the identifier;
2. the normalised electronic mail address;
3. the sanitised telephone number;
4. the professional network profile address.

A duplicate that matches none of the four keeps no original and receives no note.

**Worked example.** The selected application carries the address `laurie.poiret@aol.ru` and
the telephone number `0470`. Two other live applications exist: one with the same address
and a different number, one with the same number and no address. Both enter the duplicate
set. The first is matched by address, the second by telephone number, and both receive a
note pointing at the selected application.

### 4.3 Pool membership

`is_applicant_in_pool` is a truth value derived as follows:

1. An Application is linked to a pool **directly** when it carries at least one pool of its
   own, or when its canonical pool copy is set.
2. An Application is linked to a pool **indirectly** when it shares a normalised address, a
   sanitised telephone number or a professional network profile address with at least one
   directly linked Application. Only **active** records are searched for this step.
3. Indirect links are deliberately **not transitive**: sharing a key with a record that is
   itself only indirectly linked does not put an Application in a pool.

The searchable form of the same question, used to filter the candidates the add-to-pool
dialog offers, is: the set of Applications that carry a canonical pool copy or at least one
pool, plus every Application whose normalised address, sanitised telephone number or profile
address appears among the keys of that first set.

### 4.4 Talent pool count

`talent_pool_count` answers "in how many pools is this person?" and is computed in four
steps:

1. Applications that are not in a pool at all (§4.3) receive zero, and the computation stops
   for them.
2. Applications whose canonical pool copy is set receive the number of pools of that
   canonical copy. A talent points at itself, so a talent receives the number of its own
   pools.
3. For the remaining Applications — in a pool only indirectly — search every **active**
   Application that carries at least one pool or a canonical pool copy and that shares an
   address, a sanitised telephone number or a profile address with them. Build three maps
   from each of those three keys to the pool count of the matched record's canonical copy.
4. For each remaining Application take, **in this order**, the count found by address; if it
   has no address or no match, the count found by telephone number; failing that, the count
   found by profile address; failing all three, zero.

**Worked example.** Two pools exist, `Cool Pool` and `Other Pool`. Talent *tA* belongs to
both and carries the address `abc@example.com`, the telephone number `1234` and the profile
address `linkedin/talent`. Talent *tB* belongs to `Other Pool` only. Six further
Applications exist: *A* points at *tA* and carries no key of its own; *B* points at *tA* and
carries `def@example.com`, `6789` and `linkedin/b`; *C* carries only `def@example.com`; *D*
carries only `6789`; *E* carries only `linkedin/b`; *F* carries keys that match nothing;
*G* points at *tB*.

| Record | How it is linked | Talent pool count |
|---|---|---|
| *tA* | its own pools | 2 |
| *tB* | its own pools | 1 |
| *A* | canonical copy *tA* | 2 |
| *B* | canonical copy *tA* | 2 |
| *C* | shares the address of *B* | 2 |
| *D* | shares the telephone number of *B* | 2 |
| *E* | shares the profile address of *B* | 2 |
| *F* | nothing | 0 |
| *G* | canonical copy *tB* | 1 |

### 4.5 The live warning on the public application form

The public form asks the server, while the visitor types, whether a recent application
already exists. The question carries one of four key names, the value typed, and the
identifier of the position. The answer is one message, or none.

1. Build the matching condition for the named key:
   - the key *name* matches the applicant's name, case-insensitively;
   - the key *email* matches the normalised address exactly, after normalising the value
     typed;
   - the key *phone* matches the telephone number exactly, without sanitisation;
   - the key *linkedin* matches the professional network profile address,
     case-insensitively;
   - any other key yields a condition that matches nothing.
2. Intersect it with: the position belongs to this website or to no website, **and** the
   derived status is `ongoing` or `refused`.
3. Read the matches with elevated rights, newest first, and group them by derived status.
4. If **any** refused match is inactive, belongs to this very position, and was created
   within the last six months, answer message 1 and stop.
5. Otherwise, if there is no ongoing match, answer nothing and stop.
6. Otherwise take the **newest** ongoing match. If it belongs to this very position, answer
   message 3; otherwise answer message 4.

The four answers are given in full, with their placeholders, in
[business-rules.md](business-rules.md#7-the-public-application-form).

Note the asymmetry between the key *phone*, which compares the raw telephone number, and the
duplicate counter of §4.1, which compares the sanitised number. The same person typing the
same number in two different spellings therefore receives no warning while still being
counted as a duplicate afterwards. **compatibility finding** — a corrected behaviour would
compare the sanitised number here too.

---

## 5. Elapsed times on an Application

All three values are decimal numbers of days, computed with elevated rights, and none of
them is rounded.

### 5.1 Days to open

```formula
Days to Open = ( Assigned moment − Applied-on moment ) ÷ 86 400 seconds per day
```

Empty when the assignment moment is empty. *Applied on* is the creation moment
(`create_date`), *Assigned* is `date_open`.

### 5.2 Days to close

```formula
Days to Close = ( Hire Date moment − Applied-on moment ) ÷ 86 400 seconds per day
```

Empty when the hire date is empty.

### 5.3 Delay to close

```formula
Delay to Close = Days to Close − Days to Open
```

Empty when the assignment moment is empty or the days-to-close value is empty. This is the
only one of the three that is **stored**, and it is aggregated as an average in grouped
reads, which gives the average number of days between assignment and hire.

**Worked example.** An Application is created on 2 March 2026 at 09:00, a recruiter is
assigned on 4 March 2026 at 15:00, and the person is hired on 20 March 2026 at 09:00.

```formula
Days to Open   = ( 2 days + 6 hours ) ÷ 1 day = 194 400 s ÷ 86 400 s = 2.25 days
Days to Close  = 18 days                      = 1 555 200 s ÷ 86 400 s = 18.00 days
Delay to Close = 18.00 − 2.25                 = 15.75 days
```

---

## 6. Time in stage

The Application records, for every stage it has been in, the number of whole seconds spent
there. The map is derived on read and never stored.

1. Start the clock at the creation moment. For a record that is not yet stored, start it at
   the current moment of the running operation.
2. Read the stage-change entries of the record's thread in chronological order. Each entry
   closes one interval: add the whole seconds between the previous moment and the entry's
   moment to the stage the entry **left**, then move the previous moment to the entry's
   moment.
3. When a stage change is pending in the operation being executed and differs from the
   stored stage, insert a synthetic entry at the current moment for the stage being left, so
   that the elapsed time is not credited to the stage being entered.
4. Append a synthetic entry at the current moment carrying the current stage, which credits
   the time spent in the current stage up to now.
5. **Refusal correction.** When the Application carries both a refusal reason and a refusal
   moment, subtract from the current stage's total the seconds elapsed between the refusal
   moment and now. Without this the clock of a refused application would keep running for
   ever.

**Worked example.** An Application is created on 1 March at 08:00, moved to `Qualification`
on 3 March at 08:00, refused on 5 March at 08:00, and the durations are read on 9 March at
08:00.

```formula
New            = 3 March 08:00 − 1 March 08:00 = 172 800 seconds  ( 2 days )
Qualification  = 9 March 08:00 − 3 March 08:00 = 518 400 seconds  ( 6 days )
refusal correction = − ( 9 March 08:00 − 5 March 08:00 ) = − 345 600 seconds ( 4 days )
Qualification  = 518 400 − 345 600 = 172 800 seconds  ( 2 days )
```

The result is `New` 172 800 seconds and `Qualification` 172 800 seconds. The correction is
applied at every read, so the refused application's totals stay constant however long it
stays refused.

---

## 7. Staleness (rotting)

An Application is *stale* when it has sat in its stage longer than that stage allows.

```formula
eligible     = derived status is ongoing
               AND Hire Date is empty
               AND the stage's Days to rot is not zero

is stale     = eligible
               AND ( Last Stage Update + Days to rot days ) is earlier than now

Days Rotting = whole days between Last Stage Update and now, when stale
             = 0, otherwise
```

- *Last Stage Update* is `date_last_stage_update`, whose default is the creation moment and
  which is refreshed by every stage change and by every readiness change.
- *Days to rot* is `rotting_threshold_days` on the current stage. Zero disables staleness
  for that stage; with the shipped configuration every stage carries zero, so the feature is
  inactive until an administrator sets a threshold.
- The whole-day count truncates: a partial last day does not count.
- Changing a stage's threshold does not rewrite anything on the applications; the comparison
  always uses the current threshold against the stored update moment, so the effect of a
  change is immediate on the next read.

**Worked example.** A stage carries a threshold of 10 days. An Application's last stage
update was on 1 March at 12:00 and the report is read on 12 March at 09:00.

```formula
1 March 12:00 + 10 days = 11 March 12:00
11 March 12:00 is earlier than 12 March 09:00 → the application is stale
elapsed = 12 March 09:00 − 1 March 12:00 = 10 days 21 hours
Days Rotting = 10 days   ( the partial eleventh day is not counted )
```

**Searching.** Only equality operators are accepted on the staleness fields; any other
operator is refused with `For performance reasons, use "=" operators on rotting fields.`
When no stage in the installation defines a threshold, the feature is inactive altogether
and a search is refused with `Model configuration does not support the rotting feature`. The
search compares the stored last-update moment plus the stage's threshold against the present
moment directly in the storage layer, and ignores applications whose last update is older
than the look-back window, which is twelve months by default and is a platform-wide
parameter shared with the opportunity pipeline.

---

## 8. Skill matching

Present when the skills companion is installed. The same arithmetic is evaluated from two
sides, with three deliberate differences listed in §8.2.

### 8.1 Applicant-to-position match score

**Inputs.** The position's required skills with the progress value of each required level;
the score of the position's expected degree; the Application's *current* skills with the
progress value of each held level; the score of the Application's degree.

**Units and precision.** A level's progress is a whole number from 0 to 100. A degree's
score is a decimal from 0 to 1 and is presented to the user as a percentage. The result is a
whole number from 0 to 100.

**The position compared against** is the one named by the reading context flag
`matching_job_id` when the reader is browsing a matching list, and the Application's own
Job Position otherwise. When there is no position, or the position requires neither a skill
nor an expected degree, the score is 0 and both skill lists are empty.

```formula
position degree points  = expected degree score × 100
position total          = sum over required skills of ( required level progress )
                          + position degree points

matching skills         = the Application's current skills whose skill is required
                          by the position
missing skills          = the required skills − the matching skills

applicant degree points = applicant degree score × 100, when position degree points > 1
                        = 0,                            otherwise

applicant total         = sum over matching skills of
                              minimum of ( held level progress ,
                                           required level progress × 2 )
                          + applicant degree points

Matching Score          = round ( applicant total ÷ position total × 100 ),
                          when position total is not zero
                        = 0, when position total is zero
```

**Order of evaluation.** The position total is formed first, because it decides whether the
degree contributes on the applicant side. The cap is applied per skill, before the sum. The
division is performed on the exact sums, and only the final percentage is rounded, to the
nearest whole number, halves going to the even neighbour.

**Two deliberate properties.**

- The cap *minimum of held progress and twice the required progress* means that exceeding
  the required level pays at most double, so one very strong skill cannot hide several
  missing ones.
- The degree contributes on the applicant side only when the position states an expectation
  worth more than one point, that is a degree score above 0.01. A position that expects a
  degree of score 0.01 or less therefore ignores the candidate's degree entirely.

**Worked example.** The position requires `Skill A` at a level whose progress is 50 and
`Skill B` at a level whose progress is 100, and expects the degree `Master Degree`, whose
score is 0.90. The candidate currently holds `Skill A` at a level whose progress is 100 and
holds the degree `Bachelor Degree`, whose score is 0.70.

```formula
position degree points  = 0.90 × 100 = 90
position total          = 50 + 100 + 90 = 240
applicant degree points = 0.70 × 100 = 70        ( because 90 > 1 )
capped Skill A          = minimum of ( 100 , 50 × 2 ) = 100
applicant total         = 100 + 70 = 170
Matching Score          = round ( 170 ÷ 240 × 100 ) = round ( 70.8333… ) = 71
matching skills         = { Skill A }
missing skills          = { Skill B }
```

**Second worked example, no expected degree.** The same position without an expected degree
gives position degree points 0, a position total of 150, applicant degree points 0, an
applicant total of 100, and a score of `round ( 66.666… ) = 67`.

**Third worked example, the cap biting.** The position requires `Skill A` at progress 20 and
nothing else, with no expected degree. The candidate holds `Skill A` at progress 100. The
capped contribution is `minimum of ( 100 , 40 ) = 40`, the position total is 20, and the
score is `round ( 40 ÷ 20 × 100 ) = 200`. The value exceeds 100 and is not clamped.
**compatibility finding** — a corrected behaviour clamps the score to 100; a rebuild that
reproduces the observed behaviour must expect scores above 100 whenever a candidate exceeds
every required level.

### 8.2 Position-to-applicant match score

`applicant_matching_score` on a Job Position answers the mirror question: how well does one
named candidate match this position? It uses the arithmetic of §8.1 with three differences:

1. It is computed only when the reading context names an Application through the flag
   `active_applicant_id`; with no such flag every position reports an empty value.
2. It is empty for a position that requires **no skill**, even when the position expects a
   degree. The applicant side, by contrast, also produces a value when only a degree is
   expected.
3. The result is **not rounded**. It is the exact quotient multiplied by one hundred, and
   the screen displays it with its own precision.

With the numbers of the first worked example of §8.1 the value is 70.8333…, displayed as
70.83, where the applicant side reports 71.

**industry-standard default** — when a position requires skills whose level progress values
are all zero and expects no degree, the position total is zero and the quotient is
undefined. The applicant side guards this case and yields 0; the position side does not. A
rebuild must yield 0 on both sides rather than failing.

### 8.3 Current skills

The *current* skills of an Application are the subset of its skill lines that count today.

1. Group every skill line of the Application by the pair (Application, skill).
2. Within a group, keep the lines whose validity end is empty or is **not earlier than
   today**.
3. If the group's skill belongs to a skill type marked as a certification and step 2 kept
   nothing, keep instead the single line of that group with the **latest** validity end, so
   that an expired certification remains visible.
4. The union of the kept lines is the current-skill set.

Only current skills enter the match score; the full list, expired lines included, remains
available for history.

---

## 9. Job list filter counters and pagination

Present when the public job pages companion is installed. This is the arithmetic behind the
counters shown next to each filter value of the public job list, and behind its pager.

**Inputs.** The published positions of the website being visited; the visitor's filter
choices (country, department, office address, employment type, industry, and the four
"unspecified" switches); the search text; the page number.

**Outputs.** The positions shown on the page, the pager, and one counter map per filter.

1. Resolve each filter value into a record. A value that is not a whole number, or that
   names a record that no longer exists, is treated as absent.
2. Apply the default-country rule: when no country, department, office or employment-type
   filter is set and the visitor did not ask for every country, detect the country from the
   visitor's network address and keep it as the country filter **only** when at least one
   published position has a job location in that country.
3. Search the published positions of this website matching the search text, tolerating
   spelling mistakes unless the visitor disabled it, ordered by publication state
   descending, then sequence ascending, then remaining target descending, and limited to
   `12 × 50 = 600` results.
4. **The contamination rule.** For each of the four record-valued filters — office address,
   department, employment type, industry — recompute the set of positions that pass **every
   other** filter, group that set by the field and count each group. Add an entry *all*
   whose value is the size of that set. Move the entry for "no value" to the end of the map
   and the entry *all* to the front.
5. For the country filter do the same over the country of the position's job location.
6. Apply every filter to obtain the positions actually found and count them.
7. Compute the pager with twelve positions per page:

```formula
offset          = ( page number − 1 ) × 12 positions
positions shown = the found positions from offset to offset + 12
number of pages = the found count ÷ 12, rounded up to the next whole number
```

The contamination rule is what lets a visitor see, next to each possible value of one
filter, how many positions would be found if that value were chosen, while every other
filter stays applied.

**Worked example.** Twenty-seven published positions match the search text. The visitor then
filters on the department *Research and Development*, which leaves nine.

```formula
without the department filter : 27 positions ÷ 12 = 3 pages ( 12, 12 and 3 positions )
with the department filter    :  9 positions ÷ 12 = 1 page  ( 9 positions )
page 2 with the filter applied: offset = ( 2 − 1 ) × 12 = 12, and 12 is beyond 9 → empty
```

---

## 10. Meeting summary on an Application

**Inputs.** The Application's meetings and their start moments; today's date in the reader's
time zone. **Outputs.** The displayed date and the displayed text, both derived and
language-dependent.

1. An Application with no meeting shows the text *No Meeting* and an empty date.
2. Otherwise take the earliest start date and the latest start date of its meetings.
3. If the earliest is today or later, the displayed date is the **earliest**; otherwise it is
   the **latest**.
4. If the Application has exactly one meeting, the text is *1 Meeting*. Otherwise, if the
   displayed date is today or later, the text is *Next Meeting*; otherwise it is
   *Last Meeting*.

**Worked example.** Three meetings on 2 March, 10 March and 20 March, read on 12 March. The
earliest (2 March) is in the past, so the displayed date is the latest, 20 March. There is
more than one meeting and 20 March is not earlier than today, so the text is *Next Meeting*.

---

## 11. Colour index of a new tag and a new pool

A new Application Tag and a new Talent Pool receive a pseudo-random whole number between 1
and 11 inclusive as their colour index, drawn once at creation. The value carries no business
meaning, is never recomputed, and exists only so that new labels are visually distinct
without configuration. Every other colour index in the domain defaults to 0.

---

## 12. The periodic digest indicator

```formula
New Employees = number of employees created between the start and the end
                of the digest period, restricted to the company of the digest
```

A reader who does not hold the Officer privilege causes the computation to raise an access
error carrying the text `Do not have access, skip this data for user's digest email`, which
the digest sender catches, omitting the indicator for that recipient rather than failing the
whole digest.

---

## 13. Rounding summary

| Value | Rounding |
|---|---|
| Matching Score on an Application | To the nearest whole number, halves going to the even neighbour |
| Matching Score(%) on a Job Position | Not rounded; the exact quotient times one hundred, displayed with the screen's own precision |
| Days to Open, Days to Close, Delay to Close | Not rounded; stored and reported as decimal days |
| Time in stage | Truncated to whole seconds |
| Days Rotting | Truncated to whole days; a partial last day does not count |
| Degree score | Stored as a decimal between 0 and 1; displayed as a percentage |
| Every counter of §2, §3 and §4 | Whole numbers; nothing to round |
| Salary amounts | Stored exactly as entered. No currency, no conversion, no rounding rule of this domain. They are aggregated as averages in the analysis screens, and the average carries the full precision of the underlying decimals. |
| Job list pagination | Twelve per page; the number of pages is rounded up |

---

## 14. Reconciliation notes

| Point | Resolution |
|---|---|
| Whether the application counter always includes the record itself | One version said always; the other said only when the record carries at least one key. The second is correct, and §4.1 gives the worked example with two records reporting zero. |
| The telephone comparison of the public live warning | One version described it as an exact comparison on the raw number, the other as a comparison on the sanitised number. The raw number is used on that path and the sanitised number everywhere else; the difference is recorded in §4.5 as a compatibility finding. |
| Rounding of the match score | One version rounded both sides, the other rounded only the applicant side. Only the applicant side is rounded; §8.2 states the difference and gives both values for the same inputs. |
| Whether the match score can exceed 100 | Neither version said so explicitly. It can, whenever a candidate exceeds every required level; §8.1 gives the arithmetic and records it as a compatibility finding. |
| The first stage used by the new-application counter | Both versions describe the same query and both note that two stages of equal sequence tie. The deterministic tie-break is now stated as an industry-standard default in §2.2. |
| Days-to-open when the recruiter is cleared | One version noted that clearing the recruiter does not clear the assignment moment. Confirmed: the assignment moment is written only when a non-empty recruiter is written, and is never cleared, so Days to Open survives the recruiter being removed. |
