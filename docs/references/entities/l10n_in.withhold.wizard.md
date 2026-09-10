# Withhold Wizard (`l10n_in.withhold.wizard`)

**Transport name:** `l10n_in.withhold.wizard`  
**Storage name:** `l10n_in_withhold_wizard`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `l10n_in`

Description: Withhold Wizard

## Identity and behavior

- Company consistency is checked automatically on company-bound relations

## Fields (14)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `reference` | Reference | single line text |  |  |
| `type_name` | Type | single line text |  | computed by rule `_compute_type_name` (not stored) |
| `related_move_id` | Invoice/Bill | many to one | `account.move` | read only |
| `related_payment_id` | Payment | many to one | `account.payment` | read only |
| `tds_deduction` | tax deducted at source Deduction | selection |  | computed by rule `_compute_tds_deduction` (not stored) |
| `company_id` | Company | many to one | `res.company` | computed by rule `_compute_company_id` (not stored) |
| `currency_id` | Currency | many to one |  | related through path `company_id.currency_id` |
| `journal_id` | Journal | many to one | `account.journal` | required; computed by rule `_compute_journal` and stored; must belong to the same company; precomputed before insertion |
| `date` | Date | date |  | default computed dynamically (fields.Date.context_today) |
| `l10n_in_tds_tax_type` | Indian Tax Type | single line text |  | computed by rule `_compute_l10n_in_tds_tax_type` (not stored) |
| `l10n_in_withholding_warning` | Withholding warning | structured document |  | computed by rule `_compute_l10n_in_withholding_warning` (not stored) |
| `base` | Base Amount | monetary |  | computed by rule `_compute_base` and stored |
| `tax_id` | tax deducted at source Section | many to one | `account.tax` | required; computed by rule `_compute_tax_id` and stored |
| `amount` | tax deducted at source Amount | monetary |  | computed by rule `_compute_amount` (not stored) |

## Selection values

### `tds_deduction` (tax deducted at source Deduction)

| Value | Label |
|---|---|
| `normal` | Normal Deduction |
| `lower` | Lower Deduction |
| `higher` | Higher Deduction |
| `no` | No Deduction |

## Operations (16)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `default_get` | lifecycle override | self, fields | `l10n_in` | model |  |
| `_check_amounts` | validation | self | `l10n_in` | constrains: `base` |  |
| `_compute_l10n_in_tds_tax_type` | computation | self | `l10n_in` | depends: `related_move_id`, `related_payment_id` |  |
| `_compute_tds_deduction` | computation | self | `l10n_in` | depends: `l10n_in_tds_tax_type`, `related_move_id`, `related_payment_id` |  |
| `_compute_type_name` | computation | self | `l10n_in` | depends: `related_move_id`, `related_payment_id` |  |
| `_compute_company_id` | computation | self | `l10n_in` | depends: `related_move_id`, `related_payment_id` |  |
| `_compute_journal` | computation | self | `l10n_in` | depends: `company_id` |  |
| `_compute_l10n_in_withholding_warning` | computation | self | `l10n_in` | depends: `related_move_id`, `base` |  |
| `_compute_tax_id` | computation | self | `l10n_in` | depends: `related_move_id`, `related_payment_id` |  |
| `_compute_base` | computation | self | `l10n_in` | depends: `tax_id` |  |
| `_compute_amount` | computation | self | `l10n_in` | depends: `tax_id`, `base` |  |
| `_get_withhold_type` | preparation rule | self | `l10n_in` |  |  |
| `action_create_and_post_withhold` | user action | self | `l10n_in` |  |  |
| `_prepare_withhold_header` | preparation rule | self | `l10n_in` |  | Prepare the header for the withhold entry |
| `_prepare_withhold_move_lines` | preparation rule | self, withholding_account_id | `l10n_in` |  | Prepare the move lines for the withhold entry |
| `_validate_withhold_data_on_post` | internal rule | self, withholding_account_id | `l10n_in` |  |  |

## Validation and error messages (7)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `default_get` | UserError | TDS must be created from an Invoice or a Payment. | `l10n_in` |
| `default_get` | UserError | You can only create a withhold for only one record at a time. | `l10n_in` |
| `default_get` | UserError | TDS must be created from Posted Customer Invoices, Customer Credit Notes, Vendor Bills or Vendor Refunds. | `l10n_in` |
| `default_get` | UserError | Please set a partner on the %s before creating a withhold. | `l10n_in` |
| `_check_amounts` | ValidationError | Negative or zero values are not allowed in Base Amount for withhold | `l10n_in` |
| `_check_amounts` | ValidationError | Negative or zero values are not allowed in TDS Amount for withhold | `l10n_in` |
| `_validate_withhold_data_on_post` | UserError | Please configure the withholding account from the settings | `l10n_in` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_invoice` | yes | yes | yes | yes | `l10n_in` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `l10n_in.tds_entry_view_form` | form |  | `l10n_in_withholding_warning`, `date`, `tax_id`, `tds_deduction`, `base`, `amount`, `currency_id`, `related_move_id`, `related_payment_id`, `journal_id`, `reference` | `Apply TDS`, `Discard` |  | `l10n_in` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `l10n_in.l10n_in_withholding_entry_form_action` | Create TDS Entry | form |  |  | new | `l10n_in` |

Machine-readable definition: `../../../schemas/data/entities/l10n_in.withhold.wizard.json`; views: `../../../schemas/interfaces/views/l10n_in.withhold.wizard.json`.
