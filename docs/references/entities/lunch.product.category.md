# Lunch Product Category (`lunch.product.category`)

**Transport name:** `lunch.product.category`  
**Storage name:** `lunch_product_category`  
**Kind:** persistent entity (one table)  
**Defined by package:** `lunch`

Description: Lunch Product Category

## Identity and behavior

- Mixins (classical inheritance): `image.mixin`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (6)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Product Category | single line text |  | required; translatable |
| `company_id` | Company | many to one | `res.company` |  |
| `currency_id` | Currency | many to one | `res.currency` | related through path `company_id.currency_id` |
| `product_count` | Product Count | integer |  | computed by rule `_compute_product_count` (not stored); Help: The number of products related to this category |
| `active` | Active | boolean |  | default `True` |
| `image_1920` | Image 1920 | image |  | default computed dynamically (_default_image) |

## Operations (5)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_default_image` | preparation rule | self | `lunch` | model |  |
| `_compute_product_count` | computation | self | `lunch` |  |  |
| `_sync_active_products` | internal rule | self | `lunch` |  | Archiving related lunch product |
| `action_archive` | lifecycle override | self | `lunch` |  |  |
| `action_unarchive` | lifecycle override | self | `lunch` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_lunch_user` | no | yes | no | no | `lunch` |
| `group_lunch_manager` | yes | yes | yes | yes | `lunch` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Lunch product category: Multi Company | global (all users) | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |

## Views (4)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `lunch.lunch_product_category_view_tree` | list |  | `name`, `company_id` |  |  | `lunch` |
| `lunch.lunch_product_category_view_form` | form |  | `product_count`, `image_1920`, `name`, `company_id` | `%(lunch.lunch_product_action_statbutton)d` |  | `lunch` |
| `lunch.lunch_product_category_view_kanban` | kanban |  | `image_128`, `product_count`, `name`, `company_id` | `%(lunch.lunch_product_action_statbutton)d` |  | `lunch` |
| `lunch.lunch_product_category_view_search` | search |  | `name` |  | `Archived` | `lunch` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `lunch.lunch_product_category_action` | Product Categories | list,form,kanban |  |  |  | `lunch` |

Machine-readable definition: `../../../schemas/data/entities/lunch.product.category.json`; views: `../../../schemas/interfaces/views/lunch.product.category.json`.
