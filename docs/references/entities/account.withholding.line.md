# withholding line (`account.withholding.line`)

**Transport name:** `account.withholding.line`  
**Storage name:** `account_withholding_line`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `l10n_account_withholding_tax`

Description: withholding line

## Identity and behavior

- Mixins (classical inheritance): `analytic.mixin`
- Company consistency is checked automatically on company-bound relations

## Fields (24)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Sequence Number | single line text |  |  |
| `placeholder_value` | Placeholder Value | single line text |  | Help: Populated by the comodel during edition of the line. |
| `placeholder_type` | Placeholder Type | selection |  | required; computed by rule `_compute_placeholder_type` and stored; precomputed before insertion |
| `previous_placeholder_type` | Previous Placeholder Type | selection |  | computed by rule `_compute_placeholder_type` and stored; precomputed before insertion |
| `type_tax_use` | Type Tax Use | single line text |  | computed by rule `_compute_type_tax_use` (not stored) |
| `tax_id` | Tax | many to one | `account.tax` | required; restricted by domain `[('type_tax_use', '=', type_tax_use), ('is_withholding_tax_on_payment', '=', True)]`; must belong to the same company |
| `withholding_sequence_id` | Withholding Sequence | many to one |  | related through path `tax_id.withholding_sequence_id` |
| `source_base_amount_currency` | Source Base Amount Currency | monetary |  | currency taken from `source_currency_id` |
| `source_base_amount` | Source Base Amount | monetary |  | currency taken from `comodel_company_currency_id` |
| `source_tax_amount_currency` | Source Tax Amount Currency | monetary |  | currency taken from `source_currency_id` |
| `source_tax_amount` | Source Tax Amount | monetary |  | currency taken from `comodel_company_currency_id` |
| `source_currency_id` | Source Currency | many to one | `res.currency` |  |
| `source_tax_id` | Source Tax | many to one | `account.tax` |  |
| `original_base_amount` | Original Base Amount | monetary |  | computed by rule `_compute_original_amounts` (not stored); currency taken from `comodel_currency_id` |
| `original_tax_amount` | Original Tax Amount | monetary |  | computed by rule `_compute_original_amounts` (not stored); currency taken from `comodel_currency_id` |
| `base_amount` | Withholding base | monetary |  | computed by rule `_compute_base_amount` and stored; currency taken from `comodel_currency_id` |
| `amount` | Withholding amount | monetary |  | computed by rule `_compute_amount` and stored; currency taken from `comodel_currency_id` |
| `account_id` | Account | many to one | `account.account` | required; computed by rule `_compute_account_id` and stored; precomputed before insertion |
| `comodel_percentage_paid_factor` | Comodel Percentage Paid Factor | float |  | computed by rule `_compute_comodel_percentage_paid_factor` (not stored) |
| `comodel_date` | Comodel Date | date |  | computed by rule `_compute_comodel_date` (not stored) |
| `comodel_payment_type` | Comodel Payment Type | selection |  | computed by rule `_compute_comodel_payment_type` (not stored) |
| `company_id` | Company | many to one | `res.company` | required; computed by rule `_compute_company_id` and stored; precomputed before insertion |
| `comodel_company_currency_id` | Comodel Company Currency | many to one |  | related through path `company_id.currency_id` |
| `comodel_currency_id` | Comodel Currency | many to one | `res.currency` | required; computed by rule `_compute_comodel_currency_id` (not stored) |

## Selection values

### `placeholder_type` (Placeholder Type)

| Value | Label |
|---|---|
| `given_by_sequence` | Given By the Sequence |
| `given_by_name` | Given By the Name |
| `not_defined` | Not defined |

### `previous_placeholder_type` (Previous Placeholder Type)

| Value | Label |
|---|---|
| `given_by_sequence` | Given By the Sequence |
| `given_by_name` | Given By the Name |
| `not_defined` | Not defined |

### `comodel_payment_type` (Comodel Payment Type)

| Value | Label |
|---|---|
| `outbound` | Send Money |
| `inbound` | Receive Money |

## Operations (24)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_placeholder_type` | computation | self | `l10n_account_withholding_tax` | depends: `withholding_sequence_id`, `name` | Since the placeholder_value has to be recomputed on all lines by the comodel, we need a way to track the changed regarding the sequence and the name. Since the ORM is quite limited for such advance feature, we use a trick here: we store the current and the previous state of the placeholder to be able to detect the changes. |
| `_compute_original_amounts` | computation | self | `l10n_account_withholding_tax` | depends: `source_base_amount_currency`, `source_base_amount`, `source_tax_amount_currency`, `source_tax_amount`, `source_currency_id`, `comodel_currency_id`, `company_id`, `comodel_date`, `tax_id` | Computes the two original_xx_amount fields; that are used during computation of the withholding line base and tax amounts. These amounts correspond to the source amounts (from the payment or register payment wizard) after converting them to the line currency. |
| `_compute_base_amount` | computation | self | `l10n_account_withholding_tax` | depends: `original_base_amount`, `comodel_percentage_paid_factor` | Computation of the base amount is done by using a paid factor. This factor is unused on payments, but used for lines on the register payment wizard in order to dynamically support installments, early payment discounts,... |
| `_compute_amount` | computation | self | `l10n_account_withholding_tax` | depends: `source_tax_id`, `tax_id`, `base_amount` | Compute the tax amount by multiplying the original tax amount (amount in currency, at the time of creation) by a ratio calculated from the current base amount and the original base amount. |
| `_compute_account_id` | computation | self | `l10n_account_withholding_tax` | depends: `company_id` | If there is an account set on the withholding_tax_base_account_id, this field will be invisible and use that account as default value. |
| `_compute_type_tax_use` | computation | self | `l10n_account_withholding_tax` |  |  |
| `_compute_company_id` | computation | self | `l10n_account_withholding_tax` |  |  |
| `_compute_currency_id` | computation | self | `l10n_account_withholding_tax` |  |  |
| `_compute_comodel_percentage_paid_factor` | computation | self | `l10n_account_withholding_tax` |  |  |
| `_compute_comodel_date` | computation | self | `l10n_account_withholding_tax` |  |  |
| `_compute_comodel_payment_type` | computation | self | `l10n_account_withholding_tax` |  |  |
| `_compute_comodel_currency_id` | computation | self | `l10n_account_withholding_tax` |  |  |
| `_update_placeholders` | internal rule | self | `l10n_account_withholding_tax` |  | Update the placeholders for the lines in self; updating them sequentially so that the placeholders make sense. |
| `_constrains_base_amount` | validation | self | `l10n_account_withholding_tax` | constrains: `base_amount` | It wouldn't make sense to register a withholding tax with no base amount. |
| `_constrains_account_id` | validation | self | `l10n_account_withholding_tax` | constrains: `account_id` | The account on the line cannot be one deemed as liquidity account, otherwise it will cause issues with the final entry. |
| `_prepare_base_line_for_taxes_computation` | preparation rule | self | `l10n_account_withholding_tax` |  | Convert self to a tax base line using the correct structure needed for tax computation. This is used when preparing the journal items representing the withholding lines in the final payment entry. |
| `_prepare_withholding_amls_create_values` | preparation rule | self | `l10n_account_withholding_tax` |  | Prepare and return a list of values that will be used to create the journal items for the withholding lines.  For an invoice for 1000 with 10% withholding tax: Outstanding:              900.0 Receivable:               -1000.0 Tax withheld:             100.0 WHT base:                 1000.0 WHT base counterpart:     1000.0  :return: A list of dictionaries, each one being a journal item to be created. |
| `_get_withholding_tax_domain` | preparation rule | self, company, payment_type | `l10n_account_withholding_tax` | model | Construct and return a domain that will filter withholding taxes available for this company and payment type. |
| `_need_update_withholding_lines_placeholder` | internal rule | self | `l10n_account_withholding_tax` |  | Determines if the lines' placeholders needs update or not. |
| `_get_grouping_key` | preparation rule | self | `l10n_account_withholding_tax` |  | Helper returning the grouping key for this line; should match what is done in _prepare_withholding_lines_commands. |
| `_prepare_withholding_lines_commands` | preparation rule | self, base_lines, company | `l10n_account_withholding_tax` |  | Calculate the withholding tax amounts by using the provided tax base lines, and then compare the resulting values with the withholding line in self to determine which line should be updated, deleted or created.  :returns A list of commands that should be used to update the withholding line field in the calling model. |
| `_get_valid_liquidity_accounts` | preparation rule | self | `l10n_account_withholding_tax` |  | Get the valid liquidity accounts for the payment; we need to ensure that the line account does not match any of them. |
| `_get_comodel_partner` | preparation rule | self | `l10n_account_withholding_tax` |  | Get the partner from the comodel record; in order to have it available when required. |
| `_is_refund` | internal rule | self | `l10n_account_withholding_tax` |  | When refunding an invoice with withholding taxes, we need to ensure that the base line we use to create the final journal entry is tagged as refund correctly to ensure the correct application of the tax repartition line. :return: True if the withholding line concerns a refund. |

## Validation and error messages (3)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_constrains_base_amount` | UserError | The base amount of a withholding tax line must be above 0. | `l10n_account_withholding_tax` |
| `_constrains_account_id` | UserError | The account "%(account_name)s" is not valid to use on withholding lines. | `l10n_account_withholding_tax` |
| `_prepare_withholding_amls_create_values` | UserError | Please enter the withholding number for the tax %(tax_name)s | `l10n_account_withholding_tax` |

Machine-readable definition: `../../../schemas/data/entities/account.withholding.line.json`.
