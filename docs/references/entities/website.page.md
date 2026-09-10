# Page (`website.page`)

**Transport name:** `website.page`  
**Storage name:** `website_page`  
**Kind:** persistent entity (one table)  
**Defined by package:** `website`  
**Extended by packages:** `website_sale`, `website_livechat`, `website_hr_recruitment`, `website_project`

Description: Page

## Identity and behavior

- Mixins (classical inheritance): `website.published.multi.mixin`, `website.searchable.mixin`, `website.page_options.mixin`
- Delegation inheritance: embeds `ir.ui.view` through field `view_id`
- Default ordering: `website_id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (14)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `url` | Page uniform resource locator | single line text |  | required |
| `view_id` | View | many to one | `ir.ui.view` | required; indexed; on delete of the target: cascade |
| `view_write_uid` | Last Content Update by | many to one | `res.users` | related through path `view_id.write_uid` |
| `view_write_date` | Last Content Update on | date and time |  | related through path `view_id.write_date` |
| `website_indexed` | Is Indexed | boolean |  | default `True` |
| `date_publish` | Publishing Date | date and time |  |  |
| `menu_ids` | Related Menus | one to many | `website.menu` | inverse field `page_id` |
| `is_in_menu` | Is In Menu | boolean |  | computed by rule `_compute_website_menu` (not stored) |
| `is_homepage` | Homepage | boolean |  | computed by rule `_compute_is_homepage` (not stored) |
| `is_visible` | Is Visible | boolean |  | computed by rule `_compute_visible` (not stored) |
| `is_new_page_template` | New Page Template | boolean |  | Help: Add this page to the "+New" page templates. It will be added to the "Custom" category. |
| `website_id` | Website | many to one |  | related through path `view_id.website_id` and stored; on delete of the target: cascade |
| `arch` | Arch | multi line text |  | related through path `view_id.arch` |
| `theme_template_id` | Theme Template | many to one | `theme.website.page` | indexed (btree_not_null); not copied on duplication |

## Operations (23)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_is_homepage` | computation | self | `website` |  |  |
| `_compute_visible` | computation | self | `website` |  |  |
| `_compute_website_menu` | computation | self | `website` | depends: `menu_ids` |  |
| `_compute_website_url` | computation | self | `website` | depends: `url` |  |
| `_compute_can_publish` | computation | self | `website` | depends_context: `uid` |  |
| `_get_most_specific_pages` | preparation rule | self | `website` |  | Returns the most specific pages in self. |
| `copy_data` | lifecycle override | self, default | `website` |  |  |
| `clone_page` | operation | self, page_id, page_name, clone_menu | `website` | model | Clone a page, given its identifier :param page_id : website.page identifier |
| `unlink` | lifecycle override | self | `website` |  |  |
| `write` | lifecycle override | self, vals | `website` |  |  |
| `get_website_meta` | operation | self | `website` |  |  |
| `_search_get_detail` | search rule | self, website, order, options | `website` | model |  |
| `_search_fetch` | search rule | self, search_detail, search, limit, order | `website` | model |  |
| `action_page_debug_view` | user action | self | `website` |  |  |
| `open_website_url` | operation | self | `website` |  |  |
| `_allow_to_use_cache` | internal rule | self, request | `website_hr_recruitment`, `website_project`, `website` | model | Checks if the generated HTML content is eligible for caching. This is useful for preventing sensitive or dynamic content from being stored. |
| `_allow_cache_insertion` | internal rule | self, layout | `website_sale`, `website` | model | Determines whether a page is allowed to be served from the cache based on the current request, URL, or session. |
| `_post_process_response_from_cache` | internal rule | self, request, response | `website_livechat`, `website_sale`, `website` | model | A hook called after a response is retrieved from the cache. This method allows for post-processing, such as incrementing counters or modifying HTTP headers, without regenerating the entire page. |
| `_get_cache_key` | preparation rule | self, request | `website` | model | Allows for supplementing the base cache key with custom components (e.g., from the URL or session). This is essential for ensuring that the cache serves the correct version of a page based on specific parameters like user language or currency. |
| `_get_response` | preparation rule | self, request | `website` |  | Returns the response corresponding to the request. The response may or may not come from the cache.  The management and logic for this cache are placed directly on the `website.page` model, as it is the source of the HTML response for these records. This approach makes it easy to control caching behavior through a few new, overridable methods: -  `_allow_to_use_cache` -  `_allow_cache_insertion` -  `_post_process_response_from_cache` - `_get_cache_key`  The cache is an ORM cache from `_get_response_cached` with an added mechanism to update its values. After a certain period, the system will fe |
| `_get_response_cached` | preparation rule | self, request | `website` |  | Returns the response corresponding to the request. If the response exists and `_allow_cache_insertio` return True, this response is cached. |
| `_get_response_raw` | preparation rule | self, request | `website` |  | Returns the raw response associated with the current request. This method is called by `_get_response_cached`, which handles caching the result. It is also called directly by `_get_response` if `_allow_to_use_cache` returns False. |
| `_get_page_info` | preparation rule | self, request | `website` | model |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_website_designer` | yes | yes | yes | yes | `website` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| website.page: portal/public: read published pages | `[(4, ref('base.group_portal')), (4, ref('base.group_public'))]` | `[('website_published', '=', True)]` | True | True | True | True |

## Views (4)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `website.website_pages_form_view` | form |  | `name`, `url`, `view_id`, `website_id`, `track`, `website_indexed`, `is_published`, `date_publish`, `menu_ids` |  |  | `website` |
| `website.website_pages_tree_view` | list |  | `is_homepage`, `name`, `website_url`, `website_indexed`, `is_in_menu`, `is_seo_optimized`, `is_published`, `create_uid`, `create_date`, `view_write_uid`, `view_write_date`, `write_uid`, `write_date`, `track`, `website_id` | `action_page_debug_view` |  | `website` |
| `website.website_pages_kanban_view` | kanban |  | `is_homepage`, `name`, `website_id`, `website_url`, `is_in_menu`, `is_seo_optimized`, `is_published` | `action_page_debug_view` |  | `website` |
| `website.website_pages_view_search` | search |  | `url` |  | `Published`, `Not published`, `Tracked`, `Not tracked`, `Not SEO optimized` | `website` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `website.action_website_pages_list` | Website Pages | list,kanban |  |  |  | `website` |

Machine-readable definition: `../../../schemas/data/entities/website.page.json`; views: `../../../schemas/interfaces/views/website.page.json`.
