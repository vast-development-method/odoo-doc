# Account Lock Exception (`account.lock_exception`)

**Transport name:** `account.lock_exception`  
**Storage name:** `account_lock_exception`  
**Kind:** persistent entity (one table)  
**Defined by package:** `account`

Description: Account Lock Exception

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (13)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `active` | Active | boolean |  | default `True` |
| `state` | State | selection |  | computed by rule `_compute_state` (not stored); searchable through a search rule |
| `company_id` | Company | many to one | `res.company` | required; read only; default computed dynamically (lambda self: self.env.company) |
| `user_id` | User | many to one | `res.users` | default computed dynamically (lambda self: self.env.user) |
| `reason` | Reason | single line text |  |  |
| `end_datetime` | End Date | date and time |  |  |
| `lock_date_field` | Lock Date Field | selection |  | required; Help: Technical field identifying the changed lock date |
| `lock_date` | Changed Lock Date | date |  | Help: Technical field giving the date the lock date was changed to. |
| `company_lock_date` | Original Lock Date | date |  | not copied on duplication; Help: Technical field giving the date the company lock date at the time the exception was created. |
| `fiscalyear_lock_date` | Global Lock Date | date |  | computed by rule `_compute_lock_dates` (not stored); searchable through a search rule; Help: The date the Global Lock Date is set to by this exception. If the lock date is not changed it is set to the maximal date. |
| `tax_lock_date` | Tax Return Lock Date | date |  | computed by rule `_compute_lock_dates` (not stored); searchable through a search rule; Help: The date the Tax Lock Date is set to by this exception. If the lock date is not changed it is set to the maximal date. |
| `sale_lock_date` | Sales Lock Date | date |  | computed by rule `_compute_lock_dates` (not stored); searchable through a search rule; Help: The date the Sale Lock Date is set to by this exception. If the lock date is not changed it is set to the maximal date. |
| `purchase_lock_date` | Purchase Lock Date | date |  | computed by rule `_compute_lock_dates` (not stored); searchable through a search rule; Help: The date the Purchase Lock Date is set to by this exception. If the lock date is not changed it is set to the maximal date. |

## Selection values

### `state` (State)

| Value | Label |
|---|---|
| `active` | Active |
| `revoked` | Revoked |
| `expired` | Expired |

### `lock_date_field` (Lock Date Field)

| Value | Label |
|---|---|
| `fiscalyear_lock_date` | Global Lock Date |
| `tax_lock_date` | Tax Return Lock Date |
| `sale_lock_date` | Sales Lock Date |
| `purchase_lock_date` | Purchase Lock Date |

## State fields

State machine fields of this entity: `state`. Transitions are specified in the domain documents.

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_company_id_end_datetime_idx` | Index | `(company_id, user_id, end_datetime) WHERE active IS TRUE` |  | `account` |

## Operations (17)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_display_name` | computation | self | `account` |  |  |
| `_compute_state` | computation | self | `account` | depends: `active`, `end_datetime` |  |
| `_compute_lock_dates` | computation | self | `account` | depends: `lock_date_field`, `lock_date` |  |
| `_search_state` | search rule | self, operator, value | `account` |  |  |
| `_search_lock_date` | search rule | self, field, operator, value | `account` |  |  |
| `_search_fiscalyear_lock_date` | search rule | self, operator, value | `account` |  |  |
| `_search_tax_lock_date` | search rule | self, operator, value | `account` |  |  |
| `_search_sale_lock_date` | search rule | self, operator, value | `account` |  |  |
| `_search_purchase_lock_date` | search rule | self, operator, value | `account` |  |  |
| `_invalidate_affected_user_lock_dates` | internal rule | self | `account` |  |  |
| `create` | lifecycle override | self, vals_list | `account` | model_create_multi |  |
| `copy` | lifecycle override | self, default | `account` |  |  |
| `_recreate` | internal rule | self | `account` |  | 1. Copy all exceptions in self but update the company lock date. 2. Revoke all exceptions in self. 3. Return the new records from step 1. |
| `action_revoke` | user action | self | `account` |  | Revokes an active exception. |
| `_get_active_exceptions_domain` | preparation rule | self, company, soft_lock_date_fields | `account` | model |  |
| `_get_audit_trail_during_exception_domain` | preparation rule | self | `account` |  |  |
| `action_show_audit_trail_during_exception` | user action | self | `account` |  |  |

## Validation and error messages (3)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `create` | ValidationError | A single exception must change exactly one lock date field. | `account` |
| `copy` | UserError | You cannot duplicate a Lock Date Exception. | `account` |
| `action_revoke` | UserError | You cannot revoke Lock Date Exceptions. Ask someone with the 'Adviser' role. | `account` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `account` |
| `account.group_account_manager` | yes | yes | no | no | `account` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `account.view_account_lock_exception_form` | form |  | `active`, `state`, `display_name`, `create_uid`, `user_id`, `reason`, `create_date`, `end_datetime`, `lock_date_field`, `lock_date`, `company_lock_date` | `Revoke`, `action_show_audit_trail_during_exception` |  | `account` |

Machine-readable definition: `../../../schemas/data/entities/account.lock_exception.json`; views: `../../../schemas/interfaces/views/account.lock_exception.json`.
