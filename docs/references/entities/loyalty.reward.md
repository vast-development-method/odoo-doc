# Loyalty Reward (`loyalty.reward`)

**Transport name:** `loyalty.reward`  
**Storage name:** `loyalty_reward`  
**Kind:** persistent entity (one table)  
**Defined by package:** `loyalty`  
**Extended by packages:** `pos_loyalty`, `sale_loyalty`, `sale_loyalty_delivery`

Description: Loyalty Reward

## Identity and behavior

- Mixins (classical inheritance): `pos.load.mixin`
- Default ordering: `required_points asc`
- Display name field: `description`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (29)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `active` | Active | boolean |  | default `True` |
| `program_id` | Program | many to one | `loyalty.program` | required; indexed; on delete of the target: cascade |
| `program_type` | Program Type | selection |  | related through path `program_id.program_type` |
| `company_id` | Company | many to one |  | related through path `program_id.company_id` and stored |
| `currency_id` | Currency | many to one |  | related through path `program_id.currency_id` |
| `description` | Description | single line text |  | required; computed by rule `_compute_description` and stored; translatable; precomputed before insertion |
| `reward_type` | Reward Type | selection |  | required; default `discount`; on delete of the target: {"shipping": "set default"}; extended by packages `sale_loyalty_delivery` |
| `user_has_debug` | User Has Debug | boolean |  | computed by rule `_compute_user_has_debug` (not stored) |
| `discount` | Discount | float |  | default `10` |
| `discount_mode` | Discount Mode | selection |  | required; default `percent` |
| `discount_applicability` | Discount Applicability | selection |  | default `order` |
| `discount_product_domain` | Discount Product Domain | single line text |  | default `[]` |
| `discount_product_ids` | Discounted Products | many to many | `product.product` |  |
| `discount_product_category_id` | Discounted Prod. Categories | many to one | `product.category` |  |
| `discount_product_tag_id` | Discounted Prod. Tag | many to one | `product.tag` |  |
| `all_discount_product_ids` | All Discount Product | many to many | `product.product` | computed by rule `_compute_all_discount_product_ids` (not stored) |
| `reward_product_domain` | Reward Product Domain | single line text |  | computed by rule `_compute_reward_product_domain` (not stored) |
| `discount_max_amount` | Max Discount | monetary |  | Help: This is the max amount this reward may discount, leave to 0 for no limit. |
| `discount_line_product_id` | Discount Line Product | many to one | `product.product` | not copied on duplication; on delete of the target: restrict; Help: Product used in the sales order to apply the discount. Each reward has its own product for reporting purpose |
| `is_global_discount` | Is Global Discount | boolean |  | computed by rule `_compute_is_global_discount` (not stored) |
| `reward_product_id` | Product | many to one | `product.product` | restricted by domain `[["type", "!=", "combo"]]` |
| `reward_product_tag_id` | Product Tag | many to one | `product.tag` |  |
| `multi_product` | Multi Product | boolean |  | computed by rule `_compute_multi_product` (not stored) |
| `reward_product_ids` | Reward Products | many to many | `product.product` | computed by rule `_compute_multi_product` (not stored); searchable through a search rule; Help: These are the products that can be claimed with this rule. |
| `reward_product_qty` | Reward Product Qty | integer |  | default `1` |
| `reward_product_uom_id` | Reward Product Unit of measure | many to one | `uom.uom` | computed by rule `_compute_reward_product_uom_id` (not stored) |
| `required_points` | Points needed | float |  | default `1` |
| `point_name` | Point Name | single line text |  | read only; related through path `program_id.portal_point_name` |
| `clear_wallet` | Clear Wallet | boolean |  | default  |

## Selection values

### `reward_type` (Reward Type)

| Value | Label |
|---|---|
| `product` | Free Product |
| `discount` | Discount |
| `shipping` | Free Shipping |

### `discount_applicability` (Discount Applicability)

| Value | Label |
|---|---|
| `order` | Order |
| `cheapest` | Cheapest Product |
| `specific` | Specific Products |

## Database constraints and indexes (3)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_required_points_positive` | Constraint | `CHECK (required_points > 0)` | The required points for a reward must be strictly positive. | `loyalty` |
| `_product_qty_positive` | Constraint | `CHECK (reward_type != 'product' OR reward_product_qty > 0)` | The reward product quantity must be strictly positive. | `loyalty` |
| `_discount_positive` | Constraint | `CHECK (reward_type != 'discount' OR discount > 0)` | The discount must be strictly positive. | `loyalty` |

## Operations (27)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `default_get` | lifecycle override | self, fields | `loyalty` | model |  |
| `_get_discount_mode_select` | preparation rule | self | `loyalty` |  |  |
| `_compute_display_name` | computation | self | `loyalty` | depends: `program_id`, `description` |  |
| `_compute_reward_product_uom_id` | computation | self | `loyalty` | depends: `reward_product_id.product_tmpl_id.uom_id`, `reward_product_tag_id` |  |
| `_find_all_category_children` | internal rule | self, category_id, child_ids | `loyalty` |  |  |
| `_get_discount_product_domain` | preparation rule | self | `loyalty` |  |  |
| `_get_active_products_domain` | preparation rule | self | `loyalty` | model |  |
| `_compute_reward_product_domain` | computation | self | `loyalty` | depends: `discount_product_domain` |  |
| `_compute_all_discount_product_ids` | computation | self | `loyalty` | depends: `discount_product_ids`, `discount_product_category_id`, `discount_product_tag_id`, `discount_product_domain` |  |
| `_compute_multi_product` | computation | self | `loyalty` | depends: `reward_product_id`, `reward_product_tag_id`, `reward_type` |  |
| `_search_reward_product_ids` | search rule | self, operator, value | `loyalty` |  |  |
| `_compute_description` | computation | self | `loyalty`, `sale_loyalty_delivery` | depends: `reward_type`, `reward_product_id`, `discount_mode`, `reward_product_tag_id`, `discount`, `currency_id`, `discount_applicability`, `all_discount_product_ids` |  |
| `_compute_is_global_discount` | computation | self | `loyalty` | depends: `reward_type`, `discount_applicability`, `discount_mode` |  |
| `_compute_user_has_debug` | computation | self | `loyalty` | depends_context: `uid`; depends: `reward_type` |  |
| `_check_reward_product_id_no_combo` | validation | self | `loyalty` | constrains: `reward_product_id` |  |
| `_create_missing_discount_line_products` | internal rule | self | `loyalty` |  |  |
| `create` | lifecycle override | self, vals_list | `loyalty` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `loyalty` |  |  |
| `update_field_translations` | operation | self, field_name, translations, source_lang | `loyalty` |  |  |
| `unlink` | lifecycle override | self | `loyalty`, `pos_loyalty`, `sale_loyalty` |  |  |
| `_get_discount_product_values` | preparation rule | self | `loyalty`, `pos_loyalty`, `sale_loyalty` |  |  |
| `_load_pos_data_domain` | internal rule | self, data, config | `pos_loyalty` | model |  |
| `_load_pos_data_fields` | internal rule | self, config | `pos_loyalty` | model |  |
| `_load_pos_data_read` | internal rule | self, records, config | `pos_loyalty` | model |  |
| `_get_reward_product_domain_fields` | preparation rule | self, config | `pos_loyalty` |  |  |
| `_replace_ilike_with_in` | internal rule | self, domain_str | `pos_loyalty` |  |  |
| `_parse_domain` | internal rule | self, domain | `pos_loyalty` |  |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_reward_product_id_no_combo` | ValidationError | A reward product can't be of type "combo". | `loyalty` |

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
| Loyalty reward multi company rule | global (all users) | `['\|', ('company_id', 'in', company_ids + [False]), ('company_id', 'parent_of', company_ids)]` | True | True | True | True |

## Views (4)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `loyalty.loyalty_reward_view_form` | form |  | `program_type`, `user_has_debug`, `multi_product`, `reward_product_uom_id`, `reward_product_ids`, `all_discount_product_ids`, `reward_type`, `discount`, `discount_mode`, `discount_applicability`, `reward_product_qty`, `reward_product_id`, `reward_product_tag_id`, `discount_max_amount`, `discount_product_domain`, `discount_product_ids`, `discount_product_category_id`, `discount_product_tag_id`, `required_points`, `point_name`, `clear_wallet`, `description`, `discount_line_product_id` |  |  | `loyalty` |
| `loyalty.loyalty_reward_view_kanban` | kanban |  | `company_id`, `currency_id`, `reward_type`, `discount_applicability`, `clear_wallet`, `program_type`, `user_has_debug`, `discount`, `discount_mode`, `discount_max_amount`, `discount_product_ids`, `discount_product_category_id`, `discount_product_tag_id`, `discount_product_domain`, `reward_product_id`, `reward_product_qty`, `reward_product_tag_id`, `point_name`, `required_points`, `point_name`, `required_points`, `point_name` |  |  | `loyalty` |
| `sale_loyalty_delivery.loyalty_reward_view_form_inherit_loyalty_delivery` | group | `loyalty.loyalty_reward_view_form` | `discount_max_amount` |  |  | `sale_loyalty_delivery` |
| `sale_loyalty_delivery.loyalty_reward_view_kanban_inherit_loyalty_delivery` | div | `loyalty.loyalty_reward_view_kanban` | `discount_max_amount` |  |  | `sale_loyalty_delivery` |

Machine-readable definition: `../../../schemas/data/entities/loyalty.reward.json`; views: `../../../schemas/interfaces/views/loyalty.reward.json`.
