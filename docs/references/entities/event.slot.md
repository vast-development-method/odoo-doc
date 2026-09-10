# Event Slot (`event.slot`)

**Transport name:** `event.slot`  
**Storage name:** `event_slot`  
**Kind:** persistent entity (one table)  
**Defined by package:** `event`  
**Extended by packages:** `website_event`, `pos_event`

Description: Event Slot

## Identity and behavior

- Mixins (classical inheritance): `pos.load.mixin`
- Default ordering: `event_id, date, start_hour, end_hour, id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (14)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `event_id` | Event | many to one | `event.event` | required; indexed; on delete of the target: cascade |
| `color` | Color | integer |  | default  |
| `date` | Date | date |  | required |
| `date_tz` | Date Tz | selection |  | related through path `event_id.date_tz` |
| `start_hour` | Starting Hour | float |  | required; Help: Expressed in the event timezone. |
| `end_hour` | Ending Hour | float |  | required; Help: Expressed in the event timezone. |
| `start_datetime` | Start Datetime | date and time |  | computed by rule `_compute_datetimes` and stored |
| `end_datetime` | End Datetime | date and time |  | computed by rule `_compute_datetimes` and stored |
| `is_sold_out` | Sold Out | boolean |  | computed by rule `_compute_is_sold_out` (not stored); Help: Whether seats are sold out for this slot. |
| `registration_ids` | Attendees | one to many | `event.registration` | inverse field `event_slot_id` |
| `seats_available` | Available Seats | integer |  | read only; computed by rule `_compute_seats` (not stored) |
| `seats_reserved` | Number of Registrations | integer |  | read only; computed by rule `_compute_seats` (not stored) |
| `seats_taken` | Number of Taken Seats | integer |  | read only; computed by rule `_compute_seats` (not stored) |
| `seats_used` | Number of Attendees | integer |  | read only; computed by rule `_compute_seats` (not stored) |

## Operations (10)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_check_hours` | validation | self | `event` | constrains: `start_hour`, `end_hour` |  |
| `_check_time_range` | validation | self | `event` | constrains: `date`, `start_hour`, `end_hour` |  |
| `_compute_datetimes` | computation | self | `event` | depends: `date`, `date_tz`, `start_hour`, `end_hour` |  |
| `_compute_display_name` | computation | self | `event` | depends: `seats_available`; depends_context: `name_with_seats_availability` | Adds slot seats availability if requested by context. Always display the name without availabilities if the event is multi slots because the availability displayed won't be relative to the possible ticket combinations but only relative to the event and this will confuse the user. |
| `_compute_is_sold_out` | computation | self | `event` | depends: `event_id.seats_limited`, `seats_available` |  |
| `_compute_seats` | computation | self | `event` | depends: `event_id`, `event_id.seats_max`, `registration_ids.state`, `registration_ids.active` |  |
| `_unlink_except_if_registrations` | internal rule | self | `event` | ondelete |  |
| `_filter_open_slots` | internal rule | self | `website_event` |  |  |
| `_load_pos_data_domain` | internal rule | self, data, config | `pos_event` | model |  |
| `_load_pos_data_fields` | internal rule | self, config | `pos_event` | model |  |

## Validation and error messages (4)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_hours` | ValidationError | A slot hour must be between 0:00 and 23:59. | `event` |
| `_check_hours` | ValidationError | A slot end hour must be later than its start hour. %s | `event` |
| `_check_time_range` | ValidationError | A slot cannot be scheduled outside of its event time range.  Event:		%(event_start)s - %(event_end)s Slot:		%(slot_name)s | `event` |
| `_unlink_except_if_registrations` | UserError | The following slots cannot be deleted while they have one or more registrations linked to them: - %s | `event` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `event.group_event_registration_desk` | no | yes | no | no | `event` |
| `event.group_event_user` | yes | yes | yes | yes | `event` |
| `base.group_public` | no | yes | no | no | `website_event` |
| `base.group_portal` | no | yes | no | no | `website_event` |
| `base.group_user` | no | yes | no | no | `website_event` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Event Slot: public/portal: published read | `[(4, ref('base.group_public')), (4, ref('base.group_portal'))]` | `[('event_id.website_published', '=', True)]` | True | False | False | False |

## Views (4)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `event.view_event_slot_form` | form |  | `event_id`, `start_hour`, `end_hour`, `date_tz`, `color` |  |  | `event` |
| `event.view_event_slot_multi_create_form` | form |  | `event_id`, `date_tz`, `color` |  |  | `event` |
| `event.view_event_slot_tree` | list |  | `date`, `start_hour`, `end_hour`, `color` | `durationArrow` |  | `event` |
| `event.view_event_slot_calendar` | calendar |  | `start_datetime`, `end_datetime`, `date_tz` |  |  | `event` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `event.event_slot_action_from_event` | Slots | calendar,list,form | `[('event_id', '=', active_id)]` | `{'default_event_id': active_id}` |  | `event` |

Machine-readable definition: `../../../schemas/data/entities/event.slot.json`; views: `../../../schemas/interfaces/views/event.slot.json`.
