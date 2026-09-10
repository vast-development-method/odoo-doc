# Privacy Lookup Wizard (`privacy.lookup.wizard`)

**Transport name:** `privacy.lookup.wizard`  
**Storage name:** `privacy_lookup_wizard`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `privacy_lookup`

Description: Privacy Lookup Wizard

## Identity and behavior

- Transient maximum hours: 24

## Fields (7)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required |
| `email` | Email | single line text |  | required |
| `line_ids` | Line | one to many | `privacy.lookup.wizard.line` | inverse field `wizard_id` |
| `execution_details` | Execution Details | multi line text |  | computed by rule `_compute_execution_details` and stored |
| `log_id` | Log | many to one | `privacy.log` |  |
| `records_description` | Records Description | multi line text |  | computed by rule `_compute_records_description` (not stored) |
| `line_count` | Line Count | integer |  | computed by rule `_compute_line_count` (not stored) |

## Operations (9)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_line_count` | computation | self | `privacy_lookup` | depends: `line_ids` |  |
| `_compute_display_name` | computation | self | `privacy_lookup` |  |  |
| `_get_query_models_blacklist` | preparation rule | self | `privacy_lookup` |  |  |
| `_get_query` | preparation rule | self | `privacy_lookup` |  |  |
| `action_lookup` | user action | self | `privacy_lookup` |  |  |
| `_post_log` | internal rule | self | `privacy_lookup` |  |  |
| `_compute_execution_details` | computation | self | `privacy_lookup` | depends: `line_ids.execution_details` |  |
| `_compute_records_description` | computation | self | `privacy_lookup` | depends: `line_ids` |  |
| `action_open_lines` | user action | self | `privacy_lookup` |  |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_get_query` | UserError | Invalid email address “%s” | `privacy_lookup` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_system` | yes | yes | yes | yes | `privacy_lookup` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `privacy_lookup.privacy_lookup_wizard_view_form` | form |  | `line_count`, `email`, `name`, `records_description`, `execution_details`, `log_id`, `line_ids` | `Lookup`, `action_open_lines` |  | `privacy_lookup` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `privacy_lookup.action_privacy_lookup_wizard` | Privacy Lookup | form |  |  | current | `privacy_lookup` |

Machine-readable definition: `../../../schemas/data/entities/privacy.lookup.wizard.json`; views: `../../../schemas/interfaces/views/privacy.lookup.wizard.json`.
