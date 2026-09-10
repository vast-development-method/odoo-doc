# Transport Document (`l10n_it.ddt`)

**Transport name:** `l10n_it.ddt`  
**Storage name:** `l10n_it_ddt`  
**Kind:** persistent entity (one table)  
**Defined by package:** `l10n_it_edi`

Description: Transport Document

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `invoice_id` | Invoice Reference | one to many | `account.move` | inverse field `l10n_it_ddt_id` |
| `name` | Numero transport document | single line text |  | required; maximum length 20; Help: Transport document number |
| `date` | Data transport document | date |  | required; Help: Transport document date |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_display_name` | computation | self | `l10n_it_edi` | depends: `date` |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_invoice` | yes | yes | yes | yes | `l10n_it_edi` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `l10n_it_edi.l10n_it_ddt` | form |  | `name`, `date` |  |  | `l10n_it_edi` |
| `l10n_it_edi.l10n_it_ddt_list_view` | list |  | `name`, `date` |  |  | `l10n_it_edi` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `l10n_it_edi.action_ddt_account` | Transport Document | list,form |  |  |  | `l10n_it_edi` |

Machine-readable definition: `../../../schemas/data/entities/l10n_it.ddt.json`; views: `../../../schemas/interfaces/views/l10n_it.ddt.json`.
