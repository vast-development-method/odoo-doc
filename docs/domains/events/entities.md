# Events — Entities

Every entity of this domain, field by field: its purpose, its lifecycle, its complete field
table, its relations, its uniqueness rules, its defaults, its derived values with the rules that
produce them, its ordering, its display-name rule, its archival behaviour, its company behaviour
and the extension points other capability packages contribute to it. Validation rules are quoted
with the exact text the system shows.

## Conventions used in this file

Field tables carry these columns:

- **Field** — the storage name, reproduced exactly, in code font. Storage names are contractual:
  a replacement that must import an existing database or serve an existing integration depends on
  them character for character.
- **Full name** — the label the interface shows for that field, in words.
- **Type** — one of `boolean`, `integer`, `decimal`, `monetary`, `text`, `long_text`, `rich_text`,
  `date`, `datetime`, `binary`, `image`, `selection`, `reference`, `many_to_one`, `one_to_many`,
  `many_to_many`, `structured_data`, `properties`, followed by the entity a relation points at.
  Every `datetime` is stored in coordinated universal time and displayed in a time zone stated per
  field.
- **Required**, **Default** — whether a value must be present, and what is proposed when none is
  supplied.
- **Stored** — **stored** means the value lives in the database; **derived** means it is
  recomputed from its inputs. A derived field marked *derived, stored, editable* is recomputed when
  its inputs change but can be overwritten by hand, and the manual value then survives until an
  input changes again. *Precomputed* means the value is produced before the row is inserted, so
  that a record created through an integration always carries it.
- **Copied** — whether the value is carried over when the record is duplicated.
- **Tracked** — whether a change to the field is written into the discussion thread of the record.
- **Meaning and rules** — everything else: indexing, deletion behaviour of a relation, the
  selection values with their labels, the group that may see the field, and the business meaning.

Unless a table states otherwise, a field is optional, writable, stored, copied on duplication and
not indexed.

Every persistent entity also carries the five audit fields `id` (the surrogate integer primary
key, called the identifier in prose), `create_date` (creation instant), `create_uid` (creating
user), `write_date` (last change instant) and `write_uid` (last changing user). They are not
repeated in the per-entity tables. Entities that support archiving carry `active` (boolean,
default true; archived records are excluded from default queries); `active` is listed only where
its meaning goes beyond hiding the record.

Entities marked "carries a discussion thread" also carry the standard thread fields (message list,
follower list, unread counters) and the standard activity fields (planned activity list, next
activity date, next activity summary, next activity responsible). Those belong to the
[Messaging and activities](../messaging-and-activities/README.md) domain and are not repeated
here.

Entities marked "publishable" carry `is_published` (boolean), `website_published` (boolean alias
of the same state), `website_url` (derived web address of the public page) and the search-engine
metadata fields. Those belong to the
[Website and storefront](../website-and-storefront/README.md) domain.

A message reproduced between backticks in this folder is shown to the user verbatim; a placeholder
written between angle brackets is replaced by the value named inside it.

---

# 1. Event

**Full name:** Event. **Transport name** `event.event`, **storage name** `event_event`. **Kind:** persistent entity. **Contributed by the capability package** `event`. Generated reference page: [`event.event`](../../references/entities/event.event.md). **Carries a discussion thread.** **Publishable.** **Archivable.**

The central record of the domain. It holds the dates, the place, the seat policy, the ticket catalogue, the communication schedule, the question set, the booth catalogue, the programme and the website configuration.

**Default ordering:** by `date_begin` ascending, then `identifier` ascending.

**Display name rule:** normally the `name`. When the caller asks for availability in the name, the display name becomes:

```formula
if event_registrations_sold_out:        "<name> (Sold out)"
else if seats_limited AND seats_max: "<name> (<seats_available> seats remaining)"
else:                                    "<name>"
```

The seat count in that text is formatted with zero decimal places using the reader's language.

**Company scoping:** `company_id` may be empty. The record rule named "Event: multi-company" restricts visibility to `company_id IN (companies of the reader) OR company_id IS NULL`.

**Duplication:** the copy is named `"<name> (copy)"`. The ticket list, the booth list, the communication list, the slot list and the multi-slot flag are copied; the stage, the kanban state, the website menu and the linked sales information are not. After the copy is written, the whole website menu tree of the source event is duplicated for the new event (see section 27).

## 1.1 Identification, ownership and classification

| Field | Full name | Type | Required | Default | Stored | Copied | Tracked | Meaning and rules |
|---|---|---|---|---|---|---|---|---|
| `name` | Event | text (translatable) | yes | none | stored | yes (with `" (copy)"` appended) | no | The event title. |
| `subtitle` | Event Subtitle | text (translatable) | no | none | stored | yes | no | Short tag line shown under the title on the public page and used as the default page description and social-preview description. |
| `note` | Note | rich_text | no | none | derived, stored, editable | yes | no | Internal note. Recomputed when `event_type_id` changes: if the template has a non-empty note, the note of the template replaces it; otherwise the current value is kept. |
| `description` | Description | rich_text (translatable) | no | rendered from the shipped default description layout | stored | yes | no | Long public description. Rich-text attributes and embedded forms are preserved as entered. |
| `active` | Active | boolean | yes | true | stored | yes | no | Archived events are hidden from default queries and their communication schedules are skipped. |
| `user_id` | Responsible | many_to_one to User | no | the user creating the record | stored | yes | yes | The person responsible for the event. Deletion behavior: restrict (the platform default for a user link). |
| `company_id` | Company | many_to_one to Company | no | the active company of the creating user | stored | yes | no | Owning company. Drives the currency and the multi-company record rule. |
| `organizer_id` | Organizer | many_to_one to Contact | no | the contact of the active company | stored | yes | yes | The organising party. Used as the sender of every automatic communication and printed in the ticket footer. Must belong to the event company or to no company. |
| `event_type_id` | Template | many_to_one to Event Template | no | none | stored | yes | no | The template whose configuration is applied. Deletion behavior: set to empty. |
| `tag_ids` | Tags | many_to_many to Event Tag | no | none | derived, stored, editable | yes | no | Classification labels. Recomputed when `event_type_id` changes: if the event has no tag yet and the template has tags, the template tags are copied. |
| `stage_id` | Stage | many_to_one to Event Stage | no | the first stage by sequence | stored | no | yes | Pipeline column. Deletion behavior: restrict; a stage that is used cannot be deleted. Grouping by stage always shows every stage, including empty ones. |
| `kanban_state` | Kanban State | selection | no | `normal` | derived, stored, editable | no | yes | Progress marker. Values: `normal` (In Progress), `done` (Ready for Next Stage), `blocked` (Blocked), `cancel` (Cancelled). Recomputed when `stage_id` changes: any state other than `cancel` is reset to `normal`; a cancelled event stays cancelled when it moves stage. |
| `registration_properties_definition` | Registration Properties | properties definition | no | none | stored | yes | no | The definition of the user-defined fields that every Registration of this event will carry. |
| `lang` | Language | selection of installed languages | no | none | stored | yes | no | Forces the language of every automatic communication sent to attendees. |

## 1.2 Dates and time

| Field | Full name | Type | Required | Default | Stored | Copied | Tracked | Meaning and rules |
|---|---|---|---|---|---|---|---|---|
| `date_begin` | Start Date | datetime | yes | the current moment rounded up to the next half hour (seconds and sub-seconds cleared) | stored | yes | yes | Start of the event, entered and displayed in the reader's own time zone on the back-office form. |
| `date_end` | End Date | datetime | yes | `date_begin` plus one day | stored | yes | yes | End of the event. |
| `date_tz` | Display Timezone | selection of time zone names | yes | see derivation | derived, stored, editable, precomputed | yes | no | The time zone in which the event dates are displayed publicly, in printed tickets, in communications and in the agenda. Recomputed when `event_type_id` changes: if the template defines a default time zone, that value is taken; if the field is still empty, the time zone of the current user is taken, and finally coordinated universal time. |
| `is_ongoing` | Is Ongoing | boolean | — | — | derived, not stored, searchable | — | — | True when `date_begin <= now < date_end`. Searching on it rewrites the condition into `date_begin <= now AND date_end > now`. Only the "is in" form of the search is supported. |
| `is_finished` | Is Finished | boolean | — | — | derived, not stored, searchable | — | — | True when the end date, expressed in `date_tz`, is at or before the current moment expressed in the same zone. Searching on it rewrites into `date_end <= now`. |
| `is_one_day` | Is One Day | boolean | — | — | derived, not stored | — | — | True when `date_begin` and `date_end`, converted into `date_tz`, fall on the same calendar day. |
| `is_done` | Is Done | boolean | — | — | derived, not stored | — | — | True when the current moment is strictly after `date_end` (compared in coordinated universal time). Used by the public pages. |
| `start_today` | Start Today | boolean | — | — | derived, not stored | — | — | True when `date_begin` falls on the current calendar day, compared in coordinated universal time. |
| `start_remaining` | Remaining before start | integer (minutes) | — | — | derived, not stored | — | — | Whole minutes left before `date_begin`; zero once the event has started. |

**Constraint `_check_closing_date`** on `date_begin` and `date_end`: if `date_end < date_begin`, the save is rejected with *"The closing date cannot be earlier than the beginning date."*

## 1.3 Place and links

| Field | Full name | Type | Required | Default | Stored | Copied | Tracked | Meaning and rules |
|---|---|---|---|---|---|---|---|---|
| `address_id` | Venue | many_to_one to Contact | no | the contact of the active company | stored | yes | yes | The venue. Must belong to the event company or to no company. An empty venue marks the event as online. |
| `address_name` | Address Name | text | — | — | derived from `address.name`, not stored | — | — | The venue name, exposed for the site search index. |
| `address_search` | Address | many_to_one to Contact | — | — | derived, not stored, searchable | — | — | A search helper equal to `address_id`. A text search on it matches any of the venue `name`, `street`, `street2`, `city`, `zip`, `state_id` and `country_id`. An empty-value search returns nothing, because an event without a venue has no address to match. |
| `address_inline` | Venue (formatted for one line uses) | text | — | — | derived, not stored | — | — | The venue address on one line. Rule: take the formatted contact address, drop empty lines, join the remaining lines with `", "`; if the formatted address is blank, use the venue name; if there is no venue, the empty text. Computed with elevated rights, because anonymous visitors must be able to read it. |
| `country_id` | Country | many_to_one to Country | no | — | derived from `address.country`, stored, editable | yes | no | The country of the venue, stored to allow grouping and filtering, and editable for online events. |
| `event_url` | Online Event uniform resource locator | text | no | none | derived, stored, editable | yes | no | The link where an online event takes place. Cleared automatically whenever `address_id` is set, because the field is meant only for events with no physical location. |
| `event_share_url` | Event Share uniform resource locator | text | — | — | derived, not stored | — | — | The link used when sharing the event. It is `event_url` when set; when the website capability package is installed and the field is empty, it falls back to the absolute address of the public event page. |
| `event_register_url` | Event Registration Link | text | — | — | derived, not stored | — | — | The absolute address of the public registration page, that is the event public address followed by `/register`. |

**Constraint `_check_event_url`** on `event_url`: the value must parse into an address that has both a scheme and a host; otherwise the save is rejected with *"Please enter a valid event URL."*

**On-change `_onchange_event_url`**: while the user is typing in the form, if the entered link has no scheme or a scheme other than the two web schemes, `https://` is prefixed to it.

## 1.4 Seats

| Field | Full name | Type | Required | Default | Stored | Copied | Tracked | Meaning and rules |
|---|---|---|---|---|---|---|---|---|
| `seats_limited` | Limit Attendees | boolean | yes | see derivation | derived, stored, editable, precomputed | yes | no | Whether the maximum applies at all. Recomputed when `event_type_id` changes: it takes the template flag "has seats limitation" whenever that flag differs from the current value; a false value stays false. |
| `seats_max` | Maximum Attendees | integer | no | see derivation | derived, stored, editable | yes | no | The maximum number of attendees. When the event uses slots, the maximum applies to **each slot**, not to the whole event. Recomputed when `event_type_id` changes: without a template the current value (or zero) is kept; with a template the template maximum (or zero) is taken. |
| `seats_reserved` | Number of Registrations | integer | — | — | derived, never stored, read-only | — | — | Count of active registrations in state `open`. |
| `seats_used` | Number of Attendees | integer | — | — | derived, never stored, read-only | — | — | Count of active registrations in state `done`. |
| `seats_taken` | Number of Taken Seats | integer | — | — | derived, never stored, read-only | — | — | `seats_reserved + seats_used`. |
| `seats_available` | Available Seats | integer | — | — | derived, never stored, read-only | — | — | Remaining seats. Zero when no maximum applies. See the formula in [calculations.md](calculations.md#1-seat-counters). |
| `registration_ids` | Attendees | one_to_many to Event Registration | — | — | stored on the other side | no | — | Every attendee of the event. |

**On-change `_onchange_seats_max`**: while editing the maximum in the form, if the event limits seats, the new maximum is non-zero, the resulting available seats are zero or less, and (for a multi-slot event) at least one slot exists, the form shows a non-blocking warning titled *"Update the limit of registrations?"* with the body *"There are more registrations than this limit, the event will be sold out and the extra registrations will remain."*

## 1.5 Time slots

| Field | Full name | Type | Required | Default | Stored | Copied | Tracked | Meaning and rules |
|---|---|---|---|---|---|---|---|---|
| `is_multi_slots` | Is Multi Slots | boolean | no | false | stored | yes | no | When true, the event repeats in several time slots. Communications, the seat maximum and the ticket maximum are then applied per slot. |
| `event_slot_ids` | Slots | one_to_many to Event Slot | no | none | stored on the other side | yes | no | The slots of the event. Deletion behavior on the slot side: cascade. |
| `event_slot_count` | Slots Count | integer | — | — | derived, not stored | — | — | Number of slots, obtained by grouping slots per event. |

**Constraint `_check_slots_dates`** on `date_begin`, `date_end`, `event_slot_ids` and `is_multi_slots`: for every multi-slot event, the earliest slot start and the latest slot end must both lie inside `[date_begin, date_end]`. Otherwise the save is rejected with *"These events cannot have slots scheduled outside of their time range:"* followed by one line `- <event name>` per offending event.

## 1.6 Tickets and registration opening

| Field | Full name | Type | Required | Default | Stored | Copied | Tracked | Meaning and rules |
|---|---|---|---|---|---|---|---|---|
| `event_ticket_ids` | Event Ticket | one_to_many to Event Ticket | no | none | stored on the other side, derived, editable, precomputed | yes | no | The ticket catalogue. Recomputed when `event_type_id` changes; see the synchronisation rule below. |
| `start_sale_datetime` | Start sale date | datetime | — | — | derived, not stored | — | — | The earliest ticket sales start. Rule: gather the sales start of every ticket that is **not expired**; if that list is non-empty **and every one of those values is set**, the field is their minimum; otherwise it is empty. An empty value means "sales have always been open". |
| `event_registrations_started` | Registrations started | boolean | — | — | derived, not stored | — | — | True when there is no `start_sale_datetime`, or when the current moment expressed in `date_tz` is at or after that moment expressed in the same zone. |
| `event_registrations_open` | Registration open | boolean | — | — | derived with elevated rights, not stored | — | — | Whether anybody may still take a seat; see the formula in [calculations.md](calculations.md#3-registration-open-and-sold-out). |
| `event_registrations_sold_out` | Sold Out | boolean | — | — | derived with elevated rights, not stored | — | — | Whether no seat is left anywhere; see the same section. |

**Ticket synchronisation from the template.** When `event_type_id` is set or changed:

1. If the event has no template and no ticket, the ticket list is emptied and the rule stops.
2. Otherwise, every existing ticket that has **no registration** is removed.
3. Then, for every template ticket, one new ticket is created copying the whitelisted fields. The whitelist is `sequence`, `name`, `description`, `seats_max`, and, when the product bridge is installed, `product_id` and `price`.

Tickets that already carry registrations are therefore never destroyed by a template change.

## 1.7 Questions

| Field | Full name | Type | Required | Default | Stored | Copied | Tracked | Meaning and rules |
|---|---|---|---|---|---|---|---|---|
| `question_ids` | Questions | many_to_many to Event Question | no | the default questions | derived, stored, editable, precomputed | yes | no | The questions asked on the registration form. |
| `general_question_ids` | General Questions | many_to_many to Event Question | — | — | the same association table, filtered on `once_per_order = true` | — | — | The subset asked once per order. |
| `specific_question_ids` | Specific Questions | many_to_many to Event Question | — | — | the same association table, filtered on `once_per_order = false` | — | — | The subset asked for every attendee. |

**Question synchronisation from the template.** When `event_type_id` is set or changed:

1. Collect the questions of the event that already have at least one stored answer for this event. Those are kept.
2. If the event has no template and nothing must be kept, the list is replaced by the default questions (every active question flagged as default) and the rule stops.
3. If some questions must be kept, every other previously linked question is unlinked; otherwise the whole list is cleared.
4. Every question of the template is then linked.

## 1.8 Printed material

| Field | Full name | Type | Required | Default | Stored | Copied | Tracked | Meaning and rules |
|---|---|---|---|---|---|---|---|---|
| `badge_format` | Badge Dimension | selection | yes | `A6` | stored | yes | no | Badge layout. Values: `A4_french_fold` (A4 foldable), `A6` (A6), `four_per_sheet` (4 per sheet). |
| `badge_image` | Badge Background | image (at most 1024 by 1024) | no | none | stored | yes | no | Background picture printed behind every badge. |
| `ticket_instructions` | Ticket Instructions | rich_text (translatable) | no | see derivation | derived, stored, editable | yes | no | Text printed at the bottom of the full-page ticket. Recomputed when `event_type_id` changes: taken from the template only when the event value is empty and the template value is not. |
| `use_barcode` | Use Barcode | boolean | — | — | derived, not stored | — | — | Whether linear barcodes are printed next to the matrix codes. Reads the global parameter `event.use_event_barcode` with elevated rights; true when that parameter equals the text `True`. |
| `image_1024` | Point-of-sale image | image (at most 1024 by 1024) | no | none | stored | yes | no | Picture shown for the event in the shop-counter product grid. |
| `exhibition_map` | Exhibition Map | image (at most 1024 by 1024) | no | none | stored | yes | no | Floor plan shown on the public booth page. |

## 1.9 Website publication and menus

| Field | Full name | Type | Required | Default | Stored | Copied | Tracked | Meaning and rules |
|---|---|---|---|---|---|---|---|---|
| `website_published` | Website Published | boolean | no | false | stored | yes | yes | Whether the event page is public. Publishing posts the subtype "Event published" in the thread; unpublishing posts "Event unpublished". |
| `website_visibility` | Website Visibility | selection | yes | `public` | stored | yes | yes | Values: `public` (Public), `link` (Via a Link), `logged_users` (Logged Users). Controls listing and searching only; the event page itself is always reachable through its own link. |
| `is_visible_on_website` | Visible On Website | boolean | — | — | derived per reader, not stored, searchable | — | — | True when visibility is `public`, or the reader is participating, or the reader is signed in and visibility is `logged_users`. The search form returns `is_participating = true OR website_visibility IN (allowed values for this reader)`. |
| `is_participating` | Is Participating | boolean | — | — | derived per reader, not stored, searchable | — | — | True when the reader has a registration in state `open` or `done` on the event. See the heuristic in [calculations.md](calculations.md#12-participation-detection). |
| `website_menu` | Website Menu | boolean | no | see derivation | derived, stored, editable, precomputed | yes | no | Master switch of the per-event menu tree. Recomputed when `event_type_id` changes: at an actual template change the template value is taken; otherwise an empty value becomes false. |
| `menu_id` | Event Menu | many_to_one to Website Menu | no | none | stored | no | no | Root menu entry of the event menu tree. |
| `introduction_menu` | Introduction Menu | boolean | no | see derivation | derived, stored, editable | yes | no | Whether the "Home" page exists. Recomputed from `website_menu` at every change of that switch. |
| `register_menu` | Register Menu | boolean | no | see derivation | derived, stored, editable | yes | no | Whether the "Practical" page exists. Recomputed from `website_menu` at every change of that switch. |
| `community_menu` | Community Menu | boolean | no | see derivation | derived, stored, editable | yes | no | Whether the "Rooms" community page exists. Without the quiz capability package this is always false; with it, the same propagation rule as the booth menu applies. |
| `booth_menu` | Booth Register | boolean | no | see derivation | derived, stored, editable | yes | no | Whether the "Become exhibitor" page exists. Propagation rule below. |
| `exhibitor_menu` | Showcase Exhibitors | boolean | no | see derivation | derived, stored, editable | yes | no | Whether the "Exhibitors list" page exists. Propagation rule below. |
| `website_track` | Tracks on Website | boolean | no | see derivation | derived, stored, editable | yes | no | Whether the "Talks" and "Agenda" pages exist. Propagation rule below. |
| `website_track_proposal` | Proposals on Website | boolean | no | see derivation | derived, stored, editable | yes | no | Whether the "Propose a talk" page exists. Recomputed from `event_type_id` at an actual template change, otherwise it follows `website_track`. |
| `introduction_menu_ids`, `register_menu_ids`, `community_menu_ids`, `booth_menu_ids`, `exhibitor_menu_ids`, `track_menu_ids`, `track_proposal_menu_ids`, `other_menu_ids` | Introduction Menus, Register Menus, Event Community Menus, Event Booths Menus, Exhibitors Menus, Event Tracks Menus, Event Proposals Menus, Other Menus | one_to_many to Website Event Menu | — | — | stored on the other side, each filtered on its own menu type | no | — | The menu entries created for each switch. |

**Menu switch propagation rule** (used by `booth_menu`, `exhibitor_menu`, `website_track` and, with the quiz package, `community_menu`):

1. When `event_type_id` is set and `event_type_id` changed in this write, the switch takes the
   value of the matching switch of the template.
2. Otherwise, when `website_menu` is true and either `website_menu` changed in this write or the
   switch is currently false, the switch becomes true.
3. Otherwise, when `website_menu` is false, the switch becomes false.
4. Otherwise the current value is kept.

**Constraint `_check_website_id`** on the website link: if a website is chosen and its company differs from the event company, the save is rejected with *"The website must be from the same company as the event."*

## 1.10 Commercial and programme links

| Field | Full name | Type | Required | Default | Stored | Copied | Tracked | Visible to | Meaning |
|---|---|---|---|---|---|---|---|---|---|
| `currency_id` | Currency | many_to_one to Currency | — | — | derived from `company.currency`, read-only | — | — | everyone | The currency of the revenue figure. |
| `sale_order_lines_ids` | All sale order lines pointing to this event | one_to_many to Sales Order Line | — | — | stored on the other side | no | — | Salesperson | Every order line that points at this event. |
| `sale_price_total` | Sales (Tax Included) | monetary in `currency_id` | — | — | derived, not stored | — | — | Salesperson | Revenue including tax of the **confirmed** order lines of the event, converted into the event company currency at today's rate. |
| `lead_ids` | Leads | one_to_many to Lead | — | — | stored on the other side | no | — | Salesperson | Leads generated from this event. |
| `lead_count` | # Leads | integer | — | — | derived, not stored | — | — | Salesperson | Number of those leads. |
| `event_booth_ids` | Booths | one_to_many to Event Booth | no | none | derived, stored, editable, precomputed | yes | no | everyone | The booth catalogue; synchronised from the template as described in section 19. |
| `event_booth_count` | Total Booths | integer | — | — | derived, not stored | — | — | everyone | Total booths of the event. |
| `event_booth_count_available` | Available Booths | integer | — | — | derived, not stored | — | — | everyone | Booths still in state `available`. |
| `event_booth_category_ids` | Event Booth Category | many_to_many to Event Booth Category | — | — | derived, not stored | — | — | everyone | Distinct categories used by the booths of the event. |
| `event_booth_category_available_ids` | Event Booth Category Available | many_to_many to Event Booth Category | — | — | derived, not stored | — | — | everyone | Categories that still have at least one available booth; used by the public booth page. |
| `sponsor_ids` | Sponsors | one_to_many to Event Sponsor | — | — | stored on the other side | no | — | everyone | Sponsors and exhibitors of the event. |
| `sponsor_count` | Sponsor Count | integer | — | — | derived, not stored | — | — | everyone | Number of sponsors. |
| `track_ids` | Tracks | one_to_many to Event Track | — | — | stored on the other side | no | — | everyone | Talks of the programme. |
| `track_count` | Track Count | integer | — | — | derived, not stored | — | — | everyone | Number of talks whose stage is **not** a cancelling stage. |
| `allowed_track_tag_ids` | Available Track Tags | many_to_many to Event Track Tag | no | none | stored | yes | no | everyone | The tags a speaker may pick on the public proposal form. |
| `tracks_tag_ids` | Track Tags | many_to_many to Event Track Tag | — | — | derived, stored | — | — | everyone | The distinct tags actually used by the talks of the event, keeping only tags with a non-zero colour index. |
| `event_mail_ids` | Mail Schedule | one_to_many to Event Automated Mailing | no | see section 1.11 | derived, stored, editable | yes | no | everyone | The automatic communication schedule. |

## 1.11 Communication synchronisation from the template

When `event_type_id` is set or changed:

1. If there is no template and the event has no communication line yet, three default lines are created: one immediate line after each registration using the shipped registration confirmation message, one line one hour before the event using the shipped reminder message, and one line three days before the event using the same reminder message. The rule then stops.
2. Otherwise every existing line that has **not been sent** and has **no per-attendee trace** is deleted.
3. For every communication line of the template, the tuple (`interval_nbr`, `interval_unit`, `interval_type`, `template_ref`) is compared with the tuples of the surviving lines. A line is created only when that exact tuple is not already present.

## 1.12 Life cycle of an Event

| From | Trigger | Guard | To | Side effects |
|---|---|---|---|---|
| (none) | create | required fields present | first stage by sequence, kanban state `normal` | Default questions and default communication lines are attached; the website menu tree is created when `website_menu` is true. |
| any stage | user drags the card to another stage | — | chosen stage | Kanban state is reset to `normal` unless it is `cancel`. |
| any stage | `action_set_done` | at least one stage is flagged as the ending stage | the first ending stage by sequence | Nothing else. |
| any non-ending stage with `date_end` in the past | automatic housekeeping pass | — | the first ending stage by sequence | Runs as part of the periodic vacuum of the platform. |
| any | set `kanban_state = cancel` | — | unchanged stage | Registrations stop being open (`event_registrations_open` becomes false) and every not-yet-sent communication of the event reports the state `cancelled` and is skipped by the scheduler. |
| any | archive (`active = false`) | — | unchanged | The event disappears from default lists and its communications are skipped by the scheduler. |

---

# 2. Event Slot

**Full name:** Event Slot. **Transport name** `event.slot`, **storage name** `event_slot`. **Kind:** persistent entity. **Contributed by the capability package** `event`. Generated reference page: [`event.slot`](../../references/entities/event.slot.md).

One dated occurrence of a multi-slot event. The seat maximum of the event is applied to every slot separately.

**Default ordering:** by event, then `date`, then `start_hour`, then `end_hour`, then identifier.

**Display name rule:** `"<date in medium format>, <start hour in short format> - <end hour in short format>"`. When the caller asks for availability in the name, the event limits seats and the event is **not** multi-slot, the text becomes `"<that text> (Sold out)"` when no seat is left, or `"<that text> (<seats_available> seats remaining)"` otherwise.

| Field | Full name | Type | Required | Default | Stored | Meaning and rules |
|---|---|---|---|---|---|---|
| `event_id` | Event | many_to_one to Event | yes | none | stored, indexed | The owning event. Deletion behavior: cascade. |
| `date` | Date | date | yes | none | stored | The calendar day of the slot, expressed in the event time zone. |
| `start_hour` | Starting Hour | decimal (hours, fractional) | yes | none | stored | Start hour inside that day, expressed in the event time zone. `9.5` means 09:30. |
| `end_hour` | Ending Hour | decimal (hours, fractional) | yes | none | stored | End hour inside that day, expressed in the event time zone. |
| `date_tz` | Date Tz | selection | — | — | derived from `event.date_time_zone` | The zone in which `date`, `start_hour` and `end_hour` are read. |
| `start_datetime` | Start Datetime | datetime | — | — | derived, stored | `date` combined with `start_hour`, localised in `date_tz`, converted to coordinated universal time. |
| `end_datetime` | End Datetime | datetime | — | — | derived, stored | Same rule with `end_hour`. |
| `color` | Color | integer | no | 0 | stored | Colour used in the slot calendar. |
| `registration_ids` | Attendees | one_to_many to Event Registration | — | — | stored on the other side | Attendees of this slot. |
| `seats_reserved` | Number of Registrations | integer | — | — | derived, never stored | Active registrations of the slot in state `open`. |
| `seats_used` | Number of Attendees | integer | — | — | derived, never stored | Active registrations of the slot in state `done`. |
| `seats_taken` | Number of Taken Seats | integer | — | — | derived, never stored | The sum of the two. |
| `seats_available` | Available Seats | integer | — | — | derived, never stored | `event.seats_max − seats_taken` when that maximum is positive, otherwise zero. |
| `is_sold_out` | Sold Out | boolean | — | — | derived, not stored | True when the event limits seats and `seats_available` is zero. |

**Constraint `_check_hours`** on `start_hour` and `end_hour`:

- Both values must lie between 0 and 23.99 inclusive; otherwise *"A slot hour must be between 0:00 and 23:59."*
- The end hour must be strictly greater than the start hour; otherwise *"A slot end hour must be later than its start hour."* followed by a newline and the display name of the slot.

**Constraint `_check_time_range`** on `date`, `start_hour` and `end_hour`: both `start_datetime` and `end_datetime` must lie inside `[event.date_begin, event.date_end]`; otherwise:

```formula
A slot cannot be scheduled outside of its event time range.

Event:          <event start in medium format> - <event end in medium format>
Slot:           <slot display name>
```

**Deletion guard `_unlink_except_if_registrations`**: a slot that has at least one registration cannot be deleted. The message is *"The following slots cannot be deleted while they have one or more registrations linked to them:"* followed by one line `- <slot display name>` per slot.

**Public filter.** The public pages only offer slots that satisfy both conditions: the slot starts strictly in the future, **and** at least one combination of that slot with a ticket of the event (or with no ticket, when the event has none) still has availability, where "no limit" counts as available.

---

# 3. Event Ticket

**Full name:** Event Ticket. **Transport name** `event.event.ticket`, **storage name** `event_event_ticket`. **Kind:** persistent entity. **Contributed by the capability package** `event`. Generated reference page: [`event.event.ticket`](../../references/entities/event.event.ticket.md). Inherits the whole field set of Event Template Ticket (section 17) and adds the fields below.

**Default ordering:** by event, then `sequence`, then `name`, then identifier. When the product bridge is installed the ordering becomes event, `sequence`, `price`, `name`, identifier.

**Display name rule:** normally the `name`. When the caller asks for availability in the name:

```formula
if seats_max is zero OR the event is multi-slot: "<name>"
else if seats_available is zero:                     "<name> (Sold out)"
else:                                                "<name> (<seats_available> seats remaining)"
```

Availability is deliberately hidden for multi-slot events, because a single number cannot describe the remaining seats of every slot and ticket combination.

**Creation default:** when a ticket is created from the form of an event, and the name would otherwise be the shipped default `Registration`, the name becomes `Registration for <event name>`.

| Field | Full name | Type | Required | Default | Stored | Meaning and rules |
|---|---|---|---|---|---|---|
| `event_id` | Event | many_to_one to Event | yes | none | stored, indexed | The owning event. Deletion behavior: cascade. |
| `event_type_id` | Event Type | many_to_one to Event Template | no | none | stored | Inherited from the template ticket but not required here. Deletion behavior: set to empty. |
| `company_id` | Company | many_to_one to Company | — | — | derived from `event.company` | Used by the multi-company record rule "Event/Ticket: multi-company", which matches `event_id.company_id IN (companies of the reader) OR event_id.company_id IS NULL`. |
| `start_sale_datetime` | Registration Start | datetime | no | none | stored | Moment from which this ticket may be sold. Empty means "already open". |
| `end_sale_datetime` | Registration End | datetime | no | none | stored | Moment after which this ticket may no longer be sold. Empty means "never closes". |
| `is_launched` | Are sales launched | boolean | — | — | derived, not stored | True when `start_sale_datetime` is empty, or when that moment expressed in the event time zone is at or before the current moment expressed in the same zone. |
| `is_expired` | Is Expired | boolean | — | — | derived, not stored | True when `end_sale_datetime` is set and, expressed in the event time zone, is strictly before the current moment in the same zone. |
| `sale_available` | Is Available | boolean | — | — | derived with elevated rights, not stored | `is_launched AND NOT is_expired AND NOT is_sold_out`. With the product bridge installed, a ticket whose product is archived is never available. |
| `is_sold_out` | Sold Out | boolean | — | — | derived, not stored | `(seats_limited AND seats_available = 0) OR event.event_registrations_sold_out`. |
| `registration_ids` | Registrations | one_to_many to Event Registration | — | — | stored on the other side | Attendees holding this ticket. |
| `seats_reserved` | Reserved Seats | integer | — | — | derived, never stored | Active registrations of this ticket in state `open`. |
| `seats_used` | Used Seats | integer | — | — | derived, never stored | Active registrations of this ticket in state `done`. |
| `seats_taken` | Taken Seats | integer | — | — | derived, never stored | The sum of the two. |
| `seats_available` | Available Seats | integer | — | — | derived, never stored | `seats_max − seats_taken` when the maximum is positive, otherwise zero. |
| `limit_max_per_order` | Limit per Order | integer | no | 0 | stored | Largest number of this ticket a single order may contain. Zero switches the rule off. |
| `color` | Color | text | no | `#875A7B` | stored | Colour used on printed material for this ticket. |
| `price_incl` | Price include | decimal (product price precision) | — | — | derived, editable, not stored | The unit price with the taxes of the product applied, computed in the event company currency for one unit. Zero when there is no product or no price. |
| `price_reduce_taxinc` | Price Reduce Tax inc | decimal (product price precision) | — | — | derived with elevated rights, not stored | `price_reduce` with the taxes of the product applied. Only the taxes belonging to the event company are used. |

**Constraint `_constrains_dates_coherency`** on the two sales dates: when both are set and the start is after the end, the save is rejected with *"The stop date cannot be earlier than the start date. Please check ticket <ticket name>"*.

**Constraint `_constrains_limit_max_per_order`** on `limit_max_per_order` and `seats_max`, evaluated in this order:

1. If a seat maximum is set and the per-order limit exceeds it: *"The limit per order cannot be greater than the maximum seats number. Please check ticket <ticket name>"*.
2. If the per-order limit exceeds the absolute ceiling of 30 tickets per order: *"The limit per order cannot be greater than 30. Please check ticket <ticket name>"*.
3. If the per-order limit is negative: *"The limit per order must be positive. Please check ticket <ticket name>"*.

**Deletion guard `_unlink_except_if_registrations`**: a ticket with at least one registration cannot be deleted. The message is *"The following tickets cannot be deleted while they have one or more registrations linked to them:"* followed by one line `- <ticket name>` per ticket.

**Sales description.** The multi-line description offered to a sales order line is `"<ticket display name>\n<event display name>"`. When the product of the ticket has its own sales description, that description replaces the ticket display name, giving `"<product sales description>\n<event display name>"`.

**Public discount display.** On the storefront, a strike-through original price is shown for a ticket only when the ticket price is non-zero, a pricelist applies to the reader, the pricelist rule that matches the ticket product for a quantity of one is itself allowed to show a discount, and `price − price_reduce` is strictly positive.

---

# 4. Event Registration

**Full name:** Event Registration. **Transport name** `event.registration`, **storage name** `event_registration`. **Kind:** persistent entity. **Contributed by the capability package** `event`. Generated reference page: [`event.registration`](../../references/entities/event.registration.md). **Carries a discussion thread.** **Archivable.** Outgoing messages default to the electronic mail address of the record rather than to its contact.

One attendee seat. Registrations are the unit in which seats are counted, communications are traced, badges are printed and attendance is recorded.

**Default ordering:** by identifier descending (newest first).

**Display name rule:** the `name` when set, otherwise `"#<identifier>"`.

**Thread subject rule:** `"<event name> - Registration for <attendee name>"`, or `"<event name> - Registration #<identifier>"` when the attendee has no name.

## 4.1 Fields

| Field | Full name | Type | Required | Default | Stored | Copied | Tracked | Meaning and rules |
|---|---|---|---|---|---|---|---|---|
| `event_id` | Event | many_to_one to Event | yes | none | stored, indexed | yes | yes | The event. Deletion behavior: restrict (the platform default). |
| `event_slot_id` | Slot | many_to_one to Event Slot | no | none | stored, indexed when set | yes | yes | The chosen slot. Deletion behavior: restrict. The selectable slots are restricted to those of `event_id`. |
| `event_ticket_id` | Ticket Type | many_to_one to Event Ticket | no | none | stored, indexed when set | yes | yes | The chosen ticket. Deletion behavior: restrict. |
| `is_multi_slots` | Is Event Multi Slots | boolean | — | — | derived from `event.is_multi_slots` | — | — | Convenience flag driving the visibility of the slot field. |
| `active` | Active | boolean | yes | true | stored | yes | no | Archiving a registration removes it from every seat counter. |
| `barcode` | Barcode | text | no | a fresh pseudo-random value | stored, read-only | **no** | no | The code printed on the badge and read at the desk. Generated as the decimal text of a pseudo-random eight-byte number; a decimal rendering is longer than a hexadecimal one but produces a denser linear barcode. |
| `partner_id` | Booked by | many_to_one to Contact | no | none | stored, indexed when set | yes | yes | The contact who booked the seat. Labelled "Booked by": one contact may book many seats for different attendees. |
| `name` | Attendee Name | text | no | see derivation | derived, stored, editable, text-indexed | yes | yes | Attendee name. Filled from the contact only while empty. |
| `email` | Email | text | no | see derivation | derived, stored, editable | yes | yes | Attendee electronic mail address. Filled from the contact only while empty. |
| `phone` | Phone | text | no | see derivation | derived, stored, editable | yes | yes | Attendee telephone number. Filled from the contact only while empty, and reformatted (see below). |
| `company_name` | Company Name | text | no | see derivation | derived, stored, editable | yes | yes | Attendee company name. Filled from the contact only while empty. |
| `state` | Status | selection | yes | `open`, or the derived value when an order is involved | derived, stored, editable, precomputed, read-only in the form | **no** | yes | The life cycle state; see section 4.4. |
| `sale_status` | Sale Status | selection | no | see section 4.5 | derived, stored, precomputed, read-only | no | no | Payment situation. Values: `to_pay` (Not Sold), `sold` (Sold), `free` (Free). |
| `date_closed` | Attended Date | datetime | no | see derivation | derived, stored, editable | yes | no | The moment attendance was recorded. Set to the current moment when the state becomes `done` and the field is still empty; cleared when the state is anything else and the field is empty. Once set by hand it is left alone. |
| `event_begin_date` | Event Start Date | datetime | — | — | derived, not stored, searchable | — | — | `event_slot.start_datetime` when a slot is chosen, otherwise `event.date_begin`. The search rewrites into `(slot is set AND slot start matches) OR (slot is empty AND event start matches)`. |
| `event_end_date` | Event End Date | datetime | — | — | derived, not stored, searchable | — | — | Same rule with the end datetimes. |
| `event_date_range` | Date Range | text | — | — | derived, not stored | — | — | A human phrase such as `today`, `tomorrow`, `in 3 days`, `next week`, `next month` or `on 12 Jun 2026`; see [calculations.md](calculations.md#7-relative-date-phrase). |
| `event_organizer_id` | Event Organizer | many_to_one to Contact | — | — | derived from `event.organizer`, read-only | — | — | Shown on the form and used by messages. |
| `event_user_id` | Event Responsible | many_to_one to User | — | — | derived from `event.user`, read-only | — | — | The event responsible. |
| `company_id` | Company | many_to_one to Company | no | derived from `event.company` | derived, stored, editable | yes | no | Used by the record rule "Event/Registration: multi-company". |
| `utm_campaign_id` | Campaign | many_to_one to Campaign | no | from the campaign-tracking defaults, then from the order | derived, stored, editable, indexed | yes | no | Marketing attribution. Deletion behavior: set to empty. |
| `utm_source_id` | Source | many_to_one to Campaign Source | no | as above | derived, stored, editable, indexed | yes | no | Marketing attribution. Deletion behavior: set to empty. |
| `utm_medium_id` | Medium | many_to_one to Campaign Medium | no | as above | derived, stored, editable, indexed | yes | no | Marketing attribution. Deletion behavior: set to empty. |
| `registration_answer_ids` | Attendee Answers | one_to_many to Event Registration Answer | — | — | stored on the other side | yes | — | Every answer of this attendee. |
| `registration_answer_choice_ids` | Attendee Selection Answers | one_to_many to Event Registration Answer | — | — | the same list filtered on `question_type = simple_choice` | — | — | The selection answers, printed as chips on the badge and the ticket. |
| `mail_registration_ids` | Scheduler Emails | one_to_many to Registration Mail Scheduler | — | — | stored on the other side, read-only | no | — | The per-attendee communication traces. |
| `registration_properties` | Properties | properties | no | none | stored | yes | no | The user-defined fields whose definition lives on `event.registration_properties_definition`. |
| `visitor_id` | Visitor | many_to_one to Website Visitor | no | none | stored, indexed when set | yes | no | The anonymous or identified site visitor who registered. Deletion behavior: set to empty. |
| `sale_order_id` | Sales Order | many_to_one to Sales Order | no | none | stored | **no** | no | The order that sold the seat. Deletion behavior: **cascade** — deleting the order deletes the registration. |
| `sale_order_line_id` | Sales Order Line | many_to_one to Sales Order Line | no | none | stored, indexed when set | **no** | no | The order line that sold the seat. Deletion behavior: **cascade**. |
| `pos_order_line_id` | Point-of-sale order line | many_to_one to Point of Sale Order Line | no | none | stored, indexed when set | **no** | no | The counter line that sold the seat. Deletion behavior: **cascade**. |
| `pos_order_id` | Point-of-sale order | many_to_one to Point of Sale Order | — | — | derived from `pos_order_line_id.order_id` | — | — | The counter order. |
| `lead_ids` | Leads | many_to_many to Lead | — | — | stored, read-only | **no** | — | Leads generated from this attendee. Visible to the Salesperson group. |
| `lead_count` | # Leads | integer | — | — | derived with elevated rights, not stored | — | — | Number of those leads. |

## 4.2 Contact synchronisation

`name`, `email`, `phone` and `company_name` are each filled from the contact **only when they are still empty**, and the value is read from the *contact* address of the chosen partner, that is from the address the partner designates as its contact address (which for a company is usually a child record). An existing manual value is never overwritten.

**Telephone formatting.** On creation, a supplied telephone number is reformatted for the first country found in this order: the country of the chosen contact, then the country of the event, then the country of the active company. If formatting fails, the raw value is kept. The same reformatting is applied while editing the form whenever `phone`, `event_id` or `partner_id` changes.

**On-change `_onchange_event`**: when the event is changed in the form, a slot that belongs to another event is cleared, and a ticket that belongs to another event is cleared.

## 4.3 Constraints and database rules

| Rule | Condition | Message |
|---|---|---|
| Unique barcode (`_barcode_event_uniq`) | `unique(barcode)` | *"Barcode should be unique"* |
| `_check_event_slot` | A chosen slot must belong to the chosen event | *"Invalid event / slot choice"* |
| `_check_event_slot` | A multi-slot event requires a slot | *"Slot choice is mandatory on multi-slots events."* |
| `_check_event_ticket` | A chosen ticket must belong to the chosen event | *"Invalid event / ticket choice"* |
| `_check_seats_availability` | Every active registration in state `open` or `done` must fit in the remaining seats of its event, its slot and its ticket | See the seat verification message in [business-rules.md](business-rules.md#ev-rule-030) |

The seat check runs whenever `active`, `state`, `event_id`, `event_slot_id` or `event_ticket_id` changes. It groups the registrations being checked by event, then by (slot, ticket) pair, and asks the event to verify that the current counts do not overflow any limit. It therefore also fires when a registration is un-archived or confirmed.

## 4.4 State machine

| From | Trigger | Guard | To | Side effects |
|---|---|---|---|---|
| — | create without an order | — | `open` | Seat availability is verified; attendee-based communications are scheduled and, in synchronous mode, executed immediately. |
| — | create with an order | see section 4.5 | `draft` or `open` or `cancel` | As derived from the order state and total. |
| `draft`, `cancel` | `action_confirm` | seats available | `open` | Attendee-based communications are scheduled for this attendee and the scheduler is woken. |
| `draft`, `open`, `cancel` | `action_set_done` | seats available | `done` | `date_closed` is stamped with the current moment; the note *"Attended on <date in short format>"* is logged in the thread; attendee-based communications are scheduled if they were not. |
| any | `action_cancel` | — | `cancel` | The seat is released from every counter. Pending communication traces for this attendee are deleted the next time the scheduler runs. |
| any | `action_set_draft` | — | `draft` | The seat is released from every counter. |
| any | archive | — | unchanged | The seat is released from every counter while archived. |
| `cancel`, `draft` | un-archive or confirm | seats available, otherwise the operation is rejected | unchanged or `open` | See the seat verification rule. |

Only `open` and `done` consume a seat. `draft` and `cancel` never do, and neither does an archived registration in any state.

## 4.5 Derivation of `state` and `sale_status`

The two fields are computed together. Without the product bridge there is no `sale_status` at all and the state defaults to `open`.

**Base rule (product bridge, no order of any kind):** if the registration has no sales order and no counter order, then an empty `sale_status` becomes `free` and an empty `state` becomes `open`.

**With a sales order**, registrations are grouped by order and, for each group:

1. Registrations whose order is cancelled are set to state `cancel`. Together with the registrations already cancelled they form the "cancelled set".
2. If the **order total including tax is zero** (compared with the rounding of the order currency): every registration of the group gets `sale_status = free`, and every registration with no state or state `draft` becomes `open`.
3. Otherwise: the registrations whose order is confirmed and which are not in the cancelled set get `sale_status = sold`; all the others get `sale_status = to_pay`. The sold ones whose state is empty, `draft` or `cancel` become `open`. Every registration that is neither sold nor cancelled becomes `draft`.
4. After the group pass, any registration still without a sale status gets `free` and any registration still without a state gets `open`.

**With a counter order**, the rule is: a cancelled counter order sets state `cancel`; a counter order with a zero total sets `sale_status = free` and state `open`; any other counter order sets `sale_status = sold` and state `open`. When the counter-and-sales bridge is installed, this is refined: a counter order in state `paid`, `done` or `invoiced` gives `sale_status = sold` and state `open`, and any other counter state gives `sale_status = to_pay` and state `draft`.

Whenever this derivation moves a registration from `draft` or `cancel` to `open`, the attendee-based communication schedulers of its event are woken for that attendee.

## 4.6 Order synchronisation

When a registration is created or written with a `sale_order_line_id`, the following values are copied from that line and overwrite whatever was supplied: `event_id`, `event_slot_id`, `event_ticket_id`, `sale_order_id` and `sale_order_line_id`. The contact is copied from the order customer, **except** when the current reader is the anonymous website user and that user's own contact is the order customer, in which case the contact is left empty (an anonymous shopper must not be recorded as the attendee).

When a registration is created with a sales order, a note linking back to that order is posted in its thread.

When the slot or the ticket of a registration is changed while it is attached to a sales order, a warning activity is scheduled on that order for the event responsible (or the order salesperson, or the administrator as a last resort) describing the change from the old to the new slot or ticket.

Changing the customer on a sales order rewrites the contact of every registration attached to that order.

## 4.7 Desk check-in contract

The operation `register_attendee(barcode, event)` searches the first registration with that exact barcode and returns a summary plus a status. The situations are tested in the order of the rows and the first match decides:

| Situation | Status returned |
|---|---|
| No registration with that barcode | `invalid_ticket` |
| The registration is cancelled | `canceled_registration` |
| The registration is unconfirmed (`draft`) | `unconfirmed_registration` |
| The event of the registration is finished | `not_ongoing_event` |
| The registration is already attended (`done`) | `already_registered` |
| A specific event was requested and the registration belongs to another event | `need_manual_confirmation` |
| Otherwise | the registration is set to `done` and `confirmed_registration` is returned |

Because the attendance test precedes the event test, an attendee who has already been scanned is reported as `already_registered` even when the desk is opened for a different event.

The `invalid_ticket` answer carries no summary, because there is no registration to summarise; it carries that single word under the key `error` while every other answer carries its word under the key `status`. The summary contains the registration identifier, the attendee name, the contact, the slot display name, the ticket name, the event identifier and display name, the display text of every selection answer, the company name, the badge format, the attendance date formatted short, and whether that attendance date falls on the current day in the event time zone. With the product bridge it also contains the sale status, its readable label, and a flag "has to pay" that is true when the sale status is `to_pay`.

## 4.8 Website registration

Only these fields may be written by the public registration form: `name`, `phone`, `email`, `company_name`, `event_id`, `partner_id`, `event_slot_id`, `event_ticket_id`. Any other posted field is ignored.

---

# 5. Event Registration Answer

**Full name:** Event Registration Answer. **Transport name** `event.registration.answer`, **storage name** `event_registration_answer`. **Kind:** persistent entity. **Contributed by the capability package** `event`. Generated reference page: [`event.registration.answer`](../../references/entities/event.registration.answer.md).

One answer of one attendee to one question. Text searches on this entity match either the chosen suggestion or the typed text.

| Field | Full name | Type | Required | Default | Stored | Meaning and rules |
|---|---|---|---|---|---|---|
| `question_id` | Question | many_to_one to Event Question | yes | none | stored | The question answered. Deletion behavior: restrict. Only questions linked to the event of the registration may be chosen. |
| `registration_id` | Registration | many_to_one to Event Registration | yes | none | stored, indexed | The attendee. Deletion behavior: cascade. |
| `partner_id` | Partner | many_to_one to Contact | — | — | derived from `registration.partner` | Convenience for reporting. |
| `event_id` | Event | many_to_one to Event | — | — | derived from `registration.event` | Convenience for reporting and for the answer breakdown report. |
| `question_type` | Question Type | selection | — | — | derived from `question.question_type` | Used to decide which value field carries the answer. |
| `value_answer_id` | Suggested answer | many_to_one to Event Question Answer | no | none | stored | The chosen suggestion, for a selection question. |
| `value_text_box` | Text answer | long_text | no | none | stored | The typed answer, for every other question type. |

**Database check `_value_check`:** `value_answer_id IS NOT NULL OR COALESCE(value_text_box, '') <> ''`, with the message *"There must be a suggested value or a text value."*

**Display name rule:** the name of the chosen suggestion when the question type is `simple_choice`, otherwise the typed text.

---

# 6. Event Question

**Full name:** Event Question. **Transport name** `event.question`, **storage name** `event_question`. **Kind:** persistent entity. **Contributed by the capability package** `event`. Generated reference page: [`event.question`](../../references/entities/event.question.md). **Archivable.**

A question asked on the registration form. A question can be shared by several events and by several templates.

**Default ordering:** by `sequence`, then identifier. **Display name:** the `title`.

| Field | Full name | Type | Required | Default | Stored | Meaning and rules |
|---|---|---|---|---|---|---|
| `title` | Title | text (translatable) | yes | none | stored | The question as shown to the registrant. |
| `question_type` | Question Type | selection | yes | `simple_choice` | stored | Values: `simple_choice` (Selection), `text_box` (Text Input), `name` (Name), `email` (Email), `phone` (Phone), `company_name` (Company). The last four also write the matching field of the registration. |
| `sequence` | Sequence | integer | no | 10 | stored | Display order on the form. |
| `once_per_order` | Ask once per order | boolean | no | false | stored | When true the question is asked once for the whole order and its answer is copied onto every attendee of that order; when false it is asked for each attendee. |
| `is_mandatory_answer` | Mandatory Answer | boolean | no | false | stored | When true the public form refuses to submit without an answer. |
| `is_default` | Default question | boolean | no | false | stored | When true the question is attached to every newly created event that has no template. |
| `is_reusable` | Is Reusable | boolean | no | true | derived, stored | Whether the question may be picked for other events. Recomputed from `is_default` and the template links: a default question is always reusable. |
| `active` | Active | boolean | yes | true | stored | Archived questions are no longer offered. |
| `event_type_ids` | Event Types | many_to_many to Event Template | no | none | stored, **not copied** | The templates that include this question. |
| `event_ids` | Events | many_to_many to Event | no | none | stored, **not copied** | The events that include this question. |
| `event_count` | # Events | integer | — | — | derived, not stored | Number of events using this question, obtained by grouping events per question. |
| `answer_ids` | Answers | one_to_many to Event Question Answer | no | none | stored on the other side, copied | The suggestions of a selection question. |

**Database check `_check_default_question_is_reusable`:** `CHECK(is_default IS DISTINCT FROM TRUE OR is_reusable IS TRUE)` with the message *"A default question must be reusable."*

**Write guard.** Changing `question_type` on a question that already has at least one stored answer is rejected with *"You cannot change the question type of a question that already has answers!"* The check is applied only to the questions whose type actually differs from the requested one.

**Deletion guards.**

- A question that already has stored answers cannot be deleted: *"You cannot delete a question that has already been answered by attendees. You can archive it instead."*
- A question that is part of the default question set cannot be deleted: *"You cannot delete a default question."*

---

# 7. Event Question Answer

**Full name:** Event Question Answer. **Transport name** `event.question.answer`, **storage name** `event_question_answer`. **Kind:** persistent entity. **Contributed by the capability package** `event`. Generated reference page: [`event.question.answer`](../../references/entities/event.question.answer.md).

One suggestion of a selection question.

**Default ordering:** by `sequence`, then identifier.

| Field | Full name | Type | Required | Default | Stored | Meaning |
|---|---|---|---|---|---|---|
| `name` | Answer | text (translatable) | yes | none | stored | The suggestion text. |
| `question_id` | Question | many_to_one to Event Question | yes | none | stored, indexed | The owning question. Deletion behavior: cascade. |
| `sequence` | Sequence | integer | no | 10 | stored | Display order. |

**Deletion guard.** A suggestion already chosen by at least one attendee cannot be deleted: *"You cannot delete an answer that has already been selected by attendees."*

---

# 8. Event Automated Mailing

**Full name:** Event Automated Mailing. **Transport name** `event.mail`, **storage name** `event_mail`. **Kind:** persistent entity. **Contributed by the capability package** `event`. Generated reference page: [`event.mail`](../../references/entities/event.mail.md). Its readable name is the name of its event.

One scheduled automatic communication of one event.

| Field | Full name | Type | Required | Default | Stored | Copied | Meaning and rules |
|---|---|---|---|---|---|---|---|
| `event_id` | Event | many_to_one to Event | yes | none | stored, indexed | yes | The owning event. Deletion behavior: cascade. |
| `sequence` | Display order | integer | no | none | stored | yes | Display order in the communication tab. |
| `interval_nbr` | Interval | integer | no | 1 | stored | yes | How many units of time the interval lasts. |
| `interval_unit` | Unit | selection | yes | `hours` | stored | yes | Values: `now` (Immediately), `hours` (Hours), `days` (Days), `weeks` (Weeks), `months` (Months). A week counts as exactly seven days; a month is a calendar month. |
| `interval_type` | Trigger | selection | yes | `before_event` | stored | yes | Values: `after_sub` (After each registration), `before_event` (Before the event starts), `after_event_start` (After the event started), `after_event` (After the event ended), `before_event_end` (Before the event ends). When the event uses slots, the interval is relative to each slot instead of the whole event. |
| `template_ref` | Template | reference | yes | none | stored | yes | Points at the message to send. The allowed targets are Email Template and, with the text message package, Text Message Template. Deleting an Email Template cascades to this record. |
| `notification_type` | Send | selection | — | — | derived, not stored | — | `mail` by default; `sms` when `template_ref` points at a Text Message Template. |
| `scheduled_date` | Schedule Date | datetime | — | — | derived, stored | yes | The moment at which the communication is due; see [calculations.md](calculations.md#5-communication-schedule-dates). Recomputing it also wakes the scheduled job for the earliest of the new dates. |
| `error_datetime` | Last Error | datetime | no | none | stored | yes | The moment of the last failure. Set when a run raises; cleared when a run succeeds. |
| `mail_done` | Sent | boolean | no | false | stored, read-only | **no** | Whether the global communication has finished. |
| `mail_count_done` | # Sent | integer | no | 0 | stored, read-only | **no** | How many attendees have been reached. |
| `last_registration_id` | Last Attendee | many_to_one to Event Registration | no | none | stored | yes | The highest registration identifier already processed by a global communication, used as the resume point between batches. |
| `mail_registration_ids` | Mail Registration | one_to_many to Registration Mail Scheduler | — | — | stored on the other side | — | The per-attendee traces of an attendee-based communication. |
| `mail_slot_ids` | Mail Slot | one_to_many to Slot Mail Scheduler | — | — | stored on the other side | — | The per-slot traces of a slot-based communication. |
| `mail_state` | Global communication Status | selection | — | — | derived, not stored | — | The readable status; see the table below. |

**Derivation of `mail_state`**, evaluated in this order:

| Order | Condition | Value |
|---|---|---|
| 1 | `error_datetime` is set | `error` (Error) |
| 2 | not done and the event kanban state is `cancel` | `cancelled` (Cancelled) |
| 3 | `interval_type = after_sub` | `running` (Running) |
| 4 | `mail_done` is true | `sent` (Sent) |
| 5 | otherwise | `scheduled` (Scheduled) |

**Template validity.** Before a run, every schedule whose reference does not point at a record of the model matching its notification type, or whose referenced record no longer exists, is dropped from that run and a warning is written to the technical log. Deleting an Email Template or a Text Message Template also deletes, with elevated rights, every Event Automated Mailing and every template communication line that referenced it.

---

# 9. Registration Mail Scheduler

**Full name:** Registration Mail Scheduler. **Transport name** `event.mail.registration`, **storage name** `event_mail_registration`. **Kind:** persistent entity. **Contributed by the capability package** `event`. Generated reference page: [`event.mail.registration`](../../references/entities/event.mail.registration.md). Its readable name is the name of its scheduler.

The per-attendee trace of an attendee-based communication: it records that a given attendee is due to receive, or has received, a given message.

**Default ordering:** by `scheduled_date` descending, then identifier ascending.

| Field | Full name | Type | Required | Default | Stored | Meaning and rules |
|---|---|---|---|---|---|---|
| `scheduler_id` | Mail Scheduler | many_to_one to Event Automated Mailing | yes | none | stored, indexed | The schedule that produced this trace. Deletion behavior: cascade. |
| `registration_id` | Attendee | many_to_one to Event Registration | yes | none | stored, indexed | The attendee. Deletion behavior: cascade. |
| `scheduled_date` | Scheduled Time | datetime | — | — | derived, stored | `registration.created_on` with sub-seconds cleared, plus the interval of the scheduler. Empty when there is no registration. |
| `mail_sent` | Mail Sent | boolean | no | false | stored | Whether the message has gone out. |

**Skip rule.** A trace is eligible for sending when `mail_sent = false`, `scheduled_date` is set, and `scheduled_date <= now`. When the operation is invoked directly on traces rather than through the parent scheduler, the additional condition `registration.state IN ('open', 'done')` is applied.

---

# 10. Slot Mail Scheduler

**Full name:** Slot Mail Scheduler. **Transport name** `event.mail.slot`, **storage name** `event_mail_slot`. **Kind:** persistent entity. **Contributed by the capability package** `event`. Generated reference page: [`event.mail.slot`](../../references/entities/event.mail.slot.md). Its readable name is the name of its scheduler.

The per-slot trace of a global communication on a multi-slot event: the same schedule then fires once per slot, each time with its own due date and its own resume point.

**Default ordering:** by `scheduled_date` descending, then identifier ascending.

| Field | Full name | Type | Required | Default | Stored | Meaning and rules |
|---|---|---|---|---|---|---|
| `scheduler_id` | Mail Scheduler | many_to_one to Event Automated Mailing | yes | none | stored, indexed | The parent schedule. Deletion behavior: cascade. |
| `event_slot_id` | Slot | many_to_one to Event Slot | yes | none | stored | The slot this trace covers. Deletion behavior: cascade. |
| `scheduled_date` | Schedule Date | datetime | — | — | derived, stored | Computed from the slot start or the slot end according to the interval type of the parent; see [calculations.md](calculations.md#5-communication-schedule-dates). Recomputing it wakes the scheduled job. |
| `last_registration_id` | Last Attendee | many_to_one to Event Registration | no | none | stored | The resume point inside this slot. |
| `mail_count_done` | # Sent | integer | no | 0 | stored, read-only, **not copied** | Attendees of this slot already reached. |
| `mail_done` | Sent | boolean | no | false | stored, read-only, **not copied** | Whether this slot is finished. |

---

# 11. Event Stage

**Full name:** Event Stage. **Transport name** `event.stage`, **storage name** `event_stage`. **Kind:** persistent entity. **Contributed by the capability package** `event`. Generated reference page: [`event.stage`](../../references/entities/event.stage.md).

A column of the event pipeline.

**Default ordering:** by `sequence`, then `name`.

| Field | Full name | Type | Required | Default | Stored | Meaning |
|---|---|---|---|---|---|---|
| `name` | Stage Name | text (translatable) | yes | none | stored | Stage label. |
| `description` | Stage description | long_text (translatable) | no | none | stored | Help text shown on the column. |
| `sequence` | Sequence | integer | no | 1 | stored | Position of the column. |
| `fold` | Folded in Kanban | boolean | no | false | stored | Whether the column is collapsed. |
| `pipe_end` | End Stage | boolean | no | false | stored | Marks the stage into which finished events are moved automatically. |

---

# 12. Event Tag and Event Tag Category

## 12.1 Event Tag Category

**Full name:** Event Tag Category. **Transport name** `event.tag.category`, **storage name** `event_tag_category`. **Kind:** persistent entity. **Contributed by the capability package** `event`. Generated reference page: [`event.tag.category`](../../references/entities/event.tag.category.md). **Publishable** (per website).

**Default ordering:** by `sequence`.

| Field | Full name | Type | Required | Default | Stored | Meaning |
|---|---|---|---|---|---|---|
| `name` | Name | text (translatable) | yes | none | stored | Category label, shown as the heading of a filter block on the public event list. |
| `sequence` | Sequence | integer | no | the highest existing sequence plus one | stored | Position of the block. |
| `tag_ids` | Tags | one_to_many to Event Tag | — | — | stored on the other side | The tags of this category. |

A tag category is published by default when created. Only published categories appear in the public filter panel.

## 12.2 Event Tag

**Full name:** Event Tag. **Transport name** `event.tag`, **storage name** `event_tag`. **Kind:** persistent entity. **Contributed by the capability package** `event`. Generated reference page: [`event.tag`](../../references/entities/event.tag.md). **Publishable** (per website).

**Default ordering:** by the sequence of the category, then by `sequence`, then by identifier.

| Field | Full name | Type | Required | Default | Stored | Meaning |
|---|---|---|---|---|---|---|
| `name` | Name | text (translatable) | yes | none | stored | Tag label. |
| `sequence` | Sequence | integer | no | 0 | stored | Position inside the category. |
| `category_id` | Category | many_to_one to Event Tag Category | yes | none | stored, indexed | Owning category. Deletion behavior: cascade. |
| `category_sequence` | Category Sequence | integer | — | — | derived from `category.sequence`, stored | Kept as a stored copy purely to allow ordering tags by category without an extra join. |
| `color` | Color Index | integer | no | a pseudo-random value between 1 and 11 inclusive | stored | Display colour. **A zero or empty colour hides the tag from the kanban board and from the public site**; this is how internal-only tags are distinguished from public classification tags. |

Anonymous and portal readers only see tags whose category is published and whose colour index is neither empty nor zero.

---

# 13. Event Template

**Full name:** Event Template. **Transport name** `event.type`, **storage name** `event_type`. **Kind:** persistent entity. **Contributed by the capability package** `event`. Generated reference page: [`event.type`](../../references/entities/event.type.md).

A reusable bundle of defaults applied to a new event.

**Default ordering:** by `sequence`, then identifier.

| Field | Full name | Type | Required | Default | Stored | Meaning and rules |
|---|---|---|---|---|---|---|
| `name` | Event Template | text (translatable) | yes | none | stored | Template label. |
| `sequence` | Sequence | integer | no | 10 | stored | Display order. |
| `note` | Note | rich_text | no | none | stored | Copied into the event note when the event note is empty. |
| `has_seats_limitation` | Limited Seats | boolean | no | false | stored | Copied into `seats_limited`. |
| `seats_max` | Maximum Registrations | integer | no | see derivation | derived, stored, editable | Copied into `seats_max`. Recomputed from `has_seats_limitation`: switching the limitation off forces the maximum to zero. |
| `default_timezone` | Timezone | selection of time zone names | no | the time zone of the current user, or coordinated universal time | stored | Copied into `date_tz`. |
| `ticket_instructions` | Ticket Instructions | rich_text (translatable) | no | none | stored | Copied into the event ticket instructions when the event value is empty. |
| `event_type_ticket_ids` | Tickets | one_to_many to Event Template Ticket | no | none | stored on the other side | The ticket catalogue to copy. |
| `event_type_mail_ids` | Mail Schedule | one_to_many to Mail Scheduling on Event Category | no | the three default lines described in section 1.11 | stored on the other side | The communication schedule to copy. |
| `event_type_booth_ids` | Booths | one_to_many to Event Booth Template | no | none | stored on the other side | The booth catalogue to copy. |
| `tag_ids` | Tags | many_to_many to Event Tag | no | none | stored | Copied into the event tags when the event has none. |
| `question_ids` | Questions | many_to_many to Event Question | no | every active default question | stored, copied on duplication | The question set to copy. |
| `website_menu` | Display a dedicated menu on Website | boolean | no | false | stored | Copied into the event website menu switch. |
| `community_menu` | Community Menu | boolean | no | see derivation | derived, stored, editable | Copied into the event community switch. Recomputed to follow `website_menu`. |
| `booth_menu` | Booths on Website | boolean | no | see derivation | derived, stored, editable | Copied into the event booth page switch. Recomputed to follow `website_menu`. |
| `exhibitor_menu` | Showcase Exhibitors | boolean | no | see derivation | derived, stored, editable | Copied into the event exhibitor page switch. Recomputed to follow `website_menu`. |
| `website_track` | Tracks on Website | boolean | no | see derivation | derived, stored, editable | Copied into the event talks switch. Recomputed to follow `website_menu`. |
| `website_track_proposal` | Tracks Proposals on Website | boolean | no | see derivation | derived, stored, editable | Copied into the event proposal switch. Recomputed to follow `website_menu`. |

---

# 14. Event Template Ticket

**Full name:** Event Template Ticket. **Transport name** `event.type.ticket`, **storage name** `event_type_ticket`. **Kind:** persistent entity. **Contributed by the capability package** `event`. Generated reference page: [`event.type.ticket`](../../references/entities/event.type.ticket.md). This is also the shared definition inherited by Event Ticket.

**Default ordering:** by `sequence`, then `name`, then identifier; with the product bridge, by `sequence`, `price`, `name`, identifier.

| Field | Full name | Type | Required | Default | Stored | Meaning and rules |
|---|---|---|---|---|---|---|
| `name` | Name | text (translatable) | yes | `Registration` | stored | Ticket label. |
| `sequence` | Sequence | integer | no | 10 | stored | Display order. |
| `description` | Description | long_text (translatable) | no | see derivation | derived, stored, editable | Public description. Recomputed from the product: the sales description of the product is taken when it exists; otherwise an empty value is forced, which keeps embedded lists consistent. |
| `event_type_id` | Event Category | many_to_one to Event Template | yes on the template ticket, optional on the event ticket | none | stored | Owning template. Deletion behavior: cascade. |
| `seats_max` | Maximum Attendees | integer | no | 0 | stored | Number of seats of this ticket. Zero means unlimited. |
| `seats_limited` | Limit Attendees | boolean | — | — | derived, stored, read-only | True exactly when `seats_max` is non-zero. |
| `product_id` | Product | many_to_one to Product Variant | yes | the shipped Event Registration product | stored, indexed | The product used to sell the ticket. Only products whose service tracking is `event` may be chosen. |
| `currency_id` | Currency | many_to_one to Currency | — | — | derived from `product.currency` | Currency of `price`. |
| `price` | Price | decimal (product price precision) | no | see derivation | derived, stored, editable | The ticket price. Recomputed from the product: the sales price of the product is taken when it is non-zero; otherwise an empty price becomes zero. |
| `price_reduce` | Price Reduce | decimal (product price precision) | — | — | derived with elevated rights, per reader, not stored | `(1 − contextual discount of the product) × price`. The contextual discount is the discount the pricelist of the current reader grants on that product; see [Pricing and pricelists](../pricing-and-pricelists/calculations.md). |

**Whitelist copied onto an event ticket:** `sequence`, `name`, `description`, `seats_max`, plus `product_id` and `price` when the product bridge is installed.

---

# 15. Mail Scheduling on Event Category

**Full name:** Mail Scheduling on Event Category. **Transport name** `event.type.mail`, **storage name** `event_type_mail`. **Kind:** persistent entity. **Contributed by the capability package** `event`. Generated reference page: [`event.type.mail`](../../references/entities/event.type.mail.md). This is the communication schedule line of an Event Template.

| Field | Full name | Type | Required | Default | Stored | Meaning |
|---|---|---|---|---|---|---|
| `event_type_id` | Event Type | many_to_one to Event Template | yes | none | stored | Owning template. Deletion behavior: cascade. |
| `interval_nbr` | Interval | integer | no | 1 | stored | Same meaning as on Event Automated Mailing. |
| `interval_unit` | Unit | selection | yes | `hours` | stored | Same value list as on Event Automated Mailing. |
| `interval_type` | Trigger | selection | yes | `before_event` | stored | Same value list as on Event Automated Mailing. |
| `template_ref` | Template | reference | yes | none | stored | Same targets as on Event Automated Mailing. |
| `notification_type` | Send | selection | — | — | derived, not stored | `mail`, or `sms` when the reference points at a Text Message Template. |

The tuple copied onto an event is exactly (`interval_nbr`, `interval_unit`, `interval_type`, `template_ref`).

---

# 16. Event Booth Template

**Full name:** Event Booth Template. **Transport name** `event.type.booth`, **storage name** `event_type_booth`. **Kind:** persistent entity. **Contributed by the capability package** `event_booth`. Generated reference page: [`event.type.booth`](../../references/entities/event.type.booth.md). This is also the shared definition inherited by Event Booth.

| Field | Full name | Type | Required | Default | Stored | Meaning and rules |
|---|---|---|---|---|---|---|
| `name` | Name | text (translatable) | yes | none | stored | Booth label, for example a stand number. |
| `event_type_id` | Event Category | many_to_one to Event Template | yes on the template booth, optional on the booth | none | stored, indexed | Owning template. Deletion behavior: cascade on the template booth, set to empty on the booth. |
| `booth_category_id` | Booth Category | many_to_one to Event Booth Category | yes | the only existing category when exactly one exists, otherwise empty | stored, indexed | The class of booth. Deletion behavior: restrict. |
| `product_id` | Product | many_to_one to Product Variant | — | — | derived from `booth_category.product` | The product used to sell the booth. |
| `price` | Price | decimal | — | — | derived from `booth_category.price`, stored | The booth price. |
| `currency_id` | Currency | many_to_one to Currency | — | — | derived from `booth_category.currency` | Currency of the price. |

**Whitelist copied onto an event booth:** `name` and `booth_category_id`, plus `product_id` and `price` when the booth sales bridge is installed.

---

# 17. Event Booth Category

**Full name:** Event Booth Category. **Transport name** `event.booth.category`, **storage name** `event_booth_category`. **Kind:** persistent entity. **Contributed by the capability package** `event_booth`. Generated reference page: [`event.booth.category`](../../references/entities/event.booth.category.md). **Archivable.** Carries the standard image field set.

**Default ordering:** by `sequence` ascending.

| Field | Full name | Type | Required | Default | Stored | Visible to | Meaning and rules |
|---|---|---|---|---|---|---|---|
| `name` | Name | text (translatable) | yes | none | stored | everyone | Category label. |
| `sequence` | Sequence | integer | no | 10 | stored | everyone | Display order on the public booth page. |
| `description` | Description | rich_text (translatable) | no | none | stored | everyone | What the renter gets. Rich-text attributes are preserved. |
| `active` | Active | boolean | yes | true | stored | everyone | Archived categories are no longer offered. |
| `image_1920` | Image 1920 | image | no | see derivation | derived, stored, editable | everyone | Picture of the booth class. Recomputed from the product: when the category has no picture of its own, the product picture is used. |
| `booth_ids` | Booths | one_to_many to Event Booth | — | — | stored on the other side | Registration Desk | Booths of this class. |
| `product_id` | Product | many_to_one to Product Variant | yes | the shipped Event Booth product | stored | Registration Desk | The product used to sell the booth. Only products whose service tracking is `event_booth` may be chosen. |
| `price` | Price | decimal (product price precision) | no | see derivation | derived, stored, editable | Registration Desk | The booth price. Recomputed from the product: when the product has a non-zero sales price, the price becomes `product sales price + product extra price`. A category may therefore charge a different price from another category sharing the same product. |
| `price_incl` | Price incl | decimal (product price precision) | — | — | derived, editable, not stored | Registration Desk | `price` with the taxes of the product applied, for one unit, in the category currency. Zero when there is no product or no price. |
| `currency_id` | Currency | many_to_one to Currency | — | — | derived from `product.currency` | Registration Desk | Currency of the prices. |
| `price_reduce` | Price Reduce | decimal (product price precision) | — | — | derived with elevated rights, per reader, not stored | Registration Desk | `(1 − contextual discount of the product) × price`. |
| `price_reduce_taxinc` | Price Reduce Tax inc | decimal (product price precision) | — | — | derived with elevated rights, per reader, not stored | everyone | `price_reduce` with the taxes of the product applied. |
| `use_sponsor` | Create Sponsor | boolean | no | false | stored | everyone | When true, booking a booth of this class creates or reuses a Sponsor for the renter. |
| `sponsor_type_id` | Sponsor Level | many_to_one to Event Sponsor Level | no | none | stored | everyone | The sponsorship level given to that sponsor. |
| `exhibitor_type` | Sponsor Type | selection | no | none | stored | everyone | The sponsor kind given to that sponsor; the value list is the one of Event Sponsor (`sponsor`, `exhibitor`, `online`). |

**Constraint `_check_service_tracking`** on `product_id`: the chosen product must have its service tracking set to the booth value; otherwise the save is rejected with *"The product, <product name> , is used for Event Booth, it must have service_tracking set to "Event Booth"."*

**On-change `_onchange_use_sponsor`**: when the sponsor switch is turned on in the form and no sponsorship level is set, the level with the **highest** sequence is proposed; when no sponsor kind is set, the first value of the list is proposed.

---

# 18. Event Booth

**Full name:** Event Booth. **Transport name** `event.booth`, **storage name** `event_booth`. **Kind:** persistent entity. **Contributed by the capability package** `event_booth`. Generated reference page: [`event.booth`](../../references/entities/event.booth.md). **Carries a discussion thread.** Inherits the whole field set of Event Booth Template (section 16).

One physical booth of one event. Its whole life cycle is the two-value availability state.

| Field | Full name | Type | Required | Default | Stored | Copied | Tracked | Visible to | Meaning and rules |
|---|---|---|---|---|---|---|---|---|---|
| `event_id` | Event | many_to_one to Event | yes | none | stored, indexed | yes | no | everyone | Owning event. Deletion behavior: cascade. |
| `state` | Status | selection | yes | `available` | stored | yes | yes | everyone | Values: `available` (Available), `unavailable` (Unavailable). Grouping always shows both values. |
| `is_available` | Is Available | boolean | — | — | derived, not stored, searchable | — | — | everyone | True exactly when the state is `available`. A search rewrites `is_available in true` into `state = available` and `is_available not in true` into `state = unavailable`; other operators are refused. |
| `partner_id` | Renter | many_to_one to Contact | no | none | stored | **no** | yes | everyone | The renter. |
| `contact_name` | Renter Name | text | no | see derivation | derived, stored, editable | **no** | no | everyone | Renter name; filled from the renter contact only while empty. |
| `contact_email` | Renter Email | text | no | see derivation | derived, stored, editable | **no** | no | everyone | Renter electronic mail address; same rule. |
| `contact_phone` | Renter Phone | text | no | see derivation | derived, stored, editable | **no** | no | everyone | Renter telephone; same rule. |
| `event_booth_registration_ids` | Event Booth Registration | one_to_many to Event Booth Registration | — | — | stored on the other side | — | — | everyone | Pending reservations on this booth. |
| `sale_order_line_registration_ids` | sales order Lines with reservations | many_to_many to Sales Order Line | — | — | stored through the reservation table | **no** | — | Salesperson | The order lines that have a pending reservation on this booth. |
| `sale_order_line_id` | Final Sale Order Line | many_to_one to Sales Order Line | no | none | stored, indexed when set | **no** | no | Salesperson | The order line that finally won the booth. Deletion behavior: set to empty. |
| `sale_order_id` | Sale Order | many_to_one to Sales Order | — | — | derived from `sale_order_line.order`, stored, indexed when set | — | — | Salesperson | The winning order. |
| `is_paid` | Is Paid | boolean | no | false | stored | **no** | no | everyone | Stamped when the invoice carrying the booth is paid. |
| `use_sponsor` | Use Sponsor | boolean | — | — | derived from `booth_category.use_sponsor` | — | — | everyone | Whether booking creates a sponsor. |
| `sponsor_type_id` | Sponsor Type | many_to_one to Event Sponsor Level | — | — | derived from `booth_category.sponsor_type` | — | — | everyone | The level given to the created sponsor. |
| `sponsor_id` | Sponsor | many_to_one to Event Sponsor | no | none | stored | **no** | no | everyone | The sponsor created or reused when the booth was booked. |
| `sponsor_name`, `sponsor_email`, `sponsor_phone`, `sponsor_subtitle` | Sponsor Name, Sponsor Email, Sponsor Phone, Sponsor Slogan | text | — | — | derived from the matching field of `sponsor_id` | — | — | everyone | Convenience read-through. |
| `sponsor_website_description` | Sponsor Description | rich_text | — | — | derived from `sponsor.website_description` | — | — | everyone | Convenience read-through. |
| `sponsor_image_512` | Sponsor Logo | image | — | — | derived from `sponsor.image_512` | — | — | everyone | Convenience read-through. |

**Creation.** Booths are created without automatically subscribing the creator to their thread. Any booth created directly in state `unavailable` immediately posts the booking message described below.

**Write.** The set of booths that were `available` before the write is remembered. After the write, if the requested state was `unavailable`, the post-confirmation rule is run on exactly that set. A booth that was already unavailable therefore does not post a second message.

**Post-confirmation rule.**

1. When the booth category asks for a sponsor and the booth has a renter, a Sponsor is created or reused (section 21) and linked to the booth.
2. A message built from the booking layout is posted **on the event**, with the subtype "Booth Booked", naming the booth.

**Operation `action_confirm(additional values)`**: writes `state = unavailable` merged with the supplied values, in one write, which triggers the rule above with those values available to the sponsor creation.

**Operation `action_set_paid`**: writes `is_paid = true`.

**Deletion guard `_unlink_except_linked_sale_order`**: a booth attached to a sales order cannot be deleted. The message is *"You can't delete the following booths as they are linked to sales orders: <comma separated booth names>"*.

**Sales description of a group of booths:** `"<event display name> : \n"` followed by one line `- <booth name>` per booth.

---

# 19. Event Booth Registration

**Full name:** Event Booth Registration. **Transport name** `event.booth.registration`, **storage name** `event_booth_registration`. **Kind:** persistent entity. **Contributed by the capability package** `event_booth_sale`. Generated reference page: [`event.booth.registration`](../../references/entities/event.booth.registration.md).

A *pending* reservation: several customers may put the same booth in their basket, and the first one to confirm wins. This record is the intermediate reservation that carries the contact details until the booking is confirmed.

| Field | Full name | Type | Required | Default | Stored | Meaning and rules |
|---|---|---|---|---|---|---|
| `sale_order_line_id` | Sale Order Line | many_to_one to Sales Order Line | yes | none | stored, indexed | The line that holds the reservation. Deletion behavior: cascade. |
| `event_booth_id` | Booth | many_to_one to Event Booth | yes | none | stored, indexed | The booth being reserved. |
| `partner_id` | Partner | many_to_one to Contact | — | — | derived from `sale_order_line.order_customer`, stored | The customer. |
| `contact_name`, `contact_email`, `contact_phone` | Contact Name, Contact Email, Contact Phone | text | no | see derivation | derived, stored, editable | Filled from the customer contact only while empty. |
| `sponsor_name`, `sponsor_email`, `sponsor_phone`, `sponsor_subtitle` | Sponsor Name, Sponsor Email, Sponsor Phone, Sponsor Slogan | text | no | none | stored | Sponsor details captured on the public booking form. |
| `sponsor_website_description` | Sponsor Description | rich_text | no | none | stored | Sponsor description captured on the public booking form. |
| `sponsor_image_512` | Sponsor Logo | image | no | none | stored | Sponsor logo captured on the public booking form. |

**Database constraint `_unique_registration`:** `unique(sale_order_line_id, event_booth_id)` with the message *"There can be only one registration for a booth by sale order line"*.

**Confirmation fields.** The values handed to `action_confirm` of the booth are `sale_order_line_id`, `partner_id`, `contact_name`, `contact_email` and `contact_phone`; with the sponsor bridge installed, the six sponsor fields are appended.

**Operation `action_confirm`.** For each reservation, the collected values are written onto its booth through `action_confirm`, which marks the booth unavailable. Then every **other** pending reservation on the same booths is cancelled: a message is posted on each losing order, addressed to its salesperson, reading *"Your order has been cancelled because the following booths have been reserved"* followed by a bulleted list of the booth display names; each losing order is then cancelled, and the losing reservations are deleted.

---

# 20. Event Sponsor

**Full name:** Event Sponsor. **Transport name** `event.sponsor`, **storage name** `event_sponsor`. **Kind:** persistent entity. **Contributed by the capability package** `website_event_exhibitor`. Generated reference page: [`event.sponsor`](../../references/entities/event.sponsor.md). **Carries a discussion thread.** **Publishable.** **Archivable.**

A sponsor or exhibitor of one event, with its own public page.

**Default ordering:** by `sequence`, then by sponsorship level. **Display name:** the `name`. The public address of the page is `/event/<event slug>/exhibitor/<sponsor slug>`. Because the record has no website of its own, its base address is taken from its event.

| Field | Full name | Type | Required | Default | Stored | Meaning and rules |
|---|---|---|---|---|---|---|
| `event_id` | Event | many_to_one to Event | yes | none | stored, indexed | Owning event. |
| `partner_id` | Partner | many_to_one to Contact | yes | none | stored | The sponsoring company. Selection bypasses the ordinary contact search restrictions. |
| `sponsor_type_id` | Sponsorship Level | many_to_one to Event Sponsor Level | yes | the level with the highest sequence | stored | Sponsorship level. Selection bypasses the ordinary search restrictions. |
| `exhibitor_type` | Sponsor Type | selection | no | `sponsor` | stored | Values: `sponsor` (Footer Logo Only), `exhibitor` (Exhibitor), `online` (Online Exhibitor). Only the last two appear in the exhibitor list. |
| `sequence` | Sequence | integer | no | none | stored | Position inside the level. |
| `active` | Active | boolean | yes | true | stored | Archived sponsors disappear from the site. |
| `name` | Sponsor Name | text | no | see derivation | derived, stored, editable | Sponsor name; filled from the contact while empty. Writing it does **not** write back on the contact, because it may be event-specific. |
| `email` | Sponsor Email | text | no | see derivation | derived, stored, editable | Same rule. |
| `phone` | Sponsor Phone | text | no | see derivation | derived, stored, editable | Same rule. |
| `image_512` | Logo | image (at most 512 by 512) | no | see derivation | derived, stored, editable | Logo; same rule, taken from the contact image. |
| `image_256`, `image_128` | Image 256, Image 128 | image | — | — | derived from `image_512`, never stored | Smaller renderings. |
| `website_image_url` | Image uniform resource locator | text | — | — | derived with elevated rights, not stored | The logo address: the 256-pixel rendering of the sponsor when it has a logo, otherwise the 256-pixel rendering of the contact, otherwise the shipped default sponsor picture. |
| `subtitle` | Slogan | text | no | none | stored | Slogan. |
| `url` | Sponsor Website | text | no | see derivation | derived, stored, editable | Sponsor website. Recomputed from the contact: the contact website replaces the value whenever the contact has one, or when the field is empty. |
| `website_description` | Description | rich_text (translatable) | no | see derivation | derived, stored, editable | Public description; taken from the contact description while the value is empty. Embedded forms are sanitised, other rich-text attributes are preserved, and the field can be overridden by a user with sufficient rights. |
| `show_on_ticket` | Show on ticket | boolean | no | true | stored | Whether the sponsor logo is printed on the event tickets. |
| `hour_from` | Opening hour | decimal (hours, fractional) | no | 8.0 | stored | Daily opening hour of the virtual booth, expressed in the event time zone. |
| `hour_to` | End hour | decimal (hours, fractional) | no | 18.0 | stored | Daily closing hour, expressed in the event time zone. A value of zero is read as midnight of the following day. |
| `event_date_tz` | Timezone | selection | — | — | derived from `event.date_time_zone`, read-only | The zone of the two hours above. |
| `is_in_opening_hours` | Within opening hours | boolean | — | — | derived, not stored | Whether the virtual booth is open right now; see [calculations.md](calculations.md#10-sponsor-opening-hours). |
| `partner_name`, `partner_email`, `partner_phone` | Name, Email, Phone | text | — | — | derived from the contact | Read-through values shown next to the editable ones. |
| `country_id` | Country | many_to_one to Country | — | — | derived from `partner.country`, read-only | Country of the sponsor. |
| `country_flag_url` | Country Flag | text | — | — | derived with elevated rights, not stored | The flag picture address of that country, or empty. |

---

# 21. Event Sponsor Level

**Full name:** Event Sponsor Level. **Transport name** `event.sponsor.type`, **storage name** `event_sponsor_type`. **Kind:** persistent entity. **Contributed by the capability package** `website_event_exhibitor`. Generated reference page: [`event.sponsor.type`](../../references/entities/event.sponsor.type.md).

A sponsorship level.

**Default ordering:** by `sequence`. A **lower** sequence means a **higher** level: the shipped data gives Gold sequence 1, Silver 2 and Bronze 3.

| Field | Full name | Type | Required | Default | Stored | Meaning |
|---|---|---|---|---|---|---|
| `name` | Sponsor Level | text (translatable) | yes | none | stored | Level label. |
| `sequence` | Sequence | integer | no | the highest existing sequence plus one | stored | Ranking; lower is higher. |
| `display_ribbon_style` | Ribbon Style | selection | no | `no_ribbon` | stored | Values: `no_ribbon` (No Ribbon), `Gold` (Gold), `Silver` (Silver), `Bronze` (Bronze). Chooses the ribbon drawn on the sponsor card. |

---

# 22. Event Track

**Full name:** Event Track. **Transport name** `event.track`, **storage name** `event_track`. **Kind:** persistent entity. **Contributed by the capability package** `website_event_track`. Generated reference page: [`event.track`](../../references/entities/event.track.md). **Carries a discussion thread.** **Publishable.** **Archivable.** Its primary electronic mail address for the message gateway is `contact_email`.

One talk of the programme. Full behaviour is in [tracks-and-agenda.md](tracks-and-agenda.md).

**Default ordering:** by `priority` descending, then by `date` ascending. The public address is `/event/<event slug>/track/<track slug>`.

## 22.1 Description and management

| Field | Full name | Type | Required | Default | Stored | Tracked | Meaning and rules |
|---|---|---|---|---|---|---|---|
| `name` | Title | text (translatable) | yes | none | stored | no | Talk title. |
| `event_id` | Event | many_to_one to Event | yes | none | stored, indexed | no | Owning event. |
| `active` | Active | boolean | yes | true | stored | no | Archived talks disappear. |
| `user_id` | Responsible | many_to_one to User | no | the creating user | stored | yes | Internal person responsible for the talk. A talk proposed from the public site is created with **no** responsible. |
| `company_id` | Company | many_to_one to Company | — | — | derived from `event.company` | — | Owning company. |
| `description` | Description | rich_text (translatable) | no | none | stored | no | Abstract. Rich-text attributes and embedded forms are preserved. |
| `tag_ids` | Tags | many_to_many to Event Track Tag | no | none | stored | no | Labels used by the public filters. |
| `color` | Agenda Color | integer | no | none | stored | no | Colour of the talk block in the agenda. |
| `priority` | Priority | selection | yes | `1` | stored | no | Values: `0` (Low), `1` (Medium), `2` (High), `3` (Highest). Drives the default ordering. |
| `stage_id` | Stage | many_to_one to Event Track Stage | yes | the first stage by sequence | stored, indexed | yes | Review stage. Deletion behavior: restrict. Grouping always shows every stage. Not copied on duplication. |
| `kanban_state` | Kanban State | selection | yes | `normal` | stored | no | Values: `normal` (Grey), `done` (Green), `blocked` (Red). Not copied on duplication. Reset to `normal` whenever the stage changes without an explicit new value. |
| `kanban_state_label` | Kanban State Label | text | — | — | derived, stored | yes | The readable legend of the current kanban state, taken from the stage: the red legend when blocked, the green legend when done, the grey legend otherwise. |
| `legend_blocked`, `legend_done`, `legend_normal` | Kanban Blocked Explanation, Kanban Valid Explanation, Kanban Ongoing Explanation | text | — | — | derived from the stage, read-only | — | The three legends of the current stage. |

## 22.2 Speaker and contact

| Field | Full name | Type | Required | Default | Stored | Tracked | Meaning and rules |
|---|---|---|---|---|---|---|---|
| `partner_id` | Contact | many_to_one to Contact | no | none | stored | no | The speaker contact, when one exists. |
| `partner_name` | Name | text | no | see derivation | derived, stored, editable | yes | Speaker name; filled from the contact while empty. |
| `partner_email` | Email | text | no | see derivation | derived, stored, editable | yes | Speaker electronic mail address; filled from the contact while empty. |
| `partner_phone` | Phone | text | no | see derivation | derived, stored, editable | yes | Speaker telephone; filled from the contact while empty. |
| `partner_biography` | Biography | rich_text | no | see derivation | derived, stored, editable | no | Speaker biography; taken from the contact public description when empty. |
| `partner_function` | Job Position | text | no | see derivation | derived, stored, editable | no | Speaker job title; filled from the contact while empty. |
| `partner_company_name` | Company Name | text | no | see derivation | derived, stored, editable | no | Speaker company. When the contact is itself a company, its own name is used; otherwise, while empty, the name of its parent company is used. |
| `partner_tag_line` | Tag Line | text | — | — | derived, not stored | — | One-line speaker description; see [tracks-and-agenda.md](tracks-and-agenda.md#speaker-tag-line). |
| `image` | Speaker Photo | image (at most 256 by 256) | no | see derivation | derived, stored, editable | no | Speaker photograph; taken from the contact 256-pixel image while empty. |
| `contact_email` | Contact Email | text | no | see derivation | derived, stored, editable | yes | Operational electronic mail address. Unlike the speaker address, this one is **overwritten** by the contact address whenever a contact is set. |
| `contact_phone` | Contact Phone | text | no | see derivation | derived, stored, editable | yes | Operational telephone, with the same overwrite rule. |

## 22.3 Time, place and live state

| Field | Full name | Type | Required | Default | Stored | Meaning and rules |
|---|---|---|---|---|---|---|
| `location_id` | Location | many_to_one to Event Track Location | no | none | stored | The room. |
| `date` | Track Date | datetime | no | see derivation | derived, stored, reversible | Start. Derived as `date_end − duration` when an end is known; writing it recomputes `duration` from the two datetimes. |
| `date_end` | Track End Date | datetime | no | see derivation | derived, stored, reversible | End. Derived as `date + duration`; writing it recomputes `duration`. |
| `duration` | Duration | decimal (hours) | no | 0.5 | stored | Length in hours. |
| `is_track_live` | Is Track Live | boolean | — | — | derived, not stored | `date <= now < date_end`. |
| `is_track_soon` | Is Track Soon | boolean | — | — | derived, not stored | True when the talk has not started and starts in less than thirty minutes. |
| `is_track_today` | Is Track Today | boolean | — | — | derived, not stored | The start falls on the current calendar day in coordinated universal time. |
| `is_track_upcoming` | Is Track Upcoming | boolean | — | — | derived, not stored | `date > now`. |
| `is_track_done` | Is Track Done | boolean | — | — | derived, not stored | `date_end <= now`. |
| `is_one_day` | Is One Day | boolean | — | — | derived, not stored | Start and end fall on the same day when read in the event time zone. |
| `track_start_remaining` | Minutes before track starts | integer (seconds) | — | — | derived, not stored | Seconds left before the start; zero once started. |
| `track_start_relative` | Minutes compare to track start | integer (seconds) | — | — | derived, not stored | Seconds to the start when upcoming, seconds since the start when running or finished. |

When neither `date` nor `date_end` is set, all five booleans are false and both counters are zero.

## 22.4 Public presentation, wish list, call to action, video and quiz

| Field | Full name | Type | Required | Default | Stored | Meaning and rules |
|---|---|---|---|---|---|---|
| `website_image` | Website Image | image (at most 1024 by 1024) | no | none | stored | Picture of the talk. |
| `website_image_url` | Image uniform resource locator | text | — | — | derived with elevated rights, not stored | Address of the picture; see [tracks-and-agenda.md](tracks-and-agenda.md#talk-picture-address). |
| `header_visible`, `footer_visible` | Header Visible, Footer Visible | boolean | — | — | derived from the event, editable | Whether the site header and footer are shown on the talk page. |
| `event_track_visitor_ids` | Track Visitors | one_to_many to Track / Visitor Link | — | — | stored on the other side | Visitor links. Visible to the Event User group. |
| `wishlist_visitor_ids` | Visitor Wishlist | many_to_many to Website Visitor | — | — | derived with elevated rights, not stored, searchable | The visitors who wish-listed the talk. Visible to the Event User group. A "not in" search is refused with *"Unsupported 'Not In' operation on track wishlist visitors"*. |
| `wishlist_visitor_count` | # Wishlisted | integer | — | — | derived with elevated rights, not stored | How many. Visible to the Event User group. |
| `wishlisted_by_default` | Always Wishlisted | boolean | no | false | stored | A key talk: wish-listed for every attendee unless they opt out. |
| `is_reminder_on` | Is Reminder On | boolean | — | — | derived per reader, not stored | Whether the current reader has a reminder on this talk; see [tracks-and-agenda.md](tracks-and-agenda.md#reminder-state). |
| `website_cta` | Magic Button | boolean | no | false | stored | Show an action button while the talk plays. |
| `website_cta_title` | Button Title | text | no | none | stored | Button label. |
| `website_cta_url` | Button Target uniform resource locator | text | no | none | stored | Button target. Cleaned into a canonical web address on create and on write. |
| `website_cta_delay` | Show Button | integer (minutes) | no | none | stored | Minutes after the start before the button appears. |
| `is_website_cta_live` | Is call to action Live | boolean | — | — | derived, not stored | True when the button is enabled and `date + delay <= now <= date_end`. |
| `website_cta_start_remaining` | Minutes before call to action starts | integer (seconds) | — | — | derived, not stored | Seconds before the button appears; zero once it is live. |
| `youtube_video_url` | YouTube Video Link | text | no | none | stored | Link to the streamed or recorded video. |
| `youtube_video_id` | YouTube video identifier | text | — | — | derived, not stored | The eleven-character video code extracted from that link; empty when the link does not match. |
| `is_youtube_replay` | Is YouTube Replay | boolean | no | false | stored | Marks the video as a recording, which hides the live-only elements. |
| `is_youtube_chat_available` | Is Chat Available | boolean | — | — | derived, not stored | True when a video link exists, it is not a replay, and the talk is either live or starting soon. |
| `quiz_ids` | Quizzes | one_to_many to Quiz | — | — | stored on the other side | The quizzes attached to the talk. |
| `quiz_id` | Quiz | many_to_one to Quiz | — | — | derived, stored | The first quiz of the list. Visible to the Event User group. |
| `quiz_questions_count` | # Quiz Questions | integer | — | — | derived, not stored | Number of questions of that quiz. Visible to the Event User group. |
| `is_quiz_completed` | Is Quiz Done | boolean | — | — | derived per reader, not stored | Whether the current reader finished the quiz. |
| `quiz_points` | Quiz Points | integer | — | — | derived per reader, not stored | Points the current reader obtained. |

## 22.5 Creation, write and stage synchronisation

**On creation**, for every talk: a message built from the "new talk" layout is posted **on the event** with the subtype "New Track", and the stage synchronisation below is applied.

**On write**, if a stage is given and no kanban state is given, the kanban state is forced to `normal`; then the stage synchronisation is applied.

**Stage synchronisation.** When a talk enters a stage flagged "fully accessible", the talk is published. When it enters a stage flagged "cancelled stage", the talk is unpublished.

**Stage message.** Moving to a stage that carries an electronic mail template sends that template to the speaker as an internal note, using the light notification layout.

**Thread subtypes.** Setting the kanban state to `blocked` posts under the subtype "Track Blocked"; setting it to `done` posts under the subtype "Track Ready".

---

# 23. Event Track Stage

**Full name:** Event Track Stage. **Transport name** `event.track.stage`, **storage name** `event_track_stage`. **Kind:** persistent entity. **Contributed by the capability package** `website_event_track`. Generated reference page: [`event.track.stage`](../../references/entities/event.track.stage.md).

**Default ordering:** by `sequence`, then identifier.

| Field | Full name | Type | Required | Default | Stored | Meaning and rules |
|---|---|---|---|---|---|---|
| `name` | Stage Name | text (translatable) | yes | none | stored | Stage label. |
| `sequence` | Sequence | integer | no | 1 | stored | Column position. |
| `description` | Description | long_text (translatable) | no | none | stored | Help text of the column. |
| `color` | Color | integer | no | none | stored | Column colour. |
| `mail_template_id` | Email Template | many_to_one to Email Template | no | none | stored | Message sent to the speaker when a talk reaches this stage. Only templates written on Event Track may be chosen. |
| `legend_blocked` | Red Kanban Label | text (translatable) | no | `Blocked` | stored | Legend of the red kanban state. |
| `legend_done` | Green Kanban Label | text (translatable) | no | `Ready for Next Stage` | stored | Legend of the green kanban state. |
| `legend_normal` | Grey Kanban Label | text (translatable) | no | `In Progress` | stored | Legend of the grey kanban state. |
| `fold` | Folded in Kanban | boolean | no | false | stored | Collapse the column when empty. |
| `is_cancel` | Cancelled Stage | boolean | no | false | stored | Marks the stage as a refusal or cancellation. |
| `is_visible_in_agenda` | Visible in agenda | boolean | no | see derivation | derived, stored, editable | Talks in this stage appear in the public agenda even when unpublished. Recomputed: a cancelling stage forces false; a fully accessible stage forces true. |
| `is_fully_accessible` | Fully accessible | boolean | no | see derivation | derived, stored, editable | Talks entering this stage are published, which gives the speaker and the public a working link. Recomputed: a cancelling stage or a stage not visible in the agenda forces false. |

The two derived flags form a ladder: cancelled excludes both, fully accessible implies visible in agenda, and visible in agenda does not imply fully accessible.

---

# 24. Event Track Location, Event Track Tag and Event Track Tag Category

## 24.1 Event Track Location

**Full name:** Event Track Location. **Transport name** `event.track.location`, **storage name** `event_track_location`. **Kind:** persistent entity. **Contributed by the capability package** `website_event_track`. Generated reference page: [`event.track.location`](../../references/entities/event.track.location.md). **Default ordering:** by `sequence`, then identifier.

| Field | Full name | Type | Required | Default | Stored | Meaning |
|---|---|---|---|---|---|---|
| `name` | Location | text | yes | none | stored | Room or stage name. |
| `sequence` | Sequence | integer | no | 10 | stored | The order in which rooms appear as columns of the agenda. |

## 24.2 Event Track Tag Category

**Full name:** Event Track Tag Category. **Transport name** `event.track.tag.category`, **storage name** `event_track_tag_category`. **Kind:** persistent entity. **Contributed by the capability package** `website_event_track`. Generated reference page: [`event.track.tag.category`](../../references/entities/event.track.tag.category.md). **Default ordering:** by `sequence`.

| Field | Full name | Type | Required | Default | Stored | Meaning |
|---|---|---|---|---|---|---|
| `name` | Name | text (translatable) | yes | none | stored | Category label, used as the heading of a filter block on the talk list. |
| `sequence` | Sequence | integer | no | 10 | stored | Block position. |
| `tag_ids` | Tags | one_to_many to Event Track Tag | — | — | stored on the other side | Tags of the category. |

## 24.3 Event Track Tag

**Full name:** Event Track Tag. **Transport name** `event.track.tag`, **storage name** `event_track_tag`. **Kind:** persistent entity. **Contributed by the capability package** `website_event_track`. Generated reference page: [`event.track.tag`](../../references/entities/event.track.tag.md). **Default ordering:** by category, then `sequence`, then `name`.

| Field | Full name | Type | Required | Default | Stored | Meaning |
|---|---|---|---|---|---|---|
| `name` | Tag Name | text | yes | none | stored | Tag label. |
| `sequence` | Sequence | integer | no | 10 | stored | Position inside the category. |
| `category_id` | Category | many_to_one to Event Track Tag Category | no | none | stored, indexed when set | Owning category. Deletion behavior: set to empty. |
| `color` | Color Index | integer | no | a pseudo-random value between 1 and 11 inclusive | stored | Display colour. **A zero or empty colour hides the tag from the public site.** |
| `track_ids` | Tracks | many_to_many to Event Track | — | — | stored | Talks carrying the tag. |

**Database constraint `_name_uniq`:** `unique(name)` with the message *"Tag name already exists!"* Talk tag names are therefore unique across the whole database, not per category or per event.

---

# 25. Track / Visitor Link

**Full name:** Track / Visitor Link. **Transport name** `event.track.visitor`, **storage name** `event_track_visitor`. **Kind:** persistent entity. **Contributed by the capability package** `website_event_track`. Generated reference page: [`event.track.visitor`](../../references/entities/event.track.visitor.md).

The link between one talk and one site visitor. It carries the wish list, the opt-out from a key talk, and the quiz result.

**Default ordering:** by talk. Its readable name is the talk.

| Field | Full name | Type | Required | Default | Stored | Meaning and rules |
|---|---|---|---|---|---|---|
| `track_id` | Track | many_to_one to Event Track | yes | none | stored, indexed | The talk. Deletion behavior: cascade. |
| `visitor_id` | Visitor | many_to_one to Website Visitor | no | none | stored, indexed | The visitor. Deletion behavior: cascade. |
| `partner_id` | Partner | many_to_one to Contact | no | see derivation | derived, stored, editable, indexed | The contact behind the visitor. Filled from `visitor.partner` while empty; forced empty when neither is known. Deletion behavior: set to empty. |
| `is_wishlisted` | Is Wishlisted | boolean | no | false | stored | The visitor asked for a reminder on an ordinary talk. |
| `is_blacklisted` | Is reminder off | boolean | no | false | stored | The visitor switched the reminder **off** on a key talk. A key talk cannot be removed from the wish list, therefore the opt-out is stored separately. |
| `quiz_completed` | Completed | boolean | no | false | stored | The visitor finished the quiz of that talk. |
| `quiz_points` | Quiz Points | integer | no | 0 | stored | Points obtained. |

---

# 26. Quiz, Content Quiz Question and Question's Answer

## 26.1 Quiz

**Full name:** Quiz. **Transport name** `event.quiz`, **storage name** `event_quiz`. **Kind:** persistent entity. **Contributed by the capability package** `website_event_track_quiz`. Generated reference page: [`event.quiz`](../../references/entities/event.quiz.md).

| Field | Full name | Type | Required | Default | Stored | Meaning |
|---|---|---|---|---|---|---|
| `name` | Name | text (translatable) | yes | none | stored | Quiz title. |
| `event_track_id` | Event Track | many_to_one to Event Track | no | none | stored, read-only, indexed when set | The talk the quiz belongs to. |
| `event_id` | Event | many_to_one to Event | — | — | derived from `event_track.event`, stored, read-only | The event, kept stored for the leaderboard query. |
| `repeatable` | Unlimited Tries | boolean | no | false | stored | Whether a visitor may reset the quiz and try again. |
| `question_ids` | Questions | one_to_many to Content Quiz Question | — | — | stored on the other side | The questions. |

## 26.2 Content Quiz Question

**Full name:** Content Quiz Question. **Transport name** `event.quiz.question`, **storage name** `event_quiz_question`. **Kind:** persistent entity. **Contributed by the capability package** `website_event_track_quiz`. Generated reference page: [`event.quiz.question`](../../references/entities/event.quiz.question.md). **Default ordering:** by quiz, then `sequence`, then identifier.

| Field | Full name | Type | Required | Default | Stored | Meaning |
|---|---|---|---|---|---|---|
| `name` | Question | text (translatable) | yes | none | stored | The question. |
| `sequence` | Sequence | integer | no | none | stored | Display order. |
| `quiz_id` | Quiz | many_to_one to Quiz | yes | none | stored, indexed | Owning quiz. Deletion behavior: cascade. |
| `answer_ids` | Answer | one_to_many to Question's Answer | — | — | stored on the other side | The proposed answers. |
| `correct_answer_id` | Correct Answer | list of Question's Answer | — | — | derived, not stored | The subset of the answers flagged correct. |
| `awarded_points` | Number of Points | integer | — | — | derived, not stored | The sum of the points of **all** answers of the question. |

**Constraint `_check_answers_integrity`** on the answer list, in this order:

1. There must be exactly one correct answer: otherwise *"Question "<question name>" must have 1 correct answer to be valid."*
2. There must be at least two answers in total: otherwise *"Question "<question name>" must have 1 correct answer and at least 1 incorrect answer to be valid."*

## 26.3 Question's Answer

**Full name:** Question's Answer. **Transport name** `event.quiz.answer`, **storage name** `event_quiz_answer`. **Kind:** persistent entity. **Contributed by the capability package** `website_event_track_quiz`. Generated reference page: [`event.quiz.answer`](../../references/entities/event.quiz.answer.md). **Default ordering:** by question, then `sequence`, then identifier. Its readable name is `text_value`.

| Field | Full name | Type | Required | Default | Stored | Meaning |
|---|---|---|---|---|---|---|
| `question_id` | Question | many_to_one to Content Quiz Question | yes | none | stored, indexed | Owning question. Deletion behavior: cascade. |
| `sequence` | Sequence | integer | no | none | stored | Display order. |
| `text_value` | Answer | text (translatable) | yes | none | stored | The proposed answer. |
| `is_correct` | Correct | boolean | no | false | stored | Whether this answer is the correct one. |
| `awarded_points` | Points | integer | no | 0 | stored | Points granted when the visitor picks this answer. Points may be granted for a wrong answer as well; nothing forbids it. |
| `comment` | Extra Comment | long_text (translatable) | no | none | stored | Explanation shown to the visitor after submission when this answer was picked. |

---

# 27. Website Event Menu

**Full name:** Website Event Menu. **Transport name** `website.event.menu`, **storage name** `website_event_menu`. **Kind:** persistent entity. **Contributed by the capability package** `website_event`. Generated reference page: [`website.event.menu`](../../references/entities/website.event.menu.md). Carries search-engine metadata. Its readable name is its menu entry.

The link between one event, one menu entry of the site and, for page-backed entries, the view that renders that page.

| Field | Full name | Type | Required | Default | Stored | Meaning and rules |
|---|---|---|---|---|---|---|
| `menu_id` | Menu | many_to_one to Website Menu | no | none | stored | The menu entry. Deletion behavior: cascade. |
| `event_id` | Event | many_to_one to Event | no | none | stored, indexed when set | The event. Deletion behavior: cascade. |
| `view_id` | View | many_to_one to view definition | no | none | stored | The view rendering the page, for entries that are pages rather than links. Deletion behavior: cascade. |
| `menu_type` | Menu Type | selection | yes | none | stored | Values: `introduction` (Home), `register` (Practical), `community` (Community Menu), `other` (Other), plus `booth` (Event Booth Menus), `exhibitor` (Exhibitors Menus), `track` (Event Tracks Menus) and `track_proposal` (Event Proposals Menus) when the matching capability packages are installed. Deleting a booth-type entry cascades. |

**Deletion.** Deleting a Website Event Menu first deletes its view with elevated rights, which in turn cascades to the page and to the underlying menu entry.

**Duplication.** Copying a Website Event Menu that has a view copies the latest view carrying the same key for the website of the event, giving the copy a key suffixed with `-t<current epoch second>` to keep it unique and to stop the address parser from reading the trailing number as a record identifier. The children of the view are copied recursively with fresh unique keys. The address of the new menu entry becomes `/event/<new event slug>/page/<last part of the new view key>`. When the source entry had a page, the page is copied onto the new address as well.

**Menu entries created per event.** For every switch that is on, these entries are created under the root menu of the event:

| Menu type | Label | Sequence | Target |
|---|---|---|---|
| `introduction` | Home | 1 | A page duplicated from the shipped introduction layout. |
| `track` | Talks | 10 | A placeholder parent entry with target `#`. |
| `track` | Talks | 10 | `/event/<slug>/track`, created under the parent above. |
| `track` | Agenda | 15 | `/event/<slug>/agenda`, created under the same parent. |
| `track_proposal` | Propose a talk | 20 | `/event/<slug>/track_proposal`, created under the talks parent. |
| `exhibitor` | Exhibitors list | 60 | `/event/<slug>/exhibitors`. |
| `community` | Rooms | 80 | `/event/<slug>/community`. |
| `booth` | Become exhibitor | 90 | `/event/<slug>/booth`. |
| `register` | Practical | 100 | `/event/<slug>/register`. |

**Menu entry added by a site editor.** When an editor adds a menu entry under the root menu of an event through the site menu editor, the address of the new entry is rewritten to `/event/<event slug>/page/<entered address>` (or `/page/t<current epoch second>` when the entered address is blank or a bare anchor), and a Website Event Menu of type `other` is created for it with elevated rights.

**Deleting a menu entry from the site editor** switches off the matching boolean on the event: deleting the Home entry clears `introduction_menu`, deleting Practical clears `register_menu`, deleting Rooms clears `community_menu`, deleting Become exhibitor clears `booth_menu`, deleting Exhibitors list clears `exhibitor_menu`, and deleting Propose a talk clears `website_track_proposal`.

---

# 28. Event Sales Report

**Full name:** Event Sales Report. **Transport name** `event.sale.report`, **storage name** `event_sale_report`. **Kind:** read-only database view. **Contributed by the capability package** `event_sale`. Generated reference page: [`event.sale.report`](../../references/entities/event.sale.report.md). Its readable name is the sales order line.

One row per registration, joined with its event, slot, ticket, sales order and order line. It exists to analyse how many seats were sold, at which price and by whom.

| Field | Full name | Type | Source |
|---|---|---|---|
| `event_registration_id` | Event Registration | many_to_one to Event Registration | The registration. |
| `event_registration_name` | Attendee Name | text | Attendee name. |
| `event_registration_state` | Registration Status | selection | Values `draft` (Unconfirmed), `cancel` (Cancelled), `open` (Confirmed), `done` (Attended). |
| `event_registration_create_date` | Registration Date | date | The day the registration was created. |
| `active` | Is registration active (not archived)? | boolean | Whether the registration is not archived. |
| `company_id` | Company | many_to_one to Company | Company of the registration. |
| `event_id` | Event | many_to_one to Event | The event. |
| `event_type_id` | Event Type | many_to_one to Event Template | Template of the event. |
| `event_date_begin`, `event_date_end` | Event Start Date, Event End Date | date | The event dates. |
| `event_slot_id` | Event Slot | many_to_one to Event Slot | Slot of the registration. |
| `event_ticket_id` | Event Ticket | many_to_one to Event Ticket | Ticket of the registration. |
| `event_ticket_price` | Ticket price | decimal | The list price stored on the ticket. |
| `product_id` | Product | many_to_one to Product Variant | Product of the order line. |
| `sale_order_id` | Sale Order | many_to_one to Sales Order | The order. |
| `sale_order_line_id` | Sale Order Line | many_to_one to Sales Order Line | The order line. |
| `sale_order_date` | Order Date | datetime | Order date. |
| `sale_order_partner_id` | Customer | many_to_one to Contact | Customer. |
| `invoice_partner_id` | Invoice Address | many_to_one to Contact | Invoice address of the order. |
| `sale_order_user_id` | Salesperson | many_to_one to User | Salesperson. |
| `sale_order_state` | Sale Order Status | selection | Order state. |
| `sale_status` | Payment Status | selection | `to_pay` (Not Sold), `sold` (Sold), `free` (Free). |
| `sale_price` | Revenues | decimal | Revenue including tax attributed to this one seat. |
| `sale_price_untaxed` | Untaxed Revenues | decimal | Revenue excluding tax attributed to this one seat. |
| `is_published` | Published Events | boolean | Whether the event is published (added by the online ticketing package). |

**Per-seat revenue formula** (both variants):

```formula
value = 0                                                    when order line quantity = 0
value = order line total (including tax, or excluding tax
        for the untaxed measure)
        ÷ (order currency rate, replaced by 1.0 when it is
           zero or absent)
        ÷ order line quantity                                otherwise
```

Dividing by the order currency rate expresses the amount in the company currency of the order. The view is restricted per company by the record rule "Event Sales Report multi-company".

---

# 29. Wizards

## 29.1 Event Configurator

**Full name:** Event Configurator. **Transport name** `event.event.configurator`, **storage name** `event_event_configurator`. **Kind:** transient entity. **Contributed by the capability package** `event_sale`. Generated reference page: [`event.event.configurator`](../../references/entities/event.event.configurator.md).

Opened when a product whose service tracking is `event_id` is put on a sales order line, to pick what exactly is being sold.

| Field | Full name | Type | Required | Default | Stored | Meaning and rules |
|---|---|---|---|---|---|---|
| `product_id` | Product | many_to_one to Product Variant | no | the product of the line | stored, read-only | The ticket product. |
| `event_id` | Event | many_to_one to Event | no | none | stored | The event. |
| `is_multi_slots` | Is Multi Slots | boolean | — | — | derived from `event.is_multi_slots` | Drives the visibility of the slot field. |
| `event_slot_id` | Slot | many_to_one to Event Slot | no | see derivation | derived, stored, editable | The slot, restricted to the slots of the chosen event. Recomputed when the multi-slot flag changes: cleared for a single-slot event; pre-filled when the event has exactly one slot. |
| `event_ticket_id` | Ticket Type | many_to_one to Event Ticket | no | see derivation | derived, stored, editable | The ticket, restricted to the tickets of the chosen event. Recomputed when the event changes: pre-filled when the event has exactly one ticket using that product. |
| `has_available_tickets` | Has Available Tickets | boolean | — | — | derived, not stored | True when at least one ticket using this product belongs to an event whose end date is today or later. |

**Constraint `check_event_id`** on the three links, collecting every failure and raising them together, one per line:

- When the ticket does not belong to the event: *"Invalid ticket choice "<ticket name>" for event "<event name>"."*
- When a slot is chosen and does not belong to the event: *"Invalid slot choice "<slot name>" for event "<event name>"."*

## 29.2 Event Booth Configurator

**Full name:** Event Booth Configurator. **Transport name** `event.booth.configurator`, **storage name** `event_booth_configurator`. **Kind:** transient entity. **Contributed by the capability package** `event_booth_sale`. Generated reference page: [`event.booth.configurator`](../../references/entities/event.booth.configurator.md).

| Field | Full name | Type | Required | Default | Stored | Meaning and rules |
|---|---|---|---|---|---|---|
| `product_id` | Product | many_to_one to Product Variant | no | the product of the line | stored, read-only | The booth product. |
| `sale_order_line_id` | Sale Order Line | many_to_one to Sales Order Line | no | the line being configured | stored, read-only | The line. |
| `event_id` | Event | many_to_one to Event | yes | none | stored | The event. |
| `event_booth_category_available_ids` | Event Booth Category Available | many_to_many to Event Booth Category | — | — | derived from `event.available_booth_categories`, read-only | Categories that still have a free booth. |
| `event_booth_category_id` | Booth Category | many_to_one to Event Booth Category | yes | none | derived, stored, editable | The class of booth. Cleared whenever the event changes. |
| `event_booth_ids` | Booth | many_to_many to Event Booth | yes | none | derived, stored, editable | The booths chosen. Cleared whenever the event or the category changes. |

**Constraint `_check_if_no_booth_ids`:** at least one booth must be selected, otherwise *"You have to select at least one booth."*

## 29.3 Edit Attendee Details on Sales Confirmation

**Full name:** Edit Attendee Details on Sales Confirmation. **Transport name** `registration.editor`, **storage name** `registration_editor`. **Kind:** transient entity. **Contributed by the capability package** `event_sale`. Generated reference page: [`registration.editor`](../../references/entities/registration.editor.md).

| Field | Full name | Type | Required | Default | Meaning |
|---|---|---|---|---|---|
| `sale_order_id` | Sales Order | many_to_one to Sales Order | yes | the order in context | The order being confirmed. Deletion behavior: cascade. |
| `event_registration_ids` | Registrations to Edit | one_to_many to Edit Attendee Line on Sales Confirmation | no | built as described below | One line per seat to fill in. |

**Building the lines.** The wizard searches every registration of the order that is not cancelled and whose slot and ticket appear on the order lines. Then, for every order line that carries a ticket:

1. One line is added per existing registration of that order line, pre-filled with the event, slot, ticket, registration, name, electronic mail address and telephone of the registration.
2. Additional empty lines are added until their number equals the quantity of the order line. Each of these is pre-filled with the event, slot and ticket of the order line and with the name, electronic mail address and telephone of the **order customer**.

**Operation `action_make_registration`.** For each line: an existing registration is updated with the prepared values; a line without a registration creates a new one with the same values plus the event, slot, ticket, order and order line. Afterwards the state derivation is forced on the touched registrations, which performs the seat verification and releases the pending communications immediately rather than at the next scheduler pass. The wizard then closes.

## 29.4 Edit Attendee Line on Sales Confirmation

**Full name:** Edit Attendee Line on Sales Confirmation. **Transport name** `registration.editor.line`, **storage name** `registration_editor_line`. **Kind:** transient entity. **Contributed by the capability package** `event_sale`. Generated reference page: [`registration.editor.line`](../../references/entities/registration.editor.line.md). **Default ordering:** by identifier descending.

| Field | Full name | Type | Required | Default | Meaning |
|---|---|---|---|---|---|
| `editor_id` | Editor | many_to_one to the wizard above | no | none | The owning wizard. |
| `sale_order_line_id` | Sales Order Line | many_to_one to Sales Order Line | no | none | The line being filled. |
| `event_id` | Event | many_to_one to Event | yes | none | The event. |
| `company_id` | Company | many_to_one to Company | — | — | Derived from `event.company`. |
| `registration_id` | Original Registration | many_to_one to Event Registration | no | none | The registration being edited, when it already exists. |
| `event_slot_id` | Event Slot | many_to_one to Event Slot | no | none | The slot. |
| `event_ticket_id` | Event Ticket | many_to_one to Event Ticket | no | none | The ticket. |
| `name` | Name | text | no | the customer name | Attendee name. |
| `email` | Email | text | no | the customer electronic mail address | Attendee electronic mail address. |
| `phone` | Phone | text | no | the customer telephone | Attendee telephone. |

**Prepared values.** `partner_id` becomes the order customer; `name`, `phone` and `email` become the typed values or, when blank, the matching values of the order customer.

---

# 30. Lead generation bridge

These two entities turn attendees into leads. The Lead itself belongs to the [Customer Relationship Management](../customer-relationship-management/README.md) domain; what is described here is only the rule engine driven by registrations.

## 30.1 Event Lead Rules

**Full name:** Event Lead Rules. **Transport name** `event.lead.rule`, **storage name** `event_lead_rule`. **Kind:** persistent entity. **Contributed by the capability package** `event_crm`. Generated reference page: [`event.lead.rule`](../../references/entities/event.lead.rule.md). **Archivable.**

| Field | Full name | Type | Required | Default | Stored | Meaning and rules |
|---|---|---|---|---|---|---|
| `name` | Rule Name | text (translatable) | yes | none | stored | Rule label. |
| `active` | Active | boolean | yes | true | stored | Inactive rules never run. |
| `lead_creation_basis` | Create | selection | yes | `attendee` | stored | Values: `attendee` (Per Attendee, one lead per seat) and `order` (Per Order, one lead per batch of seats). The field is hidden on the rule form and in the rule list unless the Event Lead Generation with Sales package or the Website Event Lead Generation package is installed; with neither of them, every rule is per attendee. |
| `lead_creation_trigger` | When | selection | yes | `create` | stored | Values: `create` (Attendees are created), `confirm` (Attendees are registered), `done` (Attendees attended). |
| `event_type_ids` | Event Templates | many_to_many to Event Template | no | none | stored | Restrict to attendees of events built on these templates. Empty means no restriction. |
| `event_id` | Event | many_to_one to Event | no | none | stored | Restrict to attendees of this event. Only events of the rule company (or of no company) may be chosen. Empty means no restriction. |
| `company_id` | Company | many_to_one to Company | no | none | stored | Restrict to events of this company. Empty means no restriction. |
| `event_registration_filter` | Registrations Domain | long_text | no | none | stored | A stored condition applied to the attendees, in the language-neutral condition notation. The text `[]` is treated as "no condition". |
| `lead_type` | Lead Type | selection | yes | `lead` when the reader uses the separate lead stage, otherwise `opportunity` | stored | The kind of record produced. |
| `lead_sales_team_id` | Sales Team | many_to_one to Sales Team | no | none | stored | Team put on the produced lead. Deletion behavior: set to empty. |
| `lead_user_id` | Salesperson | many_to_one to User | no | none | stored | Salesperson put on the produced lead. |
| `lead_tag_ids` | Tags | many_to_many to Lead Tag | no | none | stored | Tags put on the produced lead. |
| `lead_ids` | Created Leads | one_to_many to Lead | — | — | stored on the other side | Leads created by this rule. Visible to the Salesperson group. |

**On-change `_onchange_lead_sales_team_id`**: choosing a team whose team leader is set proposes that leader as the salesperson.

## 30.2 Event Lead Request

**Full name:** Event Lead Request. **Transport name** `event.lead.request`, **storage name** `event_lead_request`. **Kind:** persistent entity. **Contributed by the capability package** `event_crm`. Generated reference page: [`event.lead.request`](../../references/entities/event.lead.request.md). Access logging is disabled on it and its readable name is the event.

A background job ticket created when a user asks to regenerate the leads of an event that has too many attendees to process in one pass.

**Default ordering:** by identifier ascending.

| Field | Full name | Type | Required | Default | Meaning |
|---|---|---|---|---|---|
| `event_id` | Event | many_to_one to Event | yes | none | The event being processed. Deletion behavior: cascade. |
| `event_lead_rule_ids` | Lead Rules | many_to_many to Event Lead Rules | no | none | The rules to apply; empty means every matching rule. |
| `processed_registration_id` | Processed Registration | integer | no | 0 | The identifier of the last attendee already processed, used to resume the next batch. |

**Database constraint `_uniq_event`:** `unique(event_id)` with the message *"You can only have one generation request per event at a time."*

The batch size is 200 attendees and at most 100 requests are processed per run of the scheduled job.

---

## Reconciliation notes

1. **Field identifiers.** Version M of this folder rewrote the stored field names into a readable
   form — `date_time_zone`, `seats_maximum`, `event_web_address`, `interval_number`,
   `template_reference`, `limit_maximum_per_order`, `color_index`, `is_within_opening_hours`, the
   `call_to_action_*` family, `point_of_sale_order_line` and the plural forms of every relation.
   Version P used the transport names of the entities but wrote no field tables. Storage names are
   contractual, so this file reproduces them exactly as the database carries them (`date_tz`,
   `seats_max`, `event_url`, `interval_nbr`, `template_ref`, `limit_max_per_order`, `color`,
   `is_in_opening_hours`, `website_cta` and the rest) and adds the **Full name** column that the
   documentation rules require. No behaviour was changed by that substitution; each statement was
   checked against the field catalogue of the entity concerned.
2. **Entity names.** The two versions disagreed on the readable name of six entities. This file uses
   the name each entity carries in the entity dictionary of this repository: Event Automated Mailing
   (version P: Communication Schedule; version M: Event Communication), Registration Mail Scheduler
   (version P: Communication per Attendee), Slot Mail Scheduler (version P: Communication per Slot),
   Event Sponsor Level (version M: Event Sponsor Type), Track / Visitor Link (version M: Event Track
   Visitor) and Mail Scheduling on Event Category (version P: Template Communication). The alternative
   wordings are listed in [`glossary.md`](glossary.md).
3. **Scope.** Website Event Menu (`website.event.menu`) is owned by this folder. Version P omitted it
   from its entity list while describing the per-event menu behaviour in prose; version M owned it.
   The union is documented here, in section 27.
4. **Multi-slot capacity.** Both versions state that the seat maximum of a multi-slot event applies
   per slot and that the event-level available figure uses the maximum multiplied by the number of
   slots. The source confirms both statements, and the seat counter section of
   [`calculations.md`](calculations.md#1-seat-counters) carries the worked example.
5. **The badge-scan contract.** Section 4.7 now lists the seven outcomes in the order in which the
   tests are evaluated and states that the `invalid_ticket` answer carries no summary. Both versions
   had the event-mismatch test before the already-attended test; the corrected order is shared with
   [`state-machines.md`](state-machines.md) and [`workflows.md`](workflows.md).

