# Full Reconcile (`account.full.reconcile`)

**Transport name:** `account.full.reconcile`  
**Storage name:** `account_full_reconcile`  
**Kind:** persistent entity (one table)  
**Defined by package:** `account`

Description: Full Reconcile

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `partial_reconcile_ids` | Reconciliation Parts | one to many | `account.partial.reconcile` | inverse field `full_reconcile_id` |
| `reconciled_line_ids` | Matched Journal Items | one to many | `account.move.line` | inverse field `full_reconcile_id` |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `create` | lifecycle override | self, vals_list | `account` | model_create_multi |  |
| `unlink` | lifecycle override | self | `account` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_readonly` | no | yes | no | no | `account` |
| `account.group_account_invoice` | yes | yes | yes | yes | `account` |
| `account.group_account_user` | yes | yes | yes | yes | `account` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `account.view_full_reconcile_form` | form |  | `id`, `reconciled_line_ids` |  |  | `account` |

Machine-readable definition: `../../../schemas/data/entities/account.full.reconcile.json`; views: `../../../schemas/interfaces/views/account.full.reconcile.json`.
