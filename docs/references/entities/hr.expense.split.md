# Expense Split (`hr.expense.split`)

**Transport name:** `hr.expense.split`  
**Storage name:** `hr_expense_split`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `hr_expense`  
**Extended by packages:** `sale_expense`

Description: Expense Split

## Identity and behavior

- Mixins (classical inheritance): `analytic.mixin`
- Company consistency is checked automatically on company-bound relations

## Fields (17)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Description | single line text |  | required |
| `wizard_id` | Wizard | many to one | `hr.expense.split.wizard` |  |
| `expense_id` | Expense | many to one | `hr.expense` |  |
| `product_id` | Product | many to one | `product.product` | required; restricted by domain `[["can_be_expensed", "=", true]]`; must belong to the same company |
| `tax_ids` | Tax | many to many | `account.tax` | restricted by domain `[('type_tax_use', '=', 'purchase')]`; must belong to the same company |
| `total_amount_currency` | Total In Currency | monetary |  | required; computed by rule `_compute_from_product_id` and stored |
| `tax_amount_currency` | Tax amount in Currency | monetary |  | computed by rule `_compute_tax_amount_currency` (not stored) |
| `employee_id` | Employee | many to one | `hr.employee` | required |
| `company_id` | Company | many to one | `res.company` |  |
| `currency_id` | Currency | many to one | `res.currency` |  |
| `product_has_tax` | Whether tax is defined on a selected product | boolean |  | computed by rule `_compute_product_has_tax` (not stored) |
| `product_has_cost` | Is product with non zero cost selected | boolean |  | computed by rule `_compute_from_product_id` and stored |
| `approval_state` | Approval State | selection |  | read only; not copied on duplication |
| `approval_date` | Approval Date | date and time |  | read only |
| `manager_id` | Manager | many to one | `res.users` | read only; restricted by domain `lambda self: [('all_group_ids', 'in', self.env.ref('hr_expense.group_hr_expense_team_approver').id)]` |
| `sale_order_id` | Customer to Reinvoice | many to one | `sale.order` | computed by rule `_compute_sale_order_id` and stored; restricted by domain `[('state', '=', 'sale'), ('company_id', '=', company_id)]` |
| `can_be_reinvoiced` | Can be reinvoiced | boolean |  | computed by rule `_compute_can_be_reinvoiced` (not stored) |

## State fields

State machine fields of this entity: `approval_state`. Transitions are specified in the domain documents.

## Operations (8)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `default_get` | lifecycle override | self, fields | `hr_expense` | model |  |
| `_compute_tax_amount_currency` | computation | self | `hr_expense` | depends: `total_amount_currency`, `tax_ids` |  |
| `_compute_from_product_id` | computation | self | `hr_expense` | depends: `product_id` |  |
| `_onchange_product_id` | on change | self | `hr_expense` | onchange: `product_id` | In case we switch to the product without taxes defined on it, taxes should be removed. Computed method won't be good for this purpose, as we don't want to recompute and reset taxes in case they are removed on purpose during splitting. |
| `_compute_product_has_tax` | computation | self | `hr_expense` | depends: `product_id` |  |
| `_get_values` | preparation rule | self | `hr_expense`, `sale_expense` |  |  |
| `_compute_can_be_reinvoiced` | computation | self | `sale_expense` | depends: `product_id` |  |
| `_compute_sale_order_id` | computation | self | `sale_expense` | depends: `can_be_reinvoiced` |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | yes | yes | yes | yes | `hr_expense` |

Machine-readable definition: `../../../schemas/data/entities/hr.expense.split.json`.
