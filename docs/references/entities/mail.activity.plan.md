# Activity Plan (`mail.activity.plan`)

**Transport name:** `mail.activity.plan`  
**Storage name:** `mail_activity_plan`  
**Kind:** persistent entity (one table)  
**Defined by package:** `mail`  
**Extended by packages:** `hr`, `hr_recruitment`

Description: Activity Plan

## Identity and behavior

- Default ordering: `id DESC`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (10)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required |
| `company_id` | Company | many to one | `res.company` | default computed dynamically (lambda self: self.env.company) |
| `template_ids` | Activities | one to many | `mail.activity.plan.template` | inverse field `plan_id` |
| `active` | Active | boolean |  | default `True` |
| `res_model_id` | Applies to | many to one | `ir.model` | required; computed by rule `_compute_res_model_id` and stored; on delete of the target: cascade; precomputed before insertion |
| `res_model` | Model | selection |  | required; Help: Specify a model if the activity should be specific to a model and not available when managing activities for other models. |
| `steps_count` | Steps Count | integer |  | computed by rule `_compute_steps_count` (not stored) |
| `has_user_on_demand` | Has on demand responsible | boolean |  | computed by rule `_compute_has_user_on_demand` (not stored) |
| `department_id` | Department | many to one | `hr.department` | computed by rule `_compute_department_id` and stored; indexed (btree_not_null); on delete of the target: cascade; must belong to the same company |
| `department_assignable` | Department Assignable | boolean |  | computed by rule `_compute_department_assignable` (not stored) |

## Operations (9)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_model_selection` | preparation rule | self | `mail` |  |  |
| `_compute_res_model_id` | computation | self | `mail` | depends: `res_model` |  |
| `_check_res_model_compatibility_with_templates` | validation | self | `mail` | constrains: `res_model` |  |
| `_compute_steps_count` | computation | self | `mail` | depends: `template_ids` |  |
| `_compute_has_user_on_demand` | computation | self | `mail` | depends: `template_ids.responsible_type` |  |
| `copy_data` | lifecycle override | self, default | `mail` |  |  |
| `_check_compatibility_with_model` | validation | self | `hr` | constrains: `res_model` | Check that when the model is updated to a model different from employee, there are no remaining specific values to employee. |
| `_compute_department_assignable` | computation | self | `hr_recruitment`, `hr` | depends: `res_model` |  |
| `_compute_department_id` | computation | self | `hr` | depends: `res_model` |  |

## Validation and error messages (2)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_compatibility_with_model` | UserError | Plan %(plan_names)s cannot use a department as it is used only for some HR plans. | `hr` |
| `_check_compatibility_with_model` | UserError | Plan activities %(template_names)s cannot use coach, manager or employee responsible as it is used only for employee plans. | `hr` |

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
| Manager can manage lead plans | `[(4, ref('sales_team.group_sale_manager'))]` | `[('res_model', '=', 'crm.lead')]` | False | True | True | True |
| Manager can edit employee plan | `[(4, ref('group_hr_manager'))]` | `[('res_model', '=', 'hr.employee')]` | False | True | True | True |
| Manager can manage applicant plans | `[(4, ref('hr_recruitment.group_hr_recruitment_manager'))]` | `[('res_model', '=', 'hr.applicant')]` | False | True | True | True |
| Administrators can access all activity plans. | `[Command.link(ref('base.group_system'))]` | `[(1, '=', 1)]` | True | True | True | True |
| Manager can manage project/task plans | `[(4, ref('group_project_manager'))]` | `[('res_model', 'in', ('project.project', 'project.task'))]` | False | True | True | True |
| Manager can manage sale order plans | `[(4, ref('sales_team.group_sale_manager'))]` | `[('res_model', '=', 'sale.order')]` | False | True | True | True |

## Views (10)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr.mail_activity_plan_view_form` | xpath | `mail.mail_activity_plan_view_form` | `responsible_type` |  |  | `hr` |
| `hr.mail_activity_plan_view_form_hr_employee` | xpath | `mail.mail_activity_plan_view_form_fixed_model` |  |  |  | `hr` |
| `hr.mail_activity_plan_view_tree` | xpath | `mail.mail_activity_plan_view_tree` | `department_id` |  |  | `hr` |
| `mail.mail_activity_plan_view_search` | search |  | `name` |  | `Archived`, `Model` | `mail` |
| `mail.mail_activity_plan_view_tree` | list |  | `name`, `res_model_id`, `steps_count`, `company_id` |  |  | `mail` |
| `mail.mail_activity_plan_view_tree_detailed` | xpath | `mail.mail_activity_plan_view_tree` |  |  |  | `mail` |
| `mail.mail_activity_plan_view_form` | form |  | `company_id`, `active`, `name`, `res_model`, `company_id`, `template_ids`, `company_id`, `note`, `sequence`, `activity_type_id`, `summary`, `responsible_type`, `responsible_id`, `delay_count`, `delay_unit`, `delay_from`, `next_activity_ids`, `icon`, `activity_type_id`, `summary`, `delay_count`, `delay_unit`, `delay_from`, `next_activity_ids`, `responsible_type`, `responsible_id` |  |  | `mail` |
| `mail.mail_activity_plan_view_kanban` | kanban |  | `name`, `res_model_id`, `steps_count` |  |  | `mail` |
| `mail.mail_activity_plan_view_form_fixed_model` | xpath | `mail.mail_activity_plan_view_form` |  |  |  | `mail` |
| `project.mail_activity_plan_view_form_project_and_task` | xpath | `mail.mail_activity_plan_view_form` |  |  |  | `project` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `crm.mail_activity_plan_action_lead` | Lead Activity Plans | list,kanban,form | `[('res_model', '=', 'crm.lead')]` | `{'default_res_model': 'crm.lead'}` |  | `crm` |
| `hr.mail_activity_plan_action` | Employee Plans | list,kanban,form | `[('res_model', '=', 'hr.employee'), '\|', ('company_id', 'in', allowed_company_ids), ('company_id', '=', False)]` | `{'default_res_model': 'hr.employee'}` |  | `hr` |
| `hr_recruitment.mail_activity_plan_action_config_hr_applicant` | Recruitment Plans | list,kanban,form | `[('res_model', '=', 'hr.applicant')]` | `{'default_res_model': 'hr.applicant'}` |  | `hr_recruitment` |
| `mail.mail_activity_plan_action` | Activity Plans | list,kanban,form |  |  |  | `mail` |
| `project.mail_activity_plan_action_config_project_task_plan` | Activity Plans | list,kanban,form | `[('res_model', 'in', ('project.project', 'project.task'))]` | `{'default_res_model': 'project.task'}` |  | `project` |
| `sale.mail_activity_plan_action_sale_order` | Sale Order Plans | list,kanban,form | `[('res_model', '=', 'sale.order')]` | `{'default_res_model': 'sale.order'}` |  | `sale` |

Machine-readable definition: `../../../schemas/data/entities/mail.activity.plan.json`; views: `../../../schemas/interfaces/views/mail.activity.plan.json`.
