# Incoterms (`account.incoterms`)

**Transport name:** `account.incoterms`  
**Storage name:** `account_incoterms`  
**Kind:** persistent entity (one table)  
**Defined by package:** `account`

Description: Incoterms

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required; translatable; Help: Incoterms are series of sales terms. They are used to divide transaction costs and responsibilities between buyer and seller and reflect state-of-the-art transportation practices. |
| `code` | Code | single line text |  | required; maximum length 3; Help: Incoterm Standard Code |
| `active` | Active | boolean |  | default `True`; Help: By unchecking the active field, you may hide an INCOTERM you will not use. |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_display_name` | computation | self | `account` | depends: `code` |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `account` |
| `account.group_account_manager` | yes | yes | yes | yes | `account` |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `account.view_incoterms_tree` | list |  | `active`, `code`, `name` |  |  | `account` |
| `account.account_incoterms_form` | form |  | `active`, `name`, `code` |  |  | `account` |
| `account.account_incoterms_view_search` | search |  | `name` |  | `Archived` | `account` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `account.action_incoterms_tree` | Incoterms | list,form |  |  |  | `account` |

Machine-readable definition: `../../../schemas/data/entities/account.incoterms.json`; views: `../../../schemas/interfaces/views/account.incoterms.json`.
