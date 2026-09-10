# Website Configurator Feature (`website.configurator.feature`)

**Transport name:** `website.configurator.feature`  
**Storage name:** `website_configurator_feature`  
**Kind:** persistent entity (one table)  
**Defined by package:** `website`

Description: Website Configurator Feature

## Identity and behavior

- Default ordering: `sequence`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (11)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `sequence` | Sequence | integer |  |  |
| `name` | Name | single line text |  | translatable |
| `description` | Description | single line text |  | translatable |
| `icon` | Icon | single line text |  |  |
| `iap_page_code` | In-app purchase Page Code | single line text |  | Help: Page code used to tell IAP website_service for which page a snippet list should be generated |
| `website_config_preselection` | Website Config Preselection | single line text |  | Help: Comma-separated list of website type/purpose for which this feature should be pre-selected |
| `page_view_id` | Page View | many to one | `ir.ui.view` | on delete of the target: cascade |
| `module_id` | Module | many to one | `ir.module.module` | on delete of the target: cascade |
| `feature_url` | Feature Uniform resource locator | single line text |  |  |
| `menu_sequence` | Menu Sequence | integer |  | Help: If set, a website menu will be created for the feature. |
| `menu_company` | Menu Company | boolean |  | Help: If set, add the menu as a second level menu, as a child of "Company" menu. |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_check_module_xor_page_view` | validation | self | `website` | constrains: `module_id`, `page_view_id` |  |
| `_process_svg` | background operation | theme, colors, image_mapping | `website` |  |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_module_xor_page_view` | ValidationError | One and only one of the two fields 'page_view_id' and 'module_id' should be set | `website` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `website.group_website_designer` | yes | yes | yes | yes | `website` |

Machine-readable definition: `../../../schemas/data/entities/website.configurator.feature.json`.
