# Tax audit export - Adóhatósági Ellenőrzési Adatszolgáltatás (`l10n_hu_edi.tax_audit_export`)

**Transport name:** `l10n_hu_edi.tax_audit_export`  
**Storage name:** `l10n_hu_edi_tax_audit_export`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `l10n_hu_edi`

Description: Tax audit export - Adóhatósági Ellenőrzési Adatszolgáltatás

## Fields (7)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `selection_mode` | Selection mode | selection |  | default `date` |
| `date_from` | Date From | date |  |  |
| `date_to` | Date To | date |  |  |
| `name_from` | Name From | single line text |  |  |
| `name_to` | Name To | single line text |  |  |
| `filename` | File name | single line text |  | computed by rule `_compute_filename` (not stored) |
| `export_file` | Generated File | binary |  | read only |

## Selection values

### `selection_mode` (Selection mode)

| Value | Label |
|---|---|
| `date` | By date |
| `name` | By serial number |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_filename` | computation | self | `l10n_hu_edi` | depends: `selection_mode`, `date_from`, `date_to`, `name_from`, `name_to` |  |
| `action_export` | user action | self | `l10n_hu_edi` |  |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `action_export` | UserError | No invoice to export! | `l10n_hu_edi` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_manager` | yes | yes | yes | yes | `l10n_hu_edi` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `l10n_hu_edi.l10n_hu_edi_tax_audit_export_form` | form |  | `selection_mode`, `date_from`, `name_from`, `date_to`, `name_to`, `export_file`, `filename` | `Export`, `Close` |  | `l10n_hu_edi` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `l10n_hu_edi.action_l10n_hu_edi_tax_audit_export_form` | Tax audit export - Adóhatósági Ellenőrzési Adatszolgáltatás | form |  |  | new | `l10n_hu_edi` |

Machine-readable definition: `../../../schemas/data/entities/l10n_hu_edi.tax_audit_export.json`; views: `../../../schemas/interfaces/views/l10n_hu_edi.tax_audit_export.json`.
