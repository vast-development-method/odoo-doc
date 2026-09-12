# State machines

This file specifies every state-bearing field of the Customer Relationship Management domain: the
maturity of a Lead, its position in the pipeline, its won or lost outcome, its archival flag, the
status of a Lead Generation Request, and the partner-network assignment status. For each, a table
of states, a table of transitions with guards and side effects, and a diagram.

Entities are named in full on first use with their transport and storage names: Lead (`crm.lead`,
table `crm_lead`), Stage (`crm.stage`, table `crm_stage`), Sales Team (`crm.team`, table
`crm_team`), Lead Generation Request (`crm.iap.lead.mining.request`, table
`crm_iap_lead_mining_request`).

A remark on shape. The Lead does **not** have a single status field. It has four orthogonal
dimensions, and the business meaning of a record is the combination:

| Dimension | Field | Nature |
|---|---|---|
| Maturity | `type` (type) | two values, normally one-way |
| Pipeline position | `stage_id` (stage) | an ordered set of user-defined values |
| Outcome | `won_status` (won or lost) | **derived**, never written directly |
| Existence | `active` (active) | a boolean archival flag |

Specifying them separately, and then specifying how they combine, is the only way to describe the
behaviour correctly.

---

## 1. Maturity — the `type` field

### 1.1 States

| Value | Label | Meaning |
|---|---|---|
| `lead` | Lead | An unqualified interest. May have nothing but an electronic mail address. Not expected to carry a Contact, an expected revenue or a meaningful probability. |
| `opportunity` | Opportunity | A qualified deal, expected to carry a Contact, a salesperson, a team, a stage and an expected revenue. |

The field is **required**; there is no empty state.

### 1.2 Default

`lead` when the acting user holds the group *Show Lead Menu*; `opportunity` otherwise. Records
created through the incoming electronic mail gateway on a team alias take the value stored in the
alias defaults, which is `lead` when the alias owner uses leads and the group is granted, and
`opportunity` otherwise. Records created through the public website form take `lead` when the
resolved team uses leads and `opportunity` otherwise; with no resolved team the group decides.

### 1.3 Transitions

| From | To | Trigger | Guards | Side effects |
|---|---|---|---|---|
| `lead` | `opportunity` | The conversion routine, invoked by the single-conversion wizard, by the mass-conversion wizard, by the assignment routine when it hands a lead to a member, or by the partner-network "I am interested" action. | The record must be **active**. Its outcome must not be `won`. Both are checked per record; a record failing either is silently skipped, not rejected. | `date_conversion` (conversion date) is stamped with the current instant. When the chosen Contact differs from the current one, `partner_id` (contact) is updated. When the record has no stage, one is chosen by the stage-search routine using the target team. Afterwards, if a salesperson list or a team was supplied, they are applied through the round robin. |
| `lead` | `opportunity` | A merge whose inputs include at least one opportunity. | The merge guards (at least two records, at most five unless relaxed). | The surviving record's type becomes `opportunity`. |
| `opportunity` | `lead` | Not offered by any action. | — | The field is writable, so a direct write is technically possible, but no workflow performs it and the conversion is treated as one-way. |

### 1.4 Diagram

```mermaid
stateDiagram-v2
    [*] --> Lead: created while the lead menu group is granted
    [*] --> Opportunity: created while the lead menu group is not granted
    Lead --> Opportunity: conversion (active and not won)
    Lead --> Opportunity: merge with at least one opportunity
    Opportunity --> [*]: won, lost or deleted
    Lead --> [*]: merged away or deleted
```

---

## 2. Pipeline position — the `stage_id` field

### 2.1 States

The states are the Stage records themselves; they are configuration data, not a fixed enumeration.
The delivered set is:

| Stage | Sequence | Won flag | Folded | Colour |
|---|---|---|---|---|
| New | 1 | no | no | 11 |
| Qualified | 2 | no | no | 5 |
| Proposition | 3 | no | no | 8 |
| Won | 70 | **yes** | no | 10 |

A stage may additionally be restricted to a set of teams. A record may also have **no** stage at
all, which is a meaningful state: it suppresses the automated probability entirely.

### 2.2 Transitions

| From | To | Trigger | Guards | Side effects |
|---|---|---|---|---|
| *(none)* | the first non-folded stage available to the record's team | Automatic, on creation, and whenever the team changes such that the current stage is no longer available. | The stage-search routine must find a candidate; otherwise the record keeps no stage. | `date_last_stage_update` (last stage update) is stamped. |
| any stage | any other stage | A user drags the card, edits the field, or a routine writes it. | The interface restricts the selectable stages to those with no team restriction or with the record's team among their teams; a direct write is not restricted. | `date_last_stage_update` is stamped with the current instant, but **only** when at least one record in the write actually changes stage. The stage-duration tracking accumulates the time spent in the previous stage. A tracked message is posted. The probability is recomputed unless it is manual. |
| any stage | a stage flagged as won | The same triggers, plus the explicit "mark won" action. | — | In addition to the above: `active` is forced to true, `probability` is forced to 100 and `automated_probability` is forced to 100, all inside the same write. `date_closed` (closed date) is stamped — **except** when the record was already in a won stage, in which case the closed date is left untouched so that moving between two won stages does not reset it. The frequency table receives a "reaches won" increment. The posted message uses the *Opportunity Won* subtype. |
| a stage flagged as won | a stage not flagged as won | The same triggers. | — | `date_closed` is cleared when the write leaves the probability strictly positive. The frequency table receives a "leaves won" decrement. The outcome becomes `pending` unless the record is archived **and** the probability is zero. |

### 2.3 Guard enforced by a constraint

A record whose stage is flagged as won and whose probability is not exactly 100 is rejected with
"A lead in a Won stage cannot be lost. Move it to another stage first." This is what makes "in a
won stage" and "probability 100" equivalent in practice.

### 2.4 Effect of reconfiguring a stage

Toggling a stage's won flag rewrites every lead currently in that stage: setting the flag forces
their probability and automated probability to 100; clearing it recomputes their probability from
the automatic computation. The interface warns before saving.

### 2.5 Diagram

```mermaid
stateDiagram-v2
    [*] --> NoStage: created with no stage and no candidate found
    [*] --> New: created, first non-folded stage of the team
    NoStage --> New: a team is set and a candidate stage exists
    New --> Qualified: user moves the card
    Qualified --> Proposition: user moves the card
    Proposition --> Won: mark won, or move to a won stage
    New --> Won: mark won
    Qualified --> Won: mark won
    Won --> Proposition: moved back out of the won stage
    note right of Won
        entering forces active = true,
        probability = 100, automated probability = 100,
        and stamps the closed date unless already in a won stage
    end note
    note right of NoStage
        no stage means no automated probability
    end note
```

---

## 3. Outcome — the `won_status` field

### 3.1 Nature

This field is **computed and stored**; it is never written directly. It depends on `active`,
`probability` and `stage_id`.

### 3.2 States

| Value | Label | Definition |
|---|---|---|
| `won` | Won | `probability` is exactly 100 **and** the stage is flagged as a won stage. |
| `lost` | Lost | `active` is false **and** `probability` is exactly 0. |
| `pending` | Pending | Everything else. |

The two positive definitions are checked in that order, so a record that somehow satisfied both
would be reported as won — but that situation is rejected by the constraint below.

### 3.3 The mutual-exclusion constraint

Every creation and every write that touches `active`, `stage_id` or `probability` recomputes the
outcome before and after and rejects a record that is simultaneously won and lost with the message
"The lead *the record* cannot be won and lost at the same time."

### 3.4 Transitions and their side effects on the frequency table

| From | To | Trigger | Side effect |
|---|---|---|---|
| `pending` | `won` | The record reaches probability 100 in a won stage, by the "mark won" action, by a direct stage change, or by a stage being reconfigured as won. | The frequency table is incremented in the won direction for every cell of the record, with the stage expansion "all stages". A tracked message with the *Opportunity Won* subtype is posted. |
| `won` | `pending` | The record leaves the won stage, or its probability leaves 100. | The frequency table is **decremented** in the won direction. A tracked message with the *Opportunity Restored* subtype is posted. |
| `pending` | `lost` | The "mark lost" action, or archiving a record whose probability is already zero, or setting the probability to zero on an already archived record. | The frequency table is incremented in the lost direction, with the stage expansion "stages up to and including the current one". A tracked message with the *Opportunity Lost* subtype is posted. `date_closed` is stamped. |
| `lost` | `pending` | Unarchiving, or the explicit "restore" action, or raising the probability above zero on an archived record. | The frequency table is **decremented** in the lost direction. The lost reason is cleared by the unarchive routine. A tracked message with the *Opportunity Restored* subtype is posted. |
| `lost` | `won` | Marking a lost record won (the action unarchives first, then moves it to a won stage). | Both adjustments happen in the same operation: decrement lost, increment won. |
| `won` | `lost` | Archiving a won record **and** dropping its probability to zero, in that combination. | Both adjustments: decrement won, increment lost. |

### 3.5 The two behaviours that surprise

- **Archiving a won record does not un-win it.** The record stays `won` because the stage is still
  a won stage and the probability is still 100. No frequency adjustment occurs.
- **Archiving a pending record does not lose it.** The record only becomes `lost` if its
  probability is also zero. An archived record with probability 20 is `pending`.

### 3.6 Transitions that are refused

| From | Operation | Outcome |
|---|---|---|
| `won` | Mark lost | Refused. The stage is still a won stage, so forcing the probability to zero breaks the stage constraint: "A lead in a Won stage cannot be lost. Move it to another stage first." The record stays `won`. |
| `won` | Write a probability below one hundred while the record is still in a won stage | Refused with the same message. |
| `won` | Archive | Allowed, and the record stays `won`: being lost also requires a probability of zero. No frequency counter moves. |
| `won` | Move to a stage that is not flagged as won | Allowed. The won counters are decremented. The probability is **not** forced back to a computed value by the move itself; it stays at one hundred until something recomputes it, so the record is merely "not won" because its stage is no longer a won stage. |

### 3.7 Diagram

```mermaid
stateDiagram-v2
    [*] --> Pending: created
    Pending --> Won: probability reaches 100 in a won stage
    Won --> Pending: leaves the won stage or leaves probability 100
    Pending --> Lost: archived and probability 0
    Lost --> Pending: unarchived, or probability raised above 0
    Lost --> Won: mark won (unarchives, then moves to a won stage)
    Won --> Lost: archived and probability dropped to 0
    note right of Won
        frequency table: increment won on entry, decrement won on exit
    end note
    note right of Lost
        frequency table: increment lost on entry, decrement lost on exit
    end note
```

---

## 4. Existence — the `active` field

### 4.1 States

| Value | Meaning |
|---|---|
| true | The record appears in ordinary searches and views. |
| false | The record is archived: hidden from ordinary searches, still readable by identifier, still counted by duplicate detection and by the lost-reason navigation. |

Default: true. Tracked with tracking order 72.

### 4.2 Transitions

| From | To | Trigger | Guards | Side effects |
|---|---|---|---|---|
| true | false | The generic archive operation, or the "mark lost" action. | None. | `date_closed` is stamped by the write rule whenever a write sets `active` to false. If the probability is also zero the outcome becomes `lost` and the frequency table is incremented. |
| false | true | The generic unarchive operation. | None. | Only the records that were actually inactive are affected. Their lost reason is cleared. Their probability is recomputed. If they were `lost`, the frequency table is decremented. |
| false | true | The explicit "restore" action. | None. | Everything the unarchive operation does, **plus** the manual probability is set equal to the automated probability, which re-attaches the record to the automatic computation. |

### 4.3 Why restore and unarchive differ

Unarchiving recomputes only the *automated* probability, because the manual probability is left
alone by the alignment rule once it has diverged. A record that was lost has probability 0 while
its automated probability may be, say, 18.40; after a plain unarchive it would sit at 0 with an
automated value of 18.40 and would be reported as still detached. Restore closes that gap by
copying the automated value onto the manual one.

### 4.4 Diagram

```mermaid
stateDiagram-v2
    [*] --> Active: created
    Active --> Archived: archive, or mark lost
    Archived --> Active: unarchive (clears the lost reason, recomputes the automated probability)
    Archived --> Active: restore (as unarchive, and realigns the manual probability)
```

---

## 5. The combined lifecycle of a Lead

Putting the four dimensions together, the meaningful business states are:

| Business state | `type` | `active` | Stage | `probability` | `won_status` |
|---|---|---|---|---|---|
| New unqualified lead | `lead` | true | first non-folded | computed or 0 | `pending` |
| Lead being qualified | `lead` | true | any non-won | computed | `pending` |
| Converted opportunity | `opportunity` | true | any non-won | computed | `pending` |
| Won deal | `opportunity` (normally) | true | a won stage | 100 | `won` |
| Won deal later archived | `opportunity` | false | a won stage | 100 | `won` |
| Lost deal | either | false | unchanged | 0 | `lost` |
| Restored deal | either | true | unchanged | realigned | `pending` |
| Merged away | — | — | — | — | the record no longer exists |

### 5.1 Combined diagram

```mermaid
stateDiagram-v2
    [*] --> NewLead: captured
    NewLead --> Qualifying: stage advanced, data completed
    Qualifying --> Converted: conversion wizard, or assignment to a member
    NewLead --> Converted: conversion wizard
    Converted --> Selling: quotation created
    Selling --> WonDeal: mark won
    Converted --> WonDeal: mark won
    Qualifying --> WonDeal: mark won
    NewLead --> LostDeal: mark lost
    Qualifying --> LostDeal: mark lost
    Converted --> LostDeal: mark lost
    Selling --> LostDeal: mark lost
    WonDeal --> ArchivedWon: archive
    ArchivedWon --> WonDeal: unarchive
    LostDeal --> Restored: restore
    Restored --> Qualifying: continue working the deal
    NewLead --> Merged: merged into another record
    Qualifying --> Merged: merged into another record
    Converted --> Merged: merged into another record
    Merged --> [*]
    WonDeal --> [*]
    LostDeal --> [*]
```

---

## 6. Lead Generation Request status

### 6.1 States

| Value | Label | Meaning |
|---|---|---|
| `draft` | Draft | The request is being composed. Its number is the placeholder text "New". |
| `error` | Error | The last submission failed because the external service reported insufficient credits. |
| `done` | Done | The last submission succeeded and produced leads. |

**Required**, default `draft`.

An auxiliary field records **why** a submission failed:

| Value of the error type | Label | Meaning |
|---|---|---|
| `credits` | Insufficient Credits | The service refused for lack of credits. This value is accompanied by the status `error`. |
| `no_result` | No Result | The service answered but returned nothing. The status is **left unchanged**, so a first submission that finds nothing stays `draft`. |

### 6.2 Transitions

| From | To | Trigger | Guards | Side effects |
|---|---|---|---|---|
| `draft` | `done` | Submit. | The service must answer with data. | The error type is cleared. If the number is still the placeholder, it is replaced by the next value of the numbering series. The payload is built and sent. Each returned company produces one Lead, with a note posted on it summarising the company data. The view switches to the produced leads, filtered on the requested type. |
| `draft` | `error` | Submit. | The service answers with a credit error. | The error type becomes `credits`. No lead is created. |
| `draft` | `draft` | Submit. | The service answers with no data. | The error type becomes `no_result`. No lead is created. When the request is being edited inside a dialogue, the dialogue is reopened on the same record so that the error is displayed. |
| `error` or `done` | `draft` | Reset to draft. | None. | The number is reset to the placeholder text "New", so the next submission consumes a new number from the series. |

### 6.3 Diagram

```mermaid
stateDiagram-v2
    [*] --> Draft: created
    Draft --> Done: submit, service returns companies
    Draft --> Error: submit, service reports insufficient credits
    Draft --> Draft: submit, service returns nothing (error type becomes no result)
    Error --> Draft: reset to draft (number reset to the placeholder)
    Done --> Draft: reset to draft (number reset to the placeholder)
```

---

## 7. Partner-network assignment status of an Opportunity

This is not a stored selection field; it is the combination of two fields, and it behaves as a
state machine from the point of view of the reselling partner.

### 7.1 States

| State | Condition | Meaning |
|---|---|---|
| Unassigned | `partner_assigned_id` (assigned partner) is empty | No partner has been offered the deal. |
| Offered | `partner_assigned_id` is set and the partner is not among `partner_declined_ids` (partners not interested) | The deal is waiting for the partner's answer. |
| Accepted | The partner posted the interest message; the record has been converted to an opportunity | The partner is working the deal. |
| Declined | The partner is among `partner_declined_ids` and `partner_assigned_id` is empty again | The partner refused; the deal returns to the pool but that partner is excluded from future searches. |
| Unavailable | `partner_assigned_id` is empty and the record carries the "no partner available" tag | The geographic search found nobody. |

### 7.2 Transitions

| From | To | Trigger | Guards | Side effects |
|---|---|---|---|---|
| Unassigned | Offered | The "assign partner" action, or an explicit partner choice. | The record must have a country, otherwise it is skipped and reported. | Coordinates are computed and stored if missing. A partner is selected by the widening geographic search with weighted choice. If the partner has a salesperson, that salesperson is written on the record. The partner assignment date is recomputed to today. |
| Unassigned | Unavailable | The "assign partner" action finds nobody. | — | The "no partner available" tag is added. |
| Offered | Accepted | The partner presses "I am interested" in the partner portal. | The acting portal user's commercial entity must be an ancestor of the assigned partner, otherwise "Only users with commercial partner which is a parent of the assigned partner can edit this lead." | A message "I am interested by this lead." — plus the optional comment — is posted. The record is converted to an opportunity. |
| Offered | Declined | The partner presses "I am not interested" in the partner portal. | The same access guard. | A message is posted, worded "I am not interested by this lead. I contacted the lead." or "… I have not contacted the lead." depending on the answer given, plus the optional comment. The partner's whole commercial hierarchy is unsubscribed from the record and added to the declined list. The assigned partner is cleared. If the partner marked it as unwanted solicitation, the "spam" tag is added. |
| Declined | Offered | A new "assign partner" run. | — | The declined partners are excluded from every pass of the geographic search. |

### 7.3 Diagram

```mermaid
stateDiagram-v2
    [*] --> Unassigned
    Unassigned --> Offered: assign partner (country required)
    Unassigned --> Unavailable: assign partner finds nobody
    Unavailable --> Offered: assign partner succeeds later
    Offered --> Accepted: partner declares interest (record is converted)
    Offered --> Declined: partner declines (partner added to the declined list)
    Declined --> Offered: reassigned to another partner
    Accepted --> [*]
```

---

## 8. Sales Team Member activity

### 8.1 States

| Value of `active` | Meaning |
|---|---|
| true | The membership counts towards the team's capacity, the user appears among the team's salespeople, and the pair participates in assignment. |
| false | The membership is retained for history only. |

### 8.2 Transitions

| From | To | Trigger | Guards | Side effects |
|---|---|---|---|---|
| *(none)* | true | Creating a membership, directly or by adding a user to a team's salespeople list. | In single-membership mode, the pair must not duplicate an existing **active** membership. | In single-membership mode, every other active membership of that user is archived. The member is added to the team's favourites. |
| true | false | Archiving the membership, removing the user from the team's salespeople list, archiving the user, or creating an active membership for that user in another team while in single-membership mode. | None. | The team's capacity drops by the member's capacity. The user's main team is recomputed. |
| false | true | Activating the membership. | In single-membership mode, the same duplicate check. | In single-membership mode, the user's other active memberships are archived. |

### 8.3 Diagram

```mermaid
stateDiagram-v2
    [*] --> ActiveMembership: created
    ActiveMembership --> ArchivedMembership: archived, user removed from the team, user archived, or superseded in single-membership mode
    ArchivedMembership --> ActiveMembership: activated (archives the other active memberships in single-membership mode)
```

---

## 9. Summary of every field that changes state automatically on a Lead write

This table collects, in evaluation order, everything a single write to a Lead can change beyond
the values the caller supplied. It is the authoritative reference for implementing the write path.

| Order | Condition on the incoming values | Automatic change |
|---|---|---|
| 1 | a website is supplied | the website text is cleaned (a bare domain becomes a full address) |
| 2 | a stage is supplied and at least one record's stage differs | `date_last_stage_update` is set to the current instant |
| 3 | a stage is supplied, at least one record's stage differs, and the new stage is flagged as won | `active` becomes true, `probability` becomes 100 and `automated_probability` becomes 100, all added to the same write |
| 4 | a salesperson is supplied and it is empty | `date_open` is cleared |
| 5 | a salesperson is supplied, it is not empty, and at least one record's salesperson differs | `date_open` is set to the current instant |
| 6 | the supplied probability is at least 100, **or** the supplied active flag is false | `date_closed` is set to the wall-clock instant |
| 7 | otherwise, the supplied probability is strictly greater than zero | `date_closed` is cleared |
| 8 | otherwise, the stage changed, the new stage is not won, and no probability was supplied | `date_closed` is cleared |
| 9 | the write touches `active`, `stage_id` or `probability` | the outcome of every record is captured before the write and recomputed after it, and the frequency table is adjusted accordingly |
| 10 | the new stage is a won stage | the write is split: records already in a won stage are written **without** the closed date so that it is not reset, and the others are written with it |

Step 6 is the one place where the wall clock is read rather than the transaction clock; steps 2
and 5 use the transaction clock. In practice the two differ by microseconds.

---

## 10. Telephone quality — the `phone_state` field

### 10.1 States

| Value | Label | Meaning |
|---|---|---|
| *(empty)* | *(no label)* | The record carries no telephone number, so there is nothing to judge. The empty value is itself an observation for the predictive probability and is recorded in the frequency table as the text `False`. |
| `correct` | Correct | The number could be parsed, using the record's country code as the hint when the record has a country and with no hint otherwise. |
| `incorrect` | Incorrect | Parsing the number raised an error. |

The field is computed and stored from `phone` (telephone) and from the country code of `country_id`
(country). It is never written by hand.

### 10.2 Transitions

| From | To | Trigger | Guards | Side effects |
|---|---|---|---|---|
| *(none)* | *(empty)* | Creation with no telephone. | — | None. |
| *(none)* | `correct` | Creation with a telephone that parses. | — | The value is stored at creation. |
| *(none)* | `incorrect` | Creation with a telephone that does not parse. | — | The value is stored at creation. |
| *(empty)* | `correct` | A telephone that parses is written. | — | The probability is recomputed when the telephone quality is one of the configured scoring variables. The number is also rewritten in international notation by the form rule. |
| *(empty)* | `incorrect` | A telephone that does not parse is written. | — | The probability is recomputed when the variable is configured. The number is kept exactly as typed. |
| `correct` or `incorrect` | *(empty)* | The telephone is cleared. | — | The probability is recomputed when the variable is configured. The sanitised number becomes empty, which in turn refreshes the potential duplicate count. |
| `correct` | `incorrect` | A country is written whose code makes the stored number unparsable. | The telephone is not empty. | Recomputed with no user action on the telephone itself. |
| `incorrect` | `correct` | A country is written whose code makes the stored number parsable. | The telephone is not empty. | Recomputed with no user action on the telephone itself. |
| any | recomputed from the copied value | A Contact is linked whose telephone is copied onto the lead by the synchronisation rule. | The lead had no telephone. | The quality follows the copied value inside the same write. |
| any | unchanged on the survivor | A merge. | — | The surviving record keeps its own telephone and therefore its own quality; the merged-away records are deleted and their contribution to the frequency table is not undone. |

### 10.3 Diagram

```mermaid
stateDiagram-v2
    [*] --> NoTelephone: created without a telephone
    [*] --> Correct: created with a telephone that parses
    [*] --> Incorrect: created with a telephone that does not parse
    NoTelephone --> Correct: a telephone that parses is written
    NoTelephone --> Incorrect: a telephone that does not parse is written
    Correct --> NoTelephone: telephone cleared
    Incorrect --> NoTelephone: telephone cleared
    Correct --> Incorrect: the country changes and the number no longer parses
    Incorrect --> Correct: the country changes and the number now parses
```

---

## 11. Electronic mail quality — the `email_state` field

### 11.1 States

| Value | Label | Meaning |
|---|---|---|
| *(empty)* | *(no label)* | The record carries no electronic mail address. The empty value is an observation for the predictive probability and is recorded as the text `False`. |
| `correct` | Correct | At least one of the addresses contained in the field passes format validation. |
| `incorrect` | Incorrect | The field holds one or more addresses and none of them passes format validation. |

Computed and stored from `email_from` (electronic mail address); never written by hand.

### 11.2 Transitions

| From | To | Trigger | Guards | Side effects |
|---|---|---|---|---|
| *(none)* | *(empty)* | Creation with no address. | — | None. |
| *(none)* | `correct` | Creation with at least one valid address. | — | The value is stored at creation. |
| *(none)* | `incorrect` | Creation with addresses of which none is valid. | — | The value is stored at creation. The record is then not offered the manual enrichment control. |
| *(empty)* | `correct` | An address with at least one valid form is written. | — | The normalised address and the electronic mail domain criterion are recomputed, which refreshes the potential duplicate count. |
| *(empty)* | `incorrect` | An address is written whose forms are all invalid. | — | The same recomputations; the criterion may become empty. |
| `correct` or `incorrect` | *(empty)* | The address is cleared. | — | The probability is recomputed when the electronic mail quality is one of the configured scoring variables. |
| `incorrect` | `correct` | A field holding several addresses is written of which one is valid. | — | One valid address is enough; the remaining invalid ones do not matter. |
| `correct` | `incorrect` | A field holding several addresses is written of which none is valid. | — | The whole field is judged, not only its first address. |
| any | recomputed from the copied value | A Contact is linked whose address is copied onto the lead. | The lead had no address. | The quality follows the copied value inside the same write. |

### 11.3 Diagram

```mermaid
stateDiagram-v2
    [*] --> NoAddress: created without an address
    [*] --> Correct: created with at least one valid address
    [*] --> Incorrect: created with no valid address
    NoAddress --> Correct: a valid address is written
    NoAddress --> Incorrect: an invalid address is written
    Correct --> NoAddress: address cleared
    Incorrect --> NoAddress: address cleared
    Incorrect --> Correct: one of the addresses becomes valid
    Correct --> Incorrect: no address is valid any more
```

---

## 12. Reveal View status — the `reveal_state` field

A Reveal View (`crm.reveal.view`, table `crm_reveal_view`) records one visit of one network address
to a page matched by a Lead Generation Rule, waiting to be resolved into a company.

### 12.1 States

| Value | Label | Meaning |
|---|---|---|
| `to_process` | To Process | The visit has been recorded and the identification service has not answered for it yet. This is the default. |
| `not_found` | Not Found | The service answered and could not resolve the address into a company. |

### 12.2 Transitions

| From | To | Trigger | Guards | Side effects |
|---|---|---|---|---|
| *(none)* | `to_process` | A page is served, an active rule matches the website, the path pattern, the visitor's country and the visitor's state, and no row yet exists for that pair of rule and address. | The unique index on the pair of rule and address is what enforces "no row yet exists". | One row is created. |
| `to_process` | *(deleted)* | The scheduled job resolves the address into a company. | — | One Lead is created with the values of the rule and of the company, carrying the network address, the credits consumed and the rule. The row is then deleted. |
| `to_process` | `not_found` | The scheduled job runs and the service cannot resolve the address. | — | No Lead is created. |
| `to_process` or `not_found` | *(deleted)* | The scheduled job runs and a Lead already exists for the same address inside the retention window. | — | The row is deleted before it is processed, so the same visitor is not bought twice. |
| `to_process` or `not_found` | *(deleted)* | The cleanup. | The row is older than one month. | The row is deleted. |

### 12.3 Diagram

```mermaid
stateDiagram-v2
    [*] --> ToProcess: a matching page is served
    ToProcess --> [*]: resolved into a company, a lead is created, the row is deleted
    ToProcess --> NotFound: the service cannot resolve the address
    ToProcess --> [*]: a lead already exists for the address within the retention window
    NotFound --> [*]: cleaned up after one month
```

---

## 13. Reconciliation notes

| Subject | The two statements | Resolution |
|---|---|---|
| Where the state machines live | One version gathered them in this file; the other placed them at the head of the workflow document. | They are gathered here. `workflows.md` refers to this file and does not repeat the tables. |
| Refused transitions of the outcome machine | One version listed only the transitions that happen; the other listed the refusals as rows of the same table. | The refusals are kept, in their own section 3.6, so that the transition table stays a table of things that happen. |
| The quality fields | One version treated them as ordinary computed fields; the other specified them as state machines. | They are specified as state machines here, in sections 10 and 11, because their values are a contractual enumeration, they are observed by the predictive probability, and one of them gates the manual enrichment control. |
| The stored value of an absent quality | Both versions agreed that the absence is itself an observation. | Recorded once: an empty quality is written into the frequency table as the text `False`. |
