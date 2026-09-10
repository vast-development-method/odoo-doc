# Cancel Time Off Wizard (`hr.holidays.cancel.leave`)

**Transport name:** `hr.holidays.cancel.leave`  
**Storage name:** `hr_holidays_cancel_leave`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `hr_holidays`

Description: Cancel Time Off Wizard

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `leave_id` | Time Off Request | many to one | `hr.leave` | required |
| `reason` | Reason | multi line text |  |  |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `action_cancel_leave` | user action | self | `hr_holidays` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | yes | yes | yes | yes | `hr_holidays` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr_holidays.hr_holidays_cancel_leave_form` | form |  | `leave_id`, `reason` | `Cancel Time Off`, `Discard` |  | `hr_holidays` |

Machine-readable definition: `../../../schemas/data/entities/hr.holidays.cancel.leave.json`; views: `../../../schemas/interfaces/views/hr.holidays.cancel.leave.json`.
