# SRI Payment Method (`l10n_ec.sri.payment`)

**Transport name:** `l10n_ec.sri.payment`  
**Storage name:** `l10n_ec_sri_payment`  
**Kind:** persistent entity (one table)  
**Defined by package:** `l10n_ec`

Description: SRI Payment Method

## Identity and behavior

- Default ordering: `sequence, id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (4)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `sequence` | Sequence | integer |  | default `10` |
| `name` | Name | single line text |  | translatable |
| `code` | Code | single line text |  |  |
| `active` | Active | boolean |  | default `True` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_readonly` | no | yes | no | no | `l10n_ec` |
| `account.group_account_invoice` | no | yes | no | no | `l10n_ec` |
| `account.group_account_manager` | yes | yes | yes | yes | `l10n_ec` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `l10n_ec.view_payment_method_form` | form |  | `code`, `name`, `active` |  |  | `l10n_ec` |
| `l10n_ec.view_payment_method_tree` | list |  | `sequence`, `code`, `name`, `active` |  |  | `l10n_ec` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `l10n_ec.action_account_l10n_ec_sri_payment_tree` | Payment Methods SRI | list,form |  | `{'active_test': False}` |  | `l10n_ec` |

Machine-readable definition: `../../../schemas/data/entities/l10n_ec.sri.payment.json`; views: `../../../schemas/interfaces/views/l10n_ec.sri.payment.json`.
