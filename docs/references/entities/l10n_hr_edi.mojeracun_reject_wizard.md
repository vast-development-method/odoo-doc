# MojEracun Reject Invoice Wizard (`l10n_hr_edi.mojeracun_reject_wizard`)

**Transport name:** `l10n_hr_edi.mojeracun_reject_wizard`  
**Storage name:** `l10n_hr_edi_mojeracun_reject_wizard`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `l10n_hr_edi`

Description: MojEracun Reject Invoice Wizard

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `move_id` | Move | many to one | `account.move` | required |
| `rejection_type` | Rejection reason type | selection |  | required |
| `rejection_description` | Rejection reason description | single line text |  | required |

## Selection values

### `rejection_type` (Rejection reason type)

| Value | Label |
|---|---|
| `N` | 'N' - Data discrepancy that does not affect tax calculation |
| `U` | 'U' - Data discrepancy that affects tax calculation |
| `O` | 'O' - Other |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `default_get` | lifecycle override | self, fields_list | `l10n_hr_edi` | model |  |
| `button_reject_invoice` | user action | self | `l10n_hr_edi` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_invoice` | yes | yes | yes | yes | `l10n_hr_edi` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `l10n_hr_edi.mojeracun_reject_wizard_form` | form |  | `rejection_type`, `rejection_description` | `Reject invoice` |  | `l10n_hr_edi` |

Machine-readable definition: `../../../schemas/data/entities/l10n_hr_edi.mojeracun_reject_wizard.json`; views: `../../../schemas/interfaces/views/l10n_hr_edi.mojeracun_reject_wizard.json`.
