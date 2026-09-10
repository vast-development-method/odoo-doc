# Task Recurrence (`project.task.recurrence`)

**Transport name:** `project.task.recurrence`  
**Storage name:** `project_task_recurrence`  
**Kind:** persistent entity (one table)  
**Defined by package:** `project`  
**Extended by packages:** `sale_project`

Description: Task Recurrence

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (5)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `task_ids` | Task | one to many | `project.task` | not copied on duplication; inverse field `recurrence_id` |
| `repeat_interval` | Repeat Every | integer |  | default `1` |
| `repeat_unit` | Repeat Unit | selection |  | default `week` |
| `repeat_type` | Until | selection |  | default `forever` |
| `repeat_until` | End Date | date |  |  |

## Selection values

### `repeat_unit` (Repeat Unit)

| Value | Label |
|---|---|
| `day` | Days |
| `week` | Weeks |
| `month` | Months |
| `year` | Years |

### `repeat_type` (Until)

| Value | Label |
|---|---|
| `forever` | Forever |
| `until` | Until |

## Operations (8)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_check_repeat_interval` | validation | self | `project` | constrains: `repeat_interval` |  |
| `_check_repeat_until_date` | validation | self | `project` | constrains: `repeat_type`, `repeat_until` |  |
| `_get_recurring_fields_to_copy` | preparation rule | self | `project`, `sale_project` | model |  |
| `_get_recurring_fields_to_postpone` | preparation rule | self | `project` | model |  |
| `_get_last_task_id_per_recurrence_id` | preparation rule | self | `project` |  |  |
| `_get_recurrence_delta` | preparation rule | self | `project` |  |  |
| `_create_next_occurrences` | internal rule | self, occurrences_from | `project` | model |  |
| `_create_next_occurrences_values` | internal rule | self, recurrence_by_task | `project` | model |  |

## Validation and error messages (2)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_repeat_interval` | ValidationError | The interval should be greater than 0 | `project` |
| `_check_repeat_until_date` | ValidationError | The end date should be in the future | `project` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `project.group_project_user` | yes | yes | yes | yes | `project` |

Machine-readable definition: `../../../schemas/data/entities/project.task.recurrence.json`.
