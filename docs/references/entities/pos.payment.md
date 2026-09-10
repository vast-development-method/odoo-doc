# Point of Sale Payments (`pos.payment`)

**Transport name:** `pos.payment`  
**Storage name:** `pos_payment`  
**Kind:** persistent entity (one table)  
**Defined by package:** `point_of_sale`  
**Extended by packages:** `pos_restaurant`, `pos_dpopay`, `pos_hr`, `pos_online_payment`, `pos_pine_labs`, `pos_razorpay`, `pos_restaurant_adyen`, `pos_viva_com`

Description: Point of Sale Payments

## Identity and behavior

- Mixins (classical inheritance): `pos.load.mixin`
- Default ordering: `id desc`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (34)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Label | single line text |  | read only |
| `pos_order_id` | Order | many to one | `pos.order` | required; indexed; on delete of the target: cascade |
| `amount` | Amount | monetary |  | required; currency taken from `currency_id`; Help: Total amount of the payment. |
| `payment_method_id` | Payment Method | many to one | `pos.payment.method` | required |
| `payment_date` | Date | date and time |  | required; read only; default computed dynamically (lambda self: fields.Datetime.now()) |
| `currency_id` | Currency | many to one | `res.currency` | related through path `pos_order_id.currency_id` |
| `currency_rate` | Conversion Rate | float |  | related through path `pos_order_id.currency_rate`; Help: Conversion rate from company currency to order currency. |
| `partner_id` | Customer | many to one | `res.partner` | related through path `pos_order_id.partner_id` |
| `session_id` | Session | many to one | `pos.session` | related through path `pos_order_id.session_id` and stored; indexed |
| `user_id` | Employee | many to one | `res.users` | related through path `session_id.user_id` |
| `company_id` | Company | many to one | `res.company` | related through path `pos_order_id.company_id` and stored |
| `card_type` | Type of card used | single line text |  | Help: The type of the payment card (e.g. CREDIT CARD OR DEBIT CARD) |
| `card_brand` | Brand of card | single line text |  | Help: The brand of the payment card (e.g. Visa, AMEX, ...) |
| `card_no` | Card Number(Last 4 Digit) | single line text |  |  |
| `cardholder_name` | Card Owner name | single line text |  |  |
| `payment_ref_no` | Payment reference number | single line text |  | Help: Payment reference number from payment provider terminal |
| `payment_method_authcode` | Payment APPR Code | single line text |  |  |
| `payment_method_issuer_bank` | Payment Issuer Bank | single line text |  |  |
| `payment_method_payment_mode` | Payment Mode | single line text |  |  |
| `transaction_id` | Payment Transaction identifier | single line text |  |  |
| `payment_status` | Payment Status | single line text |  |  |
| `ticket` | Payment Receipt Info | single line text |  |  |
| `is_change` | Is this payment change? | boolean |  | default  |
| `account_move_id` | Account Move | many to one | `account.move` | indexed (btree_not_null) |
| `uuid` | Uuid | single line text |  | read only; default computed dynamically (lambda self: str(uuid4())); not copied on duplication |
| `dpopay_rrn` | RRN | single line text |  | Help: Retrieval Reference Number generated for the DPO Pay transaction. |
| `dpopay_transaction_ref` | Transaction Reference | single line text |  | Help: Reference number required for Mobile Money refund transactions. |
| `dpopay_mobile_money_phone` | Mobile Money Phone Number(Last 4 Digit) | single line text |  | Help: Customer's phone number used to complete the Mobile Money payment. |
| `employee_id` | Cashier | many to one | `hr.employee` | related through path `pos_order_id.employee_id` and stored; indexed |
| `online_account_payment_id` | Online accounting payment | many to one | `account.payment` | read only |
| `pine_labs_plutus_transaction_ref` | PineLabs Transaction identifier | single line text |  | Help: Required during the refund order process: https://developer.pinelabs.com/in/instore/cloud-integration#Example-JSON-request-for-Void-ICB-on-UPI-transaction |
| `razorpay_reverse_ref_no` | Razorpay Reverse Reference No. | single line text |  |  |
| `razorpay_p2p_request_id` | Razorpay p2pRequestId | single line text |  | Help: Required to fetch payment status during the refund order process |
| `viva_com_session_id` | Viva Com Session | single line text |  | Help: Session ID of the transaction, stored so that it can be used to refund the payment. |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_unique_uuid` | Constraint | `unique (uuid)` | A payment with this uuid already exists | `point_of_sale` |

## Operations (11)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_load_pos_data_domain` | internal rule | self, data, config | `point_of_sale` | model |  |
| `_compute_display_name` | computation | self | `point_of_sale` | depends: `amount`, `currency_id` |  |
| `_check_amount` | validation | self | `point_of_sale` | constrains: `amount` |  |
| `_check_payment_method_id` | validation | self | `point_of_sale`, `pos_online_payment` | constrains: `payment_method_id` |  |
| `_create_payment_moves` | internal rule | self, is_reverse | `point_of_sale` |  |  |
| `_get_receivable_lines_for_invoice_reconciliation` | preparation rule | self, receivable_account | `point_of_sale` |  | If this payment is linked to an account.move, this returns the corresponding receivable lines that should be reconciled with the invoice's receivable lines. The introduced heuristics here is important for cases where the pos receivable account is the same as the receivable account of the customer.  - positive payment -> negative balance lines - negative payment -> positive balance lines |
| `_update_payment_line_for_tip` | internal rule | self, tip_amount | `pos_restaurant_adyen`, `pos_restaurant` |  | Inherit this method to perform reauthorization or capture on electronic payment. |
| `_compute_cashier` | computation | self | `pos_hr` | depends: `employee_id`, `user_id` |  |
| `create` | lifecycle override | self, vals_list | `pos_online_payment` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `pos_online_payment` |  |  |
| `_adyen_capture` | internal rule | self | `pos_restaurant_adyen` |  |  |

## Validation and error messages (6)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_amount` | ValidationError | You cannot edit a payment for a posted order. | `point_of_sale` |
| `_check_payment_method_id` | ValidationError | The payment method selected is not allowed in the config of the POS session. | `point_of_sale` |
| `create` | UserError | Cannot create a POS online payment without an accounting payment. | `pos_online_payment` |
| `create` | UserError | Cannot create a POS online payment without an accounting payment. | `pos_online_payment` |
| `create` | UserError | Cannot create a POS payment with a not online payment method and an online accounting payment. | `pos_online_payment` |
| `write` | UserError | Cannot edit a POS online payment essential data. | `pos_online_payment` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_pos_user` | yes | yes | yes | yes | `point_of_sale` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| PoS Payment | global (all users) | `[('company_id', 'in', company_ids)]` | True | True | True | True |

## Views (7)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `point_of_sale.view_pos_payment_form` | form |  | `session_id`, `pos_order_id`, `amount`, `currency_id`, `payment_method_id`, `payment_method_payment_mode`, `card_type`, `card_brand`, `card_no`, `cardholder_name`, `payment_method_issuer_bank`, `payment_method_authcode`, `payment_ref_no`, `transaction_id` |  |  | `point_of_sale` |
| `point_of_sale.view_pos_payment_tree` | list |  | `currency_id`, `payment_date`, `payment_method_id`, `pos_order_id`, `user_id`, `amount` |  |  | `point_of_sale` |
| `point_of_sale.view_pos_payment_search` | search |  | `name`, `amount`, `pos_order_id` |  | `Payment Method`, `Session` | `point_of_sale` |
| `pos_dpopay.view_pos_payment_form_inherited_pos_dpopay` | xpath | `point_of_sale.view_pos_payment_form` | `dpopay_rrn`, `dpopay_transaction_ref`, `dpopay_mobile_money_phone` |  |  | `pos_dpopay` |
| `pos_hr.view_pos_payment_tree_inherit` | xpath | `point_of_sale.view_pos_payment_tree` | `employee_id` |  |  | `pos_hr` |
| `pos_online_payment.view_pos_payment_form` | xpath | `point_of_sale.view_pos_payment_form` | `online_account_payment_id` |  |  | `pos_online_payment` |
| `pos_pine_labs.view_pos_payment_form_inherit_pine_labs` | xpath | `point_of_sale.view_pos_payment_form` | `pine_labs_plutus_transaction_ref` |  |  | `pos_pine_labs` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `point_of_sale.action_pos_payment_form` | Payments | list,form | `[]` | `{'search_default_group_by_payment_method': 1}` |  | `point_of_sale` |

Machine-readable definition: `../../../schemas/data/entities/pos.payment.json`; views: `../../../schemas/interfaces/views/pos.payment.json`.
