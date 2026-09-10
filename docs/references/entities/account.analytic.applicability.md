# Analytic Plan's Applicabilities (`account.analytic.applicability`)

**Transport name:** `account.analytic.applicability`  
**Storage name:** `account_analytic_applicability`  
**Kind:** persistent entity (one table)  
**Defined by package:** `analytic`  
**Extended by packages:** `account`, `sale`, `hr_expense`, `hr_timesheet`, `purchase`, `mrp_account`, `project_stock_account`

Description: Analytic Plan's Applicabilities

## Identity and behavior

- Company consistency is checked automatically on company-bound relations
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (8)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `analytic_plan_id` | Analytic Plan | many to one | `account.analytic.plan` | indexed (btree_not_null) |
| `business_domain` | Domain | selection |  | required; on delete of the target: {"stock_picking": "cascade"}; extended by packages `account`, `sale`, `hr_expense`, `hr_timesheet`, `purchase`, `mrp_account`, `project_stock_account` |
| `applicability` | Applicability | selection |  | required |
| `company_id` | Company | many to one | `res.company` | default computed dynamically (lambda self: self.env.company) |
| `account_prefix` | Financial Accounts Prefixes | single line text |  | Help: Prefix that defines which accounts from the financial accounting this applicability should apply on. |
| `product_categ_id` | Product Category | many to one | `product.category` |  |
| `display_account_prefix` | Display Account Prefix | boolean |  | computed by rule `_compute_display_account_prefix` (not stored); Help: Defines if the field account prefix should be displayed |
| `account_prefix_placeholder` | Account Prefix Placeholder | single line text |  | computed by rule `_compute_prefix_placeholder` (not stored) |

## Selection values

### `business_domain` (Domain)

| Value | Label |
|---|---|
| `general` | Miscellaneous |
| `invoice` | Invoice |
| `bill` | Vendor Bill |
| `sale_order` | Sale Order |
| `expense` | Expense |
| `timesheet` | Timesheet |
| `purchase_order` | Purchase Order |
| `manufacturing_order` | Manufacturing Order |
| `stock_picking` | Stock Picking |

### `applicability` (Applicability)

| Value | Label |
|---|---|
| `optional` | Optional |
| `mandatory` | Mandatory |
| `unavailable` | Unavailable |

## Operations (6)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `create` | lifecycle override | self, vals_list | `analytic` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `analytic` |  |  |
| `_unlink_clear_cache` | internal rule | self | `analytic` | ondelete |  |
| `_get_score` | preparation rule | self, **kwargs | `account`, `analytic` |  | Gives the score of an applicability with the parameters of kwargs |
| `_compute_prefix_placeholder` | computation | self | `account` | depends: `account_prefix`, `business_domain` |  |
| `_compute_display_account_prefix` | computation | self | `account`, `hr_expense` | depends: `business_domain` |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_user` | yes | yes | yes | yes | `account` |
| `group_analytic_accounting` | yes | yes | yes | yes | `analytic` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Analytic applicability multi company rule | global (all users) | `['\|',('company_id','=',False),('company_id', 'parent_of', company_ids)]` | True | True | True | True |

Machine-readable definition: `../../../schemas/data/entities/account.analytic.applicability.json`.
