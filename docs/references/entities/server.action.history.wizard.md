# Server Action History Wizard (`server.action.history.wizard`)

**Transport name:** `server.action.history.wizard`  
**Storage name:** `server_action_history_wizard`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `base`

Description: Server Action History Wizard

## Fields (4)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `action_id` | Action | many to one | `ir.actions.server` |  |
| `code_diff` | Code Diff | rich text |  | computed by rule `_compute_code_diff` (not stored) |
| `current_code` | Current Code | multi line text |  | read only; related through path `action_id.code` |
| `revision` | Revision | many to one | `ir.actions.server.history` | required; default computed dynamically (_default_revision); restricted by domain `[('action_id', '=', action_id), ('code', '!=', current_code)]` |

## Operations (3)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_default_revision` | preparation rule | self | `base` | model |  |
| `_compute_code_diff` | computation | self | `base` | depends: `revision` |  |
| `restore_revision` | operation | self | `base` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_system` | yes | yes | yes | no | `base` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `base.server_action_history_wizard_view` | form |  | `revision`, `code_diff` | `Restore Revision`, `Cancel` |  | `base` |

Machine-readable definition: `../../../schemas/data/entities/server.action.history.wizard.json`; views: `../../../schemas/interfaces/views/server.action.history.wizard.json`.
