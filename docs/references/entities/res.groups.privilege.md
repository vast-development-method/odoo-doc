# Privileges (`res.groups.privilege`)

**Transport name:** `res.groups.privilege`  
**Storage name:** `res_groups_privilege`  
**Kind:** persistent entity (one table)  
**Defined by package:** `base`

Description: Privileges

## Identity and behavior

- Default ordering: `sequence, name, id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (6)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required; translatable |
| `description` | Description | multi line text |  |  |
| `placeholder` | Placeholder | single line text |  | default `No`; Help: Text that is displayed as placeholder in the selection field of the user form. |
| `sequence` | Sequence | integer |  | default `100` |
| `category_id` | Category | many to one | `ir.module.category` | indexed |
| `group_ids` | Groups | one to many | `res.groups` | inverse field `privilege_id` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_user` | no | yes | no | no | `base` |
| `group_erp_manager` | yes | yes | yes | yes | `base` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `base.view_res_groups_privilege_list` | list |  | `name`, `group_ids` |  |  | `base` |
| `base.view_res_groups_privilege_form` | form |  | `name`, `sequence`, `category_id`, `placeholder`, `group_ids`, `sequence`, `name`, `description` |  |  | `base` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `base.action_res_groups_privilege` | Privileges |  |  |  |  | `base` |

Machine-readable definition: `../../../schemas/data/entities/res.groups.privilege.json`; views: `../../../schemas/interfaces/views/res.groups.privilege.json`.
