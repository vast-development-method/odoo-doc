# Thumb drive used to sign invoices in Egypt (`l10n_eg_edi.thumb.drive`)

**Transport name:** `l10n_eg_edi.thumb.drive`  
**Storage name:** `l10n_eg_edi_thumb_drive`  
**Kind:** persistent entity (one table)  
**Defined by package:** `l10n_eg_edi_eta`

Description: Thumb drive used to sign invoices in Egypt

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (5)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `user_id` | User | many to one | `res.users` | required; default computed dynamically (lambda self: self.env.user) |
| `company_id` | Company | many to one | `res.company` | required; default computed dynamically (lambda self: self.env.company) |
| `certificate` | ETA Certificate | binary |  |  |
| `pin` | ETA USB Pin | single line text |  | required |
| `access_token` | Access Token | single line text |  | required |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_user_drive_uniq` | Constraint | `unique (user_id, company_id)` | You can only have one thumb drive per user per company! | `l10n_eg_edi_eta` |

## Operations (9)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `action_sign_invoices` | user action | self, invoice_ids | `l10n_eg_edi_eta` |  |  |
| `action_set_certificate_from_usb` | user action | self | `l10n_eg_edi_eta` |  |  |
| `set_certificate` | operation | self, certificate | `l10n_eg_edi_eta` |  | This is called from the browser to set the certificate |
| `set_signature_data` | operation | self, invoices | `l10n_eg_edi_eta` |  | This is called from the browser with the signed data from the local server |
| `_get_host` | preparation rule | self | `l10n_eg_edi_eta` |  |  |
| `_serialize_for_signing` | internal rule | self, eta_inv | `l10n_eg_edi_eta` |  |  |
| `_generate_signed_attrs__` | internal rule | self, eta_invoice, signing_time | `l10n_eg_edi_eta` |  |  |
| `_generate_signer_info__` | internal rule | self, eta_invoice, signing_time, signature | `l10n_eg_edi_eta` |  |  |
| `_generate_cades_bes_signature` | internal rule | self, eta_invoice, signing_time, signature | `l10n_eg_edi_eta` |  |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_get_host` | ValidationError | Please define the host of sign tool. | `l10n_eg_edi_eta` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_manager` | yes | yes | yes | yes | `l10n_eg_edi_eta` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Only see/modify own thumb drive | global (all users) | `[('user_id', '=', user.id)]` | True | True | True | True |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `l10n_eg_edi_eta.view_l10n_eg_edi_thumb_drive_tree` | list |  | `user_id`, `certificate`, `company_id`, `pin`, `access_token` | `Get certificate` |  | `l10n_eg_edi_eta` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `l10n_eg_edi_eta.action_eta_thumb_drive_tree` | Thumb Drive | list |  |  |  | `l10n_eg_edi_eta` |

Machine-readable definition: `../../../schemas/data/entities/l10n_eg_edi.thumb.drive.json`; views: `../../../schemas/interfaces/views/l10n_eg_edi.thumb.drive.json`.
