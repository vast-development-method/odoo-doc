# Visited Pages (`website.track`)

**Transport name:** `website.track`  
**Storage name:** `website_track`  
**Kind:** persistent entity (one table)  
**Defined by package:** `website`  
**Extended by packages:** `website_sale`

Description: Visited Pages

## Identity and behavior

- Default ordering: `visit_datetime DESC`

## Fields (5)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `visitor_id` | Visitor | many to one | `website.visitor` | required; read only; indexed; on delete of the target: cascade |
| `page_id` | Page | many to one | `website.page` | read only; indexed; on delete of the target: cascade |
| `url` | Url | multi line text |  | indexed |
| `visit_datetime` | Visit Date | date and time |  | required; read only; default computed dynamically (fields.Datetime.now) |
| `product_id` | Product | many to one | `product.product` | read only; indexed (btree_not_null); on delete of the target: cascade |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `website.group_website_designer` | yes | yes | yes | yes | `website` |
| `base.group_system` | yes | yes | yes | yes | `website` |
| `sales_team.group_sale_salesman` | no | yes | no | no | `website_crm` |
| `im_livechat.im_livechat_group_user` | no | yes | no | no | `website_livechat` |

## Views (10)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `website.website_visitor_page_view_tree` | list |  | `visitor_id`, `page_id`, `url`, `visit_datetime` |  |  | `website` |
| `website.website_visitor_page_view_graph` | graph |  | `url` |  |  | `website` |
| `website.website_visitor_page_view_search` | search |  | `visitor_id`, `page_id`, `url`, `visit_datetime` |  | `Pages`, `Urls & Pages`, `Visitor`, `Page`, `Url`, `Date` | `website` |
| `website.website_visitor_track_view_tree` | list |  | `visitor_id`, `page_id`, `url`, `visit_datetime` |  |  | `website` |
| `website.website_visitor_track_view_graph` | graph |  | `url` |  |  | `website` |
| `website_sale.website_sale_visitor_page_view_tree` | list |  | `visitor_id`, `product_id`, `visit_datetime` |  |  | `website_sale` |
| `website_sale.website_sale_visitor_page_view_graph` | graph |  | `product_id` |  |  | `website_sale` |
| `website_sale.website_sale_visitor_page_view_search` | field | `website.website_visitor_page_view_search` | `url`, `product_id` |  |  | `website_sale` |
| `website_sale.website_sale_visitor_track_view_tree` | field | `website.website_visitor_track_view_tree` | `url`, `product_id` |  |  | `website_sale` |
| `website_sale.website_sale_visitor_track_view_graph` | field | `website.website_visitor_track_view_graph` | `url`, `product_id` |  |  | `website_sale` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `website.website_visitor_page_action` | Page Views History | list | `[('visitor_id', '=', active_id), ('url', '!=', False)]` |  |  | `website` |
| `website.website_visitor_view_action` | Page Views | list |  | `{'search_default_type_url': 1, 'create': False, 'edit': False, 'copy': False}` |  | `website` |
| `website_sale.website_sale_visitor_product_action` | Product Views History | list | `[('visitor_id', '=', active_id), ('product_id', '!=', False)]` |  |  | `website_sale` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `website.menu_visitor_view_menu` | Page Views | `website.menu_reporting` | `website.website_visitor_view_action` | 50 |  |

Machine-readable definition: `../../../schemas/data/entities/website.track.json`; views: `../../../schemas/interfaces/views/website.track.json`.
