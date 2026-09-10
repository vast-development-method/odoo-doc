# Declaration of Intent (`l10n_it_edi_doi.declaration_of_intent`)

**Transport name:** `l10n_it_edi_doi.declaration_of_intent`  
**Storage name:** `l10n_it_edi_doi_declaration_of_intent`  
**Kind:** persistent entity (one table)  
**Defined by package:** `l10n_it_edi_doi`

Description: Declaration of Intent

## Identity and behavior

- Mixins (classical inheritance): `mail.thread.main.attachment`, `mail.activity.mixin`
- Default ordering: `protocol_number_part1, protocol_number_part2`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (15)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `state` | State | selection |  | required; read only; default `draft`; changes are tracked in the message thread; Help: The state of this Declaration of Intent.  - 'Draft' means that the Declaration of Intent still needs to be confirmed before being usable.  - 'Active' means that the Declaration of Intent is usable.  - 'Terminated' designates that the Declaration of Intent has been marked as not to use anymore without invalidating usages of it.  - 'Revoked' means the Declaration of Intent should not have been used. You will probably need to revert previous usages of it, if any. |
| `company_id` | Company | many to one | `res.company` | required; default computed dynamically (lambda self: self.env.company._accessible_branches()[:1]); indexed |
| `partner_id` | Partner | many to one | `res.partner` | required; indexed; restricted by domain `['\|', ('is_company', '=', True), ('parent_id', '=', False)]` |
| `currency_id` | Currency | many to one | `res.currency` | required; read only; default computed dynamically (lambda self: self.env.ref('base.EUR', raise_if_not_found=False).id) |
| `issue_date` | Date of Issue | date |  | required; default computed dynamically (fields.Date.context_today); not copied on duplication; Help: Date on which the Declaration of Intent was issued |
| `start_date` | Start Date | date |  | required; not copied on duplication; Help: First date on which the Declaration of Intent is valid |
| `end_date` | End Date | date |  | required; not copied on duplication; Help: Last date on which the Declaration of Intent is valid |
| `threshold` | Threshold | monetary |  | required; Help: Total amount of allowed sales without VAT under this Declaration of Intent |
| `invoiced` | Invoiced | monetary |  | read only; computed by rule `_compute_invoiced` and stored; Help: Total amount of sales under this Declaration of Intent |
| `not_yet_invoiced` | Not Yet Invoiced | monetary |  | read only; computed by rule `_compute_not_yet_invoiced` and stored; Help: Total amount of planned sales under this Declaration of Intent (i.e. current quotation and sales orders) that can still be invoiced |
| `remaining` | Remaining | monetary |  | read only; computed by rule `_compute_remaining` and stored; Help: Remaining amount after deduction of the Invoiced and Not Yet Invoiced amounts. |
| `protocol_number_part1` | Protocol 1 | single line text |  | required; not copied on duplication |
| `protocol_number_part2` | Protocol 2 | single line text |  | required; not copied on duplication |
| `invoice_ids` | Invoices / Refunds | one to many | `account.move` | read only; not copied on duplication; inverse field `l10n_it_edi_doi_id` |
| `sale_order_ids` | Sales Orders / Quotations | one to many | `sale.order` | read only; not copied on duplication; inverse field `l10n_it_edi_doi_id` |

## Selection values

### `state` (State)

| Value | Label |
|---|---|
| `draft` | Draft |
| `active` | Active |
| `revoked` | Revoked |
| `terminated` | Terminated |

## State fields

State machine fields of this entity: `state`. Transitions are specified in the domain documents.

## Database constraints and indexes (2)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_protocol_number_unique` | Constraint | `unique(protocol_number_part1, protocol_number_part2)` | The Protocol Number of a Declaration of Intent must be unique! Please choose another one. | `l10n_it_edi_doi` |
| `_threshold_positive` | Constraint | `CHECK(threshold > 0)` | The Threshold of a Declaration of Intent must be positive. | `l10n_it_edi_doi` |

## Operations (16)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_display_name` | computation | self | `l10n_it_edi_doi` | depends: `protocol_number_part1`, `protocol_number_part2` |  |
| `_compute_invoiced` | computation | self | `l10n_it_edi_doi` | depends: `invoice_ids`, `invoice_ids.state`, `invoice_ids.l10n_it_edi_doi_amount` |  |
| `_compute_not_yet_invoiced` | computation | self | `l10n_it_edi_doi` | depends: `sale_order_ids`, `sale_order_ids.state`, `sale_order_ids.l10n_it_edi_doi_not_yet_invoiced` |  |
| `_compute_remaining` | computation | self | `l10n_it_edi_doi` | depends: `threshold`, `not_yet_invoiced`, `invoiced` |  |
| `_build_threshold_warning_message` | internal rule | self, invoiced, not_yet_invoiced | `l10n_it_edi_doi` |  | Build a warning message that will be displayed in a yellow banner on top of a document if the `remaining` of the Declaration of Intent is less than 0 when including the document or the Declaration of Intent is revoked     :param float invoiced:          The `declaration.invoiced` amount when including the document.     :param float not_yet_invoiced:  The `declaration.not_yet_invoiced` amount when including the document.     :return str:                    The warning message to be shown. |
| `_get_validity_errors` | preparation rule | self, company, partner, currency | `l10n_it_edi_doi` |  | Check whether all declarations of intent in self are valid for the specified `company`, `partner`, `date` and `currency'. Violating these constraints leads to errors in the feature. They should not be ignored. Return all errors as a list of strings. |
| `_get_validity_warnings` | preparation rule | self, company, partner, currency, date, invoiced_amount, only_blocking, sales_order | `l10n_it_edi_doi` |  | Check whether all declarations of intent in self are valid for the specified `company`, `partner`, `date` and `currency'. The checks for `date` and state of the declaration (except draft) are not considered blocking in case `invoiced_amount` is not positive. All other checks are considered blocking (prevent posting). Includes all checks from `_get_validity_errors`. The checks are different for invoices and sales orders (toggled via kwarg `sales_order`). I.e. we do not care about the date for sales orders. Return all errors as a list of strings. |
| `_fetch_valid_declaration_of_intent` | internal rule | self, company, partner, currency, date | `l10n_it_edi_doi` | model | Fetch a declaration of intent that is valid for the specified `company`, `partner`, `date` and `currency` and has not reached the threshold yet. |
| `_unlink_except_linked_to_document` | internal rule | self | `l10n_it_edi_doi` | ondelete |  |
| `action_validate` | user action | self | `l10n_it_edi_doi` |  | Move a 'draft' Declaration of Intent to 'active'. |
| `action_reset_to_draft` | user action | self | `l10n_it_edi_doi` |  | Resets an 'active' Declaration of Intent back to 'draft'. |
| `action_reactivate` | user action | self | `l10n_it_edi_doi` |  | Resets a not 'active' Declaration of Intent back to 'active'. |
| `action_revoke` | user action | self | `l10n_it_edi_doi` |  | Called by the 'revoke' button of the form view. |
| `action_terminate` | user action | self | `l10n_it_edi_doi` |  | Called by the 'terminated' button of the form view. |
| `action_open_sale_order_ids` | user action | self | `l10n_it_edi_doi` |  |  |
| `action_open_invoice_ids` | user action | self | `l10n_it_edi_doi` |  |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_unlink_except_linked_to_document` | UserError | You cannot delete Declarations of Intents that are already used on at least one Invoice or Sales Order. | `l10n_it_edi_doi` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_readonly` | no | yes | no | no | `l10n_it_edi_doi` |
| `account.group_account_invoice` | yes | yes | yes | yes | `l10n_it_edi_doi` |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `l10n_it_edi_doi.view_l10n_it_edi_doi_tree` | list |  | `currency_id`, `partner_id`, `company_id`, `protocol_number_part1`, `protocol_number_part2`, `issue_date`, `start_date`, `end_date`, `threshold`, `not_yet_invoiced`, `invoiced`, `remaining`, `state` |  |  | `l10n_it_edi_doi` |
| `l10n_it_edi_doi.view_l10n_it_edi_doi_form` | form |  | `state`, `invoice_ids`, `sale_order_ids`, `partner_id`, `company_id`, `protocol_number_part1`, `protocol_number_part2`, `issue_date`, `start_date`, `end_date`, `currency_id`, `threshold`, `not_yet_invoiced`, `invoiced`, `remaining` | `Validate`, `Terminate`, `Reset to Draft`, `Reactivate`, `Revoke`, `action_open_invoice_ids`, `action_open_sale_order_ids` |  | `l10n_it_edi_doi` |
| `l10n_it_edi_doi.view_l10n_it_edi_doi_declaration_of_intent_search` | search |  | `protocol_number_part1`, `protocol_number_part2` |  | `Active`, `Draft`, `Terminated / Revoked` | `l10n_it_edi_doi` |

Machine-readable definition: `../../../schemas/data/entities/l10n_it_edi_doi.declaration_of_intent.json`; views: `../../../schemas/interfaces/views/l10n_it_edi_doi.declaration_of_intent.json`.
