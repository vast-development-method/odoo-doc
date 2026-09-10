# Nemhandel Rejection wizard (`nemhandel.rejection.wizard`)

**Transport name:** `nemhandel.rejection.wizard`  
**Storage name:** `nemhandel_rejection_wizard`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `l10n_dk_nemhandel_response`

Description: Nemhandel Rejection wizard

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `move_ids` | Move | many to many | `account.move` | required |
| `note` | Additional note | multi line text |  |  |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `button_send` | user action | self | `l10n_dk_nemhandel_response` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_invoice` | yes | yes | yes | yes | `l10n_dk_nemhandel_response` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `l10n_dk_nemhandel_response.nemhandel_rejection_wizard_view_form` | form |  | `note` | `Send Rejection` |  | `l10n_dk_nemhandel_response` |

Machine-readable definition: `../../../schemas/data/entities/nemhandel.rejection.wizard.json`; views: `../../../schemas/interfaces/views/nemhandel.rejection.wizard.json`.
