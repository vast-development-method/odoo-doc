# Account Move Send Batch Wizard (`account.move.send.batch.wizard`)

**Transport name:** `account.move.send.batch.wizard`  
**Storage name:** `account_move_send_batch_wizard`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `account`  
**Extended by packages:** `account_peppol`, `l10n_dk_nemhandel`, `snailmail_account`

Description: Account Move Send Batch Wizard

## Identity and behavior

- Mixins (classical inheritance): `account.move.send`

## Fields (4)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `move_ids` | Move | many to many | `account.move` | required |
| `summary_data` | Summary Data | structured document |  | computed by rule `_compute_summary_data` (not stored) |
| `alerts` | Alerts | structured document |  | computed by rule `_compute_alerts` (not stored) |
| `send_by_post_stamps` | Send By Post Stamps | integer |  | computed by rule `_compute_send_by_post_stamps` (not stored) |

## Operations (6)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `default_get` | lifecycle override | self, fields | `account` | model |  |
| `_compute_summary_data` | computation | self | `account`, `snailmail_account` | depends: `move_ids` |  |
| `_compute_alerts` | computation | self | `account` | depends: `summary_data` |  |
| `_check_move_ids_constraints` | validation | self | `account` | constrains: `move_ids` |  |
| `action_send_and_print` | user action | self, force_synchronous, allow_fallback_pdf | `account_peppol`, `account`, `l10n_dk_nemhandel` |  | Launch asynchronously the generation and sending of invoices. |
| `_compute_send_by_post_stamps` | computation | self | `snailmail_account` |  |  |

## Validation and error messages (2)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `action_send_and_print` | UserError | Batch invoice sending is unavailable. Please, contact your system administrator to activate the cron to enable batch sending of invoices. | `account` |
| `action_send_and_print` | RedirectWarning | Batch invoice sending is unavailable. Please, activate the cron to enable batch sending of invoices. | `account` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_invoice` | yes | yes | yes | yes | `account` |
| `sales_team.group_sale_salesman` | yes | yes | yes | no | `sale` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Readonly Invoice Send and Print (batch) | `[(4, ref('account.group_account_invoice'))]` | `[(1, '=', 1)]` | True | True | True | True |
| Personal Invoice Send and Print (batch mode) | `[(4, ref('sales_team.group_sale_salesman'))]` | `[('move_ids.move_type', 'in', ('out_invoice', 'out_refund')), '\|', ('move_ids.invoice_user_id', '=', user.id), ('move_ids.invoice_user_id', '=', False)]` | True | True | True | True |
| All Invoice Send and Print (batch mode) | `[(4, ref('sales_team.group_sale_salesman_all_leads'))]` | `[('move_ids.move_type', 'in', ('out_invoice', 'out_refund'))]` | True | True | True | True |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `account.account_move_send_batch_wizard_form` | form |  | `move_ids`, `alerts`, `summary_data` | `Send`, `Cancel` |  | `account` |

Machine-readable definition: `../../../schemas/data/entities/account.move.send.batch.wizard.json`; views: `../../../schemas/interfaces/views/account.move.send.batch.wizard.json`.
