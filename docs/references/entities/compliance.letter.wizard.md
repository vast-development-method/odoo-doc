# Compliance Letter for EXO Number (`compliance.letter.wizard`)

**Transport name:** `compliance.letter.wizard`  
**Storage name:** `compliance_letter_wizard`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `l10n_mt_pos`

Description: Compliance Letter for EXO Number

## Fields (1)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `company_id` | Company | many to one | `res.company` | required; default computed dynamically (lambda self: self.env.company) |

## Operations (3)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `generate_letter` | operation | self | `l10n_mt_pos` |  |  |
| `_get_formatted_date` | preparation rule | self | `l10n_mt_pos` |  | Returns the formatted date as 'Date (Month, xxth, 20XX)'. |
| `_get_odoo_version` | preparation rule | self | `l10n_mt_pos` |  |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `generate_letter` | UserError | Compliance letters can only be created for companies registered in Malta. Please ensure the company's country is set to Malta. | `l10n_mt_pos` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | yes | yes | yes | yes | `l10n_mt_pos` |
| `base.group_system` | yes | yes | yes | yes | `l10n_mt_pos` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `l10n_mt_pos.view_generate_compliance_letter` | form |  | `company_id` | `Print`, `Cancel` |  | `l10n_mt_pos` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `l10n_mt_pos.action_generate_compliance_letter` | Compliance Letter | form |  |  | new | `l10n_mt_pos` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `l10n_mt_pos.menu_compliance_letter` | Compliance Letter | `l10n_mt_pos.pos_mt_statements_menu` | `l10n_mt_pos.action_generate_compliance_letter` | 10 |  |

Machine-readable definition: `../../../schemas/data/entities/compliance.letter.wizard.json`; views: `../../../schemas/interfaces/views/compliance.letter.wizard.json`.
