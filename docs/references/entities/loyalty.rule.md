# Loyalty Rule (`loyalty.rule`)

**Transport name:** `loyalty.rule`  
**Storage name:** `loyalty_rule`  
**Kind:** persistent entity (one table)  
**Defined by package:** `loyalty`  
**Extended by packages:** `pos_loyalty`, `website_sale_loyalty`

Description: Loyalty Rule

## Identity and behavior

- Mixins (classical inheritance): `pos.load.mixin`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (23)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `active` | Active | boolean |  | default `True` |
| `program_id` | Program | many to one | `loyalty.program` | required; indexed; on delete of the target: cascade |
| `program_type` | Program Type | selection |  | related through path `program_id.program_type` |
| `company_id` | Company | many to one |  | related through path `program_id.company_id` and stored |
| `currency_id` | Currency | many to one |  | related through path `program_id.currency_id` |
| `user_has_debug` | User Has Debug | boolean |  | computed by rule `_compute_user_has_debug` (not stored) |
| `product_domain` | Product Domain | single line text |  | default `[]` |
| `product_ids` | Products | many to many | `product.product` |  |
| `product_category_id` | Categories | many to one | `product.category` |  |
| `product_tag_id` | Product Tag | many to one | `product.tag` |  |
| `reward_point_amount` | Reward | float |  | default `1` |
| `reward_point_split` | Split per unit | boolean |  | default ; Help: Whether to separate reward coupons per matched unit, only applies to 'future' programs and trigger mode per money spent or unit paid... |
| `reward_point_name` | Reward Point Name | single line text |  | read only; related through path `program_id.portal_point_name` |
| `reward_point_mode` | Reward Point Mode | selection |  | required; default `order` |
| `minimum_qty` | Minimum Quantity | integer |  | default `1` |
| `minimum_amount` | Minimum Purchase | monetary |  |  |
| `minimum_amount_tax_mode` | Minimum Amount Tax Mode | selection |  | required; default `incl` |
| `mode` | Application | selection |  | computed by rule `_compute_mode` and stored |
| `code` | Discount code | single line text |  | computed by rule `_compute_code` and stored |
| `valid_product_ids` | Valid Product | many to many | `product.product` | computed by rule `_compute_valid_product_ids` (not stored); association table `Valid Products`; Help: These are the products that are valid for this rule. |
| `any_product` | Any Product | boolean |  | computed by rule `_compute_valid_product_ids` (not stored); Help: Technical field, whether all product match |
| `promo_barcode` | Barcode | single line text |  | computed by rule `_compute_promo_barcode` and stored; Help: A technical field used as an alternative to the promo code. This is automatically generated when the promo code is changed. |
| `website_id` | Website | many to one |  | related through path `program_id.website_id` and stored |

## Selection values

### `minimum_amount_tax_mode` (Minimum Amount Tax Mode)

| Value | Label |
|---|---|
| `incl` | tax included |
| `excl` | tax excluded |

### `mode` (Application)

| Value | Label |
|---|---|
| `auto` | Automatic |
| `with_code` | With a promotion code |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_reward_point_amount_positive` | Constraint | `CHECK (reward_point_amount > 0)` | Rule points reward must be strictly positive. | `loyalty` |

## Operations (14)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `default_get` | lifecycle override | self, fields | `loyalty` | model |  |
| `_get_reward_point_mode_selection` | preparation rule | self | `loyalty` |  |  |
| `_constraint_trigger_multi` | validation | self | `loyalty` | constrains: `reward_point_split` |  |
| `_constrains_code` | validation | self | `loyalty`, `website_sale_loyalty` | constrains: `code`, `active`; constrains: `code`, `website_id`, `active` |  |
| `_compute_code` | computation | self | `loyalty` | depends: `mode` |  |
| `_compute_mode` | computation | self | `loyalty` | depends: `code` |  |
| `_compute_user_has_debug` | computation | self | `loyalty` | depends_context: `uid`; depends: `mode` |  |
| `_get_valid_product_domain` | preparation rule | self | `loyalty` |  |  |
| `_get_valid_products` | preparation rule | self | `loyalty` |  |  |
| `_compute_amount` | computation | self, currency_to | `loyalty` |  |  |
| `_load_pos_data_domain` | internal rule | self, data, config | `pos_loyalty` | model |  |
| `_load_pos_data_fields` | internal rule | self, config | `pos_loyalty` | model |  |
| `_compute_valid_product_ids` | computation | self | `pos_loyalty` | depends: `product_ids`, `product_category_id`, `product_tag_id`, `product_domain` |  |
| `_compute_promo_barcode` | computation | self | `pos_loyalty` | depends: `code` |  |

## Validation and error messages (5)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_constraint_trigger_multi` | ValidationError | Split per unit is not allowed for Loyalty and eWallet programs. | `loyalty` |
| `_constrains_code` | ValidationError | The promo code must be unique. | `loyalty` |
| `_constrains_code` | ValidationError | A coupon with the same code was found. | `loyalty` |
| `_constrains_code` | ValidationError | A coupon with the same code was found. | `website_sale_loyalty` |
| `_constrains_code` | ValidationError | The promo code must be unique. | `website_sale_loyalty` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | no | no | no | `loyalty` |
| `point_of_sale.group_pos_user` | no | yes | no | no | `pos_loyalty` |
| `point_of_sale.group_pos_manager` | yes | yes | yes | yes | `pos_loyalty` |
| `sales_team.group_sale_salesman` | no | yes | no | no | `sale_loyalty` |
| `sales_team.group_sale_manager` | yes | yes | yes | yes | `sale_loyalty` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Loyalty rule multi company rule | global (all users) | `['\|', ('company_id', 'in', company_ids + [False]), ('company_id', 'parent_of', company_ids)]` | True | True | True | True |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `loyalty.loyalty_rule_view_form` | form |  | `program_type`, `user_has_debug`, `code`, `minimum_qty`, `minimum_amount`, `minimum_amount_tax_mode`, `product_domain`, `product_ids`, `product_category_id`, `product_tag_id`, `reward_point_amount`, `reward_point_name`, `reward_point_mode` |  |  | `loyalty` |
| `loyalty.loyalty_rule_view_kanban` | kanban |  | `minimum_amount_tax_mode`, `program_type`, `user_has_debug`, `reward_point_split`, `code`, `minimum_qty`, `minimum_amount`, `product_ids`, `product_category_id`, `product_tag_id`, `product_domain`, `reward_point_amount`, `reward_point_name`, `reward_point_mode` |  |  | `loyalty` |

Machine-readable definition: `../../../schemas/data/entities/loyalty.rule.json`; views: `../../../schemas/interfaces/views/loyalty.rule.json`.
