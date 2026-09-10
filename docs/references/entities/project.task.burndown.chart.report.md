# Burndown Chart (`project.task.burndown.chart.report`)

**Transport name:** `project.task.burndown.chart.report`  
**Storage name:** `project_task_burndown_chart_report`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `project`

Description: Burndown Chart

## Identity and behavior

- Default ordering: `date`

## Fields (13)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `allocated_hours` | Allocated Time | float |  | read only |
| `date` | Date | date and time |  | read only |
| `date_assign` | Assignment Date | date and time |  | read only |
| `date_deadline` | Deadline | date |  | read only |
| `date_last_stage_update` | Last Stage Update | date |  | read only |
| `state` | State | selection |  | read only |
| `is_closed` | Closing Stage | selection |  | read only |
| `milestone_id` | Milestone | many to one | `project.milestone` | read only |
| `partner_id` | Customer | many to one | `res.partner` | read only |
| `project_id` | Project | many to one | `project.project` | read only |
| `stage_id` | Stage | many to one | `project.task.type` | read only |
| `tag_ids` | Tags | many to many | `project.tags` | read only; association table `project_tags_project_task_rel` |
| `user_ids` | Assignees | many to many | `res.users` | read only; association table `project_task_user_rel` |

## Selection values

### `state` (State)

| Value | Label |
|---|---|
| `01_in_progress` | In Progress |
| `1_done` | Done |
| `04_waiting_normal` | Waiting |
| `03_approved` | Approved |
| `1_canceled` | Cancelled |
| `02_changes_requested` | Changes Requested |

### `is_closed` (Closing Stage)

| Value | Label |
|---|---|
| `closed` | Closed tasks |
| `open` | Open tasks |

## State fields

State machine fields of this entity: `state`. Transitions are specified in the domain documents.

## Operations (6)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `task_specific_fields` | operation | self | `project` |  |  |
| `_search` | search rule | self, domain, offset, limit, order, **kwargs | `project` | model |  |
| `_validate_group_by` | internal rule | self, groupby | `project` | model | Check that the both `date` and `stage_id` are part of `group_by`, otherwise raise a `UserError`.  :param groupby: List of group by fields. |
| `_determine_domains` | internal rule | self, domain | `project` | model | Compute two separated domain from the provided one: * A domain that only contains fields that are specific to `project.task.burndown.chart.report` * A domain that only contains fields that are specific to `project.task`  See `filter_domain_leaf` for more details on the new domains.  :param domain: The domain that has been passed to the read_group. :return: A tuple containing the non `project.task` specific domain and the `project.task` specific domain. |
| `_read_group_select` | internal rule | self, aggregate_spec, query | `project` |  |  |
| `_read_group` | lifecycle override | self, domain, groupby, aggregates, having, offset, limit, order | `project` |  |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_validate_group_by` | UserError | The view must be grouped by date and by Stage - Burndown chart or Is Closed - Burnup chart | `project` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `project.group_project_manager` | yes | yes | yes | yes | `project` |
| `project.group_project_user` | no | yes | no | no | `project` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Burndown chart: project visibility User | `[(4,ref('project.group_project_user'))]` | `[         '\|',             ('project_id.privacy_visibility', 'in', ['employees', 'portal']),             '\|',                 ('project_id.message_partner_ids', 'in', [user.partner_id.id]),                 ('user_ids', 'in', user.id),         ]` | True | True | True | True |
| Burndown chart: project visibility User | `[(4,ref('project.group_project_manager'))]` | `[(1, '=', 1)]` | True | True | True | True |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `project.project_task_burndown_chart_report_view_search` | search |  | `tag_ids`, `user_ids`, `stage_id`, `is_closed`, `project_id`, `milestone_id`, `partner_id` |  | `My Tasks`, `Unassigned`, `Date`, `filter_last_stage_update`, `filter_date_deadline`, `Last Month`, `Open Tasks`, `Closed Tasks`, `Date`, `Stage (Burndown Chart)`, `Is Closed (Burn-up Chart)` | `project` |
| `project.project_task_burndown_chart_report_view_graph` | graph |  | `date`, `stage_id`, `is_closed` |  |  | `project` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `project.action_project_task_burndown_chart_report` | Burndown Chart | graph | `[('project_id', '!=', False)]` | `{'search_default_project_id': active_id, 'search_default_date': 1, 'search_default_stage': 1, 'search_default_filter_date': 1}` |  | `project` |

Machine-readable definition: `../../../schemas/data/entities/project.task.burndown.chart.report.json`; views: `../../../schemas/interfaces/views/project.task.burndown.chart.report.json`.
