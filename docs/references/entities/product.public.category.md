# Website Product Category (`product.public.category`)

**Transport name:** `product.public.category`  
**Storage name:** `product_public_category`  
**Kind:** persistent entity (one table)  
**Defined by package:** `website_sale`

Description: Website Product Category

## Identity and behavior

- Mixins (classical inheritance): `website.seo.metadata`, `website.multi.mixin`, `website.searchable.mixin`, `image.mixin`
- Default ordering: `sequence, name, id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (14)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required; translatable |
| `cover_image` | Cover Image | image |  | Help: Displayed only in the Category List Snippet. |
| `sequence` | Sequence | integer |  | default computed dynamically (_default_sequence); indexed |
| `parent_id` | Parent | many to one | `product.public.category` | indexed; on delete of the target: cascade |
| `child_id` | Children Categories | one to many | `product.public.category` | inverse field `parent_id` |
| `parent_path` | Parent Path | single line text |  | indexed |
| `parents_and_self` | Parents And Self | many to many | `product.public.category` | computed by rule `_compute_parents_and_self` (not stored) |
| `product_tmpl_ids` | Product Tmpl | many to many | `product.template` | association table `product_public_category_product_template_rel` |
| `has_published_products` | Has Published Products | boolean |  | computed by rule `_compute_has_published_products` (not stored); searchable through a search rule |
| `website_description` | Description | rich text |  | translatable |
| `website_footer` | Category Footer | rich text |  | translatable |
| `show_category_title` | Show Category Title | boolean |  | default ; Help: Display the category title on the shop page. Corresponds to the 'Show Title' editor option. |
| `show_category_description` | Show Category Description | boolean |  | default `True`; Help: Display the category description on the shop page. Corresponds to the 'Show Description' editor option. |
| `align_category_content` | Align Category Content | boolean |  | default ; Help: Align the category content on the shop page. Corresponds to the 'Center Content' editor option. |

## Operations (10)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_default_sequence` | preparation rule | self | `website_sale` |  |  |
| `_compute_parents_and_self` | computation | self | `website_sale` | depends: `parent_path` |  |
| `_compute_display_name` | computation | self | `website_sale` | depends: `parents_and_self` |  |
| `_compute_has_published_products` | computation | self | `website_sale` | depends_context: `company`, `website_id` |  |
| `check_parent_id` | validation | self | `website_sale` | constrains: `parent_id` |  |
| `_search_has_published_products` | search rule | self, operator, value | `website_sale` | model |  |
| `_search_get_detail` | search rule | self, website, order, options | `website_sale` | model |  |
| `_search_render_results` | search rule | self, fetch_fields, mapping, icon, limit | `website_sale` |  |  |
| `get_available_snippet_categories` | operation | self, website_id | `website_sale` | model | Return parent categories available for selection in the dynamic category snippet.  :param int website_id: ID of the current website :return: Available parent categories :rtype: list[dict] |
| `_get_available_category_domain` | preparation rule | self, website_id | `website_sale` | model | Build a search domain for product categories to be used in dynamic snippets.  :param int website_id: ID of the current website :return: A domain to filter product categories for the given website :rtype: Domain |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `sales_team.group_sale_manager` | yes | yes | yes | yes | `website_sale` |
| `website.group_website_designer` | yes | yes | yes | no | `website_sale` |
| `base.group_public` | no | yes | no | no | `website_sale` |
| `base.group_portal` | no | yes | no | no | `website_sale` |
| `base.group_user` | no | yes | no | no | `website_sale` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Hide empty eCommerce categories to public/portal users | `[             Command.link(ref('base.group_public')),             Command.link(ref('base.group_portal')),         ]` | `[('has_published_products', '=', True)]` | True | False | False | False |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `website_sale.product_public_category_form_view` | form |  | `image_1920`, `name`, `parent_id`, `website_id`, `sequence`, `website_description`, `cover_image` |  |  | `website_sale` |
| `website_sale.product_public_category_tree_view` | list |  | `sequence`, `display_name`, `website_id` |  |  | `website_sale` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `website_sale.product_public_category_action` | eCommerce Categories | list,form |  |  |  | `website_sale` |

## Server actions

| Action | Name | State | Binding | Custom logic |
|---|---|---|---|---|
| `website_sale.dynamic_snippet_category_list` | Category List | code |  | yes |

Machine-readable definition: `../../../schemas/data/entities/product.public.category.json`; views: `../../../schemas/interfaces/views/product.public.category.json`.
