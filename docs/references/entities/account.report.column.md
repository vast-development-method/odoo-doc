# Accounting Report Column (`account.report.column`)

**Transport name:** `account.report.column`  
**Storage name:** `account_report_column`  
**Kind:** persistent entity (one table)  
**Defined by package:** `account`

Description: Accounting Report Column

## Identity and behavior

- Default ordering: `sequence, id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (8)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required; translatable |
| `expression_label` | Expression Label | single line text |  | required |
| `sequence` | Sequence | integer |  |  |
| `report_id` | Report | many to one | `account.report` | indexed (btree_not_null) |
| `sortable` | Sortable | boolean |  |  |
| `figure_type` | Figure Type | selection |  | required; default `monetary` |
| `blank_if_zero` | Blank if Zero | boolean |  | Help: When checked, 0 values will not show in this column. |
| `custom_audit_action_id` | Custom Audit Action | many to one | `ir.actions.act_window` |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_basic` | no | yes | no | no | `account` |
| `account.group_account_readonly` | no | yes | no | no | `account` |
| `account.group_account_manager` | yes | yes | yes | yes | `account` |

Machine-readable definition: `../../../schemas/data/entities/account.report.column.json`.
