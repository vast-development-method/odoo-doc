# Checks Mass Transfers (`l10n_latam.payment.mass.transfer`)

**Transport name:** `l10n_latam.payment.mass.transfer`  
**Storage name:** `l10n_latam_payment_mass_transfer`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `l10n_latam_check`

Description: Checks Mass Transfers

## Identity and behavior

- Company consistency is checked automatically on company-bound relations

## Fields (6)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `payment_date` | Payment Date | date |  | required; default computed dynamically (fields.Date.context_today) |
| `destination_journal_id` | Destination Journal | many to one | `account.journal` | restricted by domain `[('type', 'in', ('bank', 'cash')), ('id', '!=', journal_id)]`; must belong to the same company |
| `communication` | Memo | single line text |  |  |
| `journal_id` | Journal | many to one | `account.journal` | computed by rule `_compute_journal_company` (not stored); must belong to the same company |
| `company_id` | Company | many to one | `res.company` | computed by rule `_compute_journal_company` (not stored) |
| `check_ids` | Check | many to many | `l10n_latam.check` | must belong to the same company; association table `latam_tranfer_check_reltransfer_id` |

## Operations (4)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_journal_company` | computation | self | `l10n_latam_check` | depends: `check_ids` |  |
| `default_get` | lifecycle override | self, fields | `l10n_latam_check` | model |  |
| `_create_payments` | internal rule | self | `l10n_latam_check` |  | This is nedeed because we would like to create a payment of type internal transfer for each check with the counterpart journal and then, when posting a second payment will be created automatically |
| `action_create_payments` | user action | self | `l10n_latam_check` |  |  |

## Validation and error messages (5)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_compute_journal_company` | UserError | All selected checks must be on the same journal and on hand | `l10n_latam_check` |
| `default_get` | UserError | The register payment wizard should only be called on account.payment records. | `l10n_latam_check` |
| `default_get` | UserError | You have selected payments which are not checks. Please call this action from the Third Party Checks menu | `l10n_latam_check` |
| `default_get` | UserError | All the selected checks must use the same currency | `l10n_latam_check` |
| `default_get` | UserError | All the selected checks must be posted | `l10n_latam_check` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_invoice` | yes | yes | yes | no | `l10n_latam_check` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `l10n_latam_check.view_l10n_latam_payment_mass_transfer_form` | form |  | `check_ids`, `journal_id`, `company_id`, `destination_journal_id`, `payment_date`, `communication` | `Create Transfers`, `Cancel` |  | `l10n_latam_check` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `l10n_latam_check.action_view_l10n_latam_payment_mass_transfer` | Check Transfer | form |  |  | new | `l10n_latam_check` |

Machine-readable definition: `../../../schemas/data/entities/l10n_latam.payment.mass.transfer.json`; views: `../../../schemas/interfaces/views/l10n_latam.payment.mass.transfer.json`.
