# Account Cash Rounding (`account.cash.rounding`)

**Transport name:** `account.cash.rounding`  
**Storage name:** `account_cash_rounding`  
**Kind:** persistent entity (one table)  
**Defined by package:** `account`  
**Extended by packages:** `point_of_sale`, `point_of_sale`

Description: Account Cash Rounding

## Identity and behavior

- Mixins (classical inheritance): `pos.load.mixin`
- Company consistency is checked automatically on company-bound relations
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (6)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required; translatable |
| `rounding` | Rounding Precision | float |  | required; default `0.01`; Help: Represent the non-zero value smallest coinage (for example, 0.05). |
| `strategy` | Rounding Strategy | selection |  | required; default `add_invoice_line`; Help: Specify which way will be used to round the invoice amount to the rounding precision |
| `profit_account_id` | Profit Account | many to one | `account.account` | value is company dependent; on delete of the target: restrict; restricted by domain `[('account_type', 'not in', ('asset_receivable', 'liability_payable'))]`; must belong to the same company |
| `loss_account_id` | Loss Account | many to one | `account.account` | value is company dependent; on delete of the target: restrict; restricted by domain `[('account_type', 'not in', ('asset_receivable', 'liability_payable'))]`; must belong to the same company |
| `rounding_method` | Rounding Method | selection |  | required; default `HALF-UP`; Help: The tie-breaking rule used for float rounding operations |

## Selection values

### `strategy` (Rounding Strategy)

| Value | Label |
|---|---|
| `biggest_tax` | Modify tax amount |
| `add_invoice_line` | Add a rounding line |

### `rounding_method` (Rounding Method)

| Value | Label |
|---|---|
| `UP` | Up |
| `DOWN` | Down |
| `HALF-UP` | Nearest |

## Operations (7)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `validate_rounding` | validation | self | `account` | constrains: `rounding` |  |
| `round` | operation | self, amount | `account` |  | Compute the rounding on the amount passed as parameter.  :param amount: the amount to round :return: the rounded amount depending the rounding value and the rounding method |
| `compute_difference` | operation | self, currency, amount | `account` |  | Compute the difference between the base_amount and the amount after rounding. For example, base_amount=23.91, after rounding=24.00, the result will be 0.09.  :param currency: The currency. :param amount: The amount :return: round(difference) |
| `_unlink_except_pos_config` | internal rule | self | `point_of_sale` | ondelete |  |
| `_check_session_state` | validation | self | `point_of_sale` | constrains: `rounding`, `rounding_method`, `strategy` |  |
| `_load_pos_data_domain` | internal rule | self, data, config | `point_of_sale` | model |  |
| `_load_pos_data_fields` | internal rule | self, config | `point_of_sale` | model |  |

## Validation and error messages (3)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `validate_rounding` | ValidationError | Please set a strictly positive rounding value. | `account` |
| `_unlink_except_pos_config` | UserError | You cannot delete a rounding method that is used in a Point of Sale configuration. | `point_of_sale` |
| `_check_session_state` | ValidationError | You are not allowed to change the cash rounding configuration while a pos session using it is already opened. | `point_of_sale` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_readonly` | no | yes | no | no | `account` |
| `account.group_account_invoice` | yes | yes | yes | yes | `account` |
| `group_pos_user` | no | yes | no | no | `point_of_sale` |

## Views (4)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `account.rounding_form_view` | form |  | `name`, `rounding`, `strategy`, `profit_account_id`, `loss_account_id`, `rounding_method` |  |  | `account` |
| `account.rounding_search_view` | search |  | `name` |  |  | `account` |
| `account.rounding_tree_view` | list |  | `name`, `rounding`, `rounding_method` |  |  | `account` |
| `point_of_sale.pos_rounding_form_view_inherited` | xpath | `account.rounding_form_view` |  |  |  | `point_of_sale` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `account.rounding_list_action` | Cash Roundings | list,form |  |  |  | `account` |

Machine-readable definition: `../../../schemas/data/entities/account.cash.rounding.json`; views: `../../../schemas/interfaces/views/account.cash.rounding.json`.
