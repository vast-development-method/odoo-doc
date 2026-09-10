# Page Properties (`website.page.properties`)

**Transport name:** `website.page.properties`  
**Storage name:** `website_page_properties`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `website`

Description: Page Properties

## Identity and behavior

- Mixins (classical inheritance): `website.page.properties.base`

## Fields (12)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `target_model_id` | Target Model | many to one | `website.page` |  |
| `name` | Name | single line text |  | related through path `target_model_id.name` |
| `url` | Uniform resource locator | single line text |  | related through path `target_model_id.url` |
| `date_publish` | Date Publish | date and time |  | related through path `target_model_id.date_publish` |
| `website_indexed` | Website Indexed | boolean |  | related through path `target_model_id.website_indexed` |
| `visibility` | Visibility | selection |  | related through path `target_model_id.visibility` |
| `visibility_password_display` | Visibility Password Display | single line text |  | related through path `target_model_id.visibility_password_display` |
| `group_ids` | Group | many to many |  | related through path `target_model_id.group_ids` |
| `is_new_page_template` | Is New Page Template | boolean |  | related through path `target_model_id.is_new_page_template` |
| `old_url` | Old Uniform resource locator | single line text |  |  |
| `redirect_old_url` | Redirect Old Uniform resource locator | boolean |  | default  |
| `redirect_type` | Redirect Type | selection |  | required; default `301` |

## Selection values

### `redirect_type` (Redirect Type)

| Value | Label |
|---|---|
| `301` | 301 Moved permanently |
| `302` | 302 Moved temporarily |

## Operations (3)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_is_homepage` | computation | self | `website` | depends: `url`, `website_id.homepage_url` | Don't match is_homepage when url is '/' as this model's url is not the accessed route's url. |
| `create` | lifecycle override | self, vals_list | `website` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `website` |  |  |

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
| `website.website_page_properties_view_form` | xpath | `website_page_properties_base_view_form` | `name`, `website_id`, `url`, `redirect_old_url`, `redirect_type` |  |  | `website` |

Machine-readable definition: `../../../schemas/data/entities/website.page.properties.json`; views: `../../../schemas/interfaces/views/website.page.properties.json`.
