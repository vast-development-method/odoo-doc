# Lunch Product (`lunch.product`)

**Transport name:** `lunch.product`  
**Storage name:** `lunch_product`  
**Kind:** persistent entity (one table)  
**Defined by package:** `lunch`

Description: Lunch Product

## Identity and behavior

- Mixins (classical inheritance): `image.mixin`
- Default ordering: `name`
- Company consistency is checked automatically on company-bound relations
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (15)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Product Name | single line text |  | required; translatable |
| `category_id` | Product Category | many to one | `lunch.product.category` | required; must belong to the same company |
| `description` | Description | rich text |  | translatable |
| `price` | Price | float |  | required; precision `Account` |
| `supplier_id` | Vendor | many to one | `lunch.supplier` | required; must belong to the same company |
| `active` | Active | boolean |  | default `True` |
| `company_id` | Company | many to one | `res.company` | related through path `supplier_id.company_id` and stored |
| `currency_id` | Currency | many to one | `res.currency` | related through path `company_id.currency_id` |
| `new_until` | New Until | date |  |  |
| `is_new` | Is New | boolean |  | computed by rule `_compute_is_new` (not stored) |
| `favorite_user_ids` | Favorite User | many to many | `res.users` | must belong to the same company; association table `lunch_product_favorite_user_rel` |
| `is_favorite` | Is Favorite | boolean |  | computed by rule `_compute_is_favorite` (not stored); writable through an inverse rule |
| `last_order_date` | Last Order Date | date |  | computed by rule `_compute_last_order_date` (not stored) |
| `product_image` | Product Image | image |  | computed by rule `_compute_product_image` (not stored) |
| `is_available_at` | Product Availability | many to one | `lunch.location` | computed by rule `_compute_is_available_at` (not stored); searchable through a search rule |

## Operations (11)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_product_image` | computation | self | `lunch` | depends: `image_128`, `category_id.image_128` |  |
| `_compute_is_new` | computation | self | `lunch` | depends: `new_until` |  |
| `_compute_is_favorite` | computation | self | `lunch` | depends_context: `uid`; depends: `favorite_user_ids` |  |
| `_compute_last_order_date` | computation | self | `lunch` | depends_context: `uid` |  |
| `_compute_is_available_at` | computation | self | `lunch` |  | Is available_at is always false when browsing it this field is there only to search (see _search_is_available_at) |
| `_search_is_available_at` | search rule | self, operator, value | `lunch` |  |  |
| `_sync_active_from_related` | internal rule | self | `lunch` |  | Archive/unarchive product after related field is archived/unarchived |
| `_check_active_categories` | validation | self | `lunch` | constrains: `active`, `category_id` |  |
| `_check_active_suppliers` | validation | self | `lunch` | constrains: `active`, `supplier_id` |  |
| `_inverse_is_favorite` | inverse computation | self | `lunch` |  | Handled in the write() |
| `write` | lifecycle override | self, vals | `lunch` |  |  |

## Validation and error messages (2)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_active_categories` | UserError | The following product categories are archived. You should either unarchive the categories or change the category of the product. %s | `lunch` |
| `_check_active_suppliers` | UserError | The following suppliers are archived. You should either unarchive the suppliers or change the supplier of the product. %s | `lunch` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_lunch_user` | no | yes | no | no | `lunch` |
| `group_lunch_manager` | yes | yes | yes | yes | `lunch` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Lunch product: Multi Company | global (all users) | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |

## Views (6)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `lunch.lunch_product_view_search` | search |  | `name`, `category_id`, `supplier_id`, `description`, `category_id`, `supplier_id` |  | `Available Today`, `Monday`, `Tuesday`, `Wednesday`, `Thursday`, `Friday`, `Saturday`, `Sunday`, `Archived`, `Vendor`, `Category` | `lunch` |
| `lunch.lunch_product_view_tree` | list |  | `currency_id`, `name`, `category_id`, `supplier_id`, `company_id`, `description`, `price` |  |  | `lunch` |
| `lunch.lunch_product_view_tree_order` | xpath | `lunch_product_view_tree` |  |  |  | `lunch` |
| `lunch.lunch_product_view_form` | form |  | `company_id`, `currency_id`, `image_1920`, `name`, `active`, `category_id`, `supplier_id`, `price`, `new_until`, `company_id`, `description` |  |  | `lunch` |
| `lunch.view_lunch_product_kanban_order` | kanban |  | `currency_id`, `is_new`, `image_128`, `is_favorite`, `name`, `price`, `supplier_id`, `description` |  |  | `lunch` |
| `lunch.view_lunch_product_kanban` | kanban |  | `id`, `name`, `category_id`, `supplier_id`, `description`, `currency_id`, `image_128`, `name`, `price`, `supplier_id`, `description` |  |  | `lunch` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `lunch.lunch_product_action_statbutton` | Products | kanban,list,form |  | `{'search_default_group_by_supplier': 1}` |  | `lunch` |
| `lunch.lunch_product_action` | Products | list,kanban,form |  |  |  | `lunch` |
| `lunch.lunch_product_action_order` | Order Your Lunch | kanban,list | `[]` |  |  | `lunch` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `lunch.lunch_order_menu_form` | New Order |  | `lunch.lunch_product_action_order` | 1 |  |

Machine-readable definition: `../../../schemas/data/entities/lunch.product.json`; views: `../../../schemas/interfaces/views/lunch.product.json`.
