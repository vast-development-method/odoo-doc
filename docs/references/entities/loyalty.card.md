# Loyalty Coupon (`loyalty.card`)

**Transport name:** `loyalty.card`  
**Storage name:** `loyalty_card`  
**Kind:** persistent entity (one table)  
**Defined by package:** `loyalty`  
**Extended by packages:** `pos_loyalty`, `sale_loyalty`, `website_sale_loyalty`

Description: Loyalty Coupon

## Identity and behavior

- Mixins (classical inheritance): `mail.thread`, `pos.load.mixin`
- Display name field: `code`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (17)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `program_id` | Program | many to one | `loyalty.program` | default computed dynamically (lambda self: self.env.context.get('active_id', None)); indexed (btree_not_null); on delete of the target: restrict |
| `program_type` | Program Type | selection |  | related through path `program_id.program_type` |
| `company_id` | Company | many to one |  | related through path `program_id.company_id` and stored; precomputed before insertion |
| `currency_id` | Currency | many to one |  | related through path `program_id.currency_id` |
| `partner_id` | Partner | many to one | `res.partner` | indexed |
| `points` | Points | float |  | changes are tracked in the message thread |
| `point_name` | Point Name | single line text |  | read only; related through path `program_id.portal_point_name` |
| `points_display` | Points Display | single line text |  | computed by rule `_compute_points_display` (not stored) |
| `code` | Code | single line text |  | required; default computed dynamically (lambda self: self._generate_code()) |
| `expiration_date` | Expiration Date | date |  |  |
| `use_count` | Use Count | integer |  | computed by rule `_compute_use_count` (not stored) |
| `active` | Active | boolean |  | default `True` |
| `history_ids` | History | one to many | `loyalty.history` | read only; inverse field `card_id` |
| `source_pos_order_id` | PoS Order Reference | many to one | `pos.order` | Help: PoS order where this coupon was generated. |
| `source_pos_order_partner_id` | PoS Order Customer | many to one | `res.partner` | related through path `source_pos_order_id.partner_id` |
| `order_id` | Order Reference | many to one | `sale.order` | read only; Help: The sales order from which coupon is generated |
| `order_id_partner_id` | Sale Order Customer | many to one | `res.partner` | related through path `order_id.partner_id` |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_card_code_unique` | Constraint | `UNIQUE(code)` | A coupon/loyalty card must have a unique code. | `loyalty` |

## Operations (24)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_generate_code` | internal rule | self | `loyalty` | model | Barcode identifiable codes. |
| `_compute_display_name` | computation | self | `loyalty` | depends: `program_id`, `code` |  |
| `_contrains_code` | validation | self | `loyalty` | constrains: `code` |  |
| `_compute_points_display` | computation | self | `loyalty` | depends: `points`, `point_name` |  |
| `_restrict_expiration_on_loyalty` | on change | self | `loyalty` | onchange: `expiration_date` |  |
| `_format_points` | internal rule | self, points | `loyalty` |  |  |
| `_compute_use_count` | computation | self | `loyalty`, `pos_loyalty`, `sale_loyalty` |  |  |
| `_get_default_template` | preparation rule | self | `loyalty`, `pos_loyalty`, `sale_loyalty` |  |  |
| `_get_mail_author` | preparation rule | self | `loyalty`, `sale_loyalty` |  |  |
| `_get_signature` | preparation rule | self | `loyalty`, `pos_loyalty`, `sale_loyalty` |  | To be overriden |
| `_has_source_order` | internal rule | self | `loyalty`, `pos_loyalty`, `sale_loyalty` |  |  |
| `action_coupon_send` | user action | self | `loyalty` |  | Open a window to compose an email, with the default template returned by `_get_default_template` message loaded by default |
| `_send_creation_communication` | internal rule | self, force_send | `loyalty` |  | Sends the 'At Creation' communication plan if it exist for the given coupons. |
| `_send_points_reach_communication` | internal rule | self, points_changes | `loyalty` |  | Send the 'When Reaching' communicaton plans for the given coupons.  If a coupons passes multiple milestones we will only send the one with the highest target. |
| `create` | lifecycle override | self, vals_list | `loyalty` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `loyalty` |  |  |
| `action_loyalty_update_balance` | user action | self | `loyalty` |  |  |
| `_load_pos_data_domain` | internal rule | self, data, config | `pos_loyalty` | model |  |
| `_load_pos_data_fields` | internal rule | self, config | `pos_loyalty` | model |  |
| `_mail_get_partner_fields` | messaging hook | self, introspect_fields | `pos_loyalty`, `sale_loyalty` |  |  |
| `get_gift_card_status` | operation | self, gift_code, config_id | `pos_loyalty` | model |  |
| `get_loyalty_card_partner_by_code` | operation | self, code | `pos_loyalty` | model |  |
| `action_archive` | lifecycle override | self | `sale_loyalty` |  |  |
| `action_coupon_share` | user action | self | `website_sale_loyalty` |  |  |

## Validation and error messages (2)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_contrains_code` | ValidationError | A trigger with the same code as one of your coupon already exists. | `loyalty` |
| `_restrict_expiration_on_loyalty` | ValidationError | Expiration date cannot be set on a loyalty card. | `loyalty` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | no | no | no | `loyalty` |
| `point_of_sale.group_pos_user` | no | yes | yes | no | `pos_loyalty` |
| `point_of_sale.group_pos_manager` | yes | yes | yes | no | `pos_loyalty` |
| `sales_team.group_sale_salesman` | no | yes | yes | no | `sale_loyalty` |
| `sales_team.group_sale_manager` | yes | yes | yes | no | `sale_loyalty` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Loyalty card multi company rule | global (all users) | `['\|', ('company_id', 'in', company_ids + [False]), ('company_id', 'parent_of', company_ids)]` | True | True | True | True |

## Views (6)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `loyalty.loyalty_card_view_form` | form |  | `code`, `expiration_date`, `partner_id`, `points_display`, `history_ids`, `description`, `order_id`, `create_date`, `issued`, `used` | `action_loyalty_update_balance` |  | `loyalty` |
| `loyalty.loyalty_card_view_tree` | list |  | `code`, `create_date`, `points_display`, `expiration_date`, `program_id`, `partner_id` | `Send` |  | `loyalty` |
| `loyalty.loyalty_card_view_search` | search |  | `code`, `partner_id`, `program_id` |  | `Active`, `Inactive` | `loyalty` |
| `pos_loyalty.loyalty_card_view_form_inherit_pos_loyalty` | xpath | `loyalty.loyalty_card_view_form` | `source_pos_order_id` |  |  | `pos_loyalty` |
| `sale_loyalty.loyalty_card_view_form_inherit_sale_loyalty` | field | `loyalty.loyalty_card_view_form` | `partner_id`, `order_id` |  |  | `sale_loyalty` |
| `website_sale_loyalty.loyalty_card_view_tree_inherit_website_sale_loyalty` | button | `loyalty.loyalty_card_view_tree` |  | `action_coupon_send`, `Share` |  | `website_sale_loyalty` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `loyalty.loyalty_card_action` | Coupons | list,form | `[('program_id', '=', active_id)]` | `{'create': False}` |  | `loyalty` |

## Printable reports

| Report | Name | Type | Template | File name rule | Attachment rule |
|---|---|---|---|---|---|
| `loyalty.report_loyalty_card` | Coupon Code | qweb-pdf | `loyalty.loyalty_report_i18n` |  |  |
| `loyalty.report_gift_card` | Gift Card | qweb-pdf | `loyalty.gift_card_report_i18n` |  |  |

## Email templates

| Template | Name | Subject |
|---|---|---|
| `loyalty.mail_template_gift_card` | Gift Card: Gift Card Information | Your Gift Card at {{ object.company_id.name }} |
| `loyalty.mail_template_loyalty_card` | Coupon: Coupon Information | Your reward coupon from {{ object.program_id.company_id.name }} |

Machine-readable definition: `../../../schemas/data/entities/loyalty.card.json`; views: `../../../schemas/interfaces/views/loyalty.card.json`.
