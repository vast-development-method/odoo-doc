# SInvoice template (`l10n_vn_edi_viettel.sinvoice.template`)

**Transport name:** `l10n_vn_edi_viettel.sinvoice.template`  
**Storage name:** `l10n_vn_edi_viettel_sinvoice_template`  
**Kind:** persistent entity (one table)  
**Defined by package:** `l10n_vn_edi_viettel`

Description: SInvoice template

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Template Code | single line text |  | required |
| `template_invoice_type` | Template Invoice Type | selection |  | required |
| `invoice_symbols_ids` | Invoice Symbols | one to many | `l10n_vn_edi_viettel.sinvoice.symbol` | inverse field `invoice_template_id` |

## Selection values

### `template_invoice_type` (Template Invoice Type)

| Value | Label |
|---|---|
| `1` | 1 - Value-added invoice |
| `2` | 2 - Sales invoice |
| `3` | 3 - Public assets sales |
| `4` | 4 - National reserve sales |
| `5` | 5 - Invoice for national reserve sales |
| `6` | 6 - Warehouse release note |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_name_uniq` | Constraint | `unique (name)` | The template code must be unique! | `l10n_vn_edi_viettel` |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_constrains_changes` | validation | self | `l10n_vn_edi_viettel` | constrains: `name`, `template_invoice_type` | Multiple API endpoints will use these data, we should thus not allow changing them if they have been used for any invoices sent to sinvoice. |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_invoice` | no | yes | no | no | `l10n_vn_edi_viettel` |
| `account.group_account_manager` | yes | yes | yes | yes | `l10n_vn_edi_viettel` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `l10n_vn_edi_viettel.action_sinvoice_template` | Templates | list,form |  |  |  | `l10n_vn_edi_viettel` |

Machine-readable definition: `../../../schemas/data/entities/l10n_vn_edi_viettel.sinvoice.template.json`.
