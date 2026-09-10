# Request ZATCA one-time password (`l10n_sa_edi.otp.wizard`)

**Transport name:** `l10n_sa_edi.otp.wizard`  
**Storage name:** `l10n_sa_edi_otp_wizard`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `l10n_sa_edi`

Description: Request ZATCA OTP

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `l10n_sa_renewal` | PCSID Renewal | boolean |  | default ; Help: Used to decide whether we should call the PCSID renewal API or the CCSID API |
| `l10n_sa_otp` | one-time password | single line text |  | not copied on duplication; Help: OTP required to get a CCSID. Can only be acquired through the Fatoora portal. |
| `journal_id` | Journal | many to one | `account.journal` | required; default computed dynamically (lambda self: self.env.context.get('active_id')) |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `default_get` | lifecycle override | self, fields | `l10n_sa_edi` | model |  |
| `validate` | operation | self | `l10n_sa_edi` |  |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `validate` | UserError | Please provide an OTP to complete the onboarding process | `l10n_sa_edi` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_invoice` | yes | yes | yes | no | `l10n_sa_edi` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `l10n_sa_edi.l10n_sa_edi_otp_wizard_view_form` | form |  | `journal_id`, `l10n_sa_renewal`, `l10n_sa_otp` | `Confirm`, `Cancel` |  | `l10n_sa_edi` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `l10n_sa_edi.l10n_sa_edi_otp_wizard_act_window` | Enter the OTP | form |  |  | new | `l10n_sa_edi` |

Machine-readable definition: `../../../schemas/data/entities/l10n_sa_edi.otp.wizard.json`; views: `../../../schemas/interfaces/views/l10n_sa_edi.otp.wizard.json`.
