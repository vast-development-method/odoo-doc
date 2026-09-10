# Payment Capture Wizard (`payment.capture.wizard`)

**Transport name:** `payment.capture.wizard`  
**Storage name:** `payment_capture_wizard`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `payment`  
**Extended by packages:** `payment_adyen`

Description: Payment Capture Wizard

## Fields (13)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `transaction_ids` | Transaction | many to many | `payment.transaction` | read only; default computed dynamically (lambda self: self.env.context.get('active_ids')) |
| `authorized_amount` | Authorized Amount | monetary |  | computed by rule `_compute_authorized_amount` (not stored) |
| `captured_amount` | Already Captured | monetary |  | computed by rule `_compute_captured_amount` (not stored) |
| `voided_amount` | Already Voided | monetary |  | computed by rule `_compute_voided_amount` (not stored) |
| `available_amount` | Maximum Capture Allowed | monetary |  | computed by rule `_compute_available_amount` (not stored) |
| `amount_to_capture` | Amount To Capture | monetary |  | computed by rule `_compute_amount_to_capture` and stored |
| `is_amount_to_capture_valid` | Is Amount To Capture Valid | boolean |  | computed by rule `_compute_is_amount_to_capture_valid` (not stored) |
| `void_remaining_amount` | Void Remaining Amount | boolean |  |  |
| `currency_id` | Currency | many to one |  | related through path `transaction_ids.currency_id` |
| `support_partial_capture` | Support Partial Capture | boolean |  | computed by rule `_compute_support_partial_capture` (not stored); Help: Whether each of the transactions' provider supports the partial capture. |
| `has_draft_children` | Has Draft Children | boolean |  | computed by rule `_compute_has_draft_children` (not stored) |
| `has_remaining_amount` | Has Remaining Amount | boolean |  | computed by rule `_compute_has_remaining_amount` (not stored) |
| `has_adyen_tx` | Has Adyen Tx | boolean |  | computed by rule `_compute_has_adyen_tx` (not stored) |

## Operations (12)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_authorized_amount` | computation | self | `payment` | depends: `transaction_ids` |  |
| `_compute_captured_amount` | computation | self | `payment` | depends: `transaction_ids` |  |
| `_compute_voided_amount` | computation | self | `payment` | depends: `transaction_ids` |  |
| `_compute_available_amount` | computation | self | `payment` | depends: `authorized_amount`, `captured_amount`, `voided_amount` |  |
| `_compute_amount_to_capture` | computation | self | `payment` | depends: `available_amount` | Set the default amount to capture to the amount available for capture. |
| `_compute_is_amount_to_capture_valid` | computation | self | `payment` | depends: `amount_to_capture`, `available_amount` |  |
| `_compute_support_partial_capture` | computation | self | `payment` | depends: `transaction_ids` |  |
| `_compute_has_draft_children` | computation | self | `payment` | depends: `transaction_ids` |  |
| `_compute_has_remaining_amount` | computation | self | `payment` | depends: `available_amount`, `amount_to_capture` |  |
| `_check_amount_to_capture_within_boundaries` | validation | self | `payment` | constrains: `amount_to_capture` |  |
| `action_capture` | user action | self | `payment` |  |  |
| `_compute_has_adyen_tx` | computation | self | `payment_adyen` | depends: `transaction_ids` |  |

## Validation and error messages (2)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_amount_to_capture_within_boundaries` | ValidationError | The amount to capture must be positive and cannot be superior to %s. | `payment` |
| `_check_amount_to_capture_within_boundaries` | ValidationError | Some of the transactions you intend to capture can only be captured in full. Handle the transactions individually to capture a partial amount. | `payment` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | yes | yes | yes | no | `payment` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Payment Capture Wizard | global (all users) | `[('create_uid', '=', user.id)]` | True | True | True | True |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `payment.payment_capture_wizard_view_form` | form |  | `transaction_ids`, `is_amount_to_capture_valid`, `currency_id`, `support_partial_capture`, `has_draft_children`, `has_remaining_amount`, `authorized_amount`, `captured_amount`, `voided_amount`, `amount_to_capture`, `void_remaining_amount`, `available_amount` | `Capture`, `Close` |  | `payment` |
| `payment_adyen.payment_capture_wizard_view_form` | footer | `payment.payment_capture_wizard_view_form` | `has_adyen_tx` |  |  | `payment_adyen` |

Machine-readable definition: `../../../schemas/data/entities/payment.capture.wizard.json`; views: `../../../schemas/interfaces/views/payment.capture.wizard.json`.
