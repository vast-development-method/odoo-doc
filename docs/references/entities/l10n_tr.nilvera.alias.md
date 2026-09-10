# Customer Alias on Nilvera (`l10n_tr.nilvera.alias`)

**Transport name:** `l10n_tr.nilvera.alias`  
**Storage name:** `l10n_tr_nilvera_alias`  
**Kind:** persistent entity (one table)  
**Defined by package:** `l10n_tr_nilvera`

Description: Customer Alias on Nilvera

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  |  |
| `partner_id` | Partner | many to one | `res.partner` |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_readonly` | no | yes | no | no | `l10n_tr_nilvera` |
| `account.group_account_invoice` | yes | yes | yes | yes | `l10n_tr_nilvera` |
| `account.group_account_manager` | yes | yes | yes | yes | `l10n_tr_nilvera` |
| `account.group_account_user` | yes | yes | yes | yes | `l10n_tr_nilvera` |

Machine-readable definition: `../../../schemas/data/entities/l10n_tr.nilvera.alias.json`.
