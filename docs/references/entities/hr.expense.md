# Expense (`hr.expense`)

**Transport name:** `hr.expense`  
**Storage name:** `hr_expense`  
**Kind:** persistent entity (one table)  
**Defined by package:** `hr_expense`  
**Extended by packages:** `project_hr_expense`, `sale_expense`, `project_sale_expense`

Description: Expense

## Identity and behavior

- Mixins (classical inheritance): `mail.thread.main.attachment`, `mail.activity.mixin`, `analytic.mixin`
- Default ordering: `date desc, id desc`
- Company consistency is checked automatically on company-bound relations
- Posting a message requires access: read
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (50)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Description | single line text |  | required; computed by rule `_compute_name` and stored; precomputed before insertion |
| `date` | Expense Date | date |  | default computed dynamically (fields.Date.context_today) |
| `employee_id` | Employee | many to one | `hr.employee` | required; computed by rule `_compute_employee_id` and stored; default computed dynamically (_default_employee_id); changes are tracked in the message thread; indexed; restricted by domain `[["filter_for_expense", "=", true]]`; must belong to the same company; precomputed before insertion |
| `department_id` | Department | many to one | `hr.department` | computed by rule `_compute_from_employee_id` and stored; not copied on duplication |
| `manager_id` | Manager | many to one | `res.users` | computed by rule `_compute_from_employee_id` and stored; changes are tracked in the message thread; not copied on duplication; restricted by domain `lambda self: [('share', '=', False), '\|', ('employee_id.expense_manager_id', 'in', self.env.user.id), ('all_group_ids', 'in', self.env.ref('hr_expense.group_hr_expense_team_approver').ids)]` |
| `company_id` | Company | many to one | `res.company` | required; read only; default computed dynamically (lambda self: self.env.company) |
| `product_id` | Category | many to one | `product.product` | changes are tracked in the message thread; on delete of the target: restrict; restricted by domain `[["can_be_expensed", "=", true]]`; must belong to the same company |
| `product_description` | Product Description | rich text |  | computed by rule `_compute_product_description` (not stored) |
| `product_uom_id` | Unit | many to one | `uom.uom` | computed by rule `_compute_uom_id` and stored; precomputed before insertion |
| `product_has_cost` | Product Has Cost | boolean |  | computed by rule `_compute_from_product` (not stored) |
| `product_has_tax` | Whether tax is defined on a selected product | boolean |  | computed by rule `_compute_from_product` (not stored) |
| `quantity` | Quantity | float |  | required; default `1`; precision `Product Unit` |
| `description` | Internal Notes | multi line text |  |  |
| `message_main_attachment_checksum` | Message Main Attachment Checksum | single line text |  | related through path `message_main_attachment_id.checksum` |
| `nb_attachment` | Number of Attachments | integer |  | computed by rule `_compute_nb_attachment` (not stored) |
| `attachment_ids` | Attachments | one to many | `ir.attachment` | restricted by domain `[["res_model", "=", "hr.expense"]]`; inverse field `res_id` |
| `state` | Status | selection |  | read only; computed by rule `_compute_state` and stored; default `draft`; changes are tracked in the message thread; indexed; not copied on duplication |
| `approval_state` | Approval State | selection |  | read only; not copied on duplication |
| `approval_date` | Approval Date | date and time |  | read only |
| `duplicate_expense_ids` | Duplicate Expense | many to many | `hr.expense` | computed by rule `_compute_duplicate_expense_ids` (not stored) |
| `same_receipt_expense_ids` | Same Receipt Expense | many to many | `hr.expense` | computed by rule `_compute_same_receipt_expense_ids` (not stored) |
| `split_expense_origin_id` | Origin Split Expense | many to one | `hr.expense` | Help: Original expense from a split. |
| `tax_amount_currency` | Tax amount in Currency | monetary |  | computed by rule `_compute_tax_amount_currency` and stored; currency taken from `currency_id`; precomputed before insertion; Help: Tax amount in currency |
| `tax_amount` | Tax amount | monetary |  | computed by rule `_compute_tax_amount` and stored; currency taken from `company_currency_id`; precomputed before insertion; Help: Tax amount in company currency |
| `total_amount_currency` | Total In Currency | monetary |  | computed by rule `_compute_total_amount_currency` and stored; changes are tracked in the message thread; currency taken from `currency_id`; precomputed before insertion |
| `total_amount` | Total | monetary |  | computed by rule `_compute_total_amount` and stored; writable through an inverse rule; changes are tracked in the message thread; currency taken from `company_currency_id`; precomputed before insertion |
| `untaxed_amount_currency` | Total Untaxed Amount In Currency | monetary |  | computed by rule `_compute_tax_amount_currency` and stored; currency taken from `currency_id`; precomputed before insertion |
| `untaxed_amount` | Total Untaxed Amount | monetary |  | computed by rule `_compute_tax_amount` and stored; currency taken from `currency_id`; precomputed before insertion |
| `amount_residual` | Amount Due | monetary |  | read only; related through path `account_move_id.amount_residual`; currency taken from `company_currency_id` |
| `price_unit` | Unit Price | float |  | required; read only; computed by rule `_compute_price_unit` and stored; precomputed before insertion |
| `currency_id` | Currency | many to one | `res.currency` | required; computed by rule `_compute_currency_id` and stored; default computed dynamically (lambda self: self.env.company.currency_id); precomputed before insertion |
| `company_currency_id` | Report Company Currency | many to one | `res.currency` | read only; related through path `company_id.currency_id` |
| `is_multiple_currency` | Is currency_id different from the company_currency_id | boolean |  | computed by rule `_compute_is_multiple_currency` (not stored) |
| `currency_rate` | Currency Rate | float |  | read only; computed by rule `_compute_currency_rate` (not stored); changes are tracked in the message thread; precision `[16, 9]` |
| `label_currency_rate` | Label Currency Rate | single line text |  | read only; computed by rule `_compute_currency_rate` (not stored) |
| `journal_id` | Journal | many to one | `account.journal` | read only; related through path `payment_method_line_id.journal_id` |
| `selectable_payment_method_line_ids` | Selectable Payment Method Line | many to many | `account.payment.method.line` | computed by rule `_compute_selectable_payment_method_line_ids` (not stored) |
| `payment_method_line_id` | Payment Method | many to one | `account.payment.method.line` | computed by rule `_compute_payment_method_line_id` and stored; restricted by domain `[('id', 'in', selectable_payment_method_line_ids)]`; Help: The payment method used when the expense is paid by the company. |
| `account_move_id` | Journal Entry | many to one | `account.move` | read only; indexed (btree_not_null); not copied on duplication |
| `payment_mode` | Paid By | selection |  | required; default `own_account`; changes are tracked in the message thread |
| `vendor_id` | Vendor | many to one | `res.partner` |  |
| `account_id` | Account | many to one | `account.account` | computed by rule `_compute_account_id` and stored; restricted by domain `[('account_type', 'not in', ('asset_receivable', 'liability_payable', 'asset_cash', 'liability_credit_card'))]`; must belong to the same company; precomputed before insertion; Help: An expense account is expected |
| `tax_ids` | Included taxes | many to many | `account.tax` | computed by rule `_compute_tax_ids` and stored; restricted by domain `[('type_tax_use', '=', 'purchase')]`; must belong to the same company; association table `expense_tax`; precomputed before insertion; Help: Both price-included and price-excluded taxes will behave as price-included taxes for expenses. |
| `is_editable` | Is Editable By Current User | boolean |  | read only; computed by rule `_compute_is_editable` (not stored) |
| `can_reset` | Can Reset | boolean |  | read only; computed by rule `_compute_can_reset` (not stored) |
| `can_approve` | Can Approve | boolean |  | read only; computed by rule `_compute_can_approve` (not stored) |
| `former_sheet_id` | Former Report | integer |  |  |
| `sale_order_id` | Customer to Reinvoice | many to one | `sale.order` | computed by rule `_compute_sale_order_id` and stored; changes are tracked in the message thread; indexed (btree_not_null); restricted by domain `[('state', '=', 'sale')]`; must belong to the same company; Help: If the category has an expense policy, it will be reinvoiced on this sales order |
| `sale_order_line_id` | Sale Order Line | many to one | `sale.order.line` | read only; computed by rule `_compute_sale_order_id` and stored; indexed (btree_not_null) |
| `can_be_reinvoiced` | Can be reinvoiced | boolean |  | computed by rule `_compute_can_be_reinvoiced` (not stored) |

## Selection values

### `state` (Status)

| Value | Label |
|---|---|
| `draft` | Draft |
| `submitted` | Submitted |
| `approved` | Approved |
| `posted` | Posted |
| `in_payment` | In Payment |
| `paid` | Paid |
| `refused` | Refused |

### `payment_mode` (Paid By)

| Value | Label |
|---|---|
| `own_account` | Employee (to reimburse) |
| `company_account` | Company |

## State fields

State machine fields of this entity: `state`, `approval_state`. Transitions are specified in the domain documents.

## Operations (94)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_default_employee_id` | preparation rule | self | `hr_expense` | model |  |
| `_check_non_zero` | validation | self | `hr_expense` | constrains: `state`, `approval_state`, `total_amount`, `total_amount_currency` | Helper to raise when we should ensure that an expense isn't approved |
| `_check_o2o_payment` | validation | self | `hr_expense` | constrains: `account_move_id` |  |
| `_compute_currency_id` | computation | self | `hr_expense` | depends: `product_has_cost` |  |
| `_compute_is_editable` | computation | self | `hr_expense` | depends_context: `uid`; depends: `employee_id`, `manager_id`, `state` |  |
| `_onchange_product_has_cost` | on change | self | `hr_expense` | onchange: `product_has_cost` | Reset quantity to 1, in case of 0-cost product. To make sure switching non-0-cost to 0-cost doesn't keep the quantity. |
| `_compute_product_description` | computation | self | `hr_expense` | depends_context: `lang`; depends: `product_id` |  |
| `_compute_name` | computation | self | `hr_expense` | depends: `product_id` |  |
| `_set_expense_currency_rate` | internal rule | self, date_today | `hr_expense` |  |  |
| `_compute_currency_rate` | computation | self | `hr_expense` | depends: `currency_id`, `total_amount_currency`, `date` | We want the default system rate when the following change: - the currency of the expense - the total amount in foreign currency - the date of the expense this will cause the rate to be recomputed twice with possible changes but we don't have the required fields to store the override state in stable |
| `_compute_is_multiple_currency` | computation | self | `hr_expense` | depends: `currency_id`, `company_currency_id` |  |
| `_compute_from_product` | computation | self | `hr_expense` | depends: `product_id` |  |
| `_compute_uom_id` | computation | self | `hr_expense` | depends: `product_id.uom_id` |  |
| `_compute_state` | computation | self | `hr_expense` | depends: `amount_residual`, `account_move_id.state`, `account_move_id.payment_state`, `approval_state` | Compute the states of the expense as such (priority is given to the last matching state of the list):     - draft: By default     - submitted: When the approval_state is 'submitted'     - approved: When the approval_state is 'approved'     - refused: When the approval_state is 'refused'     - paid: When it is a company paid expense or the move state is neither 'draft' nor 'posted'     - in_payment (or paid): When the move state is 'posted' and it's 'payment_state' is 'in_payment' or 'paid'                             or ('partial' and there is a residual amount)     - posted: When the linked m |
| `_compute_from_employee_id` | computation | self | `hr_expense` | depends: `employee_id`, `employee_id.department_id` |  |
| `_compute_total_amount_currency` | computation | self | `hr_expense` | depends: `quantity`, `price_unit`, `tax_ids` |  |
| `_inverse_total_amount_currency` | on change | self | `hr_expense` | onchange: `total_amount_currency` |  |
| `_compute_total_amount` | computation | self | `hr_expense` | depends: `date`, `company_id`, `currency_id`, `company_currency_id`, `is_multiple_currency`, `total_amount_currency`, `product_id`, `employee_id.user_id.partner_id`, `quantity` |  |
| `_inverse_total_amount` | inverse computation | self | `hr_expense` |  | Allows to set a custom rate on the expense, and avoid the override when it makes no sense |
| `_compute_tax_ids` | computation | self | `hr_expense` | depends: `product_id`, `company_id` |  |
| `_compute_tax_amount_currency` | computation | self | `hr_expense` | depends: `total_amount_currency`, `tax_ids` | Note: as total_amount_currency can be set directly by the user (for product without cost) or needs to be computed (for product with cost), `untaxed_amount_currency` can't be computed in the same method as `total_amount_currency`. |
| `_compute_tax_amount` | computation | self | `hr_expense` | depends: `total_amount`, `currency_rate`, `tax_ids`, `is_multiple_currency` | Note: as total_amount can be set directly by the user when the currency_rate is overridden, the tax must be computed after the total_amount. |
| `_compute_price_unit` | computation | self | `hr_expense` | depends: `total_amount`, `total_amount_currency` | The price_unit is the unit price of the product if no product is set and no attachment overrides it. Otherwise it is always computed from the total_amount and the quantity else it would break the Receipt Entry when edited after creation. |
| `_compute_payment_method_line_id` | computation | self | `hr_expense` | depends: `selectable_payment_method_line_ids` |  |
| `_compute_selectable_payment_method_line_ids` | computation | self | `hr_expense` | depends: `company_id` |  |
| `_compute_account_id` | computation | self | `hr_expense` | depends: `product_id`, `company_id` |  |
| `_compute_employee_id` | computation | self | `hr_expense` | depends: `company_id` |  |
| `_compute_same_receipt_expense_ids` | computation | self | `hr_expense` | depends: `attachment_ids` |  |
| `_compute_duplicate_expense_ids` | computation | self | `hr_expense` | depends: `employee_id`, `product_id`, `total_amount_currency` |  |
| `_compute_analytic_distribution` | computation | self | `hr_expense`, `project_hr_expense`, `project_sale_expense` | depends: `product_id`, `account_id`, `employee_id` |  |
| `_compute_nb_attachment` | computation | self | `hr_expense` |  |  |
| `_compute_can_reset` | computation | self | `hr_expense` | depends_context: `uid`; depends: `employee_id`, `state` |  |
| `_compute_can_approve` | computation | self | `hr_expense` | depends_context: `uid`; depends: `employee_id` |  |
| `_unlink_except_approved` | internal rule | self | `hr_expense` | ondelete |  |
| `write` | lifecycle override | self, vals | `hr_expense` |  |  |
| `create` | lifecycle override | self, vals_list | `hr_expense`, `project_hr_expense` | model_create_multi |  |
| `_message_auto_subscribe_followers` | messaging hook | self, updated_values, subtype_ids | `hr_expense` |  |  |
| `_get_employee_from_email` | preparation rule | self, email_address | `hr_expense` | model |  |
| `_parse_product` | internal rule | self, expense_description | `hr_expense` | model | Parse the subject to find the product. Product code should be the first word of expense_description Return product.product and updated description |
| `_parse_price` | internal rule | self, expense_description, currencies | `hr_expense` | model | Return price, currency and updated description |
| `_parse_expense_subject` | internal rule | self, expense_description, currencies | `hr_expense` | model | Fetch product, price and currency info from mail subject.  Product can be identified based on product name or product code. It can be passed between [] or it can be placed at start.  When parsing, only consider currencies passed as parameter. This will fetch currency in symbol($) or ISO name (USD).  Some valid examples:     Travel by Air [TICKET] USD 1205.91     TICKET $1205.91 Travel by Air     Extra expenses 29.10EUR [EXTRA] |
| `_send_expense_success_mail` | internal rule | self, msg_dict, expense | `hr_expense` |  | Send a confirmation mail to the employee that an expense has been created by their previous mail |
| `_get_empty_list_mail_alias` | preparation rule | self | `hr_expense` | model |  |
| `_track_subtype` | messaging hook | self, init_values | `hr_expense` |  |  |
| `update_activities_and_mails` | operation | self | `hr_expense` |  | Update the "Review this expense" activity with the new state of the expense, also sends mail to approver to ask them to act |
| `_cron_send_submitted_expenses_mail` | background operation | self | `hr_expense` | model |  |
| `_send_submitted_expenses_mail` | internal rule | self | `hr_expense` |  |  |
| `get_empty_list_help` | operation | self, help_message | `hr_expense` | model |  |
| `message_new` | messaging hook | self, msg_dict, custom_values | `hr_expense` | model |  |
| `action_open_split_expense` | user action | self | `hr_expense` |  |  |
| `action_submit` | user action | self | `hr_expense` |  | Submit a draft expense to an approve, may skip to the approval step if no approver on the employee nor the expense |
| `_can_be_autovalidated` | internal rule | self | `hr_expense` |  | Check whether the given expenses can be auto-validated (no approver) |
| `action_approve` | user action | self | `hr_expense` |  | Approve an expense, pops a wizard if a duplicated expense is found to confirm they are all valid expenses |
| `action_refuse` | user action | self | `hr_expense` |  | Refuse an expense with a reason |
| `action_post` | user action | self | `hr_expense`, `project_sale_expense`, `sale_expense` |  | Post the expense, following one of those two options:     - Company-paid expenses: Create and post a payment, with an accounting entry     - Employee-paid expenses: Through a wizard, create and post a receipt |
| `action_pay` | user action | self | `hr_expense` |  | Register payment shortcut on the expense form view |
| `action_reset` | user action | self | `hr_expense` |  | Reset an expense to draft state, reversing the accounting entries if needed |
| `attach_document` | operation | self, **kwargs | `hr_expense` |  | When an attachment is uploaded as a receipt, set it as the main attachment. |
| `_get_untitled_expense_name` | preparation rule | self, *args | `hr_expense` | model | Done in a specific function to be called by hr_expense_extract to keep the same translation |
| `create_expense_from_attachments` | operation | self, attachment_ids, view_type | `hr_expense` | model | Create the expenses from files.  :return: An action redirecting to hr.expense list view. |
| `action_show_same_receipt_expense_ids` | user action | self | `hr_expense` |  |  |
| `get_expense_dashboard` | operation | self | `hr_expense` | model |  |
| `action_approve_duplicates` | user action | self | `hr_expense` |  |  |
| `action_split_wizard` | user action | self | `hr_expense` |  |  |
| `action_open_account_move` | user action | self | `hr_expense` |  |  |
| `_check_can_approve` | validation | self | `hr_expense` |  |  |
| `_get_cannot_approve_reason` | preparation rule | self | `hr_expense` |  | Returns the reason why the user cannot approve the expense |
| `_check_can_refuse` | validation | self | `hr_expense` |  |  |
| `_check_can_reset_approval` | validation | self | `hr_expense` |  |  |
| `_check_can_create_move` | validation | self | `hr_expense` |  |  |
| `_do_approve` | internal rule | self, check | `hr_expense` |  |  |
| `_do_reset_approval` | internal rule | self | `hr_expense` |  |  |
| `_do_refuse` | internal rule | self, reason | `hr_expense` |  |  |
| `_get_split_values` | preparation rule | self | `hr_expense`, `sale_expense` |  |  |
| `_get_default_responsible_for_approval` | preparation rule | self | `hr_expense` |  |  |
| `_needs_product_price_computation` | internal rule | self | `hr_expense` |  |  |
| `_post_wizard` | internal rule | self | `hr_expense` |  |  |
| `_post_without_wizard` | internal rule | self | `hr_expense` |  | Post an employee expense without any direct call for the wizard, should never be called unless in very specific flows |
| `_create_company_paid_moves` | internal rule | self | `hr_expense` |  | Creation of the account moves for the company paid expenses. -> Create an account payment (we only "log" the already paid expense so it can be reconciled) |
| `_prepare_receipts_vals` | preparation rule | self | `hr_expense` |  |  |
| `_prepare_payments_vals` | preparation rule | self | `hr_expense` |  |  |
| `_prepare_move_vals` | preparation rule | self | `hr_expense` |  |  |
| `_prepare_move_lines_vals` | preparation rule | self | `hr_expense` |  |  |
| `_prepare_base_line_for_taxes_computation` | preparation rule | self, **kwargs | `hr_expense` |  |  |
| `_get_move_line_name` | preparation rule | self | `hr_expense` |  | Helper to get the name of the account move lines related to an expense |
| `_get_base_account` | preparation rule | self | `hr_expense` |  | Returns the expense account or forces default values if none was found We need to do this as the installation process may delete the original account, and it doesn't recompute properly after Returned expense accounts are the first expense account encountered in the following list: 1. expense account of the expense itself 2. expense account of the product 3. expense account of the company 4. expense account on the purchase journal for employee expense |
| `_get_expense_account_destination` | preparation rule | self | `hr_expense` |  |  |
| `_get_outstanding_account_id` | preparation rule | self | `hr_expense` |  |  |
| `_creation_message` | internal rule | self | `hr_expense` |  |  |
| `_compute_can_be_reinvoiced` | computation | self | `sale_expense` | depends: `product_id.expense_policy` |  |
| `_compute_sale_order_id` | computation | self | `sale_expense` | depends: `can_be_reinvoiced` |  |
| `_onchange_sale_order_id` | on change | self | `sale_expense` | onchange: `sale_order_id` |  |
| `_sale_expense_reset_sol_quantities` | internal rule | self | `sale_expense` |  | Resets the quantity of a SOL created by a reinvoiced expense to 0 when the expense or its move is reset to an unfinished state  Note: Resetting the qty_delivered will raise if the product is a storable product and sale_stock is installed,       but it's fine as it doesn't make much sense to have a stored product in an expense. |
| `action_open_sale_order` | user action | self | `sale_expense` |  |  |

## Validation and error messages (30)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_default_employee_id` | ValidationError | The current user has no related employee. Please, create one. | `hr_expense` |
| `_check_non_zero` | ValidationError | Only draft expenses can have a total of 0. | `hr_expense` |
| `_check_o2o_payment` | ValidationError | Only one expense can be linked to a particular payment | `hr_expense` |
| `_inverse_total_amount_currency` | UserError | Uh-oh! You can’t edit this expense.  Reach out to the administrators, flash your best smile, and see if they'll grant you the magical access you seek. | `hr_expense` |
| `_unlink_except_approved` | UserError | You cannot delete a posted or approved expense. | `hr_expense` |
| `write` | UserError | You cannot edit the security fields of an expense manually | `hr_expense` |
| `write` | UserError | Uh-oh! You can’t edit this expense.  Reach out to the administrators, flash your best smile, and see if they'll grant you the magical access you seek. | `hr_expense` |
| `action_submit` | UserError | You do not have the required permission to submit this expense. | `hr_expense` |
| `action_submit` | UserError | You can not submit an expense without a category. | `hr_expense` |
| `action_post` | UserError | You can't post simultaneously employee-paid expenses belonging to different companies | `hr_expense` |
| `action_post` | UserError | The vendor is required for expenses using SEPA Credit Transfer as the payment method. Please set a vendor on the following expenses: %s | `hr_expense` |
| `attach_document` | AccessError | You don't have the access rights to modify this expense. | `hr_expense` |
| `attach_document` | UserError | You can't add an attachment to an expense once it has been approved. | `hr_expense` |
| `create_expense_from_attachments` | UserError | No attachment was provided | `hr_expense` |
| `create_expense_from_attachments` | UserError | Invalid attachments! | `hr_expense` |
| `create_expense_from_attachments` | UserError | You need to have at least one category that can be expensed in your database to proceed! | `hr_expense` |
| `action_split_wizard` | UserError | You cannot split an expense that is already posted. | `hr_expense` |
| `action_split_wizard` | UserError | You do not have the rights to edit this expense. | `hr_expense` |
| `_check_can_approve` | UserError | reasons | `hr_expense` |
| `_check_can_refuse` | UserError | reasons | `hr_expense` |
| `_check_can_reset_approval` | UserError | Only HR Officers, accountants, or the concerned employee can reset to draft. | `hr_expense` |
| `_check_can_reset_approval` | UserError | You cannot reset to draft an expense linked to a posted journal entry. | `hr_expense` |
| `_check_can_create_move` | UserError | You can only generate an accounting entry for approved expense(s). | `hr_expense` |
| `_check_can_create_move` | UserError | Please specify if the expenses were paid by the company, or the employee. | `hr_expense` |
| `_do_refuse` | UserError | You cannot cancel an expense linked to a posted journal entry | `hr_expense` |
| `_post_wizard` | UserError | Only expense paid by the employee can be posted with the wizard | `hr_expense` |
| `_prepare_payments_vals` | UserError | You need to add a manual payment method on the journal (%s) | `hr_expense` |
| `_get_base_account` | UserError | The system had a look at your expense, its product, your company and the journal but came back with empty hands. Give the system a hand to find an account by setting up an expense account. %(expense)s %(expense_name)s. | `hr_expense` |
| `_get_expense_account_destination` | UserError | The following expenses payment method leads to several accounts payable and this isn't supported: %(expenses)s | `hr_expense` |
| `_get_expense_account_destination` | UserError | No work contact found for the employee %(name)s, please configure one. | `hr_expense` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | yes | yes | yes | yes | `hr_expense` |
| `hr_expense.group_hr_expense_team_approver` | yes | yes | yes | yes | `hr_expense` |
| `hr_expense.group_hr_expense_manager` | yes | yes | yes | yes | `hr_expense` |
| `account.group_account_invoice` | yes | yes | yes | no | `hr_expense` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Manager Expense | `[                 (4, ref('account.group_account_user')),                 (4, ref('hr_expense.group_hr_expense_user'))]` | `[(1, '=', 1)]` | True | True | True | True |
| Team Approver Expense | `[(4, ref('hr_expense.group_hr_expense_team_approver'))]` | `['\|', '\|', '\|', '\|',                 ('employee_id.user_id', '=', user.id),                 ('employee_id.department_id.manager_id.user_id', '=', user.id),                 ('employee_id', 'child_of', user.employee_ids.ids),                 ('employee_id.expense_manager_id', '=', user.id),                 ('manager_id', '=', user.id)]` | True | True | True | True |
| Employee Expense | `[(4, ref('base.group_user'))]` | `[                 '\|', '&', ('employee_id.expense_manager_id', '=', user.id), ('state', 'in', ['draft', 'submitted', 'approved', 'refused']),                      '&', ('employee_id.user_id', '=', user.id), ('state', '=', 'draft')             ]` | True | True | True | True |
| Employees can't modify an expense that is not in draft state | `[(4, ref('base.group_user'))]` | `[                 '\|', '&', ('employee_id.user_id', '=', user.id), ('state', '!=', 'draft'),                      '&', ('employee_id.expense_manager_id', '=', user.id), ('state', 'in', ['submitted', 'approved', 'refused'])             ]` | True | False | False | False |
| Expense multi company rule | global (all users) | `[('company_id', 'in', company_ids)]` | True | True | True | True |

## Views (16)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr_expense.hr_expense_view_expenses_analysis_tree` | list |  | `is_editable`, `company_id`, `company_currency_id`, `nb_attachment`, `is_multiple_currency`, `product_has_cost`, `selectable_payment_method_line_ids`, `employee_id`, `name`, `department_id`, `date`, `product_id`, `payment_mode`, `activity_ids`, `analytic_distribution`, `account_id`, `journal_id`, `payment_method_line_id`, `manager_id`, `company_id`, `price_unit`, `quantity`, `tax_ids`, `tax_amount`, `nb_attachment`, `total_amount`, `total_amount_currency`, `currency_id`, `state` |  |  | `hr_expense` |
| `hr_expense.view_expenses_tree` | xpath | `hr_expense_view_expenses_analysis_tree` |  |  |  | `hr_expense` |
| `hr_expense.view_my_expenses_tree` | xpath | `hr_expense.view_expenses_tree` |  |  |  | `hr_expense` |
| `hr_expense.hr_expense_view_form` | form |  | `state`, `state`, `state`, `state`, `name`, `product_has_cost`, `product_has_tax`, `is_multiple_currency`, `is_editable`, `currency_id`, `company_currency_id`, `tax_amount`, `price_unit`, `nb_attachment`, `total_amount`, `duplicate_expense_ids`, `currency_rate`, `selectable_payment_method_line_ids`, `product_id`, `product_description`, `price_unit`, `quantity`, `product_uom_id`, `total_amount_currency`, `currency_id`, `tax_amount`, `total_amount`, `label_currency_rate`, `tax_ids`, `tax_amount_currency`, `employee_id`, `employee_id`, `payment_mode`, `date`, `company_id`, `manager_id`, `department_id`, `vendor_id`, `account_id`, `payment_method_line_id`, `analytic_distribution`, `description` | `Submit`, `Approve`, `Post Journal Entries`, `Submit`, `Approve`, `Post Journal Entries`, `Refuse`, `Reset`, `Reset`, `Split Expense`, `Split Expense`, `Split`, `action_open_account_move` |  | `hr_expense` |
| `hr_expense.hr_expense_view_form_without_header` | xpath | `hr_expense.hr_expense_view_form` |  |  |  | `hr_expense` |
| `hr_expense.hr_expense_view_expenses_analysis_kanban` | kanban |  | `currency_id`, `name`, `total_amount_currency`, `employee_id`, `state`, `date` |  |  | `hr_expense` |
| `hr_expense.hr_expense_kanban_view_minimal` | xpath | `hr_expense_view_expenses_analysis_kanban` |  |  |  | `hr_expense` |
| `hr_expense.hr_expense_kanban_view` | xpath | `hr_expense_view_expenses_analysis_kanban` |  |  |  | `hr_expense` |
| `hr_expense.hr_expense_kanban_view_header` | xpath | `hr_expense_view_expenses_analysis_kanban` |  |  |  | `hr_expense` |
| `hr_expense.hr_expense_view_pivot` | pivot |  | `employee_id`, `date`, `total_amount_currency` |  |  | `hr_expense` |
| `hr_expense.hr_expense_view_graph` | graph |  | `price_unit`, `quantity`, `date`, `employee_id`, `total_amount`, `tax_amount` |  |  | `hr_expense` |
| `hr_expense.hr_expense_view_search` | search |  | `name`, `department_id`, `company_id`, `employee_id` |  | `My Expenses`, `My Team`, `Paid by Company`, `Paid by Employee`, `To Submit`, `Waiting Approval`, `To Post`, `Waiting Reimbursement`, `Expense Date`, `All to Post`, `All to Pay`, `All Paid`, `My Activities`, `Late Activities`, `Today Activities`, `Future Activities`, `Employee`, `Category`, `Status`, `Expense Date`, `Company`, `Department` | `hr_expense` |
| `hr_expense.hr_expense_view_activity` | activity |  | `employee_id`, `currency_id`, `name`, `total_amount_currency` |  |  | `hr_expense` |
| `hr_expense.hr_expense_view_search_with_panel` | xpath | `hr_expense_view_search` | `state`, `employee_id`, `company_id` |  |  | `hr_expense` |
| `sale_expense.hr_expense_form_view_inherit_sale_expense` | button | `hr_expense.hr_expense_view_form` | `sale_order_id` | `action_open_account_move`, `action_open_sale_order` |  | `sale_expense` |
| `sale_expense.hr_expense_tree_view_inherit_sale_expense` | xpath | `hr_expense.view_expenses_tree` | `sale_order_id`, `can_be_reinvoiced` |  |  | `sale_expense` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `hr_expense.hr_expense_actions_my_all` | My Expenses | list,kanban,form,graph,pivot,activity |  | `{'search_default_my_open_expenses': 1}` |  | `hr_expense` |
| `hr_expense.hr_expense_actions_to_process` | Expenses to Process | list,kanban,form,graph,pivot,activity |  | `{'searchpanel_default_state': ['submitted']}` |  | `hr_expense` |
| `hr_expense.hr_expense_actions_all` | Expenses Analysis | graph,pivot,list,form |  | `{ 'searchpanel_default_state': ["draft", "submitted", "approved", "posted", "in_payment", "paid"] }` |  | `hr_expense` |
| `hr_expense.action_hr_expense_account` | Employee Expenses | list,kanban,form,pivot,graph | `[]` | `{                 'search_default_all_approved': 1,                 'search_default_all_to_pay': 1,             }` |  | `hr_expense` |
| `hr_expense.action_hr_expense_department_to_approve` | Expense to Approve | list,kanban,form,pivot,graph | `[('department_id', '=', active_id)]` | `{ 'searchpanel_default_state': ["submitted"] }` |  | `hr_expense` |
| `hr_expense.action_hr_expense_department_filtered` | Expense Analysis | graph,pivot |  | `{                 'search_default_department_id': [active_id],                 'default_department_id': active_id}` |  | `hr_expense` |
| `sale_expense.hr_expense_action_from_sale_order` | Expenses | list,form | `[('sale_order_id', '=', active_id)]` | `{'default_sale_order_id': active_id}` |  | `sale_expense` |

## Printable reports

| Report | Name | Type | Template | File name rule | Attachment rule |
|---|---|---|---|---|---|
| `hr_expense.action_report_hr_expense` | Expenses Report | qweb-pdf | `hr_expense.report_expense` | `'Expense - %s - %s' % (object.employee_id.name, (object.name).replace('/', ''))` |  |
| `hr_expense.action_report_expense_img` | Expense Report Image | qweb-pdf | `hr_expense.report_expense_img` |  |  |

## Scheduled jobs

| Job | Name | Every | Operation | Priority |
|---|---|---|---|---|
| `hr_expense.ir_cron_send_submitted_expenses_mail` | HR Expense: Send Submitted Expenses Mail | 1 weeks | `_cron_send_submitted_expenses_mail` |  |

Machine-readable definition: `../../../schemas/data/entities/hr.expense.json`; views: `../../../schemas/interfaces/views/hr.expense.json`.
