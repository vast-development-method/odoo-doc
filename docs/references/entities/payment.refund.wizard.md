# Payment Refund Wizard (`payment.refund.wizard`)

**Transport name:** `payment.refund.wizard`  
**Storage name:** `payment_refund_wizard`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `account_payment`

Description: Payment Refund Wizard

## Fields (9)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `payment_id` | Payment | many to one | `account.payment` | read only; default computed dynamically (lambda self: self.env.context.get('active_id')) |
| `transaction_id` | Payment Transaction | many to one |  | related through path `payment_id.payment_transaction_id` |
| `payment_amount` | Payment Amount | monetary |  | related through path `payment_id.amount` |
| `refunded_amount` | Refunded Amount | monetary |  | computed by rule `_compute_refunded_amount` (not stored) |
| `amount_available_for_refund` | Maximum Refund Allowed | monetary |  | related through path `payment_id.amount_available_for_refund` |
| `amount_to_refund` | Refund Amount | monetary |  | computed by rule `_compute_amount_to_refund` and stored |
| `currency_id` | Currency | many to one |  | related through path `transaction_id.currency_id` |
| `support_refund` | Refund | selection |  | computed by rule `_compute_support_refund` (not stored) |
| `has_pending_refund` | Has a pending refund | boolean |  | computed by rule `_compute_has_pending_refund` (not stored) |

## Selection values

### `support_refund` (Refund)

| Value | Label |
|---|---|
| `none` | Unsupported |
| `full_only` | Full Only |
| `partial` | Partial |

## Operations (6)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_check_amount_to_refund_within_boundaries` | validation | self | `account_payment` | constrains: `amount_to_refund` |  |
| `_compute_refunded_amount` | computation | self | `account_payment` | depends: `amount_available_for_refund` |  |
| `_compute_amount_to_refund` | computation | self | `account_payment` | depends: `amount_available_for_refund` | Set the default amount to refund to the amount available for refund. |
| `_compute_support_refund` | computation | self | `account_payment` | depends: `transaction_id.provider_id`, `transaction_id.payment_method_id` |  |
| `_compute_has_pending_refund` | computation | self | `account_payment` | depends: `payment_id` |  |
| `action_refund` | user action | self | `account_payment` |  |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_amount_to_refund_within_boundaries` | ValidationError | The amount to be refunded must be positive and cannot be superior to %s. | `account_payment` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_invoice` | yes | yes | yes | no | `account_payment` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `account_payment.payment_refund_wizard_view_form` | form |  | `has_pending_refund`, `payment_id`, `transaction_id`, `currency_id`, `support_refund`, `payment_amount`, `refunded_amount`, `amount_available_for_refund`, `amount_to_refund` | `Refund`, `Close` |  | `account_payment` |

Machine-readable definition: `../../../schemas/data/entities/payment.refund.wizard.json`; views: `../../../schemas/interfaces/views/payment.refund.wizard.json`.
