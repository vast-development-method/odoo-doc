# Grant Portal Access (`portal.wizard`)

**Transport name:** `portal.wizard`  
**Storage name:** `portal_wizard`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `portal`

Description: Grant Portal Access

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `partner_ids` | Partners | many to many | `res.partner` | default computed dynamically (_default_partner_ids) |
| `user_ids` | Users | one to many | `portal.wizard.user` | computed by rule `_compute_user_ids` and stored; inverse field `wizard_id` |
| `welcome_message` | Invitation Message | multi line text |  | Help: This text is included in the email sent to new users of the portal. |

## Operations (4)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_default_partner_ids` | preparation rule | self | `portal` |  |  |
| `_compute_user_ids` | computation | self | `portal` | depends: `partner_ids` |  |
| `action_open_wizard` | user action | self | `portal` | model | Create a "portal.wizard" and open the form view.  We need a server action for that because the one2many "user_ids" records need to exist to be able to execute an a button action on it. If they have no ID, the buttons will be disabled and we won't be able to click on them.  That's why we need a server action, to create the records and then open the form view on them. |
| `_action_open_modal` | internal rule | self | `portal` |  | Allow to keep the wizard modal open after executing the action. |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_partner_manager` | yes | yes | yes | no | `portal` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `portal.wizard_view` | form |  | `welcome_message`, `user_ids`, `partner_id`, `email`, `email_state`, `login_date`, `is_portal`, `is_internal` | `action_refresh_modal`, `action_refresh_modal`, `action_refresh_modal`, `Grant Access`, `Revoke Access`, `Re-Invite`, `Internal User`, `Close` |  | `portal` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `portal.partner_wizard_action` | Grant portal access | form |  |  | new | `portal` |

## Server actions

| Action | Name | State | Binding | Custom logic |
|---|---|---|---|---|
| `portal.partner_wizard_action_create_and_open` | Grant portal access | code |  | yes |

Machine-readable definition: `../../../schemas/data/entities/portal.wizard.json`; views: `../../../schemas/interfaces/views/portal.wizard.json`.
