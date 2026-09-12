# Event Registration (`event.registration`)

**Transport name:** `event.registration`  
**Storage name:** `event_registration`  
**Kind:** persistent entity (one table)  
**Defined by package:** `event`  
**Extended by packages:** `event_product`, `event_sale`, `event_crm`, `event_crm_sale`, `mass_mailing_event`, `website_event`, `pos_event`, `pos_event_sale`, `website_event_crm`

Description: Event Registration

## Identity and behavior

- Mixins (classical inheritance): `mail.thread`, `mail.activity.mixin`, `pos.load.mixin`
- Default ordering: `id desc`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (34)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `event_id` | Event | many to one | `event.event` | required; changes are tracked in the message thread; indexed |
| `is_multi_slots` | Is Event Multi Slots | boolean |  | related through path `event_id.is_multi_slots` |
| `event_slot_id` | Slot | many to one | `event.slot` | changes are tracked in the message thread; indexed (btree_not_null); on delete of the target: restrict; restricted by domain `[('event_id', '=', event_id)]` |
| `event_ticket_id` | Ticket Type | many to one | `event.event.ticket` | changes are tracked in the message thread; indexed (btree_not_null); on delete of the target: restrict |
| `active` | Active | boolean |  | default `True` |
| `barcode` | Barcode | single line text |  | read only; default computed dynamically (lambda self: self._get_random_barcode()); not copied on duplication |
| `utm_campaign_id` | Campaign | many to one | `utm.campaign` | computed by rule `_compute_utm_campaign_id` and stored; indexed; on delete of the target: set null; extended by packages `event_sale` |
| `utm_source_id` | Source | many to one | `utm.source` | computed by rule `_compute_utm_source_id` and stored; indexed; on delete of the target: set null; extended by packages `event_sale` |
| `utm_medium_id` | Medium | many to one | `utm.medium` | computed by rule `_compute_utm_medium_id` and stored; indexed; on delete of the target: set null; extended by packages `event_sale` |
| `partner_id` | Booked by | many to one | `res.partner` | changes are tracked in the message thread; indexed (btree_not_null) |
| `name` | Attendee Name | single line text |  | computed by rule `_compute_name` and stored; changes are tracked in the message thread; indexed (trigram) |
| `email` | Email | single line text |  | computed by rule `_compute_email` and stored; changes are tracked in the message thread |
| `phone` | Phone | single line text |  | computed by rule `_compute_phone` and stored; changes are tracked in the message thread |
| `company_name` | Company Name | single line text |  | computed by rule `_compute_company_name` and stored; changes are tracked in the message thread |
| `date_closed` | Attended Date | date and time |  | computed by rule `_compute_date_closed` and stored |
| `event_begin_date` | Event Start Date | date and time |  | computed by rule `_compute_event_begin_date` (not stored); searchable through a search rule |
| `event_end_date` | Event End Date | date and time |  | computed by rule `_compute_event_end_date` (not stored); searchable through a search rule |
| `event_date_range` | Date Range | single line text |  | computed by rule `_compute_date_range` (not stored) |
| `event_organizer_id` | Event Organizer | many to one |  | read only; related through path `event_id.organizer_id` |
| `event_user_id` | Event Responsible | many to one |  | read only; related through path `event_id.user_id` |
| `company_id` | Company | many to one | `res.company` | related through path `event_id.company_id` and stored |
| `state` | Status | selection |  | computed by rule `_compute_registration_status` and stored; default ; changes are tracked in the message thread; not copied on duplication; precomputed before insertion; Help: Unconfirmed: registrations in a pending state waiting for an action (specific case, notably with sale status) Registered: registrations considered taken by a client Attended: registrations for which the attendee attended the event Cancelled: registrations cancelled manually; extended by packages `event_sale` |
| `registration_answer_ids` | Attendee Answers | one to many | `event.registration.answer` | inverse field `registration_id` |
| `registration_answer_choice_ids` | Attendee Selection Answers | one to many | `event.registration.answer` | restricted by domain `[["question_type", "=", "simple_choice"]]`; inverse field `registration_id` |
| `mail_registration_ids` | Scheduler Emails | one to many | `event.mail.registration` | read only; inverse field `registration_id` |
| `registration_properties` | Properties | properties |  |  |
| `sale_status` | Sale Status | selection |  | read only; computed by rule `_compute_registration_status` and stored; precomputed before insertion |
| `sale_order_id` | Sales Order | many to one | `sale.order` | not copied on duplication; on delete of the target: cascade |
| `sale_order_line_id` | Sales Order Line | many to one | `sale.order.line` | indexed (btree_not_null); not copied on duplication; on delete of the target: cascade |
| `lead_ids` | Leads | many to many | `crm.lead` | read only; not copied on duplication; visible only to groups `sales_team.group_sale_salesman` |
| `lead_count` | # Leads | integer |  | computed by rule `_compute_lead_count` (not stored) |
| `visitor_id` | Visitor | many to one | `website.visitor` | indexed (btree_not_null); on delete of the target: set null |
| `pos_order_id` | PoS Order | many to one |  | related through path `pos_order_line_id.order_id` |
| `pos_order_line_id` | PoS Order Line | many to one | `pos.order.line` | indexed (btree_not_null); not copied on duplication; on delete of the target: cascade |

## Selection values

### `state` (Status)

| Value | Label |
|---|---|
| `draft` | Unconfirmed |
| `open` | Registered |
| `done` | Attended |
| `cancel` | Cancelled |

### `sale_status` (Sale Status)

| Value | Label |
|---|---|
| `to_pay` | Not Sold |
| `sold` | Sold |
| `free` | Free |

## State fields

State machine fields of this entity: `state`, `sale_status`. Transitions are specified in the domain documents.

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_barcode_event_uniq` | Constraint | `unique(barcode)` | Barcode should be unique | `event` |

## Operations (66)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_random_barcode` | preparation rule | self | `event` | model | Generate a string representation of a pseudo-random 8-byte number for barcode generation.  A decimal serialisation is longer than a hexadecimal one *but* it generates a more compact barcode (Code128C rather than Code128A).  Generate 8 bytes (64 bits) barcodes as 16 bytes barcodes are not compatible with all scanners. |
| `_check_seats_availability` | validation | self | `event` | constrains: `active`, `state`, `event_id`, `event_slot_id`, `event_ticket_id` |  |
| `default_get` | lifecycle override | self, fields | `event` | model |  |
| `_compute_name` | computation | self | `event` | depends: `partner_id` |  |
| `_compute_email` | computation | self | `event` | depends: `partner_id` |  |
| `_compute_phone` | computation | self | `event` | depends: `partner_id` |  |
| `_compute_company_name` | computation | self | `event` | depends: `partner_id` |  |
| `_compute_date_closed` | computation | self | `event` | depends: `state` |  |
| `_compute_date_range` | computation | self | `event` | depends: `event_id`, `event_slot_id`, `partner_id` |  |
| `_compute_event_begin_date` | computation | self | `event` | depends: `event_id`, `event_slot_id` |  |
| `_search_event_begin_date` | search rule | self, operator, value | `event` | model |  |
| `_compute_event_end_date` | computation | self | `event` | depends: `event_id`, `event_slot_id` |  |
| `_search_event_end_date` | search rule | self, operator, value | `event` | model |  |
| `_check_event_slot` | validation | self | `event` | constrains: `event_id`, `event_slot_id` |  |
| `_check_event_ticket` | validation | self | `event` | constrains: `event_id`, `event_ticket_id` |  |
| `_synchronize_partner_values` | internal rule | self, partner, fnames | `event` |  |  |
| `_onchange_event` | on change | self | `event` | onchange: `event_id` |  |
| `_onchange_phone_validation` | on change | self | `event` | onchange: `phone`, `event_id`, `partner_id` |  |
| `register_attendee` | operation | self, barcode, event_id | `event` | model |  |
| `create` | lifecycle override | self, vals_list | `event_crm`, `event_sale`, `event`, `pos_event` | model_create_multi | Trigger rules based on registration creation, and check state for rules based on confirmed / done attendees. |
| `write` | lifecycle override | self, vals | `event_crm`, `event_sale`, `event`, `pos_event` |  | Update the lead values depending on fields updated in registrations. There are 2 main use cases    * first is when we update the partner_id of multiple registrations. It     happens when a public user fill its information when they register to     an event;   * second is when we update specific values of one registration like     updating question answers or a contact information (email, phone);  Also trigger rules based on confirmed and done attendees (state written to open and done). |
| `_compute_display_name` | computation | self | `event` |  | Custom display_name in case a registration is nott linked to an attendee |
| `action_set_draft` | user action | self | `event` |  |  |
| `action_confirm` | user action | self | `event` |  |  |
| `action_set_done` | user action | self | `event` |  | Close Registration |
| `action_cancel` | user action | self | `event` |  |  |
| `action_send_badge_email` | user action | self | `event` |  | Open a window to compose an email, with the template - 'event_badge' message loaded by default |
| `_update_mail_schedulers` | internal rule | self | `event` |  | Update schedulers to set them as running again, and cron to be called as soon as possible. |
| `_mail_template_default_values` | messaging hook | self | `event` | model |  |
| `_message_compute_subject` | messaging hook | self | `event` |  |  |
| `_message_add_default_recipients` | messaging hook | self | `event` |  |  |
| `_message_post_after_hook` | messaging hook | self, message, msg_vals | `event` |  |  |
| `_get_registration_summary` | preparation rule | self | `event_sale`, `event` |  |  |
| `_has_order` | internal rule | self | `event_product`, `event_sale`, `pos_event` |  |  |
| `_compute_registration_status` | computation | self | `event_product`, `event_sale`, `pos_event_sale`, `pos_event` | depends: `sale_order_id.state`, `sale_order_id.currency_id`, `sale_order_id.amount_total`; depends: `pos_order_id.state`, `pos_order_id.currency_id`, `pos_order_id.amount_total`; depends: `pos_order_id.state` |  |
| `_compute_utm_campaign_id` | computation | self | `event_sale` | depends: `sale_order_id` |  |
| `_compute_utm_source_id` | computation | self | `event_sale` | depends: `sale_order_id` |  |
| `_compute_utm_medium_id` | computation | self | `event_sale` | depends: `sale_order_id` |  |
| `action_view_sale_order` | user action | self | `event_sale` |  |  |
| `_synchronize_so_line_values` | internal rule | self, so_line | `event_sale` |  |  |
| `_sale_order_registration_data_change_notify` | internal rule | self, new_record_field, new_record | `event_sale` |  |  |
| `_compute_field_value` | computation | self, field | `event_sale` |  |  |
| `_get_event_registration_ids_from_order` | preparation rule | self | `event_sale` |  |  |
| `_compute_lead_count` | computation | self | `event_crm` | depends: `lead_ids` |  |
| `_load_records_create` | internal rule | self, values | `event_crm` |  | In import mode: do not run rules those are intended to run when customers buy tickets, not when bootstrapping a database. |
| `_load_records_write` | internal rule | self, values | `event_crm` |  | In import mode: do not run rules those are intended to run when customers buy tickets, not when bootstrapping a database. |
| `_apply_lead_generation_rules` | internal rule | self, event_lead_rules | `event_crm` |  |  |
| `_update_leads` | internal rule | self, new_vals, lead_tracked_vals | `event_crm` |  | Update leads linked to some registrations. Update is based depending on updated fields, see `_get_lead_contact_fields()` and `_get_lead_ description_fields()`. Main heuristic is    * check attendee-based leads, for each registration recompute contact     information if necessary (changing partner triggers the whole contact     computation); update description if necessary;   * check order-based leads, for each existing group-based lead, only     partner change triggers a contact and description update. We consider     that group-based rule works mainly with the main contact and less     wi |
| `_get_lead_values` | preparation rule | self, rule | `event_crm`, `website_event_crm` |  | Get lead values from registrations. Self can contain multiple records in which case first found non void value is taken. Note that all registrations should belong to the same event.  :returns: values used for create / write on a lead :rtype: dict |
| `_get_lead_contact_values` | preparation rule | self | `event_crm` |  | Specific management of contact values. Rule creation basis has some effect on contact management    * in attendee mode: keep registration partner only if partner phone and     email match. Indeed lead are synchronized with their contact and it     would imply rewriting on partner, and therefore on other documents;   * in batch mode: if a customer is found use it as main contact. Registrations     details are included in lead description;  :returns: values used for create / write on a lead :rtype: dict |
| `_get_lead_description` | preparation rule | self, prefix, line_counter, line_suffix | `event_crm` |  | Build the description for the lead using a prefix for all generated lines. For example to enumerate participants or inform of an update in the information of a participant.  :returns: complete description for a lead taking into   account all registrations contained in self :rtype: str |
| `_get_lead_description_registration` | preparation rule | self, line_suffix | `event_crm`, `website_event_crm` |  | Build the description line specific to a given registration. |
| `_get_lead_tracked_values` | preparation rule | self | `event_crm` |  | Tracked values are based on two subset of fields to track in order to fill or update leads. Two main use cases are    * description fields: registration contact fields: email, phone, ...     on registration. Other fields are added by inheritance like     question answers;   * contact fields: registration contact fields + partner_id field as     contact of a lead is managed specifically. Indeed email and phone     synchronization of lead / partner_id implies paying attention to     not rewrite partner values from registration values.  Tracked values are therefore the union of those two field se |
| `_get_lead_grouping` | preparation rule | self, rules, rule_to_new_regs | `event_crm_sale`, `event_crm` |  | Perform grouping of registrations in order to enable order-based lead creation and update existing groups with new registrations.  Heuristic in event is the following. Registrations created in multi-mode are grouped by event and creation_date. Customer use case: website_event flow creates several registrations in a create-multi. Cron use case: when running a rule on existing registrations, grouping on event only is not sufficient, create_date is a safe bet for registration groups.  Update is not supported as there is no way to determine if a registration is part of an existing batch.  :param r |
| `_get_lead_contact_fields` | preparation rule | self | `event_crm` | model | Get registration fields linked to lead contact. Those are used notably to see if an update of lead is necessary or to fill contact values in `_get_lead_contact_values())` |
| `_get_lead_description_fields` | preparation rule | self | `event_crm`, `website_event_crm` | model | Get registration fields linked to lead description. Those are used notably to see if an update of lead is necessary or to fill description in `_get_lead_description())` |
| `_find_first_notnull` | internal rule | self, field_name | `event_crm` |  | Small tool to extract the first not nullvalue of a field: its value or the ids if this is a relational field. |
| `_convert_value` | internal rule | self, value, field_name | `event_crm` |  | Small tool because convert_to_write is touchy |
| `_mailing_get_default_domain` | messaging hook | self, mailing | `mass_mailing_event` |  |  |
| `_get_website_registration_allowed_fields` | preparation rule | self | `website_event` |  |  |
| `get_base_url` | operation | self | `website_event` |  |  |
| `_load_pos_data_domain` | internal rule | self, data, config | `pos_event` | model |  |
| `_load_pos_data_fields` | internal rule | self, config | `pos_event` | model |  |
| `_populate_creation_vals` | internal rule | self, vals_list | `pos_event` |  |  |
| `_update_available_seat` | internal rule | self | `pos_event` |  |  |
| `action_view_pos_order` | user action | self | `pos_event` |  |  |

## Validation and error messages (3)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_event_slot` | ValidationError | Invalid event / slot choice | `event` |
| `_check_event_slot` | ValidationError | Slot choice is mandatory on multi-slots events. | `event` |
| `_check_event_ticket` | ValidationError | Invalid event / ticket choice | `event` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| all internal users | no | no | no | no | `event` |
| `event.group_event_registration_desk` | yes | yes | yes | no | `event` |
| `event.group_event_manager` | yes | yes | yes | yes | `event` |
| `point_of_sale.group_pos_user` | yes | yes | yes | no | `pos_event` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Event/Registration: multi-company | global (all users) | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |

## Views (19)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `event.view_event_registration_tree` | list |  | `barcode`, `active`, `create_date`, `name`, `partner_id`, `email`, `phone`, `company_name`, `event_id`, `event_slot_id`, `event_ticket_id`, `activity_ids`, `state`, `company_id`, `message_needaction`, `activity_exception_decoration` | `Registered`, `Mark as Attending`, `Cancel` |  | `event` |
| `event.view_event_registration_form` | form |  | `active`, `state`, `name`, `email`, `phone`, `company_name`, `event_id`, `event_slot_id`, `event_ticket_id`, `barcode`, `partner_id`, `create_date`, `date_closed`, `utm_campaign_id`, `utm_medium_id`, `utm_source_id`, `registration_properties`, `registration_answer_ids`, `event_id`, `question_id`, `question_type`, `value_answer_id`, `value_text_box`, `event_id`, `question_type`, `question_id`, `value_answer_id`, `value_text_box` | `Send by Email`, `Registered`, `Attended`, `Cancel Registration` |  | `event` |
| `event.event_registration_view_kanban` | kanban |  | `name`, `state`, `active`, `barcode`, `name`, `state`, `event_id`, `company_name`, `registration_properties`, `event_slot_id`, `event_ticket_id` |  |  | `event` |
| `event.view_event_registration_calendar` | calendar |  | `event_id`, `name`, `registration_properties` |  |  | `event` |
| `event.view_event_registration_pivot` | pivot |  | `event_id` |  |  | `event` |
| `event.view_event_registration_graph` | graph |  | `event_id` |  |  | `event` |
| `event.view_registration_search` | search |  | `id`, `name`, `company_id`, `partner_id`, `event_ticket_id`, `event_id`, `event_user_id`, `event_organizer_id` |  | `Ongoing Events`, `Taken`, `Unconfirmed`, `Registered`, `Attended`, `Registration Date`, `Event Start Date`, `Attended Date`, `My Activities`, `Late Activities`, `Today Activities`, `Future Activities`, `Last 30 days`, `Archived`, `Partner`, `Event`, `Slot`, `Ticket Type`, `Status`, `Registration Date`, `Registration Date`, `Campaign`, `Medium`, `Source`, `Properties` | `event` |
| `event.event_registration_view_search_event_specific` | xpath | `view_registration_search` |  |  |  | `event` |
| `event_crm.event_registration_view_form` | xpath | `event.view_event_registration_form` | `lead_count` | `%(crm_lead_action_from_registration)d` |  | `event_crm` |
| `event_product.view_event_registration_ticket_tree` | field | `event.view_event_registration_tree` | `state`, `sale_status` |  |  | `event_product` |
| `event_product.event_registration_view_graph` | field | `event.view_event_registration_graph` | `event_id`, `sale_status` |  |  | `event_product` |
| `event_product.event_registration_ticket_view_form` | xpath | `event.view_event_registration_form` |  |  |  | `event_product` |
| `event_sale.view_event_registration_ticket_tree` | field | `event.view_event_registration_tree` | `event_id`, `sale_order_id` |  |  | `event_sale` |
| `event_sale.event_registration_ticket_view_form` | xpath | `event_product.event_registration_ticket_view_form` |  | `action_view_sale_order` |  | `event_sale` |
| `pos_event.event_registration_ticket_view_form` | xpath | `event_product.event_registration_ticket_view_form` |  | `action_view_pos_order` |  | `pos_event` |
| `website_event.event_registration_view_form` | xpath | `event.view_event_registration_form` | `visitor_id` |  |  | `website_event` |
| `website_event.event_registration_view_tree` | xpath | `event.view_event_registration_tree` | `visitor_id` |  |  | `website_event` |
| `website_event.event_registration_view_kanban` | xpath | `event.event_registration_view_kanban` | `registration_answer_choice_ids` |  |  | `website_event` |
| `website_event.event_registration_view_search` | field | `event.view_registration_search` | `partner_id`, `registration_answer_ids` |  |  | `website_event` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `event.act_event_registration_from_event` | Attendees | list,kanban,form,calendar,graph | `[('event_id', '=', active_id)]` | `{             'default_event_id': active_id,             'name_with_seats_availability': True,             'search_default_taken': True,         }` |  | `event` |
| `event.event_registration_action_kanban` | Attendees | kanban,list,form | `[('event_id', '=', active_id)]` | `{'default_event_id': active_id, 'is_registration_desk_view': True}` |  | `event` |
| `event.event_registration_action` | Attendees | kanban,list,form |  | `{'search_default_filter_is_ongoing': True, 'is_registration_desk_view': True}` |  | `event` |
| `event.event_registration_action_tree` | Event registrations | list,kanban,form,calendar,graph |  |  |  | `event` |
| `event.action_registration` | Attendees | graph,pivot,kanban,list,form |  | `{                 'search_default_filter_last_month_creation': 1,                 'search_default_taken': 1,                 'search_default_status': 2,                 'search_default_group_by_create_date_day': 3,                 'search_default_group_event': 1,                 'registration_view_hide_group_by_create_date_week': 1,             }` |  | `event` |
| `event.event_registration_action_stats_from_event` | Registration statistics | graph,pivot,kanban,list,form | `[('event_id', '=', active_id)]` | `{                 'default_event_id': active_id,                 'search_default_group_by_create_date_day': 1,                 'search_default_status': 2,                 'registration_view_hide_group_by_create_date_week': 1,             }` |  | `event` |
| `event_crm.event_registration_action_from_lead` | Event registrations | list,kanban,form,calendar,graph | `[('lead_ids', '=', active_id)]` | `{'create': False}` |  | `event_crm` |
| `website_event.event_registration_action_from_visitor` | Registrations | kanban,list,form | `[('visitor_id', 'in', [active_id])]` |  |  | `website_event` |

## Printable reports

| Report | Name | Type | Template | File name rule | Attachment rule |
|---|---|---|---|---|---|
| `event.action_report_event_registration_full_page_ticket` | Full Page Ticket | qweb-pdf | `event.event_registration_report_template_full_page_ticket` | `'Full Page Ticket - %s - %s' % ((object.event_id.name or 'Event').replace('/',''), (object.name or '').replace('/',''))` |  |
| `event.action_report_event_registration_badge` | Badge | qweb-pdf | `event.event_registration_report_template_badge` | `'Badge - %s - %s' % ((object.event_id.name or 'Event').replace('/',''), (object.name or '').replace('/',''))` |  |
| `event.action_report_event_registration_responsive_html_ticket` | Responsive Html Full Page Ticket | qweb-pdf | `event.event_registration_report_template_responsive_html_ticket` |  |  |
| `event.action_report_event_registration_attendee_list` | Attendee List | qweb-pdf | `event.event_registration_attendee_list` | `'Attendee List'` |  |

## Email templates

| Template | Name | Subject |
|---|---|---|
| `event.event_registration_mail_template_badge` | Event: Registration Badge | Your badge for {{ object.event_id.name }} |
| `event.event_subscription` | Event: Registration Confirmation | Your registration at {{ object.event_id.name }} |
| `event.event_reminder` | Event: Reminder | {{ object.event_id.name }}: {{ object.event_date_range }} |

Machine-readable definition: `../../../schemas/data/entities/event.registration.json`; views: `../../../schemas/interfaces/views/event.registration.json`.
