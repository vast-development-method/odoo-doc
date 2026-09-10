# Payment Methods (`account.payment.method.line`)

**Transport name:** `account.payment.method.line`  
**Storage name:** `account_payment_method_line`  
**Kind:** persistent entity (one table)  
**Defined by package:** `account`  
**Extended by packages:** `account_payment`, `l10n_it_edi`

Description: Payment Methods

## Identity and behavior

- Default ordering: `sequence, id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (13)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | computed by rule `_compute_name` and stored |
| `sequence` | Sequence | integer |  | default `10` |
| `payment_method_id` | Payment Method | many to one | `account.payment.method` | required; restricted by domain `[('payment_type', '=?', payment_type), ('id', 'in', available_payment_method_ids)]` |
| `payment_account_id` | Payment Account | many to one | `account.account` | not copied on duplication; on delete of the target: restrict; restricted by domain `['\|', ('account_type', 'in', ('asset_current', 'liability_current')), ('id', '=', default_account_id)]`; must belong to the same company |
| `journal_id` | Journal | many to one | `account.journal` | indexed (btree_not_null); must belong to the same company |
| `default_account_id` | Default Account | many to one |  | related through path `journal_id.default_account_id` |
| `code` | Code | single line text |  | related through path `payment_method_id.code` |
| `payment_type` | Payment Type | selection |  | related through path `payment_method_id.payment_type` |
| `company_id` | Company | many to one |  | related through path `journal_id.company_id` |
| `available_payment_method_ids` | Available Payment Method | many to many |  | related through path `journal_id.available_payment_method_ids` |
| `payment_provider_id` | Payment Provider | many to one | `payment.provider` | computed by rule `_compute_payment_provider_id` and stored; restricted by domain `[('code', '=', code)]` |
| `payment_provider_state` | Payment Provider State | selection |  | related through path `payment_provider_id.state` |
| `l10n_it_payment_method` | Italian Payment Method | selection |  | default `MP05` |

## State fields

State machine fields of this entity: `payment_provider_state`. Transitions are specified in the domain documents.

## Operations (9)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_display_name` | computation | self | `account` | depends: `journal_id`; depends_context: `hide_payment_journal_id` |  |
| `_compute_name` | computation | self | `account_payment`, `account` | depends: `payment_method_id.name`; depends: `payment_provider_id.name` |  |
| `_ensure_unique_name_for_journal` | validation | self | `account` | constrains: `name` |  |
| `unlink` | lifecycle override | self | `account` |  | Payment method lines which are used in a payment should not be deleted from the database, only the link betweend them and the journals must be broken. |
| `_auto_toggle_account_to_reconcile` | internal rule | self, account_id | `account` | model | This method is deprecated and will be removed. Automatically toggle the account to reconcile if allowed.  :param account_id: The id of an account.account. |
| `_compute_payment_provider_id` | computation | self | `account_payment` | depends: `payment_method_id` |  |
| `_unlink_except_active_provider` | internal rule | self | `account_payment` | ondelete | Ensure we don't remove an account.payment.method.line that is linked to a provider in the test or enabled state. |
| `action_open_provider_form` | user action | self | `account_payment` |  |  |
| `_get_l10n_it_payment_method_selection_code` | preparation rule | self | `l10n_it_edi` |  |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_unlink_except_active_provider` | UserError | You can't delete a payment method that is linked to a provider in the enabled or test state. Linked providers(s): %s | `account_payment` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `account` |
| `account.group_account_invoice` | yes | yes | yes | yes | `account` |
| `group_pos_manager` | no | yes | no | no | `point_of_sale` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `account.view_account_payment_method_line_tree` | list |  | `name`, `journal_id` |  |  | `account` |
| `account.view_account_payment_method_line_kanban_mobile` | kanban |  | `display_name` |  |  | `account` |

Machine-readable definition: `../../../schemas/data/entities/account.payment.method.line.json`; views: `../../../schemas/interfaces/views/account.payment.method.line.json`.
