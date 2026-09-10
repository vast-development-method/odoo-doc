# Work Location (`hr.work.location`)

**Transport name:** `hr.work.location`  
**Storage name:** `hr_work_location`  
**Kind:** persistent entity (one table)  
**Defined by package:** `hr`  
**Extended by packages:** `hr_homeworking`

Description: Work Location

## Identity and behavior

- Default ordering: `name`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (6)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `active` | Active | boolean |  | default `True` |
| `name` | Work Location | single line text |  | required |
| `company_id` | Company | many to one | `res.company` | required; default computed dynamically (lambda self: self.env.company) |
| `location_type` | Cover Image | selection |  | required; default `office` |
| `address_id` | Work Address | many to one | `res.partner` | required; must belong to the same company |
| `location_number` | Location Number | single line text |  |  |

## Selection values

### `location_type` (Cover Image)

| Value | Label |
|---|---|
| `home` | Home |
| `office` | Office |
| `other` | Other |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_unlink_except_used_by_employee` | internal rule | self | `hr_homeworking` | ondelete |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_unlink_except_used_by_employee` | UserError | You cannot delete locations that are being used by your employees | `hr_homeworking` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `hr` |
| `group_hr_manager` | yes | yes | yes | yes | `hr` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr.hr_work_location_tree_view` | list |  | `active`, `name`, `location_type`, `company_id` |  |  | `hr` |
| `hr.hr_work_location_form_view` | form |  | `active`, `name`, `address_id`, `location_type`, `company_id`, `company_id` |  |  | `hr` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `hr.hr_work_location_action` | Work Locations | list,form |  |  |  | `hr` |

Machine-readable definition: `../../../schemas/data/entities/hr.work.location.json`; views: `../../../schemas/interfaces/views/hr.work.location.json`.
