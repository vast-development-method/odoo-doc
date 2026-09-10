# Pricelist (`product.pricelist`)

**Transport name:** `product.pricelist`  
**Storage name:** `product_pricelist`  
**Kind:** persistent entity (one table)  
**Defined by package:** `product`  
**Extended by packages:** `point_of_sale`, `website_sale`, `loyalty`, `partnership`

Description: Pricelist

## Identity and behavior

- Mixins (classical inheritance): `mail.thread`, `mail.activity.mixin`, `pos.load.mixin`
- Default ordering: `sequence, id, name`
- Display name search fields: `["name", "currency_id"]`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (12)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Pricelist Name | single line text |  | required; translatable |
| `active` | Active | boolean |  | default `True`; Help: If unchecked, it will allow you to hide the pricelist without removing it. |
| `sequence` | Sequence | integer |  | default `16` |
| `currency_id` | Currency | many to one | `res.currency` | required; default computed dynamically (_default_currency_id); changes are tracked in the message thread |
| `company_id` | Company | many to one | `res.company` | default computed dynamically (lambda self: self.env.company); changes are tracked in the message thread |
| `country_group_ids` | Country Groups | many to many | `res.country.group` | changes are tracked in the message thread; association table `res_country_group_pricelist_rel` |
| `item_ids` | Pricelist Rules | one to many | `product.pricelist.item` | restricted by domain `lambda self: self._domain_item_ids()`; inverse field `pricelist_id` |
| `website_id` | Website | many to one | `website` | default computed dynamically (_default_website); changes are tracked in the message thread; on delete of the target: restrict; restricted by domain `[('company_id', '=?', company_id)]`; Help: If you want a pricelist to be available on a website,you must fill in this field or make it selectable.Otherwise, the pricelist will not apply to any website. |
| `code` | E-commerce Promotional Code | single line text |  | visible only to groups `base.group_user` |
| `selectable` | Selectable | boolean |  | Help: Allow the end user to choose this price list |
| `partners_count` | Partners Count | integer |  | computed by rule `_compute_partners_count` (not stored) |
| `partners_label` | Partners Label | single line text |  | related through path `company_id.partnership_label` |

## Operations (34)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_default_currency_id` | preparation rule | self | `product` |  |  |
| `_base_domain_item_ids` | internal rule | self | `product` |  |  |
| `_domain_item_ids` | internal rule | self | `product` |  |  |
| `_compute_display_name` | computation | self | `product` | depends: `currency_id` |  |
| `write` | lifecycle override | self, vals | `product`, `website_sale` |  |  |
| `copy_data` | lifecycle override | self, default | `product` |  |  |
| `_get_products_price` | preparation rule | self, products, *args, **kwargs | `product` |  | Compute the pricelist prices for the specified products, quantity & uom.  Note: self and self.ensure_one()  :param products: recordset of products (product.product/product.template) :param float quantity: quantity of products requested (in given uom) :param currency: record of currency (res.currency) (optional) :param uom: unit of measure (uom.uom record) (optional)     If not specified, prices returned are expressed in product uoms :param date: date to use for price computation and currency conversions (optional) :type date: date or datetime  :returns: {product_id: product price}, considering |
| `_get_product_price` | preparation rule | self, product, *args, **kwargs | `product` |  | Compute the pricelist price for the specified product, qty & uom.  Note: self and self.ensure_one()  :param product: product record (product.product/product.template) :param float quantity: quantity of products requested (in given uom) :param currency: record of currency (res.currency) (optional) :param uom: unit of measure (uom.uom record) (optional)     If not specified, prices returned are expressed in product uoms :param date: date to use for price computation and currency conversions (optional) :type date: date or datetime  :returns: unit price of the product, considering pricelist rules  |
| `_get_product_price_rule` | preparation rule | self, product, *args, **kwargs | `product` |  | Compute the pricelist price & rule for the specified product, qty & uom.  Note: self and self.ensure_one()  :param product: product record (product.product/product.template) :param float quantity: quantity of products requested (in given uom) :param currency: record of currency (res.currency) (optional) :param uom: unit of measure (uom.uom record) (optional)     If not specified, prices returned are expressed in product uoms :param date: date to use for price computation and currency conversions (optional) :type date: date or datetime  :returns: (product unit price, applied pricelist rule id)  |
| `_get_product_rule` | preparation rule | self, product, *args, **kwargs | `product` |  | Compute the pricelist price & rule for the specified product, qty & uom.  Note: self and self.ensure_one()  :param product: product record (product.product/product.template) :param float quantity: quantity of products requested (in given uom) :param currency: record of currency (res.currency) (optional) :param uom: unit of measure (uom.uom record) (optional)     If not specified, prices returned are expressed in product uoms :param date: date to use for price computation and currency conversions (optional) :type date: date or datetime  :returns: applied pricelist rule id :rtype: int or False |
| `_compute_price_rule` | computation | self, products, quantity, currency, uom, date, compute_price, **kwargs | `product` |  | Low-level method - Mono pricelist, multi products Returns: dict{product_id: (price, suitable_rule) for the given pricelist}  Note: self and self.ensure_one()  :param products: recordset of products (product.product/product.template) :param float quantity: quantity of products requested (in given uom) :param currency: record of currency (res.currency)                  note: currency.ensure_one() :param uom: unit of measure (uom.uom record)     If not specified, prices returned are expressed in product uoms :param date: date to use for price computation and currency conversions :type date: date  |
| `_get_applicable_rules` | preparation rule | self, products, date, **kwargs | `product` |  |  |
| `_get_applicable_rules_domain` | preparation rule | self, products, date, **kwargs | `product` |  |  |
| `_price_get` | internal rule | self, product, quantity, **kwargs | `product` |  | Multi pricelist, mono product - returns price per pricelist |
| `_compute_price_rule_multi` | computation | self, products, quantity, uom, date, **kwargs | `product` |  | Low-level method - Multi pricelist, multi products Returns: dict{product_id: dict{pricelist_id: (price, suitable_rule)} } |
| `_get_country_pricelist_multi` | preparation rule | self, country_ids | `product` |  |  |
| `_get_partner_pricelist_multi` | preparation rule | self, partner_ids | `product` | model | Retrieve the applicable pricelist for given partners in a given company.  It will return the first found pricelist in this order: First, the pricelist of the specific property (res_id set), this one         is created when saving a pricelist on the partner form view. Else, it will return the pricelist of the partner country group Else, it will return the generic property (res_id not set) Else, it will return the first available pricelist if any  :return: a dict {partner_id: pricelist} |
| `_get_partner_pricelist_multi_search_domain_hook` | preparation rule | self, company_id | `product`, `website_sale` |  |  |
| `_get_partner_pricelist_multi_filter_hook` | preparation rule | self | `product`, `website_sale` |  |  |
| `get_import_templates` | operation | self | `product` | model |  |
| `_unlink_except_used_as_rule_base` | internal rule | self | `product` | ondelete |  |
| `action_open_pricelist_report` | user action | self | `product` | readonly |  |
| `_load_pos_data_domain` | internal rule | self, data, config | `point_of_sale` | model |  |
| `_load_pos_data_fields` | internal rule | self, config | `point_of_sale` | model |  |
| `_load_pos_data_read` | internal rule | self, records, config | `point_of_sale` | model |  |
| `_default_website` | preparation rule | self | `website_sale` |  | Find the first company's website, if there is one. |
| `_check_websites_in_company` | validation | self | `website_sale` | constrains: `company_id`, `website_id` | Prevent misconfiguration multi-website/multi-companies.  If the record has a company, the website should be from that company. |
| `create` | lifecycle override | self, vals_list | `website_sale` | model_create_multi |  |
| `unlink` | lifecycle override | self | `website_sale` |  |  |
| `_is_available_on_website` | internal rule | self, website | `website_sale` |  | To be able to be used on a website, a pricelist should either: - Have its `website_id` set to current website (specific pricelist). - Have no `website_id` set and should be `selectable` (generic pricelist)   or should have a `code` (generic promotion). - Have no `company_id` or a `company_id` matching its website one.  Note: A pricelist without a website_id, not selectable and without a       code is a backend pricelist.  Change in this method should be reflected in `_get_website_pricelists_domain`. |
| `_is_available_in_country` | internal rule | self, country_code | `website_sale` |  |  |
| `_get_website_pricelists_domain` | preparation rule | self, website | `website_sale` |  | Check above `_is_available_on_website` for explanation. Change in this method should be reflected in `_is_available_on_website`. |
| `action_archive` | lifecycle override | self | `loyalty` |  |  |
| `_compute_partners_count` | computation | self | `partnership` |  |  |

## Validation and error messages (3)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_unlink_except_used_as_rule_base` | UserError | You cannot delete pricelist(s): (%(pricelists)s) They are used within pricelist(s): %(other_pricelists)s | `product` |
| `_check_websites_in_company` | ValidationError | Only the company's websites are allowed. Leave the Company field empty or select a website from that company. | `website_sale` |
| `action_archive` | UserError | This pricelist may not be archived. It is being used for active promotion programs: %s | `loyalty` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_pos_manager` | yes | yes | yes | yes | `point_of_sale` |
| `group_pos_user` | no | yes | no | no | `point_of_sale` |
| `base.group_user` | no | yes | no | no | `product` |
| `base.group_partner_manager` | no | yes | no | no | `product` |
| `group_product_manager` | yes | yes | yes | yes | `product` |
| `sales_team.group_sale_salesman` | no | yes | no | no | `sale` |
| `sales_team.group_sale_manager` | yes | yes | yes | yes | `sale` |
| `stock.group_stock_manager` | yes | yes | yes | yes | `stock` |
| `base.group_public` | no | yes | no | no | `website_sale` |
| `base.group_portal` | no | yes | no | no | `website_sale` |
| `base.group_user` | no | yes | no | no | `website_sale` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| product pricelist company rule | global (all users) | `['\|', ('company_id', 'parent_of', company_ids), ('company_id', '=', False)]` | True | True | True | True |
| product pricelist company rule | global (all users) | `['\|', ('company_id', 'in', [False, website.company_id.id]), ('company_id', 'in', company_ids)]` | True | True | True | True |

## Views (7)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `partnership.view_partner_pricelist_form` | sheet | `product.product_pricelist_view` | `partners_count`, `partners_label` | `partnership.action_pricelist_partners` |  | `partnership` |
| `product.product_pricelist_view_search` | search |  | `name`, `currency_id` |  | `Archived` | `product` |
| `product.product_pricelist_view_tree` | list |  | `sequence`, `name`, `country_group_ids`, `currency_id`, `company_id` |  |  | `product` |
| `product.product_pricelist_view_kanban` | kanban |  | `name`, `currency_id` |  |  | `product` |
| `product.product_pricelist_view` | form |  | `name`, `currency_id`, `active`, `company_id`, `country_group_ids`, `item_ids`, `product_tmpl_id`, `name`, `price`, `min_quantity`, `date_start`, `date_end`, `base`, `price_discount`, `applied_on`, `compute_price` | `Print` |  | `product` |
| `website_sale.website_sale_pricelist_form_view` | notebook | `product.product_pricelist_view` | `company_id`, `website_id`, `selectable`, `code` |  |  | `website_sale` |
| `website_sale.website_sale_pricelist_tree_view` | field | `product.product_pricelist_view_tree` | `currency_id`, `selectable`, `website_id` |  |  | `website_sale` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `product.product_pricelist_action2` | Pricelists | list,kanban,form |  | `{"default_base":'list_price'}` |  | `product` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `point_of_sale.pos_config_menu_action_product_pricelist` |  | `point_of_sale.pos_config_menu_catalog` | `product.product_pricelist_action2` | 20 | `product.group_product_pricelist` |
| `sale.menu_product_pricelist_main` | Pricelists |  | `product.product_pricelist_action2` | 30 | `product.group_product_pricelist` |
| `website_sale.menu_catalog_pricelists` | Pricelists |  | `product.product_pricelist_action2` | 3 | `product.group_product_pricelist` |

Machine-readable definition: `../../../schemas/data/entities/product.pricelist.json`; views: `../../../schemas/interfaces/views/product.pricelist.json`.
