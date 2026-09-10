# Website rewrite (`website.rewrite`)

**Transport name:** `website.rewrite`  
**Storage name:** `website_rewrite`  
**Kind:** persistent entity (one table)  
**Defined by package:** `website`

Description: Website rewrite

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (8)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required |
| `website_id` | Website | many to one | `website` | indexed; on delete of the target: cascade |
| `active` | Active | boolean |  | default `True` |
| `url_from` | uniform resource locator from | single line text |  | indexed |
| `route_id` | Route | many to one | `website.route` |  |
| `url_to` | uniform resource locator to | single line text |  |  |
| `redirect_type` | Action | selection |  | default `302`; Help: Type of redirect/Rewrite:          301 Moved permanently: The browser will keep in cache the new url.         302 Moved temporarily: The browser will not keep in cache the new url and ask again the next time the new url.         404 Not Found: If you want remove a specific page/controller (e.g. Ecommerce is installed, but you don't want /shop on a specific website)         308 Redirect / Rewrite: If you want rename a controller with a new url. (Eg: /shop -> /garden - Both url will be accessible but /shop will automatically be redirected to /garden) |
| `sequence` | Sequence | integer |  |  |

## Selection values

### `redirect_type` (Action)

| Value | Label |
|---|---|
| `404` | 404 Not Found |
| `301` | 301 Moved permanently |
| `302` | 302 Moved temporarily |
| `308` | 308 Redirect / Rewrite |

## Operations (9)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_onchange_route_id` | on change | self | `website` | onchange: `route_id` |  |
| `_check_url_to` | validation | self | `website` | constrains: `url_to`, `url_from`, `redirect_type` |  |
| `_compute_display_name` | computation | self | `website` | depends: `redirect_type` |  |
| `create` | lifecycle override | self, vals_list | `website` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `website` |  |  |
| `unlink` | lifecycle override | self | `website` |  |  |
| `_invalidate_routing` | internal rule | self | `website` |  |  |
| `refresh_routes` | operation | self | `website` |  |  |
| `get_import_templates` | operation | self | `website` | model |  |

## Validation and error messages (10)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_url_to` | ValidationError | "URL to" can not be empty. | `website` |
| `_check_url_to` | ValidationError | "URL from" can not be empty. | `website` |
| `_check_url_to` | ValidationError | URL must not start with '#'. | `website` |
| `_check_url_to` | ValidationError | base URL of 'URL to' should not be same as 'URL from'. | `website` |
| `_check_url_to` | ValidationError | "URL to" must start with a leading slash. | `website` |
| `_check_url_to` | ValidationError | "URL to" cannot be set to "/". To change the homepage content, use the "Homepage URL" field in the website settings or the page properties on any custom page. | `website` |
| `_check_url_to` | ValidationError | "URL to" cannot be set to an existing page. | `website` |
| `_check_url_to` | ValidationError | "URL to" must contain parameter %s used in "URL from". | `website` |
| `_check_url_to` | ValidationError | "URL to" cannot contain parameter %s which is not used in "URL from". | `website` |
| `_check_url_to` | ValidationError | "URL to" is invalid: %s | `website` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| all internal users | no | no | no | no | `website` |
| `group_website_designer` | yes | yes | yes | yes | `website` |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `website.view_website_rewrite_form` | form |  | `name`, `redirect_type`, `url_from`, `route_id`, `url_to`, `website_id`, `active`, `sequence` | `Refresh route's list` |  | `website` |
| `website.action_website_rewrite_tree` | list |  | `sequence`, `redirect_type`, `name`, `url_from`, `url_to`, `website_id`, `active`, `create_date`, `write_date`, `create_uid`, `write_uid` |  |  | `website` |
| `website.view_rewrite_search` | search |  | `url_from`, `url_to` |  | `404 Not Found`, `301 Moved permanently`, `302 Moved temporarily`, `308 Redirect / Rewrite`, `Archived`, `Redirection Type`, `Created by` | `website` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `website.action_website_rewrite_list` | Rewrite |  |  |  |  | `website` |

Machine-readable definition: `../../../schemas/data/entities/website.rewrite.json`; views: `../../../schemas/interfaces/views/website.rewrite.json`.
