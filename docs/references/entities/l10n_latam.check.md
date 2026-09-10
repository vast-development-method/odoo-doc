# Account payment check (`l10n_latam.check`)

**Transport name:** `l10n_latam.check`  
**Storage name:** `l10n_latam_check`  
**Kind:** persistent entity (one table)  
**Defined by package:** `l10n_latam_check`

Description: Account payment check

## Identity and behavior

- Mixins (classical inheritance): `mail.thread`, `mail.activity.mixin`
- Company consistency is checked automatically on company-bound relations
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (16)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `payment_id` | Payment | many to one | `account.payment` | required; on delete of the target: cascade |
| `operation_ids` | Operation | many to many | `account.payment` | read only; must belong to the same company; association table `l10n_latam_check_account_payment_rel` |
| `current_journal_id` | Current Journal | many to one | `account.journal` | computed by rule `_compute_current_journal` and stored |
| `name` | Number | single line text |  |  |
| `bank_id` | Bank | many to one | `res.bank` | computed by rule `_compute_bank_id` and stored |
| `issuer_vat` | Issuer Value-added tax | single line text |  | computed by rule `_compute_issuer_vat` and stored |
| `payment_date` | Payment Date | date |  | required |
| `amount` | Amount | monetary |  |  |
| `outstanding_line_id` | Outstanding Line | many to one | `account.move.line` | read only; must belong to the same company |
| `issue_state` | Issue State | selection |  | computed by rule `_compute_issue_state` and stored |
| `payment_method_code` | Payment Method Code | single line text |  | related through path `payment_id.payment_method_code` |
| `partner_id` | Partner | many to one |  | related through path `payment_id.partner_id` |
| `original_journal_id` | Original Journal | many to one |  | related through path `payment_id.journal_id` |
| `company_id` | Company | many to one |  | related through path `payment_id.company_id` and stored |
| `currency_id` | Currency | many to one |  | related through path `payment_id.currency_id` |
| `payment_method_line_id` | Payment Method Line | many to one |  | related through path `payment_id.payment_method_line_id` and stored |

## Selection values

### `issue_state` (Issue State)

| Value | Label |
|---|---|
| `handed` | Handed |
| `debited` | Debited |
| `voided` | Voided |

## State fields

State machine fields of this entity: `issue_state`. Transitions are specified in the domain documents.

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_unique` | UniqueIndex | `(name, payment_method_line_id) WHERE outstanding_line_id IS NOT NULL` |  | `l10n_latam_check` |

## Operations (17)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_onchange_name` | on change | self | `l10n_latam_check` | onchange: `name` |  |
| `_prepare_void_move_vals` | preparation rule | self | `l10n_latam_check` |  |  |
| `_compute_issue_state` | computation | self | `l10n_latam_check` | depends: `outstanding_line_id.amount_residual` |  |
| `action_void` | user action | self | `l10n_latam_check` |  |  |
| `_get_last_operation` | preparation rule | self | `l10n_latam_check` |  |  |
| `_compute_current_journal` | computation | self | `l10n_latam_check` | depends: `payment_id.state`, `operation_ids.state` |  |
| `button_open_payment` | user action | self | `l10n_latam_check` |  |  |
| `button_open_check_operations` | user action | self | `l10n_latam_check` |  | Redirect the user to the invoice(s) paid by this payment. :return:    An action on account.move. |
| `action_show_reconciled_move` | user action | self | `l10n_latam_check` |  |  |
| `action_show_journal_entry` | user action | self | `l10n_latam_check` |  |  |
| `_get_reconciled_move` | preparation rule | self | `l10n_latam_check` |  |  |
| `_constrains_min_amount` | validation | self | `l10n_latam_check` | constrains: `amount` |  |
| `_compute_bank_id` | computation | self | `l10n_latam_check` | depends: `payment_method_line_id.code`, `payment_id.partner_id` |  |
| `_compute_issuer_vat` | computation | self | `l10n_latam_check` | depends: `payment_method_line_id.code`, `payment_id.partner_id` |  |
| `_clean_issuer_vat` | on change | self | `l10n_latam_check` | onchange: `issuer_vat` |  |
| `_check_issuer_vat` | validation | self | `l10n_latam_check` | constrains: `issuer_vat` |  |
| `_unlink_if_payment_is_draft` | internal rule | self | `l10n_latam_check` | ondelete |  |

## Validation and error messages (2)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_constrains_min_amount` | ValidationError | The amount of the check must be greater than 0 | `l10n_latam_check` |
| `_unlink_if_payment_is_draft` | UserError | Can't delete a check if payment is In Process! | `l10n_latam_check` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_readonly` | no | yes | no | no | `l10n_latam_check` |
| `account.group_account_invoice` | yes | yes | yes | yes | `l10n_latam_check` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Latam Check company rule | global (all users) | `[('company_id', 'in', company_ids)]` | True | True | True | True |

## Views (7)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `l10n_latam_check.view_account_payment_search` | search | `False` | `name`, `partner_id`, `original_journal_id`, `company_id` |  | `Payment Date`, `Handed`, `Voided`, `Debited`, `My Activities`, `Late Activities`, `Today Activities`, `Future Activities`, `Partner`, `Payment Date`, `State`, `Company` | `l10n_latam_check` |
| `l10n_latam_check.view_account_payment_third_party_checks_search` | filter | `view_account_payment_search` |  |  | `checks_on_hand` | `l10n_latam_check` |
| `l10n_latam_check.view_account_check_calendar` | calendar |  | `amount` |  |  | `l10n_latam_check` |
| `l10n_latam_check.view_account_check_pivot` | pivot |  | `payment_date`, `payment_date`, `amount` |  |  | `l10n_latam_check` |
| `l10n_latam_check.l10n_latam_check_view_form` | form |  | `outstanding_line_id`, `issue_state`, `name`, `payment_date`, `original_journal_id`, `current_journal_id`, `amount`, `bank_id`, `issuer_vat`, `currency_id`, `company_id` | `Void Check`, `button_open_check_operations`, `button_open_payment`, `action_show_journal_entry`, `action_show_reconciled_move` |  | `l10n_latam_check` |
| `l10n_latam_check.view_account_own_check_tree` | list |  | `payment_date`, `name`, `original_journal_id`, `company_id`, `payment_method_line_id`, `partner_id`, `amount`, `currency_id`, `issue_state` |  |  | `l10n_latam_check` |
| `l10n_latam_check.view_account_third_party_check_tree` | field | `view_account_own_check_tree` | `issue_state` |  |  | `l10n_latam_check` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `l10n_latam_check.action_own_check` | Own Checks | list,form,calendar,graph,pivot | `[('outstanding_line_id', '!=', False)]` | `{'search_default_checks_on_hand': True}` |  | `l10n_latam_check` |
| `l10n_latam_check.action_third_party_check` | Third Party Checks | list,form,calendar,graph,pivot | `[('payment_method_code', '=', 'new_third_party_checks'), ('payment_id.state', '!=', 'draft')]` | `{'search_default_checks_on_hand': 1}` |  | `l10n_latam_check` |

Machine-readable definition: `../../../schemas/data/entities/l10n_latam.check.json`; views: `../../../schemas/interfaces/views/l10n_latam.check.json`.
