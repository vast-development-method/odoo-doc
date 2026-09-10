# Google Calendar Account Reset (`google.calendar.account.reset`)

**Transport name:** `google.calendar.account.reset`  
**Storage name:** `google_calendar_account_reset`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `google_calendar`

Description: Google Calendar Account Reset

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `user_id` | User | many to one | `res.users` | required |
| `delete_policy` | User's Existing Events | selection |  | required; default `dont_delete`; Help: This will only affect events for which the user is the owner |
| `sync_policy` | Next Synchronization | selection |  | required; default `new` |

## Selection values

### `delete_policy` (User's Existing Events)

| Value | Label |
|---|---|
| `dont_delete` | Leave them untouched |
| `delete_google` | Delete from the current Google Calendar account |
| `delete_odoo` | Delete from Odoo |
| `delete_both` | Delete from both |

### `sync_policy` (Next Synchronization)

| Value | Label |
|---|---|
| `new` | Synchronize only new events |
| `all` | Synchronize all existing events |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `reset_account` | operation | self | `google_calendar` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_system` | yes | yes | yes | no | `google_calendar` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `google_calendar.google_calendar_reset_account_view_form` | form |  | `delete_policy`, `sync_policy` | `Confirm`, `Cancel` |  | `google_calendar` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `google_calendar.google_calendar_reset_account_action` |  | form |  |  | new | `google_calendar` |

Machine-readable definition: `../../../schemas/data/entities/google.calendar.account.reset.json`; views: `../../../schemas/interfaces/views/google.calendar.account.reset.json`.
