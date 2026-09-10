# Wizard to post Manufacturing work in progress account move (`mrp.account.wip.accounting`)

**Transport name:** `mrp.account.wip.accounting`  
**Storage name:** `mrp_account_wip_accounting`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `mrp_account`

Description: Wizard to post Manufacturing WIP account move

## Fields (6)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `date` | Date | date |  | default computed dynamically (fields.Datetime.now) |
| `reversal_date` | Reversal Date | date |  | required; computed by rule `_compute_reversal_date` and stored |
| `journal_id` | Journal | many to one | `account.journal` | required |
| `reference` | Reference | single line text |  |  |
| `line_ids` | work in progress accounting lines | one to many | `mrp.account.wip.accounting.line` | computed by rule `_compute_line_ids` and stored; inverse field `wip_accounting_id` |
| `mo_ids` | Manufacturing order | many to many | `mrp.production` |  |

## Operations (6)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `default_get` | lifecycle override | self, fields | `mrp_account` | model |  |
| `_get_overhead_account` | preparation rule | self | `mrp_account` |  |  |
| `_get_line_vals` | preparation rule | self, productions, date | `mrp_account` |  |  |
| `_compute_reversal_date` | computation | self | `mrp_account` | depends: `date` |  |
| `_compute_line_ids` | computation | self | `mrp_account` | depends: `date` |  |
| `confirm` | operation | self | `mrp_account` |  |  |

## Validation and error messages (2)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `confirm` | UserError | Please make sure the total credit amount equals the total debit amount. | `mrp_account` |
| `confirm` | UserError | Reversal date must be after the posting date. | `mrp_account` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_manager` | yes | yes | yes | no | `mrp_account` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mrp_account.view_wip_accounting_form` | form |  | `journal_id`, `reference`, `date`, `reversal_date`, `mo_ids`, `line_ids`, `account_id`, `label`, `debit`, `credit`, `currency_id` | `Post WIP`, `Discard` |  | `mrp_account` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `mrp_account.action_wip_accounting` | Post WIP Accounting Entry | form |  |  | new | `mrp_account` |

Machine-readable definition: `../../../schemas/data/entities/mrp.account.wip.accounting.json`; views: `../../../schemas/interfaces/views/mrp.account.wip.accounting.json`.
