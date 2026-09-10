# Theme Utils (`theme.utils`)

**Transport name:** `theme.utils`  
**Storage name:** `theme_utils`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `website`  
**Extended by packages:** `website_sale`

Description: Theme Utils

## Operations (9)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_post_copy` | internal rule | self, mod | `website` |  |  |
| `_reset_default_config` | internal rule | self | `website` | model |  |
| `_toggle_asset` | internal rule | self, key, active | `website` | model |  |
| `_toggle_view` | internal rule | self, xml_id, active | `website` | model |  |
| `enable_asset` | operation | self, name | `website` | model |  |
| `disable_asset` | operation | self, name | `website` | model |  |
| `enable_view` | operation | self, xml_id | `website_sale`, `website` | model | Override of `theme.utils` to disable all category style templates when enabling one. |
| `disable_view` | operation | self, xml_id | `website` | model |  |
| `_footer_templates` | internal rule | self | `website_sale` |  |  |

Machine-readable definition: `../../../schemas/data/entities/theme.utils.json`.
