# text message Templates (`sms.template`)

**Transport name:** `sms.template`  
**Storage name:** `sms_template`  
**Kind:** persistent entity (one table)  
**Defined by package:** `sms`  
**Extended by packages:** `event_sms`

Description: SMS Templates

## Identity and behavior

- Mixins (classical inheritance): `mail.render.mixin`, `template.reset.mixin`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (5)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | translatable |
| `model_id` | Applies to | many to one | `ir.model` | required; on delete of the target: cascade; restricted by domain `["&", ["is_mail_thread_sms", "=", true], ["transient", "=", false]]`; Help: The type of document this template can be used with |
| `model` | Related Document Model | single line text |  | read only; related through path `model_id.model` and stored; indexed |
| `body` | Body | single line text |  | required; translatable |
| `sidebar_action_id` | Sidebar action | many to one | `ir.actions.act_window` | read only; not copied on duplication; Help: Sidebar action to make this template available on records of the related document model |

## Operations (7)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `default_get` | lifecycle override | self, fields | `sms` | model |  |
| `_compute_render_model` | computation | self | `sms` | depends: `model` |  |
| `copy_data` | lifecycle override | self, default | `sms` |  |  |
| `unlink` | lifecycle override | self | `event_sms`, `sms` |  |  |
| `action_create_sidebar_action` | user action | self | `sms` |  |  |
| `action_unlink_sidebar_action` | user action | self | `sms` |  |  |
| `_search` | search rule | self, domain, *args, **kwargs | `event_sms` | model | Context-based hack to filter reference field in a m2o search box to emulate a domain the ORM currently does not support.  As we can not specify a domain on a reference field, we added a context key `filter_template_on_event` on the template reference field. If this key is set, we add our domain in the `domain` in the `_search` method to filtrate the SMS templates. |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `sales_team.group_sale_manager` | yes | yes | yes | yes | `crm_sms` |
| `event.group_event_manager` | yes | yes | yes | yes | `event_sms` |
| `hr.group_hr_manager` | yes | yes | yes | yes | `hr_presence` |
| `project.group_project_manager` | yes | yes | yes | yes | `project_sms` |
| `sales_team.group_sale_manager` | yes | yes | yes | yes | `sale_sms` |
| all internal users | no | no | no | no | `sms` |
| `base.group_user` | no | yes | no | no | `sms` |
| `base.group_system` | yes | yes | yes | yes | `sms` |
| `stock.group_stock_manager` | yes | yes | yes | yes | `stock_sms` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| SMS Template: sale manager CUD on opportunity / partner templates | `[(4, ref('sales_team.group_sale_manager'))]` | `[('model_id.model', 'in', ('crm.lead', 'res.partner'))]` | False | True | True | True |
| SMS Template: event manager CUD on event / registrations templates | `[(4, ref('event.group_event_manager'))]` | `[('model_id.model', 'in', ('event.event', 'event.registration'))]` | False | True | True | True |
| SMS Template: hr manager CUD on employee templates | `[(4, ref('hr.group_hr_manager'))]` | `[('model_id.model', '=', 'hr.employee')]` | False | True | True | True |
| SMS Template: project manager CUD on project/task | `[(4, ref('project.group_project_manager'))]` | `[('model', 'in', ('project.task', 'project.project'))]` | False | True | True | True |
| SMS Template: sale manager CUD on sale orders | `[(4, ref('sales_team.group_sale_manager'))]` | `[('model_id.model', 'in', ('sale.order', 'res.partner'))]` | False | True | True | True |
| SMS Template: system group granted all | `[(4, ref('base.group_system'))]` | `[(1, '=', 1)]` | True | True | True | True |
| SMS Template: stock manager CUD on stock picking templates | `[(4, ref('stock.group_stock_manager'))]` | `[('model_id.model', '=', 'stock.picking')]` | False | True | True | True |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `sms.sms_template_view_form` | form |  | `template_fs`, `sidebar_action_id`, `name`, `model_id`, `model`, `lang`, `body` | `Reset Template`, `action_create_sidebar_action`, `action_unlink_sidebar_action`, `%(sms_template_preview_action)d` |  | `sms` |
| `sms.sms_template_view_tree` | list |  | `name`, `model_id` |  |  | `sms` |
| `sms.sms_template_view_search` | search |  | `name`, `model_id` |  |  | `sms` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `sms.sms_template_action` | Templates | list,form |  |  |  | `sms` |

Machine-readable definition: `../../../schemas/data/entities/sms.template.json`; views: `../../../schemas/interfaces/views/sms.template.json`.
