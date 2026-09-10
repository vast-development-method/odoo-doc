# Payment Terms Line (`account.payment.term.line`)

**Transport name:** `account.payment.term.line`  
**Storage name:** `account_payment_term_line`  
**Kind:** persistent entity (one table)  
**Defined by package:** `account`

Description: Payment Terms Line

## Identity and behavior

- Default ordering: `id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (7)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `value` | Value | selection |  | required; default `percent`; Help: Select here the kind of valuation related to this payment terms line. |
| `value_amount` | Due | float |  | computed by rule `_compute_value_amount` and stored; precision `Payment Terms`; Help: For percent enter a ratio between 0-100. |
| `delay_type` | Delay Type | selection |  | required; default `days_after` |
| `display_days_next_month` | Display Days Next Month | boolean |  | computed by rule `_compute_display_days_next_month` (not stored) |
| `days_next_month` | Days on the next month | single line text |  | default `10`; maximum length 2 |
| `nb_days` | Days | integer |  | computed by rule `_compute_days` and stored |
| `payment_id` | Payment Terms | many to one | `account.payment.term` | required; indexed; on delete of the target: cascade |

## Selection values

### `value` (Value)

| Value | Label |
|---|---|
| `percent` | Percent |
| `fixed` | Fixed |

### `delay_type` (Delay Type)

| Value | Label |
|---|---|
| `days_after` | Days after invoice date |
| `days_after_end_of_month` | Days after end of month |
| `days_after_end_of_next_month` | Days after end of next month |
| `days_end_of_month_on_the` | Days end of month on the |

## Operations (6)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_due_date` | preparation rule | self, date_ref | `account` |  |  |
| `_check_valid_char_value` | validation | self | `account` | constrains: `days_next_month` |  |
| `_compute_display_days_next_month` | computation | self | `account` | depends: `delay_type` |  |
| `_check_percent` | validation | self | `account` | constrains: `value`, `value_amount` |  |
| `_compute_days` | computation | self | `account` | depends: `payment_id` |  |
| `_compute_value_amount` | computation | self | `account` | depends: `payment_id` |  |

## Validation and error messages (3)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_valid_char_value` | ValidationError | The days added must be a number and has to be between 0 and 31. | `account` |
| `_check_valid_char_value` | ValidationError | The days added must be between 0 and 31. | `account` |
| `_check_percent` | ValidationError | Percentages on the Payment Terms lines must be between 0 and 100. | `account` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `account` |
| `account.group_account_manager` | yes | yes | yes | yes | `account` |

Machine-readable definition: `../../../schemas/data/entities/account.payment.term.line.json`.
