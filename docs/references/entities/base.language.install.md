# Install Language (`base.language.install`)

**Transport name:** `base.language.install`  
**Storage name:** `base_language_install`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `base`  
**Extended by packages:** `website`

Description: Install Language

## Fields (4)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `lang_ids` | Languages | many to many | `res.lang` | required; default computed dynamically (_default_lang_ids); association table `res_lang_install_rel` |
| `overwrite` | Overwrite Existing Terms | boolean |  | default `True`; Help: If you check this box, your customized translations will be overwritten and replaced by the official ones. |
| `first_lang_id` | First Lang | many to one | `res.lang` | computed by rule `_compute_first_lang_id` (not stored); Help: Used when the user only selects one language and is given the option to switch to it |
| `website_ids` | Websites to translate | many to many | `website` |  |

## Operations (6)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_default_lang_ids` | preparation rule | self | `base` | model | Display the selected language when using the 'Update Terms' action from the language list view |
| `_compute_first_lang_id` | computation | self | `base` |  |  |
| `lang_install` | operation | self | `base`, `website` |  |  |
| `reload` | operation | self | `base` |  |  |
| `switch_lang` | operation | self | `base` |  |  |
| `default_get` | lifecycle override | self, fields | `website` | model |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_system` | yes | yes | yes | no | `base` |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `base.language_install_view_form_lang_switch` | form |  | `first_lang_id`, `first_lang_id` | `Close`, `switch_lang` |  | `base` |
| `base.view_base_language_install` | form |  | `lang_ids`, `overwrite` | `Add`, `Cancel` |  | `base` |
| `website.view_base_language_install` | group | `base.view_base_language_install` | `website_ids` |  |  | `website` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `base.action_view_base_language_install` | Add Languages | form |  |  | new | `base` |

Machine-readable definition: `../../../schemas/data/entities/base.language.install.json`; views: `../../../schemas/interfaces/views/base.language.install.json`.
