# Bank setup manual config (`account.setup.bank.manual.config`)

**Transport name:** `account.setup.bank.manual.config`  
**Storage name:** `account_setup_bank_manual_config`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `account`  
**Extended by packages:** `l10n_ch`

Description: Bank setup manual config

## Identity and behavior

- Delegation inheritance: embeds `res.partner.bank` through field `res_partner_bank_id`
- Company consistency is checked automatically on company-bound relations

## Fields (8)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `res_partner_bank_id` | Resource Partner Bank | many to one | `res.partner.bank` | required; on delete of the target: cascade |
| `new_journal_name` | New Journal Name | single line text |  | required; writable through an inverse rule; default computed dynamically (lambda self: self.linked_journal_id.name); Help: Will be used to name the Journal related to this bank account |
| `linked_journal_id` | Journal | many to one | `account.journal` | computed by rule `_compute_linked_journal_id` (not stored); writable through an inverse rule; must belong to the same company |
| `bank_bic` | Bic | single line text |  | related through path `bank_id.bic` |
| `num_journals_without_account_bank` | Num Journals Without Account Bank | integer |  | default computed dynamically (lambda self: self._number_unlinked_journal('bank')) |
| `num_journals_without_account_credit` | Num Journals Without Account Credit | integer |  | default computed dynamically (lambda self: self._number_unlinked_journal('credit')) |
| `company_id` | Company | many to one | `res.company` | required; computed by rule `_compute_company_id` (not stored) |
| `l10n_ch_display_qr_bank_options` | Localization Ch Display Quick response Bank Options | boolean |  | computed by rule `_compute_l10n_ch_display_qr_bank_options` (not stored) |

## Operations (11)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_number_unlinked_journal` | internal rule | self, journal_type | `account` |  |  |
| `_onchange_acc_number` | on change | self | `account` | onchange: `acc_number` |  |
| `create` | lifecycle override | self, vals_list | `account` | model_create_multi | This wizard is only used to setup an account for the current active company, so we always inject the corresponding partner when creating the model. |
| `_onchange_new_journal_related_data` | on change | self | `account` | onchange: `linked_journal_id` |  |
| `_compute_linked_journal_id` | computation | self | `account` | depends: `journal_id` |  |
| `default_linked_journal_id` | operation | self, journal_type | `account` |  |  |
| `set_linked_journal_id` | operation | self | `account` |  | Called when saving the wizard. |
| `validate` | operation | self | `account` |  | Called by the validation button of this wizard. Serves as an extension hook in account_bank_statement_import. |
| `_compute_company_id` | computation | self | `account` |  |  |
| `_onchange_recompute_qr_iban` | on change | self | `l10n_ch` | onchange: `acc_number` |  |
| `_compute_l10n_ch_display_qr_bank_options` | computation | self | `l10n_ch` | depends: `partner_id`, `company_id` |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_manager` | yes | yes | yes | no | `account` |

## Views (4)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `account.setup_bank_account_wizard` | form |  | `acc_number`, `bank_id`, `bank_bic`, `linked_journal_id` | `Create`, `Cancel` |  | `account` |
| `account.setup_credit_card_account_wizard` | form |  | `acc_number`, `bank_id`, `linked_journal_id` | `Create`, `Cancel` |  | `account` |
| `base_iban.setup_bank_account_iban_wizard` | xpath | `account.setup_bank_account_wizard` |  |  |  | `base_iban` |
| `l10n_ch.setup_bank_account_wizard_inherit` | field | `account.setup_bank_account_wizard` | `bank_bic`, `l10n_ch_qr_iban` |  |  | `l10n_ch` |

Machine-readable definition: `../../../schemas/data/entities/account.setup.bank.manual.config.json`; views: `../../../schemas/interfaces/views/account.setup.bank.manual.config.json`.
