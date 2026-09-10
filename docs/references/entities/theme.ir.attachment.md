# Theme Attachments (`theme.ir.attachment`)

**Transport name:** `theme.ir.attachment`  
**Storage name:** `theme_ir_attachment`  
**Kind:** persistent entity (one table)  
**Defined by package:** `website`

Description: Theme Attachments

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (4)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required |
| `key` | Key | single line text |  | required |
| `url` | Uniform resource locator | single line text |  |  |
| `copy_ids` | Attachment using a copy of me | one to many | `ir.attachment` | read only; not copied on duplication; inverse field `theme_template_id` |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_convert_to_base_model` | internal rule | self, website, **kwargs | `website` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_system` | yes | yes | yes | yes | `website` |

Machine-readable definition: `../../../schemas/data/entities/theme.ir.attachment.json`.
