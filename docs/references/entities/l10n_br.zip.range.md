# Brazilian city zip range (`l10n_br.zip.range`)

**Transport name:** `l10n_br.zip.range`  
**Storage name:** `l10n_br_zip_range`  
**Kind:** persistent entity (one table)  
**Defined by package:** `l10n_br`

Description: Brazilian city zip range

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `city_id` | City | many to one | `res.city` | required |
| `start` | From | single line text |  | required |
| `end` | To | single line text |  | required |

## Database constraints and indexes (2)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_uniq_start` | Constraint | `unique(start)` | The "from" zip must be unique | `l10n_br` |
| `_uniq_end` | Constraint | `unique("end")` | The "to" zip must be unique. | `l10n_br` |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_check_range` | validation | self | `l10n_br` | constrains: `start`, `end` |  |

## Validation and error messages (2)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_range` | ValidationError | Invalid zip range format: %(start)s %(end)s. It should follow this format: 01000-001 | `l10n_br` |
| `_check_range` | ValidationError | Start should be less than end: %(start)s %(end)s | `l10n_br` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_partner_manager` | yes | yes | yes | yes | `l10n_br` |
| `base.group_user` | no | yes | no | no | `l10n_br` |

Machine-readable definition: `../../../schemas/data/entities/l10n_br.zip.range.json`.
