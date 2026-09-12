# Synchronize a record with Microsoft Calendar (`microsoft.calendar.sync`)

**Transport name:** `microsoft.calendar.sync`  
**Storage name:** `microsoft_calendar_sync`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `microsoft_calendar`

Description: Synchronize a record with Microsoft Calendar

## Fields (4)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `microsoft_id` | Organizer event Id | single line text |  | indexed; not copied on duplication |
| `ms_universal_event_id` | Universal event Id | single line text |  | indexed; not copied on duplication |
| `need_sync_m` | Need Sync M | boolean |  | default `True`; not copied on duplication |
| `active` | Active | boolean |  | default `True` |

## Operations (29)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `write` | lifecycle override | self, vals | `microsoft_calendar` |  |  |
| `create` | lifecycle override | self, vals_list | `microsoft_calendar` | model_create_multi |  |
| `_get_microsoft_service` | preparation rule | self | `microsoft_calendar` | model |  |
| `_get_synced_events` | preparation rule | self | `microsoft_calendar` |  | Get events already synced with Microsoft Outlook. |
| `unlink` | lifecycle override | self | `microsoft_calendar` |  |  |
| `_write_from_microsoft` | internal rule | self, microsoft_event, vals | `microsoft_calendar` |  |  |
| `_create_from_microsoft` | internal rule | self, microsoft_event, vals_list | `microsoft_calendar` | model |  |
| `_sync_system2microsoft` | internal rule | self | `microsoft_calendar` |  |  |
| `_cancel_microsoft` | internal rule | self | `microsoft_calendar` |  |  |
| `_sync_recurrence_microsoft2system` | internal rule | self, microsoft_events, new_events | `microsoft_calendar` |  |  |
| `_update_microsoft_recurrence` | internal rule | self, recurrence, events | `microsoft_calendar` |  | Update the system events from Outlook recurrence and events. |
| `_sync_microsoft2system` | internal rule | self, microsoft_events | `microsoft_calendar` | model | Synchronize Microsoft recurrences in the system. Creates new recurrences, updates existing ones. :return: synchronized system |
| `_check_old_event_update_required` | validation | self, lower_bound_day_range, update_time_diff | `microsoft_calendar` |  | Checks if an old event in the system should be updated locally. This verification is necessary because sometimes events in the system have the same state in Microsoft and even so they trigger updates locally due to a second or less of update time difference, thus spamming unwanted emails on Microsoft side. |
| `_microsoft_delete` | internal rule | self, user_id, event_id, timeout | `microsoft_calendar` |  | Once the event has been really removed from the system database, remove it from the Outlook calendar.  Note that all self attributes to use in this method must be provided as method parameters because 'self' won't exist when this method will be really called due to @after_commit decorator. |
| `_microsoft_patch` | internal rule | self, user_id, event_id, values, timeout | `microsoft_calendar` |  | Once the event has been really modified in the system database, modify it in the Outlook calendar.  Note that all self attributes to use in this method must be provided as method parameters because 'self' may have been modified between the call of '_microsoft_patch' and its execution, due to @after_commit decorator. |
| `_microsoft_insert` | internal rule | self, values, timeout | `microsoft_calendar` |  | Once the event has been really added in the system database, add it in the Outlook calendar.  Note that all self attributes to use in this method must be provided as method parameters because 'self' may have been modified between the call of '_microsoft_insert' and its execution, due to @after_commit decorator. |
| `_microsoft_attendee_answer` | internal rule | self, answer, params, timeout | `microsoft_calendar` |  |  |
| `_get_microsoft_records_to_sync` | preparation rule | self, full_sync | `microsoft_calendar` |  | Return records that should be synced from the system to Microsoft :param full_sync: If True, all events attended by the user are returned :return: events |
| `_microsoft_to_system_values` | internal rule | self, microsoft_event, default_reminders, default_values, with_ids | `microsoft_calendar` | model | Implements this method to return a dict of the system values corresponding to the Microsoft event given as parameter :return: dict of the system formatted values |
| `_get_microsoft_graph_timeout` | preparation rule | self | `microsoft_calendar` | model | Return Microsoft Graph request timeout (seconds).  Keep current behavior by default (5s), but allow admins to increase it through a system parameter. |
| `_microsoft_values` | internal rule | self, fields_to_sync, initial_values | `microsoft_calendar` |  | Implements this method to return a dict with values formatted according to the Microsoft Calendar API :return: dict of Microsoft formatted values |
| `_ensure_attendees_have_email` | internal rule | self | `microsoft_calendar` |  |  |
| `_get_microsoft_sync_domain` | preparation rule | self | `microsoft_calendar` |  | Return a domain used to search records to synchronize. e.g. return a domain to synchronize records owned by the current user. |
| `_get_microsoft_synced_fields` | preparation rule | self | `microsoft_calendar` |  | Return a set of field names. Changing one of these fields marks the record to be re-synchronized. |
| `_restart_microsoft_sync` | internal rule | self | `microsoft_calendar` | model | Turns on the microsoft synchronization for all the events of a given user. |
| `_extend_microsoft_domain` | internal rule | self, domain | `microsoft_calendar` |  | Extends the sync domain based on the full_sync_m context parameter. In case of full sync it shouldn't include already synced events. |
| `_get_event_user_m` | preparation rule | self, user_id | `microsoft_calendar` |  | Return the correct user to send the request to Microsoft. It's possible that a user creates an event and sets another user as the organizer. Using self.env.user will cause some issues, and it might not be possible to use this user for sending the request, so this method gets the appropriate user accordingly. |
| `_need_video_call` | internal rule | self | `microsoft_calendar` |  | Implement this method to return True if the event needs a video call :return: bool |
| `_is_microsoft_insertion_blocked` | internal rule | self, sender_user | `microsoft_calendar` |  | Returns True if the record insertion to Microsoft should be blocked. This is a necessary step for ensuring data match between the system and Microsoft, as it prevents attendees to synchronize new records on behalf of the owners, otherwise the event ownership would be lost in Outlook and it would block the future record synchronization for the original owner. |

Machine-readable definition: `../../../schemas/data/entities/microsoft.calendar.sync.json`.
