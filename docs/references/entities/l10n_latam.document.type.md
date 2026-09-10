# Latam Document Type (`l10n_latam.document.type`)

**Transport name:** `l10n_latam.document.type`  
**Storage name:** `l10n_latam_document_type`  
**Kind:** persistent entity (one table)  
**Defined by package:** `l10n_latam_invoice_document`  
**Extended by packages:** `l10n_ar`, `l10n_cl`, `l10n_ec`, `l10n_uy`

Description: Latam Document Type

## Identity and behavior

- Default ordering: `sequence, id`
- Display name search fields: `["name", "code"]`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (12)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `active` | Active | boolean |  | default `True` |
| `sequence` | Sequence | integer |  | required; default `10`; Help: To set in which order show the documents type taking into account the most commonly used first |
| `country_id` | Country | many to one | `res.country` | required; indexed; Help: Country in which this type of document is valid |
| `name` | Name | single line text |  | required; translatable; Help: The document name |
| `doc_code_prefix` | Document Code Prefix | single line text |  | Help: Prefix for Documents Codes on Invoices and Account Moves. For eg. 'FA ' will build 'FA 0001-0000001' Document Number |
| `code` | Code | single line text |  | Help: Code used by different localizations |
| `report_name` | Name on Reports | single line text |  | translatable; Help: Name that will be printed in reports, for example "CREDIT NOTE" |
| `internal_type` | Internal Type | selection |  | Help: Analog to odoo account.move.move_type but with more options allowing to identify the kind of document we are working with. (not only related to account.move, could be for documents of other models like stock.picking); extended by packages `l10n_cl`, `l10n_ec` |
| `l10n_ar_letter` | Letters | selection |  | values provided by rule `_get_l10n_ar_letters`; Help: Letters defined by the ARCA that can be used to identify the documents presented to the government and that depends on the operation type, the responsibility of both the issuer and the receptor of the document |
| `purchase_aliquots` | Purchase Aliquots | selection |  | Help: Raise an error if a vendor bill is miss encoded. "Not Zero" means the VAT taxes are required for the invoices related to this document type, and those with "Zero" means that only "VAT Not Applicable" tax is allowed. |
| `l10n_cl_active` | Active in localization | boolean |  | Help: This boolean enables document to be included on invoicing |
| `l10n_ec_check_format` | Check Number Format EC | boolean |  | default  |

## Selection values

### `internal_type` (Internal Type)

| Value | Label |
|---|---|
| `invoice` | Invoices |
| `debit_note` | Debit Notes |
| `credit_note` | Credit Notes |
| `all` | All Documents |
| `invoice_in` | Purchase Invoices |
| `receipt_invoice` | Receipt Invoice |
| `stock_picking` | Stock Delivery |
| `purchase_liquidation` | Purchase Liquidation |
| `withhold` | Withhold |

### `purchase_aliquots` (Purchase Aliquots)

| Value | Label |
|---|---|
| `not_zero` | Not Zero |
| `zero` | Zero |

## Operations (6)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_format_document_number` | internal rule | self, document_number | `l10n_ar`, `l10n_cl`, `l10n_ec`, `l10n_latam_invoice_document`, `l10n_uy` |  | Method to be inherited by different localizations. The purpose of this method is to allow: * making validations on the document_number. If it is wrong it should raise an exception * format the document_number against a pattern and return it |
| `_compute_display_name` | computation | self | `l10n_latam_invoice_document` | depends: `code` |  |
| `_get_l10n_ar_letters` | preparation rule | self | `l10n_ar` |  | Return the list of values of the selection field. |
| `_is_doc_type_vendor` | internal rule | self | `l10n_cl` |  |  |
| `_is_doc_type_export` | internal rule | self | `l10n_cl` |  |  |
| `_is_doc_type_electronic_ticket` | internal rule | self | `l10n_cl` |  |  |

## Validation and error messages (4)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_format_document_number` | UserError | %(value)s is not a valid value for %(field)s. The document number must be entered with a dash (-) and a maximum of 5 characters for the first part and 8 for the second. The following are examples of valid numbers: * 1-1 * 0001-00000001 * 00001-00000001 | `l10n_ar` |
| `_format_document_number` | UserError | %(value)s is not a valid value for %(field)s. The number of import Dispatch must be 16 characters. | `l10n_ar` |
| `_format_document_number` | UserError | Ecuadorian Document %s must be like 001-001-123456789 | `l10n_ec` |
| `_format_document_number` | UserError | %(document_number)s is not a valid value for %(document_type)s. The document number must be entered with a maximum of 2 letters for the first part and 7 numbers for the second. The following are examples of valid document numbers: - XX0000001  - YY0000123  - A0000001 | `l10n_uy` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `l10n_latam_invoice_document` |
| `account.group_account_manager` | yes | yes | yes | no | `l10n_latam_invoice_document` |

## Views (8)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `l10n_ar.view_document_type_form` | field | `l10n_latam_invoice_document.view_document_type_form` | `doc_code_prefix`, `l10n_ar_letter`, `purchase_aliquots` |  |  | `l10n_ar` |
| `l10n_ar.view_document_type_tree` | field | `l10n_latam_invoice_document.view_document_type_tree` | `doc_code_prefix`, `l10n_ar_letter` |  |  | `l10n_ar` |
| `l10n_ar.view_document_type_filter` | field | `l10n_latam_invoice_document.view_document_type_filter` | `code`, `l10n_ar_letter` |  | `Argentinean Documents` | `l10n_ar` |
| `l10n_ec.view_document_type_conf_form` | xpath | `l10n_latam_invoice_document.view_document_type_form` | `l10n_ec_check_format` |  |  | `l10n_ec` |
| `l10n_ec.view_document_type_conf_tree` | xpath | `l10n_latam_invoice_document.view_document_type_tree` | `l10n_ec_check_format` |  |  | `l10n_ec` |
| `l10n_latam_invoice_document.view_document_type_form` | form |  | `code`, `name`, `doc_code_prefix`, `report_name`, `internal_type`, `country_id` |  |  | `l10n_latam_invoice_document` |
| `l10n_latam_invoice_document.view_document_type_tree` | list |  | `code`, `name`, `doc_code_prefix`, `report_name`, `internal_type`, `country_id`, `active` |  |  | `l10n_latam_invoice_document` |
| `l10n_latam_invoice_document.view_document_type_filter` | search |  | `name`, `code`, `internal_type`, `country_id` |  | `Active`, `Archived`, `Internal Type`, `Localization` | `l10n_latam_invoice_document` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `l10n_ar.action_document_type_argentina` | Document Types |  |  | `{'search_default_localization': 1}` |  | `l10n_ar` |
| `l10n_latam_invoice_document.action_document_type` | Document Types |  | `['\|', ('active', '=', True), ('active', '=', False)]` | `{"search_default_active":1}` |  | `l10n_latam_invoice_document` |

Machine-readable definition: `../../../schemas/data/entities/l10n_latam.document.type.json`; views: `../../../schemas/interfaces/views/l10n_latam.document.type.json`.
