# Account Move Send Wizard (`account.move.send.wizard`)

**Transport name:** `account.move.send.wizard`  
**Storage name:** `account_move_send_wizard`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `account`  
**Extended by packages:** `account_edi_ubl_cii`, `account_peppol`, `l10n_dk_nemhandel`, `l10n_es_edi_facturae`, `l10n_fr_pdp`, `l10n_ke_edi_tremol`, `l10n_ro_edi`, `l10n_rs_edi`

Description: Account Move Send Wizard

## Identity and behavior

- Mixins (classical inheritance): `account.move.send`, `mail.composer.mixin`

## Fields (20)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `move_id` | Move | many to one | `account.move` | required |
| `company_id` | Company | many to one | `res.company` | related through path `move_id.company_id` |
| `alerts` | Alerts | structured document |  | computed by rule `_compute_alerts` (not stored) |
| `sending_methods` | Sending Methods | structured document |  | computed by rule `_compute_sending_methods` (not stored); writable through an inverse rule |
| `sending_method_checkboxes` | Sending Method Checkboxes | structured document |  | computed by rule `_compute_sending_method_checkboxes` and stored; precomputed before insertion |
| `display_attachments_widget` | Display Attachments Widget | boolean |  | computed by rule `_compute_display_attachments_widget` (not stored) |
| `extra_edis` | Extra Edis | structured document |  | computed by rule `_compute_extra_edis` (not stored); writable through an inverse rule |
| `extra_edi_checkboxes` | Extra Electronic data interchange Checkboxes | structured document |  | computed by rule `_compute_extra_edi_checkboxes` and stored; precomputed before insertion |
| `invoice_edi_format` | Invoice Electronic data interchange Format | selection |  | computed by rule `_compute_invoice_edi_format` (not stored) |
| `pdf_report_id` | Invoice report | many to one | `ir.actions.report` | computed by rule `_compute_pdf_report_id` and stored; restricted by domain `[('id', 'in', available_pdf_report_ids)]` |
| `available_pdf_report_ids` | Available Portable Document Format Report | one to many | `ir.actions.report` | computed by rule `_compute_available_pdf_report_ids` (not stored) |
| `display_pdf_report_id` | Display Portable Document Format Report | boolean |  | computed by rule `_compute_display_pdf_report_id` (not stored) |
| `template_id` | Template | many to one |  | computed by rule `_compute_template_id` and stored; restricted by domain `[('model', '=', 'account.move')]` |
| `lang` | Lang | single line text |  | computed by rule `_compute_lang` (not stored) |
| `mail_partner_ids` | To | many to many | `res.partner` | computed by rule `_compute_mail_partners` and stored |
| `mail_attachments_widget` | Mail Attachments Widget | structured document |  | computed by rule `_compute_mail_attachments_widget` and stored |
| `attachments_not_supported` | Attachments Not Supported | structured document |  | computed by rule `_compute_attachments_not_supported` (not stored) |
| `model` | Related Document Model | single line text |  | computed by rule `_compute_model` and stored |
| `res_ids` | Related Document identifiers | multi line text |  | computed by rule `_compute_res_ids` and stored |
| `template_name` | Template Name | single line text |  |  |

## Operations (36)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `default_get` | lifecycle override | self, fields | `account` | model |  |
| `_compute_alerts` | computation | self | `account` | depends: `sending_methods`, `extra_edis`, `mail_partner_ids` |  |
| `_compute_sending_methods` | computation | self | `account` | depends: `sending_method_checkboxes` |  |
| `_inverse_sending_methods` | inverse computation | self | `account` |  |  |
| `_compute_sending_method_checkboxes` | computation | self | `account_peppol`, `account`, `l10n_dk_nemhandel` | depends: `move_id` | Select one applicable sending method given the following priority 1. preferred method set on partner, 2. email, |
| `_compute_display_attachments_widget` | computation | self | `account` | depends: `invoice_edi_format` |  |
| `_compute_extra_edis` | computation | self | `account` | depends: `extra_edi_checkboxes` |  |
| `_inverse_extra_edis` | inverse computation | self | `account` |  |  |
| `_compute_extra_edi_checkboxes` | computation | self | `account`, `l10n_es_edi_facturae`, `l10n_ro_edi` | depends: `move_id` |  |
| `_compute_invoice_edi_format` | computation | self | `account` | depends: `move_id`, `sending_methods` |  |
| `_compute_pdf_report_id` | computation | self | `account` | depends: `move_id` |  |
| `_compute_available_pdf_report_ids` | computation | self | `account` | depends: `move_id` |  |
| `_compute_display_pdf_report_id` | computation | self | `account` | depends: `move_id` | Show PDF template selection if there are more than 1 template available for invoices. |
| `_compute_template_id` | computation | self | `account` | depends: `move_id` |  |
| `_compute_lang` | computation | self | `account` | depends: `template_id` |  |
| `_compute_mail_partners` | computation | self | `account` | depends: `template_id`, `lang` |  |
| `_compute_subject` | computation | self | `account` | depends: `template_id`, `lang` |  |
| `_compute_body` | computation | self | `account` | depends: `template_id`, `lang` |  |
| `_compute_mail_attachments_widget` | computation | self | `account` | depends: `template_id`, `invoice_edi_format`, `extra_edis`, `pdf_report_id` |  |
| `_compute_res_ids` | computation | self | `account` | depends: `template_id` |  |
| `_compute_model` | computation | self | `account` | depends: `template_id` |  |
| `_compute_can_edit_body` | computation | self | `account` | depends: `sending_methods` |  |
| `_compute_render_model` | computation | self | `account` | depends: `model` |  |
| `open_template_creation_wizard` | operation | self | `account` |  | Hit save as template button: opens a wizard that prompts for the template's subject. `create_mail_template` is called when saving the new wizard. |
| `create_mail_template` | operation | self | `account` |  | Creates a mail template with the current mail composer's fields. |
| `cancel_save_template` | operation | self | `account` |  | Restore old subject when canceling the 'save as template' action as it was erased to let user give a more custom input. |
| `_compute_attachments_not_supported` | computation | self | `account_edi_ubl_cii`, `account` | depends: `invoice_edi_format`, `mail_attachments_widget` |  |
| `_check_move_id_constraints` | validation | self | `account` | constrains: `move_id` |  |
| `_get_selected_checkboxes` | preparation rule | self, json_checkboxes | `account` | model |  |
| `_get_sending_settings` | preparation rule | self | `account` |  |  |
| `_update_preferred_settings` | internal rule | self | `account` |  | If the partner's settings are not set, we use them as partner's default. |
| `_action_download` | internal rule | self, attachments | `account` | model | Download the PDF attachment, or a zip of attachments if there are more than one. |
| `action_send_and_print` | user action | self, allow_fallback_pdf | `account_peppol`, `account`, `l10n_dk_nemhandel`, `l10n_es_edi_facturae`, `l10n_ke_edi_tremol` |  | Create invoice documents and send them. |
| `_get_peppol_checkbox_label` | preparation rule | self, default_label | `account_peppol`, `l10n_fr_pdp` |  |  |
| `_get_peppol_checkbox_addendum_disable_reason` | preparation rule | self | `account_peppol`, `l10n_fr_pdp` |  |  |
| `_onchange_extra_edi_checkboxes` | on change | self | `l10n_rs_edi` | onchange: `extra_edi_checkboxes` |  |

## Validation and error messages (4)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `create_mail_template` | UserError | Template creation from composer requires a valid model. | `account` |
| `action_send_and_print` | UserError | Partner doesn't have a valid Peppol configuration. | `account_peppol` |
| `action_send_and_print` | UserError | Partner doesn't have a valid Nemhandel configuration. | `l10n_dk_nemhandel` |
| `action_send_and_print` | UserError | self._get_l10n_ke_edi_tremol_warning_message(warning_moves) | `l10n_ke_edi_tremol` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_invoice` | yes | yes | yes | yes | `account` |
| `sales_team.group_sale_salesman` | yes | yes | yes | no | `sale` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Readonly Invoice Send and Print (single) | `[(4, ref('account.group_account_invoice'))]` | `[(1, '=', 1)]` | True | True | True | True |
| Personal Invoice Send and Print (single mode) | `[(4, ref('sales_team.group_sale_salesman'))]` | `[('move_id.move_type', 'in', ('out_invoice', 'out_refund')), '\|', ('move_id.invoice_user_id', '=', user.id), ('move_id.invoice_user_id', '=', False)]` | True | True | True | True |
| All Invoice Send and Print (single mode) | `[(4, ref('sales_team.group_sale_salesman_all_leads'))]` | `[('move_id.move_type', 'in', ('out_invoice', 'out_refund'))]` | True | True | True | True |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `account.account_move_send_wizard_form` | form |  | `move_id`, `attachments_not_supported`, `alerts`, `extra_edi_checkboxes`, `sending_method_checkboxes`, `pdf_report_id`, `pdf_report_id`, `model`, `res_ids`, `render_model`, `pdf_report_id`, `mail_partner_ids`, `subject`, `body`, `mail_attachments_widget`, `mail_attachments_widget`, `template_id` | `Send`, `Generate`, `Discard` |  | `account` |

Machine-readable definition: `../../../schemas/data/entities/account.move.send.wizard.json`; views: `../../../schemas/interfaces/views/account.move.send.wizard.json`.
