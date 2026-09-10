# Record of QRIS transactions (`l10n_id.qris.transaction`)

**Transport name:** `l10n_id.qris.transaction`  
**Storage name:** `l10n_id_qris_transaction`  
**Kind:** persistent entity (one table)  
**Defined by package:** `l10n_id`  
**Extended by packages:** `l10n_id_pos`

Description: Record of QRIS transactions

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (8)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `model` | Model | single line text |  |  |
| `model_id` | Model identifier | single line text |  |  |
| `qris_invoice_id` | Qris Invoice | single line text |  | read only |
| `qris_amount` | Qris Amount | integer |  | read only |
| `qris_content` | Qris Content | single line text |  | read only |
| `qris_creation_datetime` | Qris Creation Datetime | date and time |  | read only |
| `bank_id` | Bank | many to one | `res.partner.bank` | Help: Bank used to generate the current QRIS transaction |
| `paid` | Paid | boolean |  | Help: Payment Status of QRIS |

## Operations (6)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_supported_models` | preparation rule | self | `l10n_id_pos`, `l10n_id` |  |  |
| `_constraint_model` | validation | self | `l10n_id` | constrains: `model` |  |
| `_get_record` | preparation rule | self | `l10n_id_pos`, `l10n_id` |  | Get the backend invoice record that the qris transaction is handling To be overriden in other modules |
| `_get_latest_transaction` | preparation rule | self, model, model_id | `l10n_id` | model | Find latest transaction associated to the model and model_id |
| `_l10n_id_get_qris_qr_statuses` | internal rule | self | `l10n_id` |  | Fetch the result of the transaction  :param invoice_bank_id (Model <res.partner.bank>): bank (with QRIS configuration) :returns tuple(bool, dict): paid/unpaid status and status_response from QRIS |
| `_gc_remove_pointless_qris_transactions` | background operation | self | `l10n_id` | autovacuum | Removes unpaid transactions that have been for more than 35 minutes. These can no longer be paid and status will no longer change |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_constraint_model` | ValidationError | QRIS capability is not extended to model %s yet! | `l10n_id` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_invoice` | yes | yes | yes | yes | `l10n_id` |

Machine-readable definition: `../../../schemas/data/entities/l10n_id.qris.transaction.json`.
