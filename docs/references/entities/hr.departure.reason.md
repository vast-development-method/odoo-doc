# Departure Reason (`hr.departure.reason`)

**Transport name:** `hr.departure.reason`  
**Storage name:** `hr_departure_reason`  
**Kind:** persistent entity (one table)  
**Defined by package:** `hr`

Description: Departure Reason

## Identity and behavior

- Default ordering: `sequence`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (4)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `sequence` | Sequence | integer |  | default `10` |
| `name` | Reason | single line text |  | required; translatable |
| `country_id` | Country | many to one | `res.country` | default computed dynamically (lambda self: self.env.company.country_id) |
| `country_code` | Country Code | single line text |  | related through path `country_id.code` |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_default_departure_reasons` | preparation rule | self | `hr` | model |  |
| `_unlink_except_default_departure_reasons` | internal rule | self | `hr` | ondelete |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_unlink_except_default_departure_reasons` | UserError | Default departure reasons cannot be deleted. | `hr` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_hr_user` | yes | yes | yes | yes | `hr` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Departure Reason: multi company | global (all users) | `[('country_code', 'in', user.env.companies.mapped('country_code') + [False])]` | True | True | True | True |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr.hr_departure_reason_view_list` | list |  | `sequence`, `name`, `country_code` |  |  | `hr` |
| `hr.hr_departure_reason_view_form` | form |  | `sequence`, `name`, `country_code` |  |  | `hr` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `hr.hr_departure_reason_action` | Departure Reasons | list |  |  |  | `hr` |

Machine-readable definition: `../../../schemas/data/entities/hr.departure.reason.json`; views: `../../../schemas/interfaces/views/hr.departure.reason.json`.
