# Account Tag (`account.account.tag`)

**Transport name:** `account.account.tag`  
**Storage name:** `account_account_tag`  
**Kind:** persistent entity (one table)  
**Defined by package:** `account`

Description: Account Tag

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (7)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Tag Name | single line text |  | required; translatable |
| `applicability` | Applicability | selection |  | required; default `accounts` |
| `color` | Color Index | integer |  |  |
| `active` | Active | boolean |  | default `True`; Help: Set active to false to hide the Account Tag without removing it. |
| `country_id` | Country | many to one | `res.country` | Help: Country for which this tag is available, when applied on taxes. |
| `report_expression_id` | Report Expression | many to one | `account.report.expression` | computed by rule `_compute_report_expression_id` (not stored) |
| `balance_negate` | Balance Negate | boolean |  | computed by rule `_compute_report_expression_id` (not stored) |

## Selection values

### `applicability` (Applicability)

| Value | Label |
|---|---|
| `accounts` | Accounts |
| `taxes` | Taxes |
| `products` | Products |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_name_uniq` | Constraint | `unique(name, applicability, country_id)` | A tag with the same name and applicability already exists in this country. | `account` |

## Operations (9)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_display_name` | computation | self | `account` | depends: `applicability`, `country_id`; depends_context: `company` |  |
| `_compute_report_expression_id` | computation | self | `account` | depends: `name` |  |
| `_field_to_sql` | internal rule | self, alias, field_expr, query | `account` |  |  |
| `create` | lifecycle override | self, vals_list | `account` | model_create_multi |  |
| `_get_tax_tags` | preparation rule | self, tag_name, country_id | `account` | model | Returns all the tax tags corresponding to the tag name given in parameter in the specified country. |
| `_get_tax_tags_domain` | preparation rule | self, formula, country_id | `account` | model | Returns a domain to search for all the tax tags corresponding to the formula given in parameter in the specified country. |
| `_get_related_tax_report_expressions` | preparation rule | self | `account` |  |  |
| `_unlink_except_master_tags` | internal rule | self | `account` | ondelete |  |
| `_translate_tax_tags` | internal rule | self, langs, tag_ids | `account` |  | Translate tax tags having the same name as report lines. |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_unlink_except_master_tags` | UserError | You cannot delete this account tag (%s), it is used on the chart of account definition. | `account` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `account` |
| `account.group_account_user` | yes | yes | yes | yes | `account` |
| `account.group_account_readonly` | no | yes | no | no | `account` |
| `account.group_account_invoice` | no | yes | no | no | `account` |
| `group_purchase_user` | no | yes | no | no | `purchase` |
| `sales_team.group_sale_salesman` | no | yes | no | no | `sale` |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `account.account_tag_view_form` | form |  | `active`, `name`, `applicability`, `country_id` |  |  | `account` |
| `account.account_tag_view_tree` | list |  | `name`, `applicability`, `country_id` |  |  | `account` |
| `account.account_tag_view_search` | search |  | `name` |  | `Archived` | `account` |

Machine-readable definition: `../../../schemas/data/entities/account.account.tag.json`; views: `../../../schemas/interfaces/views/account.account.tag.json`.
