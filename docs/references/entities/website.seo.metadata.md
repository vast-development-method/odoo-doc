# search engine optimization metadata (`website.seo.metadata`)

**Transport name:** `website.seo.metadata`  
**Storage name:** `website_seo_metadata`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `website`

Description: SEO metadata

## Fields (6)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `is_seo_optimized` | search engine optimization optimized | boolean |  | computed by rule `_compute_is_seo_optimized` and stored |
| `website_meta_title` | Website meta title | single line text |  | translatable |
| `website_meta_description` | Website meta description | multi line text |  | translatable |
| `website_meta_keywords` | Website meta keywords | single line text |  | translatable |
| `website_meta_og_img` | Website opengraph image | single line text |  |  |
| `seo_name` | Seo name | single line text |  | translatable |

## Operations (3)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_is_seo_optimized` | computation | self | `website` | depends: `website_meta_title`, `website_meta_description`, `website_meta_keywords` |  |
| `_default_website_meta` | preparation rule | self | `website` |  | This method will return default meta information. It return the dict contains meta property as a key and meta content as a value. e.g. 'og:type': 'website'.  Override this method in case you want to change default value from any model. e.g. change value of og:image to product specific images instead of default images |
| `get_website_meta` | operation | self | `website` |  | This method will return final meta information. It will replace default values with user's custom value (if user modified it from the seo popup of frontend)  This method is not meant for overridden. To customize meta values override `_default_website_meta` method instead of this method. This method only replaces user custom values in defaults. |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_public` | no | yes | no | no | `website` |
| `base.group_portal` | no | yes | no | no | `website` |
| `base.group_user` | no | yes | no | no | `website` |
| `group_website_designer` | yes | yes | yes | yes | `website` |

Machine-readable definition: `../../../schemas/data/entities/website.seo.metadata.json`.
