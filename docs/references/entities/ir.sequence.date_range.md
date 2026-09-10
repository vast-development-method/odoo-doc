# Sequence Date Range (`ir.sequence.date_range`)

**Transport name:** `ir.sequence.date_range`  
**Storage name:** `ir_sequence_date_range`  
**Kind:** persistent entity (one table)  
**Defined by package:** `base`

Description: Sequence Date Range

## Identity and behavior

- Display name field: `sequence_id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (5)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `date_from` | From | date |  | required |
| `date_to` | To | date |  | required |
| `sequence_id` | Main Sequence | many to one | `ir.sequence` | required; on delete of the target: cascade |
| `number_next` | Next Number | integer |  | required; default `1`; Help: Next number of this sequence |
| `number_next_actual` | Actual Next Number | integer |  | computed by rule `_get_number_next_actual` (not stored); writable through an inverse rule; Help: Next number that will be used. This number can be incremented frequently so the displayed value might already be obsolete |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_unique_range_per_sequence` | Constraint | `UNIQUE(sequence_id, date_from, date_to)` | You cannot create two date ranges for the same sequence with the same date range. | `base` |

## Operations (8)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_number_next_actual` | preparation rule | self | `base` |  | Return number from ir_sequence row when no_gap implementation, and number from postgres sequence when standard implementation. |
| `_set_number_next_actual` | internal rule | self | `base` |  |  |
| `default_get` | lifecycle override | self, fields | `base` | model |  |
| `_next` | internal rule | self | `base` |  |  |
| `_alter_sequence` | internal rule | self, number_increment, number_next | `base` |  |  |
| `create` | lifecycle override | self, vals_list | `base` | model_create_multi | Create a sequence, in implementation == standard a fast gaps-allowed PostgreSQL sequence is used. |
| `unlink` | lifecycle override | self | `base` |  |  |
| `write` | lifecycle override | self, vals | `base` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_user` | no | yes | no | no | `base` |
| `group_system` | yes | yes | yes | yes | `base` |

Machine-readable definition: `../../../schemas/data/entities/ir.sequence.date_range.json`.
