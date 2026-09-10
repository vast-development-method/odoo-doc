# Page Properties Base (`website.page.properties.base`)

**Transport name:** `website.page.properties.base`  
**Storage name:** `website_page_properties_base`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `website`

Description: Page Properties Base

## Fields (8)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `target_model_id` | Target Model | reference |  | required; values provided by rule `_selection_target_model_id` |
| `website_id` | Website | many to one | `website` | required |
| `menu_ids` | Menu | one to many | `website.menu` | computed by rule `_compute_menu_ids` (not stored) |
| `is_in_menu` | Is In Menu | boolean |  | computed by rule `_compute_is_in_menu` (not stored); writable through an inverse rule |
| `url` | Uniform resource locator | single line text |  | required |
| `is_homepage` | Homepage | boolean |  | computed by rule `_compute_is_homepage` (not stored); writable through an inverse rule |
| `can_publish` | Can Publish | boolean |  | computed by rule `_compute_can_publish` (not stored) |
| `is_published` | Is Published | boolean |  | computed by rule `_compute_is_published` (not stored); writable through an inverse rule |

## Operations (13)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_selection_target_model_id` | internal rule | self | `website` |  |  |
| `_get_menu_domain` | preparation rule | self, url | `website` |  |  |
| `_compute_menu_ids` | computation | self | `website` | depends: `url`, `website_id` |  |
| `_compute_is_in_menu` | computation | self | `website` | depends: `menu_ids` |  |
| `_inverse_is_in_menu` | inverse computation | self | `website` |  |  |
| `_compute_is_homepage` | computation | self | `website` | depends: `url`, `website_id.homepage_url` |  |
| `_inverse_is_homepage` | inverse computation | self | `website` |  |  |
| `_compute_can_publish` | computation | self | `website` | depends: `target_model_id` |  |
| `_compute_is_published` | computation | self | `website` | depends: `target_model_id` |  |
| `_inverse_is_published` | inverse computation | self | `website` |  |  |
| `_get_ir_ui_view_unpublish_group` | preparation rule | self | `website` |  |  |
| `_is_ir_ui_view_unpublished` | internal rule | self, view | `website` |  |  |
| `_is_ir_ui_view_published` | internal rule | self, view | `website` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_public` | no | no | no | no | `website` |
| `base.group_portal` | no | no | no | no | `website` |
| `base.group_user` | no | yes | no | no | `website` |
| `group_website_designer` | yes | yes | yes | yes | `website` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `website.website_page_properties_base_view_form` | form |  | `is_in_menu`, `is_homepage`, `is_published` | `Edit Menu` |  | `website` |

Machine-readable definition: `../../../schemas/data/entities/website.page.properties.base.json`; views: `../../../schemas/interfaces/views/website.page.properties.base.json`.
