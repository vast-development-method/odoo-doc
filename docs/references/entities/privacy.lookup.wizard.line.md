# Privacy Lookup Wizard Line (`privacy.lookup.wizard.line`)

**Transport name:** `privacy.lookup.wizard.line`  
**Storage name:** `privacy_lookup_wizard_line`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `privacy_lookup`

Description: Privacy Lookup Wizard Line

## Identity and behavior

- Transient maximum hours: 24

## Fields (10)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `wizard_id` | Wizard | many to one | `privacy.lookup.wizard` |  |
| `res_id` | Resource identifier | integer |  | required |
| `res_name` | Resource name | single line text |  | computed by rule `_compute_res_name` and stored |
| `res_model_id` | Related Document Model | many to one | `ir.model` | on delete of the target: cascade |
| `res_model` | Document Model | single line text |  | read only; related through path `res_model_id.model` and stored |
| `resource_ref` | Record | reference |  | computed by rule `_compute_resource_ref` (not stored); writable through an inverse rule; values provided by rule `_selection_target_model` |
| `has_active` | Has Active | boolean |  | computed by rule `_compute_has_active` and stored |
| `is_active` | Is Active | boolean |  |  |
| `is_unlinked` | Is Unlinked | boolean |  |  |
| `execution_details` | Execution Details | single line text |  | default  |

## Operations (10)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_selection_target_model` | internal rule | self | `privacy_lookup` | model |  |
| `_compute_resource_ref` | computation | self | `privacy_lookup` | depends: `res_model`, `res_id`, `is_unlinked` |  |
| `_set_resource_ref` | internal rule | self | `privacy_lookup` |  |  |
| `_compute_has_active` | computation | self | `privacy_lookup` | depends: `res_model_id` |  |
| `_compute_res_name` | computation | self | `privacy_lookup` | depends: `res_model`, `res_id` |  |
| `_onchange_is_active` | on change | self | `privacy_lookup` | onchange: `is_active` |  |
| `action_unlink` | user action | self | `privacy_lookup` |  |  |
| `action_archive_all` | user action | self | `privacy_lookup` |  |  |
| `action_unlink_all` | user action | self | `privacy_lookup` |  |  |
| `action_open_record` | user action | self | `privacy_lookup` |  |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `action_unlink` | UserError | The record is already unlinked. | `privacy_lookup` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_system` | yes | yes | yes | no | `privacy_lookup` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `privacy_lookup.privacy_lookup_wizard_line_view_tree` | list |  | `res_model_id`, `res_name`, `res_model`, `resource_ref`, `res_id`, `has_active`, `execution_details`, `is_active`, `is_unlinked` | `action_open_record`, `Delete` |  | `privacy_lookup` |
| `privacy_lookup.privacy_lookup_wizard_line_view_search` | search |  | `res_model_id`, `has_active`, `is_active` |  | `Can be archived`, `Archived`, `Model` | `privacy_lookup` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `privacy_lookup.action_privacy_lookup_wizard_line` | Privacy Lookup Line | list |  | `{'search_default_group_by_res_model_id': 1, 'no_create_edit': True}` | current | `privacy_lookup` |

## Server actions

| Action | Name | State | Binding | Custom logic |
|---|---|---|---|---|
| `privacy_lookup.ir_actions_server_archive_all` | Archive Selection | code |  | yes |
| `privacy_lookup.ir_actions_server_unlink_all` | Delete Selection | code |  | yes |

Machine-readable definition: `../../../schemas/data/entities/privacy.lookup.wizard.line.json`; views: `../../../schemas/interfaces/views/privacy.lookup.wizard.line.json`.
