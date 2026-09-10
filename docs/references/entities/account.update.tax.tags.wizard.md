# Update Tax Tags Wizard (`account.update.tax.tags.wizard`)

**Transport name:** `account.update.tax.tags.wizard`  
**Storage name:** `account_update_tax_tags_wizard`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `account_update_tax_tags`

Description: Update Tax Tags Wizard

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `company_id` | Company | many to one | `res.company` | required; read only; default computed dynamically (lambda self: self.env.company) |
| `date_from` | Starting from | date |  | required; computed by rule `_compute_date_from` and stored; Help: Date from which journal items will be updated. |
| `display_lock_date_warning` | Display Lock Date Warning | boolean |  | computed by rule `_compute_display_lock_date_warning` (not stored) |

## Operations (4)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_date_from` | computation | self | `account_update_tax_tags` | depends: `company_id` |  |
| `_compute_display_lock_date_warning` | computation | self | `account_update_tax_tags` | depends: `date_from` |  |
| `_modify_tag_to_aml_relation` | internal rule | self, company_id, date_from | `account_update_tax_tags` |  | Update Journal Items' tax grids to match current taxes' configuration. The next query work in 3 steps. 1) Get a duo: (aml_id, tag_id or NULL) for each aml involved with the tax.     This first step is achieved in the 3 first table: base line, tax line, fusion.     a) As the base line and tax line aren't linked to tax in the same way, we need to gather them separately.     The base line get its repartition line by going through the tax.     b) The tax lines already have their repartition line linked. 2) Delete the previous relation aml - tag for each aml appearing in the previous queries. 3) In |
| `update_amls_tax_tags` | operation | self | `account_update_tax_tags` |  |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `update_amls_tax_tags` | UserError | Update with children taxes that are child of multiple parents is not supported. | `account_update_tax_tags` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_manager` | yes | yes | yes | no | `account_update_tax_tags` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `account_update_tax_tags.view_account_update_tax_tags_wizard_form` | form |  | `company_id`, `date_from` | `Update`, `Discard` |  | `account_update_tax_tags` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `account_update_tax_tags.action_open_wizard` | Update tax tags on existing Journal Entries | form |  |  | new | `account_update_tax_tags` |

Machine-readable definition: `../../../schemas/data/entities/account.update.tax.tags.wizard.json`; views: `../../../schemas/interfaces/views/account.update.tax.tags.wizard.json`.
