# Preset to create journal entries during a invoices and payments matching (`account.reconcile.model`)

**Transport name:** `account.reconcile.model`  
**Storage name:** `account_reconcile_model`  
**Kind:** persistent entity (one table)  
**Defined by package:** `account`

Description: Preset to create journal entries during a invoices and payments matching

## Identity and behavior

- Mixins (classical inheritance): `mail.thread`
- Default ordering: `sequence, id`
- Company consistency is checked automatically on company-bound relations
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (16)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `active` | Active | boolean |  | default `True` |
| `name` | Name | single line text |  | required; translatable |
| `sequence` | Sequence | integer |  | required; default `10` |
| `company_id` | Company | many to one | `res.company` | required; read only; default computed dynamically (lambda self: self.env.company) |
| `trigger` | Trigger | selection |  | required; default `manual`; changes are tracked in the message thread; Help: Validate the statement line automatically (reconciliation based on your rule). |
| `next_activity_type_id` | Next Activity | many to one | `mail.activity.type` |  |
| `can_be_proposed` | Can Be Proposed | boolean |  | computed by rule `_compute_can_be_proposed` and stored; not copied on duplication |
| `mapped_partner_id` | Mapped Partner | many to one | `res.partner` | computed by rule `_compute_partner_mapping` and stored; not copied on duplication |
| `match_journal_ids` | Journals | many to many | `account.journal` | restricted by domain `[('type', 'in', ('bank', 'cash', 'credit'))]`; must belong to the same company; Help: The reconciliation model will only be available from the selected journals. |
| `match_amount` | Amount | selection |  | changes are tracked in the message thread; Help: The reconciliation model will only be applied when the amount being lower than, greater than or between specified amount(s). |
| `match_amount_min` | Amount Min Parameter | float |  | changes are tracked in the message thread |
| `match_amount_max` | Amount Max Parameter | float |  | changes are tracked in the message thread |
| `match_label` | Label | selection |  | changes are tracked in the message thread; Help: The reconciliation model will only be applied when either the statement line label, the transaction details or the note matches the following:         * Contains: The statement line must contains this string (case insensitive).         * Not Contains: Negation of "Contains".         * Match Regex: Define your own regular expression. |
| `match_label_param` | Label Parameter | single line text |  | changes are tracked in the message thread |
| `match_partner_ids` | Partners | many to many | `res.partner` | Help: The reconciliation model will only be applied to the selected customers/vendors. |
| `line_ids` | Line | one to many | `account.reconcile.model.line` | inverse field `model_id` |

## Selection values

### `trigger` (Trigger)

| Value | Label |
|---|---|
| `manual` | Manual |
| `auto_reconcile` | Automated |

### `match_amount` (Amount)

| Value | Label |
|---|---|
| `lower` | Is lower than or equal to |
| `greater` | Is greater than or equal to |
| `between` | Is between |

### `match_label` (Label)

| Value | Label |
|---|---|
| `contains` | Contains |
| `not_contains` | Not Contains |
| `match_regex` | Match Regex |

## Operations (7)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_check_match_label_param` | validation | self | `account` | constrains: `match_label`, `match_label_param` |  |
| `_compute_can_be_proposed` | computation | self | `account` | depends: `mapped_partner_id`, `match_label`, `match_amount`, `match_partner_ids`, `trigger` |  |
| `_compute_partner_mapping` | computation | self | `account` | depends: `match_label`, `line_ids.partner_id`, `line_ids.account_id` |  |
| `action_set_manual` | user action | self | `account` |  |  |
| `action_set_auto_reconcile` | user action | self | `account` |  |  |
| `action_reconcile_stat` | user action | self | `account` |  |  |
| `copy_data` | lifecycle override | self, default | `account` |  |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_match_label_param` | UserError | The regex is not valid | `account` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_readonly` | no | yes | no | no | `account` |
| `account.group_account_invoice` | yes | yes | no | no | `account` |
| `account.group_account_basic` | yes | yes | yes | yes | `account` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Account reconcile model template company rule | global (all users) | `[('company_id', 'parent_of', company_ids)]` | True | True | True | True |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `account.view_account_reconcile_model_tree` | list |  | `sequence`, `name`, `trigger`, `match_journal_ids` |  |  | `account` |
| `account.view_account_reconcile_model_form` | form |  | `trigger`, `active`, `company_id`, `name`, `match_journal_ids`, `match_partner_ids`, `match_amount`, `match_amount_min`, `match_amount_max`, `match_label`, `match_label_param`, `next_activity_type_id`, `line_ids`, `company_id`, `sequence`, `partner_id`, `account_id`, `amount_type`, `amount_string`, `tax_ids`, `analytic_distribution`, `label` | `Set Manual`, `Automate`, `Journal Entries` |  | `account` |
| `account.view_account_reconcile_model_search` | search |  | `name` |  | `Automated`, `With tax`, `Archived`, `Journals Availability`, `Automation` | `account` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `account.action_account_reconcile_model` | Reconciliation Models | list,form |  |  |  | `account` |

Machine-readable definition: `../../../schemas/data/entities/account.reconcile.model.json`; views: `../../../schemas/interfaces/views/account.reconcile.model.json`.
