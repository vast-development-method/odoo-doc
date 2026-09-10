# Theme user interface View (`theme.ir.ui.view`)

**Transport name:** `theme.ir.ui.view`  
**Storage name:** `theme_ir_ui_view`  
**Kind:** persistent entity (one table)  
**Defined by package:** `website`

Description: Theme UI View

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (11)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required |
| `key` | Key | single line text |  |  |
| `type` | Type | single line text |  |  |
| `priority` | Priority | integer |  | required; default computed dynamically (DEFAULT_SEQUENCE) |
| `mode` | Mode | selection |  |  |
| `active` | Active | boolean |  | default `True` |
| `arch` | Arch | multi line text |  | translatable |
| `arch_fs` | Arch Fs | single line text |  | default computed dynamically (compute_arch_fs) |
| `inherit_id` | Inherit | reference |  |  |
| `copy_ids` | Views using a copy of me | one to many | `ir.ui.view` | read only; not copied on duplication; inverse field `theme_template_id` |
| `customize_show` | Customize Show | boolean |  |  |

## Selection values

### `mode` (Mode)

| Value | Label |
|---|---|
| `primary` | Base view |
| `extension` | Extension View |

### `inherit_id` (Inherit)

| Value | Label |
|---|---|
| `ir.ui.view` | ir.ui.view |
| `theme.ir.ui.view` | theme.ir.ui.view |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `compute_arch_fs` | operation | self | `website` |  |  |
| `_convert_to_base_model` | internal rule | self, website, **kwargs | `website` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_system` | yes | yes | yes | yes | `website` |

Machine-readable definition: `../../../schemas/data/entities/theme.ir.ui.view.json`.
