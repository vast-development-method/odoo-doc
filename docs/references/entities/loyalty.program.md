# Loyalty Program (`loyalty.program`)

**Transport name:** `loyalty.program`  
**Storage name:** `loyalty_program`  
**Kind:** persistent entity (one table)  
**Defined by package:** `loyalty`  
**Extended by packages:** `pos_loyalty`, `sale_loyalty`, `sale_loyalty_delivery`, `website_sale_loyalty`

Description: Loyalty Program

## Identity and behavior

- Mixins (classical inheritance): `pos.load.mixin`, `website.multi.mixin`
- Default ordering: `sequence`
- Display name field: `name`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (37)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Program Name | single line text |  | required; translatable |
| `active` | Active | boolean |  | default `True` |
| `sequence` | Sequence | integer |  | not copied on duplication |
| `company_id` | Company | many to one | `res.company` | default computed dynamically (lambda self: self.env.company) |
| `currency_id` | Currency | many to one | `res.currency` | required; computed by rule `_compute_currency_id` and stored; precomputed before insertion |
| `currency_symbol` | Currency Symbol | single line text |  | related through path `currency_id.symbol` |
| `pricelist_ids` | Pricelist | many to many | `product.pricelist` | restricted by domain `[('currency_id', '=', currency_id)]`; Help: This program is specific to this pricelist set. |
| `total_order_count` | Total Order Count | integer |  | computed by rule `_compute_total_order_count` (not stored) |
| `rule_ids` | Conditional rules | one to many | `loyalty.rule` | computed by rule `_compute_from_program_type` and stored; inverse field `program_id` |
| `reward_ids` | Rewards | one to many | `loyalty.reward` | computed by rule `_compute_from_program_type` and stored; inverse field `program_id` |
| `communication_plan_ids` | Communication Plan | one to many | `loyalty.mail` | computed by rule `_compute_from_program_type` and stored; inverse field `program_id` |
| `mail_template_id` | Email template | many to one | `mail.template` | computed by rule `_compute_mail_template_id` (not stored); writable through an inverse rule |
| `trigger_product_ids` | Trigger Product | many to many |  | related through path `rule_ids.product_ids` |
| `coupon_ids` | Coupon | one to many | `loyalty.card` | inverse field `program_id` |
| `coupon_count` | Coupon Count | integer |  | computed by rule `_compute_coupon_count` (not stored) |
| `coupon_count_display` | Items | single line text |  | computed by rule `_compute_coupon_count_display` (not stored) |
| `program_type` | Program Type | selection |  | required; default `promotion` |
| `date_from` | Start Date | date |  | Help: The start date is included in the validity period of this program |
| `date_to` | End date | date |  | Help: The end date is included in the validity period of this program |
| `limit_usage` | Limit Usage | boolean |  |  |
| `max_usage` | Max Usage | integer |  |  |
| `applies_on` | Applies On | selection |  | required; computed by rule `_compute_from_program_type` and stored; default `current` |
| `trigger` | Trigger | selection |  | computed by rule `_compute_from_program_type` and stored; Help: Automatic: Customers will be eligible for a reward automatically in their cart.         Use a code: Customers will be eligible for a reward if they enter a code. |
| `portal_visible` | Portal Visible | boolean |  | default ; Help: Show in web portal, PoS customer ticket, eCommerce checkout, the number of points available          and used by reward. |
| `portal_point_name` | Portal Point Name | single line text |  | computed by rule `_compute_portal_point_name` and stored; default `Points`; translatable |
| `is_nominative` | Is Nominative | boolean |  | computed by rule `_compute_is_nominative` (not stored) |
| `is_payment_program` | Is Payment Program | boolean |  | computed by rule `_compute_is_payment_program` (not stored) |
| `payment_program_discount_product_id` | Discount Product | many to one | `product.product` | read only; computed by rule `_compute_payment_program_discount_product_id` (not stored); Help: Product used in the sales order to apply the discount. |
| `available_on` | Available On | boolean |  | Help: Manage where your program should be available for use. |
| `pos_config_ids` | Point of Sales | many to many | `pos.config` | computed by rule `_compute_pos_config_ids` and stored; Help: Restrict publishing to those shops. Note: A program will only be used in the shops using the same currency as the program. |
| `pos_order_count` | PoS Order Count | integer |  | computed by rule `_compute_pos_order_count` (not stored) |
| `pos_ok` | Point of Sale | boolean |  | default `True` |
| `pos_report_print_id` | Print Report | many to one | `ir.actions.report` | computed by rule `_compute_pos_report_print_id` (not stored); writable through an inverse rule; restricted by domain `[["model", "=", "loyalty.card"]]`; Help: This is used to print the generated gift cards from PoS. |
| `order_count` | Order Count | integer |  | computed by rule `_compute_order_count` (not stored) |
| `sale_ok` | Sales | boolean |  | default `True` |
| `ecommerce_ok` | Available on Website | boolean |  | default `True` |
| `show_non_published_product_warning` | Show Non Published Product Warning | boolean |  | computed by rule `_compute_show_non_published_product_warning` (not stored) |

## Selection values

### `program_type` (Program Type)

| Value | Label |
|---|---|
| `coupons` | Coupons |
| `gift_card` | Gift Card |
| `loyalty` | Loyalty Cards |
| `promotion` | Promotions |
| `ewallet` | eWallet |
| `promo_code` | Discount Code |
| `buy_x_get_y` | Buy X Get Y |
| `next_order_coupons` | Next Order Coupons |

### `applies_on` (Applies On)

| Value | Label |
|---|---|
| `current` | Current order |
| `future` | Future orders |
| `both` | Current & Future orders |

### `trigger` (Trigger)

| Value | Label |
|---|---|
| `auto` | Automatic |
| `with_code` | Use a code |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_check_max_usage` | Constraint | `CHECK (limit_usage = False OR max_usage > 0)` | Max usage must be strictly positive if a limit is used. | `loyalty` |

## Operations (36)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `default_get` | lifecycle override | self, fields | `loyalty` | model |  |
| `_check_pricelist_currency` | validation | self | `loyalty` | constrains: `currency_id`, `pricelist_ids` |  |
| `_check_date_from_date_to` | validation | self | `loyalty` | constrains: `date_from`, `date_to` |  |
| `_constrains_reward_ids` | validation | self | `loyalty` | constrains: `reward_ids` |  |
| `_compute_total_order_count` | computation | self | `loyalty`, `pos_loyalty`, `sale_loyalty` |  |  |
| `_compute_coupon_count_display` | computation | self | `loyalty` | depends: `coupon_count`, `program_type` |  |
| `_compute_mail_template_id` | computation | self | `loyalty` | depends: `communication_plan_ids.mail_template_id` |  |
| `_inverse_mail_template_id` | inverse computation | self | `loyalty` |  |  |
| `_compute_currency_id` | computation | self | `loyalty` | depends: `company_id` |  |
| `_compute_coupon_count` | computation | self | `loyalty` | depends: `coupon_ids` |  |
| `_compute_is_nominative` | computation | self | `loyalty` | depends: `program_type`, `applies_on` |  |
| `_compute_is_payment_program` | computation | self | `loyalty` | depends: `program_type` |  |
| `_compute_payment_program_discount_product_id` | computation | self | `loyalty` | depends: `reward_ids.discount_line_product_id` |  |
| `_program_items_name` | internal rule | self | `loyalty` | model |  |
| `_program_type_default_values` | internal rule | self | `loyalty`, `sale_loyalty_delivery` | model |  |
| `_compute_from_program_type` | computation | self | `loyalty` | depends: `program_type` |  |
| `_compute_portal_point_name` | computation | self | `loyalty` | depends: `currency_id`, `program_type` |  |
| `_get_valid_products` | preparation rule | self, products | `loyalty` |  | Returns a dict containing the products that match per rule of the program |
| `action_open_loyalty_cards` | user action | self | `loyalty` |  |  |
| `_unlink_except_active` | internal rule | self | `loyalty` | ondelete |  |
| `write` | lifecycle override | self, vals | `loyalty` |  |  |
| `get_program_templates` | operation | self | `loyalty`, `sale_loyalty_delivery` | model | Returns the templates to be used for promotional programs. |
| `create_from_template` | operation | self, template_id | `loyalty` | model | Creates the program from the template id defined in `get_program_templates`.  Returns an action leading to that new record. |
| `_get_template_values` | preparation rule | self | `loyalty`, `sale_loyalty_delivery` | model | Returns the values to create a program using the template keys defined above. |
| `create` | lifecycle override | self, vals_list | `loyalty` | model_create_multi | trigger_product_ids will overwrite product ids defined in a loyalty rule in certain instances. Thus, it should be explicitly removed from an incoming vals dict unless, of course, it was actually a visible field. |
| `_load_pos_data_domain` | internal rule | self, data, config | `pos_loyalty` | model |  |
| `_load_pos_data_fields` | internal rule | self, config | `pos_loyalty` | model |  |
| `_load_pos_data_read` | internal rule | self, records, config | `pos_loyalty` | model |  |
| `_unrelevant_records` | internal rule | self, config | `pos_loyalty` |  |  |
| `_compute_pos_report_print_id` | computation | self | `pos_loyalty` | depends: `communication_plan_ids.pos_report_print_id` |  |
| `_inverse_pos_report_print_id` | inverse computation | self | `pos_loyalty` |  |  |
| `_compute_pos_config_ids` | computation | self | `pos_loyalty` | depends: `pos_ok` |  |
| `_compute_pos_order_count` | computation | self | `pos_loyalty` |  |  |
| `_compute_order_count` | computation | self | `sale_loyalty` |  |  |
| `_compute_show_non_published_product_warning` | computation | self | `website_sale_loyalty` | depends: `program_type`, `trigger_product_ids.website_published` |  |
| `action_program_share` | user action | self | `website_sale_loyalty` |  |  |

## Validation and error messages (5)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_pricelist_currency` | UserError | The loyalty program's currency must be the same as all it's pricelists ones. | `loyalty` |
| `_check_date_from_date_to` | UserError | The validity period's start date must be anterior or equal to its end date. | `loyalty` |
| `_constrains_reward_ids` | ValidationError | A program must have at least one reward. | `loyalty` |
| `_unlink_except_active` | UserError | You can not delete a program in an active state | `loyalty` |
| `_inverse_pos_report_print_id` | UserError | You must set '%(mail_template)s' before setting '%(report)s'. | `pos_loyalty` |

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
| Loyalty program multi company rule | global (all users) | `['\|', ('company_id', 'in', company_ids + [False]), ('company_id', 'parent_of', company_ids)]` | True | True | True | True |

## Views (9)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `loyalty.loyalty_program_view_form` | form |  | `coupon_count`, `active`, `applies_on`, `name`, `program_type`, `trigger_product_ids`, `payment_program_discount_product_id`, `mail_template_id`, `currency_id`, `currency_symbol`, `pricelist_ids`, `portal_point_name`, `portal_point_name`, `portal_visible`, `portal_visible`, `trigger`, `trigger`, `applies_on`, `applies_on`, `date_from`, `date_to`, `limit_usage`, `max_usage`, `total_order_count`, `company_id`, `company_id`, `available_on`, `portal_point_name`, `rule_ids`, `reward_ids`, `reward_ids`, `communication_plan_ids` | `Generate Coupons`, `Generate Gift Cards`, `Generate eWallet`, `action_open_loyalty_cards` |  | `loyalty` |
| `loyalty.loyalty_program_view_tree` | list |  | `sequence`, `name`, `program_type`, `coupon_count_display`, `company_id` |  |  | `loyalty` |
| `loyalty.loyalty_program_view_search` | search |  | `name` |  | `Archived` | `loyalty` |
| `loyalty.loyalty_program_gift_ewallet_view_form` | form | `loyalty_program_view_form` |  |  |  | `loyalty` |
| `pos_loyalty.loyalty_program_view_form_inherit_pos_loyalty` | field | `loyalty.loyalty_program_view_form` | `mail_template_id`, `pos_report_print_id` |  |  | `pos_loyalty` |
| `pos_loyalty.loyalty_program_view_tree_inherit_pos_loyalty` | field | `loyalty.loyalty_program_view_tree` | `company_id`, `pos_config_ids` |  |  | `pos_loyalty` |
| `sale_loyalty.loyalty_program_view_form_inherit_sale_loyalty` | xpath | `loyalty.loyalty_program_view_form` |  |  |  | `sale_loyalty` |
| `website_sale_loyalty.loyalty_program_view_form_inherit_website_sale_loyalty` | xpath | `sale_loyalty.loyalty_program_view_form_inherit_sale_loyalty` |  |  |  | `website_sale_loyalty` |
| `website_sale_loyalty.loyalty_program_view_tree_inherit_website_sale_loyalty` | field | `loyalty.loyalty_program_view_tree` | `coupon_count_display`, `website_id` |  |  | `website_sale_loyalty` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `loyalty.loyalty_program_discount_loyalty_action` | Discount & Loyalty | list,form | `[('program_type', 'not in', ('gift_card', 'ewallet'))]` |  |  | `loyalty` |
| `loyalty.loyalty_program_gift_ewallet_action` | Gift cards & eWallet | list,form | `[('program_type', 'in', ('gift_card', 'ewallet'))]` | `{'menu_type': 'gift_ewallet', 'default_program_type': 'gift_card'}` |  | `loyalty` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `pos_loyalty.menu_discount_loyalty_type_config` | Discount & Loyalty | `point_of_sale.pos_config_menu_catalog` | `loyalty.loyalty_program_discount_loyalty_action` | 91 | `point_of_sale.group_pos_manager` |
| `pos_loyalty.menu_gift_ewallet_type_config` | Gift cards & eWallet | `point_of_sale.pos_config_menu_catalog` | `loyalty.loyalty_program_gift_ewallet_action` | 92 | `point_of_sale.group_pos_manager` |
| `sale_loyalty.menu_discount_loyalty_type_config` |  | `sale.product_menu_catalog` | `loyalty.loyalty_program_discount_loyalty_action` | 40 | `sales_team.group_sale_manager` |
| `sale_loyalty.menu_gift_ewallet_type_config` |  | `sale.product_menu_catalog` | `loyalty.loyalty_program_gift_ewallet_action` | 50 | `sales_team.group_sale_manager` |
| `website_sale_loyalty.menu_discount_loyalty_type_config` |  |  | `loyalty.loyalty_program_discount_loyalty_action` |  |  |
| `website_sale_loyalty.menu_gift_ewallet_type_config` |  |  | `loyalty.loyalty_program_gift_ewallet_action` |  |  |

Machine-readable definition: `../../../schemas/data/entities/loyalty.program.json`; views: `../../../schemas/interfaces/views/loyalty.program.json`.
