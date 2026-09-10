# Turkish Tax Codes (GIB Codes) (`l10n_tr_nilvera_einvoice_extended.account.tax.code`)

**Transport name:** `l10n_tr_nilvera_einvoice_extended.account.tax.code`  
**Storage name:** `l10n_tr_nilvera_einvoice_extended_account_tax_code`  
**Kind:** persistent entity (one table)  
**Defined by package:** `l10n_tr_nilvera_einvoice_extended`

Description: Turkish Tax Codes (GIB Codes)

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (4)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Reason | single line text |  | required; translatable |
| `code` | Reason Code | integer |  | required |
| `percentage` | Percentage | float |  |  |
| `code_type` | Code Type | selection |  | required |

## Selection values

### `code_type` (Code Type)

| Value | Label |
|---|---|
| `withholding` | Withholding |
| `exception` | Exception |
| `export_exception` | Export Exception |
| `export_registration` | Export Registration |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_display_name` | computation | self | `l10n_tr_nilvera_einvoice_extended` | depends: `name`, `percentage` |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_readonly` | no | yes | no | no | `l10n_tr_nilvera_einvoice_extended` |
| `account.group_account_basic` | yes | yes | yes | yes | `l10n_tr_nilvera_einvoice_extended` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `l10n_tr_nilvera_einvoice_extended.l10n_tr_nilvera_einvoice_extended_account_tax_code_view_list` | list |  | `name`, `code`, `code_type`, `percentage` |  |  | `l10n_tr_nilvera_einvoice_extended` |
| `l10n_tr_nilvera_einvoice_extended.l10n_tr_nilvera_einvoice_extended_account_tax_code_view_form` | form |  | `name`, `code`, `code_type`, `percentage` |  |  | `l10n_tr_nilvera_einvoice_extended` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `l10n_tr_nilvera_einvoice_extended.action_l10n_tr_nilvera_einvoice_extended_account_tax_code_list` | GIB Codes |  |  |  |  | `l10n_tr_nilvera_einvoice_extended` |

Machine-readable definition: `../../../schemas/data/entities/l10n_tr_nilvera_einvoice_extended.account.tax.code.json`; views: `../../../schemas/interfaces/views/l10n_tr_nilvera_einvoice_extended.account.tax.code.json`.
