# Tax Group (`account.tax.group`)

**Transport name:** `account.tax.group`  
**Storage name:** `account_tax_group`  
**Kind:** persistent entity (one table)  
**Defined by package:** `account`  
**Extended by packages:** `point_of_sale`, `l10n_ar`, `l10n_ec`

Description: Tax Group

## Identity and behavior

- Mixins (classical inheritance): `pos.load.mixin`
- Default ordering: `sequence asc, id`
- Company consistency is checked automatically on company-bound relations
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (13)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required; translatable |
| `sequence` | Sequence | integer |  | default `10` |
| `company_id` | Company | many to one | `res.company` | required; default computed dynamically (lambda self: self.env.company) |
| `tax_payable_account_id` | Tax Payable Account | many to one | `account.account` | must belong to the same company; Help: Tax current account used as a counterpart to the Tax Closing Entry when in favor of the authorities. |
| `tax_receivable_account_id` | Tax Receivable Account | many to one | `account.account` | must belong to the same company; Help: Tax current account used as a counterpart to the Tax Closing Entry when in favor of the company. |
| `advance_tax_payment_account_id` | Tax Advance Account | many to one | `account.account` | must belong to the same company; Help: Downpayments posted on this account will be considered by the Tax Closing Entry. |
| `country_id` | Country | many to one | `res.country` | computed by rule `_compute_country_id` and stored; precomputed before insertion; Help: The country for which this tax group is applicable. |
| `country_code` | Country Code | single line text |  | related through path `country_id.code` |
| `preceding_subtotal` | Preceding Subtotal | single line text |  | translatable; Help: If set, this value will be used on documents as the label of a subtotal excluding this tax group before displaying it. If not set, the tax group will be displayed after the 'Untaxed amount' subtotal. |
| `pos_receipt_label` | PoS receipt label | single line text |  |  |
| `l10n_ar_tribute_afip_code` | Tribute ARCA Code | selection |  | read only; indexed |
| `l10n_ar_vat_afip_code` | value-added tax ARCA Code | selection |  | read only; indexed |
| `l10n_ec_type` | Type Ecuadorian Tax | selection |  | Help: Ecuadorian taxes subtype |

## Selection values

### `l10n_ar_tribute_afip_code` (Tribute ARCA Code)

| Value | Label |
|---|---|
| `01` | 01 - National Taxes |
| `02` | 02 - Provincial Taxes |
| `03` | 03 - Municipal Taxes |
| `04` | 04 - Internal Taxes |
| `06` | 06 - VAT perception |
| `07` | 07 - IIBB perception |
| `08` | 08 - Municipal Taxes Perceptions |
| `09` | 09 - Other Perceptions |
| `99` | 99 - Others |

### `l10n_ar_vat_afip_code` (value-added tax ARCA Code)

| Value | Label |
|---|---|
| `0` | Not Applicable |
| `1` | Untaxed |
| `2` | Exempt |
| `3` | 0% |
| `4` | 10.5% |
| `5` | 21% |
| `6` | 27% |
| `8` | 5% |
| `9` | 2,5% |

## Operations (4)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_country_id` | computation | self | `account` | depends: `company_id` |  |
| `_load_pos_data_domain` | internal rule | self, data, config | `point_of_sale` | model |  |
| `_load_pos_data_fields` | internal rule | self, config | `point_of_sale` | model |  |
| `check_uninstall_required` | operation | self | `l10n_ar` | ondelete | Make sure we don't uninstall a required tax group |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `check_uninstall_required` | UserError | The tax group '%s' can't be removed, since it is required in the Argentinian localization. | `l10n_ar` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `account` |
| `account.group_account_readonly` | no | yes | no | no | `account` |
| `account.group_account_invoice` | no | yes | no | no | `account` |
| `account.group_account_manager` | yes | yes | yes | yes | `account` |
| `sales_team.group_sale_salesman` | no | yes | no | no | `sale` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Tax group multi-company | global (all users) | `[('company_id', 'parent_of', company_ids)]` | True | True | True | True |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `account.account_tax_group_view_search` | search |  | `name`, `country_id` |  | `Country` | `account` |
| `account.view_tax_group_tree` | list |  | `sequence`, `name`, `country_id`, `company_id`, `company_id`, `country_code`, `tax_payable_account_id`, `tax_receivable_account_id`, `advance_tax_payment_account_id`, `preceding_subtotal` |  |  | `account` |
| `account.view_tax_group_form` | form |  | `company_id`, `name`, `country_id`, `company_id`, `sequence`, `pos_receipt_label`, `tax_payable_account_id`, `tax_receivable_account_id`, `advance_tax_payment_account_id`, `preceding_subtotal` |  |  | `account` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `account.action_tax_group` | Tax Groups | list,form |  |  |  | `account` |

Machine-readable definition: `../../../schemas/data/entities/account.tax.group.json`; views: `../../../schemas/interfaces/views/account.tax.group.json`.
