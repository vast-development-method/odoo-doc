# Transifex Translation (`transifex.translation`)

**Transport name:** `transifex.translation`  
**Storage name:** `transifex_translation`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `transifex`

Description: Transifex Translation

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_transifex_projects` | preparation rule | self | `transifex` |  | get the transifex project name for each module  .tx/config files contains the project reference first section is [main], after '[system-16.sale]'  :rtype: dict :return: {module_name: tx_project_name} |
| `_update_transifex_url` | internal rule | self, translations | `transifex` |  | Update translations' Transifex URL  :param translations: the translations to update, may be a recordset or a list of dicts.     The elements of `translations` must have the fields/keys 'source', 'module', 'lang',     and the field/key 'transifex_url' is updated on them. |

Machine-readable definition: `../../../schemas/data/entities/transifex.translation.json`.
