# Code Translation (`transifex.code.translation`)

**Transport name:** `transifex.code.translation`  
**Storage name:** `transifex_code_translation`  
**Kind:** persistent entity (one table)  
**Defined by package:** `transifex`

Description: Code Translation

## Fields (5)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `source` | Code | multi line text |  |  |
| `value` | Translation Value | multi line text |  |  |
| `module` | Module | single line text |  | Help: Module this term belongs to |
| `lang` | Language | selection |  | values provided by rule `_get_languages` |
| `transifex_url` | Transifex uniform resource locator | single line text |  | computed by rule `_compute_transifex_url` (not stored); Help: Propose a modification in the official version of the system |

## Operations (5)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_languages` | preparation rule | self | `transifex` |  |  |
| `_compute_transifex_url` | computation | self | `transifex` |  |  |
| `_load_code_translations` | internal rule | self, module_names, langs | `transifex` |  |  |
| `_open_code_translations` | internal rule | self | `transifex` |  |  |
| `reload` | operation | self | `transifex` | model |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_system` | no | yes | no | no | `transifex` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `transifex.transifex_code_translation_tree_view` | list |  | `source`, `value`, `module`, `lang`, `transifex_url` |  |  | `transifex` |
| `transifex.transifex_code_translation_view_search` | search |  | `module`, `lang`, `source`, `value` |  | `Not Translated`, `Module` | `transifex` |

## Server actions

| Action | Name | State | Binding | Custom logic |
|---|---|---|---|---|
| `transifex.action_code_translations` | Transifex Code Translations | code |  | yes |

## Scheduled jobs

| Job | Name | Every | Operation | Priority |
|---|---|---|---|---|
| `transifex.transifex_code_translation_reload` | Transifex: Reload code translations | 7 days | `reload` |  |

Machine-readable definition: `../../../schemas/data/entities/transifex.code.translation.json`; views: `../../../schemas/interfaces/views/transifex.code.translation.json`.
