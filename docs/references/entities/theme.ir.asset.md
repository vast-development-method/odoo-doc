# Theme Asset (`theme.ir.asset`)

**Transport name:** `theme.ir.asset`  
**Storage name:** `theme_ir_asset`  
**Kind:** persistent entity (one table)  
**Defined by package:** `website`

Description: Theme Asset

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (9)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `key` | Key | single line text |  |  |
| `name` | Name | single line text |  | required |
| `bundle` | Bundle | single line text |  | required |
| `directive` | Directive | selection |  | default computed dynamically (APPEND_DIRECTIVE) |
| `path` | Path | single line text |  | required |
| `target` | Target | single line text |  |  |
| `active` | Active | boolean |  | default `True` |
| `sequence` | Sequence | integer |  | required; default computed dynamically (DEFAULT_SEQUENCE) |
| `copy_ids` | Assets using a copy of me | one to many | `ir.asset` | read only; not copied on duplication; inverse field `theme_template_id` |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_convert_to_base_model` | internal rule | self, website, **kwargs | `website` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_system` | yes | yes | yes | yes | `website` |

Machine-readable definition: `../../../schemas/data/entities/theme.ir.asset.json`.
