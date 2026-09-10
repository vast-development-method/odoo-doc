# Tax Repartition Line (`account.tax.repartition.line`)

**Transport name:** `account.tax.repartition.line`  
**Storage name:** `account_tax_repartition_line`  
**Kind:** persistent entity (one table)  
**Defined by package:** `account`

Description: Tax Repartition Line

## Identity and behavior

- Default ordering: `document_type, repartition_type, sequence, id`
- Company consistency is checked automatically on company-bound relations
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (11)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `factor_percent` | % | float |  | required; default `100`; precision `[16, 12]`; Help: Factor to apply on the account move lines generated from this distribution line, in percents |
| `factor` | Factor Ratio | float |  | computed by rule `_compute_factor` (not stored); Help: Factor to apply on the account move lines generated from this distribution line |
| `repartition_type` | Based On | selection |  | required; default `tax`; Help: Base on which the factor will be applied. |
| `document_type` | Related to | selection |  | required |
| `account_id` | Account | many to one | `account.account` | restricted by domain `[('account_type', 'not in', ('asset_receivable', 'liability_payable', 'off_balance'))]`; must belong to the same company; Help: Account on which to post the tax amount |
| `tag_ids` | Tax Grids | many to many | `account.account.tag` | on delete of the target: restrict; restricted by domain `[["applicability", "=", "taxes"]]` |
| `tax_id` | Tax | many to one | `account.tax` | indexed (btree_not_null); on delete of the target: cascade; must belong to the same company |
| `company_id` | Company | many to one | `res.company` | related through path `tax_id.company_id` and stored; Help: The company this distribution line belongs to. |
| `sequence` | Sequence | integer |  | default `1`; Help: The order in which distribution lines are displayed and matched. For refunds to work properly, invoice distribution lines should be arranged in the same order as the credit note distribution lines they correspond to. |
| `use_in_tax_closing` | Tax Closing Entry | boolean |  | computed by rule `_compute_use_in_tax_closing` and stored; precomputed before insertion |
| `tag_ids_domain` | tag domain | binary |  | computed by rule `_compute_tag_ids_domain` (not stored); Help: Dynamic domain used for the tag that can be set on tax |

## Selection values

### `repartition_type` (Based On)

| Value | Label |
|---|---|
| `base` | Base |
| `tax` | of tax |

### `document_type` (Related to)

| Value | Label |
|---|---|
| `invoice` | Invoice |
| `refund` | Refund |

## Operations (5)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_tag_ids_domain` | computation | self | `account` | depends: `company_id.multi_vat_foreign_country_ids`, `company_id.account_fiscal_country_id` |  |
| `_compute_use_in_tax_closing` | computation | self | `account` | depends: `account_id`, `repartition_type` |  |
| `_compute_factor` | computation | self | `account` | depends: `factor_percent` |  |
| `_onchange_repartition_type` | on change | self | `account` | onchange: `repartition_type` |  |
| `_get_aml_target_tax_account` | preparation rule | self, force_caba_exigibility | `account` |  | Get the default tax account to set on a business line.  :return: An account.account record or an empty recordset. |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `account` |
| `account.group_account_readonly` | no | yes | no | no | `account` |
| `account.group_account_invoice` | no | yes | no | no | `account` |
| `account.group_account_manager` | yes | yes | yes | yes | `account` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Tax Repartition multi-company | global (all users) | `['\|',('company_id','=',False), ('company_id', 'parent_of', company_ids)]` | True | True | True | True |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `account.tax_repartition_line_tree` | list |  | `sequence`, `factor_percent`, `repartition_type`, `account_id`, `tag_ids`, `use_in_tax_closing`, `company_id`, `tag_ids_domain` |  |  | `account` |

Machine-readable definition: `../../../schemas/data/entities/account.tax.repartition.line.json`; views: `../../../schemas/interfaces/views/account.tax.repartition.line.json`.
