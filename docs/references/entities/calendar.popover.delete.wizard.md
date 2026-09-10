# Calendar Popover Delete Wizard (`calendar.popover.delete.wizard`)

**Transport name:** `calendar.popover.delete.wizard`  
**Storage name:** `calendar_popover_delete_wizard`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `calendar`

Description: Calendar Popover Delete Wizard

## Identity and behavior

- Mixins (classical inheritance): `mail.composer.mixin`

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `calendar_event_id` | Calendar Event | many to one | `calendar.event` |  |
| `delete` | Delete | selection |  | default `one` |
| `recipient_ids` | Recipients | many to many | `res.partner` | computed by rule `_compute_recipient_ids` (not stored) |

## Selection values

### `delete` (Delete)

| Value | Label |
|---|---|
| `one` | Delete this event |
| `next` | Delete this and following events |
| `all` | Delete all the events |

## Operations (6)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `close` | operation | self | `calendar` |  |  |
| `_compute_recipient_ids` | computation | self | `calendar` | depends: `calendar_event_id` | Compute the recipients by combining the record's partner and attendees partners. |
| `_compute_subject` | computation | self | `calendar` | depends: `calendar_event_id` | Compute the subject by rendering the template's subject field based on the event. |
| `_compute_body` | computation | self | `calendar` | depends: `calendar_event_id` | Compute the body by rendering the template's body HTML field based on the event. |
| `action_delete` | user action | self | `calendar` |  | Delete the event based on the specified deletion type.  :return: Action URL to redirect to the calendar view |
| `action_send_mail_and_delete` | user action | self | `calendar` |  | Send email notification and delete the event based on the specified deletion type. |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | yes | yes | yes | yes | `calendar` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `calendar.calendar_popover_delete_view` | form |  | `delete` | `Submit`, `Cancel` |  | `calendar` |
| `calendar.view_event_delete_wizard_form` | form |  | `calendar_event_id`, `recipient_ids`, `subject`, `body` | `Send and delete`, `Delete`, `Discard` |  | `calendar` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `calendar.action_event_delete_wizard` | Event Cancel Wizard | form |  | `{}` | new | `calendar` |

Machine-readable definition: `../../../schemas/data/entities/calendar.popover.delete.wizard.json`; views: `../../../schemas/interfaces/views/calendar.popover.delete.wizard.json`.
