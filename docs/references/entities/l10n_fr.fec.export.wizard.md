# Fichier Echange Informatise (`l10n_fr.fec.export.wizard`)

**Transport name:** `l10n_fr.fec.export.wizard`  
**Storage name:** `l10n_fr_fec_export_wizard`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `l10n_fr_account`

Description: Fichier Echange Informatise

## Fields (6)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `date_from` | Start Date | date |  | required; default computed dynamically (lambda self: self.env.context.get('report_dates', {}).get('date_from')) |
| `date_to` | End Date | date |  | required; default computed dynamically (lambda self: self.env.context.get('report_dates', {}).get('date_to')) |
| `filename` | Filename | single line text |  | read only; maximum length 256 |
| `test_file` | Test File | boolean |  |  |
| `export_type` | Export Type | selection |  | required; default `official` |
| `excluded_journal_ids` | Excluded Journals | many to many | `account.journal` | restricted by domain `[('company_id', 'parent_of', current_company_id)]` |

## Selection values

### `export_type` (Export Type)

| Value | Label |
|---|---|
| `official` | Official FEC report (posted entries only) |
| `nonofficial` | Non-official FEC report (posted and unposted entries) |

## Operations (7)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_onchange_export_file` | on change | self | `l10n_fr_account` | onchange: `test_file` |  |
| `_get_base_domain` | preparation rule | self | `l10n_fr_account` |  |  |
| `_do_query_unaffected_earnings` | internal rule | self | `l10n_fr_account` |  | Compute the sum of ending balances for all accounts that are of a type that does not bring forward the balance in new fiscal years. This is needed because we have to display only one line for the initial balance of all expense/revenue accounts in the FEC. |
| `_get_company_legal_data` | preparation rule | self, company | `l10n_fr_account` |  | Dom-Tom are excluded from the EU's fiscal territory Those regions do not have SIREN sources:     https://www.service-public.fr/professionnels-entreprises/vosdroits/F23570     http://www.douane.gouv.fr/articles/a11024-tva-dans-les-dom  * Returns the siren if the company is french or an empty siren for dom-tom * For non-french companies -> returns the complete vat number |
| `_get_fec_stream` | preparation rule | self | `l10n_fr_account` |  |  |
| `generate_fec` | operation | self | `l10n_fr_account` |  |  |
| `create_fec_report_action` | operation | self | `l10n_fr_account` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_user` | yes | yes | yes | no | `l10n_fr_account` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `l10n_fr_account.fec_export_wizard_view` | form |  | `date_from`, `date_to`, `test_file`, `export_type`, `excluded_journal_ids` | `Generate`, `Cancel` |  | `l10n_fr_account` |

Machine-readable definition: `../../../schemas/data/entities/l10n_fr.fec.export.wizard.json`; views: `../../../schemas/interfaces/views/l10n_fr.fec.export.wizard.json`.
