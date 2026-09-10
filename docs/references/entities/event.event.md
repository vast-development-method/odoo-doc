# Event (`event.event`)

**Transport name:** `event.event`  
**Storage name:** `event_event`  
**Kind:** persistent entity (one table)  
**Defined by package:** `event`  
**Extended by packages:** `event_booth`, `event_product`, `event_sale`, `event_crm`, `hr_skills_event`, `mass_mailing_event`, `mass_mailing_event_sms`, `website_event`, `website_event_track`, `mass_mailing_event_track`, `mass_mailing_event_track_sms`, `pos_event`, `website_event_booth`, `website_event_exhibitor`, `website_event_track_quiz`

Description: Event

## Identity and behavior

- Mixins (classical inheritance): `mail.thread`, `mail.activity.mixin`, `website.seo.metadata`, `website.published.multi.mixin`, `website.cover_properties.mixin`, `website.searchable.mixin`, `website.page_visibility_options.mixin`, `pos.load.mixin`
- Default ordering: `date_begin, id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (93)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Event | single line text |  | required; translatable |
| `note` | Note | rich text |  | computed by rule `_compute_note` and stored |
| `description` | Description | rich text |  | default computed dynamically (_default_description); translatable |
| `active` | Active | boolean |  | default `True` |
| `user_id` | Responsible | many to one | `res.users` | default computed dynamically (lambda self: self.env.user); changes are tracked in the message thread |
| `use_barcode` | Use Barcode | boolean |  | computed by rule `_compute_use_barcode` (not stored) |
| `company_id` | Company | many to one | `res.company` | default computed dynamically (lambda self: self.env.company) |
| `organizer_id` | Organizer | many to one | `res.partner` | default computed dynamically (lambda self: self.env.company.partner_id); changes are tracked in the message thread; must belong to the same company |
| `event_type_id` | Template | many to one | `event.type` | on delete of the target: set null; Help: Choose a template to auto-fill tickets, communications, descriptions and other fields. |
| `event_mail_ids` | Mail Schedule | one to many | `event.mail` | computed by rule `_compute_event_mail_ids` and stored; inverse field `event_id` |
| `tag_ids` | Tags | many to many | `event.tag` | computed by rule `_compute_tag_ids` and stored |
| `registration_properties_definition` | Registration Properties | properties definition |  |  |
| `kanban_state` | Kanban State | selection |  | computed by rule `_compute_kanban_state` and stored; default `normal`; changes are tracked in the message thread; not copied on duplication |
| `stage_id` | Stage | many to one | `event.stage` | default computed dynamically (_get_default_stage_id); changes are tracked in the message thread; not copied on duplication; on delete of the target: restrict |
| `seats_max` | Maximum Attendees | integer |  | computed by rule `_compute_seats_max` and stored; Help: For each event you can define a maximum registration of seats(number of attendees), above this number the registrations are not accepted. If the event has multiple slots, this maximum number is applied per slot. |
| `seats_limited` | Limit Attendees | boolean |  | required; computed by rule `_compute_seats_limited` and stored; precomputed before insertion |
| `seats_reserved` | Number of Registrations | integer |  | read only; computed by rule `_compute_seats` (not stored) |
| `seats_available` | Available Seats | integer |  | read only; computed by rule `_compute_seats` (not stored) |
| `seats_used` | Number of Attendees | integer |  | read only; computed by rule `_compute_seats` (not stored) |
| `seats_taken` | Number of Taken Seats | integer |  | read only; computed by rule `_compute_seats` (not stored) |
| `registration_ids` | Attendees | one to many | `event.registration` | inverse field `event_id` |
| `is_multi_slots` | Is Multi Slots | boolean |  | Help: Allow multiple time slots. The communications, the maximum number of attendees and the maximum number of tickets registrations are defined for each time slot instead of the whole event. |
| `event_slot_ids` | Slots | one to many | `event.slot` | inverse field `event_id` |
| `event_slot_count` | Slots Count | integer |  | computed by rule `_compute_event_slot_count` (not stored) |
| `event_ticket_ids` | Event Ticket | one to many | `event.event.ticket` | computed by rule `_compute_event_ticket_ids` and stored; inverse field `event_id`; precomputed before insertion |
| `event_registrations_started` | Registrations started | boolean |  | computed by rule `_compute_event_registrations_started` (not stored); Help: registrations have started if the current datetime is after the earliest starting date of tickets. |
| `event_registrations_open` | Registration open | boolean |  | computed by rule `_compute_event_registrations_open` (not stored); Help: Registrations are open if: - the event is not ended or not cancelled - there are seats available on event - the tickets are sellable (if ticketing is used) |
| `event_registrations_sold_out` | Sold Out | boolean |  | computed by rule `_compute_event_registrations_sold_out` (not stored); Help: The event is sold out if no more seats are available on event. If ticketing is used and all tickets are sold out, the event will be sold out. |
| `start_sale_datetime` | Start sale date | date and time |  | computed by rule `_compute_start_sale_date` (not stored); Help: If ticketing is used, contains the earliest starting sale date of tickets. |
| `date_tz` | Display Timezone | selection |  | required; computed by rule `_compute_date_tz` and stored; precomputed before insertion; Help: Indicates the timezone in which the event dates/times will be displayed on the website. |
| `date_begin` | Start Date | date and time |  | required; changes are tracked in the message thread; Help: When the event is scheduled to take place (expressed in your local timezone on the form view). |
| `date_end` | End Date | date and time |  | required; changes are tracked in the message thread |
| `is_ongoing` | Is Ongoing | boolean |  | computed by rule `_compute_time_data` (not stored); searchable through a search rule; Help: Whether event has begun; extended by packages `website_event` |
| `is_one_day` | Is One Day | boolean |  | computed by rule `_compute_field_is_one_day` (not stored) |
| `is_finished` | Is Finished | boolean |  | computed by rule `_compute_is_finished` (not stored); searchable through a search rule |
| `address_id` | Venue | many to one | `res.partner` | default computed dynamically (lambda self: self.env.company.partner_id.id); changes are tracked in the message thread; must belong to the same company |
| `address_search` | Address | many to one | `res.partner` | computed by rule `_compute_address_search` (not stored); searchable through a search rule |
| `address_inline` | Venue (formatted for one line uses) | single line text |  | computed by rule `_compute_address_inline` (not stored) |
| `country_id` | Country | many to one | `res.country` | related through path `address_id.country_id` and stored |
| `event_url` | Online Event uniform resource locator | single line text |  | computed by rule `_compute_event_url` and stored; Help: Link where the online event will take place. |
| `event_share_url` | Event Share uniform resource locator | single line text |  | computed by rule `_compute_event_share_url` (not stored) |
| `lang` | Language | selection |  | Help: All the communication emails sent to attendees will be translated in this language. |
| `badge_format` | Badge Dimension | selection |  | required; default `A6` |
| `badge_image` | Badge Background | image |  |  |
| `ticket_instructions` | Ticket Instructions | rich text |  | computed by rule `_compute_ticket_instructions` and stored; translatable; Help: This information will be printed on your tickets. |
| `question_ids` | Questions | many to many | `event.question` | computed by rule `_compute_question_ids` and stored; association table `event_event_event_question_rel`; precomputed before insertion |
| `general_question_ids` | General Questions | many to many | `event.question` | restricted by domain `[["once_per_order", "=", true]]`; association table `event_event_event_question_rel` |
| `specific_question_ids` | Specific Questions | many to many | `event.question` | restricted by domain `[["once_per_order", "=", false]]`; association table `event_event_event_question_rel` |
| `event_booth_ids` | Booths | one to many | `event.booth` | computed by rule `_compute_event_booth_ids` and stored; inverse field `event_id`; precomputed before insertion |
| `event_booth_count` | Total Booths | integer |  | computed by rule `_compute_event_booth_count` (not stored) |
| `event_booth_count_available` | Available Booths | integer |  | computed by rule `_compute_event_booth_count` (not stored) |
| `event_booth_category_ids` | Event Booth Category | many to many | `event.booth.category` | computed by rule `_compute_event_booth_category_ids` (not stored) |
| `event_booth_category_available_ids` | Event Booth Category Available | many to many | `event.booth.category` | computed by rule `_compute_event_booth_category_available_ids` (not stored); Help: Booth Category for which booths are still available. Used in frontend |
| `currency_id` | Currency | many to one | `res.currency` | read only; related through path `company_id.currency_id` |
| `sale_order_lines_ids` | All sale order lines pointing to this event | one to many | `sale.order.line` | visible only to groups `sales_team.group_sale_salesman`; inverse field `event_id` |
| `sale_price_total` | Sales (Tax Included) | monetary |  | computed by rule `_compute_sale_price_total` (not stored); visible only to groups `sales_team.group_sale_salesman` |
| `lead_ids` | Leads | one to many | `crm.lead` | visible only to groups `sales_team.group_sale_salesman`; inverse field `event_id`; Help: Leads generated from this event |
| `lead_count` | # Leads | integer |  | computed by rule `_compute_lead_count` (not stored); visible only to groups `sales_team.group_sale_salesman` |
| `subtitle` | Event Subtitle | single line text |  | translatable |
| `is_participating` | Is Participating | boolean |  | computed by rule `_compute_is_participating` (not stored); searchable through a search rule |
| `is_visible_on_website` | Visible On Website | boolean |  | computed by rule `_compute_is_visible_on_website` (not stored); searchable through a search rule |
| `event_register_url` | Event Registration Link | single line text |  | computed by rule `_compute_event_register_url` (not stored) |
| `website_visibility` | Website Visibility | selection |  | required; default `public`; changes are tracked in the message thread; Help: Defines the Visibility of the Event on the Website and searches.              Note that the EventEvent is however always available via its link. |
| `website_published` | Website Published | boolean |  | changes are tracked in the message thread |
| `website_menu` | Website Menu | boolean |  | computed by rule `_compute_website_menu` and stored; precomputed before insertion; Help: Allows to display and manage event-specific menus on website. |
| `menu_id` | Event Menu | many to one | `website.menu` | not copied on duplication |
| `introduction_menu` | Introduction Menu | boolean |  | computed by rule `_compute_website_menu_data` and stored |
| `introduction_menu_ids` | Introduction Menus | one to many | `website.event.menu` | restricted by domain `[["menu_type", "=", "introduction"]]`; inverse field `event_id` |
| `address_name` | Address Name | single line text |  | related through path `address_id.name` |
| `register_menu` | Register Menu | boolean |  | computed by rule `_compute_website_menu_data` and stored |
| `register_menu_ids` | Register Menus | one to many | `website.event.menu` | restricted by domain `[["menu_type", "=", "register"]]`; inverse field `event_id` |
| `community_menu` | Community Menu | boolean |  | computed by rule `_compute_community_menu` and stored; Help: Display community tab on website |
| `community_menu_ids` | Event Community Menus | one to many | `website.event.menu` | restricted by domain `[["menu_type", "=", "community"]]`; inverse field `event_id` |
| `other_menu_ids` | Other Menus | one to many | `website.event.menu` | restricted by domain `[["menu_type", "=", "other"]]`; inverse field `event_id` |
| `is_done` | Is Done | boolean |  | computed by rule `_compute_time_data` (not stored) |
| `start_today` | Start Today | boolean |  | computed by rule `_compute_time_data` (not stored); Help: Whether event is going to start today if still not ongoing |
| `start_remaining` | Remaining before start | integer |  | computed by rule `_compute_time_data` (not stored); Help: Remaining time before event starts (minutes) |
| `track_ids` | Tracks | one to many | `event.track` | inverse field `event_id` |
| `track_count` | Track Count | integer |  | computed by rule `_compute_track_count` (not stored) |
| `website_track` | Tracks on Website | boolean |  | computed by rule `_compute_website_track` and stored |
| `website_track_proposal` | Proposals on Website | boolean |  | computed by rule `_compute_website_track_proposal` and stored |
| `track_menu_ids` | Event Tracks Menus | one to many | `website.event.menu` | restricted by domain `[["menu_type", "=", "track"]]`; inverse field `event_id` |
| `track_proposal_menu_ids` | Event Proposals Menus | one to many | `website.event.menu` | restricted by domain `[["menu_type", "=", "track_proposal"]]`; inverse field `event_id` |
| `allowed_track_tag_ids` | Available Track Tags | many to many | `event.track.tag` | association table `event_allowed_track_tags_rel` |
| `tracks_tag_ids` | Track Tags | many to many | `event.track.tag` | computed by rule `_compute_tracks_tag_ids` and stored; association table `event_track_tags_rel` |
| `image_1024` | PoS Image | image |  |  |
| `exhibition_map` | Exhibition Map | image |  |  |
| `booth_menu` | Booth Register | boolean |  | computed by rule `_compute_booth_menu` and stored |
| `booth_menu_ids` | Event Booths Menus | one to many | `website.event.menu` | restricted by domain `[["menu_type", "=", "booth"]]`; inverse field `event_id` |
| `sponsor_ids` | Sponsors | one to many | `event.sponsor` | inverse field `event_id` |
| `sponsor_count` | Sponsor Count | integer |  | computed by rule `_compute_sponsor_count` (not stored) |
| `exhibitor_menu` | Showcase Exhibitors | boolean |  | computed by rule `_compute_exhibitor_menu` and stored |
| `exhibitor_menu_ids` | Exhibitors Menus | one to many | `website.event.menu` | restricted by domain `[["menu_type", "=", "exhibitor"]]`; inverse field `event_id` |

## Selection values

### `kanban_state` (Kanban State)

| Value | Label |
|---|---|
| `normal` | In Progress |
| `done` | Ready for Next Stage |
| `blocked` | Blocked |
| `cancel` | Cancelled |

### `badge_format` (Badge Dimension)

| Value | Label |
|---|---|
| `A4_french_fold` | A4 foldable |
| `A6` | A6 |
| `four_per_sheet` | 4 per sheet |

### `website_visibility` (Website Visibility)

| Value | Label |
|---|---|
| `public` | Public |
| `link` | Via a Link |
| `logged_users` | Logged Users |

## State fields

State machine fields of this entity: `kanban_state`. Transitions are specified in the domain documents.

## Operations (116)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `default_get` | lifecycle override | self, fields | `event` | model |  |
| `get_kiosk_url` | operation | self | `event` |  |  |
| `_get_default_stage_id` | preparation rule | self | `event` |  |  |
| `_default_description` | preparation rule | self | `event` |  |  |
| `_default_event_mail_ids` | preparation rule | self | `event` |  |  |
| `_lang_get` | internal rule | self | `event` | model |  |
| `_default_question_ids` | preparation rule | self | `event` |  |  |
| `_compute_use_barcode` | computation | self | `event` |  |  |
| `_compute_event_share_url` | computation | self | `event`, `website_event` | depends: `website_url` | Get the URL to use to redirect to the event, overriden in website for fallback. |
| `_compute_question_ids` | computation | self | `event` | depends: `event_type_id` | Update event questions from its event type. Depends are set only on event_type_id itself to emulate an onchange. Changing event type content itself should not trigger this method.  When synchronizing questions:    * lines with no registered answers for the event are removed;   * type lines are added; |
| `_compute_seats` | computation | self | `event` | depends: `event_slot_count`, `is_multi_slots`, `seats_max`, `registration_ids.state`, `registration_ids.active` | Determine available, reserved, used and taken seats. |
| `_compute_event_registrations_started` | computation | self | `event` | depends: `date_tz`, `start_sale_datetime` |  |
| `_compute_event_registrations_open` | computation | self | `event` | depends: `date_tz`, `event_registrations_started`, `date_end`, `seats_available`, `seats_limited`, `seats_max`, `event_ticket_ids.sale_available` | Compute whether people may take registrations for this event  * for cancelled events, registrations are not open; * event.date_end -> if event is done, registrations are not open anymore; * event.start_sale_datetime -> lowest start date of tickets (if any; start_sale_datetime   is False if no ticket are defined, see _compute_start_sale_date); * any ticket is available for sale (seats available) if any; * seats are unlimited or seats are available; |
| `_compute_start_sale_date` | computation | self | `event` | depends: `event_ticket_ids.start_sale_datetime` | Compute the start sale date of an event. Currently lowest starting sale date of tickets if they are used, of False. |
| `_compute_event_registrations_sold_out` | computation | self | `event` | depends: `event_slot_ids`, `event_ticket_ids.sale_available`, `seats_available`, `seats_limited` | Note that max seats limits for events and sum of limits for all its tickets may not be equal to enable flexibility. E.g. max 20 seats for ticket A, 20 seats for ticket B     * With max 20 seats for the event     * Without limit set on the event (=40, but the customer didn't explicitly write 40) When the event is multi slots, instead of checking if every tickets is sold out, checking if every slot-ticket combination is sold out. |
| `_compute_is_ongoing` | computation | self | `event` | depends: `date_begin`, `date_end` |  |
| `_search_is_ongoing` | search rule | self, operator, value | `event` |  |  |
| `_compute_field_is_one_day` | computation | self | `event` | depends: `date_begin`, `date_end`, `date_tz` |  |
| `_compute_is_finished` | computation | self | `event` | depends: `date_end` |  |
| `_search_is_finished` | search rule | self, operator, value | `event` |  |  |
| `_compute_date_tz` | computation | self | `event` | depends: `event_type_id` |  |
| `_compute_event_slot_count` | computation | self | `event` | depends: `event_slot_ids` |  |
| `_compute_address_search` | computation | self | `event` | depends: `address_id` |  |
| `_search_address_search` | search rule | self, operator, value | `event` |  |  |
| `_compute_seats_max` | computation | self | `event` | depends: `event_type_id` | Update event configuration from its event type. Depends are set only on event_type_id itself, not its sub fields. Purpose is to emulate an onchange: if event type is changed, update event configuration. Changing event type content itself should not trigger this method. |
| `_compute_seats_limited` | computation | self | `event` | depends: `event_type_id` | Update event configuration from its event type. Depends are set only on event_type_id itself, not its sub fields. Purpose is to emulate an onchange: if event type is changed, update event configuration. Changing event type content itself should not trigger this method. |
| `_compute_event_mail_ids` | computation | self | `event` | depends: `event_type_id` | Update event configuration from its event type. Depends are set only on event_type_id itself, not its sub fields. Purpose is to emulate an onchange: if event type is changed, update event configuration. Changing event type content itself should not trigger this method.  When synchronizing mails:    * lines that are not sent and have no registrations linked are remove;   * type lines are added; |
| `_compute_tag_ids` | computation | self | `event` | depends: `event_type_id` | Update event configuration from its event type. Depends are set only on event_type_id itself, not its sub fields. Purpose is to emulate an onchange: if event type is changed, update event configuration. Changing event type content itself should not trigger this method. |
| `_compute_event_ticket_ids` | computation | self | `event` | depends: `event_type_id` | Update event configuration from its event type. Depends are set only on event_type_id itself, not its sub fields. Purpose is to emulate an onchange: if event type is changed, update event configuration. Changing event type content itself should not trigger this method.  When synchronizing tickets:    * lines that have no registrations linked are remove;   * type lines are added;  Note that updating event_ticket_ids triggers _compute_start_sale_date (start_sale_datetime computation) so ensure result to avoid cache miss. |
| `_compute_note` | computation | self | `event` | depends: `event_type_id` |  |
| `_compute_kanban_state` | computation | self | `event` | depends: `stage_id` |  |
| `_compute_ticket_instructions` | computation | self | `event` | depends: `event_type_id` |  |
| `_compute_address_inline` | computation | self | `event` | depends: `address_id` | Use venue address if available, otherwise its name, finally ''. |
| `_compute_event_url` | computation | self | `event` | depends: `address_id` | Reset url field as it should only be used for events with no physical location. |
| `_check_slots_dates` | validation | self | `event` | constrains: `date_begin`, `date_end`, `event_slot_ids`, `is_multi_slots` |  |
| `_check_closing_date` | validation | self | `event` | constrains: `date_begin`, `date_end` |  |
| `_check_event_url` | validation | self | `event` | constrains: `event_url` |  |
| `_onchange_event_url` | on change | self | `event` | onchange: `event_url` | Correct the url by adding scheme if it is missing. |
| `_onchange_seats_max` | on change | self | `event` | onchange: `seats_max` |  |
| `_search` | search rule | self, domain, *a, **kw | `event` |  |  |
| `_compute_display_name` | computation | self | `event` | depends: `event_registrations_sold_out`, `seats_limited`, `seats_max`, `seats_available`; depends_context: `name_with_seats_availability` | Adds ticket seats availability if requested by context. |
| `copy_data` | lifecycle override | self, default | `event` |  |  |
| `_mail_get_operation_for_mail_message_operation` | messaging hook | self, message_operation | `event` |  |  |
| `_set_tz_context` | internal rule | self | `event` |  |  |
| `_get_seats_availability` | preparation rule | self, slot_tickets | `event` |  | Get availabilities for given combinations of slot / ticket. Returns a list following input order. None denotes no limit. |
| `_verify_seats_availability` | internal rule | self, slot_tickets | `event` |  | Check event seats availability, for combinations of slot / ticket.  :param slot_tickets: a list of tuples(slot, ticket, count). Slot and   ticket are optional, depending on event configuration. If count is 0   it is a simple check current values do not overflow limit. If count   is given, it serves as a check there are enough remaining seats. :raises ValidationError: if the event / slot / ticket do not have   enough available seats |
| `action_open_slot_calendar` | user action | self | `event` |  |  |
| `action_set_done` | user action | self | `event` |  | Action which will move the events into the first next (by sequence) stage defined as "Ended" (if they are not already in an ended stage) |
| `_get_date_range_str` | preparation rule | self, start_datetime, lang_code | `event` |  |  |
| `_get_external_description` | preparation rule | self | `event` |  | Description of the event shortened to maximum 1900 characters to leave some space for addition by sub-modules. Meant to be used for external content (ics/icalc/Gcal).  Reference Docs for URL limit -: https://stackoverflow.com/questions/417142/what-is-the-maximum-length-of-a-url-in-different-browsers |
| `_get_external_description_url_encoded` | preparation rule | self | `event` |  | Get a url-encoded version of the description for mail templates. |
| `_get_ics_file` | preparation rule | self, slot | `event` |  | Returns iCalendar file for the event invitation. :param slot: If a slot is given, schedule with the given slot datetimes :returns a dict of .ics file content for each event |
| `_get_tickets_access_hash` | preparation rule | self, registration_ids | `event` |  | Returns the ground truth hash for accessing the tickets in route /event/<int:event_id>/my_tickets. The dl links are always made event-dependant, hence the method linked to the record in self. |
| `_gc_mark_events_done` | background operation | self | `event` | autovacuum | move every ended events in the next 'ended stage' |
| `_compute_event_booth_ids` | computation | self | `event_booth` | depends: `event_type_id` | Update event configuration from its event type. Depends are set only on event_type_id itself, not its sub fields. Purpose is to emulate an onchange: if event type is changed, update event configuration. Changing event type content itself should not trigger this method.  When synchronizing booths:    * lines that are available are removed;   * template lines are added; |
| `_get_booth_stat_count` | preparation rule | self | `event_booth` |  |  |
| `_compute_event_booth_count` | computation | self | `event_booth` | depends: `event_booth_ids`, `event_booth_ids.state` |  |
| `_compute_event_booth_category_ids` | computation | self | `event_booth` | depends: `event_booth_ids.booth_category_id` |  |
| `_compute_event_booth_category_available_ids` | computation | self | `event_booth` | depends: `event_booth_ids.is_available` |  |
| `_compute_sale_price_total` | computation | self | `event_sale` | depends: `company_id.currency_id`, `sale_order_lines_ids.price_total`, `sale_order_lines_ids.currency_id`, `sale_order_lines_ids.company_id`, `sale_order_lines_ids.order_id.date_order` | Takes only confirmed the sale.order.lines related to this event and converts amounts from the currency of the sale order to the currency of the event company.  To avoid extra overhead, we use conversion rates as of 'today'. Meaning we have a number that can change over time, but using the conversion rates at the time of the related sale.order would mean thousands of extra requests as we would have to do one conversion per sale.order (and a sale.order is created every time we sell a single event ticket). |
| `action_view_linked_orders` | user action | self | `event_sale` |  | Redirects to only the confirmed orders linked to the current events |
| `_compute_lead_count` | computation | self | `event_crm` | depends: `lead_ids` |  |
| `action_generate_leads` | user action | self, event_lead_rules | `event_crm` |  | Re-generate leads based on event.lead.rules. The method is ran synchronously if there is a low amount of registrations, otherwise it goes through a CRON job that runs in batches. |
| `create` | lifecycle override | self, vals_list | `hr_skills_event`, `website_event` | model_create_multi |  |
| `action_mass_mailing_attendees` | user action | self | `mass_mailing_event_sms`, `mass_mailing_event` |  |  |
| `action_invite_contacts` | user action | self | `mass_mailing_event_sms`, `mass_mailing_event` |  |  |
| `_default_cover_properties` | preparation rule | self | `website_event` |  |  |
| `_compute_is_participating` | computation | self | `website_event` | depends: `registration_ids`; depends_context: `uid` |  |
| `_search_is_participating` | search rule | self, operator, value | `website_event` | model |  |
| `_fetch_is_participating_events` | internal rule | self | `website_event` | model | Heuristic  * public, no visitor: not participating as we have no information; * check only confirmed and attended registrations, a draft registration   does not make the attendee participating; * public and visitor: check visitor is linked to a registration. As   visitors are merged on the top parent, current visitor check is   sufficient even for successive visits; * logged, no visitor: check partner is linked to a registration. Do   not check the email as it is not really secure; * logged as visitor: check partner or visitor are linked to a   registration; |
| `_compute_is_visible_on_website` | computation | self | `website_event` | depends_context: `uid`; depends: `website_visibility`, `is_participating` |  |
| `_search_is_visible_on_website` | search rule | self, operator, value | `website_event` | model |  |
| `_compute_event_register_url` | computation | self | `website_event` | depends: `website_url` |  |
| `_compute_website_menu` | computation | self | `website_event` | depends: `event_type_id` | Also ensure a value for website_menu as it is a trigger notably for track related menus. |
| `_compute_community_menu` | computation | self | `website_event_track_quiz`, `website_event` | depends: `event_type_id`, `website_menu`, `community_menu` | Set False in base module. Sub modules will add their own logic (meet or track_quiz). |
| `_compute_website_menu_data` | computation | self | `website_event` | depends: `website_menu` | Synchronize with website_menu at change and let people update them at will afterwards. |
| `_compute_time_data` | computation | self | `website_event` | depends: `date_begin`, `date_end` | Compute start and remaining time. Do everything in UTC as we compute only time deltas here. |
| `_compute_website_url` | computation | self | `website_event` | depends: `name` |  |
| `_check_website_id` | validation | self | `website_event` | constrains: `website_id` |  |
| `copy` | lifecycle override | self, default | `website_event` |  |  |
| `copy_event_menus` | operation | self, old_events | `website_event_booth`, `website_event_exhibitor`, `website_event_track`, `website_event` |  |  |
| `write` | lifecycle override | self, vals | `website_event` |  |  |
| `toggle_website_menu` | operation | self, val | `website_event` |  |  |
| `_get_menu_update_fields` | preparation rule | self | `website_event_booth`, `website_event_exhibitor`, `website_event_track`, `website_event` |  | " Return a list of fields triggering a split of menu to activate / menu to de-activate. Due to saas-13.3 improvement of menu management this is done using side-methods to ease inheritance.  :returns: list of fields, each of which triggering a menu update   like website_menu, website_track, ... :rtype: list |
| `_get_menu_type_field_matching` | preparation rule | self | `website_event_booth`, `website_event_exhibitor`, `website_event_track`, `website_event` |  |  |
| `_split_menus_state_by_field` | internal rule | self | `website_event` |  | For each field linked to a menu, get the set of events having this menu activated and de-activated. Purpose is to find those whose value changed and update the underlying menus.  :returns: key = name of field triggering a website menu update, get {   'activated': subset of self having its menu currently set to True   'deactivated': subset of self having its menu currently set to False } |
| `_get_menus_update_by_field` | preparation rule | self, menus_state_by_field, force_update | `website_event` |  | For each field linked to a menu, get the set of events requiring this menu to be activated or de-activated based on previous recorded value.  :param menus_state_by_field: see ``_split_menus_state_by_field``; :param force_update: list of field to which we force update of menus. This   is used notably when a direct write to a stored editable field messes with   its pre-computed value, notably in a transient mode (aka demo for example);  :returns: key = name of field triggering a website menu update, get {   'activated': subset of self having its menu toggled to True   'deactivated': subset of se |
| `_get_website_menu_entries` | preparation rule | self | `website_event_booth`, `website_event_exhibitor`, `website_event_track`, `website_event` |  | Method returning menu entries to display on the website view of the event, possibly depending on some options in inheriting modules.  Each menu entry is a tuple containing :   * name: menu item name   * url: if set, url to a route (do not use xml_id in that case);   * xml_id: template linked to the page (do not use url in that case);   * sequence: specific sequence of menu entry to be set on the menu;   * menu_type: type of menu entry (used in inheriting modules to ease     menu management; not used in this module in 13.3 due to technical     limitations);   * parent_menu_type: menu_type of al |
| `_update_website_menus` | internal rule | self, menus_update_by_field | `website_event_booth`, `website_event_exhibitor`, `website_event_track`, `website_event` |  | Synchronize event configuration and its menu entries for frontend.  :param menus_update_by_field: see ``_get_menus_update_by_field`` |
| `_update_website_menu_entry` | internal rule | self, fname_bool, fname_o2m, fmenu_type | `website_event` |  | Generic method to create menu entries based on a flag on event. This method is a bit obscure, but is due to preparation of adding new menus entries and pages for event in a stable version, leading to some constraints while developing.  :param fname_bool: field name (e.g. website_track) :param fname_o2m: o2m linking towards website.event.menu matching the   boolean fields (normally an entry of website.event.menu with type matching   the boolean field name) :param fmenu_type: |
| `_create_menu` | internal rule | self, sequence, name, url, xml_id, menu_type, parent_menu_type | `website_event` |  | Create a new menu for the current event.  If url: create a website menu. Menu leads directly to the URL that should be a valid route.  If xml_id: create a new page using the qweb template given by its xml_id. Take its url back thanks to new_page of website, then link it to a menu. Template is duplicated and linked to a new url, meaning each menu will have its own copy of the template. This is currently limited to one menu: introduction(Home).  :param menu_type: type of menu. Mainly used for inheritance purpose   allowing more fine-grain tuning of menus. :param parent_menu_type: The type of the |
| `google_map_link` | operation | self, zoom | `website_event` |  | Temporary method for stable |
| `_google_map_link` | internal rule | self, zoom | `website_event` |  |  |
| `_track_subtype` | messaging hook | self, init_values | `website_event` |  |  |
| `_get_event_resource_urls` | preparation rule | self, slot | `website_event` |  | Prepare the Google and iCal urls for the event. :param slot: If a slot is given, prepare the urls for the given slot. Returns:     The google and iCal url in a dictionnary |
| `_default_website_meta` | preparation rule | self | `website_event` |  |  |
| `get_backend_menu_id` | operation | self | `website_event` |  |  |
| `_search_build_dates` | search rule | self | `website_event` | model |  |
| `_search_get_detail` | search rule | self, website, order, options | `website_event` | model |  |
| `_search_render_results` | search rule | self, fetch_fields, mapping, icon, limit | `website_event` |  |  |
| `_compute_track_count` | computation | self | `website_event_track` |  |  |
| `_compute_website_track` | computation | self | `website_event_track` | depends: `event_type_id`, `website_menu` | Propagate event_type configuration (only at change); otherwise propagate website_menu updated value. Also force True is track_proposal changes. |
| `_compute_website_track_proposal` | computation | self | `website_event_track` | depends: `event_type_id`, `website_track` | Propagate event_type configuration (only at change); otherwise propagate website_track updated value (both together True or False at update). |
| `_compute_tracks_tag_ids` | computation | self | `website_event_track` | depends: `track_ids.tag_ids`, `track_ids.tag_ids.color` |  |
| `_has_published_track` | internal rule | self | `website_event_track` |  |  |
| `toggle_website_track` | operation | self, val | `website_event_track` |  |  |
| `toggle_website_track_proposal` | operation | self, val | `website_event_track` |  |  |
| `action_mass_mailing_track_speakers` | user action | self | `mass_mailing_event_track_sms`, `mass_mailing_event_track` |  |  |
| `_load_pos_data_domain` | internal rule | self, data, config | `pos_event` | model |  |
| `_load_pos_data_fields` | internal rule | self, config | `pos_event` | model |  |
| `get_slot_tickets_availability_pos` | operation | self, slot_ticket_ids | `pos_event` |  |  |
| `_compute_booth_menu` | computation | self | `website_event_booth` | depends: `event_type_id`, `website_menu` |  |
| `toggle_booth_menu` | operation | self, val | `website_event_booth` |  |  |
| `_compute_sponsor_count` | computation | self | `website_event_exhibitor` |  |  |
| `_compute_exhibitor_menu` | computation | self | `website_event_exhibitor` | depends: `event_type_id`, `website_menu`, `exhibitor_menu` |  |
| `toggle_exhibitor_menu` | operation | self, val | `website_event_exhibitor` |  |  |

## Validation and error messages (6)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_slots_dates` | ValidationError | These events cannot have slots scheduled outside of their time range: %(event_names)s | `event` |
| `_check_closing_date` | ValidationError | The closing date cannot be earlier than the beginning date. | `event` |
| `_check_event_url` | ValidationError | Please enter a valid event URL. | `event` |
| `_verify_seats_availability` | ValidationError | There are not enough seats available for %(event_name)s: %(sold_out_info)s | `event` |
| `action_generate_leads` | UserError | Only Event Managers are allowed to re-generate all leads. | `event_crm` |
| `_check_website_id` | ValidationError | The website must be from the same company as the event. | `website_event` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `event.group_event_registration_desk` | no | yes | no | no | `event` |
| `event.group_event_user` | yes | yes | yes | no | `event` |
| `event.group_event_manager` | yes | yes | yes | yes | `event` |
| `point_of_sale.group_pos_user` | no | yes | no | no | `pos_event` |
| `base.group_public` | no | yes | no | no | `website_event` |
| `base.group_portal` | no | yes | no | no | `website_event` |
| `base.group_user` | no | yes | no | no | `website_event` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Event: multi-company | global (all users) | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |
| Event: public/portal: published read | `[(4, ref('base.group_public')), (4, ref('base.group_portal'))]` | `[('website_published', '=', True)]` | True | False | False | False |

## Views (27)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `event.view_event_form` | form |  | `stage_id`, `seats_taken`, `active`, `company_id`, `kanban_state`, `name`, `active`, `date_begin`, `date_end`, `date_tz`, `is_multi_slots`, `event_slot_count`, `lang`, `event_type_id`, `tag_ids`, `organizer_id`, `user_id`, `company_id`, `address_id`, `event_url`, `seats_limited`, `seats_max`, `event_ticket_ids`, `event_mail_ids`, `sequence`, `template_ref`, `interval_nbr`, `interval_unit`, `interval_type`, `scheduled_date`, `mail_count_done`, `mail_state`, `question_ids`, `sequence`, `title`, `is_mandatory_answer`, `once_per_order`, `question_type`, `answer_ids`, `is_default`, `is_reusable`, `badge_format`, `badge_image`, `ticket_instructions`, `note` | `%(event_barcode_action_main_view)d`, `%(event.event_registration_action_stats_from_event)d`, `%(event.act_event_registration_from_event)d`, `action_open_slot_calendar`, `Stats` |  | `event` |
| `event.view_event_tree` | list |  | `name`, `address_id`, `organizer_id`, `user_id`, `company_id`, `date_begin`, `date_end`, `tag_ids`, `seats_taken`, `seats_used`, `seats_max`, `seats_reserved`, `stage_id`, `message_needaction`, `activity_exception_decoration` |  |  | `event` |
| `event.event_event_view_activity` | activity |  | `user_id`, `name`, `date_begin` |  |  | `event` |
| `event.event_event_view_form_quick_create` | form |  | `name`, `date_begin`, `date_end` |  |  | `event` |
| `event.view_event_kanban` | kanban |  | `stage_id`, `date_begin`, `date_end`, `name`, `address_id`, `seats_taken`, `activity_ids`, `user_id`, `kanban_state` |  |  | `event` |
| `event.view_event_calendar` | calendar |  | `user_id`, `seats_taken`, `seats_reserved`, `seats_used`, `event_type_id` |  |  | `event` |
| `event.view_event_search` | search |  | `name`, `event_type_id`, `user_id`, `address_search`, `stage_id`, `activity_user_id`, `activity_type_id`, `question_ids` |  | `My Events`, `Upcoming/Running`, `Start Date`, `Archived`, `My Activities`, `Late Activities`, `Today Activities`, `Future Activities`, `Online`, `Responsible`, `Template`, `Stage`, `Start Date`, `Venue` | `event` |
| `event_booth.event_event_view_form` | div | `event.view_event_form` | `event_booth_count_available`, `event_booth_count` | `%(event_booth_action_from_event)d` |  | `event_booth` |
| `event_crm.event_view_form` | xpath | `event.view_event_form` | `lead_count` | `%(crm_lead_action_from_event)d` |  | `event_crm` |
| `event_crm.event_view_tree` | xpath | `event.view_event_tree` | `lead_count` |  |  | `event_crm` |
| `event_sale.view_event_form_inherit_ticket` | xpath | `event.view_event_form` | `currency_id`, `sale_price_total` | `action_view_linked_orders` |  | `event_sale` |
| `mass_mailing_event.event_event_view_form_inherit_mass_mailing` | xpath | `event.view_event_form` | `event_registrations_open` | `Invite`, `Invite`, `Contact Attendees` |  | `mass_mailing_event` |
| `mass_mailing_event_track.event_event_view_form_inherit_mass_mailing_track` | xpath | `event.view_event_form` |  | `Contact Speakers` |  | `mass_mailing_event_track` |
| `pos_event.event_event_view_form_inherit_pos_event` | xpath | `event.view_event_form` | `image_1024` |  |  | `pos_event` |
| `website_event.event_event_view_form` | xpath | `event.view_event_form` | `website_id`, `website_published`, `website_id`, `website_visibility`, `event_register_url` | `Publish` |  | `website_event` |
| `website_event.event_event_view_list` | field | `event.view_event_tree` | `company_id`, `company_id`, `website_id` |  |  | `website_event` |
| `website_event.event_event_view_search` | xpath | `event.view_event_search` |  |  | `Published` | `website_event` |
| `website_event.event_pages_tree_view` | xpath | `event.view_event_tree` |  |  |  | `website_event` |
| `website_event.event_pages_kanban_view` | xpath | `event.view_event_kanban` |  |  |  | `website_event` |
| `website_event.event_event_view_form_website_create` | xpath | `event.view_event_form` |  |  |  | `website_event` |
| `website_event_booth.event_event_view_form` | field | `website_event.event_event_view_form` | `badge_format`, `exhibition_map` |  |  | `website_event_booth` |
| `website_event_exhibitor.event_event_view_form` | field | `website_event.event_event_view_form` | `website_url`, `sponsor_count` | `%(event_sponsor_action_from_event)d` |  | `website_event_exhibitor` |
| `website_event_exhibitor.event_event_view_list` | field | `event.view_event_tree` | `stage_id`, `sponsor_count` |  |  | `website_event_exhibitor` |
| `website_event_sale.event_form_mandatory_company` | xpath | `event.view_event_form` |  |  |  | `website_event_sale` |
| `website_event_track.event_event_view_form` | xpath | `website_event.event_event_view_form` | `track_count` | `%(action_event_track_from_event)d` |  | `website_event_track` |
| `website_event_track.event_event_view_list` | field | `event.view_event_tree` | `stage_id`, `track_count` |  |  | `website_event_track` |
| `website_event_track_quiz.event_event_view_form` | xpath | `website_event.event_event_view_form` |  |  |  | `website_event_track_quiz` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `event.action_event_view` | Events | kanban,calendar,list,form,pivot,graph,activity |  |  |  | `event` |
| `hr_skills_event.event_training_onsite_action` | Onsite Courses | kanban,calendar,list,form,pivot,graph,activity | `[('registration_ids', 'any', [('partner_id.employee', '=', True)])]` | `{'hr_skills_event_add_employee': True}` |  | `hr_skills_event` |
| `website_event.action_event_pages_list` | Event Pages | list,kanban |  | `{'create_action': 'website_event.event_event_action_add'}` |  | `website_event` |
| `website_event.event_event_action_add` | New Event | form |  | `{'default_address_id': False}` | new | `website_event` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `event.menu_event_event` |  |  | `event.action_event_view` |  |  |

## Server actions

| Action | Name | State | Binding | Custom logic |
|---|---|---|---|---|
| `event_crm.action_generate_leads` | Generate Leads | code |  | yes |

## Printable reports

| Report | Name | Type | Template | File name rule | Attachment rule |
|---|---|---|---|---|---|
| `event.action_report_event_event_full_page_ticket` | Full Page Ticket Example | qweb-pdf | `event.event_event_report_template_full_page_ticket` | `'Full Page Ticket - %s' % (object.name or 'Event').replace('/','')` |  |
| `event.action_report_event_event_badge` | Badge Example | qweb-pdf | `event.event_event_report_template_badge` | `'Badge - %s' % (object.name or 'Event').replace('/','')` |  |
| `event.action_report_event_event_attendee_list` | Attendee List | qweb-pdf | `event.event_event_attendee_list` | `'Attendee List - %s' % (object.name)` |  |

Machine-readable definition: `../../../schemas/data/entities/event.event.json`; views: `../../../schemas/interfaces/views/event.event.json`.
