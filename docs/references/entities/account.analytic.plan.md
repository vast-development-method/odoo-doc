# Analytic Plans (`account.analytic.plan`)

**Transport name:** `account.analytic.plan`  
**Storage name:** `account_analytic_plan`  
**Kind:** persistent entity (one table)  
**Defined by package:** `analytic`  
**Extended by packages:** `stock_account`

Description: Analytic Plans

## Identity and behavior

- Default ordering: `sequence asc, id`
- Display name field: `complete_name`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (15)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required; writable through an inverse rule; translatable |
| `description` | Description | multi line text |  |  |
| `parent_id` | Parent | many to one | `account.analytic.plan` | writable through an inverse rule; indexed (btree_not_null); on delete of the target: cascade; restricted by domain `['!', ('id', 'child_of', id)]` |
| `parent_path` | Parent Path | single line text |  | indexed (btree) |
| `root_id` | Root | many to one | `account.analytic.plan` | computed by rule `_compute_root_id` (not stored); searchable through a search rule |
| `children_ids` | Childrens | one to many | `account.analytic.plan` | inverse field `parent_id` |
| `children_count` | Children Plans Count | integer |  | computed by rule `_compute_children_count` (not stored) |
| `complete_name` | Complete Name | single line text |  | computed by rule `_compute_complete_name` and stored; recursive dependency |
| `account_ids` | Accounts | one to many | `account.analytic.account` | inverse field `plan_id` |
| `account_count` | Analytic Accounts Count | integer |  | computed by rule `_compute_analytic_account_count` (not stored) |
| `all_account_count` | All Analytic Accounts Count | integer |  | computed by rule `_compute_all_analytic_account_count` (not stored) |
| `color` | Color | integer |  | default computed dynamically (_default_color) |
| `sequence` | Sequence | integer |  | default `10` |
| `default_applicability` | Default Applicability | selection |  | value is company dependent |
| `applicability_ids` | Applicability | one to many | `account.analytic.applicability` | restricted by domain `[('company_id', '=', current_company_id)]`; inverse field `analytic_plan_id` |

## Selection values

### `default_applicability` (Default Applicability)

| Value | Label |
|---|---|
| `optional` | Optional |
| `mandatory` | Mandatory |
| `unavailable` | Unavailable |

## Operations (29)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_default_color` | preparation rule | self | `analytic` |  |  |
| `_auto_init` | lifecycle override | self | `analytic` |  |  |
| `__get_all_plans` | internal rule | self | `analytic` |  |  |
| `_get_all_plans` | preparation rule | self | `analytic` |  |  |
| `_strict_column_name` | internal rule | self | `analytic` |  |  |
| `_column_name` | internal rule | self | `analytic` |  |  |
| `_inverse_name` | inverse computation | self | `analytic` |  |  |
| `_inverse_parent_id` | inverse computation | self | `analytic` |  |  |
| `_compute_root_id` | computation | self | `analytic` | depends: `parent_id`, `parent_path` |  |
| `_search_root_id` | search rule | self, operator, value | `analytic` |  |  |
| `_compute_complete_name` | computation | self | `analytic` | depends: `name`, `parent_id.complete_name` |  |
| `_compute_analytic_account_count` | computation | self | `analytic` | depends: `account_ids` |  |
| `_compute_all_analytic_account_count` | computation | self | `analytic` | depends: `account_ids`, `children_ids` |  |
| `_compute_children_count` | computation | self | `analytic` | depends: `children_ids` |  |
| `_onchange_parent_id` | on change | self | `analytic` | onchange: `parent_id` |  |
| `action_view_analytical_accounts` | user action | self | `analytic` |  |  |
| `action_view_children_plans` | user action | self | `analytic` |  |  |
| `get_relevant_plans` | operation | self, **kwargs | `analytic` | model | Returns the list of plans that should be available. This list is computed based on the applicabilities of root plans. |
| `_get_applicability` | preparation rule | self, **kwargs | `analytic` |  | Returns the applicability of the best applicability line or the default applicability |
| `unlink` | lifecycle override | self | `analytic` |  |  |
| `_hierarchy_name` | internal rule | self | `analytic` |  |  |
| `_is_subplan_field_used` | internal rule | self, field | `analytic` |  | Return `True` if there are analytic plans still on the same hierarchy level as what the field was created for.  :param field: the recordset of a field created to group by sub plan :rtype: bool |
| `_find_plan_column` | internal rule | self, model | `analytic` |  |  |
| `_find_related_field` | internal rule | self, model | `analytic` |  |  |
| `_sync_all_plan_column` | internal rule | self | `analytic` |  |  |
| `_sync_plan_column` | internal rule | self, model | `analytic` |  |  |
| `create` | lifecycle override | self, vals_list | `analytic` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `analytic` |  |  |
| `_calculate_distribution_amount` | internal rule | self, amount, percentage, total_percentage, distribution_on_each_plan | `stock_account` |  | Ensures that the total amount distributed across all lines always adds up to exactly `amount` per plan. We try to correct for compounding rounding errors by assigning the exact outstanding amount when we detect that a line will close out a plan's total percentage. However, since multiple plans can be assigned to a line, with different prior distributions, there is the possible edge case that one line closes out two (or more) tallies with different compounding errors. This means there is no one correct amount that we can assign to a line that will correctly close out both all plans. This is des |

## Validation and error messages (2)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `__get_all_plans` | UserError | A 'Project' plan needs to exist and its id needs to be set as `analytic.project_plan` in the system variables | `analytic` |
| `_onchange_parent_id` | UserError | You cannot add a parent to the base plan '%s' | `analytic` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_user` | yes | yes | yes | yes | `account` |
| `group_analytic_accounting` | yes | yes | yes | yes | `analytic` |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `account.account_analytic_plan_form_view_inherit_account` | data | `analytic.account_analytic_plan_form_view` | `display_account_prefix`, `account_prefix_placeholder`, `account_prefix`, `product_categ_id` |  |  | `account` |
| `analytic.account_analytic_plan_form_view` | form |  | `children_count`, `all_account_count`, `name`, `parent_id`, `default_applicability`, `color`, `applicability_ids`, `business_domain`, `company_id`, `applicability` | `action_view_children_plans`, `action_view_analytical_accounts` |  | `analytic` |
| `analytic.account_analytic_plan_tree_view` | list |  | `sequence`, `name`, `default_applicability`, `color` |  |  | `analytic` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `analytic.account_analytic_plan_action` | Analytic Plans | list,form | `[('parent_id', '=', False)]` |  |  | `analytic` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `account.account_analytic_plan_menu` | Analytic Plans |  | `analytic.account_analytic_plan_action` | 30 | `analytic.group_analytic_accounting` |

Machine-readable definition: `../../../schemas/data/entities/account.analytic.plan.json`; views: `../../../schemas/interfaces/views/account.analytic.plan.json`.
