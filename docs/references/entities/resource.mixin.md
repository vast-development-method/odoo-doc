# Resource Mixin (`resource.mixin`)

**Transport name:** `resource.mixin`  
**Storage name:** `resource_mixin`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `resource`

Description: Resource Mixin

## Fields (4)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `resource_id` | Resource | many to one | `resource.resource` | required; indexed; on delete of the target: restrict |
| `company_id` | Company | many to one | `res.company` | related through path `resource_id.company_id` and stored; default computed dynamically (lambda self: self.env.company); indexed; precomputed before insertion |
| `resource_calendar_id` | Working Hours | many to one | `resource.calendar` | related through path `resource_id.calendar_id` and stored; default computed dynamically (lambda self: self.env.company.resource_calendar_id); indexed |
| `tz` | Timezone | selection |  | related through path `resource_id.tz`; Help: This field is used in order to define in which timezone the resources will work. |

## Operations (9)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `create` | lifecycle override | self, vals_list | `resource` | model_create_multi |  |
| `_prepare_resource_values` | preparation rule | self, vals, tz | `resource` |  |  |
| `copy_data` | lifecycle override | self, default | `resource` |  |  |
| `_get_calendars` | preparation rule | self, date_from | `resource` |  |  |
| `_get_work_days_data_batch` | preparation rule | self, from_datetime, to_datetime, compute_leaves, calendar, domain | `resource` |  | By default the resource calendar is used, but it can be changed using the `calendar` argument.  `domain` is used in order to recognise the leaves to take, None means default value ('time_type', '=', 'leave')  Returns a dict {'days': n, 'hours': h} containing the quantity of working time expressed as days and as hours. |
| `_get_leave_days_data_batch` | preparation rule | self, from_datetime, to_datetime, calendar, domain | `resource` |  | By default the resource calendar is used, but it can be changed using the `calendar` argument.  `domain` is used in order to recognise the leaves to take, None means default value ('time_type', '=', 'leave')  Returns a dict {'days': n, 'hours': h} containing the number of leaves expressed as days and as hours. |
| `_adjust_to_calendar` | internal rule | self, start, end | `resource` |  |  |
| `_list_work_time_per_day` | internal rule | self, from_datetime, to_datetime, calendar, domain | `resource` |  | By default the resource calendar is used, but it can be changed using the `calendar` argument.  `domain` is used in order to recognise the leaves to take, None means default value ('time_type', '=', 'leave')  Returns a list of tuples (day, hours) for each day containing at least an attendance. |
| `list_leaves` | operation | self, from_datetime, to_datetime, calendar, domain | `resource` |  | By default the resource calendar is used, but it can be changed using the `calendar` argument.  `domain` is used in order to recognise the leaves to take, None means default value ('time_type', '=', 'leave')  Returns a list of tuples (day, hours, resource.calendar.leaves) for each leave in the calendar. |

Machine-readable definition: `../../../schemas/data/entities/resource.mixin.json`.
