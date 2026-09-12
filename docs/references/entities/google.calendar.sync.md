# Synchronize a record with Google Calendar (`google.calendar.sync`)

**Transport name:** `google.calendar.sync`  
**Storage name:** `google_calendar_sync`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `google_calendar`

Description: Synchronize a record with Google Calendar

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `google_id` | Google Calendar Id | single line text |  | indexed (btree_not_null); not copied on duplication |
| `need_sync` | Need Sync | boolean |  | default `True`; not copied on duplication |
| `active` | Active | boolean |  | default `True` |

## Operations (26)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `write` | lifecycle override | self, vals | `google_calendar` |  |  |
| `create` | lifecycle override | self, vals_list | `google_calendar` | model_create_multi |  |
| `_handle_allday_recurrences_edge_case` | internal rule | self, records, vals_list | `google_calendar` |  | When creating 'All Day' recurrent event, the first event is wrongly synchronized as a single event and then its recurrence creates a duplicated event. We must manually set the 'need_sync' attribute as False in order to avoid this unwanted behavior. |
| `unlink` | lifecycle override | self | `google_calendar` |  | We can't delete an event that is also in Google Calendar. Otherwise we would have no clue that the event must must deleted from Google Calendar at the next sync. |
| `_from_google_ids` | internal rule | self, google_ids | `google_calendar` |  |  |
| `_sync_system2google` | internal rule | self, google_service | `google_calendar` |  |  |
| `_cancel` | internal rule | self | `google_calendar` |  |  |
| `_sync_google2system` | internal rule | self, google_events, write_dates, default_reminders | `google_calendar` | model | Synchronize Google recurrences in the system. Creates new recurrences, updates existing ones.  :param google_events: Google recurrences to synchronize in the system :param write_dates: A dictionary mapping the system record IDs to their write dates. :param default_reminders: :return: synchronized system recurrences |
| `_google_error_handling` | internal rule | self, http_error | `google_calendar` |  |  |
| `_google_delete` | internal rule | self, google_service, google_id, timeout | `google_calendar` |  |  |
| `_google_patch` | internal rule | self, google_service, google_id, values, timeout | `google_calendar` |  |  |
| `_get_post_sync_values` | preparation rule | self, request_values, google_values | `google_calendar` |  | Return the values to be written in the event right after its insertion in Google side. |
| `_need_video_call` | internal rule | self | `google_calendar` |  | Implement this method to return True if the event needs a video call :return: bool |
| `_google_insert` | internal rule | self, google_service, values, timeout | `google_calendar` |  |  |
| `_get_records_to_sync` | preparation rule | self, full_sync | `google_calendar` |  | Return records that should be synced from the system to Google  :param full_sync: If True, all events attended by the user are returned :return: events |
| `_check_any_records_to_sync` | validation | self | `google_calendar` |  | Returns True if there are pending records to be synchronized from the system to Google, False otherwise. |
| `_write_from_google` | internal rule | self, gevent, vals | `google_calendar` |  |  |
| `_create_from_google` | internal rule | self, gevents, vals_list | `google_calendar` | model |  |
| `_get_sync_partner` | preparation rule | self, emails | `google_calendar` | model |  |
| `_system_values` | internal rule | self, google_event, default_reminders | `google_calendar` | model | Implements this method to return a dict of the system values corresponding to the Google event given as parameter :return: dict of the system formatted values |
| `_google_values` | internal rule | self | `google_calendar` |  | Implements this method to return a dict with values formatted according to the Google Calendar API :return: dict of Google formatted values |
| `_get_sync_domain` | preparation rule | self | `google_calendar` |  | Return a domain used to search records to synchronize. e.g. return a domain to synchronize records owned by the current user. |
| `_get_google_synced_fields` | preparation rule | self | `google_calendar` |  | Return a set of field names. Changing one of these fields marks the record to be re-synchronized. |
| `_restart_google_sync` | internal rule | self | `google_calendar` | model | Turns on the google synchronization for all the events of a given user. |
| `_get_event_user` | preparation rule | self | `google_calendar` |  | Return the correct user to send the request to Google. It's possible that a user creates an event and sets another user as the organizer. Using self.env.user will cause some issues, and It might not be possible to use this user for sending the request, so this method gets the appropriate user accordingly. |
| `_is_google_insertion_blocked` | internal rule | self, sender_user | `google_calendar` |  | Returns True if the record insertion to Google should be blocked. This is a necessary step for ensuring data match between the system and Google, as it avoids that events have permanently the wrong organizer in Google by not synchronizing records through owner and not through the attendees. |

Machine-readable definition: `../../../schemas/data/entities/google.calendar.sync.json`.
