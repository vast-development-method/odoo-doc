# Calendar Attendee Information (`calendar.attendee`)

**Transport name:** `calendar.attendee`  
**Storage name:** `calendar_attendee`  
**Kind:** persistent entity (one table)  
**Defined by package:** `calendar`  
**Extended by packages:** `google_calendar`, `microsoft_calendar`

Description: Calendar Attendee Information

## Identity and behavior

- Default ordering: `create_date ASC`
- Display name field: `common_name`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (10)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `event_id` | Meeting linked | many to one | `calendar.event` | required; indexed; on delete of the target: cascade |
| `recurrence_id` | Recurrence | many to one | `calendar.recurrence` | related through path `event_id.recurrence_id` |
| `partner_id` | Attendee | many to one | `res.partner` | required; read only; on delete of the target: cascade |
| `email` | Email | single line text |  | related through path `partner_id.email` |
| `phone` | Phone | single line text |  | related through path `partner_id.phone` |
| `common_name` | Common name | single line text |  | computed by rule `_compute_common_name` and stored |
| `access_token` | Invitation Token | single line text |  | default computed dynamically (_default_access_token) |
| `mail_tz` | Mail Tz | selection |  | computed by rule `_compute_mail_tz` (not stored); Help: Timezone used for displaying time in the mail template |
| `state` | Status | selection |  | default `needsAction` |
| `availability` | Available/Busy | selection |  | read only |

## Selection values

### `availability` (Available/Busy)

| Value | Label |
|---|---|
| `free` | Available |
| `busy` | Busy |

## State fields

State machine fields of this entity: `state`. Transitions are specified in the domain documents.

## Operations (18)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_default_access_token` | preparation rule | self | `calendar` |  |  |
| `_compute_common_name` | computation | self | `calendar` | depends: `partner_id`, `partner_id.name`, `email` |  |
| `_compute_mail_tz` | computation | self | `calendar` |  |  |
| `create` | lifecycle override | self, vals_list | `calendar` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `calendar` |  |  |
| `unlink` | lifecycle override | self | `calendar` |  |  |
| `copy` | lifecycle override | self, default | `calendar` |  |  |
| `_unsubscribe_partner` | internal rule | self | `calendar` |  |  |
| `_mail_template_default_values` | messaging hook | self | `calendar` | model |  |
| `_message_add_default_recipients` | messaging hook | self | `calendar` |  |  |
| `_send_invitation_emails` | internal rule | self | `calendar` |  | Hook to be able to override the invitation email sending process. Notably inside appointment to use a different mail template from the appointment type. |
| `_notify_attendees` | internal rule | self, mail_template, notify_author, force_send | `calendar` |  | Notify attendees about event main changes (invite, cancel, ...) based on template.  :param mail_template: a mail.template record :param force_send: if set to True, the mail(s) will be sent immediately (instead of the next queue processing) |
| `_should_notify_attendee` | internal rule | self, notify_author | `calendar` |  | Utility method that determines if the attendee should be notified. By default, we do not want to notify (aka no message and no mail) the current user if he is part of the attendees. But for reminders, mail_notify_author could be forced (Override in appointment to ignore that rule and notify all attendees if it's an appointment) |
| `do_tentative` | user action | self | `calendar`, `google_calendar`, `microsoft_calendar` |  | Makes event invitation as Tentative. |
| `do_accept` | user action | self | `calendar`, `google_calendar`, `microsoft_calendar` |  | Marks event invitation as Accepted. |
| `do_decline` | user action | self | `calendar`, `google_calendar`, `microsoft_calendar` |  | Marks event invitation as Declined. |
| `_sync_event` | internal rule | self | `google_calendar` |  |  |
| `_microsoft_sync_event` | internal rule | self, answer | `microsoft_calendar` |  |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `copy` | UserError | You cannot duplicate a calendar attendee. | `calendar` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_portal` | no | no | no | no | `calendar` |
| `base.group_user` | yes | yes | yes | yes | `calendar` |
| `hr_holidays.group_hr_holidays_user` | yes | yes | yes | yes | `hr_holidays` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Own attendees | `[(4, ref('base.group_portal'))]` | `[(1, '=', 1)]` | True | True | True | True |

## Email templates

| Template | Name | Subject |
|---|---|---|
| `calendar.calendar_template_meeting_invitation` | Calendar: Meeting Invitation | Invitation to {{ object.event_id.name }} |
| `calendar.calendar_template_meeting_changedate` | Calendar: Date Updated | {{ object.event_id.name }}: Date updated |
| `calendar.calendar_template_meeting_reminder` | Calendar: Reminder | {{ object.event_id.name }} - Reminder |

Machine-readable definition: `../../../schemas/data/entities/calendar.attendee.json`.
