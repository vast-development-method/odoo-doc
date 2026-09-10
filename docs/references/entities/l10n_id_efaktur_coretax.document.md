# E-Faktur Document (`l10n_id_efaktur_coretax.document`)

**Transport name:** `l10n_id_efaktur_coretax.document`  
**Storage name:** `l10n_id_efaktur_coretax_document`  
**Kind:** persistent entity (one table)  
**Defined by package:** `l10n_id_efaktur_coretax`

Description: E-Faktur Document

## Identity and behavior

- Mixins (classical inheritance): `mail.thread.main.attachment`, `mail.activity.mixin`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (5)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | computed by rule `_compute_name` and stored |
| `company_id` | Company | many to one | `res.company` | required; read only; default computed dynamically (lambda self: self.env.company) |
| `active` | Active | boolean |  | default `True` |
| `invoice_ids` | Invoice | one to many | `account.move` | restricted by domain `[('move_type', 'in', ['out_invoice', 'out_refund']), ('company_id', '=', company_id), ('l10n_id_coretax_document', '=', False), ('state', '=', 'posted')]`; inverse field `l10n_id_coretax_document` |
| `attachment_id` | Attachment | many to one | `ir.attachment` | read only |

## Operations (5)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_name` | computation | self | `l10n_id_efaktur_coretax` | depends: `invoice_ids` |  |
| `action_download` | user action | self | `l10n_id_efaktur_coretax` |  | Download E-Faktur of related attachment |
| `_generate_xml` | internal rule | self, regenerate | `l10n_id_efaktur_coretax` |  | Generate the XML file as content and save it as attachment in this record |
| `_generate_efaktur_invoice` | internal rule | self | `l10n_id_efaktur_coretax` |  | Generate E-Faktur for customer invoice. Prepare data, load XML template and |
| `action_regenerate` | user action | self | `l10n_id_efaktur_coretax` |  |  |

## Validation and error messages (2)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_generate_xml` | UserError | Some documents don't have a transaction code: %s | `l10n_id_efaktur_coretax` |
| `_generate_xml` | UserError | Some documents are not Customer Invoices: %s | `l10n_id_efaktur_coretax` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_invoice` | yes | yes | yes | yes | `l10n_id_efaktur_coretax` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| E-Faktur document multi-company | global (all users) | `[('company_id', 'in', company_ids)]` | True | True | True | True |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `l10n_id_efaktur_coretax.l10n_id_efaktur_document_form_view` | form |  | `attachment_id`, `name`, `company_id`, `invoice_ids`, `name`, `invoice_date`, `amount_untaxed_in_currency_signed`, `amount_tax_signed`, `amount_total_in_currency_signed`, `currency_id`, `status_in_payment` | `Download`, `Regenerate File` |  | `l10n_id_efaktur_coretax` |
| `l10n_id_efaktur_coretax.l10n_id_efaktur_document_list_view` | list |  | `name`, `invoice_ids` |  |  | `l10n_id_efaktur_coretax` |
| `l10n_id_efaktur_coretax.l10n_id_efaktur_document_filter_view` | search |  |  |  | `Active`, `Inactive` | `l10n_id_efaktur_coretax` |

## Server actions

| Action | Name | State | Binding | Custom logic |
|---|---|---|---|---|
| `l10n_id_efaktur_coretax.download_efaktur` | Download E-Faktur | code |  | yes |

Machine-readable definition: `../../../schemas/data/entities/l10n_id_efaktur_coretax.document.json`; views: `../../../schemas/interfaces/views/l10n_id_efaktur_coretax.document.json`.
