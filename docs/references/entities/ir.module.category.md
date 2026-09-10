# Application (`ir.module.category`)

**Transport name:** `ir.module.category`  
**Storage name:** `ir_module_category`  
**Kind:** persistent entity (one table)  
**Defined by package:** `base`

Description: Application

## Identity and behavior

- Default ordering: `sequence, name, id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (10)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required; translatable |
| `parent_id` | Parent Application | many to one | `ir.module.category` | indexed |
| `child_ids` | Child Applications | one to many | `ir.module.category` | inverse field `parent_id` |
| `module_ids` | Modules | one to many | `ir.module.module` | inverse field `category_id` |
| `privilege_ids` | Privileges | one to many | `res.groups.privilege` | inverse field `category_id` |
| `description` | Description | multi line text |  | translatable |
| `sequence` | Sequence | integer |  |  |
| `visible` | Visible | boolean |  | default `True` |
| `exclusive` | Exclusive | boolean |  |  |
| `xml_id` | External identifier | single line text |  | computed by rule `_compute_xml_id` (not stored) |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_xml_id` | computation | self | `base` |  |  |
| `_check_parent_not_circular` | validation | self | `base` | constrains: `parent_id` |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_parent_not_circular` | ValidationError | Error ! You cannot create recursive categories. | `base` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_erp_manager` | no | yes | no | no | `base` |
| `base.group_user` | no | yes | no | no | `base_install_request` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `base.view_module_category_form` | form |  | `name`, `parent_id`, `sequence`, `description` |  |  | `base` |

Machine-readable definition: `../../../schemas/data/entities/ir.module.category.json`; views: `../../../schemas/interfaces/views/ir.module.category.json`.
