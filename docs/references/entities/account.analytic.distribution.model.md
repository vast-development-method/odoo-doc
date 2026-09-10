# Analytic Distribution Model (`account.analytic.distribution.model`)

**Transport name:** `account.analytic.distribution.model`  
**Storage name:** `account_analytic_distribution_model`  
**Kind:** persistent entity (one table)  
**Defined by package:** `analytic`  
**Extended by packages:** `account`

Description: Analytic Distribution Model

## Identity and behavior

- Mixins (classical inheritance): `analytic.mixin`
- Default ordering: `sequence, id desc`
- Display name field: `create_date`
- Company consistency is checked automatically on company-bound relations
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (8)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `sequence` | Sequence | integer |  | default `10` |
| `partner_id` | Partner | many to one | `res.partner` | on delete of the target: cascade; Help: Select a partner for which the analytic distribution will be used (e.g. create new customer invoice or Sales order if we select this partner, it will automatically take this as an analytic account) |
| `partner_category_id` | Partner Category | many to one | `res.partner.category` | on delete of the target: cascade; Help: Select a partner category for which the analytic distribution will be used (e.g. create new customer invoice or Sales order if we select this partner, it will automatically take this as an analytic account) |
| `company_id` | Company | many to one | `res.company` | default computed dynamically (lambda self: self.env.company); on delete of the target: cascade; Help: Select a company for which the analytic distribution will be used (e.g. create new customer invoice or Sales order if we select this company, it will automatically take this as an analytic account) |
| `account_prefix` | Accounts Prefix | single line text |  | Help: This analytic distribution will apply to all financial accounts sharing the prefix specified. |
| `product_id` | Product | many to one | `product.product` | on delete of the target: cascade; must belong to the same company; Help: Select a product for which the analytic distribution will be used (e.g. create new customer invoice or Sales order if we select this product, it will automatically take this as an analytic account) |
| `product_categ_id` | Product Category | many to one | `product.category` | on delete of the target: cascade; Help: Select a product category which will use analytic account specified in analytic default (e.g. create new customer invoice or Sales order if we select this product, it will automatically take this as an analytic account) |
| `prefix_placeholder` | Prefix Placeholder | single line text |  | computed by rule `_compute_prefix_placeholder` (not stored) |

## Operations (6)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_check_company_accounts` | validation | self | `analytic` | constrains: `company_id` | Ensure accounts specific to a company isn't used in any distribution model that wouldn't be specific to the company |
| `_get_distribution` | preparation rule | self, vals | `analytic` | model | Returns the combined distribution from all matching models based on the vals dict provided This method should be called to prefill analytic distribution field on several models |
| `_get_default_search_domain_vals` | preparation rule | self | `account`, `analytic` | model |  |
| `_get_applicable_models` | preparation rule | self, vals | `account`, `analytic` | model |  |
| `_create_domain` | internal rule | self, fname, value | `account`, `analytic` |  |  |
| `_compute_prefix_placeholder` | computation | self | `account` | depends: `analytic_precision` |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_company_accounts` | UserError | You defined a distribution with analytic account(s) belonging to a specific company but a model shared between companies or with a different company | `analytic` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_readonly` | no | yes | no | no | `account` |
| `account.group_account_invoice` | yes | yes | yes | yes | `account` |
| `group_analytic_accounting` | yes | yes | yes | yes | `analytic` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Analytic distribution model multi company rule | global (all users) | `['\|',('company_id','=',False),('company_id', 'parent_of', company_ids)]` | True | True | True | True |

## Views (4)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `account.account_analytic_distribution_model_tree_inherit` | data | `analytic.account_analytic_distribution_model_tree_view` | `prefix_placeholder`, `account_prefix`, `product_id`, `product_categ_id` |  |  | `account` |
| `account.account_analytic_distribution_model_form_inherit` | data | `analytic.account_analytic_distribution_model_form_view` | `prefix_placeholder`, `account_prefix`, `product_id`, `product_categ_id` |  |  | `account` |
| `analytic.account_analytic_distribution_model_tree_view` | list |  | `sequence`, `partner_id`, `partner_category_id`, `company_id`, `company_id`, `analytic_distribution` |  |  | `analytic` |
| `analytic.account_analytic_distribution_model_form_view` | form |  | `partner_id`, `partner_category_id`, `company_id`, `company_id`, `analytic_distribution` |  |  | `analytic` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `analytic.action_analytic_distribution_model` | Analytic Distribution Models | list,form |  |  |  | `analytic` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `account.menu_analytic__distribution_model` | Analytic Distribution Models |  | `analytic.action_analytic_distribution_model` | 10 | `analytic.group_analytic_accounting` |

Machine-readable definition: `../../../schemas/data/entities/account.analytic.distribution.model.json`; views: `../../../schemas/interfaces/views/account.analytic.distribution.model.json`.
