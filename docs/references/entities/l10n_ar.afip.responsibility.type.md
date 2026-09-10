# ARCA Responsibility Type (`l10n_ar.afip.responsibility.type`)

**Transport name:** `l10n_ar.afip.responsibility.type`  
**Storage name:** `l10n_ar_afip_responsibility_type`  
**Kind:** persistent entity (one table)  
**Defined by package:** `l10n_ar`  
**Extended by packages:** `l10n_ar_pos`

Description: ARCA Responsibility Type

## Identity and behavior

- Mixins (classical inheritance): `pos.load.mixin`
- Default ordering: `sequence`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (4)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required; indexed (trigram) |
| `sequence` | Sequence | integer |  |  |
| `code` | Code | single line text |  | required; indexed |
| `active` | Active | boolean |  | default `True` |

## Database constraints and indexes (2)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_name_uniq` | Constraint | `unique(name)` | Name must be unique! | `l10n_ar` |
| `_code_uniq` | Constraint | `unique(code)` | Code must be unique! | `l10n_ar` |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_load_pos_data_fields` | internal rule | self, config | `l10n_ar_pos` | model |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `l10n_ar` |
| `base.group_portal` | no | yes | no | no | `l10n_ar` |
| `base.group_public` | no | yes | no | no | `l10n_ar` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `l10n_ar.view_afip_responsibility_type_form` | form |  | `name`, `code`, `active` |  |  | `l10n_ar` |
| `l10n_ar.view_afip_responsibility_type_tree` | list |  | `name`, `code`, `active` |  |  | `l10n_ar` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `l10n_ar.action_afip_responsibility_type` | ARCA Responsibility Types |  |  |  |  | `l10n_ar` |

Machine-readable definition: `../../../schemas/data/entities/l10n_ar.afip.responsibility.type.json`; views: `../../../schemas/interfaces/views/l10n_ar.afip.responsibility.type.json`.
