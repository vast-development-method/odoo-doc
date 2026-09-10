# Transaction Lipa na M-PESA (`transaction.lipa.na.mpesa`)

**Transport name:** `transaction.lipa.na.mpesa`  
**Storage name:** `transaction_lipa_na_mpesa`  
**Kind:** persistent entity (one table)  
**Defined by package:** `pos_safaricom`

Description: Transaction Lipa na M-PESA

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (5)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `trans_id` | Transaction identifier | single line text |  |  |
| `name` | Name | single line text |  |  |
| `amount` | Amount | integer |  |  |
| `number` | Number | single line text |  |  |
| `received_at` | Received At | date and time |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `point_of_sale.group_pos_user` | yes | yes | yes | yes | `pos_safaricom` |

Machine-readable definition: `../../../schemas/data/entities/transaction.lipa.na.mpesa.json`.
