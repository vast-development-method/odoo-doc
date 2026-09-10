# Partner Tags (`res.partner.category`)

**Transport name:** `res.partner.category`  
**Storage name:** `res_partner_category`  
**Kind:** persistent entity (one table)  
**Defined by package:** `base`  
**Extended by packages:** `l10n_tr_nilvera_einvoice`

Description: Partner Tags

## Identity and behavior

- Default ordering: `name, id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (7)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required; translatable |
| `color` | Color | integer |  | default computed dynamically (_get_default_color) |
| `parent_id` | Category | many to one | `res.partner.category` | indexed; on delete of the target: cascade |
| `child_ids` | Child Tags | one to many | `res.partner.category` | inverse field `parent_id` |
| `active` | Active | boolean |  | default `True`; Help: The active field allows you to hide the category without removing it. |
| `parent_path` | Parent Path | single line text |  | indexed |
| `partner_ids` | Partners | many to many | `res.partner` | not copied on duplication |

## Operations (8)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_default_color` | preparation rule | self | `base` |  |  |
| `_check_parent_id` | validation | self | `base` | constrains: `parent_id` |  |
| `_compute_display_name` | computation | self | `base` | depends: `parent_id` | Return the categories' display name, including their direct parent by default. |
| `_search_display_name` | search rule | self, operator, value | `base` | model |  |
| `_get_categories_from_xml_ids` | preparation rule | self, xml_ids_list | `l10n_tr_nilvera_einvoice` |  |  |
| `_get_l10n_tr_official_categories` | preparation rule | self | `l10n_tr_nilvera_einvoice` |  |  |
| `_get_l10n_tr_official_mandatory_categories` | preparation rule | self | `l10n_tr_nilvera_einvoice` |  |  |
| `_unlink_l10n_tr_official_category` | internal rule | self | `l10n_tr_nilvera_einvoice` | ondelete | Prevent the deletion of Nilvera official TR categories |

## Validation and error messages (2)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_parent_id` | ValidationError | You can not create recursive tags. | `base` |
| `_unlink_l10n_tr_official_category` | UserError | The Contact Tag(s) cannot be deleted because it is used in Türkiye electronic integrations. | `l10n_tr_nilvera_einvoice` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_user` | no | yes | no | no | `base` |
| `group_partner_manager` | yes | yes | yes | yes | `base` |
| `sales_team.group_sale_manager` | no | yes | no | no | `crm` |
| `sales_team.group_sale_salesman` | yes | yes | yes | no | `crm` |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `base.view_partner_category_form` | form |  | `name`, `color`, `parent_id`, `active` |  |  | `base` |
| `base.view_partner_category_list` | list |  | `name`, `parent_id`, `color` |  |  | `base` |
| `base.res_partner_category_view_search` | search |  | `name`, `display_name` |  | `Archived`, `Category`, `Color` | `base` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `base.action_partner_category_form` | Contact Tags |  |  |  |  | `base` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `contacts.menu_partner_category_form` | Contact Tags | `res_partner_menu_config` | `base.action_partner_category_form` | 1 |  |

Machine-readable definition: `../../../schemas/data/entities/res.partner.category.json`; views: `../../../schemas/interfaces/views/res.partner.category.json`.
