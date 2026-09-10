# Activity plan template (`mail.activity.plan.template`)

**Transport name:** `mail.activity.plan.template`  
**Storage name:** `mail_activity_plan_template`  
**Kind:** persistent entity (one table)  
**Defined by package:** `mail`  
**Extended by packages:** `hr`, `hr_fleet`

Description: Activity plan template

## Identity and behavior

- Default ordering: `sequence, id`
- Display name field: `summary`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (14)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `plan_id` | Plan | many to one | `mail.activity.plan` | required; indexed; on delete of the target: cascade |
| `res_model` | Resource Model | selection |  | related through path `plan_id.res_model` |
| `company_id` | Company | many to one |  | related through path `plan_id.company_id` |
| `sequence` | Sequence | integer |  | default `10` |
| `activity_type_id` | Activity Type | many to one | `mail.activity.type` | required; default computed dynamically (lambda self: self.env.ref('mail.mail_activity_data_todo')); on delete of the target: restrict; restricted by domain `['\|', ('res_model', '=', False), '&', ('res_model', '!=', False), ('res_model', '=', parent.res_model)]` |
| `delay_count` | Interval | integer |  | default ; Help: Number of days/week/month before executing the action after or before the scheduled plan date. |
| `delay_unit` | Delay units | selection |  | required; default `days`; Help: Unit of delay |
| `delay_from` | Trigger | selection |  | required; default `before_plan_date` |
| `icon` | Icon | single line text |  | read only; related through path `activity_type_id.icon` |
| `summary` | Summary | single line text |  | computed by rule `_compute_summary` and stored |
| `responsible_type` | Assignment | selection |  | required; computed by rule `_compute_responsible_type` and stored; default `on_demand`; on delete of the target: {"fleet_manager": "set default"}; extended by packages `hr`, `hr_fleet` |
| `responsible_id` | Assigned to | many to one | `res.users` | computed by rule `_compute_responsible_id` and stored; must belong to the same company |
| `note` | Note | rich text |  | computed by rule `_compute_note` and stored |
| `next_activity_ids` | Next Activities | many to many | `mail.activity.type` | computed by rule `_compute_next_activity_ids` and stored |

## Selection values

### `delay_unit` (Delay units)

| Value | Label |
|---|---|
| `days` | days |
| `weeks` | weeks |
| `months` | months |

### `delay_from` (Trigger)

| Value | Label |
|---|---|
| `before_plan_date` | Before Plan Date |
| `after_plan_date` | After Plan Date |

### `responsible_type` (Assignment)

| Value | Label |
|---|---|
| `on_demand` | Ask at launch |
| `other` | Default user |
| `coach` | Coach |
| `manager` | Manager |
| `employee` | Employee |
| `fleet_manager` | Fleet Manager |

## Operations (12)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_check_activity_type_res_model` | validation | self | `mail` | constrains: `activity_type_id`, `plan_id` | Check that the plan models are compatible with the template activity type model. Note that it depends also on "activity_type_id.res_model" and "plan_id.res_model". That's why this method is called by those models when the mentioned fields are updated. |
| `_check_responsible` | validation | self | `mail` | constrains: `responsible_id`, `responsible_type` | Ensure that responsible_id is set when responsible is set to "other". |
| `_compute_next_activity_ids` | computation | self | `mail` | depends: `activity_type_id` | Update next activities only when changing activity type on template. Any change on type configuration should not be propagated. |
| `_compute_note` | computation | self | `mail` | depends: `activity_type_id` |  |
| `_compute_responsible_id` | computation | self | `mail` | depends: `activity_type_id`, `responsible_type` |  |
| `_compute_responsible_type` | computation | self | `mail` | depends: `activity_type_id` |  |
| `_compute_summary` | computation | self | `mail` | depends: `activity_type_id` |  |
| `_get_date_deadline` | preparation rule | self, base_date | `mail` |  | Return the deadline of the activity to be created given the base date. |
| `_determine_responsible` | internal rule | self, on_demand_responsible, applied_on_record | `hr_fleet`, `hr`, `mail` |  | Determine the responsible for the activity based on the template for the given record and on demand responsible.  Based on the responsible_type, this method will determine the responsible to set on the activity for the given record (applied_on_record). Following the responsible_type: - on_demand: on_demand_responsible is used as responsible (allow to set it when using the template) - other: the responsible field is used (preset user at the template level)  Other module can extend it and base the responsible on the record on which the activity will be set. Ex.: 'coach' on employee record will a |
| `_check_responsible_hr` | validation | self | `hr` | constrains: `plan_id`, `responsible_type` | Ensure that hr types are used only on employee model |
| `_get_closest_parent_user` | preparation rule | self, employee, responsible, error_message | `hr` |  |  |
| `_check_responsible_hr_fleet` | validation | self | `hr_fleet` | constrains: `plan_id`, `responsible_type` | Ensure that hr types are used only on employee model |

## Validation and error messages (4)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_activity_type_res_model` | ValidationError | The activity type "%(activity_type_name)s" is not compatible with the plan "%(plan_name)s" because it is limited to the model "%(activity_type_model)s". | `mail` |
| `_check_responsible` | ValidationError | When selecting "Default user" assignment, you must specify a responsible. | `mail` |
| `_check_responsible_hr` | ValidationError | Those responsible types are limited to Employee plans. | `hr` |
| `_check_responsible_hr_fleet` | ValidationError | Fleet Manager is limited to Employee plans. | `hr_fleet` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `sales_team.group_sale_manager` | yes | yes | yes | yes | `crm` |
| `group_hr_manager` | yes | yes | yes | yes | `hr` |
| `hr_recruitment.group_hr_recruitment_manager` | yes | yes | yes | yes | `hr_recruitment` |
| `base.group_user` | no | yes | no | no | `mail` |
| `base.group_system` | yes | yes | yes | yes | `mail` |
| `project.group_project_manager` | yes | yes | yes | yes | `project` |
| `sales_team.group_sale_manager` | yes | yes | yes | yes | `sale` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Manager can manage lead plan templates | `[(4, ref('sales_team.group_sale_manager'))]` | `[('plan_id.res_model', '=', 'crm.lead')]` | False | True | True | True |
| Manager can edit employee plan template | `[(4, ref('group_hr_manager'))]` | `[('plan_id.res_model', '=', 'hr.employee')]` | False | True | True | True |
| Manager can manage applicant plan templates | `[(4, ref('hr_recruitment.group_hr_recruitment_manager'))]` | `[('plan_id.res_model', '=', 'hr.applicant')]` | False | True | True | True |
| Administrators can access all activity plan templates. | `[Command.link(ref('base.group_system'))]` | `[(1, '=', 1)]` | True | True | True | True |
| Manager can manage project/task plan templates | `[(4, ref('group_project_manager'))]` | `[('plan_id.res_model', 'in', ('project.project', 'project.task'))]` | False | True | True | True |
| Manager can manage sale order plan templates | `[(4, ref('sales_team.group_sale_manager'))]` | `[('plan_id.res_model', '=', 'sale.order')]` | False | True | True | True |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr.mail_activity_plan_template_view_form` | xpath | `mail.mail_activity_plan_template_view_form` | `responsible_type`, `responsible_type` |  |  | `hr` |
| `mail.mail_activity_plan_template_view_tree` | list |  | `activity_type_id`, `summary`, `responsible_type`, `delay_count`, `delay_unit`, `delay_from` |  |  | `mail` |
| `mail.mail_activity_plan_template_view_form` | form |  | `company_id`, `res_model`, `activity_type_id`, `summary`, `responsible_type`, `responsible_id`, `delay_count`, `delay_unit`, `delay_from`, `note` |  |  | `mail` |

Machine-readable definition: `../../../schemas/data/entities/mail.activity.plan.template.json`; views: `../../../schemas/interfaces/views/mail.activity.plan.template.json`.
