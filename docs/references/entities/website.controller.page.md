# Model Page (`website.controller.page`)

**Transport name:** `website.controller.page`  
**Storage name:** `website_controller_page`  
**Kind:** persistent entity (one table)  
**Defined by package:** `website`

Description: Model Page

## Identity and behavior

- Mixins (classical inheritance): `website.published.multi.mixin`, `website.searchable.mixin`
- Delegation inheritance: embeds `ir.ui.view` through field `view_id`
- Default ordering: `website_id, id DESC`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (9)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `view_id` | Listing view | many to one | `ir.ui.view` | required; indexed; on delete of the target: cascade |
| `record_view_id` | Record view | many to one | `ir.ui.view` | on delete of the target: cascade |
| `menu_ids` | Related Menus | one to many | `website.menu` | inverse field `controller_page_id` |
| `website_id` | Website | many to one |  | related through path `view_id.website_id` and stored; on delete of the target: cascade |
| `name` | The name is used to generate the uniform resource locator and is shown in the browser title bar | single line text |  | required; computed by rule `_compute_name` and stored; writable through an inverse rule; precomputed before insertion |
| `name_slugified` | uniform resource locator | single line text |  | computed by rule `_compute_name_slugified` and stored; writable through an inverse rule; precomputed before insertion; Help: The name of the page usable in a URL |
| `url_demo` | Demo uniform resource locator | single line text |  | computed by rule `_compute_url_demo` (not stored) |
| `record_domain` | Domain | single line text |  | Help: Domain to restrict records that can be viewed publicly |
| `default_layout` | Default Layout | selection |  | default `grid` |

## Selection values

### `default_layout` (Default Layout)

| Value | Label |
|---|---|
| `grid` | Grid |
| `list` | List |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_unique_name_slugified` | Constraint | `UNIQUE(name_slugified)` | url should be unique | `website` |

## Operations (11)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_check_user_has_model_access` | validation | self | `website` |  |  |
| `_compute_name` | computation | self | `website` | depends: `view_id` |  |
| `_inverse_name` | inverse computation | self | `website` |  |  |
| `_compute_name_slugified` | computation | self | `website` | depends: `model_id`, `name` |  |
| `_inverse_name_slugified` | inverse computation | self | `website` |  |  |
| `_compute_url_demo` | computation | self | `website` | depends: `name_slugified` |  |
| `_default_is_published` | preparation rule | self | `website` |  |  |
| `create` | lifecycle override | self, vals_list | `website` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `website` |  |  |
| `unlink` | lifecycle override | self | `website` |  |  |
| `open_website_url` | operation | self | `website` |  |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_user_has_model_access` | ValidationError | A page must be set to display a concrete model. | `website` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_website_designer` | yes | yes | yes | yes | `website` |
| `website_page_controller_expose` | no | yes | no | no | `website` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| website.controller.page: portal/public: read published pages | `[(4, ref('website.website_page_controller_expose'))]` | `[('website_published', '=', True)]` | True | True | True | True |

## Views (4)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `website.website_controller_pages_form_view` | form |  | `website_id`, `name`, `name_slugified`, `website_published`, `model_id`, `model`, `record_domain`, `default_layout`, `view_id`, `name`, `menu_ids` |  |  | `website` |
| `website.website_controller_pages_tree_view` | list |  | `name`, `website_id`, `website_id`, `url_demo`, `is_published` |  |  | `website` |
| `website.website_controller_pages_kanban_view` | kanban |  | `website_published`, `name`, `website_id`, `url_demo` |  |  | `website` |
| `website.website_controller_pages_search_view` | search |  | `model` |  | `Model`, `Website` | `website` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `website.action_website_controller_pages_list` | Website Model Pages | list,kanban,form |  |  |  | `website` |

Machine-readable definition: `../../../schemas/data/entities/website.controller.page.json`; views: `../../../schemas/interfaces/views/website.controller.page.json`.
