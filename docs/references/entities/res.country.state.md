# Country state (`res.country.state`)

**Transport name:** `res.country.state`  
**Storage name:** `res_country_state`  
**Kind:** persistent entity (one table)  
**Defined by package:** `base`  
**Extended by packages:** `point_of_sale`, `l10n_in`

Description: Country state

## Identity and behavior

- Mixins (classical inheritance): `pos.load.mixin`
- Default ordering: `code, id`
- Display name search fields: `["name", "code"]`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (4)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `country_id` | Country | many to one | `res.country` | required; indexed |
| `name` | State Name | single line text |  | required; Help: Administrative divisions of a country. E.g. Fed. State, Department, Canton |
| `code` | State Code | single line text |  | required; Help: The state code. |
| `l10n_in_tin` | TIN Number | single line text |  | maximum length 2; Help: TIN number-first two digits |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_name_code_uniq` | Constraint | `unique(country_id, code)` | The code of the state must be unique by country! | `base` |

## Operations (5)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `name_search` | operation | self, name, domain, operator, limit | `base` | model |  |
| `_search_display_name` | search rule | self, operator, value | `base` | model |  |
| `_get_name_search_domain` | preparation rule | self, name, operator | `base` |  |  |
| `_compute_display_name` | computation | self | `base` | depends: `country_id`; depends_context: `formatted_display_name` |  |
| `_load_pos_data_fields` | internal rule | self, config | `point_of_sale` | model |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_public` | no | yes | no | no | `base` |
| `base.group_portal` | no | yes | no | no | `base` |
| `base.group_user` | no | yes | no | no | `base` |
| `group_partner_manager` | yes | yes | yes | yes | `base` |

## Views (5)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `base.view_country_state_tree` | list |  | `name`, `code`, `country_id` |  |  | `base` |
| `base.view_country_state_form` | form |  | `name`, `code`, `country_id` |  |  | `base` |
| `base.view_country_state_search` | search |  | `name`, `country_id` |  | `Country` | `base` |
| `l10n_in.l10n_in_view_country_state_form_inherit` | field | `base.view_country_state_form` | `code`, `l10n_in_tin` |  |  | `l10n_in` |
| `l10n_in.l10n_in_view_country_state_tree_inherit` | field | `base.view_country_state_tree` | `code`, `l10n_in_tin` |  |  | `l10n_in` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `base.action_country_state` | Fed. States |  |  |  |  | `base` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `contacts.menu_country_state_partner` |  | `menu_localisation` | `base.action_country_state` | 2 |  |

Machine-readable definition: `../../../schemas/data/entities/res.country.state.json`; views: `../../../schemas/interfaces/views/res.country.state.json`.
