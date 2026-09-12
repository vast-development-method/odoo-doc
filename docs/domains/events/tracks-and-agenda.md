# Tracks and agenda

The programme of an event: how a talk is proposed, reviewed, scheduled, published, watched, remembered, quizzed and ranked. The field-by-field definition of every entity named here is in [entities.md](entities.md); this file carries the behaviour, the derivations that were only summarised there, and the complete algorithms of the agenda, the suggestions, the wish list and the quiz.

## Contents

1. [The programme model](#1-the-programme-model)
2. [Review stages](#2-review-stages)
3. [Speaker and contact information](#3-speaker-and-contact-information)
4. [Scheduling a talk](#4-scheduling-a-talk)
5. [The public talk list](#5-the-public-talk-list)
6. [The agenda grid](#6-the-agenda-grid)
7. [The talk page](#7-the-talk-page)
8. [Wish list and reminders](#8-wish-list-and-reminders)
9. [Live video](#9-live-video)
10. [Talk suggestions](#10-talk-suggestions)
11. [Quizzes](#11-quizzes)
12. [Leaderboard](#12-leaderboard)
13. [Rooms, tags and tag categories](#13-rooms-tags-and-tag-categories)
14. [The event application on a mobile device](#14-the-event-application-on-a-mobile-device)

---

## 1. The programme model

| Entity | Role |
|---|---|
| Event Track | one talk: title, abstract, speaker, room, date, duration, tags, stage, publication, picture, video, action button, quiz. |
| Event Track Stage | one column of the review pipeline, carrying the three flags that drive agenda visibility and publication. |
| Event Track Location | one room or stage, with a sequence that orders the columns of the agenda. |
| Event Track Tag and Event Track Tag Category | the labels used by the public filters, grouped in categories. |
| Track / Visitor Link | the link between one talk and one site visitor: wish list, opt-out, quiz completion and quiz points. |
| Quiz, Content Quiz Question, Question's Answer | the quiz attached to a talk. |

An event switches the programme on with two flags: `website_track` shows the talk pages and the agenda, `website_track_proposal` shows the proposal form. The second is forced on whenever the first is switched on, and forced off whenever the first is switched off; the two can then be set independently.

The number of talks shown on an event counts only talks whose stage is **not** a cancelling stage. The tag cloud of an event, used by the public filters, contains every tag used by its talks whose colour index is not zero.

---

## 2. Review stages

A stage carries three independent flags plus a folding flag and an optional message template.

| Flag | Meaning |
|---|---|
| `is_cancel` | the stage means "refused" or "cancelled" |
| `is_visible_in_agenda` | talks in this stage appear in the public agenda and the public talk list even when they are not published |
| `is_fully_accessible` | talks entering this stage are published, which gives the speaker and the public a working link |

The flags are kept coherent automatically, forming the ladder of `EV-RULE-080`:

```formula
is_cancel = true            ⇒ is_visible_in_agenda = false and is_fully_accessible = false
is_fully_accessible = true  ⇒ is_visible_in_agenda = true
is_visible_in_agenda = false⇒ is_fully_accessible = false
```

**Entering a stage.** When a talk is written into a stage:

1. the kanban state is reset to `normal` unless a new kanban state is written in the same operation;
2. the talk is published when the stage is fully accessible, and unpublished when the stage is a cancelling stage; a stage that is neither leaves the publication flag untouched;
3. when the stage carries a message template, that template is sent to the speaker as an **internal note**, using the light notification layout, and the sent message is not kept as an attachment log.

**Kanban legends.** Each stage carries three legends, shown next to the kanban state of a talk: a red legend (default `Blocked`), a green legend (default `Ready for Next Stage`) and a grey legend (default `In Progress`). The talk stores the legend matching its current kanban state in a tracked field, so that the discussion thread records the transition in words rather than in codes.

**Thread subtypes.** Setting the kanban state to `blocked` posts under the subtype "Track Blocked"; setting it to `done` posts under the subtype "Track Ready", which is subscribed by default.

---

## 3. Speaker and contact information

A talk carries two contact blocks that behave differently.

**The speaker block** (`partner_name`, `partner_email`, `partner_phone`, `partner_biography`, `partner_function`, `partner_company_name`, `image`) is filled from the chosen contact **only while empty**. A value typed by the organiser or posted by the speaker is never overwritten. The company name has its own rule: when the contact is itself a company its own name is used, otherwise, while empty, the name of its parent company is used. The picture is taken from the 256-pixel image of the contact while empty.

**The operational block** (`contact_email`, `contact_phone`) is **overwritten** by the contact whenever a contact is set. It is the address the organiser uses to reach the speaker, and it is the primary address of the talk for the incoming-message gateway.

### Speaker tag line

The one-line speaker description is derived as follows:

```formula
if partner_name is empty:            tag_line = empty
else if partner_function is set:
        if partner_company_name is set:  tag_line = "<name>, <function> at <company>"
        else:                            tag_line = "<name>, <function>"
else if partner_company_name is set:     tag_line = "<name> from <company>"
else:                                    tag_line = "<name>"
```

**Worked examples.**

| Name | Function | Company | Tag line |
|---|---|---|---|
| Ada Lovelace | Chief Analyst | Analytical Engines | `Ada Lovelace, Chief Analyst at Analytical Engines` |
| Ada Lovelace | Chief Analyst | — | `Ada Lovelace, Chief Analyst` |
| Ada Lovelace | — | Analytical Engines | `Ada Lovelace from Analytical Engines` |
| Ada Lovelace | — | — | `Ada Lovelace` |
| — | Chief Analyst | Analytical Engines | empty |

### Default recipients and contact creation

When a message is sent from the talk and the talk has no contact with a usable address, the recipient list falls back to the speaker address. When the organiser posts a message from the talk and creates a contact through the suggested-recipient mechanism, and the talk still has no contact, the created contact is matched against the operational address, or failing that the speaker address, and then written on **every** talk that has no contact, carries that address and is not in a cancelling stage.

---

## 4. Scheduling a talk

### Start, end and duration

The three values form a triangle; writing any one of them recomputes the third:

```formula
date      = date_end − duration hours
date_end  = date + duration hours
duration  = (date_end − date) in hours     when either datetime is written directly
```

The default duration is `0.5` hours. A talk may have no date at all, in which case it is shown in the public list under the heading `Coming soon`, sorted with the key talks first.

### Derived time flags

All of them are computed in coordinated universal time, because only differences matter; the full formulas are in [calculations.md](calculations.md#11-talk-times-the-action-button-window-and-the-agenda-grid).

| Flag | Meaning |
|---|---|
| `is_track_live` | the talk is running now |
| `is_track_soon` | the talk starts in less than thirty minutes |
| `is_track_today` | the talk starts on the current calendar day |
| `is_track_upcoming` | the talk has not started |
| `is_track_done` | the talk has ended |
| `is_one_day` | the start and the end fall on the same day in the event display time zone |
| `track_start_remaining` | seconds before the start, zero once started |
| `track_start_relative` | seconds to the start when upcoming, seconds since the start otherwise |

A talk with neither a start nor an end has all five booleans false and both counters zero.

### The action button

An organiser may show a button while the talk plays: a title, a target address and a delay in minutes after the start. The target address is cleaned into a canonical web address on creation and on every write. The button is live between `date + delay` and `date_end`, and the page shows a countdown of `website_cta_start_remaining` seconds until then.

---

## 5. The public talk list

**Which talks are shown.** The base condition is:

```formula
event = this event AND ( is_published = true OR stage.is_visible_in_agenda = true )
```

A reader who is not at least a Registration Desk user gets the additional condition `is_published = true`. The organiser therefore previews the announced but unpublished talks while the public sees only the published ones.

**Search and filters.**

- Free text matches the talk title or the speaker name.
- Tags are grouped by category: inside a category the selected tags are combined with "or", and the categories are combined with "and". Selecting "age 10-12" and "football" returns talks tagged both; adding "age 12-15" returns talks tagged football and tagged with either age.
- A wish-list filter keeps only the talks for which the reader has a reminder on. It is applied after the search, because the reminder state cannot be expressed as a database condition.
- More than one tag passed through a plain page request is permanently redirected to the plain talk list, to stop crawlers from exploring every combination.

**Grouping.** Talks are read ordered by publication descending then by start date ascending, then arranged as:

1. `tracks_live`: the talks running now;
2. `tracks_soon`: the talks starting in less than thirty minutes and not yet running;
3. one group per calendar day, in the display time zone, ordered ascending, each holding the talks starting on that day;
4. a final group named `Coming soon` holding the talks with no date, ordered with the key talks first.

A day group is collapsed by default when **all** of its talks are finished **and** the event still has at least one talk that is not finished.

---

## 6. The agenda grid

The agenda is a table of quarter-hour rows by room, one table per day.

**Candidates.** Every talk of the event satisfying the base condition of the talk list **and** having a start date. The publication restriction for non-organisers applies the same way.

**Rooms.** The distinct rooms of the candidate talks, ordered by room sequence then identifier. A talk without a room is kept and occupies **every** room column of the rows it spans, which is how a plenary session is drawn across the whole table.

**Algorithm.**

1. For each talk, convert the start into the display time zone and round it **down** to the previous quarter hour:

```formula
rounded = start with seconds and sub-seconds cleared,
          and minutes replaced by 15 × ⌊minutes ÷ 15⌋
```

2. Compute the rounded end as `rounded start + duration hours`, rounded down the same way; a missing duration counts as `0.25` hours.
3. The number of rows the talk spans is `((rounded end − rounded start) in hours) × 4`.
4. Split the span per calendar day: rows falling on the next day open a new bucket for that day.
5. Collect the days from the calendar dates of the rounded starts, sorted ascending.
6. For each day, the table runs from the earliest rounded start of that day to the latest rounded end of that day, in quarter-hour rows. Each row carries, per room, the talks that start on that row, with:
   - the number of rows they span;
   - their **real** start and end times, formatted in the short time format of the reader language, not the rounded ones;
   - the list of `(row, room)` cells they occupy, so that no empty cell is drawn over a talk.
7. Per day, the number of talks and the rooms actually used are counted, again ordered by sequence then identifier.

**Worked example.** Display time zone `Europe/Brussels`, one day with three talks:

| Talk | Real start | Duration | Room | Rounded start | Rounded end | Rows |
|---|---|---|---|---|---|---|
| Opening | 09:00 | 0.5 | none | 09:00 | 09:30 | 2 |
| Deep dive | 10:07 | 1.5 | Main Hall | 10:00 | 11:30 | 6 |
| Workshop | 10:20 | 0.5 | Room B | 10:15 | 10:45 | 2 |

The day runs from `09:00` to `11:30`, that is ten rows of fifteen minutes. `Opening` has no room and therefore occupies both room columns on the rows `09:00` and `09:15`. `Deep dive` occupies the Main Hall column on the rows `10:00` through `11:15` and displays `10:07 AM - 11:37 AM`. `Workshop` occupies the Room B column on the rows `10:15` and `10:30`.

---

## 7. The talk page

The page is reachable at `/event/<event slug>/track/<talk slug>` and requires the event to have its talk menu on. A reader who cannot read the talk is refused; a talk whose event cannot be reached from the current website gives "not found".

The page shows the abstract, the speaker block, the picture, the video when one is set, the action button when it is live, the quiz when one is attached, the reminder control and a sidebar of at most ten suggested talks. Two display options are honoured: a wide layout, which the live package turns on automatically when the talk carries a video and is a replay, is starting soon, is live or is finished; and an editing hint for an Event User.

### Talk picture address

```formula
if the talk has a website picture:      the 1024-pixel rendering of that picture
else if the talk has a video code:      the maximum-resolution thumbnail of that video,
                                        taken from the video service
else:                                   one of the two shipped default pictures,
                                        chosen by the parity of the talk identifier
```

The alternation between two default pictures keeps a list of talks from looking repetitive.

---

## 8. Wish list and reminders

A reminder is stored on the Track / Visitor Link link, which carries two booleans that must be read together:

| Field | Meaning |
|---|---|
| `is_wishlisted` | the visitor explicitly asked for a reminder on an ordinary talk |
| `is_blacklisted` | the visitor explicitly removed the reminder from a **key** talk (`wishlisted_by_default`) |

A key talk is on for everybody by default, therefore opting out needs its own flag; an ordinary talk is off by default, therefore opting in needs its own flag.

### Reminder state

The reminder state of a talk **for the current reader** is derived as follows:

1. Read the current site visitor from the request.
2. When the reader is anonymous and there is no visitor at all, the state is simply `wishlisted_by_default`.
3. Otherwise build the identity condition:
   - anonymous reader with a visitor: `visitor_id = that visitor`;
   - signed-in reader with a visitor: `partner_id = the contact of the reader OR visitor_id = that visitor`;
   - signed-in reader without a visitor: `partner_id = the contact of the reader`.
4. Read the links matching that condition for the talks being computed, with elevated rights.
5. For each talk:

```formula
if a link exists:   is_reminder_on = link.is_wishlisted
                                     OR (wishlisted_by_default AND NOT link.is_blacklisted)
else:               is_reminder_on = wishlisted_by_default
```

**Worked example.** A talk is a key talk. A visitor who never touched it has `is_reminder_on = true`. The visitor switches the reminder off: the link is created with `is_blacklisted = true` and the state becomes false. The visitor switches it on again: `is_blacklisted` becomes false and the state returns to true. An ordinary talk behaves the mirror way through `is_wishlisted`.

### Toggle procedure

1. Fetch the talk. A reader who cannot read it is served with elevated rights only for this operation; a talk whose event is not reachable from the current website gives "not found".
2. Force-create the link when the visitor is switching the reminder **on**, or when the talk is a key talk. For an anonymous reader a site visitor record is created; the last-visit stamp of the visitor is refreshed.
3. **Ordinary talk:** answer `ignored` when no link exists, or when `is_wishlisted` already equals the requested state; otherwise write `is_wishlisted`.
4. **Key talk:** answer `ignored` when no link exists, or when `is_blacklisted` differs from the requested state; otherwise write `is_blacklisted = NOT requested state`.
5. Answer the new reminder state.

### Counters and searches

`wishlist_visitor_ids` and `wishlist_visitor_count` are read with elevated rights from the links whose `is_wishlisted` is true, and are visible to the Event User group. A "not in" search on them is refused (`EV-RULE-088`). The same pair exists on the visitor side: `event_track_wishlisted_ids` and `event_track_wishlisted_count`.

**Retention.** A visitor with at least one talk link is never purged by the inactive-visitor housekeeping. Merging two visitors moves the links to the surviving visitor and fills the contact of the links that had none.

### Reminder by electronic mail

The visitor may ask for the calendar links of a talk to be sent to an address. The refusals, in order, are the four messages of `EV-RULE-085`. The message is rendered in the language of the visitor session for an anonymous reader, and in the language of the user otherwise. The calendar dates carried by that message are the talk dates when the talk has them, and the **event** dates otherwise; in the second case the message carries the warning of the calendar file described in [interfaces.md](interfaces.md#4-exported-files-and-outgoing-links), so that the visitor knows to check again later.

---

## 9. Live video

A talk may carry a video link. The eleven-character video code is extracted from the link with a pattern that accepts the short form, the direct form, the user form, the embedded form, the live form and the watch form; a link that does not match leaves the code empty.

| Field | Rule |
|---|---|
| `is_youtube_replay` | set by the organiser; marks the video as a recording, which hides the live-only elements such as the chat |
| `is_youtube_chat_available` | true when a video link exists, the video is not a replay, and the talk is either live or starting soon |
| picture | when the talk has no picture of its own but has a video code, the maximum-resolution thumbnail of the video is used |
| wide layout | turned on automatically when the talk carries a video and is a replay, is starting soon, is live, or is finished |
| chat on a small screen | the chat is considered disabled when the requesting device identifies itself as a mobile device of the three common families |

---

## 10. Talk suggestions

At the end of a talk the page proposes what to watch next. The ranking is the sorted tuple of [calculations.md](calculations.md#17-talk-suggestion-ranking), applied to every other talk of the same event, restricted by the visibility condition of the talk list, and by the presence of a video link when the suggestion is requested by the live player.

The tuple, sorted descending so that a true value or a larger number comes first:

1. published;
2. "started less than ten minutes ago and still running";
3. "still to come";
4. the negated remaining time, so that the soonest comes first among the upcoming ones;
5. the reader has a reminder on it;
6. it is **not** a key talk, so that an ordinary talk wins a tie against a key talk of equal rank;
7. the number of tags shared with the current talk;
8. the room is the same as the room of the current talk;
9. a pseudo-random integer between 0 and 20.

The random component makes equally ranked talks rotate between readers. The live player asks for exactly one suggestion and shows the name and picture of the current talk next to the name, the speaker and the address of the suggestion; with the quiz bridge it also says whether the quiz of the **current** talk should be offered before moving on, which is true when the talk has a quiz that the reader has not completed.

---

## 11. Quizzes

### Structure

A quiz belongs to exactly one talk, has a name, an unlimited-tries flag and a list of questions. Each question has a name, a sequence and a list of answers. Each answer has a text, a sequence, a correctness flag, an optional explanatory comment and a number of points.

Two derivations help the organiser:

```formula
question.correct_answer  = the answers of the question whose correctness flag is true
question.awarded_points  = Σ over every answer of the question of its points
```

The integrity rule (`EV-RULE-082`) requires exactly one correct answer and at least two answers in total. Note that **points and correctness are independent**: an organiser may award points to a partially right answer.

### Taking a quiz

The submission procedure is in [workflows.md](workflows.md#27-answer-a-quiz) and the scoring in [calculations.md](calculations.md#13-quiz-scoring-and-the-leaderboard). The two gates are `track_quiz_done` for a second attempt and `quiz_incomplete` when the submitted answers do not cover every question exactly once.

The answer returned to the reader carries, per question: the points of the chosen answer, the text of the correct answer, whether the chosen answer was correct and the explanatory comment of the chosen answer. The reader therefore learns the right answer immediately.

### Reset

Resetting sets `quiz_completed` to false and `quiz_points` to zero on the visitor link. It is allowed when the quiz has unlimited tries, and always allowed to an Event Administrator so that a quiz can be tested.

### Per-reader figures on a talk

`is_quiz_completed` and `quiz_points` are computed for the current reader using the same identity resolution as the reminder state: the visitor of the request, the contact of the reader, or both. A talk without a quiz reports false and zero. An anonymous reader with no visitor at all reports false and zero.

### The community menu

The quiz package makes the community menu of an event follow the website menu: switching the website menu on switches the community menu on, and switching it off switches it off. Without the quiz package the community menu is forced off, and the community page answers with the page-not-found view.

---

## 12. Leaderboard

The leaderboard ranks the site visitors of one event by the points they collected across the talks of that event.

1. Sum `quiz_points` per visitor over the links whose talk belongs to the event, keeping only visitors with a visitor record and a strictly positive sum.
2. Order by the sum **descending**, then by the visitor identifier **ascending**; the identifier tie-break makes the ranking stable and favours the visitor who arrived first.
3. Assign positions `1, 2, 3, …` walking the ordered list. The counter advances on **every** row, including rows hidden by a name search, therefore a searched row keeps its true rank.
4. A name search keeps only the visitors whose display name contains the searched text, compared without case.
5. The top three of the full ranking are returned separately for the podium.
6. Pagination: 30 visitors per page, at most 5 page links. When the reader is ranked and no page was requested, the page containing the reader is opened, computed as `⌈position ÷ 30⌉`, and the page scrolls to that row.

**Worked example.** Twelve visitors are ranked; the reader is at position 34 of a ranking of 95. Opening the leaderboard without a page opens page `⌈34 ÷ 30⌉ = 2` and scrolls to the reader. The pager shows at most 5 of the 4 pages, that is all of them.

The display name of an anonymous visitor is the attendee name of its latest registration, which is why an anonymous quiz taker who registered to the event appears under their real name rather than as an anonymous entry.

---

## 13. Rooms, tags and tag categories

**Rooms** carry only a name and a sequence. The sequence orders the columns of the agenda. A room is referenced by talks and by the location line of the calendar files.

**Tags** carry a name, a sequence, a colour index and an optional category. Two rules govern them:

- the name is unique across the whole system (`EV-RULE-081`);
- a colour index of zero or empty hides the tag from the public site entirely, which is how internal-only labels are kept out of the filters.

The default colour of a new tag is a pseudo-random value between 1 and 11 inclusive.

**Tag categories** carry a name, a sequence and their tags. They group the filter blocks of the public talk list and give the "or inside a category, and between categories" semantics described in section 5.

An event may restrict the tags offered on its proposal form through its list of allowed talk tags; its tag cloud, used by the public filters, is derived from the tags actually used by its talks, excluding colourless ones.

---

## 14. The event application on a mobile device

The programme package turns the event site into an installable application.

| Piece | Rule |
|---|---|
| Application name | the `events_app_name` of the website, defaulting to `<website name> Events`, required (`EV-RULE-090`) |
| Application icon | derived from the site icon: the icon is cropped to a square on its larger side, then resized to 512 by 512 and re-encoded as a lossless raster image. A site icon in a vector format is skipped and no application icon is produced. |
| Manifest | served at `/event/manifest.webmanifest`, carrying the application name and the icon |
| Background worker | served at `/event/service-worker.js`, scoped to the event pages |
| Offline page | served at `/event/offline`, shown when the event site is opened without a connection |

---

## Reconciliation notes

1. **Provenance.** This topic file comes from version M. Version P covered the same ground inside its
   capability table — sessions with speakers, stages, live broadcasting, quizzes, wish lists and the
   community page — and every one of those subjects is specified here.
2. **Vocabulary.** Version P called a programme item a session; this folder calls it a talk, because
   the entity is Event Track. The synonym is recorded in [`glossary.md`](glossary.md).
3. **Identifiers.** The action-button fields are named `website_cta`, `website_cta_title`,
   `website_cta_url`, `website_cta_delay`, `is_website_cta_live` and `website_cta_start_remaining`,
   which is what the database carries; version M had renamed them. Their full names are in
   [`entities.md`](entities.md).
