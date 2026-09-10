# SInvoice symbol (`l10n_vn_edi_viettel.sinvoice.symbol`)

**Transport name:** `l10n_vn_edi_viettel.sinvoice.symbol`  
**Storage name:** `l10n_vn_edi_viettel_sinvoice_symbol`  
**Kind:** persistent entity (one table)  
**Defined by package:** `l10n_vn_edi_viettel`

Description: SInvoice symbol

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Symbol | single line text |  | required |
| `invoice_template_id` | Invoice Template | many to one | `l10n_vn_edi_viettel.sinvoice.template` | required; indexed |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_name_template_uniq` | Constraint | `unique (name, invoice_template_id)` | The combination symbol/template must be unique! | `l10n_vn_edi_viettel` |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_constrains_changes` | validation | self | `l10n_vn_edi_viettel` | constrains: `name`, `invoice_template_id` | Multiple API endpoints will use these data, we should thus not allow changing them if they have been used for any invoices sent to sinvoice. |
| `_compute_display_name` | computation | self | `l10n_vn_edi_viettel` | depends: `name`, `invoice_template_id` | As we allow multiple of the same symbol name, we need to also display the template to differentiate. |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_constrains_changes` | UserError | You cannot change the symbol value or template of the symbol %s because it has already been used to send invoices. | `l10n_vn_edi_viettel` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_invoice` | no | yes | no | no | `l10n_vn_edi_viettel` |
| `account.group_account_manager` | yes | yes | yes | yes | `l10n_vn_edi_viettel` |
| `point_of_sale.group_pos_user` | no | yes | no | no | `l10n_vn_edi_viettel_pos` |
| `point_of_sale.group_pos_manager` | yes | yes | yes | yes | `l10n_vn_edi_viettel_pos` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `l10n_vn_edi_viettel.action_sinvoice_symbol` | Symbols | list,form |  |  |  | `l10n_vn_edi_viettel` |

Machine-readable definition: `../../../schemas/data/entities/l10n_vn_edi_viettel.sinvoice.symbol.json`.
